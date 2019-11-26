.. index:: contract, state variable, function, event, struct, enum, function;modifier

.. _contract_structure:

***********************
컨트랙트의 구조
***********************

솔리디티의 컨트랙트는 객체-지향 언어의 클래스들와 유사합니다.
각각의 컨트랙트는 :ref:`structure-state-variables`, :ref:`structure-functions`,
:ref:`structure-function-modifiers`, :ref:`structure-events`, :ref:`structure-struct-types` and :ref:`structure-enum-types`의 선언들이 포함될 수 있습니다.
또한, 다른 컨트랙트를 상속할 수도 있습니다.

:ref:`libraries<libraries>`와 :ref:`interfaces<interfaces>`로 불리는 특별한 종류의 컨트랙트들도 있습니다.

이 섹션에서는 빠른 개요를 제공하며, :ref:`contracts<contracts>`섹션에서는 이보다 자세한 내용을 다룹니다.

.. _structure-state-variables:

상태 변수
===============

상태 변수는 값이 영구적으로 컨트랙트 스토리지에 저장되는 변수입니다.

::

    pragma solidity >=0.4.0 <0.6.0;

    contract SimpleStorage {
        uint storedData; // State variable
        // ...
    }

유효한 상태 변수의 타입은 :ref:`types`를, 선택 가능한 가시성(접근제어자)은 :ref:`visibility-and-getters`를 참고하십시오.

.. _structure-functions:

함수
=========

함수는 컨트랙트 내의 실행가능한 코드의 단위입니다.

::

    pragma solidity >=0.4.0 <0.6.0;

    contract SimpleAuction {
        function bid() public payable { // Function
            // ...
        }
    }

:ref:`function-calls`은 내부적으로 또는 외부적으로 발생할 수 있으며,
계약 내부 호출인지, 외부호출인지에 따라 다른 :ref:`가시성<visibility-and-getters>`을 지닙니다.

.. _structure-function-modifiers:

함수 수정자
==================

함수 수정자는 선언 방식으로 함수의 의미를 변경하기 위해 사용할 수 있습니다.
(컨트랙트 섹션의 :ref:`modifiers` 참고).

::

    pragma solidity >=0.4.22 <0.6.0;

    contract Purchase {
        address public seller;

        modifier onlySeller() { // Modifier
            require(
                msg.sender == seller,
                "Only seller can call this."
            );
            _;
        }

        function abort() public view onlySeller { // Modifier usage
            // ...
        }
    }

.. _structure-events:

이벤트
======

이벤트는 편리한 EVM 로깅 기능과의 인터페이스입니다.

::

    pragma solidity >=0.4.21 <0.6.0;

    contract SimpleAuction {
        event HighestBidIncreased(address bidder, uint amount); // Event

        function bid() public payable {
            // ...
            emit HighestBidIncreased(msg.sender, msg.value); // Triggering event
        }
    }

이벤트의 선언과 dapp 내에서 사용되는 법에 대해서는 컨트랙트 섹션의 :ref:`events`를 참고하세요.

.. _structure-struct-types:

구조체 타입
=============

구조체는 여러 변수를 묶을 수 있는 사용자 정의 타입입니다. (타입 섹션의 :ref:`structs`를 참고).

::

    pragma solidity >=0.4.0 <0.6.0;

    contract Ballot {
        struct Voter { // Struct
            uint weight;
            bool voted;
            address delegate;
            uint vote;
        }
    }

.. _structure-enum-types:

열거형(Enum) 타입
==========

열거형(Enum)은 사용자 정의 '상수 값'의 유한 집합을 만들 수 있습니다. (타입 섹션의 :ref:`enums`을 참고).

::

    pragma solidity >=0.4.0 <0.6.0;

    contract Purchase {
        enum State { Created, Locked, Inactive } // Enum
    }
