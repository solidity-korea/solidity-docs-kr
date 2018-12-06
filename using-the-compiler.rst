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
를 사용하여 분할 파일들을 모두 출력하도록 명령하는 것이 나을것 입니다.

명령줄 컴파일러는 자동적으로 파일 시스템으로 부터 수입된(imported) 파일들을 읽습니다. 그러나
아래의 방법으로 ``prefix=path``를 사용해 리다이렉트할 경로를 제공하는 것 또한 가능합니다.



::

    solc github.com/ethereum/dapp-bin/=/usr/local/lib/dapp-bin/ =/usr/local/lib/fallback file.sol

이것은 본질적으로 ``/usr/local/lib/dapp-bin``아래에 있는 ``github.com/ethereum/dapp-bin/``로 시작하는 것을 검색하라고 컴파일러에게
지시합니다. 그리고 만약 거기에 있는 파일을 찾지 못한다면, 컴파일러는 ``/usr/local/lib/fallback``를 살펴 볼것입니다. (공백의 접두사는 항상 일치한다.)
``solc``은 외부에 재 맵핑 대상 외부와 명시적으로 소스파일 위치가 명시된 디렉터리 외부에 있는 파일 시스템으로 부터 
파일 시스템로 부터 파일들을 읽지 않습니다. 그러므로 ``import "/etc/passwd";``와 같은 것들은 오직 재맵핑(remapping)으로서 ``=/`` 추가하는 경우에만
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

These JSON formats are used by the compiler API as well as are available through ``solc``. These are subject to change,
some fields are optional (as noted), but it is aimed at to only make backwards compatible changes.

The compiler API expects a JSON formatted input and outputs the compilation result in a JSON formatted output.

Comments are of course not permitted and used here only for explanatory purposes.

Input Description
-----------------

