# ABC Deployments
This is the repo that contains all the deployments for [ABC](https://gewis.nl/en/association/committee/ABC) projects. This is seperate from their respective repositories to prevent a cyclic problem in CI/CD. We can release new images of the software, and seperately change the config here to accomodate that release. This repository is open source for transparency, not specifically for contributions from outside the [ABC](https://gewis.nl/en/association/committee/ABC). But you are always welcome to open a PR.

## Ownership policy
There is a folder for each project that the ABC deploys, the teamlead of each project is also the codeowner of that folder. That means that everybody can submit PRs, but only code owners can approve those PRs. The owner of the full project is the ABC Chair/Secr, for meta information.

## Kubernetes implementation
Each project is free to choose the way they implement, as the size and requirement of each project differs drastically. They will all be sourced from the [k8-apps](https://github.com/GEWIS/k8s-apps) repo. If you want to add a project, you can open a PR there to have it added.

### Secrets
Secrets must be pushed as well, but can (or well should) be encrypted using kubeseal. You can seal them as follows:
#### Workflow

1. Fetch the kubeseal public cert (requires GEWIS network access):
   ```bash
   curl -s https://sealed-secrets.gewis.nl/v1/cert.pem > ../cert.pem
   ```

2. Copy a template and fill in real values:
   ```bash
   cp env.production.template.yaml env.prod.yaml
   # Edit env.production.yaml with real values
   ```

3. Seal the secret:
   ```bash
   kubeseal --cert ../cert.pem \
     --format yaml \
     --scope namespace-wide \
     < env.production.yaml \
     > /path/to/sealed-backend-env.yaml
   ```

4. Delete the plaintext file:
   ```bash
   rm backend-env.production.yaml
   ```

5. The sealed secret in `path/to/sealed-backend-env.yaml` is safe to commit.

## DNS Records, Database access, etc
Everything that cannot be configured here, can be requested at the CBC.