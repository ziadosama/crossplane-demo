# Crossplane Demo

- [Crossplane Demo](#crossplane-demo)
  - [Setup](#setup)
    - [Enable the Crossplane Helm Chart repository:](#enable-the-crossplane-helm-chart-repository)
    - [Run the Helm dry-run to see all the Crossplane components Helm installs.](#run-the-helm-dry-run-to-see-all-the-crossplane-components-helm-installs)
    - [Install the Crossplane components using helm install.](#install-the-crossplane-components-using-helm-install)
    - [Install the AWS S3 provider into the Kubernetes cluster with a Kubernetes configuration file.](#install-the-aws-s3-provider-into-the-kubernetes-cluster-with-a-kubernetes-configuration-file)
    - [Create kubernetes secret with AWS credentials](#create-kubernetes-secret-with-aws-credentials)
    - [Apply the ProviderConfig with the this Kubernetes configuration file:](#apply-the-providerconfig-with-the-this-kubernetes-configuration-file)
  - [Apply Single resource](#apply-single-resource)
    - [Apply single managed resource](#apply-single-managed-resource)
  - [Apply full composition](#apply-full-composition)

## Setup
### Enable the Crossplane Helm Chart repository:
```
helm repo add \
crossplane-stable https://charts.crossplane.io/stable
helm repo update
```
### Run the Helm dry-run to see all the Crossplane components Helm installs.
```
helm install crossplane \
crossplane-stable/crossplane \
--dry-run --debug \
--namespace crossplane-system \
--create-namespace
```
### Install the Crossplane components using helm install.
```
helm install crossplane \
crossplane-stable/crossplane \
--namespace crossplane-system \
--create-namespace
```
### Install the AWS S3 provider into the Kubernetes cluster with a Kubernetes configuration file.
```
cat <<EOF | kubectl apply -f -
apiVersion: pkg.crossplane.io/v1
kind: Provider
metadata:
  name: provider-aws-s3
spec:
  package: xpkg.upbound.io/upbound/provider-aws-s3:v0.37.0
EOF
```
### Create kubernetes secret with AWS credentials
```
kubectl create secret \
generic aws-secret \
-n crossplane-system \
--from-file=creds=./aws-credentials.txt
```
### Apply the ProviderConfig with the this Kubernetes configuration file:
```
cat <<EOF | kubectl apply -f -
apiVersion: aws.upbound.io/v1beta1
kind: ProviderConfig
metadata:
  name: default
spec:
  credentials:
    source: Secret
    secretRef:
      namespace: crossplane-system
      name: aws-secret
      key: creds
EOF
```
## Apply Single resource

### Apply single managed resource
```
kubectl apply -f s3.yaml
```
## Apply full composition
```
### Apply API
```
kubectl apply -f composition/definition.yaml
```
### Apply composition containing managed resources
```
kubectl apply -f composition/composition.yaml
```
### Create resources
```
kubectl apply -f composition/claim.yaml
```