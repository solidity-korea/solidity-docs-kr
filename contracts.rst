.. index:: ! contract

.. _contracts:

##########
컨트랙트
##########

Contracts in Solidity are similar to classes in object-oriented languages. They
contain persistent data in state variables and functions that can modify these
variables. Calling a function on a different contract (instance) will perform
an EVM function call and thus switch the context such that state variables are
inaccessible.

솔리디티 안에서 컨트랙트는 객체지향 언어의 클래스들과 비슷합니다. 컨트랙트는 상태 변수 안에 지속적인 데이터를 
포함할 수 있으며, 이러한 변수들을 수정할 수 있는 함수들을 포함할 수 있습니다. 다른 컨트랙트(인스턴스)의 함수를 
호출하는 것은 EVM 함수콜을 시행할 것이고, 컨텍스트의 스위치가 일어나 상태 변수에 접근할 수 없게 된다.

.. index:: ! contract;creation, constructor

******************
컨트랙트 생성하기
******************

Contracts can be created "from outside" via Ethereum transactions or from within Solidity contracts.
컨트랙트는 이더리움 트랜젝션 혹은 솔리디티 컨트랙트를 통하여 "밖으로 부터" 생성이 될 수 있다.

IDEs, such as `Remix <https://remix.ethereum.org/>`_, make the creation process seamless using UI elements.
`Remix <https://remix.ethereum.org/>`_와 같은 IDE들은 UI 요소들을 이용하여 통합된 생성 프로세스를 제공한다.

Creating contracts programmatically on Ethereum is best done via using the JavaScript API `web3.js <https://github.com/ethereum/web3.js>`_.
It has a function called `web3.eth.Contract <https://web3js.readthedocs.io/en/1.0/web3-eth-contract.html#new-contract>`_
to facilitate contract creation.
프로그램적으로 이더리움에 스마트컨트랙트를 생성하는 제일 좋은 방법은 Javascript API인 `web3.js <https://github.com/ethereum/web3.js>`_를 이용하는 것이다.
`web3.eth.Contract <https://web3js.readthedocs.io/en/1.0/web3-eth-contract.html#new-contract>`_라는 함수를 통하여 스마트컨트랙트 생성을 촉진한다.

When a contract is created, its constructor_  (a function declared with the ``constructor`` keyword) is executed once.
컨트랙트가 생성되면, 그것의 constructor_ (``constructor`` 키워드로 지정된 함수)가 한 번 실행된다.

A constructor is optional. Only one constructor is allowed, which means
overloading is not supported.
생성자는 선택적이다. 하나의 생성자만 허용된다. 즉 다시 말해 오버로딩은 지원되지 않는다.

After the constructor has executed, the final code of the contract is deployed to the
blockchain. This code includes all public and external functions and all functions
that are reachable from there through function calls. The deployed code does not
include the constructor code or internal functions only called from the constructor.
생성자가 실행 된 이후, 컨트랙트의 최종적인 코드가 블록체인에 배포된다. 이 코느는 모든 퍼블릭과 익스터널 함수들을 포함하며
모든 함수들은 함수 콜을 통해서 접근 가능하다. 배포된 코드는 생성자 코드나 생성자로부터 호출되는 내부 함수를 포함하지 않는다.

.. index:: constructor;arguments

Internally, constructor arguments are passed :ref:`ABI encoded <ABI>` after the code of
the contract itself, but you do not have to care about this if you use ``web3.js``.
내적으로, 스마트컨트랙트 그 자체의 코드 다음에 생성자 인자들은 :ref:`ABI encoded <ABI>` 를 통해 전달된다.
그러나,  ``web3.js``를 사용한다면 신경을 쓸 필요는 없다.

If a contract wants to create another contract, the source code
(and the binary) of the created contract has to be known to the creator.
This means that cyclic creation dependencies are impossible.
만약에 컨트랙트가 다른 컨트랙트를 생성하길 원한다면, 생성된 컨트랙트의 소스코드 (그리고 바이너리)를 생성자가 알고 있어야한다.
사이클을 갖고 있는 의존성 있는 생성은 불가능하다는 것을 의미한다.

::

    pragma solidity >=0.4.22 <0.6.0;

    contract OwnedToken {
        // TokenCreator is a contract type that is defined below.
        // It is fine to reference it as long as it isnot used
        // to create a new contract.
        // TokenCreator는 밑에 정의된 컨트랙트 타입이다
        // 새로운 컨트랙트를 생성하는데 사용되지 않는다면, 이것을 인용하는 것은 문제가 없다.
        TokenCreator creator;
        address owner;
        bytes32 name;

        // This is the constructor which registers the
        // creator and the assigned name.
        // 이 부분은 만든이와 할당된 이름을 등록하는 생성자이다.
        constructor(bytes32 _name) public {
            // State variables are accessed via their name
            // and not via e.g. this.owner. This also applies
            // to functions and especially in the constructors,
            // you can only call them like that ("internally"),
            // because the contract itself does not exist yet.
            // 상태 변수는 그들의 이름을 통해서 접근 가능하며, this.owner 같은 형태로는 접근이 불가능하다.
            // 이것은 함수에게도 적용이 되는데, 특히 생성자가 그렇다. 당신은 그것을들 ("내부적으로") 콜 할 수 있다.
            // 왜냐하면, 컨트랙트 그 자체가 아직 존재하지 않기 떄문이다.
            owner = msg.sender;
            // We do an explicit type conversion from `address`
            // to `TokenCreator` and assume that the type of
            // the calling contract is TokenCreator, there is
            // no real way to check that.
            // 우리는 `address`로부터 `TokenCreate`로 명시적 타입 변경을 한다.
            // 그리고, 우리는 함수를 호출하는 컨트랙트를 TokenCreator라고 가정한다.
            // 이것을 실제로 확인할 방법은 없다.
            creator = TokenCreator(msg.sender);
            name = _name;
        }

        function changeName(bytes32 newName) public {
            // Only the creator can alter the name --
            // the comparison is possible since contracts
            // are explicitly convertible to addresses.
            // 오직 생성자만이 이름을 변경할 수 있다.
            // 비교 연산은 명시적으로 어드레스로 변경될 수 있어야만 작동된다.
            if (msg.sender == address(creator))
                name = newName;
        }

        function transfer(address newOwner) public {
            // Only the current owner can transfer the token.
            // 오직 현재 주인만이 토큰을 전송할 수 있다
            if (msg.sender != owner) return;

            // We also want to ask the creator if the transfer
            // is fine. Note that this calls a function of the
            // contract defined below. If the call fails (e.g.
            // due to out-of-gas), the execution also fails here.
            // 우리는 만든이에게 전송이 괜찮은지 물어보는 걸 원한다.
            // 이것은 밑에 정의된 컨트랙트의 함수를 부른다는 것을 주지한다.
            // 만약 콜이 실패할 경우 (e.g. 가스 부족), 여기서 작동이 멈출 것이다.
            if (creator.isTokenTransferOK(owner, newOwner))
                owner = newOwner;
        }
    }

    contract TokenCreator {
        function createToken(bytes32 name)
           public
           returns (OwnedToken tokenAddress)
        {
            // Create a new Token contract and return its address.
            // From the JavaScript side, the return type is simply
            // `address`, as this is the closest type available in
            // the ABI.
            // 새로운 토큰 컨트랙트를 들고 그것의 어드레스를 반환한다.
            // 자바스크립트 사이드에서는 리턴 타입은 단순한 `address`이며,
            // 이것은 ABI에서 가능한 가장 가까운 타입이다.
            return new OwnedToken(name);
        }

        function changeName(OwnedToken tokenAddress, bytes32 name) public {
            // Again, the external type of `tokenAddress` is
            // simply `address`.
            // 다시 말하면, tokenAddress의 외적 타입은 단순하게 address이다.
            tokenAddress.changeName(name);
        }

        function isTokenTransferOK(address currentOwner, address newOwner)
            public
            pure
            returns (bool ok)
        {
            // Check some arbitrary condition.
            // 임의적인 컨디션을 확인 한다
            return keccak256(abi.encodePacked(currentOwner, newOwner))[0] == 0x7f;
        }
    }

.. index:: ! visibility, external, public, private, internal

.. _visibility-and-getters:

**********************
Visibility and Getters
**********************

Since Solidity knows two kinds of function calls (internal
ones that do not create an actual EVM call (also called
a "message call") and external
ones that do), there are four types of visibilities for
functions and state variables.
솔리디티는 두 종류의 함수 콜(내부 함수라는 실질적인 (메세지 콜이라고 불리는)
EVM 콜을 만들지 않는 것과 외부 콜이라는 EVM콜을 하는 것)을 알고 있지만,
사실은 함수와 상태 변수에 대한 4 가지의 가시성이 존재한다

