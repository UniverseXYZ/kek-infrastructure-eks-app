# kek-infrastructure-eks-app

## Deploying new versions

There are several environments in [universe-apps/environments](universe-apps/environments). To release a new versions, you must update the image `newTag` field in the environment's `kustomization.yaml` file. For example, to update the `dev` environment, you must edit [universe-apps/environments/dev/kustomization.yaml](universe-apps/environments/dev/kustomization.yaml):

``` yaml
apiversion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: dev
resources:
  - namespace.yaml
  - sources
...
images:
  - name: 076129510628.dkr.ecr.us-east-1.amazonaws.com/kekbackend
    newTag: NEW_VERSION_HERE
```

## Kubernetes access

### Prereq

* [kubectl](https://kubernetes.io/docs/tasks/tools/) v1.9.6 or later
* [awscli](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
* [aws-iam-authenticator](https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html)

### AWS config

Generate IAM access/secret keys for your AWS account and configure `awscli` to use them using `aws configure`.

### Kubectl config

Get the cluster `KUBECONFIG` file:

* DEV

``` sh
aws eks update-kubeconfig --name dev-i-universe-xyz
```

* PROD

``` sh
aws eks update-kubeconfig --name prod-i-universe-xyz
```

### Namespaces

* DEV cluster
** `dev` - development environment
** `alpha` - alpha environment

* PROD cluster
** `prod` - production environment


## DB access

Databases run in private networks and not accessible from the outside world. To connect to a database, you need Kubernetes access.

Username: `kekdao`
DB Name: `kekdao`

Get the password with:

``` sh
kubectl -n dev get secret api -o jsonpath='{.data.KD_DB_PASSWORD}' | base64 --decode
```

Then create a tunnel:

``` sh
kubectl -n dev port-forward $(kubectl -n dev get pods -l component=postgres-proxy -o jsonpath='{.items[0].metadata.name}') 5432
```

Replace `-n dev` with the name of the namespace/environment you need. You can now connect to `localhost:5432`
