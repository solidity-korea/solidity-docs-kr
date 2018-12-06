******************
컴파일러 사용하기
******************

.. index:: ! commandline compiler, compiler;commandline, ! solc, ! linker

.. _commandline-compiler:

명령줄 컴파일러 사용하기
******************************

.. note::
    이 섹션은  :ref:`solcjs <solcjs>`에 적용 되지 않습니다.

Solidity 저장소의 빌드 대상 중 하나는 Solidity 명령줄 컴파일러인 ``solc``입니다,
``solc --help``를 사용하면 모든 옵션에 대한 설명을 제공합니다. 컴파일러는 간단한 구문 트리 (구문 분석 트리)를 통한 간단한 바이너리 및 어셈블리부터 가스 사용량 평가에 이르기까지 다양한 출력을 생성 할 수 있습니다.
만약 단일 파일만 컴파일 하기를 원한다면, ``solc --bin sourceFile.sol``를 실행하세요. 바이너리가 출력 될 것입니다. 당신의 컨트렉트를 
배치(deploy)하기전에 ``solc --optimize --bin sourceFile.sol``를 이용하여 컴파일하는 동안 최적화기(Optimizer)를 활성화 시키세요.
만약 조금더 진보된 다른 형태의 ``solc``의 결과를 얻기를 원한다면, 아마도 ``solc -o outputDirectory --bin --ast --asm sourceFile.sol``
를 사용하여 분할 파일들을 모두 출력하도록 명령하는 것이 좋을 것입니다.

명령줄 컴파일러는 자동적으로 파일 시스템으로 부터 수입된(imported) 파일들을 읽습니다. 그러나
아래의 방법으로 ``prefix=path``를 사용해 리다이렉트할 경로를 제공하는 것 또한 가능합니다.



::

    solc github.com/ethereum/dapp-bin/=/usr/local/lib/dapp-bin/ =/usr/local/lib/fallback file.sol

이것은 본질적으로 ``/usr/local/lib/dapp-bin``아래에 있는 ``github.com/ethereum/dapp-bin/``로 시작하는 것을 검색하라고 컴파일러에게
지시합니다. 그리고 만약 거기에 있는 파일을 찾지 못한다면, 컴파일러는 ``/usr/local/lib/fallback``를 살펴 볼것입니다. (공백의 접두사는 항상 일치한다.)
``solc``은 외부에 재 맵핑 대상 외부와 명시적으로 소스파일 위치가 명시된 디렉터리 외부에 있는 파일 시스템으로 부터 
파일 시스템로 부터 파일들을 읽지 않습니다. 그러므로 ``import "/etc/passwd";``와 같은 것들은 오직 재맵 핑(remapping)으로서 ``=/`` 추가하는 경우에만
작동합니다.

만약 재맵핑으로 인해 여러 일치 항목이 있는 경우 가장 긴 공통 접두사가 있는 항목이 선택됩니다.

보안상의 이유들로 컴파일러는 액세스 할 수 있는 디렉터리를 제한합니다. 명령 줄에 명시된 소스파일 경로(및 해당 하위 디렉터리)와 재맵핑에 의해 정의 된 경로들은 
import 문에서 허용됩니다, 그러나 다른 모든 것들은 거절됩니다. 추가적인 경로(및 그것의 하위 디렉터리)는 ``--allow-paths /sample/path,/another/sample/path`` 스위치를 통해 허용 될 수 있습니다.

만약 컨트렉트가 :ref:`libraries <libraries>`를 사용한다면, 바이트코드가 ``__LibraryName______``의 하위 문자열(substrings)를 포함한다는 것을 알아야합니다.
그 지점에 당신의 라이브러리 주소가 삽입될것을 의미하는 링커로써 ``solc``을 사용 할 수 있습니다.

각각의 라이브러리 주소를 제공하기 위해서 커맨드에 ``--libraries "Math:0x12345678901234567890 Heap:0xabcdef0123456"``를 추가하던지 파일(한줄에 하나의 라이브러리)에 문자열을 저장하고 ``--libraries fileName``를 사용하여 ``solc``를 실행하세요

