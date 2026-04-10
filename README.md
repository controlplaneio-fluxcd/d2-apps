# d2-apps

> [!NOTE]
> This repository is part of the reference architecture for the
> [ControlPlane Enterprise for Flux CD](https://fluxcd.control-plane.io/).
>
> The `d2` reference architecture comprised of
> [d2-fleet](https://github.com/controlplaneio-fluxcd/d2-fleet),
> [d2-infra](https://github.com/controlplaneio-fluxcd/d2-infra) and
> [d2-apps](https://github.com/controlplaneio-fluxcd/d2-apps)
> is a set of best practices and production-ready examples for using Flux Operator
> and OCI Artifacts to manage the continuous delivery of Kubernetes infrastructure and
> applications on multi-cluster multi-tenant environments.
>
> Download the guide: [Flux D2 Architectural Reference](https://raw.githubusercontent.com/controlplaneio-fluxcd/distribution/main/guides/ControlPlane_Flux_D2_Reference_Architecture_Guide.pdf)

## Scope and Access Control

This repository is managed by the dev teams who are responsible for
the delivery of applications to the Kubernetes cluster fleet.

This repository is used to define the application components such as:

- Flux OCIRepositories (pointing to the application Helm charts in container registries)
- Flux HelmReleases for the applications with custom configuration per environment

This repository is reconciled on the cluster fleet by Flux as the **namespace admin**
and can't contain Kubernetes cluster-wide definitions such as CRDs, Cluster Roles, Namespaces, etc.

The platform team that manages the [d2-fleet repository](https://github.com/controlplaneio-fluxcd/d2-fleet)
is responsible for assigning the namespaces to the dev teams and configuring Flux with the
necessary RBAC to reconcile the `d2-apps` repository across the cluster fleet.

The platform team is also responsible for setting up any cluster-wide resources that the applications
depend on, such as CRD controllers, Ingress classes, Storage classes, etc. The cluster components
managed by the platform team are defined in the
[d2-infra repository](https://github.com/controlplaneio-fluxcd/d2-infra).

## Repository Structure

This repository contains the following directories:

- The **components** dir contains Flux HelmReleases that define how the applications
  are deployed to the cluster fleet and which configuration should be used for each environment.
- The **update-policies** dir contains the Flux policies for automating the application's
  version updates of the OCI charts referred in Helm releases.

The application components are grouped by namespace, and are defined in a directory with the following structure:

```sh
./components/
└── namespace
    ├── base
    │   ├── kustomization.yaml
    │   └── release1.yaml
    │   └── release2.yaml
    ├── production
    │   ├── kustomization.yaml
    │   ├── release1-values.yaml
    │   └── release2-values.yaml
    └── staging
        ├── kustomization.yaml
        ├── release1-values.yaml
        └── release2-values.yaml
```

## OCI Artifacts

Each component is published to a dedicated OCI repository, for example, the `frontend` component
is published to `oci://ghcr.io/controlplaneio-fluxcd/d2-apps/frontend` and is tagged as:

- `latest` for the main branch commits that modify the component.
- `vX.Y.Z` for the release tags matching the Git tag format `<component>/vX.Y.Z`.
- `latest-stable` points to the latest artifact tagged as `vX.Y.Z`.

GitHub workflows defined in `.github/workflows` are responsible for publishing and signing
the component artifacts to GHCR using the
[controlplaneio-fluxcd/distribution/actions/push](https://github.com/controlplaneio-fluxcd/distribution/tree/main/actions/push)
GitHub Action.

Workflows:

- [push-artifact](https://github.com/controlplaneio-fluxcd/d2-apps/blob/main/.github/workflows/push-artifact.yaml) - triggered on commits to main branch, publishes the component artifact tagged as `latest`.
- [release-artifact](https://github.com/controlplaneio-fluxcd/d2-apps/blob/main/.github/workflows/release-artifact.yaml) - triggered on Git tags matching the format `<component>/vX.Y.Z`, publishes the component artifact tagged as `vX.Y.Z` and updates the `latest-stable` tag to point to the new release.
