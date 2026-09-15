# datahub-v2-web-ui-helm

Helm chart deploying [datahub-v2-web-ui](https://github.com/Abdessalam7/datahub-v2-web-ui)
(FastAPI backend + React/Nginx frontend) to Kubernetes.

## Layout

```
Chart.yaml
values.yaml
templates/
  backend-deployment.yaml   backend Deployment, reads COS creds from a Secret
  backend-service.yaml
  backend-configmap.yaml    APP_ENV / LOG_LEVEL
  frontend-deployment.yaml  frontend Deployment, mounts the two nginx ConfigMaps
  frontend-service.yaml
  nginx-configmap.yaml      nginx server block (proxies /api/ to the backend)
  nginx-main-configmap.yaml nginx.conf
  ingress.yaml
```

The backend expects a Secret named `cos-credentials` (see
`values.yaml`'s `backend.cosSecret`) with keys `COS_ENDPOINT_URL`,
`COS_ACCESS_KEY_ID`, `COS_SECRET_ACCESS_KEY`, `COS_REGION`,
`COS_BUCKET_NAME` — these names must match `backend/config.py`'s
pydantic Settings fields exactly (`cos_endpoint_url` → `COS_ENDPOINT_URL`,
etc.), not the shorter `COS_ENDPOINT`/`COS_API_KEY`/`COS_BUCKET` names
used in one abandoned draft of this chart.

## Cleanup notes from import

This chart was assembled from a working directory that mixed the real
`datahub-v2-web` chart with several unrelated/stray files. The following
were **not** carried over:

- `Email`, a Python script manipulating a GPG private key — unrelated to
  this chart, flagged separately as a security concern.
- `templates/cos-secret` and `templates/test` — shell one-liners that
  **contained real, live COS credentials in plaintext**
  (access key, secret key, bucket name). **Rotate these credentials** —
  deleting the file does not remove them from git history.
- `values.yml`, `deployment.yml`, `configmap-instances.yml` — these
  reference `pysmoke.fullname` and the `pysmoke-test` image, i.e. they
  belong to the smoke-tests chart, not this one; they were misplaced in
  the same folder.
- `values-additions.yml` — a scratch "to merge into values.yaml" note.
- `datahub-v2-web-helm-v2.zip` — a zip archive of the chart itself.
- `templates/backend-deployment.yml` (the `.yml` duplicate of
  `backend-deployment.yaml`) — superseded; its COS env var names
  (`COS_ENDPOINT_URL`, `COS_ACCESS_KEY_ID`, ...) were the correct ones
  matching the backend and were merged into `backend-deployment.yaml`,
  replacing that file's stale `COS_ENDPOINT`/`COS_API_KEY`/
  `COS_INSTANCE_CRN`/`COS_BUCKET` names, which do not match what
  `backend/config.py` actually reads.

## Suggested follow-up

Given the leaked plaintext credentials above, consider replacing the
manual `kubectl create secret` step with an `ExternalSecret` (the same
External Secrets Operator + Vault `SecretStore` pattern already used
elsewhere in this environment, e.g. for the image pull secret) so the
COS credentials are never typed or committed in plaintext again. Not
included here since it requires the actual Vault KV path for these
credentials, which wasn't available at the time this chart was
assembled.
