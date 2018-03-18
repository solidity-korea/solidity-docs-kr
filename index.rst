Solidity
========

.. image:: logo.svg
    :width: 120px
    :alt: Solidity logo
    :align: center

Solidity는 스마트 컨트랙트를 구현하기 위한 컨트랙트 기반의 고급 프로그래밍 언어입니다.
Solidity는 C++, Python, 그리고 JavaScript의 영향을 받아 만들어졌습니다.
그리고 Ethereum Virtual Machine(EVM)에서 구동되도록 설계되었습니다.

Solidity는 정적 타입이며, 상속, 라이브러리 그리고 복잡한 사용자 정의 자료형을 지원합니다.

문서에서 살펴볼 수 있듯이 투표, 크라우드 펀딩, 블라인드 옥션,
멀티 시그 월랫 등 다양한 컨트랙트를 작성할 수 있습니다.

.. note::
    Solidity를 연습하기 가장 좋은 방법은 현재
    `Remix <https://remix.ethereum.org/>`_
    (로딩되는데 다소 시간이 걸릴 수 있습니다.)를 사용하는 것입니다.
    Remix는 Solidity 스마트 컨트랙트를 작성하고, 배포하고, 실행할 수 있는 웹 브라우저 기반의 IDE입니다.

.. warning::
    소프트웨어는 사람에 의해 만들어지기 때문에 버그가 생길 수 있습니다. 따라서 스마트 컨트랙트는 잘 알려진 모범사례들을 참고하여 작성되어야합니다.
    스마트 컨트랙트를 작성할 때는 코드리뷰, 테스팅, 회고 그리고 정확성 증명을 해야합니다. 또한 사용자가 코드 작성자보다 코드를 더 신뢰하는 경우가 있다는 것을 기억해야합니다.
    마지막으로, 블록체인 자체적으로 주의해야할 사항들이 있습니다. 다음 섹션을 참조해 주세요. :ref:`security_considerations`.

Notice for Korean
------------

아직 번역이 진행중입니다. 누구나 참여하실 수 있으며 해당 `solidity-korea/solidity-docs-kr repo <https://github.com/solidity-korea/solidity-docs-kr>`_ 에 편하게 Pull Request 주셔서 참여하실 수 있습니다.

번역
------------

This documentation is translated into several languages by community volunteers, but the English version stands as a reference.

* `Simplified Chinese <http://solidity-cn.readthedocs.io>`_ (in progress)
* `Spanish <https://solidity-es.readthedocs.io>`_
* `Russian <https://github.com/ethereum/wiki/wiki/%5BRussian%5D-%D0%A0%D1%83%D0%BA%D0%BE%D0%B2%D0%BE%D0%B4%D1%81%D1%82%D0%B2%D0%BE-%D0%BF%D0%BE-Solidity>`_ (rather outdated)


유용한 링크
------------

* `Ethereum <https://ethereum.org>`_

* `Changelog <https://github.com/ethereum/solidity/blob/develop/Changelog.md>`_

* `Story Backlog <https://www.pivotaltracker.com/n/projects/1189488>`_

* `Source Code <https://github.com/ethereum/solidity/>`_

* `Ethereum Stackexchange <https://ethereum.stackexchange.com/>`_

* `Gitter Chat <https://gitter.im/ethereum/solidity/>`_

Solidity 통합 도구들
-------------------------------

* `Remix <https://remix.ethereum.org/>`_
    별도의 서버없이 컴파일러와 런타임 환경을 제공하는 브라우저 기반의 IDE

* `IntelliJ IDEA plugin <https://plugins.jetbrains.com/plugin/9475-intellij-solidity>`_
    IntelliJ IDEA 를 위한 Solidity 플러그인 (기타 모든 JetBrains IDE 포함)

* `Visual Studio Extension <https://visualstudiogallery.msdn.microsoft.com/96221853-33c4-4531-bdd5-d2ea5acc4799/>`_
    Solidity plugin for Microsoft Visual Studio that includes the Solidity compiler.

* `Package for SublimeText — Solidity language syntax <https://packagecontrol.io/packages/Ethereum/>`_
    Solidity syntax highlighting for SublimeText editor.