Functions have to be specified as being ``external``,
``public``, ``internal`` or ``private``.
For state variables, ``external`` is not possible.
함수는 ``external``(외부), ``public``, ``internal`` (내부),
혹은 ``private` 으로 특징지어져야만한다. 상태 변수에게는 ``external``은
불가능하다. 

``external``:
    External functions are part of the contract interface,
    which means they can be called from other contracts and
    via transactions. An external function ``f`` cannot be called
    internally (i.e. ``f()`` does not work, but ``this.f()`` works).
    External functions are sometimes more efficient when
    they receive large arrays of data.
    외부 함수는 트랜젝션이나 다른 컨트랙트로부터 호출이 될 수 있는 컨트랙트 인터페이스의
    한 종류이다. 외부 함수는 ``f()``는 내부적으로 호출 될 수 없다. (i.e. ``f()`` 는 작동하지 않지만,
    ``this.f()``는 작동한다.) 외부 함수들은 데이터의 거대한 배열을 받을 떄 종종 더 효율적일 떄가 있다.

``public``:
    Public functions are part of the contract interface
    and can be either called internally or via
    messages. For public state variables, an automatic getter
    function (see below) is generated.
    퍼블릭 함수들은 컨트랙트의 인터페이스로 내부적으로 호출되거나 메세지를 통해서호출 될 수 있다.
    퍼블릭 상태 함수들은 자동적으로 getter 함수(밑에 참조)가 생성된다.

``internal``:
    Those functions and state variables can only be
    accessed internally (i.e. from within the current contract
    or contracts deriving from it), without using ``this``.
    이러한 함수들과 상태 변수들은 ``this`` 사용하지 않은 상태에 내부적으로만 접근이 가능하다. (i.e. 현재 컨트랙트로부터나
    그 함수로부터 유도된 컨트랙트들로부터만)

``private``:
    Private functions and state variables are only
    visible for the contract they are defined in and not in
    derived contracts.
    프라이빗 함수들과 상태 변수들은 그것들이 정의된 컨트랙트에만 보이며, 유도된 컨트랙트에서는 보이지 않는다.

.. note::
    Everything that is inside a contract is visible to
    all observers external to the blockchain. Making something ``private``
    only prevents other contracts from accessing and modifying
    the information, but it will still be visible to the
    whole world outside of the blockchain.
    컨트랙트 안에 있는 모든 것들은 블록체인을 관찰하는 모든 관찰자에게 노출된다.
    ``private``한 무언가를 만들어도 다른 컨트랙트가 정보에 접근하거나 수정하는 것을 막을 뿐이며,
    블록체인 밖의 모든 세상에 보일 것입니다.

The visibility specifier is given after the type for
state variables and between parameter list and
return parameter list for functions.
접근 지정자는 상태 변수의 타입이 정해진 이후에 주어져야하며, 
매개변수 리스트와 함수들의 리턴 매개변수 리스트 사이에 있어야한다.

::

    pragma solidity >=0.4.16 <0.6.0;

    contract C {
        function f(uint a) private pure returns (uint b) { return a + 1; }
        function setData(uint a) internal { data = a; }
        uint public data;
    }

In the following example, ``D``, can call ``c.getData()`` to retrieve the value of
``data`` in state storage, but is not able to call ``f``. Contract ``E`` is derived from
``C`` and, thus, can call ``compute``.
다음의 예시에서 ``D``는 ``c.getData()``를 호출 함으로써 상태 저장소 안의 ``data``의 값을 회수할 수 있다. 그러나 ``f``를 호출할 수는 없다.
컨트랙트 ``E``sms ``C``로부터 유도 되었으므로 ``compute``를 호출 할 수 있다.

::

    pragma solidity >=0.4.0 <0.6.0;

    contract C {
        uint private data;

        function f(uint a) private pure returns(uint b) { return a + 1; }
        function setData(uint a) public { data = a; }
        function getData() public view returns(uint) { return data; }
        function compute(uint a, uint b) internal pure returns (uint) { return a + b; }
    }

    // This will not compile
    // 이것은 컴파일이 되지 않을 것입니다.
    contract D {
        function readData() public {
            C c = new C();
            uint local = c.f(7); // error: member `f` is not visible
            // 오류 : `f` 멤버에 접근 불가
            c.setData(3);
            local = c.getData();
            local = c.compute(3, 5); // error: member `compute` is not visible
            // 오류 : `compute` 멤버에 접근 불가
        }
    }

    contract E is C {
        function g() public {
            C c = new C();
            uint val = compute(3, 5); // access to internal member (from derived to parent contract)
            // (부모 컨트랙트로부터 유도된) 내부 멤버에 접근
        }
    }

.. index:: ! getter;function, ! function;getter
.. _getter-functions:

Getter Functions
================

The compiler automatically creates getter functions for
all **public** state variables. For the contract given below, the compiler will
generate a function called ``data`` that does not take any
arguments and returns a ``uint``, the value of the state
variable ``data``. State variables can be initialized
when they are declared.
컴파일러는 자동적으로 모든 **public** 상태 변수에 대해 getter 함수를 만들 것입니다.
다음에 주어진 컨트랙트에서는, 컴파일러는 아무런 인자를 받지 않고, 상태 변수 ``data``의 값인
``uint`` 타입을 반환하는 ``data``라고 불리워지는 함수를 생성합니다.
상태 변수는 선언과 동시에 초기화 될 수 있습니다.


::

    pragma solidity >=0.4.0 <0.6.0;

    contract C {
        uint public data = 42;
    }

    contract Caller {
        C c = new C();
        function f() public view returns (uint) {
            return c.data();
        }
    }

The getter functions have external visibility. If the
symbol is accessed internally (i.e. without ``this.``),
it evaluates to a state variable.  If it is accessed externally
(i.e. with ``this.``), it evaluates to a function.
getter 함수는 external 접근성을 갑고 있습니다. 만약 심볼이 내부적으로 내부적으로 (i.e. ``this`` 없이) 접근된다면,
이것은 상태 변수로 평가됩니다. 만약, 외부적으로 접근(i.e. ``this``와 함께) 된다면, 함수로 평가됩니다.

::

    pragma solidity >=0.4.0 <0.6.0;

    contract C {
        uint public data;
        function x() public returns (uint) {
            data = 3; // internal access
            return this.data(); // external access
        }
    }

If you have a ``public`` state variable of array type, then you can only retrieve
single elements of the array via the generated getter function. This mechanism
exists to avoid high gas costs when returning an entire array. You can use
arguments to specify which individual element to return, for example
``data(0)``. If you want to return an entire array in one call, then you need
to write a function, for example:
만약 배열 타입의 ``public`` 상태 변수를 갖고 있다면, 그렇다면 getter 함수로 생성된 배열의 단일 요소들을 얻을 수 있습니다.
이 매커니즘은 전체 배열을 반환하여 높은 가스 비용을 무는 것을 피하기 위함입니다. 리턴을 받은 하나의 요소를 명시하는 인자를 ``data(0)``와 정할 수 있습니다.
만약 한 번의 호출로 모든 배열을 반환 받기 원한다면, 다음과 같은 함수를 작성해야할 것입니다.


::

  pragma solidity >=0.4.0 <0.6.0;

  contract arrayExample {
    // public state variable
    uint[] public myArray;

    // Getter function generated by the compiler
    // 컴파일러를 통해 만들어진 getter
    /*
    function myArray(uint i) returns (uint) {
        return myArray[i];
    }
    */

    // function that returns entire array
    // 전체 배열을 반환하는 함수
    function getArray() returns (uint[] memory) {
        return myArray;
    }
  }

Now you can use ``getArray()`` to retrieve the entire array, instead of
``myArray(i)``, which returns a single element per call.
한 호출마다 하나의 요소를 반환하는``myArray(i)`` 대신에 ``getArray()``를 사용함으로써 전체 배열을 얻을 수 있습니다.

The next example is more complex:
다음의 예제는 좀 더 복잡합니다.

::

    pragma solidity >=0.4.0 <0.6.0;

    contract Complex {
        struct Data {
            uint a;
            bytes3 b;
            mapping (uint => uint) map;
        }
        mapping (uint => mapping(bool => Data[])) public data;
    }

It generates a function of the following form. The mapping in the struct is omitted
because there is no good way to provide the key for the mapping:
이것은 다음과 같은 형태의 함수를 생성합니다. 구조체에서의 맵핑이 빠지게 되는데, 그 이유는 맵핑에 대한 키를 제공하는 방법이 없기 떄문입니다.

::

    function data(uint arg1, bool arg2, uint arg3) public returns (uint a, bytes3 b) {
        a = data[arg1][arg2][arg3].a;
        b = data[arg1][arg2][arg3].b;
    }

.. index:: ! function;modifier

.. _modifiers:

******************
함수 Modifier
******************

Modifiers can be used to easily change the behaviour of functions.  For example,
they can automatically check a condition prior to executing the function. Modifiers are
inheritable properties of contracts and may be overridden by derived contracts.
Modifier는 쉽게 함수들의 행동을 바꾸기 위해 사용됩니다. 예를 들어, 함수를 실행시키기전에 상태를 자동적으로 검사할 수 있습니다.
Modifier들은 컨트랙트의 상속가능한 부분이고, 유도된 컨트랙트에서 오버라이드 될 수 있습니다.

::

    pragma solidity >0.4.99 <0.6.0;

    contract owned {
        constructor() public { owner = msg.sender; }
        address payable owner;

        // This contract only defines a modifier but does not use
        // it: it will be used in derived contracts.
        // The function body is inserted where the special symbol
        // `_;` in the definition of a modifier appears.
        // This means that if the owner calls this function, the
        // function is executed and otherwise, an exception is
        // thrown.
        // 이 컨트랙트는 modifier만 정의를 합니다. 그러나 그것을 사용하지는 않습니다. 상속 받은 컨트랙트에서만 사용될 뿐입니다.
        // 함수의 바디는 modifier를 나타내는 특별한 심볼인 `_;`을 사용합니다.
        // 이것은 주인이 이 함수를 실행할 경우 이 함수가 실행된다는 의미이며, 다른 사람이 실행할 경우 익셉션이 발생한다는 뜻입니다.
        modifier onlyOwner {
            require(
                msg.sender == owner,
                "Only owner can call this function."
            );
            _;
        }
    }

    contract mortal is owned {
        // This contract inherits the `onlyOwner` modifier from
        // `owned` and applies it to the `close` function, which
        // causes that calls to `close` only have an effect if
        // they are made by the stored owner.
        // 이 컨트랙트는 `owned`로부터 `onlyOwner` modifier를 상속 받고, `close` 함수에 그것을 적용 시킵니다.
        // 이를 통해, 저장된 owner로부터 만들어진 호출들로부터 발생한 `close`만이 영향을 주게 됩니다.
        function close() public onlyOwner {
            selfdestruct(owner);
        }
    }

    contract priced {
        // Modifiers can receive arguments:
        // Modifier는 인자를 받을 수 있습니다.
        modifier costs(uint price) {
            if (msg.value >= price) {
                _;
            }
        }
    }

    contract Register is priced, owned {
        mapping (address => bool) registeredAddresses;
        uint price;

        constructor(uint initialPrice) public { price = initialPrice; }

        // It is important to also provide the
        // `payable` keyword here, otherwise the function will
        // automatically reject all Ether sent to it.
        // `payable` 키워드를 여기에 넣는 것은 중요합니다.
        // 그렇지 않으면 모든 컨트랙트에 보낸 이더리움 전송을 자동으로 함수가 거절할 것이기 떄문입니다.
        function register() public payable costs(price) {
            registeredAddresses[msg.sender] = true;
        }

        function changePrice(uint _price) public onlyOwner {
            price = _price;
        }
    }

    contract Mutex {
        bool locked;
        modifier noReentrancy() {
            require(
                !locked,
                "Reentrant call."
            );
            locked = true;
            _;
            locked = false;
        }

        /// This function is protected by a mutex, which means that
        /// reentrant calls from within `msg.sender.call` cannot call `f` again.
        /// The `return 7` statement assigns 7 to the return value but still
        /// executes the statement `locked = false` in the modifier.
        // 이 함수는 뮤택스로부터 보호받을 것입니다. 이는 `msg.sender.call`이내의 재진입 호출이 `f`를 다시 호출 할 수 없다는 뜻입니다.
        // `rureun 7` 스테이트먼트는 7을 리턴 밸류로 할당하겠지만 아직 modifier 속의 `locked = false`를 실행 시킬 것입니다.
        function f() public noReentrancy returns (uint) {
            (bool success,) = msg.sender.call("");
            require(success);
            return 7;
        }
    }

Multiple modifiers are applied to a function by specifying them in a
whitespace-separated list and are evaluated in the order presented.
공백으로 분리되고, 주어진 순서대로 평가되어지는 여러 개의 modifier들이 함수에 적용 될 수 있습니다.

.. warning::
    In an earlier version of Solidity, ``return`` statements in functions
    having modifiers behaved differently.
    솔리디티 초창기 버전에서는 modifiers를 갖고 있는 함수 안의 ``return`` 선언은 다르게 작동하였습니다.

Explicit returns from a modifier or function body only leave the current
modifier or function body. Return variables are assigned and
control flow continues after the "_" in the preceding modifier.
modifier나 함수 바디의 명백한 리턴은 현재의 modifier나 함수 몸체만을 남깁니다.
리턴 변수는 앞선 modifier의 "_" 이후에 할당 되고 컨트롤 플로우가 지속 됩니다.

Arbitrary expressions are allowed for modifier arguments and in this context,
all symbols visible from the function are visible in the modifier. Symbols
introduced in the modifier are not visible in the function (as they might
change by overriding).
modifier 인자에 대해 임의적인 표현이 허용되고, 이런 면에서 함수에서 접근가능한 모든 심볼들은 modifier에게도 접근 가능합니다.
modifier에서 도입된 심볼들은 함수에게는 (그들이 오버라이딩을 통해 바뀔 수 있기에) 접근 가능하지 않습니다.

.. index:: ! constant

************************
상수 상태의 변수들
************************

State variables can be declared as ``constant``. In this case, they have to be
assigned from an expression which is a constant at compile time. Any expression
that accesses storage, blockchain data (e.g. ``now``, ``address(this).balance`` or
``block.number``) or
execution data (``msg.value`` or ``gasleft()``) or makes calls to external contracts is disallowed. Expressions
that might have a side-effect on memory allocation are allowed, but those that
might have a side-effect on other memory objects are not. The built-in functions
``keccak256``, ``sha256``, ``ripemd160``, ``ecrecover``, ``addmod`` and ``mulmod``
are allowed (even though they do call external contracts).
상태 변수들은 ``constant``(상수)로 정의될 수 있습니다. 이러한 상황에서, 그들은 컴파일 타임에 상수인 표현식이 되어야합니다.
어떠한 표션식도 스토리지에 접근하거나, 블록체인 데이터에 접근하거나(e.g. ``now``, ``address(this).balance`` 혹은 ``block.number``)
혹은 데이터를 실행하거나 (``msg.value`` or ``gasleft()``) 혹은 외부 컨트랙트에 호출을 하는 것은 모두 허가되지 않습니다.
메모리 할당에 사이드 이팩트를 갖을 수 있는 포현식은 허가 됩니다. 그러나, 이러한 사이드 이팩트가 다른 메모리 오브젝트에 영향을 준다면 그렇지 않습니다.
내장 함수인 ``keccak256``, ``sha256``, ``ripemd160``, ``ecrecover``, ``addmod``, 그리고 ``mulmod`` 는 허가 됩니다. (심지어 그것들이 외부 컨트랙트를 호출한다고 하더라도요)

The reason behind allowing side-effects on the memory allocator is that it
should be possible to construct complex objects like e.g. lookup-tables.
This feature is not yet fully usable.
메모리 할당자에 대한 사이드 이팩트를 허용하는 이유는 룩업 데이블과 같은 복잡한 오브젝트들을 구축하는데 도움이 되기 때문입니다.
이러한 기능들은 완벽하게 사용할 수 있지는 않습니다.

The compiler does not reserve a storage slot for these variables, and every occurrence is
replaced by the respective constant expression (which might be computed to a single value by the optimizer).
컴파일러는 이러한 변수들을 위해 스토리지 슬롯을 예약하지 않습니다. 그리고 (옵티마이저에 의해 하나의 값으로 계산되어질) 각각의 모든 상수 표현식으로 치환될 것입니다.

Not all types for constants are implemented at this time. The only supported types are
value types and strings.
상수들에 대한 모든 타입들이 적용된 것은 아닙니다. 값 타입과 문자열 타입만 지원됩니다.

::

    pragma solidity >=0.4.0 <0.6.0;

    contract C {
        uint constant x = 32**22 + 8;
        string constant text = "abc";
        bytes32 constant myHash = keccak256("abc");
    }

.. index:: ! functions

.. _functions:

*********
함수
*********

.. index:: ! view function, function;view

.. _view-functions:

View Functions
==============

Functions can be declared ``view`` in which case they promise not to modify the state.
함수는 state를 변화하지 않는다는 걸 약속하는 `view`로 선언 될 수 있습니다.

.. note::
  If the compiler's EVM target is Byzantium or newer (default) the opcode
  ``STATICCALL`` is used for ``view`` functions which enforces the state
  to stay unmodified as part of the EVM execution. For library ``view`` functions
  ``DELEGATECALL`` is used, because there is no combined ``DELEGATECALL`` and ``STATICCALL``.
  This means library ``view`` functions do not have run-time checks that prevent state
  modifications. This should not impact security negatively because library code is
  usually known at compile-time and the static checker performs compile-time checks.
  만약 컴파일러의 EVM 타겟이 비잔티움 혹은 그보다 최신이라면, ``STATICCALL`` opcode가 EVM 실행시 상태가 변화하지 않도록 강제하도록 ``view`` 함수들을 위해서 사용 될 것입니다.
  ``DELEGATECALL`` 사용되면, 라이브러리에게 ``view`` 함수들은 상태 변경을 막기 위한 런타임 확인을 시행하지 않습니다. 이것은 보안적으로 부정적인 영향을 미치지는 않습니다.
  라이브러리 코드들은 정적 분석기가 컴파일 타임에 확인을 하고, 그 이후에도 알 수 있기 떄문입니다.

The following statements are considered modifying the state:
다음과 같은 선언은 상태를 바꾼다고 가정합니다.

#. Writing to state variables.
#. :ref:`Emitting events <events>`.
#. :ref:`Creating other contracts <creating-contracts>`.
#. Using ``selfdestruct``.
#. Sending Ether via calls.
#. Calling any function not marked ``view`` or ``pure``.
#. Using low-level calls.
#. Using inline assembly that contains certain opcodes.
#. 상태 변수에 쓰는 것
#. :ref:`Emitting events <events>`.
#. :ref:`Creating other contracts <creating-contracts>`.
#. ``selfdestruct``를 사용하는 것
#. 호출을 통해 이더리움을 보내는 것
#. ``view``나 ``pure``라고 적히지 않은 어떠한 함수를 쓰는 것
#. 저 수준 호출을 하는 것
#. 특정한 opcode를 포함하고 있는 인라인 어셈블리를 사용하는 것  

::

    pragma solidity >0.4.99 <0.6.0;

    contract C {
        function f(uint a, uint b) public view returns (uint) {
            return a * (b + 42) + now;
        }
    }

.. note::
  ``constant`` on functions used to be an alias to ``view``, but this was dropped in version 0.5.0.
  함수의에 ``constant``를 사용하는 것은 ``view``를 지칭하는 것이였지만, 0.5.0 버전부터는 제외 되었습니다.

.. note::
  Getter methods are automatically marked ``view``.
  Getter 메소드들은 자동으로 ``view``로 마킹됩니다.

.. note::
  Prior to version 0.5.0, the compiler did not use the ``STATICCALL`` opcode
  for ``view`` functions.
  This enabled state modifications in ``view`` functions through the use of
  invalid explicit type conversions.
  By using  ``STATICCALL`` for ``view`` functions, modifications to the
  state are prevented on the level of the EVM.
  0.5.0 이전에는 컴파일러가 ``STATICALL`` opcode를 ``view`` 함수에 사용하지 않았습니다.
  이는 ``view`` 함수에게 잘못된 명시적 타입 변경을 통해 상태를 변경할 수 있게 하였습니다.
  ``STATICALL``을 ``view`` 함수들에 쓰게 되면서, EVM 수준에서 상태를 변경하는 것을 막을 수 있었습니다.

.. index:: ! pure function, function;pure

.. _pure-functions:

순수 함수
==============

Functions can be declared ``pure`` in which case they promise not to read from or modify the state.
``pure``로 선언되어지는 함수들은 상태를 읽거나 수정할 수 없음을 약속합니다.

.. note::
  If the compiler's EVM target is Byzantium or newer (default) the opcode ``STATICCALL`` is used,
  which does not guarantee that the state is not read, but at least that it is not modified.
  만약 컴파일러의 EVM 타겟이 비잔티움 혹은 그 이후라면, (기본) 오피코드인 수정 되지 않는 것을 보장하지만,
  상태가 읽히지 않을 것이라는 것을 보장하지는 않는 ``STATICALL``이 사용됩니다.

In addition to the list of state modifying statements explained above, the following are considered reading from the state:
추가적으로 상태를 수정하는 상태들의 집합이 위에 설명되었습니다. 다음의 것들은 상태로부터 읽는 것으로 간주됩니다.

#. Reading from state variables.
#. Accessing ``address(this).balance`` or ``<address>.balance``.
#. Accessing any of the members of ``block``, ``tx``, ``msg`` (with the exception of ``msg.sig`` and ``msg.data``).
#. Calling any function not marked ``pure``.
#. Using inline assembly that contains certain opcodes.

#. 상태변수로부터 읽기
#. ``address(this).balance`` 혹은 ``address.balance``에 접근하기
#. ``block``, ``tx``, ``msg`` (``msg.sig`` 와 ``msg.data``라는 예외가 존재함) 의 멤버에 접근하는 것들
#. ``pure``라고 적히지 않은 함수들 호출하기
#. 특정 opcode를 포함하고 있는 인라인 어셈블리를 사용하기

::

    pragma solidity >0.4.99 <0.6.0;

    contract C {
        function f(uint a, uint b) public pure returns (uint) {
            return a * (b + 42);
        }
    }

.. note::
  Prior to version 0.5.0, the compiler did not use the ``STATICCALL`` opcode
  for ``pure`` functions.
  This enabled state modifications in ``pure`` functions through the use of
  invalid explicit type conversions.
  By using  ``STATICCALL`` for ``pure`` functions, modifications to the
  state are prevented on the level of the EVM.
  0.5.0 버전 이전에는, 컴파일러는 ``STATICCALL`` opcode를 ``pure`` 함수에 사용하지 않았습니다.
  이것은 ``pure`` 함수가 잘못된 명시적 타입 변환을 통해 상태를 수정할 수 있게 합니다.
  ``STATICALL``을 ``pure`` 함수에게 사용함으로써, EVM 수준에서 상태의 변경이 막히게 됩니다.

.. warning::
  It is not possible to prevent functions from reading the state at the level
  of the EVM, it is only possible to prevent them from writing to the state
  (i.e. only ``view`` can be enforced at the EVM level, ``pure`` can not).
  함수가 EVM 수준에서 상태를 읽는 것을 막지는 못합니다. 단순히 상태를 쓰지 못하는 것을 막을 뿐입니다.
  (i.e. 오직 ``view``만이 EVM 수준에서 강제할 수 있습니다. ``pure``는 그렇지 못 합니다.)

.. warning::
  Before version 0.4.17 the compiler did not enforce that ``pure`` is not reading the state.
  It is a compile-time type check, which can be circumvented doing invalid explicit conversions
  between contract types, because the compiler can verify that the type of the contract does
  not do state-changing operations, but it cannot check that the contract that will be called
  at runtime is actually of that type.
  0.4.17 이전에는 컴파일러가 ``pure``가 상태를 읽지 못하도록 강제하지 않았습니다. 이것은 컴파일 시간의 타입 체크이고,
  컨트랙트 타입 사이에서 잘못된 명시적 변환을 통해 우회할 수 있었습니다. 컴파일러가 컨트랙트가 상태 변경 오퍼레이션을 하지 않는다는
  것만 확인했기 떄문입니다. 그리고 그것은 컨트랙트가 그 타입으로 명확하게 런타임에서 호출될 것이라는 것을 체크하지는 못합니다.

.. index:: ! fallback function, function;fallback

.. _fallback-function:

Fallback Function
=================

A contract can have exactly one unnamed function. This function cannot have
arguments, cannot return anything and has to have ``external`` visibility.
It is executed on a call to the contract if none of the other
functions match the given function identifier (or if no data was supplied at
all).
함수는 명확하게 하나의 이름이 없는 함수를 갖을 수 있습니다. 이러한 함수는 인자를 가질 수 없고, 어떤 것도 반환할 수 없으며,
``external`` 접근이 가능해야합니다. 이것은 주어진 함수 식별자가 어떠한 다른 함수들에게도 맞지 않을 경우에 실행됩니다. 
(혹은 어떠한 데이터도 주어지지 않았을 때)

Furthermore, this function is executed whenever the contract receives plain
Ether (without data). Additionally, in order to receive Ether, the fallback function
must be marked ``payable``. If no such function exists, the contract cannot receive
Ether through regular transactions.
게다가 이 함수는 단순히 (데이터 없이) 이더리움을 받았을 때도 실행됩니다. 이더리움을 받기 위해, fallback 함수는 
``payable``이라고 명시가 되어야합니다. 만약 이러한 함수가 존재하지 않는다면, 일반적인 트랜젝션을 통해 
컴트랙트는 이더리움을 받을 수 없습니다.

In the worst case, the fallback function can only rely on 2300 gas being
available (for example when `send` or `transfer` is used), leaving little
room to perform other operations except basic logging. The following operations
will consume more gas than the 2300 gas stipend:
최악의 상황에서는 fallback 함수는 기본적인 로깅을 제외한 다른 작업을 할 수 없을 정도의 작은 수준의 2300가스만 사용이 가능한 상황에 놓여질 수 있습니다. (예를 들어 `send`나 `transfer`가 사용될 때)
다음의 작업은 2300 가스 이상 사용이 될 것입니다.


- Writing to storage
- Creating a contract
- Calling an external function which consumes a large amount of gas
- Sending Ether

- 스토리지에 쓰기
- 컨트랙트 생성
- 엄청난 양의 가스를 소비하는 외부 함수 부르기
- 이더리움 전송

Like any function, the fallback function can execute complex operations as long as there is enough gas passed on to it.
다른 어떠한 함수와 같이, fallback 함수는 가스가 들어오는 만큼 복잡한 작업을 할 수 있습니다.

.. note::
    Even though the fallback function cannot have arguments, one can still use ``msg.data`` to retrieve
    any payload supplied with the call.
    fallback 함수가 인자를 갖을 수 없다고 하지만, ``msg.data``를 이용하여 호출에서 제공하는 어떠한 페이로드라도 회수 할 수 있습니다.

.. warning::
    The fallback function is also executed if the caller meant to call
    a function that is not available. If you want to implement the fallback
    function only to receive ether, you should add a check
    like ``require(msg.data.length == 0)`` to prevent invalid calls.
    fallback 함수가 실행 되었다는 것은 존재하지 않는 함수를 호출하려고 했다는 것을 의미합니다.
    이더리움을 받으려고 fallback 함수를 쓴다면, ``require(msg.data.length == 0)``를 이용하여
    잘못된 콜들을 막으십시오.

.. warning::
    Contracts that receive Ether directly (without a function call, i.e. using ``send`` or ``transfer``)
    but do not define a fallback function
    throw an exception, sending back the Ether (this was different
    before Solidity v0.4.0). So if you want your contract to receive Ether,
    you have to implement a payable fallback function.
    컨트랙트가 (함수 콜 없이 ``send`` 또는 ``transfer``를 이용하여) 직접적으로 이더리움을 받을 경우, fallback 함수가 없으면 
    예외를 발생시키고 이더리움을 다시 되돌려 보냅니다. (이것은 솔리디티 버전 0.4.0 이전에는 다릅니다)
    그렇기에 만약에 이더리움을 받기 원한다면, payable fallback function을 이용하십시오.

.. warning::
    A contract without a payable fallback function can receive Ether as a recipient of a `coinbase transaction` (aka `miner block reward`)
    or as a destination of a ``selfdestruct``.
    payable fallback 함수가 없는 컨트랙트도 이더리움을 `coinbase transcation` (aka `miner block reward`) 혹은 ``selfdestruct``의 목적지로써 받을 수 있습니다.

    A contract cannot react to such Ether transfers and thus also cannot reject them. This is a design choice of the EVM and Solidity cannot work around it.
    이러한 이더리움 전송을 대응할 수 없고, 그것을 거부 할 수도 없습니다. 이 EVM과 솔리디티의 디자인에 있어서 피할 수 없었던 부분입니다.

    It also means that ``address(this).balance`` can be higher than the sum of some manual accounting implemented in a contract (i.e. having a counter updated in the fallback function).
    이것은 ``address(this).balance``가 수동으로 정산 기능이 도입된 컨트랙트 내에서의 값보다 크게 나올 수 있다는 것입니다. (i.e. fallback function이 발동 될 때마다 카운터가 올라가는 구조) 

::

    pragma solidity >0.4.99 <0.6.0;

    contract Test {
        // This function is called for all messages sent to
        // this contract (there is no other function).
        // Sending Ether to this contract will cause an exception,
        // because the fallback function does not have the `payable`
        // modifier.
        // 이 함수는 이 컨트랙트에 전송된 모든 메세지에 의해 호출 된다. (다른 함수가 없기 떄문이다)
        // 이더리움을 이 컨트랙트에 보내는 것은 익셉션을 발생시킨다.
        // 왜냐하면 이 함수는 `payable` modifier가 존재하지 않기 때문이다.
        function() external { x = 1; }
        uint x;
    }


    // This contract keeps all Ether sent to it with no way
    // to get it back.
    // 이 컨트랙트는 모든 이더리움을 보관하지만, 이것을 다시 돌려 받을 방법은 없다.
    contract Sink {
        function() external payable { }
    }

    contract Caller {
        function callTest(Test test) public returns (bool) {
            (bool success,) = address(test).call(abi.encodeWithSignature("nonExistingFunction()"));
            require(success);
            // results in test.x becoming == 1.
            // test.x 가 1이 된다.

            // address(test) will not allow to call ``send`` directly, since ``test`` has no payable
            // fallback function. It has to be converted to the ``address payable`` type via an
            // intermediate conversion to ``uint160`` to even allow calling ``send`` on it.
            // address(test)는 ``send``를 직접적으로 호출 할 수 없다. ``test``는 payable fallback 함수가 없기 때문이다.
            // ``uint160``으로 직접적 변환을 통해 ``send``를 호출 할 수 있는 ``address payable`` 타입으로 변경할 필요가 있다.
            address payable testPayable = address(uint160(address(test)));

            // If someone sends ether to that contract,
            // the transfer will fail, i.e. this returns false here.
            // 만약 이더리움을 그 컨트랙트로 보낸다면, 전송은 실패할 것이다. i.e. 이것은 false를 반환한다.
            return testPayable.send(2 ether);
        }
    }

.. index:: ! overload

.. _overload-function:

함수 오버로딩
====================

A contract can have multiple functions of the same name but with different parameter
types.
This process is called "overloading" and also applies to inherited functions.
The following example shows overloading of the function
``f`` in the scope of contract ``A``.
컨트랙트는 다른 인자 타입을 갖는 여러개의 같은 이름을 갖는 함수를 가질 수 있다.
이러한 방식은 "오버로딩"불리며, 상속 받은 함수에도 역시 적용 된다. 다음의 예는 ``A`` 컨트랙트의 스코프 안에 있는 ``f``에 대한 오버로딩에 대해서 보여준다.


::

    pragma solidity >=0.4.16 <0.6.0;

    contract A {
        function f(uint _in) public pure returns (uint out) {
            out = _in;
        }

        function f(uint _in, bool _really) public pure returns (uint out) {
            if (_really)
                out = _in;
        }
    }

Overloaded functions are also present in the external interface. It is an error if two
externally visible functions differ by their Solidity types but not by their external types.
오버로딩된 함수는 외부 인터페이스에 역시나 보인다. 두 외부에 접근 가능한 함수가 외부 타입이 아니라 그것의 솔리디티 타입이 다르다면 에러가 발생한다.

::

    pragma solidity >=0.4.16 <0.6.0;

    // This will not compile
    contract A {
        function f(B _in) public pure returns (B out) {
            out = _in;
        }

        function f(address _in) public pure returns (address out) {
            out = _in;
        }
    }

    contract B {
    }


Both ``f`` function overloads above end up accepting the address type for the ABI although
they are considered different inside Solidity.
두 ``f`` 함수가 ABI에서는 어드레스 타입을 둘 다 받아들이는 것으로 오버로딩이 되었지만, 솔리디티 상에서는 다르게 취급이 된다.

Overload resolution and Argument matching
-----------------------------------------

Overloaded functions are selected by matching the function declarations in the current scope
to the arguments supplied in the function call. Functions are selected as overload candidates
if all arguments can be implicitly converted to the expected types. If there is not exactly one
candidate, resolution fails.


.. note::
    Return parameters are not taken into account for overload resolution.

::

    pragma solidity >=0.4.16 <0.6.0;

    contract A {
        function f(uint8 _in) public pure returns (uint8 out) {
            out = _in;
        }

        function f(uint256 _in) public pure returns (uint256 out) {
            out = _in;
        }
    }

Calling ``f(50)`` would create a type error since ``50`` can be implicitly converted both to ``uint8``
and ``uint256`` types. On another hand ``f(256)`` would resolve to ``f(uint256)`` overload as ``256`` cannot be implicitly
converted to ``uint8``.

.. index:: ! event

.. _events:

******
Events
******

Solidity events give an abstraction on top of the EVM's logging functionality.
Applications can subscribe and listen to these events through the RPC interface of an Ethereum client.

Events are inheritable members of contracts. When you call them, they cause the
arguments to be stored in the transaction's log - a special data structure
in the blockchain. These logs are associated with the address of the contract,
are incorporated into the blockchain, and stay there as long as a block is
accessible (forever as of the Frontier and Homestead releases, but this might
change with Serenity). The Log and its event data is not accessible from within
contracts (not even from the contract that created them).

It is possible to request a simple payment verification (SPV) for logs, so if
an external entity supplies a contract with such a verification, it can check
that the log actually exists inside the blockchain. You have to supply block headers
because the contract can only see the last 256 block hashes.

You can add the attribute ``indexed`` to up to three parameters which adds them
to a special data structure known as :ref:`"topics" <abi_events>` instead of
the data part of the log. If you use arrays (including ``string`` and ``bytes``)
as indexed arguments, its Keccak-256 hash is stored as a topic instead, this is
because a topic can only hold a single word (32 bytes).

All parameters without the ``indexed`` attribute are :ref:`ABI-encoded <ABI>`
into the data part of the log.

Topics allow you to search for events, for example when filtering a sequence of
blocks for certain events. You can also filter events by the address of the
contract that emitted the event.

For example, the code below uses the web3.js ``subscribe("logs")``
`method <https://web3js.readthedocs.io/en/1.0/web3-eth-subscribe.html#subscribe-logs>`_ to filter
logs that match a topic with a certain address value:

