# Hostname Controller Deep Dive

### 1). Hostname Operator Listening Loop

The Akash Provider Hostname Operator command -  `hostname-operator`  invokes initial controller variable and logging settings.

> [Source code reference location](https://github.com/akash-network/provider/blob/e7aa0b5b81957a130f1dc584f335c6f9e41db6b1/operator/hostnameoperator/hostname\_operator.go)

This logic begins with the call of the `doHostnameOperator` function from Hostname Operator command.  Eventually in this function the `run` method is called with an operator struct passed in.  The `run` function - covered in detail shortly - will begin a listening loop for new ingress controller entries.

```
func doHostnameOperator(cmd *cobra.Command) error {
    ....
    	group.Go(func() error {
		return op.run(ctx)
	})
    ....
}
```

The `operator` struct of which `op` of type `hostnameOperator` is passed into the `run` method as mentioned.

```
type hostnameOperator struct {
	hostnames map[string]managedHostname

	leasesIgnored operatorcommon.IgnoreList

	client cluster.Client

	log log.Logger

	cfg    operatorcommon.OperatorConfig
	server operatorcommon.OperatorHTTP

	flagHostnamesData  operatorcommon.PrepareFlagFn
	flagIgnoreListData operatorcommon.PrepareFlagFn
}
```

### 2). Hostname Operator Listening Loop

&#x20;The `run` method invokes loop listening for new ingress controller entries.

A perpetual for loop is created and the upstream `monitorUntilError` method is called.  The `monitorUntilError` - as covered in detail next - will listen on an event bus for new Kubernetes Ingress Controller entries.

```
func (op *hostnameOperator) run(parentCtx context.Context) error {
	op.log.Debug("hostname operator start")

	for {
		...
		err := op.monitorUntilError(parentCtx)
		...
	}
}
```

### 3). Collect and Store Current Ingress Controller Entries

The `monitorUntilError` method calls the `GetHostnameDeploymentConnections` - located in `provider/cluster/kube/client_ingress.go` - which makes a call to the Kubernetes API server for a list of Ingress Controller entries.

The Ingress Controller entries are stored in the `connections` variable which is ranged/looped through and added to the `hostnameOperator` struct map of hostnames.  Future deployments will have their hostname added to the complete, current map when new `providerhost` custom resources are created.

```
func (op *hostnameOperator) monitorUntilError(parentCtx context.Context) error {
	....
	op.log.Info("starting observation")

	connections, err := op.client.GetHostnameDeploymentConnections(ctx)
	....

	for _, conn := range connections {
		leaseID := conn.GetLeaseID()
		hostname := conn.GetHostname()
		entry := managedHostname{
			lastEvent:           nil,
			presentLease:        leaseID,
			presentServiceName:  conn.GetServiceName(),
			presentExternalPort: uint32(conn.GetExternalPort()),
		}

		op.hostnames[hostname] = entry
		op.log.Debug("identified existing hostname connection",
			"hostname", hostname,
			"lease", entry.presentLease,
			"service", entry.presentServiceName,
			"port", entry.presentExternalPort)
	}
}
```

### 4). Monitor Kubernetes for New Provider Host Custom Resources

The  `ObserveHostnameState` function - located within provider/cluster/kube/client\_hostname\_connections.go - monitors for new `providerhost` custom resource adds, updates, or deletes.

The `ObserveHostnameState` method returns new events on a channel which is then taken off the channel within a select block.&#x20;

Finally the event - stored in the `ev` variable once it is pulled off the channel - is passed into the `applyEvent` method.

```
	....
	
	events, err := op.client.ObserveHostnameState(ctx)
	if err != nil {
		cancel()
		return err
	}

loop:
	for {
		select {
		....

		case ev, ok := <-events:
			if !ok {
				exitError = operatorcommon.ErrObservationStopped
				break loop
			}
			err = op.applyEvent(ctx, ev)
			if err != nil {
				op.log.Error("failed applying event", "err", err)
				exitError = err
				break loop
			}
		case <-pruneTicker.C:
			op.prune()
		case <-prepareTicker.C:
			if err := op.server.PrepareAll(); err != nil {
				op.log.Error("preparing web data failed", "err", err)
			}

		}
	}

	cancel()
	op.log.Debug("hostname operator done")
	return exitError
```

### Apply the Event/Hostname Addition

The `applyEvent` method - located in the same file `hostname_operator.go` file as the `run` function - matches the event type (I.e. `ProviderResourceAdd`).  The event type was set prior via the `ObserveHostnameState` method.\
\
Following the path of a new `providerhost` resource add as an example - the matched event is then passed to the `applyAddOrUpdateEvent` method.

```
func (op *hostnameOperator) applyEvent(ctx context.Context, ev ctypes.HostnameResourceEvent) error {
	op.log.Debug("apply event", "event-type", ev.GetEventType(), "hostname", ev.GetHostname())
	switch ev.GetEventType() {
	...
	case ctypes.ProviderResourceAdd, ctypes.ProviderResourceUpdate:
		if op.isEventIgnored(ev) {
			op.log.Info("ignoring event for", "lease", ev.GetLeaseID().String())
			return nil
		}
		err := op.applyAddOrUpdateEvent(ctx, ev)
	...

}
```