* `Etheratom <https://github.com/0mkara/etheratom>`_
    Plugin for the Atom editor that features syntax highlighting, compilation and a runtime environment (Backend node & VM compatible).

* `Atom Solidity Linter <https://atom.io/packages/linter-solidity>`_
    Plugin for the Atom editor that provides Solidity linting.

* `Atom Solium Linter <https://atom.io/packages/linter-solium>`_
    Configurable Solidty linter for Atom using Solium as a base.

* `Solium <https://github.com/duaraghav8/Solium/>`_
    Linter to identify and fix style and security issues in Solidity.
    
* `Solhint <https://github.com/protofire/solhint>`_
    Solidity linter that provides security, style guide and best practice rules for smart contract validation.

* `Visual Studio Code extension <http://juan.blanco.ws/solidity-contracts-in-visual-studio-code/>`_
    Solidity plugin for Microsoft Visual Studio Code that includes syntax highlighting and the Solidity compiler.

* `Emacs Solidity <https://github.com/ethereum/emacs-solidity/>`_
    Plugin for the Emacs editor providing syntax highlighting and compilation error reporting.

* `Vim Solidity <https://github.com/tomlion/vim-solidity/>`_
    Plugin for the Vim editor providing syntax highlighting.

* `Vim Syntastic <https://github.com/scrooloose/syntastic>`_
    Plugin for the Vim editor providing compile checking.

Discontinued:

* `Mix IDE <https://github.com/ethereum/mix/>`_
    Qt based IDE for designing, debugging and testing solidity smart contracts.

* `Ethereum Studio <https://live.ether.camp/>`_		
    Specialized web IDE that also provides shell access to a complete Ethereum environment.

Solidity Tools
--------------

* `Dapp <https://dapp.readthedocs.io>`_
    Build tool, package manager, and deployment assistant for Solidity.

* `Solidity REPL <https://github.com/raineorshine/solidity-repl>`_
    Try Solidity instantly with a command-line Solidity console.

* `solgraph <https://github.com/raineorshine/solgraph>`_
    Visualize Solidity control flow and highlight potential security vulnerabilities.

* `evmdis <https://github.com/Arachnid/evmdis>`_
    EVM Disassembler that performs static analysis on the bytecode to provide a higher level of abstraction than raw EVM operations.

* `Doxity <https://github.com/DigixGlobal/doxity>`_
    Documentation Generator for Solidity.

Third-Party Solidity Parsers and Grammars
-----------------------------------------

* `solidity-parser <https://github.com/ConsenSys/solidity-parser>`_
    Solidity parser for JavaScript

* `Solidity Grammar for ANTLR 4 <https://github.com/federicobond/solidity-antlr4>`_
    Solidity grammar for the ANTLR 4 parser generator

Language Documentation
----------------------

On the next pages, we will first see a :ref:`simple smart contract <simple-smart-contract>` written
in Solidity followed by the basics about :ref:`blockchains <blockchain-basics>`
and the :ref:`Ethereum Virtual Machine <the-ethereum-virtual-machine>`.

The next section will explain several *features* of Solidity by giving
useful :ref:`example contracts <voting>`
Remember that you can always try out the contracts
`in your browser <https://remix.ethereum.org>`_!

The last and most extensive section will cover all aspects of Solidity in depth.

If you still have questions, you can try searching or asking on the
`Ethereum Stackexchange <https://ethereum.stackexchange.com/>`_
site, or come to our `gitter channel <https://gitter.im/ethereum/solidity/>`_.
Ideas for improving Solidity or this documentation are always welcome!

Contents
========

:ref:`Keyword Index <genindex>`, :ref:`Search Page <search>`

.. toctree::
   :maxdepth: 2

   introduction-to-smart-contracts.rst
   installing-solidity.rst
   solidity-by-example.rst
   solidity-in-depth.rst
   security-considerations.rst
   using-the-compiler.rst
   metadata.rst
   abi-spec.rst
   julia.rst
   style-guide.rst
   common-patterns.rst
   bugs.rst
   contributing.rst
   frequently-asked-questions.rst
