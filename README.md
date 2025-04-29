# Exemplar production ready ES deploy to k8s

This repo provides a Helm chart for deploying an Elasticsearch cluster to
Kubernetes. It is designed to be production-ready, with focus on security,
scalability, and ease of use.

## Design

For production ready deployment I am going to use the [ECK operator][1]. It
provides an easy way to deploy production-ready ES with respect to time it
takes to get it up and running.

[1]: https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s

As for other options: manual deployment is out of question and custom Helm
chart would be simpler to deploy, but harder to make production ready

### Storage

The reclaim policy must be `Retain` to keep the data even if the PVC is
deleted. This creates demands for proper monitoring and removal of unused PVs.

To ensure the volumes get provisioned to the same node, I am going to use
`WaitForFirstConsumer` volume binding mode. This might increase the time of
provisioning, but it ensures that the volumes are provisioned efficiently
with respect to the pods.

### Secrets

For this solution I am going to use Kubernetes Secrets. It is not encrypted!
For production environment I would definitely either use encryption of Secrets
in `etcd`, or look into either 3rd party solution such as Hashicorp Vault or the
respective cloud providers secret management solution.

#### Critical questions

- How to manage secrets renewal and rotation?

### HA

For HA [ES docs][2] recommend at least:

- three master-eligible nodes
- two nodes of each role
- two copies of each shard

[2]: https://www.elastic.co/docs/deploy-manage/production-guidance/availability-and-resilience

### Resiliency

Depending on the environment size ES provides guides to how to set each up
resiliently. For true production ready cluster it is necessary to know the
environment ES is gonna be deployed at and design with this information in
mind.

Here are docs to ES's recommendations for:

- [small cluster][3]
- [large cluster][4]

[3]: https://www.elastic.co/docs/deploy-manage/production-guidance/availability-and-resilience/resilience-in-small-clusters
[4]: https://www.elastic.co/docs/deploy-manage/production-guidance/availability-and-resilience/resilience-in-larger-clusters

Based on the [docs][3] and my production expectations I am going to use three
nodes and [as the documentation recommends][5] set each of these as both: a
data node and a master-eligible node. I am also going to create a single copy
of each shard, so there are always two copies distributed through the three nodes.
This makes the cluster resilient to the failure of any single node, which
ought to create enough resiliency for my production environment.

[5]: https://www.elastic.co/docs/deploy-manage/production-guidance/availability-and-resilience/resilience-in-small-clusters#high-availability-cluster-design-three-nodes

This is the part I would always redesign from start based on my requirements
for processing speed and resource restrictions.

### Node placement in k8s cluster

Each ES node will be placed on a different node in the k8s cluster and they
should never run together on a single worker node. I will ensure this using
anti-affinity for pods and taints with tolerations for nodes.

### Backups

ES uses `Snapshot` feature for backups. Given more time and
resources I would set it up to create these backups and store them in off-site
storage. Ideally using 3-2-1 backup strategy.

### Certificate management

I am going to use in-built self-signed certificate manager from ECK operator.
For larger environment with stable domain name, or with custom CA authority I
would look into official docs and build upon the solution already present in
the company based on the [ES docs][6]

[6]: https://www.elastic.co/docs/deploy-manage/security/k8s-https-settings

## Sources

- Kagi search engine
- Perplexity.ai space [whalebones-task](https://www.perplexity.ai/search/create-multiple-approaches-for-QYzzeh80QXeMe6RQYdzIlw)
- [minikube official docs](https://minikube.sigs.k8s.io/docs/)
- [k8s official docs](https://kubernetes.io/docs/home/)
- [helm official docs](https://helm.sh/docs/)
- [elastic official docs](https://www.elastic.co/docs)
- my previous projects for helm chart inspiration