만약 ``solc``가 ``--link``옵션과 함께 호출 된다면, 모든 입력 파일 들은 주어진 ``__LibraryName____`` 형식의 연결(link)되지 않은 (16진수로 인코드 된)바이더리들로 해석되어 현재 위치에 연결 됩니다(만약 입력이 stdin으로 부터 읽어온것이라면, 그것은 stdout으로 쓰여집니다). ``--libraries``를 제외한 모든 옵션 
은 이 경우에 무시됩니다(``-o`` 포함)

만약 ``solc``가 옵션 ``--standard-json``과 함께 호출 되었다면, 그것은 표준 입력에서 JSON 입력(아래에 설명)을 기대합니다 그리고 표준출력의 JSON출력을 반환합니다..

.. _compiler-api:

컴파일러 입력과 출력 JSON 설명
******************************************

이러한 JSON 형식은 ``solc``를 통해 가능한것과 같이 컴파일러 API에 의해 사용됩니다. 이러한 것들은 변화될 수 있고, 몇몇 필드는 선택적이지만(앞에서 말한대로), 이전 버전과의 호환성을 위해서만 사용됩니다.

컴파일러 API는 JSON 형식으로 입력을 기대하고 JSON 형식으로 컴파일의 결과를 출력합니다.

주석은 당연히 설명목적으로만 허용되고 여기에 사용되지 않습니다.

입력 설명
-----------------

.. code-block:: none

    {
      // Required: Source code language, such as "Solidity", "serpent", "lll", "assembly", etc.
      language: "Solidity",
      // Required
      sources:
      {
		// 여기에 이 키는 소스코드 파일들의 "전역(global)" 이름들입니다.
        // 임포트는 재맵핑을 통해 다른 파일을 사용할 수 있습니다.
        "myFile.sol":
        {
          // 선택적 : 소스 파일의 keccak256 해시
          // 이것은 URL을 통해 임포트된 경우 내용을 검증하기 위해 사용됩니다.
          "keccak256": "0x123...",
          // Required (만약 "content"가 사용되지 않았다면 아래를 보세요): 소스파일의 URL
          // URL(s) should be imported in this order and the result checked against the
          // keccak256 hash (if available). If the hash doesn't match or none of the
          // URL은 여기 안에 임포트 되어야만 합니다. 그리고 결과는 keccak256 해시에 대해
          // 확인해야 합니다. (가능한 경우에). 만약 해시가 맞지 않거나 성공한 URL이 없다면
          // 에러가 발생해야 합니다.
          "urls":
          [
            "bzzr://56ab...",
            "ipfs://Qma...",
            "file:///tmp/path/to/file.sol"
          ]
        },
        "mortal":
        {
          // Optional: 소스파일의 keccak256 해시
          "keccak256": "0x234...",
          // Required (만약 "urls"가 사용되지 않으면): 소스 파일의 리터럴 내용
          "content": "contract mortal is owned { function kill() { if (msg.sender == owner) selfdestruct(owner); } }"
        }
      },
      // Optional
      settings:
      {
        // Optional: 재맵핑의 정렬된 리스트
        remappings: [ ":g/dir" ],
        // Optional: 최적화기 (enabled defaults to false)
        optimizer: {
          enabled: true,
          runs: 500
        },
        evmVersion: "byzantium", // Version of the EVM to compile for. Affects type checking and code generation. Can be homestead, tangerineWhistle, spuriousDragon, byzantium or constantinople
        // Metadata settings (optional)
        metadata: {
          // URL이 아닌 리터럴 내용만 사용하세요. (기본값 : false)
          useLiteralContent: true
        },
        // 라이브러리들의 주소. 만약 모든 라이브러리가 여기에 주어지지 않는다면, 그것은 출력 데이터가 다른 연결되지 않은 객체를 초례할 수 있습니다.
        libraries: {
          // 최상위 레벨 키는 라이브러리가 사용된 소스파일의 이름입니다.
          // 만약 재맵핑이 사용되었다면, 재 맵핑이 적용된 후에, 이 소스 파일은 전역 경로가 일치해야 합니다.
          // 만약 이 키가 빈 문자열이라면, 그것은 전역 수준을 참조합니다.
          "myFile.sol": {
            "MyLib": "0x123123..."
          }
        }
        // The following can be used to select desired outputs.
        // 아래의 코드는 원하는 출력을 선택하는데 사용할 수 있습니다.
        // 만약 이 필드가 누락 된다면, 컴파일러는 불러오고 타입을 체크 합니다. 그러나 에러러부터 어떠한 에러도 생성하지 않습니다.
        
        // 첫번째 레벨의 키는 파일 이름이고 두번재는 컨트렉트 이름입니다. 여기서 빈 계약이름은 파일 자체를 나타냅니다,
        // star가 컨트렉트의 모든 내용을 참조하는 동안.
        //
        // 아래는 가능한 출력 타입입니다.
        //   abi - ABI
        //   ast - 모든 소스파일의 AST
        //   legacyAST - 모든 소스파일의 legacy AST
        //   devdoc - 개발자 문서 (natspec)
        //   userdoc - 사용자 문서 (natspec)
        //   metadata - 메타데이터
        //   ir - desugaring이전의 새로운 어셈블리 형식
        //   evm.assembly - desugaring이후의 새로운 어셈블리 형식
        //   evm.legacyAssembly - 이전 스타일의 JSON형식 어셈블리
        //   evm.bytecode.object - 바이트 코드 객체
        //   evm.bytecode.opcodes - Opcodes 리스트
        //   evm.bytecode.sourceMap - 소스 맵핑 (디버그에 유용함)
        //   evm.bytecode.linkReferences - 링크 참조 (if unlinked object)
        //   evm.deployedBytecode* - 배포된 바이트코드 (evm.bytecode과 동일한 옵션을 가짐)
        //   evm.methodIdentifiers - 해시함수 리스트
        //   evm.gasEstimates - 가스 측정함수
        //   ewasm.wast - eWASM S-expressions format (not supported atm)
        //   ewasm.wasm - eWASM 바이터리 데이터 (not supported atm)
        //
        // Note that using a using `evm`, `evm.bytecode`, `ewasm`, etc. will select every
        // target part of that output. Additionally, `*` can be used as a wildcard to request everything.
        //
        outputSelection: {
          // Enable the metadata and bytecode outputs of every single contract.
          "*": {
            "*": [ "metadata", "evm.bytecode" ]
          },
          // Enable the abi and opcodes output of MyContract defined in file def.
          "def": {
            "MyContract": [ "abi", "evm.bytecode.opcodes" ]
          },
          // Enable the source map output of every single contract.
          "*": {
            "*": [ "evm.bytecode.sourceMap" ]
          },
          // Enable the legacy AST output of every single file.
          "*": {
            "": [ "legacyAST" ]
          }
        }
      }
    }


