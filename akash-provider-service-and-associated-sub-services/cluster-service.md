# Cluster Service

## 1).  Cluster Service Initiation

### Provider Service Calls/Initiates the Cluster Service

> [Source code reference location](https://github.com/akash-network/provider/blob/e7aa0b5b81957a130f1dc584f335c6f9e41db6b1/service.go#L101)

```
	cluster, err := cluster.NewService(ctx, session, bus, cclient, ipOperatorClient, waiter, clusterConfig)
	if err != nil {
		cancel()
		<-bc.lc.Done()
		return nil, err
	}
```

The parameters for `cluster.NewService` include the Provider's Kubernetes cluster settings.

```
	clusterConfig := cluster.NewDefaultConfig()
	clusterConfig.InventoryResourcePollPeriod = cfg.InventoryResourcePollPeriod
	clusterConfig.InventoryResourceDebugFrequency = cfg.InventoryResourceDebugFrequency
	clusterConfig.InventoryExternalPortQuantity = cfg.ClusterExternalPortQuantity
	clusterConfig.CPUCommitLevel = cfg.CPUCommitLevel
	clusterConfig.MemoryCommitLevel = cfg.MemoryCommitLevel
	clusterConfig.StorageCommitLevel = cfg.StorageCommitLevel
	clusterConfig.BlockedHostnames = cfg.BlockedHostnames
	clusterConfig.DeploymentIngressStaticHosts = cfg.DeploymentIngressStaticHosts
	clusterConfig.DeploymentIngressDomain = cfg.DeploymentIngressDomain
	clusterConfig.ClusterSettings = cfg.ClusterSettings
```

These settings are defined in the flags used when the `provider-services run` command is issued.

Example flag made available within the `provider/cmd/provider-services/cmd/run.go` file for Ingress Domain declaration.

```
const (
	...
	FlagDeploymentIngressDomain          = "deployment-ingress-domain"
	....
)
```

## 2). Cluster NewService Function

> [Source code reference location](https://github.com/akash-network/provider/blob/e7aa0b5b81957a130f1dc584f335c6f9e41db6b1/cluster/service.go)

The `NewService` function within `provider/cluster/service.go` invokes:

* Subscription to RPC Node pubsub bus via the `bus.Subscribe` method call
* The call of the `findDeployments` function to discover current deployments in the Kubernetes cluster.  This function is defined in the same file - `service.go` - as the cluster NewService function exists in.
* The call of the `newInventoryService` function which will track new/existing orders and create an inventory reservation when the provider bids on a deployment.

```
func NewService(ctx context.Context, session session.Session, bus pubsub.Bus, client Client, ipOperatorClient operatorclients.IPOperatorClient, waiter waiter.OperatorWaiter, cfg Config) (Service, error) {
	...

	sub, err := bus.Subscribe()
	if err != nil {
		return nil, err
	}

	deployments, err := findDeployments(ctx, log, client, session)
	if err != nil {
		sub.Close()
		return nil, err
	}

	inventory, err := newInventoryService(cfg, log, lc.ShuttingDown(), sub, client, ipOperatorClient, waiter, deployments)
	if err != nil {
		sub.Close()
		return nil, err
	}
```

## 3). Additional Inventory Service Notes

As described in the previous section the invoke of the NewService function spawns a call of the `newInventoryService` function.

The `newInventoryService` function is defined in `provider/cluster/inventory.go`.

When the Provider's bid engine determines that it should bid on a new deployment the `Reserve` method is called.  Downstream logic places this reservation into inventory.

In summation this Bid Engine logic is the mechanism in which the Provider reserves Kubernetes resources and places the reservation into inventory while the bid is pending.

> [Source code reference location](https://github.com/akash-network/provider/blob/95458f90c22c3be343efa7402ba4ac72100e251c/bidengine/order.go)

```
		case result := <-shouldBidCh:
			....
			clusterch = runner.Do(metricsutils.ObserveRunner(func() runner.Result {
				v := runner.NewResult(o.cluster.Reserve(o.orderID, group))
				return v
			}, reservationDuration))
```

