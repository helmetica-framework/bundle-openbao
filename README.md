# Bundle OpenBao

A Helmetica bundle chart for OpenBao.

## Seal Secret Bootstrap

This chart includes a Helm `post-install` hook job that creates a seal secret with a random value.

- The job only runs on install (not upgrade).
- If the secret already exists, the job exits without changing it.
- Configure it through `sealSecret.*` in `values.yaml`.

Important: `sealSecret.name` must match `openbao.server.extraSecretEnvironmentVars[].secretName`.
