.. _security_considerations:

#######################
보안 측면 고려사항
#######################

소프트웨어를 원하는대로 개발하는 것은 어렵지 않지만, 다른 사람이 아예 다른 의도로 작동시키는걸 막는 것은 어렵습니다.

솔리디티에서는 모든 스마트 컨트랙트가 공개적으로 실행되고 대부분의 소스코드를 확인할 수 있는 경우가 많습니다. 따라서 이런 보안 측면의 고려사항은 특히 더 중요합니다.

물론 보안에 얼마나 신경을 써야하는 지는 상황에 따라 다릅니다. 가령, 웹 서비스는 공격자를 포함한 대중에게 공개되어야하고, 누구나 접근할 수 있어야하며, 어떤 때에는 오픈소스로 관리되는 경우도 있습니다. 만약 웹 서비스에 중요치 않은 정보만 저장한다면 문제가 되지 않지만, 은행 계좌와 같은 정보를 관리한다면 더욱 조심할 필요가 있죠.

이 장에서는 조심해야할 문제들과 일반적인 보안관련 패턴들을 다룹니다. 하지만 이는 완벽한 해결법이 아닙니다. 즉, 스마트 컨트랙트 상에는 버그가 없더라도, 컴파일러나 플랫폼 자체에 버그가 있을 수 있다는 얘기죠.

언제나 그렇듯이, 이 문서는 오픈 소스 기반의 문서이기 때문에, 보안에 대한 문제가 생긴다면 주저없이 내용을 추가해주시기 바랍니다.

********
유의사항
********

개인 정보와 무작위성
==================================

스마트 컨트랙트 상의 모든 정보는 공개적으로 보여집니다. 심지어 지역 변수 및 상태 변수가 ``private``으로 선언되었다고해도 마찬가지죠.

만약 당신이 채굴자의 부정 행위를 막고자 한다면, 난수를 생성하는 것이 어느정도 유용할 수 있습니다.

Re-Entrancy
===========

Any interaction from a contract (A) with another contract (B) and any transfer
of Ether hands over control to that contract (B). This makes it possible for B
to call back into A before this interaction is completed. To give an example,
the following code contains a bug (it is just a snippet and not a
complete contract):

::

    pragma solidity ^0.4.0;

    // THIS CONTRACT CONTAINS A BUG - DO NOT USE
    contract Fund {
        /// Mapping of ether shares of the contract.
        mapping(address => uint) shares;
        /// Withdraw your share.
        function withdraw() public {
            if (msg.sender.send(shares[msg.sender]))
                shares[msg.sender] = 0;
        }
    }

The problem is not too serious here because of the limited gas as part
of ``send``, but it still exposes a weakness: Ether transfer can always
include code execution, so the recipient could be a contract that calls
back into ``withdraw``. This would let it get multiple refunds and
basically retrieve all the Ether in the contract. In particular, the
following contract will allow an attacker to refund multiple times
as it uses ``call`` which forwards all remaining gas by default:

::

    pragma solidity ^0.4.0;

    // THIS CONTRACT CONTAINS A BUG - DO NOT USE
    contract Fund {
        /// Mapping of ether shares of the contract.
        mapping(address => uint) shares;
        /// Withdraw your share.
        function withdraw() public {
            if (msg.sender.call.value(shares[msg.sender])())
                shares[msg.sender] = 0;
        }
    }

To avoid re-entrancy, you can use the Checks-Effects-Interactions pattern as
outlined further below:

::

    pragma solidity ^0.4.11;

    contract Fund {
        /// Mapping of ether shares of the contract.
        mapping(address => uint) shares;
        /// Withdraw your share.
        function withdraw() public {
            var share = shares[msg.sender];
            shares[msg.sender] = 0;
            msg.sender.transfer(share);
        }
    }

Note that re-entrancy is not only an effect of Ether transfer but of any
function call on another contract. Furthermore, you also have to take
multi-contract situations into account. A called contract could modify the
state of another contract you depend on.

Gas Limit and Loops
===================

Loops that do not have a fixed number of iterations, for example, loops that depend on storage values, have to be used carefully:
Due to the block gas limit, transactions can only consume a certain amount of gas. Either explicitly or just due to
normal operation, the number of iterations in a loop can grow beyond the block gas limit which can cause the complete
contract to be stalled at a certain point. This may not apply to ``constant`` functions that are only executed
to read data from the blockchain. Still, such functions may be called by other contracts as part of on-chain operations
and stall those. Please be explicit about such cases in the documentation of your contracts.

Sending and Receiving Ether
===========================

- Neither contracts nor "external accounts" are currently able to prevent that someone sends them Ether.
  Contracts can react on and reject a regular transfer, but there are ways
  to move Ether without creating a message call. One way is to simply "mine to"
  the contract address and the second way is using ``selfdestruct(x)``.

- If a contract receives Ether (without a function being called), the fallback function is executed.
  If it does not have a fallback function, the Ether will be rejected (by throwing an exception).
  During the execution of the fallback function, the contract can only rely
  on the "gas stipend" (2300 gas) being available to it at that time. This stipend is not enough to access storage in any way.
  To be sure that your contract can receive Ether in that way, check the gas requirements of the fallback function
  (for example in the "details" section in Remix).

- There is a way to forward more gas to the receiving contract using
  ``addr.call.value(x)()``. This is essentially the same as ``addr.transfer(x)``,
  only that it forwards all remaining gas and opens up the ability for the
  recipient to perform more expensive actions (and it only returns a failure code
  and does not automatically propagate the error). This might include calling back
  into the sending contract or other state changes you might not have thought of.
  So it allows for great flexibility for honest users but also for malicious actors.

