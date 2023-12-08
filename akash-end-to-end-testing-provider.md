# Akash End to End Testing (Provider)

## Overview

The Akash Provider repository defines current end to end testing coverage in the i[ntegration package](https://github.com/akash-network/provider/tree/main/integration).

The E2E testing strategy and execution uses the [Testify package suite](https://github.com/stretchr/testify) with the current suite of tests listed in the `e2e_test.go` file such as:

```
func TestIntegrationTestSuite(t *testing.T) {
	integrationTestOnly(t)

	suite.Run(t, new(E2EContainerToContainer))
	suite.Run(t, new(E2EAppNodePort))
	suite.Run(t, new(E2EDeploymentUpdate))
	suite.Run(t, new(E2EApp))
	suite.Run(t, new(E2EPersistentStorageDefault))
	suite.Run(t, new(E2EPersistentStorageBeta2))
	suite.Run(t, new(E2EPersistentStorageDeploymentUpdate))
	suite.Run(t, new(E2EMigrateHostname))
	suite.Run(t, new(E2EJWTServer))
	suite.Run(t, new(E2ECustomCurrency))
	suite.Run(t, &E2EIPAddress{IntegrationTestSuite{ipMarketplace: true}})
}
```

\
\