.. code-block:: javascript

    var options = {
        fromBlock: 0,
        address: web3.eth.defaultAccount,
        topics: ["0x0000000000000000000000000000000000000000000000000000000000000000", null, null]
    };
    web3.eth.subscribe('logs', options, function (error, result) {
        if (!error)
            console.log(result);
    })
        .on("data", function (log) {
            console.log(log);
        })
        .on("changed", function (log) {
    });


The hash of the signature of the event is one of the topics, except if you
declared the event with the ``anonymous`` specifier. This means that it is
not possible to filter for specific anonymous events by name.

::

    pragma solidity >=0.4.21 <0.6.0;

    contract ClientReceipt {
        event Deposit(
            address indexed _from,
            bytes32 indexed _id,
            uint _value
        );

        function deposit(bytes32 _id) public payable {
            // Events are emitted using `emit`, followed by
            // the name of the event and the arguments
            // (if any) in parentheses. Any such invocation
            // (even deeply nested) can be detected from
            // the JavaScript API by filtering for `Deposit`.
            emit Deposit(msg.sender, _id, msg.value);
        }
    }

The use in the JavaScript API is as follows:

::

    var abi = /* abi as generated by the compiler */;
    var ClientReceipt = web3.eth.contract(abi);
    var clientReceipt = ClientReceipt.at("0x1234...ab67" /* address */);

    var event = clientReceipt.Deposit();

    // watch for changes
    event.watch(function(error, result){
        // result contains non-indexed arguments and topics
        // given to the `Deposit` call.
        if (!error)
            console.log(result);
    });


    // Or pass a callback to start watching immediately
    var event = clientReceipt.Deposit(function(error, result) {
        if (!error)
            console.log(result);
    });

