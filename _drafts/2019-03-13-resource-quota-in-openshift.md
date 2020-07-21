---
layout: post
title: Resource Quota in Openshift
permalink: openshift-resource-quota
date: 2018-03-13
---
In entreprise deployment of Openshift, The Openshift cluster is meant to be shared and utilized by several development teams to deploy their apps. Hence, it is required to have a logically isolated work environments. This can be achieved whenever a project is created in Openshift, But Openshift project does not enforce limits and quotas by default.

Generally speaking, it is good practice to enforce quotas on an Openshift cluster. Quotas ensure that no single user is able to utilize more than what has been allocated and is a critical component in driving overall cluster utilization. We enforce user quota by enabling the ResourceQuota controller.

This controller ensures that any newly declared Pods are first evaluated against the current quota utilization for the given project (namespace). By performing this check during workload onboarding, we give immediate notice to a user that his Pod will or will not fit within the quota. Note, too, that when a quota is defined for a project (namespace), all Pod definitions (even if originating from another resource, such as Deployments or ReplicaSets) are required to specify resource requests and limits.

Quotas may be implemented for an ever-expanding list of resources, but some of the most common include CPU, memory, and volumes. It is also possible to place quotas on the number of distinct Openshift resources (e.g., Pods, Deployments, Jobs, and more) within a Namespace.

The following tutorial will focus on how to define enable quota on various distinct Openshift resources:

### 1. Configuring Basic Quota (Limiting 2 Pods)

Enabling ResourceQuota controller on a project (e.g demo project) can be done `declaratively` via a yaml file:

```sh
$ cat quota.yml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: count-quotas
spec:
  hard:
    pods: "2"
```
```
# oc create -f quota.yaml -n demo
resourcequota/count-quotas created
```

Alternatively, same quota as above can be enabled by running `imperative` command as below:
```
# oc create quota count-quotas --hard=pods=2
```

Let’s create one Pod to utilize the pods quota.
```
# cat podQuota.yaml
apiVersion: v1
kind: Pod
metadata:
  name: quota-pod-1
spec:
  containers:
  - name: quota-container
    image: busybox
    imagePullPolicy: IfNotPresent    
    command: ['sh', '-c', 'echo Pod is Running ; sleep 3600']
         
  restartPolicy: Never
  terminationGracePeriodSeconds: 0
```
Nothing special about this Pod spec. Pods need no special action or spec definition to use quota-limited resource

> __*Notes:*__ 
> * The pod above is set to have parameter `terminationGracePeriodSeconds: 0`
> * By default, all deletes are graceful within 30 seconds. This allows time for shutdown routines to run.
> * I set it to 0 to speed up the deletion of the many Pods throughout this tutorial.

To create the pod as defined above, we run the CLI command:
```
# oc create -f podQuota.yaml
pod/quota-pod-1 created
```
To check current quota utilization:
```
# oc describe quota count-quotas
Name:       count-quotas
Namespace:  demo
Resource    Used  Hard
--------    ----  ----
pods        1     2
```

We want to exceed quota, so we need more Pods.
Modify the pod name from pod definition above to another name (eg. quota-pod-2), then create another pod.

```
# oc create -f podQuota.yaml
pod/quota-pod-2 created
```

Now, investigate our quota:
```
# oc describe quota count-quotas
Name:       count-quotas
Namespace:  demo
Resource    Used  Hard
--------    ----  ----
pods        2     2

```

We want to exceed quota, so we need one more Pods.
```
# oc create -f podQuota.yaml
Error from server (Forbidden): error when creating "podQuota.yaml": pods "quota-pod-3" is forbidden: exceeded quota: object-counts, requested: pods=1, used: pods=2, limited: pods=2
```

As expected, the 3rd pod creation is failed as it excedeed quota (Quota enforced)

Delete Pods and resourcequota objects.

```
# oc delete all --all -n demo
# oc delete quota count-quotas -n demo
```
<br>

### 2. CPU Quota on Requests and Limits

