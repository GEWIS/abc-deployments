# Sealed Secrets

This directory contains templates and sealed secrets for each environment.

## Secrets overview

| Secret | Purpose |
|--------|---------|
| `backend-env` | Database, LDAP, Stripe, SMTP, and other app credentials |
| `backend-jwt` | RSA private key for JWT signing (mounted at `/app/config/jwt.key`) |
| `docker-registry-pull` | Image pull credentials for `abc.docker-registry.gewis.nl` |

## Workflow

1. Fetch the kubeseal public cert (requires GEWIS network access):
   ```bash
   curl -s https://sealed-secrets.gewis.nl/v1/cert.pem > ../cert.pem
   ```

2. Copy a template and fill in real values:
   ```bash
   cp backend-env.production.template.yaml backend-env.production.yaml
   # Edit backend-env.production.yaml with real values
   ```

3. Seal the secret:
   ```bash
   kubeseal --cert ../cert.pem \
     --format yaml \
     --scope namespace-wide \
     < backend-env.production.yaml \
     > ../../overlays/production/sealed-backend-env.yaml
   ```

4. Delete the plaintext file:
   ```bash
   rm backend-env.production.yaml
   ```

5. The sealed secret in `overlays/<env>/sealed-backend-env.yaml` is safe to commit.

## JWT key workflow

Generate an RSA key and add it to the JWT template:

```bash
cp backend-jwt.production.template.yaml backend-jwt.production.yaml
# Paste the PEM under JWT_PRIVATE_KEY, indented two spaces per line:
#   JWT_PRIVATE_KEY: |
#     -----BEGIN PRIVATE KEY-----
#     ...
#     -----END PRIVATE KEY-----
# Or generate a new key for test:
#   JWT_PRIVATE_KEY: |
#     $(openssl genrsa 2048 | sed 's/^/    /')
```

Seal it:

```bash
kubeseal --cert ../cert.pem \
  --format yaml \
  --scope namespace-wide \
  < backend-jwt.production.yaml \
  > ../../overlays/production/sealed-backend-jwt.yaml

rm backend-jwt.production.yaml
```

When migrating from the config PVC, use the **existing** key from the PVC instead
of generating a new one, so active JWTs remain valid.

## Pull secret

Name: `docker-registry-pull` in each sudosos namespace.

1. Copy the template for the environment and fill in real values:
   ```bash
   cp docker-registry-pull.production.template.yaml docker-registry-pull.production.yaml
   # Edit with real registry credentials
   # (same flow with .test. for the test namespace)
   ```

2. Seal it:
   ```bash
   kubeseal --cert ../cert.pem \
     --format yaml \
     --scope namespace-wide \
     < docker-registry-pull.production.yaml \
     > ../../overlays/production/sealed-docker-registry-pull.yaml

   rm docker-registry-pull.production.yaml
   ```

3. The base manifests already reference it via
   `imagePullSecrets: docker-registry-pull` in every pod template that pulls
   from `abc.docker-registry.gewis.nl` (backend, backend-cron, docs, frontend,
   frontend-beta, pos, pos-develop, pdf, pdf-compiler); the sealed secret is
   committed in `overlays/<env>/sealed-docker-registry-pull.yaml`.

## Rotating secrets

Repeat the workflow steps for the relevant template. The sealed-secrets controller
will detect the update and unseal the new values.
