.. index:: ! installing

.. _installing-solidity:

################################
Solidity 컴파일러 설치하기
################################

버전 관리
==========

Solidity 버전 관리는 `semantic versioning <https://semver.org>`_ 를 따르며 **nightly development builds** 가 배포될 수 도 있습니다.
nightly builds는 작동이 보장되지 않고 최선의 노력을 다하겠지만 문서화되지 않거나 잘못된 변경사항이 있을 수 있습니다.
우리는 최신 배포판을 사용하길 권장합니다. 아래의 패키지 인스톨러는 최신판을 사용할 예정입니다.

Remix
=====

*간단한 컨트랙트 작성과 빠른 Solidity 학습에는 Remix를 추천합니다.*

`Remix 온라인에 접속하세요 <https://remix.ethereum.org/>`_, 아무것도 설치 할 필요 없습니다.
인터넷 접속이 안되는 환경에서 Remix를 사용하길 원한다면, https://github.com/ethereum/remix-live/tree/gh-pages 로 가서 ``.zip`` 파일을 내려받으세요.

이 페이지의 다음 항목들은 커맨드라인 Solidity 컴파일러 소프트웨어를 설치하는 내용을 열거하고 있습니다.
큰 규모의 컨트랙트를 작업하거나 더 많은 편집 옵션이 필요하다면 커맨드라인 컴파일러를 사용하세요.

.. _solcjs:

npm / Node.js
=============

Solidity 컴파일러인 `solcjs` 를 설치하려면 간편한 `npm` 을 사용하세요.
`solcjs` 프로그램은 현 페이지 아래에 설명 된 컴파일러에 접근하는 방법보다 더 적은 기능을 가지고 있습니다.
:ref:`commandline-compiler` 문서는 여러분이 모든 기능을 갖춘 컴파일러 `solc` 을 사용한다고 가정합니다.
`solcjs` 의 사용법은 `저장소 <https://github.com/ethereum/solc-js>`_ 에 문서화 되어 있습니다.

주석: solc-js 프로젝트는 Emscripten을 사용하는 C++ `solc` 에서 파생되었습니다. 
solc-js와 Emscripten는 동일한 컴파일러 소스 코드를 사용합니다. 
`solc-js` 는 Javascript 프로젝트에서 사용될 수 있습니다(예: Remix). 자세한 내용은 solc-js 저장소를 참조하십시오.

.. code-block:: bash

    npm install -g solc

.. note::

    실행가능한 커맨드라인의 이름은 `solcjs` 입니다.

    `solc` 의 명령어가 `solcjs` 에서 작동하지 않듯이
    `solcjs` 의 커맨드라인 옵션은 `solc` 그리고 `geth` 같은 도구들에서 호환되지 않습니다.
    
Docker
======

우리는 컴파일러가 포함된 최신 Docker 빌드를 제공합니다.
``stable`` 저장소에는 배포된 버전이 포함되어 있으며
``nightly`` 저장소에는 잠재적으로 불안정한 변경사항이 포함된 개발 브랜치의 버전이 포함되어 있습니다.

.. code-block:: bash

    docker run ethereum/solc:stable --version

현재, Docker 이미지에는 컴파일러 실행 파일만 포함되어 있습니다,
그러므로 소스와 출력 디렉터리를 연결하기 위해선 몇 가지 추가 작업을 해야 합니다.

바이너리 패키지
===============

Solidity 바이너리 패키지는 `solidity/releases <https://github.com/ethereum/solidity/releases>`_ 에서 이용 가능합니다.

Ubuntu에서 사용 가능한 PPA도 있습니다. 안정된 최신 버전은 아래 명령어를 통해 설치가능합니다.

.. code-block:: bash

    sudo add-apt-repository ppa:ethereum/ethereum
    sudo apt-get update
    sudo apt-get install solc

개발중인 최신 버전을 사용하고 싶다면 아래 명령어를 이용하세요:

.. code-block:: bash

    sudo add-apt-repository ppa:ethereum/ethereum
    sudo add-apt-repository ppa:ethereum/ethereum-dev
    sudo apt-get update
    sudo apt-get install solc

우리는 또한 `snap package <https://snapcraft.io/>`_ 를 배포하고 있습니다. 이 패키지는 `지원되는 모든 Linux 배포판 <https://snapcraft.io/docs/core/install>`_ 에 설치할 수 있습니다. solc의 안정된 최신 버전을 설치하려면 다음 명령어를 이용하세요:

.. code-block:: bash

    sudo snap install solc

최신 변경사항이 포함된 Solidity의 최신 개발버전을 테스트하는 데 도움을 주고 싶다면 다음을 따르세요:

.. code-block:: bash

    sudo snap install solc --edge

개발 중인 최신 버전뿐이지만 Arch Linux 역시 패키지가 있습니다:

.. code-block:: bash

    pacman -S solidity

