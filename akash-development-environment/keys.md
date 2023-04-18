# Keys

Each configuration creates four [keys](https://github.com/akash-network/provider/blob/gpu/\_run/common.mk#L40..L43): They keys are assigned to the targets and under normal circumstances there is no need to alter it. However, it can be done with setting `KEY_NAME`:

```shell
# create provider from **provider** key
make provider-create

# create provider from custom key
KEY_NAME=other make provider-create
```
