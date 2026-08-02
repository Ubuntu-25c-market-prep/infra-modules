# infra-modules

Reusable Terraform modules consumed by `infra-aws`. Nothing here is applied
directly — modules are called, never run.

**Owner:** `@infra` · **Wave:** 1–2

## Why a separate repository

Modules version independently of the configurations that use them. A change to
the VPC module should not force a re-plan of the cluster layer, and a consumer
should be able to stay on `vpc/v1.2.0` while another moves to `v1.3.0`.

## Layout

```
<module>/
├── main.tf
├── variables.tf     every input has description + type
├── outputs.tf       every output has description
├── versions.tf      required_version + required_providers, no backend
├── README.md        inputs, outputs, and a worked example
└── examples/
    └── basic/       a runnable example that plans cleanly
```

Modules never contain a `backend` block, a `provider` block, or a hardcoded
account id. Providers are passed in by the caller.

## Versioning

Tags are **per module**: `<module>/vX.Y.Z`.

| Change | Bump |
|---|---|
| New optional variable, new output | minor |
| Bug fix, no interface change | patch |
| Removed or renamed variable/output, changed default, resource replacement | **major** |

A change that forces resource replacement is a major bump even if the interface
is identical. The interface is not the only contract; so is "applying this will
not destroy your database".

## Consuming a module

Always pin a tag. Never a branch.

```hcl
module "vpc" {
  source = "git::https://github.com/Ubuntu-25c-market-prep/infra-modules.git//vpc?ref=vpc/v1.2.0"

  name       = "u25c-shared"
  cidr_block = "10.0.0.0/16"
  az_count   = 3
}
```

A `ref` pointing at `main` means your infrastructure changes when someone else
merges. That is not a dependency, it is a surprise.

## Releasing

1. Pull request against `main`, reviewed by `@infra`.
2. `examples/basic` must plan cleanly.
3. Update the module's README — inputs, outputs, example.
4. Tag: `git tag vpc/v1.3.0 && git push origin vpc/v1.3.0`.
5. Consumers bump their `ref` in a separate pull request, so the module release
   and its adoption are independently revertible.

## Standards

[Terraform Standards](https://github.com/Ubuntu-25c-market-prep/ops-program/blob/main/docs/terraform-standards.md) ·
[State Strategy](https://github.com/Ubuntu-25c-market-prep/ops-program/blob/main/docs/terraform-state-strategy.md)
