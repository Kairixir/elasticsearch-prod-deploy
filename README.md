# Exemplar production ready ES deploy to k8s

This repo provides a Helm chart for deploying an Elasticsearch cluster to
Kubernetes. It is designed to be production-ready, with focus on security,
scalability, and ease of use.

## Design

For production ready deployment I am going to use the [ECK operator][1]. It
provides an easy way to deploy production-ready ES with respect to time it
needs to be up and running.

[1]: https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s

