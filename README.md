# dataflow-cms-translation-app

Production NiFi flow app for CMS content translation.

Flow ownership:

1. Consume `batch.cms.content.translation.requests.v1`.
2. Build an OpenWebUI wrapper request from the Kafka message.
3. Invoke `cms-translation-service.ollama.svc.cluster.local`.
4. Upsert the translated JSON through `cms-content-resolver.directus.svc.cluster.local`.
5. Refresh the shell cache for the requested `page_slug` and `language`.
6. Route failures to `batch.cms.content.translation.requests.dlq.v1`.

The app intentionally does not use `dataflow-example-app`.

The app references the shared `dataflow/nifi-external` NiFi cluster but does
not own that cluster or its TLS auth secret.

The NiFi Registry flow ID is manifest-owned:

- `81895831-2aeb-45d8-ae08-16a60b74a4ad`

The `cms-translation-nifi-registry-bootstrap` Sync hook creates that Registry flow and version `1` if they do not already exist.

Required Vault values before first production sync:

- `secret/data/k8s-kafka-nifi-registry-bucket-id#value`

Cache refresh bearer tokens are Keycloak access tokens and expire. Do not store
a static cache refresh access token in Vault; the NiFi flow should mint one from
Keycloak client credentials at runtime when the refresh step is enabled.
