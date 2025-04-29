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

### Backups

ES uses `Snapshot` feature for backups. Given more time and
resources I would set it up to create these backups and store them in off-site
storage. Ideally using 3-2-1 backup strategy.
