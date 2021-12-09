# kek-infrastructure-eks-app
## WARNING: This repo is highly sensitive; all changes to it are automatically pulled into the Kubernetes clusters via FluxCD
## Only make changes to this if you know exactly what you are doing!

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

* [kubectl](https://kubernetes.io/docs/tasks/tools/) v1.19.6 or later
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

`dev` - development environment

`alpha` - alpha environment

* PROD cluster

`prod` - production environment

### Logs

To check the logs for a pod, first get the pod list:

``` sh
kubectl -n alpha get pods

NAME                              READY   STATUS    RESTARTS   AGE
api-796cc97745-dx5s4              1/1     Running   4          21h
api-796cc97745-jgvpf              1/1     Running   3          21h
notifications-7c947bb4dc-ccqhs    1/1     Running   3          21h
postgres-proxy-55bbf876c5-pkbsk   1/1     Running   0          21h
redis-master-0                    1/1     Running   0          21h
scrape-f556b767d-kws4t            1/1     Running   2          21h
```

Then tail the logs for the desired pod:

``` sh
kubectl -n alpha logs -f scrape-f556b767d-kws4t
...
time="2021-05-26T11:02:25Z" level=info msg="[core] processing block" block=12508123
time="2021-05-26T11:02:30Z" level=error msg="[core] Post \"https://mainnet.infura.io/v3/25763b5ba15644a8b3079d3ee755bce5\": context deadline exceeded (Client.Timeout exceeded while awaiting headers)" block=12508123
time="2021-05-26T11:02:32Z" level=info msg="[core] processing block" block=12508119
time="2021-05-26T11:02:37Z" level=error msg="[core] Post \"https://mainnet.infura.io/v3/25763b5ba15644a8b3079d3ee755bce5\": context deadline exceeded (Client.Timeout exceeded while awaiting headers)" block=12508119
time="2021-05-26T11:02:39Z" level=info msg="[core] processing block" block=12508123
time="2021-05-26T11:02:45Z" level=error msg="[core] Post \"https://mainnet.infura.io/v3/25763b5ba15644a8b3079d3ee755bce5\": context deadline exceeded (Client.Timeout exceeded while awaiting headers)" block=12508123
time="2021-05-26T11:02:47Z" level=info msg="[core] processing block" block=12508119
```

Alternatively, if there are multiple replicas of a service, you can use `labels` to tail all pods from the same terminal:

``` sh
kubectl -n alpha logs -f -l component=api
...
[GIN] 2021/05/26 - 11:04:52 | 200 |      1.2064ms |    85.118.76.43 | GET      /api/notifications/list?target=0x67b93852482113375666a310ac292D61dDD4bbb9
[GIN] 2021/05/26 - 11:04:58 | 200 |     573.229µs |   10.107.114.55 | GET      /health
[GIN] 2021/05/26 - 11:05:05 | 200 |      609.92µs |   10.107.114.55 | GET      /health
[GIN] 2021/05/26 - 11:05:08 | 200 |      602.34µs |   10.107.114.55 | GET      /health
[GIN] 2021/05/26 - 11:05:12 | 200 |    1.281461ms |   10.107.114.55 | GET      /health
```

You can check the labels of a Pod with:

``` sh
kubectl -n alpha describe pod api-796cc97745-jgvpf

Name:         api-796cc97745-jgvpf
Namespace:    alpha
Priority:     0
Node:         ip-10-107-114-55.ec2.internal/10.107.114.55
Start Time:   Tue, 25 May 2021 16:54:32 +0300
Labels:       app=backend
              component=api
              environment=alpha
```

**NOTE**: replace `-n alpha` with the name of the namespace.

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