In Openshift (Kubernetes), Quota is utilized by specifying two different resource metrics: 
* __*Resource requests*__ specify the minimum amount of a resource required to run the application. 
* __*Resource limits*__ specify the maximum amount of a resource that an application can consume. 




### Resource Requests: Minimum Required Resources

With Kubernetes, a Pod requests the resources required to run its containers. Kubernetes guarantees that these resources are available to the Pod. The most commonly requested resources are CPU and memory, but Kubernetes has support for other resource types as well, such as GPUs and more.

For example, to request that the apache container lands on a machine with half a CPU free and gets 128 MB of memory allocated to it, we define the Pod as shown in example below:

_Example ``pod-request.yaml``_
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: apache-demo
spec:
  containers:
    - image: zelid/apache:v1
      name: apache
      resources:
        requests:
          cpu: "500m"
          memory: "128Mi"
      ports:
        - containerPort: 8080
          name: http
          protocol: TCP
```

>**NOTE:**
>Resources are requested per container, not per Pod. The total resources requested by the Pod is the sum of all resources requested by all containers in the Pod. The reason for this
>is that in many cases the different containers have very different CPU requirements. For example, in the web server and data synchronizer Pod, the web server is user-facing and
>likely needs a great deal of CPU, while the data synchronizer can make do with very little.

### REQUEST LIMIT DETAILS

Requests are used when scheduling Pods to nodes. The Kubernetes scheduler will ensure that the sum of all requests of all Pods on a node does not exceed the capacity of the node. Therefore, a Pod is guaranteed to have at least the requested resources when running on the node. Importantly, “request” specifies a minimum. It does not specify a maximum cap on the resources a Pod may use. To explore what this means, let’s look at an example.

Imagine that we have container whose code attempts to use all available CPU cores. Suppose that we create a Pod with this container that requests 0.5 CPU. Kubernetes schedules this Pod onto a machine with a total of 2 CPU cores.

As long as it is the only Pod on the machine, it will consume all 2.0 of the available cores, despite only requesting 0.5 CPU.

If a second Pod with the same container and the same request of 0.5 CPU lands on the machine, then each Pod will receive 1.0 cores.

If a third identical Pod is scheduled, each Pod will receive 0.66 cores. Finally, if a fourth identical Pod is scheduled, each Pod will receive the 0.5 core it requested, and the node will be at capacity.

CPU requests are implemented using the cpu-shares functionality in the Linux kernel.

>**NOTE:**
>Memory requests are handled similarly to CPU, but there is an important difference. If a container is over its memory request, the OS can’t just remove memory from the process, 
>because it’s been allocated. Consequently, when the system runs out of memory, the kubelet terminates containers whose memory usage is greater than their requested memory. These 
>containers are automatically restarted, but with less available memory on the machine for the container to consume.

Since resource requests guarantee resource availability to a Pod, they are critical to ensuring that containers have sufficient resources in high-load situations.

### Capping Resource Usage with Limits
In addition to setting the resources required by a Pod, which establishes the minimum resources available to the Pod, you can also set a maximum on a Pod’s resource usage via resource limits.

In our previous example we created a kuard Pod that requested a minimum of 0.5 of a core and 128 MB of memory. In the Pod manifest in Example below, we extend this configuration to add a limit of 1.0 CPU and 256 MB of memory.

Example ``pod-limits.yaml``
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kuard
spec:
  containers:
    - image: gcr.io/kuar-demo/kuard-amd64:blue
      name: kuard
      resources:
        requests:
          cpu: "500m"
          memory: "128Mi"
        limits:
          cpu: "1000m"
          memory: "256Mi"
      ports:
        - containerPort: 8080
          name: http
          protocol: TCP
```
When you establish limits on a container, the kernel is configured to ensure that consumption cannot exceed these limits. A container with a CPU limit of 0.5 cores will only ever get 0.5 cores, even if the CPU is otherwise idle. A container with a memory limit of 256 MB will not be allowed additional memory (e.g., malloc will fail) if its memory usage exceeds 256 MB.



### Reference:

* https://medium.com/@Alibaba_Cloud/kubernetes-resource-quotas-f2161607444e
