###############
자주 쓰이는 패턴
###############

.. index:: withdrawal

.. _withdrawal_pattern:

*************************
컨트랙트에서의 출금
*************************

Effect 이후 기금송금에 있어 가장 권장되는 방법은
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
            // 리엔트란시(re-entrancy) 공격을 예방하기 위해
            // 송금하기 전에 보류중인 환불을 0으로 기억해 두십시오.
            pendingWithdrawals[msg.sender] = 0;
            msg.sender.transfer(amount);
        }
    }

다음은 직관적인 송금패턴과 정반대인 패턴입니다.

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
                // 현재의 라인이 문제의 원인이 될 수 있습니다. (아래에서 설명됨)
                richest.transfer(msg.value);
                richest = msg.sender;
                mostSent = msg.value;
                return true;
            } else {
                return false;
            }
        }
    }

본 예제에서, 반드시 알아둬야 할 것은, 공격자가 실패
fallback함수를 가진 컨트랙트 주소를 ``richest`` 로 만들어
컨트랙트를 할 수 없는 상태로 만들 수 있다는 점입니다.
(예를들어 ``revert()`` 를 사용하거나 단순히 2300개 이상의 가스를 소비시킴으로써)
그렇게하면, "poisoned" 컨트랙트에 기금을 ``transfer`` 하기위해 송금이
요청 될 때마다 컨트랙트와 더불어 ``becomeRichest`` 도 실패 할
것이고, 이는 컨트랙트가 영원히 진행되지 않게 만들 것입니다.

반대로, 첫번째 예제에서 "withdraw" 패턴을 사용한다면
공격자의 출금만 실패할 것이고, 나머지 컨트랙트는 제대로 동작 할 것입니다.

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
        // 이것들은 건설단계에서 할당됩니다.
        // 여기서, `msg.sender` 는 
        // 이 계약을 생성하는 계정입니다.
        address public owner = msg.sender;
        uint public creationTime = now;

        // 수정자를 사용하여 함수의
        // 본문을 변경할 수 있습니다.
        // 이 수정자가 사용되면, 
        // 함수가 특정 주소에서 호출 된
        // 경우에만 통과하는 검사가
        // 추가됩니다.
        modifier onlyBy(address _account)
        {
            require(msg.sender == _account);
            // "_;" 를 깜빡하지 마세요! 수정자가
            // 사용 될 때, "_;"가 실제 함수
            // 본문으로 대체됩니다.
            _;
        }

        /// `_newOwner` 를 이 컨트랙트의
        /// 새 소유자로 만듭니다.
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

        /// 소유권 정보를 지우십시오.
        /// 컨트랙트가 생성된 후 6주가 
        /// 지나야 호출 될 수 있습니다.
        function disown()
            public
            onlyBy(owner)
            onlyAfter(creationTime + 6 weeks)
        {
            delete owner;
        }

        // 이 수정자는 함수 호출과 관련된
        // 특정 요금을 요구합니다.
        // 호출자가 너무 많은 금액을 송금했을시, 
        // 함수 본문 이후에만 환급이 됩니다.
        // 이는 솔리디티 0.4.0 이전의 버전에서는 위험하였습니다.
        // `_;` 이후의 부분은 스킵될 가능성이 있습니다.
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
            // 몇 가지의 예제 조건
            if (uint(owner) & 0 == 1)
                // 이는 솔리디티 0.4.0 이전의
                // 버전에서는 환불되지 않았습니다.
                return;
            // 초과 요금에 대한 환불
        }
    }

함수호출에 대한 액세스를 제한 할 수 있는 보다 특별한
방법에 대해서는 다음 예제에서 설명합니다.

.. index:: state machine

*************
상태 머신
*************

컨트랙트는 종종 상태머신인 것처럼 동작합니다. 다시 말해,
컨트랙트들이 다르게 동작하거나 다른 함수들에 의해 호출되는
어떠한 **단계** 를 가지고 있습니다.
함수 호출은 종종 단계를 끝내고 컨트랙트를 다음 단계로 전환
시킵니다. (특히 컨트랙트 모델이 **상호작용** 인 경우에)
또한, **시간** 의 특정 지점에서 일부 단계에 자동으로 도달
하는 것이 일반적입니다.

예를 들어 "블라인드 입찰을 수락하는" 단계에서 시작하여
"옥션 결과 결정"으로 끝나는 "공개 입찰"로 전환하는
블라인드 옥션 컨트랙트가 있습니다.

.. index:: function;modifier

이 상황에서 상태를 모델링하고 컨트랙트의 잘못된 사용을
방지하기 위해 함수 수정자를 사용할 수 있습니다.

예제
=======

다음의 예제에서,
수정자 ``atStage`` 는 함수가 특정 단계에서만
호출되도록 보장해 줍니다.

자동 timed transitions 는
모든 함수에서 사용되는 수정자 ``timeTransitions``
에 의해 처리됩니다.

.. 알아두기::
    **수정자 주문 관련 사항**.
    atStage 수정자가 timedTransitions 수정자와
    결합된다면, 후미에 반드시 이 결합에 대해 언급하여,
    새로운 단계가 고려되도록 하십시오.

마지막으로, 수정자 ``transitionNext`` 는 함수가 끝났을 때,
자동적으로 다음 단계로 넘어가도록 하기 위해 사용될 수 있습니다.

.. 알아두기::
    **수정자는 스킵 될 수 있습니다**.
    이는 솔리디티 0.4.0 이전 버전에서만 적용됩니다:
    수정자는 단순히 코드를 교체하고 함수 호출을 사용하지
    않음으로써 적용되므로, 함수 자체에서 return을
    사용하면, transitionNext 수정자의 코드를 스킵
    할 수 있습니다. 이를 사용하고 싶다면, 반드시
    그 함수들에서 nextStage를 수동으로 호출해야 합니다.
    0.4.0 버전 부터는 함수가 명시적으로 return 되는
    경우에도 수정자 코드가 실행됩니다.

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

        // 이 부분이 현재의 단계입니다.
        Stages public stage = Stages.AcceptingBlindedBids;

        uint public creationTime = now;

        modifier atStage(Stages _stage) {
            require(stage == _stage);
            _;
        }

        function nextStage() internal {
            stage = Stages(uint(stage) + 1);
        }

        // timed transitions를 수행하십시오. 
        // 이 수정자를 먼저 언급해야 합니다. 그렇지 않으면,
        // Guards가 새로운 단계를 고려하지 않을 것 입니다.
        modifier timedTransitions() {
            if (stage == Stages.AcceptingBlindedBids &&
                        now >= creationTime + 10 days)
                nextStage();
            if (stage == Stages.RevealBids &&
                    now >= creationTime + 12 days)
                nextStage();
            // 다른 단계는 거래에 의해 전환됩니다.
            _;
        }

        // 수정자의 순서가 중요합니다!
        function bid()
            public
            payable
            timedTransitions
            atStage(Stages.AcceptingBlindedBids)
        {
            // 우리는 여기서 그것을 구현하지 않을 것입니다.
        }

        function reveal()
            public
            timedTransitions
            atStage(Stages.RevealBids)
        {
        }

        // 이 수정자는 함수가 완료된 후
        // 다음 단계로 이동합니다.
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