출력 설명
------------------

.. code-block:: none

    {
      // 선택적 : 에러나 경고가 발생 했는지 나타내지 않습니다.
      errors: [
        {
          // Optional: 소스 파일안 위치.
          sourceLocation: {
            file: "sourceFile.sol",
            start: 0,
            end: 100
          ],
          // 의무적 : "TypeError", "InternalCompilerError", "Exception"등 과 같은 에러 타입
          // 아래 타입 리스트를 보세요.
          type: "TypeError",
          // 의무적 : "general", "ewasm"등과 같은 에러가 발생한 컴포넌트
          component: "general",
          // 의무적 ("error" or "warning")
          severity: "error",
          // 의무적
          message: "Invalid keyword"
          // 선택적 : 소스 위치를 포함한 형식을 갖춘 메세지
          formattedMessage: "sourceFile.sol:100: Invalid keyword"
        }
      ],
      // 이것은 파일 수준 출력을 포함합니다. 이것은 outputSelection 설정에 의해 제한되고 걸러질 수 있습니다.
      sources: {
        "sourceFile.sol": {
          // Identifier (used in source maps)
          id: 1,
          // The AST object
          ast: {},
          // The legacy AST object
          legacyAST: {}
        }
      },
      // 이것은 컨트렉트 수준 출력을 포함합니다. 이것은 outputSelection설정에 의해 제한되고 걸러질 수 있습니다.
      contracts: {
        "sourceFile.sol": {
          // 만갹 사용된 언어가 컨트렉트 이름을 가지고 있지 않다면, 이 필드는 빈 문자열과 같아야 합니다.
          "ContractName": {
            // 이더리움 컨트렉트 ABI. 만약 비어있다면, 이것은 빈 배열을 나타냅니다.
            // https://github.com/ethereum/wiki/wiki/Ethereum-Contract-ABI 를 확인해 보세요.
            abi: [],
            // 메타데이터 출력 문서를 확인해보세요.
            metadata: "{...}",
            // 사용자 문서 (natspec)
            userdoc: {},
            // 개발자 문서 (natspec)
            devdoc: {},
            // 중간 표현 (string)
            ir: "",
            // EVM-related outputs
            evm: {
              // Assembly (string)
              assembly: "",
              // 이전 스타일의 어셈블리 (object)입니다.
              legacyAssembly: {},
              // 바이트 코드와 자세한 내용.
              bytecode: {
                // 16진수 인 바이트 코드입니다.
                object: "00fe",
                // OPcodes 리스트 (string)입니다.
                opcodes: "",
                // 문자열로써 소스 맵핑입니다. 소스 맵핑 정의를 확인해 보세요.
                sourceMap: "",
                // 주어졌다면, 이것은 연결되지 않은 객체입니다.
                linkReferences: {
                  "libraryFile.sol": {
                    // Byte offsets into the bytecode. Linking replaces the 20 bytes located there.
                    "Library1": [
                      { start: 0, length: 20 },
                      { start: 200, length: 20 }
                    ]
                  }
                }
              },
              // 위와 같은 레이아웃 입니다.
              deployedBytecode: { },
              // 해시 함수 리스트 입니다.
              methodIdentifiers: {
                "delegate(address)": "5c19a95c"
              },
              // Function gas estimates
              // 가스 예측 함수 입니다. 
              gasEstimates: {
                creation: {
                  codeDepositCost: "420000",
                  executionCost: "infinite",
                  totalCost: "infinite"
                },
                external: {
                  "delegate(address)": "25000"
                },
                internal: {
                  "heavyLifting()": "infinite"
                }
              }
            },
            // 출력과 연관된 eWASM입니다.
            ewasm: {
              // S-expressions 형식입니다.
              wast: "",
              // Binary format (hex string)
              // 바이너리 형식 (hex string)
              wasm: ""
            }
          }
        }
      }
    }


