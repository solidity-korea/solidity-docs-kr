.. index:: style, coding style

#############
스타일 가이드
#############

************
소개
************

이 가이드는 솔리디티 코드를 작성할 때 지켜야 할 코딩 규칙에 대해 설명합니다.
좀더 나은 규칙이 발견되면 기존의 규칙은 삭제될 수 있고 시간이 지나면서 가이드는 바뀔 수 있습니다.

많은 프로젝트들이 각각의 스타일 가이드를 만들 것입니다.
충돌하는 지점이 발생하면, 프로젝트의 스타일 가이드를 우선적으로 따릅니다.

이 가이드 권장사항 규격은 파이썬의 `pep8 style guide <https://www.python.org/dev/peps/pep-0008/>`_ 를 따왔습니다.

가이드의 목표는 솔리디티 코드를 작성할 때 최고의 방법을 알려드리는 것이 아닙니다.
가이드의 목표는 **일관성** 을 유지하는 것입니다.
파이썬의 `pep8 <https://www.python.org/dev/peps/pep-0008/#a-foolish-consistency-is-the-hobgoblin-of-little-minds>`_ 인용구는 이 철학을 잘 소개하고 있습니다.

    스타일 가이드는 일관성에 관한 것입니다. 일관성있는 스타일 가이드는 매우 중요합니다.
    일관성있는 프로젝트, 모듈, 함수 또한 매우 중요합니다.
    하지만 제일 중요한 것은 일관성을 가지지 않는 경우가 언제인지를 아는 것입니다.
    때때로 스타일 가이드가 적용되지 않을 때가 있습니다.
    의심을 갖고, 최선의 판단을 내리십시오.
    다른 예를 보고 제일 나은 방향을 결정하십시오. 그리고 물어보는 걸 주저하지 마세요!

***********
코드 레이아웃
***********


들여쓰기
===========

들여쓰기마다 4 스페이스를 사용합니다.

탭 or 스페이스
==============

스페이스를 선호합니다.

탭과 스페이스를 섞어 사용하는 것을 피하세요.

빈 줄
===========

상위 선언 괄호는 2개의 빈 줄을 사이로 둡니다.

Yes::

    contract A {
        ...
    }


    contract B {
        ...
    }


    contract C {
        ...
    }

No::

    contract A {
        ...
    }
    contract B {
        ...
    }

    contract C {
        ...
    }

컨트랙트 내의 선언 괄호는 1개의 빈 줄을 사이로 둡니다.

함수 스텁이나 추상 컨트랙트를 만드는 등 1줄 코드 사이에는 빈 줄을 생략합니다.

Yes::

    contract A {
        function spam() public;
        function ham() public;
    }


    contract B is A {
        function spam() public {
            ...
        }

        function ham() public {
            ...
        }
    }

No::

    contract A {
        function spam() public {
            ...
        }
        function ham() public {
            ...
        }
    }

.. _maximum_line_length:

최대 행 길이
===================

`PEP 8 recommendation <https://www.python.org/dev/peps/pep-0008/#maximum-line-length>`_ of 79 (or 99) 줄의
내용들을 숙지하시면 도움이 될 것입니다.

줄바꿈은 다음과 같은 가이드라인을 따르세요.

1. 첫 번째 인자는 여는 괄호 옆에 붙이지 않습니다.
2. 한 번의 들여쓰기를 사용합니다.
3. 각각의 인자는 각 줄마다 위치합니다.
4. 끝을 내는 :code:`);` 는 마지막 줄에 혼자 남겨질 수 있게 하세요.

함수 호출

Yes::

    thisFunctionCallIsReallyLong(
        longArgument1,
        longArgument2,
        longArgument3
    );

No::

    thisFunctionCallIsReallyLong(longArgument1,
                                  longArgument2,
                                  longArgument3
    );

    thisFunctionCallIsReallyLong(longArgument1,
        longArgument2,
        longArgument3
    );

    thisFunctionCallIsReallyLong(
        longArgument1, longArgument2,
        longArgument3
    );

    thisFunctionCallIsReallyLong(
    longArgument1,
    longArgument2,
    longArgument3
    );

    thisFunctionCallIsReallyLong(
        longArgument1,
        longArgument2,
        longArgument3);

