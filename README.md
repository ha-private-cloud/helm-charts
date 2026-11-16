# charts

The Helm charts for the ClusterKeep apps. Each app's OpenTofu config installs its chart with a `helm_release` and passes in the values for each environment, so the charts only hold sensible defaults.

| Chart | App | Ingress |
|---|---|---|
| `clusterkeep-ui` | Front page, joining and logging in | Yes |
| `storage-ui` | File browser | Yes |
| `admin-ui` | Admin console | Yes |
| `auth-api` | Identity provider | Yes |
| `storage-api` | Storage backend | No, it's internal only |

Every chart has the same shape: a Deployment, a Service, a ServiceAccount and, apart from storage-api, an Ingress for ingress-nginx. Production is reached through the Cloudflare Tunnel rather than the ingress.

## Security defaults

Every pod runs as a non-root user with a read-only root filesystem, no privilege escalation, every Linux capability dropped and the `RuntimeDefault` seccomp profile. A writable `/tmp` is mounted as an `emptyDir`. Pods are spread across nodes and roll out one at a time without dropping below the replica count.

storage-api also mounts its shared NFS volume at `/data`, named by `dataClaimName`.

## How they're used

The `pipelines` workflows sync this repo onto the CI bastion before every build and release, and each app's `tofu/` points at `../../charts/<app>`. Changes here reach an environment the next time that app deploys there.

To check a chart renders before pushing:

```sh
helm lint storage-api --set dataClaimName=example
helm template example storage-ui --set image.tag=example
```