.. code-block:: none

    {
      // Required: Source code language, such as "Solidity", "serpent", "lll", "assembly", etc.
      language: "Solidity",
      // Required
      sources:
      {
        // The keys here are the "global" names of the source files,
        // imports can use other files via remappings (see below).
        "myFile.sol":
        {
          // Optional: keccak256 hash of the source file
          // It is used to verify the retrieved content if imported via URLs.
          "keccak256": "0x123...",
          // Required (unless "content" is used, see below): URL(s) to the source file.
          // URL(s) should be imported in this order and the result checked against the
          // keccak256 hash (if available). If the hash doesn't match or none of the
          // URL(s) result in success, an error should be raised.
          "urls":
          [
            "bzzr://56ab...",
            "ipfs://Qma...",
            "file:///tmp/path/to/file.sol"
          ]
        },
        "mortal":
        {
          // Optional: keccak256 hash of the source file
          "keccak256": "0x234...",
          // Required (unless "urls" is used): literal contents of the source file
          "content": "contract mortal is owned { function kill() { if (msg.sender == owner) selfdestruct(owner); } }"
        }
      },
      // Optional
      settings:
      {
        // Optional: Sorted list of remappings
        remappings: [ ":g/dir" ],
        // Optional: Optimizer settings (enabled defaults to false)
        optimizer: {
          enabled: true,
          runs: 500
        },
        evmVersion: "byzantium", // Version of the EVM to compile for. Affects type checking and code generation. Can be homestead, tangerineWhistle, spuriousDragon, byzantium or constantinople
        // Metadata settings (optional)
        metadata: {
          // Use only literal content and not URLs (false by default)
          useLiteralContent: true
        },
        // Addresses of the libraries. If not all libraries are given here, it can result in unlinked objects whose output data is different.
        libraries: {
          // The top level key is the the name of the source file where the library is used.
          // If remappings are used, this source file should match the global path after remappings were applied.
          // If this key is an empty string, that refers to a global level.
          "myFile.sol": {
            "MyLib": "0x123123..."
          }
        }
        // The following can be used to select desired outputs.
        // If this field is omitted, then the compiler loads and does type checking, but will not generate any outputs apart from errors.
        // The first level key is the file name and the second is the contract name, where empty contract name refers to the file itself,
        // while the star refers to all of the contracts.
        //
        // The available output types are as follows:
        //   abi - ABI
        //   ast - AST of all source files
        //   legacyAST - legacy AST of all source files
        //   devdoc - Developer documentation (natspec)
        //   userdoc - User documentation (natspec)
        //   metadata - Metadata
        //   ir - New assembly format before desugaring
        //   evm.assembly - New assembly format after desugaring
        //   evm.legacyAssembly - Old-style assembly format in JSON
        //   evm.bytecode.object - Bytecode object
        //   evm.bytecode.opcodes - Opcodes list
        //   evm.bytecode.sourceMap - Source mapping (useful for debugging)
        //   evm.bytecode.linkReferences - Link references (if unlinked object)
        //   evm.deployedBytecode* - Deployed bytecode (has the same options as evm.bytecode)
        //   evm.methodIdentifiers - The list of function hashes
        //   evm.gasEstimates - Function gas estimates
        //   ewasm.wast - eWASM S-expressions format (not supported atm)
        //   ewasm.wasm - eWASM binary format (not supported atm)
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


Output Description
------------------

.. code-block:: none

    {
      // Optional: not present if no errors/warnings were encountered
      errors: [
        {
          // Optional: Location within the source file.
          sourceLocation: {
            file: "sourceFile.sol",
            start: 0,
            end: 100
          ],
          // Mandatory: Error type, such as "TypeError", "InternalCompilerError", "Exception", etc.
          // See below for complete list of types.
          type: "TypeError",
          // Mandatory: Component where the error originated, such as "general", "ewasm", etc.
          component: "general",
          // Mandatory ("error" or "warning")
          severity: "error",
          // Mandatory
          message: "Invalid keyword"
          // Optional: the message formatted with source location
          formattedMessage: "sourceFile.sol:100: Invalid keyword"
        }
      ],
      // This contains the file-level outputs. In can be limited/filtered by the outputSelection settings.
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
      // This contains the contract-level outputs. It can be limited/filtered by the outputSelection settings.
      contracts: {
        "sourceFile.sol": {
          // If the language used has no contract names, this field should equal to an empty string.
          "ContractName": {
            // The Ethereum Contract ABI. If empty, it is represented as an empty array.
            // See https://github.com/ethereum/wiki/wiki/Ethereum-Contract-ABI
            abi: [],
            // See the Metadata Output documentation (serialised JSON string)
            metadata: "{...}",
            // User documentation (natspec)
            userdoc: {},
            // Developer documentation (natspec)
            devdoc: {},
            // Intermediate representation (string)
            ir: "",
            // EVM-related outputs
            evm: {
              // Assembly (string)
              assembly: "",
              // Old-style assembly (object)
              legacyAssembly: {},
              // Bytecode and related details.
              bytecode: {
                // The bytecode as a hex string.
                object: "00fe",
                // Opcodes list (string)
                opcodes: "",
                // The source mapping as a string. See the source mapping definition.
                sourceMap: "",
                // If given, this is an unlinked object.
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
              // The same layout as above.
              deployedBytecode: { },
              // The list of function hashes
              methodIdentifiers: {
                "delegate(address)": "5c19a95c"
              },
              // Function gas estimates
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
            // eWASM related outputs
            ewasm: {
              // S-expressions format
              wast: "",
              // Binary format (hex string)
              wasm: ""
            }
          }
        }
      }
    }


Error types
~~~~~~~~~~~

1. ``JSONError``: JSON input doesn't conform to the required format, e.g. input is not a JSON object, the language is not supported, etc.
2. ``IOError``: IO and import processing errors, such as unresolvable URL or hash mismatch in supplied sources.
3. ``ParserError``: Source code doesn't conform to the language rules.
4. ``DocstringParsingError``: The NatSpec tags in the comment block cannot be parsed.
5. ``SyntaxError``: Syntactical error, such as ``continue`` is used outside of a ``for`` loop.
6. ``DeclarationError``: Invalid, unresolvable or clashing identifier names. e.g. ``Identifier not found``
7. ``TypeError``: Error within the type system, such as invalid type conversions, invalid assignments, etc.
8. ``UnimplementedFeatureError``: Feature is not supported by the compiler, but is expected to be supported in future versions.
9. ``InternalCompilerError``: Internal bug triggered in the compiler - this should be reported as an issue.
10. ``Exception``: Unknown failure during compilation - this should be reported as an issue.
11. ``CompilerError``: Invalid use of the compiler stack - this should be reported as an issue.
12. ``FatalError``: Fatal error not processed correctly - this should be reported as an issue.
13. ``Warning``: A warning, which didn't stop the compilation, but should be addressed if possible.
