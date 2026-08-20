# Bundle OpenBao

A Helmetica bundle chart for OpenBao.

## Configuration

The user facing surface is small, everything else is computed from it with CEL expressions
that chrysopoeia evaluates before the values reach Helm:

- `hostname`: the public hostname. Empty (the default) disables the ingress, otherwise the
  ingress, its hosts and its TLS entry are derived from it. The certificate is stored in
  `<claim name>-tls`.
- `platform`: `openshift` or `kubernetes`. Drives `global.openshift` and the ingress class.
- `openbao.ui.enabled`: toggles the UI, both in the chart and in the raft server config.

Because these fields are computed, they are not part of the generated CRD. Installing this chart
with plain Helm instead of through chrysopoeia leaves the `cel:` expressions as literal strings.

## Seal Secret Bootstrap

This chart includes a Helm `pre-install` hook job that creates a seal secret with a random value.

- The job only runs on install (not upgrade).
- If the secret already exists, the job exits without changing it.
- Configure it through `sealSecret.*` in `values.yaml`.

The secret is named `<claim name>-seal`. Both `sealSecret.name` and
`openbao.server.extraSecretEnvironmentVars[].secretName` compute that name from `claim.name`,
so they cannot drift apart. Neither may be derived from the other: CEL expressions never see
each other's results.

## Verifying the expressions

The expressions are compiled and type-checked by chrysopoeia. To check them without a cluster,
run them through `pkg/celvalues` of a chrysopoeia checkout:

```go
prep, err := celvalues.New(valuesYaml)          // compile, reports chart author errors
out, err := prep.Apply(claimValues, celvalues.Claim{Name: "dev", Namespace: "bao"})
```
