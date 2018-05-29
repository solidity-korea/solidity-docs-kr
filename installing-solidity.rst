.. index:: ! installing

.. _installing-solidity:

################################
솔리디티 컴파일러 설치하기
################################

버전 관리
==========

솔리디티 버전 관리는 `semantic versioning <https://semver.org>`_ 를 따르며 **nightly development builds** 가 배포될 수 도 있습니다.
nightly builds는 작동이 보장되지 않고 최선의 노력을 다하겠지만 문서화되지 않거나 잘못된 변경사항이 있을 수 있습니다.
우리는 최신 배포판을 사용하길 권장합니다. 아래의 패키지 인스톨러는 최신판을 사용할 예정입니다.

리믹스
=====

*간단한 컨트랙트 작성과 빠른 솔리디티 학습에는 리믹스를 추천합니다.*

`리믹스 온라인에 접속하세요 <https://remix.ethereum.org/>`_, 아무것도 설치 할 필요 없습니다.
인터넷 접속이 안되는 환경에서 리믹스를 사용하길 원한다면, https://github.com/ethereum/browser-solidity/tree/gh-pages 로 가서 .ZIP파일을 내려받으세요.

이 페이지의 다음 항목들은 커맨드라인 솔리디티 컴파일러 소프트웨어를 설치하는 내용을 열거하고 있습니다.
큰 규모의 컨트랙트를 작업하거나 더 많은 편집 옵션이 필요하다면 커맨드라인 컴파일러를 사용하세요.

.. _solcjs:

npm / Node.js
=============

솔리디티 컴파일러인 `solcjs`를 설치하려면 간편한 `npm`을 사용하세요.
페이지 아래에 열거된 다른 방법으로 설치하는 것보다 간단한 만큼 `solcjs`은 간소화된 기능을 갖추고 있습니다.
:ref:`commandline-compiler` 문서는 당신이 모든 기능을 갖춘 컴파일러를 사용하고 있다고 가정합니다.
그러므로 `npm`을 통해 `solcjs`를 설치했다면 이 문서를 읽는 걸 멈추고 `solc-js <https://github.com/ethereum/solc-js>`_ 여기서 계속 진행하세요.

주석: solc-js 프로젝트는 Emscripten을 사용하는 C++ `solc`에서 파생되었습니다. 
`solc-js`는 자바스크립트 프로젝트에서 사용될 수 있습니다(예: 리믹스). 자세한 내용은 solc-js 저장소를 참조하십시오.

.. code:: bash

    npm install -g solc

.. note::

    커맨드라인의 이름은 `solcjs`입니다.

    `solc`의 명령어가 `solcjs`에서 작동하지 않듯이
    `solcjs`의 커맨드라인 옵션은 `solc` 그리고 `geth`같은 도구들에서 호환되지 않습니다.
    
도커
======

우리는 컴파일러가 포함된 최신 도커 빌드를 제공합니다.
``stable`` 저장소에는 배포된 버전이 포함되어 있으며
``nightly`` 저장소에는 잠재적으로 불안정한 변경사항이 포함된 개발 브랜치의 버전이 포함되어 있습니다.

.. code:: bash

    docker run ethereum/solc:stable solc --version

현재, 도커 이미지에는 컴파일러 실행 파일만 포함되어 있습니다,
그러므로 소스와 출력 디렉터리를 연결하기 위해선 몇 가지 추가 작업을 해야 합니다.

바이너리 패키지
===============

솔리디티 바이너리 패키지는 `solidity/releases <https://github.com/ethereum/solidity/releases>`_ 에서 이용 가능합니다.

우분투에서 사용 가능한 PPA도 있습니다. 안정된 최신 버전은 아래 명령어를 통해 설치가능합니다.

.. code:: bash

    sudo add-apt-repository ppa:ethereum/ethereum
    sudo apt-get update
    sudo apt-get install solc

개발중인 최신 버전을 사용하고 싶다면 아래 명령어를 이용하세요:

