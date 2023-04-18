# Development Environment General Behavior

All examples are located within [\_run](https://github.com/akash-network/provider/blob/gpu/\_run) directory. Commands are implemented as `make` targets.

There are three ways we use to set up the Kubernetes cluster.

* kind
* minukube
* ssh

Both `kind` and `minikube` are e2e, i.e. the configuration is capable of spinning up cluster and the local host, whereas `ssh` expects cluster to be configured before use.
