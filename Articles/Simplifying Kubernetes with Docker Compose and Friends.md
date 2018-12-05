Simplifying Kubernetes with Docker Compose and Friends

Today we’re happy to announce we’re open sourcing our support for using Docker [Compose on Kubernetes](https://github.com/docker/compose-on-kubernetes). We’ve had this capability in Docker Enterprise for a little while but as of today you will be able to use this on any Kubernetes cluster you choose.

![Compose on Kubernetes](../_resources/342b6896feef4576afc78aa4e6c794f0.png)

### **Why do I need Compose if I already have Kubernetes?**

The Kubernetes API is really quite large. There are more than 50 first-class objects in the latest release, from Pods and Deployments to ValidatingWebhookConfiguration and ResourceQuota. This can lead to a verbosity in configuration, which then needs to be managed by you, the developer. Let’s look at a concrete example of that.

The [Sock Shop](https://github.com/microservices-demo/microservices-demo) is the canonical example of a microservices application. It consists of multiple services using different technologies and backends, all packaged up as Docker images. It also provides example configurations using different tools, including both Compose and raw Kubernetes configuration. Let’s have a look at the relative sizes of those configurations:

```
$ git clone https://github.com/microservices-demo/microservices-demo.git
$ cd deployment/kubernetes/manifests
$ (Get-ChildItem -Recurse -File | Get-Content | Measure-Object -line).Lines
908
$ cd ../../docker-compose
$ (Get-Content docker-compose.yml | Measure-Object -line).Lines
174
```

Describing the exact same multi-service application using just the raw Kubernetes objects takes more than 5 times the amount of configuration than with Compose. That’s not just an upfront cost to author – it’s also an ongoing cost to maintain. The Kubernetes API is amazingly general purpose – it exposes low-level primitives for building the full range of distributed systems. Compose meanwhile isn’t an API but a high-level tool focused on developer productivity. That’s why combining them together makes sense. For the common case of a set of interconnected web services, Compose provides an abstraction that simplifies Kubernetes configuration. For everything else you can still drop down to the raw Kubernetes API primitives. Let’s see all that in action.

First we need to install the Compose on Kubernetes controller into your Kubernetes cluster. This controller uses the standard Kubernetes extension points to introduce the \`Stack\` to the Kubernetes API. You can use any Kubernetes cluster you like, but if you don’t already have one available then remember that [Docker Desktop](https://www.docker.com/products/docker-desktop) comes with Kubernetes and the Compose controller built-in, and enabling it is as simple as ticking a box in the settings.

To install the controller manually on any Kubernetes cluster, see the [full documentation](https://github.com/docker/compose-typescript) for the current installation instructions.

Next let’s write a simple Compose file:

```
version: "3.7"
services:
  web:
    image: dockerdemos/lab-web
    ports:
     \- "33000:80"
  words:
    image: dockerdemos/lab-words
    deploy:
      replicas: 3
      endpoint_mode: dnsrr
  db:
    image: dockerdemos/lab-db
```

We’ll then use the docker client to deploy this to a Kubernetes cluster running the controller:

```
$ docker stack deploy --orchestrator=kubernetes -c docker-compose.yml words
Waiting for the stack to be stable and running...
db: Ready       \[pod status: 1/1 ready, 0/1 pending, 0/1 failed\]
web: Ready      \[pod status: 1/1 ready, 0/1 pending, 0/1 failed\]
words: Ready    \[pod status: 1/3 ready, 2/3 pending, 0/3 failed\]
Stack words is stable and running
```

We can then interact with those objects via the Kubernetes API. Here you can see we’ve created the lower-level objects like Services, Pods, Deployments and ReplicaSets automatically:

```
$ kubectl get all
NAME                       READY  STATUS RESTARTS   AGE
pod/db-85849797f6-bhpm8    1/1  Running  0     57s
pod/web-7974f485b7-j7nvt   1/1  Running  0     57s
pod/words-8fd6c974-44r4s   1/1  Running  0     57s
pod/words-8fd6c974-7c59p   1/1  Running  0     57s
pod/words-8fd6c974-zclh5   1/1  Running  0     57s

NAME                    TYPE  CLUSTER-IP      EXTERNAL-IP  PORT(S) AGE
service/db              ClusterIP None       &lt;none&gt; 55555/TCP 57s
service/kubernetes      ClusterIP 10.96.0.1       &lt;none&gt; 443/TCP 4d
service/web             ClusterIP None       &lt;none&gt; 55555/TCP 57s
service/web-published   LoadBalancer  10.102.236.49   localhost  33000:31910/TCP  57s
service/words           ClusterIP None       &lt;none&gt; 55555/TCP 57s

NAME                    DESIRED  CURRENT  UP-TO-DATE   AVAILABLE  AGE
deployment.apps/db      1  1  1         1  57s
deployment.apps/web     1  1  1         1  57s
deployment.apps/words   3  3  3         3  57s

NAME                             DESIRED  CURRENT  READY  AGE
replicaset.apps/db-85849797f6    1  1  1  57s
replicaset.apps/web-7974f485b7   1  1  1  57s
replicaset.apps/words-8fd6c974   3  3  3  57s
```

It’s important to note that this isn’t a one-time conversion. The Compose on Kubernetes API Server introduces the Stack resource to the Kubernetes API. So we can query and manage everything at the same level of abstraction as we’re building the application. That makes delving into the details above useful for understanding how things work, or debugging issues, but not required most of the time:

```
$ kubectl get stack
NAME      STATUS   PUBLISHED PORTS   PODS  AGE 
words     Running  33000      5/5 4m
```

### **Integration with other Kubernetes tools**

Because Stack is now a native Kubernetes object, you can work with it using other Kubernetes tools. As an example, save the as \`stack.yaml\`:

```
kind: Stack
apiVersion: compose.docker.com/v1beta2
metadata:
 name: hello
spec:
 services:
 \- name: hello
   image: garethr/skaffold-example
   ports:
   \- mode: ingress
     target: 5678
     published: 5678
     protocol: tcp
```

You can use a tool like [Skaffold](https://github.com/GoogleContainerTools/skaffold) to have the image automatically rebuild and the Stack automatically redeployed whenever you change any of the details of your application. This makes for a great local inner-loop development experience. The following \`skaffold.yaml\` configuration file is all you need.

```
apiVersion: skaffold/v1alpha5
kind: Config
build:
 tagPolicy:
   sha256: {}
 artifacts:
 \- image: garethr/skaffold-example
 local:
   useBuildkit: true
deploy:
 kubectl:
   manifests:
     \- stack.yaml
```

### **The future**

We already have some thoughts about a Helm plugin to make describing your application with Compose and deploying with Helm as easy as possible. We have lots of other ideas for helping to simplify the developer experience of working with Kubernetes too, without losing any of the power of the platform. We also want to work with the wider Cloud Native community, so if you have ideas and suggestions please let us know.

Kubernetes is designed to be extended, and we hope you like what we’ve been able to release today. If you’re one of the millions of Compose users you can now more easily move to and manage your applications on Kubernetes. If you’re a Kubernetes user struggling with too much low-level configuration then give Compose a try. Let us know in the comments what you think, and head over to GitHub to try things out and even open your first PR:

*   [Compose on Kubernetes controller](https://github.com/docker/compose-typescript)

[docker compose](https://blog.docker.com/tag/docker-compose/), [dockercon](https://blog.docker.com/tag/dockercon/), [Kubernetes](https://blog.docker.com/tag/kubernetes/), [open source](https://blog.docker.com/tag/open-source/)