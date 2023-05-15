## Guestbook Example

This example shows how to build a simple multi-tier web application using
Kubernetes and Docker. The application consists of a web front end, Redis
master for storage, and replicated set of Redis slaves, all for which we will
create Kubernetes deployments, pods, and services.

This is based on [IBM/guestbook](https://github.com/IBM/guestbook/tree/master).

### Notes

 * As you read the correspoding `README.md` files, you will see `kubectl` commands that describes deployed resources. The output of these commands can be slightly vary depending on the version of your kubectl.
 * The guestbook applications uses Kubernetes service type `LoadBalancer` to expose the service onto an external IP address. If you are using `Minikube`, keep in mind that no real external load balancer is created. You
 can still access the service with the node port that is assigned to load balancer on `Minikube`.
