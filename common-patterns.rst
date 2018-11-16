###############
자주 쓰이는 패턴
###############

.. index:: withdrawal

.. _withdrawal_pattern:

*************************
컨트랙트에서의 출금
*************************

effect 이후 기금송금에 있어 가장 권장하는 방법은
출금 패턴을 사용하는 것입니다. Effect의 결과로 Ether를 송금하는
가장 직관적인 방법은 직접 ``send`` 를 호출하는 것이겠지만,
잠재적인 보안위협을 초래 할 수 있으므로 권장하지 않습니다.
:ref:`security_consideration` 페이지에서 보안에 대해 더 알아 볼 수 있습니다. 


다음은 `King of the Ether <https://www.kingoftheether.com/>`_ 에서 
영감을 받아 작성된 "richest"가 되기 위해 가장 많은 돈을
컨트랙트로 송금하는 실제 출금패턴 예제입니다.

다음의 컨트랙트에서 당신이 "richest"를 빼앗긴다면, 새롭게 "richest"
가 된 사람으로부터 기금을 돌려 받을 것입니다.

::

    pragma solidity ^0.4.11;

    contract WithdrawalContract {
        address public richest;
        uint public mostSent;

        mapping (address => uint) pendingWithdrawals;

        function WithdrawalContract() public payable {
            richest = msg.sender;
            mostSent = msg.value;
        }

        function becomeRichest() public payable returns (bool) {
            if (msg.value > mostSent) {
                pendingWithdrawals[richest] += msg.value;
                richest = msg.sender;
                mostSent = msg.value;
                return true;
            } else {
                return false;
            }
        }

        function withdraw() public {
            uint amount = pendingWithdrawals[msg.sender];
            // Remember to zero the pending refund before
            // sending to prevent re-entrancy attacks
            pendingWithdrawals[msg.sender] = 0;
            msg.sender.transfer(amount);
        }
    }

다음은 직관적인 전송패턴과 정반대인 패턴입니다.

::

    pragma solidity ^0.4.11;

    contract SendContract {
        address public richest;
        uint public mostSent;

        function SendContract() public payable {
            richest = msg.sender;
            mostSent = msg.value;
        }

        function becomeRichest() public payable returns (bool) {
            if (msg.value > mostSent) {
                // This line can cause problems (explained below).
                richest.transfer(msg.value);
                richest = msg.sender;
                mostSent = msg.value;
                return true;
            } else {
                return false;
            }
        }
    }

본 예제에서, 반드시 알아둬야 할 것은, 공격자(해커)가 실패
fallback함수를 가진 컨트랙트 주소를 ``richest`` 로 만들어
컨트랙트를 할 수 없는 상태로 만들 수 있다는 점입니다.
(예를들어 ``revert()`` 를 사용하거나 단순히 2300개 이상의 가스를 소비시킴으로써)
그렇게하면, "poisoned" 컨트랙트에 기금을 ``transfer`` 하기위해 송금이
요청 될 때마다 컨트랙트와 더불어 ``becomeRichest`` 도 실패 할
것이고, 이는 컨트랙트가 영원히 진행되지 않게 만들 것입니다.

반대로, 첫번째 예제에서 "withdraw" 패턴을 사용한다면
공격자(해커)의 출금만 실패할 것이고, 나머지 컨트랙트는 제대로 동작 할 것입니다.

.. index:: access;restricting

******************
제한된 액세스
******************

제한된 액세스는 컨트랙트에서의 일반적인 패턴입니다.
알아둬야 할 것은, 다른사람이나 컴퓨터가 당신의
컨트랙트 상태의 내용을 읽는것을 결코 제한 할 수
없다는 것입니다. 암호화를 사용함으로써 컨트랙트를
더 읽기 어렵게 만들 수 있습니다. 하지만 컨트랙트가
데이터를 읽으려고 한다면, 다른 모든 사람들 또한
당신의 데이터를 읽을 수 있을것입니다.

**다른 컨트랙트들** 이 컨트랙트 상태를
읽지 못 하도록 권한을 제한 할 수 있습니다.
상태변수를 ``public`` 으로 선언하지 않는 한,
이 제한은 디폴트로 제공됩니다.

게다가, 컨트랙트 상태를 수정하거나 컨트랙트 함수를 호출
할 수 있는 사람을 제한 할 수 있습니다.
다음이 바로 본 섹션에 대한 내용입니다.

.. index:: function;modifier

**function modifiers** 를 사용하면 이런 제한을
매우 알아보기 쉽게 할 수 있습니다.

