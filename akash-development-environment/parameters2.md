# Parameters2

Parameters for use within the Runbooks detailed later in this guide.

| Name                | Default value                                                                                     | Effective on target(s)                                                                                                                                            |
| ------------------- | ------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| SKIP\_BUILD         | false                                                                                             |                                                                                                                                                                   |
| DSEQ                | 1                                                                                                 | <ul><li>deployment-* </li><li>lease-* </li><li>bid-* </li><li>send-manifest</li></ul>                                                                             |
| OSEQ                | 1                                                                                                 | <ul><li>deployment-* </li><li>lease-* </li><li>bid-* </li><li>send-manifest</li></ul>                                                                             |
| GSEQ                | 1                                                                                                 | <ul><li>deployment-* </li><li>lease-* </li><li>bid-* </li><li>send-manifest</li></ul>                                                                             |
| KUSTOMIZE\_INSTALLS | <p>Depends on runbook.<br>Refer to each runbook's <code>Makefile</code> to see default value.</p> | <ul><li>kustomize-init</li><li>kustomize-templates</li><li>kustomize-set-images</li><li>kustomize-configure-services </li><li>kustomize-deploy-services</li></ul> |

