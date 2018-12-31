###
LLL
###

.. _lll:

LLL은 s-expressions 문법을 사용하는 EVM의 저수준 언어입니다.

Solidity 저장소는 LLL 컴파일러가 포함되어 있으며 어셈블러 하위시스템을 Solidity와 공유합니다.
하지만, 컴파일을 유지하는 것과 별개로 다른 개선점은 없습니다.

특별히 요청하지 않는 한 빌드되지 않습니다.

.. code-block:: bash

    $ cmake -DLLL=ON ..
    $ cmake --build .

.. warning::

    LLL 코드베이스는 제공되지 않을것이며, 향후 Solidity 저장소에서 삭제될 예정입니다.