.. code:: bash

    sudo add-apt-repository ppa:ethereum/ethereum
    sudo add-apt-repository ppa:ethereum/ethereum-dev
    sudo apt-get update
    sudo apt-get install solc

우리는 또한 `snap package <https://snapcraft.io/>`_ 를 배포하고 있습니다. 이 패키지는 `지원되는 모든 리눅스 배포판 <https://snapcraft.io/docs/core/install>`_에 설치할 수 있습니다. solc의 안정된 최신 버전을 설치하려면 다음 명령어를 이용하세요:

.. code:: bash

    sudo snap install solc

또는 개발 브랜치의 최신 변경사항이 포함된 불안정한 solc를 테스트하는 데 도움을 주고 싶다면:

.. code:: bash

    sudo snap install solc --edge

개발 중인 최신 버전뿐이지만 아치 리눅스 역시 패키지가 있습니다:

.. code:: bash

    pacman -S solidity

Jenkins에서 TravisCI로 마이그레이션 하는 과정에서 Homebrew의 pre-built bottles를 빠뜨렸습니다.
그렇지만 여전히 소스에서 빌드는 가능합니다. 우리는 곧 pre-built bottles를 다시 추가할 예정입니다.

.. code:: bash

    brew update
    brew upgrade
    brew tap ethereum/ethereum
    brew install solidity
    brew linkapps solidity

솔리디티의 특정 버전이 필요한 경우, 깃허브에서 직접 Homebrew formula를 설치할 수 있습니다.

`깃허브의 solidity.rb 커밋 내역 <https://github.com/ethereum/homebrew-ethereum/commits/master/solidity.rb>`_을 참조하세요.

``solidity.rb``의 특정 커밋의 raw file 링크를 찾을 때까지 히스토리 링크를 따라가세요.

``brew``를 사용하여 설치하십시오:

.. code:: bash

    brew unlink solidity
    # Install 0.4.8
    brew install https://raw.githubusercontent.com/ethereum/homebrew-ethereum/77cce03da9f289e5a3ffe579840d3c5dc0a62717/solidity.rb

젠투 리눅스 또한 ``emerge``를 이용해 설치할 수 있는 솔리디티 패키지를 제공합니다:

.. code:: bash

    emerge dev-lang/solidity

.. _building-from-source:

소스에서 빌드하기
====================

저장소 복제
--------------------

소스 코드를 복제하기 위해, 아래의 명령어를 실행하세요:

.. code:: bash

    git clone --recursive https://github.com/ethereum/solidity.git
    cd solidity

솔리디티 개발을 돕고 싶다면, 솔리디티 프로젝트를 포크하고 두 번째 원격 저장소로 개인 저장소를 추가하세요:

.. code:: bash

    cd solidity
    git remote add personal git@github.com:[username]/solidity.git

솔리디티는 서브모듈을 가지고 있습니다. 제대로 로드되었는지 확인하세요:\

.. code:: bash

   git submodule update --init --recursive

필수 설치 항목 - 맥OS
---------------------

맥OS의 경우, 반드시 최신 버전의 `Xcode<https://developer.apple.com/xcode/download/>`_가 설치되어야 합니다.
여기에는 `Clang C++ compiler <https://en.wikipedia.org/wiki/Clang>`_, `Xcode IDE <https://en.wikipedia.org/wiki/Xcode>`_ 와 그 외 OS X에서 C++ 애플리케이션을 빌드하기 위한 애플 개발도구들이 포함되어 있습니다.
Xcode를 처음 설치하거나 새 버전을 설치했다면, 커맨드라인에서 빌드하기 전 라이선스에 동의해야 합니다:

.. code:: bash

    sudo xcodebuild -license accept

외부 의존 항목을 설치하기 위해 OSX 빌드는 `Homebrew <http://brew.sh>`_ 패키지 매니저를 필요로 합니다.
혹시 처음부터 다시 시작하고 싶다면, 여기 `Homebrew 삭제 <https://github.com/Homebrew/homebrew/blob/master/share/doc/homebrew/FAQ.md#how-do-i-uninstall-homebrew>`_하는 방법입니다.

