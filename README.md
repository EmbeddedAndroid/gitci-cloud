# git://ci Cloud Deployments

This repository holds all the Kubernetes [Namespace](http://kubernetes.io/docs/user-guide/namespaces/), [Service](http://kubernetes.io/docs/user-guide/services/) and [Deployment](http://kubernetes.io/docs/user-guide/deployments/), [PersistentVolume](http://kubernetes.io/docs/user-guide/persistent-volumes/) and [PersistentVolumeClaim](http://kubernetes.io/docs/user-guide/persistent-volumes/#persistentvolumeclaims) files needed
to run git://ci in the cloud.

Cloud here is intended to be the [Google Cloud Platform](https://cloud.google.com/) using [Google Container Engine](https://cloud.google.com/container-engine/).

## Repository Structure

All Kubernetes files are stored in the [k8s/](./k8s/) directory.
Helper scripts are stored in the [bin/](./bin/) directory.

### File Naming Conventions

Services:

* File names are numbered starting from 000 to 099.
* Following is the prefix `srv`.
* Then the given name for the service to create.

Deployments:

* File names are numbered starting from 900 to 999.
* Following is the prefix `deploy`.
* Then the given name for the deployment to create.

Service and Deployment given names, where possible, should be the same.

Namespaces:

This is a special case, and it should be needed only once, at first run of the
deployment.

The file naming structure follows the Service one, but with the `ns` prefix: the
namespace is needed before everything else is deployed.

Persistent volumes:

* File names are numbered starting from 300 to 399.
* Following is the prefix `pv`.
* Following is the prefix `rw` or `ro` for read-write and read-only access.
* Then the given name for the persistent volume.

Persistent volume claims:

* File names are numbered starting from 400 to 499.
* Following is the prefix `pvc`.
* Following is the prefix `rw` or `ro` for read-write and read-only access.
* Then the given name for the persistent volume claim.

Examples:

* 000-ns-bar.yml
* 000-srv-foo.yml
* 300-pv-rw-pd.yml
* 400-pvc-rw-pd.yml
* 900-deploy-foo.yml

#### Other Naming Conventions

Namespace, service, deployment, container and volume names should, in order,
start with the following prefixes:

* ns
* srv
* deploy
* con
* pv
* pvc

Examples:

```yaml
metadata:
    name: ns-foo
```

```yaml
metadata:
    name: srv-bar
```

## How to deploy

For best practices:

1. Namespace should be the first to be created.
2. Services should be created before Deployments and Pods.
3. PersistentVolume and PersistentVolumeClaims should be created before
Deployments and Pods.

It is assumed that you have installed on your computer the [Google Cloud SDK](https://cloud.google.com/sdk/),
and it has been correctly initialized (`gcloud init`, etc...).

To deploy all in a single batch, using the `kubectl` command line tool:

    kubectl create -f ./k8s/

To make sure that the previous command runs correctly, a few things need to be
created beforehand that is not possible to create with Kubernetes configuration
files:

* Volumes
* Assign a static IP address (see Notes)

### Notes

* The [srv-gitci](./k8s/000-srv-gitci.yml) Service, the one that enables all
traffic through the deployment, is a [LoadBalancer](http://kubernetes.io/docs/user-guide/services/#type-loadbalancer):
it will request an IP address from Compute Engine and setup the correct
forwarding rules. The assigned IP should be set as
static in order to point DNS entries at. Either do it beforehand and set the
`loadBalancerIP` in the Service file to that address, or after running the
service creation. This can be done from the Compute Engine web interface.
* Namespace, services, persistent volumes and persistent volume claims must
all be ran just once per deployment, unless they need to be destroyed and
recreated from scratch.
