英文地址：[https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)

## 介绍

---

Kubernetes的DNS schedules a DNS Pod and Service on the cluster, and configures the kubelets to tell individual containers to use the DNS Service’s IP to resolve DNS names.

### What things get DNS names? {#what-things-get-dns-names}

Every Service defined in the cluster \(including the DNS server itself\) is assigned a DNS name. By default, a client Pod’s DNS search list will include the Pod’s own namespace and the cluster’s default domain. This is best illustrated by example:

Assume a Service named`foo`in the Kubernetes namespace`bar`. A Pod running in namespace`bar`can look up this service by simply doing a DNS query for`foo`. A Pod running in namespace`quux`can look up this service by doing a DNS query for`foo.bar`.

The following sections detail the supported record types and layout that is supported. Any other layout or names or queries that happen to work are considered implementation details and are subject to change without warning. For more up-to-date specification, see[Kubernetes DNS-Based Service Discovery](https://github.com/kubernetes/dns/blob/master/docs/specification.md).

  


