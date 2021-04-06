# Crossplane Configuration for HashiCorp Vault on GKE

This Crossplane Configuration defines compositions to run
a highly available Vault on GKE backed by a GCS bucket, configured to
auto unseal with Cloud KMS and authenticate using GCP workload identity.

It provides a control plane API to provision fully configured GKE clusters,
with secure networking, and a highly available Vault backed by a GCS bucket and
configured to auto unseal with Cloud KMS using GCP workload identity -- all
composed using cloud service primitives from the [Crossplane GCP
Provider](https://doc.crds.dev/github.com/crossplane/provider-gcp). App
deployments can securely connect to the infrastructure they need using
secrets distributed directly to the app namespace.

## Quick Start

#### Install the Crossplane kubectl extension (for convenience)

```console
curl -sL https://raw.githubusercontent.com/crossplane/crossplane/master/install.sh | sh
cp kubectl-crossplane /usr/local/bin
```

#### Install the Platform Configuration

```console
CONFIGURATION_PACKAGE=registry.upbound.io/hasan/vault-on-gke:v0.1.0

kubectl crossplane install configuration ${CONFIGURATION_PACKAGE}
kubectl get pkg
```

#### GCP Provider Setup

Set up your GCP account keyfile by following the instructions on:
https://crossplane.io/docs/v1.0/getting-started/install-configure.html#select-provider

Ensure that the following roles are added to your service account:

* `roles/compute.networkAdmin`
* `roles/container.admin`
* `roles/iam.serviceAccountUser`

Then create the secret using the given `creds.json` file:

```console
kubectl create secret generic gcp-creds -n crossplane-system --from-file=key=./creds.json
```

Create the `ProviderConfig`, ensuring to set the `projectID` to your specific GCP project:

```console
kubectl apply -f examples/provider-default-gcp.yaml
```

## Building Configuration Package

Create a `Repository` called `vault-on-gke` in your Upbound Cloud `Organization`.

Set these to match your settings:

```console
UPBOUND_ORG=acme
UPBOUND_ACCOUNT_EMAIL=me@acme.io
REPO=vault-on-gke
VERSION_TAG=v0.0.1
REGISTRY=registry.upbound.io
PLATFORM_CONFIG=${REGISTRY:+$REGISTRY/}${UPBOUND_ORG}/${REPO}:${VERSION_TAG}
```

Clone the GitHub repo.

```console
git clone https://github.com/upbound/vault-on-gke.git
cd vault-on-gke
```

Login to your container registry.

```console
docker login ${REGISTRY} -u ${UPBOUND_ACCOUNT_EMAIL}
```

Build package.

```console
kubectl crossplane build configuration --name package.xpkg --package-root configuration
```

Push package to registry.

```console
kubectl crossplane push configuration ${PLATFORM_CONFIG} -f configuration/package.xpkg
```

Install package into an Upbound `Platform` instance.

```console
kubectl crossplane install configuration ${PLATFORM_CONFIG}
```

The cloud service primitives that can be used in a `Composition` today are
listed in the Crossplane provider docs:

* [Crossplane GCP Provider](https://doc.crds.dev/github.com/crossplane/provider-gcp)

To learn more see [Configuration
Packages](https://crossplane.io/docs/v0.14/getting-started/package-infrastructure.html).
