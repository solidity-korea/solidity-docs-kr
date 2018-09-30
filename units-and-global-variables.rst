**************************************
단위 및 전역 변수
**************************************

.. index:: wei, finney, szabo, ether

이더 단위
===========

Ether를 더 작은 단위로 변환하기 위해 숫자리터럴 뒤에 ``wei``, ``finney``, ``szabo``, ``ether`` 라는 접미사가 붙을 수 있습니다.
Ether통화를 나타내는 숫자리터럴에 접미사가 붙지 않으면 Wei가 붙어있다고 간주합니다.
예. ``2 ether == 2000 finney`` 는 ``true`` 로 평가됩니다.

.. index:: time, seconds, minutes, hours, days, weeks, years

시간 단위
==========

숫자리터럴 뒤에 붙는 ``seconds``, ``minutes``, ``hours``, ``days``, ``weeks``, ``years`` 와 같은 접미사는
시간 단위를 변환하는데 사용될 수 있으며 기본 단위는 seconds이고 다음과 같이 변환됩니다:

 * ``1 == 1 seconds``
 * ``1 minutes == 60 seconds``
 * ``1 hours == 60 minutes``
 * ``1 days == 24 hours``
 * ``1 weeks == 7 days``
 * ``1 years == 365 days``

이 단위들을 사용해 달력에서 계산을 할 땐 주의가 필요합니다.
왜냐하면 `윤초 <https://en.wikipedia.org/wiki/Leap_second>`_ 로 인해
모든 1 years가 항상 365 days와 동일한건 아니며 모든 1 days가 항상 24 hours와 동일한건 아니니까요.
윤초는 예측할 수 없기 때문에, 정확한 달력 라이브러리는 신뢰할수 있는 외부로부터 업데이트 되어야 합니다.

이 접미사들을 변수 뒤에 붙일순 없습니다.
만약 days를 포함하는 입력변수를 반복하고싶다면 다음과 같은 방식으로 할 수 있습니다::

    function f(uint start, uint daysAfter) public {
        if (now >= start + daysAfter * 1 days) {
          // ...
        }
    }

특수한 변수와 함수
===============================

전역 네임스페이스에는(global namespace) 특수한 변수와 함수가 존재하며 이들은 주로 블록체인에 관한 정보를 제공하는데 사용됩니다.

.. index:: block, coinbase, difficulty, number, block;number, timestamp, block;timestamp, msg, data, gas, sender, value, now, gas price, origin

블록 및 트랜잭션 속성
--------------------------------

- ``block.blockhash(uint blockNumber) returns (bytes32)``: 주어진 블록의 해시 - 현재 블록을 제외한 가장 최근 256개의 블록에 대해서만 작동함
- ``block.coinbase`` (``address``): 현재 블록 채굴자의 address
- ``block.difficulty`` (``uint``): 현재 블록 난이도
- ``block.gaslimit`` (``uint``): 현재 블록 gaslimit
- ``block.number`` (``uint``): 현재 블록 번호
- ``block.timestamp`` (``uint``): unix epoch 이후의 현재 블록 타임스탬프
- ``gasleft() returns (uint256)``: 잔여 가스
- ``msg.data`` (``bytes``): 완전한 calldata
- ``msg.gas`` (``uint``): 잔여 가스 -  0.4.21버전에서 제거되었으며 ``gasleft()`` 로 대체됨
- ``msg.sender`` (``address``): 메세지 발신자 (현재 호출)
- ``msg.sig`` (``bytes4``): calldata의 첫 4바이트(즉, 함수 식별자)
- ``msg.value`` (``uint``): 메세지와 함께 전송된 wei 수
- ``now`` (``uint``): 현재 블록 타임스탬프(``block.timestamp`` 의 별칭)
- ``tx.gasprice`` (``uint``): 트랜잭션의 가스 가격
- ``tx.origin`` (``address``): 트랜잭션의 발신자 (full call chain)

.. note::
    ``msg.sender`` 와 ``msg.value`` 를 포함한 모든 ``msg`` 의 멤버 값은 **외부** 햠수 호출에 의해 바뀔 수 있습니다.
    외부함수 호출에는 라이브러리 함수 호출도 포함됩니다.

.. note::
    랜덤한 값을 선택하기 위해 ``block.timestamp``, ``now``, ``block.blockhash`` 를 사용하지 마세요.

    채굴 커뮤니티의 악의적인 행위를 예로 들어보자면 특정 해시에서 casino payout function을 실행시키고 
    돈을 받지 못한다면 다시 다른 해시에 대해서 동일한 행위를 시도하는것이 가능합니다.

    현재 블록의 타임스탬프는 마지막 블록의 타임스태프보다 무조건적으로 커야 합니다만,
    유일하게 보장되는건, 현재 블록의 타임스탬프는 canonical chain의 연속된 두 블록의 타임스탬프 사이에 있을것이라는 것입니다.

.. note::
    확장성 문제로 인해 모든 블록의 블록해시가 사용가능한것은 아닙니다.
    가장 최근 256개 블록의 해시에 대해서만 접근이 가능하며 다른 해시값은 0이 됩니다.

.. index:: assert, revert, require

에러 처리
--------------

``assert(bool condition)``:
    조건이 충족되지 않으면 예외를 발생시킵니다 - 내부 에러에 사용됩니다.

``require(bool condition)``:
    조건이 충족되지 않으면 예외를 발생시킵니다 - 입력 또는 외부 요소의 에러에 사용됩니다.

``revert()``:
    실행을 중단하고 변경된 상태를 되돌립니다.

.. index:: keccak256, ripemd160, sha256, ecrecover, addmod, mulmod, cryptography,

수학 및 암호화 함수
----------------------------------------

