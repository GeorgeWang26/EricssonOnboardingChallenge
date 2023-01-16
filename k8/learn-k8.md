# Kubernetes
https://matthewpalmer.net/kubernetes-app-developer/articles/kubernetes-deployment-tutorial-example-yaml.html
https://matthewpalmer.net/kubernetes-app-developer/articles/service-kubernetes-example-tutorial.html

## Cluster
k8 cluster consts of one or more nodes (VM/physical machine) and one control plane (users interact with the cluster through this control plane)


## Node
Nodes are the ones that actually run containers. Every node has a "kubelet", so it can communicate with control plane. Kubelet also manages the pods (multiple pods can run on one node) and containers in the node.

In each node, it also has a contrainer tool (docker/containerd)


## kubectl
After cluster is created, user interact with the cluster using kubectl. When using kubectl, client is the host (user/client), server is the control plane.

Pods live in the cluster and are running on a private, isolated k8 network. So by default, they are visible to other pods in the same cluster, but not outside the network (aka, outside the cluster, so on the host machine). By using kubectl, interactions are made with the API endpoins on the control plane, which is in the cluster network. This way, communication with pods can be made from outside the cluster (with proxy or by exposing the pods?????).


## Deployment
Deployment tells k8 (control plane) how to create&update instances of applications. Deployment includes the container image(s) for the application, and the number of replicas want to run. This can be changed later


## Pod
Pod is a group of one or more containers that all live in the same node, so same physical or virtual machine in the cluster. Usually don't need to create pod directly, they are created automatically when using deployment

k8 (control plane) manage pods on each node instead of the containers directly. It is sufficient, and argubly better to have "one-container-per-pod", so new containers can be added easily

Pod is the smallest deployable unit in k8. It is a group of containers with shared storage and network resources and a specification of how to run these containers. Pod models an application specific "logical lost": it contains one or more containers for the same application that are relatively tightly coupled. 

In non-cloud context, applications executed on the same physical or virtual machine are similar to cloud applications executed on the same logical host. non-cloud localhost is like the cloud logical host

From a network standpoint, each container within the pod shares the same networking namespace. This gives each container access to the same network resources, such as the pod's IP address and port space. **Containers within the same pod can also communicate with each other over localhost.**




## Services
services expose pods to outside the cluster, and it is better than proxy. Becuase when sending API request through proxy, the POD_NAME needs to be specified, but if pod dies and is replaced with new one, the POD_NAME will change. And this will affect the way pods are accessed from outside the cluster. Will need to replace destination with the new POD_NAME. But with services, this is not needed, everything is abstracted and encapsulated

Types of services
- ClusterIP: only reachable from inside the cluster
- NodePort: expose service on same port of every node who are in the deployment (include all replicas), service is accessible from outside with NODE_IP:NODE_PORT
- LoadBalancer: assigns a fixed, external IP to the service that is visible from outside the cluster
- ExternalName: map the service to a domain name (ie: foo.bar.com)

Services use "labels and selector" to find the wanted pod. Labels are key-value pairs (ie, app: A) that can be attached to k8 objects (pod) at creation time or later on, and can be modified anytime. Selector will find the wanted pod through key-value pair matching (ie, app=A)


## minikube commands
- `minikube start/stop/delete`
- `minikube profile list`


## kubectl commands
- `kubectl version`
- `kubectl cluster-info`
- `kubectl get nodes/deployments/pods/services`
- `kubectl describe nodes/deployments/pods/services`
- `kubectl create deployment DEPLOYMENT_NAME --image=DOCKER_IMAGE`
    - searched for a suitable node where an instance of the application could be run
    - scheduled the application to run on that Node
    - configured the cluster to reschedule the instance on a new Node when needed
- `kubectl expose deployment/DEPLOYMENT_NAME --type=SERVICE_TYPE --port=POD_PORT`
- `kubectl delete deployment/service NAME` or use tags
- `kubectl apply/delete -f FILE`
- `kubectl label pods POD_NAME LABEL_KEY=LABEL_VAL`
- `kubectl logs POD_NAME`
    - print the logs from containers in a pod (all stdout will be stored as logs for containers in pod)
- `kubectl exec POD_NAME -- CMD`
    - execute a command on a container in a pod, this can be ANY container, since they all share same storage and network space, it's like they are all localhost
- `kubectl exec -it POD_NAME -- /bin/bash`
    - tab into terminal of a container in a pod, just like with regular docker. This can be ANY container for same reason as above
- `kubectl proxy`
    - start a proxy (on port 8001) that forward communications into the cluster-wide, private network
    - `curl http://localhost:8001/`
        - get all available paths
    - `curl http://localhost:8001/api/v1/namespaces/default/pods/POD_NAME/proxy/API_ENDPOINT`
        - interact with application (container in pod) through proxy




## Hypothesis of how it works
Since all images/containers can be whereever, so it doesnt matter if they are on same node or distributed among the cluster, and if the node is a VM or physical machine or even same machine as control plane. This is a HUGE upside of using k8, cuz at deployment, all you need to know is "who are in the cluster" and "what images do you want to distribute in the cluster", then k8 control plane will automatically distribute/manage/schedule the images to different nodes in the cluster. The communication between containers are done using the k8 network (centered around control plane), so it doesnt matter if the containers are in same node or not.
