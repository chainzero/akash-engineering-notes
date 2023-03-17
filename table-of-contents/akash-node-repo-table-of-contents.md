# Akash Node Repo Table of Contents

| Directory | Brief Description                                          | Prominent SubDirectories/Files                                                                                                        | Available Docs                                                      |
| --------- | ---------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------- |
| app       | Cosmos SDK module registration and store definitions       | app.go                                                                                                                                | [Akash App](akash-node-repo-table-of-contents.md#akash-app)         |
| cmd       | Cobra registration of `akash` root/sub-commands and flags. | <p>main.go<br>root.go<br></p>                                                                                                         |                                                                     |
| proto     | Akash API definitions via protobuf                         | <p>node/proto/akash/deployment<br>node/proto/akash/market<br>node/proto/akash/provider</p>                                            | [Akash API](akash-node-repo-table-of-contents.md#akash-api)         |
| types     | Types derived from protobuf's Go client (protoc)           | node/types/v1beta2/                                                                                                                   |                                                                     |
| x         | Cosmos SDK Modules                                         | <ul><li>node/x/&#x3C;nodule_name>/client</li><li>node/x/&#x3C;nodule_name>/handler</li><li>node/x/&#x3C;nodule_name>/keeper</li></ul> | [Akash Modules](akash-node-repo-table-of-contents.md#akash-modules) |

##

## Akash App

## Akash API



## Akash Modules

* Akash API - Deployment Creation
* Akash API - Lease Creation