우리는 Homebrew를 통해 Solidity 컴파일러를 build-from-source 버전으로 배포합니다.
pre-built bottles 는 현재 지원되지 않습니다.

.. code-block:: bash

    brew update
    brew upgrade
    brew tap ethereum/ethereum
    brew install solidity

Solidity의 특정 버전이 필요한 경우, 깃허브에서 직접 Homebrew formula를 설치할 수 있습니다.

`깃허브의 solidity.rb 커밋 내역 <https://github.com/ethereum/homebrew-ethereum/commits/master/solidity.rb>`_ 을 참조하세요.

``solidity.rb`` 의 특정 커밋의 raw file 링크를 찾을 때까지 히스토리 링크를 따라가세요.

``brew`` 를 사용하여 설치하십시오:

.. code-block:: bash

    brew unlink solidity
    # Install 0.4.8
    brew install https://raw.githubusercontent.com/ethereum/homebrew-ethereum/77cce03da9f289e5a3ffe579840d3c5dc0a62717/solidity.rb

Gentoo Linux 또한 ``emerge`` 를 이용해 설치할 수 있는 Solidity 패키지를 제공합니다:

.. code-block:: bash

    emerge dev-lang/solidity

.. _building-from-source:

소스에서 빌드하기
====================

필수 설치 항목 - Linux
---------------------
Solidity를 Linux에서 빌드 하기위해 다음 의존 항목을 설치해야 합니다:

+-----------------------------------+-------------------------------------------------+
| 소프트웨어                        | 설명                                            |
+===================================+=================================================+
| `Git for Linux`_                  | Github에서 소스를 검색하기 위한 커맨드라인 툴   |
+-----------------------------------+-------------------------------------------------+

.. _Git for Linux: https://git-scm.com/download/linux


필수 설치 항목 - macOS
---------------------

macOS의 경우, 반드시 최신 버전의 `Xcode<https://developer.apple.com/xcode/download/>`_ 가 설치되어야 합니다.
여기에는 `Clang C++ compiler <https://en.wikipedia.org/wiki/Clang>`_, `Xcode IDE <https://en.wikipedia.org/wiki/Xcode>`_ 와 그 외 OS X에서 C++ 애플리케이션을 빌드하기 위한 애플 개발도구들이 포함되어 있습니다.
Xcode를 처음 설치하거나 새 버전을 설치했다면, 커맨드라인에서 빌드하기 전 라이선스에 동의해야 합니다:

.. code-block:: bash

    sudo xcodebuild -license accept

외부 의존 항목을 설치하기 위해 OSX 빌드는 `Homebrew <http://brew.sh>`_ 패키지 매니저를 필요로 합니다.
혹시 처음부터 다시 시작하고 싶다면, 여기 `Homebrew 삭제 <https://github.com/Homebrew/homebrew/blob/master/share/doc/homebrew/FAQ.md#how-do-i-uninstall-homebrew>`_ 하는 방법입니다.

필수 설치 항목 - Windows
-----------------------

Solidity의 Windows 빌드를 위해 아래의 의존 항목들을 설치해야 합니다:

+-----------------------------------+--------------------------------------------------+
| 소프트웨어                        | 설명                                             |
+===================================+==================================================+
| `Git for Windows`_                | Github에서 소스를 검색하기 위한 커맨드라인 도구. |
+-----------------------------------+--------------------------------------------------+
| `CMake`_                          | 크로스 플랫폼 빌드 파일 생성기.                  |
+-----------------------------------+--------------------------------------------------+
| `Visual Studio 2017 Build Tools`_ | C++ 컴파일러                                     |
+-----------------------------------+--------------------------------------------------+
| `Visual Studio 2017`_ (Optional)  | C++ 컴파일러 및 개발 환경.                       |
+-----------------------------------+--------------------------------------------------+

IDE가 하나 뿐이고, 컴파일러와 라이브러리만 있으면, 
Visual Studio 2017 Build Tools를 설치할 수 있습니다.

Visual Studio 2017은 IDE와 필요한 컴파일러와 라이브러리를 모두 제공합니다.
따라서 IDE가 없는 상황에서 Solidity 개발을 생각하고 있다면, 
Visual Studio 2017이 모든 setup을 쉽게 해 줄 수 있는 선택이 될 것입니다.

다음은 Visual Studio 2017 Build Tools나 Visual Studio 2017에서 
설치해야하는 구성요소 목록입니다.

* Visual Studio C++ core features
* VC++ 2017 v141 toolset (x86,x64)
* Windows Universal CRT SDK
* Windows 8.1 SDK
* C++/CLI support

.. _Git for Windows: https://git-scm.com/download/win
.. _CMake: https://cmake.org/download/
.. _Visual Studio 2017: https://www.visualstudio.com/vs/
.. _Visual Studio 2017 Build Tools: https://www.visualstudio.com/downloads/#build-tools-for-visual-studio-2017

저장소에서 Clone
--------------------

소스코드를 Clone 하기 위해서는, 다음 명령어를 수행하세요:

.. code-block:: bash

    git clone --recursive https://github.com/ethereum/solidity.git
    cd solidity

Solidity 개발을 돕고싶다면,
Solidity를 포크하고, 자신의 포크를 second 원격으로 추가하세요:

.. code-block:: bash

    git remote add personal git@github.com:[username]/solidity.git


외부 의존 항목
---------------------

macOS, Windows 외 수많은 Linux 배포판에 필요한 모든 
외부 의존 항목을 설치하는 도우미 스크립트가 있습니다.

.. code-block:: bash

    ./scripts/install_deps.sh

Windows에선 아래와 같습니다:

.. code-block:: bat

    scripts\install_deps.bat


커맨드라인 빌드
------------------

** 빌드하기 전 외부 의존 항목을(윗부분 참조) 반드시 설치해야 합니다.**

Solidity 프로젝트는 빌드를 구성하기 위해 CMake를 사용합니다.
반복된 빌드 속도를 높이기 위해서 ccache를 설치하는 것이 좋습니다.
CMake가 자동으로 ccache를 선택할것입니다.
Solidity 빌드는 Linux, macOS 및 기타 Unix에서 매우 유사하게 진행됩니다:

.. code-block:: bash

    mkdir build
    cd build
    cmake .. && make

또는 조금 더 쉬운 방법:

.. code-blcok:: bash
    
    #주석: 이 명령어는 바이너리 solc와 soltest를 usr/local/bin에 설치할 것입니다.
    ./scripts/build.sh

Windows에서는:

.. code-block:: bash

    mkdir build
    cd build
    cmake -G "Visual Studio 15 2017 Win64" ..

이 명령어의 결과로 해당 빌드 디렉터리에 **solidity.sln** 가 생성됩니다.
이 파일을 더블클릭하면 Visual Studio가 실행됩니다.
우리는 **Release** 환경설정을 빌드하는 걸 제안합니다.

또 다른 방법으로는 Windows 커맨드라인에서 아래와같이 빌드를 진행할 수 있습니다:

.. code-block:: bash

    cmake --build . --config Release

CMake 옵션
=============

CMake 옵션을 알고 싶다면 ``cmake .. -LH`` 명령어를 실행하십시오.

.. _stm_solvers_build:

STM Solvers
-----------
Solidity는 SMT solvers에 대해 빌드 될 수 있으며, 시스템에서 발견되면
디폴트로 수행될것입니다. 각 solver는 `cmake` 옵션에 의해 비활성화 될 수 있습니다.

*주석 : 경우에 따라 빌드 실패의 잠재적인 해결방법이 될 수도 있습니다.*


빌드 폴더에서는 디폴트로 사용하도록 설정되어 있기 때문에, 사용하지 않도록 설정 할 수 있습니다.

.. code-block:: bash

    # Z3 SMT Solver 만 비활성화
    cmake .. -DUSE_Z3=OFF

    # CVC4 SMT Solver 만 비활성화
    cmake .. -DUSE_CVC4=OFF

    # Z3와 CVC4 모두 비활성화
    cmake .. -DUSE_CVC4=OFF -DUSE_Z3=OFF


버전 문자열 상세하게 보기
============================

Solidity 버전 문자열은 네 부분으로 구성됩니다:

- 버전 숫자
- pre-release 태그, 대개 ``develop.YYYY.MM.DD`` 나 ``nightly.YYYY.MM.DD`` 형태를 지님
- 다음과 같은 형태의 커밋 ``commit.GITHASH``
- 플랫폼 및 컴파일러에 대한 세부 정보를 포함하는 몇 가지 항목

로컬에서 수정된 부분이 있다면, 커밋 뒤에 ``.mod`` 가 붙습니다.

이 부분들은 Semver(Semantic Versioning)에 따라 필요에 의해 결합됩니다. 여기서 Solidity pre-release 태그는 Semver의 pre-release 태그와 같고
Solidity 커밋 및 플랫폼은 결합되어 Semver 빌드 메타데이터를 구성합니다.

release 예: ``0.4.8+commit.60cc1668.Emscripten.clang``.

pre-release 예: ``0.4.9-nightly.2017.1.17+commit.6ecb4aa3.Emscripten.clang``

버전 관리에 대한 중요한 정보
======================================

릴리즈가 일어난 후에, 패치 버전은 변경된다.
변경사항이 합쳐질 때, 버전은 semver와 변경 정도에 따라 변경된다.
따라서, 배포는 항상 ``prerelease`` 태그를 제외한 현재의 nightly build버전으로 이루어진다.

예:

0. 0.4.0가 배포된다
1. 지금부터 nightly build는 0.4.1 버전이다
2. 어떠한 변경사항이 없을 경우 - 버전은 변화가 없다
3. 변경사항이 있을 경우 - 버전은 0.5.0이 된다
4. 0.5.0가 배포된다

이 동작은 :ref:`version pragma <version_pragma>` 와 함께 작동합니다.