값 할당

Yes::

    thisIsALongNestedMapping[being][set][to_some_value] = someFunction(
        argument1,
        argument2,
        argument3,
        argument4
    );

No::

    thisIsALongNestedMapping[being][set][to_some_value] = someFunction(argument1,
                                                                       argument2,
                                                                       argument3,
                                                                       argument4);

이벤트 정의와 구현

Yes::

    event LongAndLotsOfArgs(
        adress sender,
        adress recipient,
        uint256 publicKey,
        uint256 amount,
        bytes32[] options
    );

    LongAndLotsOfArgs(
        sender,
        recipient,
        publicKey,
        amount,
        options
    );

No::

    event LongAndLotsOfArgs(adress sender,
                            adress recipient,
                            uint256 publicKey,
                            uint256 amount,
                            bytes32[] options);

    LongAndLotsOfArgs(sender,
                      recipient,
                      publicKey,
                      amount,
                      options);

소스 파일 인코딩
====================

UTF-8 이나 ASCII 인코딩을 선호합니다.

임포트
=======

import 문법은 항상 파일 최상단에 위치합니다.

Yes::

    import "owned";


    contract A {
        ...
    }


    contract B is owned {
        ...
    }

No::

    contract A {
        ...
    }


    import "owned";


    contract B is owned {
        ...
    }

함수 순서
==================

순서를 지키면 호출 가능한 함수들이 무엇인지, 생성자나 실패 예외처리가 어디있는지를 쉽게 찾을 수 있게 도움을 줍니다.

함수들은 다음과 같은 순서로 묶습니다.

- 생성자
- (있다면) fallback function
- external
- public
- internal
- private

그룹 내에서는 ``constant`` 함수는 마지막에 두세요.

Yes::

    contract A {
        function A() public {
            ...
        }

        function() public {
            ...
        }

        // External functions
        // ...

        // External functions that are constant
        // ...

        // Public functions
        // ...

        // Internal functions
        // ...

        // Private functions
        // ...
    }

No::

    contract A {

        // External functions
        // ...

        // Private functions
        // ...

        // Public functions
        // ...

        function A() public {
            ...
        }

        function() public {
            ...
        }

        // Internal functions
        // ...
    }

표현식 내 공백
=========================

다음과 같은 상황일 때 여분의 공백을 두는 걸 피하세요:
소, 중, 대괄호 안 바로 옆에 붙은 경우. 단, 한 줄의 함수 선언은 예외

Yes::

    spam(ham[1], Coin({name: "ham"}));

No::

    spam( ham[ 1 ], Coin( { name: "ham" } ) );

Exception::

    function singleLine() public { spam(); }

콤마나 세미콜론 바로 전에 붙은 경우:

Yes::

    function spam(uint i, Coin coin) public;

No::

    function spam(uint i , Coin coin) public ;

값을 할당할 때나 다른 연산자와 정렬을 맞추려는 공백이 1개를 넘는 경우:

Yes::

    x = 1;
    y = 2;
    long_variable = 3;

No::

    x             = 1;
    y             = 2;
    long_variable = 3;

fallback 함수 안에 공백을 넣지 마세요:

Yes::

    function() public {
        ...
    }

No::

    function () public {
        ...
    }

구조 잡기
==================

컨트랙트, 라이브러리, 함수나 구조체의 내용을 담는 중괄호는 다음을 따라야 합니다:

* 선언부의 같은 줄에 괄호를 여세요
* 선언부와 같은 들여쓰기면서 고유한 마지막 한 줄에 괄호를 닫으세요
* 여는 괄호는 한 칸의 공백이 들어갑니다

Yes::

    contract Coin {
        struct Bank {
            address owner;
            uint balance;
        }
    }

No::

    contract Coin
    {
        struct Bank {
            address owner;
            uint balance;
        }
    }