The output of the above looks like the following (trimmed):

.. code-block:: json

  {
     "returnValues": {
         "_from": "0x1111…FFFFCCCC",
         "_id": "0x50…sd5adb20",
         "_value": "0x420042"
     },
     "raw": {
         "data": "0x7f…91385",
         "topics": ["0xfd4…b4ead7", "0x7f…1a91385"]
     }
  }

.. index:: ! log

Low-Level Interface to Logs
===========================

It is also possible to access the low-level interface to the logging
mechanism via the functions ``log0``, ``log1``, ``log2``, ``log3`` and ``log4``.
``logi`` takes ``i + 1`` parameter of type ``bytes32``, where the first
argument will be used for the data part of the log and the others
as topics. The event call above can be performed in the same way as

::

    pragma solidity >=0.4.10 <0.6.0;

    contract C {
        function f() public payable {
            uint256 _id = 0x420042;
            log3(
                bytes32(msg.value),
                bytes32(0x50cb9fe53daa9737b786ab3646f04d0150dc50ef4e75f59509d83667ad5adb20),
                bytes32(uint256(msg.sender)),
                bytes32(_id)
            );
        }
    }

where the long hexadecimal number is equal to
``keccak256("Deposit(address,bytes32,uint256)")``, the signature of the event.

Additional Resources for Understanding Events
==============================================

