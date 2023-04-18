# Use Runbook

## Overview

> _**NOTE**_ - this runbook requires three simultaneous terminals

For the purpose of documentation clarity we will refer to these terminal sessions as:

* terminal1
* terminal2
* terminal3

### STEP 1 - Open Runbook

> _**NOTE**_ - run the commands in this step on terminal1, terminal2, and terminal3&#x20;

Run this step on all three terminal sessions to ensure we are in the correct directory for later steps.

```
cd ~/go/src/github.com/akash-network/provider/_run/kube
```

### STEP 2 - Create and Provision Local Kind Kubernetes Cluster

> _**NOTE**_ - run this command in this step on terminal1 only

```
make kube-cluster-setup
```

#### Expected/Example Output

```
MacBook-Pro kube % cd ~/go/src/github.com/akash-network/provider/_run/kube
MacBook-Pro kube %
MacBook-Pro kube %
MacBook-Pro kube % make kube-cluster-setup
make -C /Users/scarruthers/go/src/github.com/akash-network/provider bins
make[1]: Entering directory '/Users/scarruthers/go/src/github.com/akash-network/provider'
creating .cache dir structure...
mkdir -p /Users/scarruthers/go/src/github.com/akash-network/provider/.cache
mkdir -p /Users/scarruthers/go/src/github.com/akash-network/provider/.cache/bin
mkdir -p /Users/scarruthers/go/src/github.com/akash-network/provider/.cache/include
mkdir -p /Users/scarruthers/go/src/github.com/akash-network/provider/.cache/versions
mkdir -p /Users/scarruthers/go/src/github.com/akash-network/provider/.cache
mkdir -p /Users/scarruthers/go/src/github.com/akash-network/provider/.cache/tests
mkdir -p /Users/scarruthers/go/src/github.com/akash-network/provider/.cache/run
installing modvendor v0.3.0 ...
rm -f /Users/scarruthers/go/src/github.com/akash-network/provider/.cache/bin/modvendor
GOBIN=/Users/scarruthers/go/src/github.com/akash-network/provider/.cache/bin GO111MODULE=on go install github.com/goware/modvendor@v0.3.0
go: downloading github.com/goware/modvendor v0.3.0
go: downloading github.com/mattn/go-zglob v0.0.2-0.20191112051448-a8912a37f9e7
rm -rf "/Users/scarruthers/go/src/github.com/akash-network/provider/.cache/versions/modvendor/"
mkdir -p "/Users/scarruthers/go/src/github.com/akash-network/provider/.cache/versions/modvendor/"
touch /Users/scarruthers/go/src/github.com/akash-network/provider/.cache/versions/modvendor/v0.3.0
GO111MODULE=on go mod tidy
go: downloading github.com/boz/go-lifecycle v0.1.1-0.20190620234137-5139c86739b8
go: downloading github.com/cosmos/cosmos-sdk v0.45.15
go: downloading github.com/akash-network/akash-api v0.0.11
go: downloading github.com/akash-network/node v0.23.0-rc7
go: downloading github.com/akash-network/cometbft v0.34.27-akash
go: downloading k8s.io/client-go v0.26.1
go: downloading k8s.io/api v0.26.1
<OUTPUT TRUNCATED>
```

### STEP 3 - Start Akash Node

> _**NOTE**_ - run this command in this step on terminal2 only

```
make node-run
```

### STEP 4 - Create a Provider

> _**NOTE**_ - run this command in this step on terminal1 only

```
make provider-create
```

### STEP 5 - Create and Verify Test Deployment

> _**NOTE**_ - run the commands in this step on terminal1 only

#### Create the Deployment

* Take note of the deplpyment ID (DSEQ) generated for use in subsequent steps

```
make deployment-create
```

#### Query Deployments

```
make query-deployments
```

#### Query Orders

* Steps ensure that an order is created for the deployment after a short period of time

```
make query-orders
```

#### Query Bids

* Step ensures the Provider services daemon bids on the test deployment

```
make query-bids
```

### STEP 6 - Test Lease Creation for the Test Deployment

> _**NOTE**_ - run the commands in this step on terminal1 only

#### Create Lease

```
make lease-create
```

#### Query Lease

```
make query-leases
```

#### Ensure Provider Received Lease Create Message

* Should see "pending" inventory in the provider status and for the test deployment

```
make provider-status
```

### STEP 7 - Send Manifest

> _**NOTE**_ - run the commands in this step on terminal1 only

#### Send the Manifest to the Provider

```
make send-manifest
```

#### Check Status of  Deployment

```
make provider-lease-status
```

#### Ping the Deplpyment to Ensure Liveness

```
 make provider-lease-ping
```

### STEP 8 - Verify Service Status

> _**NOTE**_ - run the commands in this step on terminal1 only

#### Query Lease Status

```
make provider-lease-status
```

#### Fetch Pod Logs

* Note that this will fetch the logs for all pods in the Kubernetes cluster.  Filter/search for the test deployment's ID (DSEQ) for related activities.

```
make provider-lease-logs
```