같은 권장 형태가 ``if``, ``else``, ``while`` 그리고 ``for`` 구문을 구성할때도 쓰입니다.
특히 ``if``, ``while``, ``for`` 에서 소괄호로 쌓인 조건부의 경우 앞뒤로 공백 한 칸씩을 추가해야 합니다.

Yes::

    if (...) {
        ...
    }

    for (...) {
        ...
    }

No::

    if (...)
    {
        ...
    }

    while(...){
    }

    for (...) {
        ...;}

*한 줄* 과 하나의 구현체만 가지고 있는 경우엔 괄호를 생략할 수 있습니다.

Yes::

    if (x < 10)
        x += 1;

No::

    if (x < 10)
        someArray.push(Coin({
            name: 'spam',
            value: 42
        }));

``else`` 나 ``else if`` 구문을 가진 ``if`` 문의 경우 ``if`` 문의 닫는 괄호와 같은 줄에 ``else`` 문이 위치해야 합니다.
이것은 다른 블록 형태의 구조 규칙과는 예외입니다.

Yes::

    if (x < 3) {
        x += 1;
    } else if (x > 7) {
        x -= 1;
    } else {
        x = 5;
    }


    if (x < 3)
        x += 1;
    else
        x -= 1;

No::

    if (x < 3) {
        x += 1;
    }
    else {
        x -= 1;
    }

함수 선언
====================

짧은 함수 선언의 경우 함수 내용부의 여는 괄호는 선언부와 같은 줄에 위치합니다.
닫는 괄호는 함수 선언부와 같은 들여쓰기를 가집니다.
여는 괄호는 한 칸의 공백 뒤에 위치합니다.

Yes::

    function increment(uint x) public pure returns (uint) {
        return x + 1;
    }

    function increment(uint x) public pure onlyowner returns (uint) {
        return x + 1;
    }

No::

    function increment(uint x) public pure returns (uint)
    {
        return x + 1;
    }

    function increment(uint x) public pure returns (uint){
        return x + 1;
    }

    function increment(uint x) public pure returns (uint) {
        return x + 1;
        }

    function increment(uint x) public pure returns (uint) {
        return x + 1;}

생성자를 포함한 모든 함수에 대해 접근 제어자를 명시적으로 적습니다.

Yes::

    function explicitlyPublic(uint val) public {
        doSomething();
    }

No::

    function implicitlyPublic(uint val) {
        doSomething();
    }

함수의 접근 제어자는 커스텀 키워드보다 앞에 옵니다.

Yes::

    function kill() public onlyowner {
        selfdestruct(owner);
    }

No::

    function kill() onlyowner public {
        selfdestruct(owner);
    }

함수가 많은 인자를 갖고 있을 땐, 각 인자마다 같은 들여쓰기로 한 줄씩 작성하는 것을 권장합니다.
열고 닫는 중괄호 또한 같은 규칙으로 한 줄씩 작성합니다.

Yes::

    function thisFunctionHasLotsOfArguments(
        address a,
        address b,
        address c,
        address d,
        address e,
        address f
    )
        public
    {
        doSomething();
    }

No::

    function thisFunctionHasLotsOfArguments(address a, address b, address c,
        address d, address e, address f) public {
        doSomething();
    }

    function thisFunctionHasLotsOfArguments(address a,
                                            address b,
                                            address c,
                                            address d,
                                            address e,
                                            address f) public {
        doSomething();
    }

    function thisFunctionHasLotsOfArguments(
        address a,
        address b,
        address c,
        address d,
        address e,
        address f) public {
        doSomething();
    }

제어자를 가진 긴 함수의 경우 각 제어자는 각각의 줄을 가지며 작성합니다.

Yes::

    function thisFunctionNameIsReallyLong(address x, address y, address z)
        public
        onlyowner
        priced
        returns (address)
    {
        doSomething();
    }

    function thisFunctionNameIsReallyLong(
        address x,
        address y,
        address z,
    )
        public
        onlyowner
        priced
        returns (address)
    {
        doSomething();
    }