- `Javascript documentation <https://github.com/ethereum/wiki/wiki/JavaScript-API#contract-events>`_
- `Example usage of events <https://github.com/debris/smart-exchange/blob/master/lib/contracts/SmartExchange.sol>`_
- `How to access them in js <https://github.com/debris/smart-exchange/blob/master/lib/exchange_transactions.js>`_

.. index:: ! inheritance, ! base class, ! contract;base, ! deriving

***********
Inheritance
***********

Solidity supports multiple inheritance by copying code including polymorphism.

All function calls are virtual, which means that the most derived function
is called, except when the contract name is explicitly given.

When a contract inherits from other contracts, only a single
contract is created on the blockchain, and the code from all the base contracts
is copied into the created contract.

The general inheritance system is very similar to
`Python's <https://docs.python.org/3/tutorial/classes.html#inheritance>`_,
especially concerning multiple inheritance, but there are also
some :ref:`differences <multi-inheritance>`.

Details are given in the following example.

::

    pragma solidity >0.4.99 <0.6.0;

    contract owned {
        constructor() public { owner = msg.sender; }
        address payable owner;
    }

    // Use `is` to derive from another contract. Derived
    // contracts can access all non-private members including
    // internal functions and state variables. These cannot be
    // accessed externally via `this`, though.
    contract mortal is owned {
        function kill() public {
            if (msg.sender == owner) selfdestruct(owner);
        }
    }

    // These abstract contracts are only provided to make the
    // interface known to the compiler. Note the function
    // without body. If a contract does not implement all
    // functions it can only be used as an interface.
    contract Config {
        function lookup(uint id) public returns (address adr);
    }

    contract NameReg {
        function register(bytes32 name) public;
        function unregister() public;
     }

    // Multiple inheritance is possible. Note that `owned` is
    // also a base class of `mortal`, yet there is only a single
    // instance of `owned` (as for virtual inheritance in C++).
    contract named is owned, mortal {
        constructor(bytes32 name) public {
            Config config = Config(0xD5f9D8D94886E70b06E474c3fB14Fd43E2f23970);
            NameReg(config.lookup(1)).register(name);
        }

        // Functions can be overridden by another function with the same name and
        // the same number/types of inputs.  If the overriding function has different
        // types of output parameters, that causes an error.
        // Both local and message-based function calls take these overrides
        // into account.
        function kill() public {
            if (msg.sender == owner) {
                Config config = Config(0xD5f9D8D94886E70b06E474c3fB14Fd43E2f23970);
                NameReg(config.lookup(1)).unregister();
                // It is still possible to call a specific
                // overridden function.
                mortal.kill();
            }
        }
    }

    // If a constructor takes an argument, it needs to be
    // provided in the header (or modifier-invocation-style at
    // the constructor of the derived contract (see below)).
    contract PriceFeed is owned, mortal, named("GoldFeed") {
       function updateInfo(uint newInfo) public {
          if (msg.sender == owner) info = newInfo;
       }

       function get() public view returns(uint r) { return info; }

       uint info;
    }