필수 설치 항목 - 윈도우
-----------------------

솔리디티의 윈도우 빌드를 위해 아래의 의존 항목들을 설치해야 합니다:

+------------------------------+-------------------------------------------------------+
| 소프트웨어                      | 설명                                                   |
+==============================+=======================================================+
| `Git for Windows`_           | Github에서 소스를 검색하기 위한 커맨드라인 도구.                |
+------------------------------+-------------------------------------------------------+
| `CMake`_                     | 크로스 플랫폼 빌드 파일 생성기.                              |
+------------------------------+-------------------------------------------------------+
| `Visual Studio 2015`_        | C++ 컴파일러 및 개발 환경.                                 |
+------------------------------+-------------------------------------------------------+

.. _Git for Windows: https://git-scm.com/download/win
.. _CMake: https://cmake.org/download/
.. _Visual Studio 2015: https://www.visualstudio.com/products/vs-2015-product-editions


외부 의존 항목
---------------------

맥OS, 윈도우, 수많은 리눅스 배포판상에서 모든 필수 외부 의존 항목을 설치하는 "원 버튼"스크립트가 있습니다.
여러 단계로 이루어진 수동 프로세스이지만 한줄의 명령어로 실행가능합니다:

.. code:: bash

    ./scripts/install_deps.sh

윈도우에선 아래와 같습니다:

.. code:: bat

    scripts\install_deps.bat


커맨드라인 빌드
------------------

** 빌드하기 전 외부 의존 항목을(윗부분 참조) 반드시 설치해야 합니다.**

솔리디티 프로젝트는 빌드를 구성하기 위해 CMake를 사용합니다.
솔리디티 빌드는 리눅스, 맥OS 및 기타 유닉스에서 매우 유사하게 진행됩니다:

.. code:: bash

    mkdir build
    cd build
    cmake .. && make

또는 조금 더 쉬운 방법:

.. code:: bash
    
    #주석: 이 명령어는 바이너리 solc와 soltest를 usr/local/bin에 설치할 것입니다.
    ./scripts/build.sh

윈도우에서는:

.. code:: bash

    mkdir build
    cd build
    cmake -G "Visual Studio 14 2015 Win64" ..

이 명령어의 결과로 해당 빌드 디렉터리에 **solidity.sln** 가 생성됩니다.
이 파일을 더블클릭하면 Visual Studio가 실행됩니다.
우리는 ** RelWithDebugInfo ** 환경설정을 빌드하는 걸 제안합니다.

또 다른 방법으로는 윈도우 커맨드라인에서 아래와같이 빌드를 진행할 수 있습니다:

.. code:: bash

    cmake --build . --config RelWithDebInfo

CMake 옵션
=============

CMake 옵션을 알고 싶다면 ``cmake .. -LH`` 명령어를 실행하십시오.

버전 문자열 상세하게 보기
============================

솔리디티 버전 문자열은 네 부분으로 구성됩니다:

- 버전 숫자
- pre-release 태그, 대개 ``develop.YYYY.MM.DD`` 나 ``nightly.YYYY.MM.DD`` 형태를 지님
- 다음과 같은 형태의 커밋 ``commit.GITHASH``
- 플랫폼 및 컴파일러에 대한 세부 정보를 포함하는 몇 가지 항목

로컬에서 수정된 부분이 있다면, 커밋 뒤에 ``.mod``가 붙습니다.

이 부분들은 Semver(Semantic Versioning)에 따라 필요에 의해 결합됩니다. 여기서 솔리디티 pre-release 태그는 Semver의 pre-release 태그와 같고
솔리디티 커밋 및 플랫폼은 결합되어 Semver 빌드 메타데이터를 구성합니다.

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

이 동작은 :ref:`version pragma <version_pragma>`와 함께 작동합니다.