No::

    function thisFunctionNameIsReallyLong(address x, address y, address z)
                                          public
                                          onlyowner
                                          priced
                                          returns (address) {
        doSomething();
    }

    function thisFunctionNameIsReallyLong(address x, address y, address z)
        public onlyowner priced returns (address)
    {
        doSomething();
    }

    function thisFunctionNameIsReallyLong(address x, address y, address z)
        public
        onlyowner
        priced
        returns (address) {
        doSomething();
    }

많은 줄의 출력 파라미터와 리턴문은 :ref:`최대 행 길이 <maximum_line_length>` 섹션에서 언급된
긴 줄을 묶는 권장사항을 따라야 합니다.

Yes::

    function thisFunctionNameIsReallyLong(
        address a,
        address b,
        address c
    )
        public
        returns (
            address someAddressName,
            uint256 LongArgument,
            uint256 Argument
        )
    {
        doSomething()

        return (
            veryLongReturnArg1,
            veryLongReturnArg2,
            veryLongReturnArg3
        );
    }

No::

    function thisFunctionNameIsReallyLong(
        address a,
        address b,
        address c
    )
        public
        returns (address someAddressName,
                 uint256 LongArgument,
                 uint256 Argument)
    {
        doSomething()

        return (veryLongReturnArg1,
                veryLongReturnArg1,
                veryLongReturnArg1);
    }

인자가 필요한 상속 생성자의 경우, 선언부가 길거나 읽기 어려울 때
제어자와 같은 방식으로 새 줄에 기본 생성자를 떨어뜨려 놓는 것을 권장합니다.

Yes::

    contract A is B, C, D {
        function A(uint param1, uint param2, uint param3, uint param4, uint param5)
            B(param1)
            C(param2, param3)
            D(param4)
            public
        {
            // do something with param5
        }
    }

No::

    contract A is B, C, D {
        function A(uint param1, uint param2, uint param3, uint param4, uint param5)
        B(param1)
        C(param2, param3)
        D(param4)
        public
        {
            // do something with param5
        }
    }

    contract A is B, C, D {
        function A(uint param1, uint param2, uint param3, uint param4, uint param5)
            B(param1)
            C(param2, param3)
            D(param4)
            public {
            // do something with param5
        }
    }

단일문의 짧은 함수를 선언할 때 한 줄에 모두 작성할 수 있습니다.

허용::

    function shortFunction() public { doSomething(); }

함수 선언에 대한 이 가이드라인들은 가독성을 높이기 위함입니다.
이 가이드는 함수 선언의 모든 경우를 다루지는 않으므로 작성자는 최선의 판단을 내려야 합니다.

매핑
========

TODO

변수 선언
=====================

배열 변수를 선언할 때, 타입과 중괄호 사이 공백을 두지 않습니다.

Yes::

    uint[] x;

No::

    uint [] x;


기타 권장사항
=====================

* 문자열은 작은 따옴표보다 큰 따옴표를 사용합니다.

Yes::

    str = "foo";
    str = "Hamlet says, 'To be or not to be...'";

No::

    str = 'bar';
    str = '"Be yourself; everyone else is already taken." -Oscar Wilde';

* 연산자 사이에 공백을 둡니다.

Yes::

    x = 3;
    x = 100 / 10;
    x += 3 + 4;
    x |= y && z;

No::

    x=3;
    x = 100/10;
    x += 3+4;
    x |= y&&z;

* 우선순위를 나타내기 위해 우선순위가 더 높은 연산자는 공백을 제외할 수 있습니다.
  이로 인해 가독성을 늘릴 수 있으며, 연산자 양쪽에는 같은 공백을 사용해야 합니다.

Yes::

    x = 2**3 + 5;
    x = 2*y + 3*z;
    x = (a+b) * (a-b);

No::

    x = 2** 3 + 5;
    x = y+z;
    x +=1;


******************
명명 규칙
******************