Note that above, we call ``mortal.kill()`` to "forward" the
destruction request. The way this is done is problematic, as
seen in the following example::

    pragma solidity >=0.4.22 <0.6.0;

    contract owned {
        constructor() public { owner = msg.sender; }
        address payable owner;
    }

    contract mortal is owned {
        function kill() public {
            if (msg.sender == owner) selfdestruct(owner);
        }
    }

    contract Base1 is mortal {
        function kill() public { /* do cleanup 1 */ mortal.kill(); }
    }

    contract Base2 is mortal {
        function kill() public { /* do cleanup 2 */ mortal.kill(); }
    }

    contract Final is Base1, Base2 {
    }

A call to ``Final.kill()`` will call ``Base2.kill`` as the most
derived override, but this function will bypass
``Base1.kill``, basically because it does not even know about
``Base1``.  The way around this is to use ``super``::

    pragma solidity >=0.4.22 <0.6.0;

    contract owned {
        constructor() public { owner = msg.sender; }
        address payable owner;
    }

    contract mortal is owned {
        function kill() public {
            if (msg.sender == owner) selfdestruct(owner);
        }
    }

    contract Base1 is mortal {
        function kill() public { /* do cleanup 1 */ super.kill(); }
    }


    contract Base2 is mortal {
        function kill() public { /* do cleanup 2 */ super.kill(); }
    }

    contract Final is Base1, Base2 {
    }