``addmod(uint x, uint y, uint k) returns (uint)``:
    compute ``(x + y) % k`` where the addition is performed with arbitrary precision and does not wrap around at ``2**256``. Assert that ``k != 0`` starting from version 0.5.0.
``mulmod(uint x, uint y, uint k) returns (uint)``:
    compute ``(x * y) % k`` where the multiplication is performed with arbitrary precision and does not wrap around at ``2**256``. Assert that ``k != 0`` starting from version 0.5.0.
``keccak256(...) returns (bytes32)``:
    :ref:`(tightly packed) 인자 <abi_packed_mode>` 의 Ethereum-SHA-3 (Keccak-256) 해시를 계산합니다.
``sha256(...) returns (bytes32)``:
    :ref:`(tightly packed) 인자 <abi_packed_mode>` 의 SHA-256 해시를 계산합니다
``sha3(...) returns (bytes32)``:
    ``keccak256`` 의 별칭
``ripemd160(...) returns (bytes20)``:
    :ref:`(tightly packed) 인자 <abi_packed_mode>` 의 RIPEMD-160 해시를 계산합니다 
``ecrecover(bytes32 hash, uint8 v, bytes32 r, bytes32 s) returns (address)``:
    타원 곡선 서명으로부터 address와 연관된 공개 키를 복구하며 오류시엔 0을 반환합니다.
    (`사용 예시 <https://ethereum.stackexchange.com/q/1777/222>`_)

위에서 "tightly packed"는 인자가 패딩(padding)없이 연결됨을 의미합니다.
이것은 다음이 모두 동일하다는 것을 의미합니다::

    keccak256("ab", "c")
    keccak256("abc")
    keccak256(0x616263)
    keccak256(6382179)
    keccak256(97, 98, 99)

패딩이 필요하다면, 명시적 형변환을 통해 패딩을 할 수 있습니다:
``keccak256("\x00\x12")`` 는 ``keccak256(uint16(0x12))`` 와 동일합니다.

상수는 상수를 저장하는데 필요한 최소 바이트 수만을 사용하여 압축됩니다.
``keccak256(0) == keccak256(uint8(0))`` 와 ``keccak256(0x12345678) == keccak256(uint32(0x12345678))`` 는 이에 대한 예시입니다.

*프라이빗 블록체인* 에서 ``sha256``, ``ripemd160`` 또는 ``ecrecover`` 를 실행할때 Out-of-Gas상태가 될 수 있습니다.
그 이유는 이러한 것들이 precompiled contracts라고 불리우는 형태로 구현되었기 때문입니다.
그리고 이런 컨트랙트는 오직 그들이 첫번째 메세지를 받은 이후에만 실제로 존재합니다(비록 컨트랙트 코드가 하드코딩되었을지라도).
존재하지 않는 컨트랙트로 메세지를 보내는건 훨씬 비싸며 그렇기에 Out-of-Gas 에러를 발생시킵니다.
이 문제에 대한 해결책은 실제 컨트랙트를 사용하기 전에 1 wei를 각 컨트랙트에 전송해보는것입니다.
이렇게 해도 문제될건 없습니다.

.. index:: balance, send, transfer, call, callcode, delegatecall
.. _address_related:

Address 관련
---------------

``<address>.balance`` (``uint256``):
    :ref:`address` 의 잔액(Wei 단위)
``<address>.transfer(uint256 amount)``:
    주어진 양만큼의 Wei를 :ref:`address` 로 전송합니다. 실패시 에러를 발생시키고 2300 gas를 전달하며 이 값은 변경할 수 없습니다.
``<address>.send(uint256 amount) returns (bool)``:
    주어진 양만큼의 Wei를 :ref:`address` 로 전송합니다. 실패시 ``false`` 를 반환하고 2300 gas를 전달하며 이 값은 변경할 수 없습니다.
``<address>.call(...) returns (bool)``:
    로우 레벨 수준에서의 ``CALL`` 을 수행합니다. 실패시 ``false`` 를 반환하고 모든 gas를 전달하며 이 값은 변경가능합니다.
``<address>.callcode(...) returns (bool)``:
    로우 레벨 수준에서의 ``CALLCODE`` 를 수행합니다. 실패시 ``false`` 를 반환하고 모든 gas를 전달하며 이 값은 변경가능합니다.
``<address>.delegatecall(...) returns (bool)``:
    로우 레벨 수준에서의 ``DELEGATECALL`` 을 수행합니다. 실패시 ``false`` 를 반환하고 모든 gas를 전달하며 이 값은 변경가능합니다.

자세한 내용은 :ref:`address` 섹션을 참조하십시오.

.. warning::
    ``send`` 를 사용할 땐 몇가지 주의사항이 있습니다: call stack의 깊이가 1024라면 전송은 실패하며(이것은 항상 호출자에 의해 강제 될 수 있습니다) 그리고
    수신자의 gas가 전부 소모되어도 실패합니다. 그러므로 안전한 Ether 전송을 위해서, 항상 ``send`` 의 반환값을 확인하고, ``transfer`` 를 사용하세요: 혹은 더 좋은 방법은 수신자가 돈을 인출하는 패턴을 사용하는 것입니다. 

.. note::
    ``callcode`` 의 사용은 권장되지 않으며 추후 제거 될 예정입니다.

.. index:: this, selfdestruct

컨트랙트 관련
----------------

``this`` (current contract's type):
    현재의 컨트랙트, :ref:`address` 로 명시적 변환 가능합니다.


``selfdestruct(address recipient)``:
    현재의 컨트랙트를 파기하고 자금을 주어진 :ref:`address` 로 전송합니다.

``suicide(address recipient)``:
    ``selfdestruct`` 의 별칭

뿐만 아니라, 현재 컨트랙트의 모든 함수는 직접적으로 호출 가능합니다.
