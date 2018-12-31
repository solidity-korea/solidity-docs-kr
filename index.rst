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
    Solidity 컴파일러가 포함된 Microsoft Visual Studio 플러그인

* `Package for SublimeText — Solidity language syntax <https://packagecontrol.io/packages/Ethereum/>`_
    Sublime Text 를 위한 Solidity 문법 강조기

* `Etheratom <https://github.com/0mkara/etheratom>`_
    문법 강조, 편집, 실행 환경 (백엔드 노드 및 VM 과 호환 가능한) 을 제공하는 Atom editor 플러그인

* `Atom Solidity Linter <https://atom.io/packages/linter-solidity>`_
    Solidity linting 을 제공하는 Atom editor 플러그인

* `Atom Solium Linter <https://atom.io/packages/linter-solium>`_
    Solium 기반으로, 사용자 설정이 가능한 Atom editor 용 Solidity linter

* `Solium <https://github.com/duaraghav8/Solium/>`_
    Solidity 에서 코드 스타일이나 보안 이슈를 수정하고 확인하기 위한 linter

* `Solhint <https://github.com/protofire/solhint>`_
    Smart Contract 검증을 위한 Solidity linter. 보안 사항 및 스타일 가이드, 최적의 관례 사항(역주: for-loop 에서 index 변수명을 i 로 축약하는 것 등)을 제공함.

* `Visual Studio Code extension <http://juan.blanco.ws/solidity-contracts-in-visual-studio-code/>`_
    문법 강조 기능과 컴파일러를 제공하는 Microsoft Visual Studio Code 플러그인

* `Emacs Solidity <https://github.com/ethereum/emacs-solidity/>`_
    문법 강조 기능과 편집 에러 알림을 제공하는 Emacs editor 플러그인

* `Vim Solidity <https://github.com/tomlion/vim-solidity/>`_
    문법 강조 기능을 제공하는 Vim editor 플러그인

* `Vim Syntastic <https://github.com/scrooloose/syntastic>`_
    컴파일 확인이 가능한 Vim editor 플러그인

지원이 중지된 도구들:

* `Mix IDE <https://github.com/ethereum/mix/>`_
    스마트 컨트랙에 대해 디자인, 디버깅, 테스팅이 가능한 Qt 기반의 IDE

* `Ethereum Studio <https://live.ether.camp/>`_
    완벽한 Ethereum 환경에 대한 shell 액세스를 제공하는 특수(특화된) 웹 IDE.
    Specialized web IDE that also provides shell access to a complete Ethereum environment.

Solidity 도구들
--------------

* `Dapp <https://dapp.readthedocs.io>`_
    Solidity 를 위한 빌드 도구, 패키지 매니저, 배포 도우미 도구

* `Solidity REPL <https://github.com/raineorshine/solidity-repl>`_
    커맨드 라인 기반으로 Solidity 를 바로 사용해볼 수 있는 도구

* `solgraph <https://github.com/raineorshine/solgraph>`_
    Solidity 흐름을 시각화 하고, 잠재적인 보안 위협을 강조해주는 도구

* `evmdis <https://github.com/Arachnid/evmdis>`_
    Raw EVM operations 보다 높은 추상화를 제공하기 위해 바이트 코드에 직접 정적 분석을 수행하는 EVM Disassembler

* `Doxity <https://github.com/DigixGlobal/doxity>`_
    Solidity 를 위한 문서 생성기

서드파티 Solidity 파서와 문법
-----------------------------------------

* `solidity-parser <https://github.com/ConsenSys/solidity-parser>`_
    Javascript 를 위한 Solidity 파서

* `Solidity Grammar for ANTLR 4 <https://github.com/federicobond/solidity-antlr4>`_
    Solidity grammar for the ANTLR 4 parser generator

Language Documentation
----------------------

다음 페이지들 부터, Solidity 로 작성된 :ref:`간단한 smart contract <simple-smart-contract>` 과 :ref:`blockchains <blockchain-basics>`의 기초, 그리고 :ref:`Ethereum Virtual Machine <the-ethereum-virtual-machine>`. 에 대해서 알아보도록 하겠습니다.

다음 섹션은 Solidity 에서 제공하는 몇가지 유용한 *기능*을 살펴보겠습니다. :ref:`example contracts <voting>`
또한 지금 사용하시는 `브라우저 <https://remix.ethereum.org>`_! 에서도 저희 코드를 실행시켜볼 수 있다는 것을 잊지마세요!

마지막 섹션에서는 Solidity 의 모든 측면에 대해서 심도 있게 다룹니다.

이외에 질문이 있으시다면, `Ethereum Stackexchange <https://ethereum.stackexchange.com/>`_ 에서 검색이나 직접 질문하실 수 있으며 `gitter 채널 <https://gitter.im/ethereum/solidity/>`_ 에서도 가능합니다.

Solidity 나 이 문서에 대해 발전을 위한 아이디어는 항상 환영합니다. :)


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
   resource.rst
   using-the-compiler.rst
   metadata.rst
   abi-spec.rst
   yul.rst
   style-guide.rst
   common-patterns.rst
   bugs.rst
   contributing.rst
   frequently-asked-questions.rst
   lll.rst