If ``Base2`` calls a function of ``super``, it does not simply
call this function on one of its base contracts.  Rather, it
calls this function on the next base contract in the final
inheritance graph, so it will call ``Base1.kill()`` (note that
the final inheritance sequence is -- starting with the most
derived contract: Final, Base2, Base1, mortal, owned).
The actual function that is called when using super is
not known in the context of the class where it is used,
although its type is known. This is similar for ordinary
virtual method lookup.

.. index:: ! constructor

.. _constructor:

Constructors
============

A constructor is an optional function declared with the ``constructor`` keyword
which is executed upon contract creation, and where you can run contract
initialisation code.

Before the constructor code is executed, state variables are initialised to
their specified value if you initialise them inline, or zero if you do not.

After the constructor has run, the final code of the contract is deployed
to the blockchain. The deployment of
the code costs additional gas linear to the length of the code.
This code includes all functions that are part of the public interface
and all functions that are reachable from there through function calls.
It does not include the constructor code or internal functions that are
only called from the constructor.

Constructor functions can be either ``public`` or ``internal``. If there is no
constructor, the contract will assume the default constructor, which is
equivalent to ``constructor() public {}``. For example:

::

    pragma solidity >0.4.99 <0.6.0;

    contract A {
        uint public a;

        constructor(uint _a) internal {
            a = _a;
        }
    }

    contract B is A(1) {
        constructor() public {}
    }

A constructor set as ``internal`` causes the contract to be marked as :ref:`abstract <abstract-contract>`.

.. warning ::
    Prior to version 0.4.22, constructors were defined as functions with the same name as the contract.
    This syntax was deprecated and is not allowed anymore in version 0.5.0.


.. index:: ! base;constructor

Arguments for Base Constructors
===============================

The constructors of all the base contracts will be called following the
linearization rules explained below. If the base constructors have arguments,
derived contracts need to specify all of them. This can be done in two ways::

    pragma solidity >=0.4.22 <0.6.0;

    contract Base {
        uint x;
        constructor(uint _x) public { x = _x; }
    }

    // Either directly specify in the inheritance list...
    contract Derived1 is Base(7) {
        constructor() public {}
    }

    // or through a "modifier" of the derived constructor.
    contract Derived2 is Base {
        constructor(uint _y) Base(_y * _y) public {}
    }

One way is directly in the inheritance list (``is Base(7)``).  The other is in
the way a modifier is invoked as part of
the derived constructor (``Base(_y * _y)``). The first way to
do it is more convenient if the constructor argument is a
constant and defines the behaviour of the contract or
describes it. The second way has to be used if the
constructor arguments of the base depend on those of the
derived contract. Arguments have to be given either in the
inheritance list or in modifier-style in the derived constructor.
Specifying arguments in both places is an error.

If a derived contract does not specify the arguments to all of its base
contracts' constructors, it will be abstract.

.. index:: ! inheritance;multiple, ! linearization, ! C3 linearization

.. _multi-inheritance:

Multiple Inheritance and Linearization
======================================

Languages that allow multiple inheritance have to deal with
several problems.  One is the `Diamond Problem <https://en.wikipedia.org/wiki/Multiple_inheritance#The_diamond_problem>`_.
Solidity is similar to Python in that it uses "`C3 Linearization <https://en.wikipedia.org/wiki/C3_linearization>`_"
to force a specific order in the directed acyclic graph (DAG) of base classes. This
results in the desirable property of monotonicity but
disallows some inheritance graphs. Especially, the order in
which the base classes are given in the ``is`` directive is
important: You have to list the direct base contracts
in the order from "most base-like" to "most derived".
Note that this order is the reverse of the one used in Python.

Another simplifying way to explain this is that when a function is called that
is defined multiple times in different contracts, the given bases
are searched from right to left (left to right in Python) in a depth-first manner,
stopping at the first match. If a base contract has already been searched, it is skipped.

In the following code, Solidity will give the
error "Linearization of inheritance graph impossible".

::

    pragma solidity >=0.4.0 <0.6.0;

    contract X {}
    contract A is X {}
    // This will not compile
    contract C is A, X {}

The reason for this is that ``C`` requests ``X`` to override ``A``
(by specifying ``A, X`` in this order), but ``A`` itself
requests to override ``X``, which is a contradiction that
cannot be resolved.



Inheriting Different Kinds of Members of the Same Name
======================================================

When the inheritance results in a contract with a function and a modifier of the same name, it is considered as an error.
This error is produced also by an event and a modifier of the same name, and a function and an event of the same name.
As an exception, a state variable getter can override a public function.

.. index:: ! contract;abstract, ! abstract contract

.. _abstract-contract:

******************
Abstract Contracts
******************

Contracts are marked as abstract when at least one of their functions lacks an implementation as in the following example (note that the function declaration header is terminated by ``;``)::

    pragma solidity >=0.4.0 <0.6.0;

    contract Feline {
        function utterance() public returns (bytes32);
    }

Such contracts cannot be compiled (even if they contain implemented functions alongside non-implemented functions), but they can be used as base contracts::

    pragma solidity >=0.4.0 <0.6.0;

    contract Feline {
        function utterance() public returns (bytes32);
    }

    contract Cat is Feline {
        function utterance() public returns (bytes32) { return "miaow"; }
    }

If a contract inherits from an abstract contract and does not implement all non-implemented functions by overriding, it will itself be abstract.

Note that a function without implementation is different from a :ref:`Function Type <function_types>` even though their syntax looks very similar.

Example of function without implementation (a function declaration)::

    function foo(address) external returns (address);

Example of a Function Type (a variable declaration, where the variable is of type ``function``)::

    function(address) external returns (address) foo;

Abstract contracts decouple the definition of a contract from its implementation providing better extensibility and self-documentation and
facilitating patterns like the `Template method <https://en.wikipedia.org/wiki/Template_method_pattern>`_ and removing code duplication.
Abstract contracts are useful in the same way that defining methods in an interface is useful. It is a way for the designer of the abstract contract to say "any child of mine must implement this method".


.. index:: ! contract;interface, ! interface contract

.. _interfaces:

**********
Interfaces
**********

Interfaces are similar to abstract contracts, but they cannot have any functions implemented. There are further restrictions:

- They cannot inherit other contracts or interfaces.
- All declared functions must be external.
- They cannot declare a constructor.
- They cannot declare state variables.

Some of these restrictions might be lifted in the future.

Interfaces are basically limited to what the Contract ABI can represent, and the conversion between the ABI and
an interface should be possible without any information loss.

Interfaces are denoted by their own keyword:

::

    pragma solidity >=0.4.11 <0.6.0;

    interface Token {
        enum TokenType { Fungible, NonFungible }
        struct Coin { string obverse; string reverse; }
        function transfer(address recipient, uint amount) external;
    }

Contracts can inherit interfaces as they would inherit other contracts.

Types defined inside interfaces and other contract-like structures
can be accessed from other contracts: ``Token.TokenType`` or ``Token.Coin``.

.. index:: ! library, callcode, delegatecall

.. _libraries:

*********
Libraries
*********

Libraries are similar to contracts, but their purpose is that they are deployed
only once at a specific address and their code is reused using the ``DELEGATECALL``
(``CALLCODE`` until Homestead)
feature of the EVM. This means that if library functions are called, their code
is executed in the context of the calling contract, i.e. ``this`` points to the
calling contract, and especially the storage from the calling contract can be
accessed. As a library is an isolated piece of source code, it can only access
state variables of the calling contract if they are explicitly supplied (it
would have no way to name them, otherwise). Library functions can only be
called directly (i.e. without the use of ``DELEGATECALL``) if they do not modify
the state (i.e. if they are ``view`` or ``pure`` functions),
because libraries are assumed to be stateless. In particular, it is
not possible to destroy a library.

.. note::
    Until version 0.4.20, it was possible to destroy libraries by
    circumventing Solidity's type system. Starting from that version,
    libraries contain a :ref:`mechanism<call-protection>` that
    disallows state-modifying functions
    to be called directly (i.e. without ``DELEGATECALL``).

