# Akash Node Repo Table of Contents

| Directory | Brief Description                                          | Prominent Sub Directories                                                                                                             | Available Docs                                                      |
| --------- | ---------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------- |
| app       | Cosmos SDK module and keystore initiation                  |                                                                                                                                       |                                                                     |
| cmd       | Cobra registration of `akash` root/sub-commands and flags. |                                                                                                                                       |                                                                     |
| proto     | Akash API definitions via protobuf                         |                                                                                                                                       |                                                                     |
| types     | Types derived from protobuf's Go client (protoc)           |                                                                                                                                       |                                                                     |
| x         | Cosmos SDK Modules                                         | <ul><li>node/x/&#x3C;nodule_name>/client</li><li>node/x/&#x3C;nodule_name>/handler</li><li>node/x/&#x3C;nodule_name>/keeper</li></ul> | [Akash Modules](akash-node-repo-table-of-contents.md#akash-modules) |

## Akash Modules

* Akash API - Deployment Creation
* Akash API - Lease Creation