명명 규칙은 넓게 사용될 때 더욱 강력합니다. 이를 통해 중요하고 많은 *메타* 정보들을 전달할 수 있습니다.
이 권장사항은 가독성을 높이기 위한 것으로, 규칙이 아니라 사물의 이름을 통해 정보를 전달하기 위함입니다.
마지막으로 코드의 일관성은 이 문서의 규칙들로 항상 대체할 수 있어야 합니다.

명명 스타일
=============

혼동을 줄이기 위한 예로, 다음 이름들은 모두 다른 명명 스타일을 가지고 있습니다.

* ``b`` (소문자 한 글자)
* ``B`` (대문자 한 글자)
* ``lowercase``
* ``lower_case_with_underscores``
* ``UPPERCASE``
* ``UPPER_CASE_WITH_UNDERSCORES``
* ``CapitalizedWords`` (or CapWords)
* ``mixedCase`` (첫 글자가 소문자로 CapitalizedWords와 다르다!)
* ``Capitalized_Words_With_Underscores``

.. note:: CapWords 스타일에서 이니셜을 사용할 땐 이니셜 모두를 대문자로 씁니다. 즉, HttpServerError 대신 HTTPServerError를 사용하는 것입니다. mixedCase 스타일에서 이니셜을 사용할 땐 이니셜을 대문자로 쓰되, 이름의 처음으로 쓰일 땐 소문자를 사용합니다. 즉, XMLHTTPRequest 보다 xmlHTTPRequest이 낫습니다.

피해야 할 명명법
==============

* ``l`` - 문자열 el 의 소문자
* ``O`` - 문자열 oh 의 대문자
* ``I`` - 문자열 eye의 대문자

한 글자 변수 이름은 사용하지 마세요. 종종 숫자 1과 0과도 구별하기 어려울 때가 있습니다.

컨트랙트와 라이브러리 이름
==========================

컨트랙트와 라이브러리는 CapWords 스타일을 사용합니다.
예: ``SimpleToken``, ``SmartBank``, ``CertificateHashRepository``, ``Player``.

구조체 이름
==========================

구조체는 CapWords 스타일을 사용합니다.
예: ``MyCoin``, ``Position``, ``PositionXY``.

이벤트 이름
===========

이벤트는 CapWords 스타일을 사용합니다.
예: ``Deposit``, ``Transfer``, ``Approval``, ``BeforeTransfer``, ``AfterTransfer``.

함수 이름
==============

생성자가 아닌 함수는 mixedCase 스타일을 사용합니다.
예: ``getBalance``, ``transfer``, ``verifyOwner``, ``addMember``, ``changeOwner``.

함수 인자 이름
=======================

함수 인자는 mixedCase 스타일을 사용합니다.
예: ``initialSupply``, ``account``, ``recipientAddress``, ``senderAddress``, ``newOwner``.

커스텀 구조체에서 동작하는 라이브러리 함수를 작성할 때,
구조체는 함수의 첫 번째 인자여야 하며 ``self`` 라는 이름을 가집니다.

지역, 상태 변수 이름
==============================

mixedCase 스타일을 사용합니다.
예: ``totalSupply``, ``remainingSupply``, ``balancesOf``, ``creatorAddress``, ``isPreSale``, ``tokenExchangeRate``.

상수
=========

상수는 밑줄과 함께 대문자를 사용합니다.
예: ``MAX_BLOCKS``, ``TOKEN_NAME``, ``TOKEN_TICKER``, ``CONTRACT_VERSION``.

제어자 이름
==============

mixedCase 스타일을 사용합니다.
예: ``onlyBy``, ``onlyAfter``, ``onlyDuringThePreSale``.

열거형
=====

열거형은 CapWords 스타일을 사용합니다.
예: ``TokenGroup``, ``Frame``, ``HashStyle``, ``CharacterLocation``.

명명 충돌의 방지
==========================

* ``single_trailing_underscore_``

이 규칙은 원하는 이름이 예약어와 충돌할 때 제안됩니다.

일반 권장사항
=======================

TODO
