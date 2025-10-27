# Version Information

## Nhost Version

This template is based on **Nhost @nhost/dashboard@2.39.0** (latest as of deployment).

### Component Versions

- **Hasura GraphQL Engine**: v2.46.0-ce
- **Nhost Auth**: 0.40.2
- **Nhost Storage**: 0.7.2
- **Nhost Functions**: 22-1.4.0
- **Nhost Dashboard**: 2.39.0
- **PostgreSQL**: 16

## Update Policy

This template tracks the official Nhost component versions. To update:

1. Check the [Nhost GitHub releases](https://github.com/nhost/nhost/releases) for the latest versions
2. Update the image tags in `.do/deploy.template.yaml` and `.do/examples/production.yaml`
3. Review the Nhost [CHANGELOG](https://github.com/nhost/nhost/blob/main/CHANGELOG.md) for breaking changes
4. Test the deployment before pushing to production

### Checking for Updates

```bash
# Check latest Nhost releases
gh api repos/nhost/nhost/releases/latest --jq '.tag_name'

# Check Docker Hub for latest image versions
curl -s https://registry.hub.docker.com/v2/repositories/nhost/auth/tags | jq -r '.results[].name' | head -5
curl -s https://registry.hub.docker.com/v2/repositories/nhost/storage/tags | jq -r '.results[].name' | head -5
curl -s https://registry.hub.docker.com/v2/repositories/nhost/graphql-engine/tags | jq -r '.results[].name' | head -5
```

## Compatibility

- **DigitalOcean App Platform**: Compatible with current platform
- **PostgreSQL**: Requires version 12 or higher (template uses 16)
- **DigitalOcean Spaces**: S3-compatible storage for file uploads

## Last Updated

- **Template Created**: 2025-10-26
- **Last Verified**: 2025-10-26