에러 타입
~~~~~~~~~~~

1. "JSONError" :  JSON 입력은 요구된 형식에 일치하지 않습니다. 예시) 입력이 json 오브젝트가 아닙니다. 그 언어는 지원되지 않습니다. 등.
2. "IOError" : IO와 임포트 과정에서의 에러들입니다, 분석될 수 없는 URL이나 공급된 소스에서의 해시 불일치와 같은것들이 있습니다.
3. "ParserError" : 소스코드는 언어 원칙에 일치하지 않습니다.
4. "DocstringParsingError" : 커맨드 블록에서 NatSpec 태그는 분석될 수 없습니다.
5. "SyntaxError" : Syntactical error는 "continue"가 "for" 반복 외부에서 사용 되는것등이 있습니다.
6. "DeclarationError" : 유효하지 않거나 혹은 의결 할수 없는(unresolvable),  식별자 이름충돌입니다. 예시 "identifier not found" 식별자가 발견되지 않음
7. "TypeError" : 유효하지 않은 타입 변경, 유효하지 않은 할당(assignment) 등과 같은 type system내의 에러입니다.
8. "UnimplementedFeatureError" : 기능이 컴파일러에 의해 지원되지 않습니다. 하지만 미래 번전에서는 지원될 것으로 예상됩니다.
9 "internalCompilerError" : 컴파일러에 의해 촉발되는 내부의 버그 - 이것은 문제로서 보고되어져야 합니다.
10 "Exception" : 컴파일러 도중에 알려지지 않은 실패 - 이것은 문제로서 보고되어야 합니다.
11. "CompilerError" : 유효하지 않은 컴파일러 스택의 사용 - 이것은 문제로서 되어야 합니다.
12. "FatalError" : 치명적 오류가 바르게 처리되지 않음 - 이는 문제로서 기록되어야 합니다.
13. "Warning" : 단순 경고, 컴파일러를 중단 하지는 않지만, 가능하다면 다뤄져야 합니다.