- If you want to send Ether using ``address.transfer``, there are certain details to be aware of:

  1. If the recipient is a contract, it causes its fallback function to be executed which can, in turn, call back the sending contract.
  2. Sending Ether can fail due to the call depth going above 1024. Since the caller is in total control of the call
     depth, they can force the transfer to fail; take this possibility into account or use ``send`` and make sure to always check its return value. Better yet,
     write your contract using a pattern where the recipient can withdraw Ether instead.
  3. Sending Ether can also fail because the execution of the recipient contract
     requires more than the allotted amount of gas (explicitly by using ``require``,
     ``assert``, ``revert``, ``throw`` or
     because the operation is just too expensive) - it "runs out of gas" (OOG).
     If you use ``transfer`` or ``send`` with a return value check, this might provide a
     means for the recipient to block progress in the sending contract. Again, the best practice here is to use
     a :ref:`"withdraw" pattern instead of a "send" pattern <withdrawal_pattern>`.

Callstack Depth
===============

External function calls can fail any time because they exceed the maximum
call stack of 1024. In such situations, Solidity throws an exception.
Malicious actors might be able to force the call stack to a high value
before they interact with your contract.

Note that ``.send()`` does **not** throw an exception if the call stack is
depleted but rather returns ``false`` in that case. The low-level functions
``.call()``, ``.callcode()`` and ``.delegatecall()`` behave in the same way.

tx.origin
=========

Never use tx.origin for authorization. Let's say you have a wallet contract like this:

::

    pragma solidity ^0.4.11;

    // THIS CONTRACT CONTAINS A BUG - DO NOT USE
    contract TxUserWallet {
        address owner;

        function TxUserWallet() public {
            owner = msg.sender;
        }

        function transferTo(address dest, uint amount) public {
            require(tx.origin == owner);
            dest.transfer(amount);
        }
    }

Now someone tricks you into sending ether to the address of this attack wallet:

::

    pragma solidity ^0.4.11;

    interface TxUserWallet {
        function transferTo(address dest, uint amount) public;
    }

    contract TxAttackWallet {
        address owner;

        function TxAttackWallet() public {
            owner = msg.sender;
        }

        function() public {
            TxUserWallet(msg.sender).transferTo(owner, msg.sender.balance);
        }
    }

If your wallet had checked ``msg.sender`` for authorization, it would get the address of the attack wallet, instead of the owner address. But by checking ``tx.origin``, it gets the original address that kicked off the transaction, which is still the owner address. The attack wallet instantly drains all your funds.


Minor Details
=============

- In ``for (var i = 0; i < arrayName.length; i++) { ... }``, the type of ``i`` will be ``uint8``, because this is the smallest type that is required to hold the value ``0``. If the array has more than 255 elements, the loop will not terminate.
- The ``constant`` keyword for functions is currently not enforced by the compiler.
  Furthermore, it is not enforced by the EVM, so a contract function that "claims"
  to be constant might still cause changes to the state.
- Types that do not occupy the full 32 bytes might contain "dirty higher order bits".
  This is especially important if you access ``msg.data`` - it poses a malleability risk:
  You can craft transactions that call a function ``f(uint8 x)`` with a raw byte argument
  of ``0xff000001`` and with ``0x00000001``. Both are fed to the contract and both will
  look like the number ``1`` as far as ``x`` is concerned, but ``msg.data`` will
  be different, so if you use ``keccak256(msg.data)`` for anything, you will get different results.

***************
Recommendations
***************

Restrict the Amount of Ether
============================

Restrict the amount of Ether (or other tokens) that can be stored in a smart
contract. If your source code, the compiler or the platform has a bug, these
funds may be lost. If you want to limit your loss, limit the amount of Ether.

Keep it Small and Modular
=========================

Keep your contracts small and easily understandable. Single out unrelated
functionality in other contracts or into libraries. General recommendations
about source code quality of course apply: Limit the amount of local variables,
the length of functions and so on. Document your functions so that others
can see what your intention was and whether it is different than what the code does.

Use the Checks-Effects-Interactions Pattern
===========================================

Most functions will first perform some checks (who called the function,
are the arguments in range, did they send enough Ether, does the person
have tokens, etc.). These checks should be done first.

As the second step, if all checks passed, effects to the state variables
of the current contract should be made. Interaction with other contracts
should be the very last step in any function.

Early contracts delayed some effects and waited for external function
calls to return in a non-error state. This is often a serious mistake
because of the re-entrancy problem explained above.

Note that, also, calls to known contracts might in turn cause calls to
unknown contracts, so it is probably better to just always apply this pattern.

Include a Fail-Safe Mode
========================

While making your system fully decentralised will remove any intermediary,
it might be a good idea, especially for new code, to include some kind
of fail-safe mechanism:

You can add a function in your smart contract that performs some
self-checks like "Has any Ether leaked?",
"Is the sum of the tokens equal to the balance of the contract?" or similar things.
Keep in mind that you cannot use too much gas for that, so help through off-chain
computations might be needed there.

If the self-check fails, the contract automatically switches into some kind
of "failsafe" mode, which, for example, disables most of the features, hands over
control to a fixed and trusted third party or just converts the contract into
a simple "give me back my money" contract.


*******************
Formal Verification
*******************

Using formal verification, it is possible to perform an automated mathematical
proof that your source code fulfills a certain formal specification.
The specification is still formal (just as the source code), but usually much
simpler.

Note that formal verification itself can only help you understand the
difference between what you did (the specification) and how you did it
(the actual implementation). You still need to check whether the specification
is what you wanted and that you did not miss any unintended effects of it.