::

    pragma solidity ^0.4.11;

    contract AccessRestriction {
        // These will be assigned at the construction
        // phase, where `msg.sender` is the account
        // creating this contract.
        address public owner = msg.sender;
        uint public creationTime = now;

        // Modifiers can be used to change
        // the body of a function.
        // If this modifier is used, it will
        // prepend a check that only passes
        // if the function is called from
        // a certain address.
        modifier onlyBy(address _account)
        {
            require(msg.sender == _account);
            // Do not forget the "_;"! It will
            // be replaced by the actual function
            // body when the modifier is used.
            _;
        }

        /// Make `_newOwner` the new owner of this
        /// contract.
        function changeOwner(address _newOwner)
            public
            onlyBy(owner)
        {
            owner = _newOwner;
        }

        modifier onlyAfter(uint _time) {
            require(now >= _time);
            _;
        }

        /// Erase ownership information.
        /// May only be called 6 weeks after
        /// the contract has been created.
        function disown()
            public
            onlyBy(owner)
            onlyAfter(creationTime + 6 weeks)
        {
            delete owner;
        }

        // This modifier requires a certain
        // fee being associated with a function call.
        // If the caller sent too much, he or she is
        // refunded, but only after the function body.
        // This was dangerous before Solidity version 0.4.0,
        // where it was possible to skip the part after `_;`.
        modifier costs(uint _amount) {
            require(msg.value >= _amount);
            _;
            if (msg.value > _amount)
                msg.sender.send(msg.value - _amount);
        }

        function forceOwnerChange(address _newOwner)
            public
            costs(200 ether)
        {
            owner = _newOwner;
            // just some example condition
            if (uint(owner) & 0 == 1)
                // This did not refund for Solidity
                // before version 0.4.0.
                return;
            // refund overpaid fees
        }
    }

함수호출에 대한 액세스를 제한 할 수 있는 보다 특별한
방법에 대해서는 다음 예제에서 설명합니다.

.. index:: state machine

*************
상태 머신
*************

Contracts often act as a state machine, which means
that they have certain **stages** in which they behave
differently or in which different functions can
be called. A function call often ends a stage
and transitions the contract into the next stage
(especially if the contract models **interaction**).
It is also common that some stages are automatically
reached at a certain point in **time**.

An example for this is a blind auction contract which
starts in the stage "accepting blinded bids", then
transitions to "revealing bids" which is ended by
"determine auction outcome".

.. index:: function;modifier

Function modifiers can be used in this situation
to model the states and guard against
incorrect usage of the contract.

Example
=======

In the following example,
the modifier ``atStage`` ensures that the function can
only be called at a certain stage.

Automatic timed transitions
are handled by the modifier ``timeTransitions``, which
should be used for all functions.

.. note::
    **Modifier Order Matters**.
    If atStage is combined
    with timedTransitions, make sure that you mention
    it after the latter, so that the new stage is
    taken into account.

Finally, the modifier ``transitionNext`` can be used
to automatically go to the next stage when the
function finishes.

.. note::
    **Modifier May be Skipped**.
    This only applies to Solidity before version 0.4.0:
    Since modifiers are applied by simply replacing
    code and not by using a function call,
    the code in the transitionNext modifier
    can be skipped if the function itself uses
    return. If you want to do that, make sure
    to call nextStage manually from those functions.
    Starting with version 0.4.0, modifier code
    will run even if the function explicitly returns.

::

    pragma solidity ^0.4.11;

    contract StateMachine {
        enum Stages {
            AcceptingBlindedBids,
            RevealBids,
            AnotherStage,
            AreWeDoneYet,
            Finished
        }

        // This is the current stage.
        Stages public stage = Stages.AcceptingBlindedBids;

        uint public creationTime = now;

        modifier atStage(Stages _stage) {
            require(stage == _stage);
            _;
        }

        function nextStage() internal {
            stage = Stages(uint(stage) + 1);
        }

        // Perform timed transitions. Be sure to mention
        // this modifier first, otherwise the guards
        // will not take the new stage into account.
        modifier timedTransitions() {
            if (stage == Stages.AcceptingBlindedBids &&
                        now >= creationTime + 10 days)
                nextStage();
            if (stage == Stages.RevealBids &&
                    now >= creationTime + 12 days)
                nextStage();
            // The other stages transition by transaction
            _;
        }

        // Order of the modifiers matters here!
        function bid()
            public
            payable
            timedTransitions
            atStage(Stages.AcceptingBlindedBids)
        {
            // We will not implement that here
        }

        function reveal()
            public
            timedTransitions
            atStage(Stages.RevealBids)
        {
        }

        // This modifier goes to the next stage
        // after the function is done.
        modifier transitionNext()
        {
            _;
            nextStage();
        }

        function g()
            public
            timedTransitions
            atStage(Stages.AnotherStage)
            transitionNext
        {
        }

        function h()
            public
            timedTransitions
            atStage(Stages.AreWeDoneYet)
            transitionNext
        {
        }

        function i()
            public
            timedTransitions
            atStage(Stages.Finished)
        {
        }
    }