Libraries can be seen as implicit base contracts of the contracts that use them.
They will not be explicitly visible in the inheritance hierarchy, but calls
to library functions look just like calls to functions of explicit base
contracts (``L.f()`` if ``L`` is the name of the library). Furthermore,
``internal`` functions of libraries are visible in all contracts, just as
if the library were a base contract. Of course, calls to internal functions
use the internal calling convention, which means that all internal types
can be passed and types :ref:`stored in memory <data-location>` will be passed by reference and not copied.
To realize this in the EVM, code of internal library functions
and all functions called from therein will at compile time be pulled into the calling
contract, and a regular ``JUMP`` call will be used instead of a ``DELEGATECALL``.

.. index:: using for, set

The following example illustrates how to use libraries (but manual method
be sure to check out :ref:`using for <using-for>` for a
more advanced example to implement a set).

::

    pragma solidity >=0.4.22 <0.6.0;

    library Set {
      // We define a new struct datatype that will be used to
      // hold its data in the calling contract.
      struct Data { mapping(uint => bool) flags; }

      // Note that the first parameter is of type "storage
      // reference" and thus only its storage address and not
      // its contents is passed as part of the call.  This is a
      // special feature of library functions.  It is idiomatic
      // to call the first parameter `self`, if the function can
      // be seen as a method of that object.
      function insert(Data storage self, uint value)
          public
          returns (bool)
      {
          if (self.flags[value])
              return false; // already there
          self.flags[value] = true;
          return true;
      }

      function remove(Data storage self, uint value)
          public
          returns (bool)
      {
          if (!self.flags[value])
              return false; // not there
          self.flags[value] = false;
          return true;
      }

      function contains(Data storage self, uint value)
          public
          view
          returns (bool)
      {
          return self.flags[value];
      }
    }

    contract C {
        Set.Data knownValues;

        function register(uint value) public {
            // The library functions can be called without a
            // specific instance of the library, since the
            // "instance" will be the current contract.
            require(Set.insert(knownValues, value));
        }
        // In this contract, we can also directly access knownValues.flags, if we want.
    }

Of course, you do not have to follow this way to use
libraries: they can also be used without defining struct
data types. Functions also work without any storage
reference parameters, and they can have multiple storage reference
parameters and in any position.

The calls to ``Set.contains``, ``Set.insert`` and ``Set.remove``
are all compiled as calls (``DELEGATECALL``) to an external
contract/library. If you use libraries, be aware that an
actual external function call is performed.
``msg.sender``, ``msg.value`` and ``this`` will retain their values
in this call, though (prior to Homestead, because of the use of ``CALLCODE``, ``msg.sender`` and
``msg.value`` changed, though).

The following example shows how to use :ref:`types stored in memory <data-location>` and
internal functions in libraries in order to implement
custom types without the overhead of external function calls:

::

    pragma solidity >=0.4.16 <0.6.0;

    library BigInt {
        struct bigint {
            uint[] limbs;
        }

        function fromUint(uint x) internal pure returns (bigint memory r) {
            r.limbs = new uint[](1);
            r.limbs[0] = x;
        }

        function add(bigint memory _a, bigint memory _b) internal pure returns (bigint memory r) {
            r.limbs = new uint[](max(_a.limbs.length, _b.limbs.length));
            uint carry = 0;
            for (uint i = 0; i < r.limbs.length; ++i) {
                uint a = limb(_a, i);
                uint b = limb(_b, i);
                r.limbs[i] = a + b + carry;
                if (a + b < a || (a + b == uint(-1) && carry > 0))
                    carry = 1;
                else
                    carry = 0;
            }
            if (carry > 0) {
                // too bad, we have to add a limb
                uint[] memory newLimbs = new uint[](r.limbs.length + 1);
                uint i;
                for (i = 0; i < r.limbs.length; ++i)
                    newLimbs[i] = r.limbs[i];
                newLimbs[i] = carry;
                r.limbs = newLimbs;
            }
        }

        function limb(bigint memory _a, uint _limb) internal pure returns (uint) {
            return _limb < _a.limbs.length ? _a.limbs[_limb] : 0;
        }

        function max(uint a, uint b) private pure returns (uint) {
            return a > b ? a : b;
        }
    }

    contract C {
        using BigInt for BigInt.bigint;

        function f() public pure {
            BigInt.bigint memory x = BigInt.fromUint(7);
            BigInt.bigint memory y = BigInt.fromUint(uint(-1));
            BigInt.bigint memory z = x.add(y);
            assert(z.limb(1) > 0);
        }
    }

As the compiler cannot know where the library will be
deployed at, these addresses have to be filled into the
final bytecode by a linker
(see :ref:`commandline-compiler` for how to use the
commandline compiler for linking). If the addresses are not
given as arguments to the compiler, the compiled hex code
will contain placeholders of the form ``__Set______`` (where
``Set`` is the name of the library). The address can be filled
manually by replacing all those 40 symbols by the hex
encoding of the address of the library contract.

.. note::
    Manually linking libraries on the generated bytecode is discouraged, because
    it is restricted to 36 characters.
    You should ask the compiler to link the libraries at the time
    a contract is compiled by either using
    the ``--libraries`` option of ``solc`` or the ``libraries`` key if you use
    the standard-JSON interface to the compiler.

Restrictions for libraries in comparison to contracts:

- No state variables
- Cannot inherit nor be inherited
- Cannot receive Ether

(These might be lifted at a later point.)

.. _call-protection:

Call Protection For Libraries
=============================

As mentioned in the introduction, if a library's code is executed
using a ``CALL`` instead of a ``DELEGATECALL`` or ``CALLCODE``,
it will revert unless a ``view`` or ``pure`` function is called.

The EVM does not provide a direct way for a contract to detect
whether it was called using ``CALL`` or not, but a contract
can use the ``ADDRESS`` opcode to find out "where" it is
currently running. The generated code compares this address
to the address used at construction time to determine the mode
of calling.

More specifically, the runtime code of a library always starts
with a push instruction, which is a zero of 20 bytes at
compilation time. When the deploy code runs, this constant
is replaced in memory by the current address and this
modified code is stored in the contract. At runtime,
this causes the deploy time address to be the first
constant to be pushed onto the stack and the dispatcher
code compares the current address against this constant
for any non-view and non-pure function.

.. index:: ! using for, library

.. _using-for:

*********
Using For
*********

The directive ``using A for B;`` can be used to attach library
functions (from the library ``A``) to any type (``B``).
These functions will receive the object they are called on
as their first parameter (like the ``self`` variable in Python).

The effect of ``using A for *;`` is that the functions from
the library ``A`` are attached to *any* type.

In both situations, *all* functions in the library are attached,
even those where the type of the first parameter does not
match the type of the object. The type is checked at the
point the function is called and function overload
resolution is performed.

The ``using A for B;`` directive is active only within the current
contract, including within all of its functions, and has no effect
outside of the contract in which it is used. The directive
may only be used inside a contract, not inside any of its functions.

By including a library, its data types including library functions are
available without having to add further code.

Let us rewrite the set example from the
:ref:`libraries` in this way::

    pragma solidity >=0.4.16 <0.6.0;

    // This is the same code as before, just without comments
    library Set {
      struct Data { mapping(uint => bool) flags; }

      function insert(Data storage self, uint value)
          public
          returns (bool)
      {
          if (self.flags[value])
            return false; // already there
          self.flags[value] = true;
          return true;
      }

      function remove(Data storage self, uint value)
          public
          returns (bool)
      {
          if (!self.flags[value])
              return false; // not there
          self.flags[value] = false;
          return true;
      }

      function contains(Data storage self, uint value)
          public
          view
          returns (bool)
      {
          return self.flags[value];
      }
    }

    contract C {
        using Set for Set.Data; // this is the crucial change
        Set.Data knownValues;

        function register(uint value) public {
            // Here, all variables of type Set.Data have
            // corresponding member functions.
            // The following function call is identical to
            // `Set.insert(knownValues, value)`
            require(knownValues.insert(value));
        }
    }

It is also possible to extend elementary types in that way::

    pragma solidity >=0.4.16 <0.6.0;

    library Search {
        function indexOf(uint[] storage self, uint value)
            public
            view
            returns (uint)
        {
            for (uint i = 0; i < self.length; i++)
                if (self[i] == value) return i;
            return uint(-1);
        }
    }

    contract C {
        using Search for uint[];
        uint[] data;

        function append(uint value) public {
            data.push(value);
        }

        function replace(uint _old, uint _new) public {
            // This performs the library function call
            uint index = data.indexOf(_old);
            if (index == uint(-1))
                data.push(_new);
            else
                data[index] = _new;
        }
    }

Note that all library calls are actual EVM function calls. This means that
if you pass memory or value types, a copy will be performed, even of the
``self`` variable. The only situation where no copy will be performed
is when storage reference variables are used