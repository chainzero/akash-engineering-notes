# Manifest Service

## Overview

## Visualization

## Code Review

### 1). Provider Service Calls/Initiates the Manifest Service

> [Source code reference location](https://github.com/akash-network/provider/blob/e7aa0b5b81957a130f1dc584f335c6f9e41db6b1/service.go#L101)

```
	manifest, err := manifest.NewService(ctx, session, bus, cluster.HostnameService(), manifestConfig)
	if err != nil {
		session.Log().Error("creating manifest handler", "err", err)
		cancel()
		<-cluster.Done()
		<-bidengine.Done()
		<-bc.lc.Done()
		return nil, err
	}
```

### 2). Manifest Calls/Initiates an Event Bus to Monitor Lease Won Events

The `NewService` function called from `provider/manifest/service.go` subscribes to a RPC node event bus for new lease won processing.

Eventually the `run` method in this package is called with a service type passed in.

> [Source code reference location](https://github.com/akash-network/provider/blob/e7aa0b5b81957a130f1dc584f335c6f9e41db6b1/manifest/service.go)

```
func NewService(ctx context.Context, session session.Session, bus pubsub.Bus, hostnameService clustertypes.HostnameServiceClient, cfg ServiceConfig) (Service, error) {
	session = session.ForModule("provider-manifest")

	sub, err := bus.Subscribe()
	if err != nil {
		return nil, err
	}

	s := &service{
		session:         session,
		bus:             bus,
		sub:             sub,
		statusch:        make(chan chan<- *Status),
		mreqch:          make(chan manifestRequest),
		activeCheckCh:   make(chan isActiveCheck),
		managers:        make(map[string]*manager),
		managerch:       make(chan *manager),
		lc:              lifecycle.New(),
		hostnameService: hostnameService,
		config:          cfg,

		watchdogch: make(chan dtypes.DeploymentID),
		watchdogs:  make(map[dtypes.DeploymentID]*watchdog),
	}

	go s.lc.WatchContext(ctx)
	go s.run()

	return s, nil
}
```

### 3). Monitor Service Loop is Created to React to New Lease Won Events

Within the run function of `provider/manifest/service.go` an endless for loop monitors for events placed onto a channel.

```
	for {
		select {

		case err := <-s.lc.ShutdownRequest():
			s.lc.ShutdownInitiated(err)
			break loop

		case ev := <-s.sub.Events():
			switch ev := ev.(type) {

			case event.LeaseWon:
				if ev.LeaseID.GetProvider() != s.session.Provider().Address().String() {
					continue
				}
				s.session.Log().Info("lease won", "lease", ev.LeaseID)
				s.handleLease(ev, true)
```

The handleLease method is called ...

```
func (s *service) handleLease(ev event.LeaseWon, isNew bool) {
	// Only run this if configured to do so
	if isNew && s.config.ManifestTimeout > time.Duration(0) {
		// Create watchdog if it does not exist AND a manifest has not been received yet
		if watchdog := s.watchdogs[ev.LeaseID.DeploymentID()]; watchdog == nil {
			watchdog = newWatchdog(s.session, s.lc.ShuttingDown(), s.watchdogch, ev.LeaseID, s.config.ManifestTimeout)
			s.watchdogs[ev.LeaseID.DeploymentID()] = watchdog
		}
	}

	manager := s.ensureManager(ev.LeaseID.DeploymentID())

	manager.handleLease(ev)
}
```

New Manifest Manager instance is initiated.  When the manager instance is returned to the service `handleLease` method, a manager handleLease method is called which lives in `provider/manager/manager.go`.

```
func (s *service) ensureManager(did dtypes.DeploymentID) (manager *manager) {
	manager = s.managers[dquery.DeploymentPath(did)]
	if manager == nil {
		manager = newManager(s, did)
		s.managers[dquery.DeploymentPath(did)] = manager
	}
	return manager
}
```
