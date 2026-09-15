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

## Requirements

The backend expects a Secret named `cos-credentials` in the target namespace
(see `values.yaml`'s `backend.cosSecret`) with keys `COS_ENDPOINT_URL`,
`COS_ACCESS_KEY_ID`, `COS_SECRET_ACCESS_KEY`, `COS_REGION`,
`COS_BUCKET_NAME`, matching `backend/config.py`'s pydantic Settings fields.
This chart does not create that Secret — provision it separately (an
`ExternalSecret` sourced from Vault is recommended over a manually created
plaintext Secret).

An image pull secret (`global.imagePullSecret`) must also exist in the
namespace.
