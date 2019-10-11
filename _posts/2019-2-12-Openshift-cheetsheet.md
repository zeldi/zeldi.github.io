---
layout: post
title: Openshift Cheetsheet
permalink: openshift-cheatsheet
date: 2019-01-09
---

At the moment I am working on OpenShift, here I am writing stuff that is exciting and I keep forgetting. The list has (almost) no order and is very random. 
By the way: there is also an official cheat sheet

### Basic Openshift Commands

**Create and switch project**

    oc new-project thespark
    oc projects
    oc project thespark
    oc delete project thespark

__Expose applications__

    oc expose dc/hallogo --port=8080 (generates a service from deployment configuration)
    oc expose svc/hallogo (generares a route from service)

__Describe etc. resource types, print to yaml__

    oc describe bc
    oc describe dc
    oc get secret gitlab-hallospark -o yaml
    oc get user
    oc get nodes

__Templates__

    oc get templates -n openshift 
    oc describe template postgresql-persistent -n openshift
    oc get template jenkins-pipeline-example -n openshift -o yaml
    oc export all --as-template=javapg

__Image streams__

update an external image stream with the newest version: (e.g. iceScrum):

    oc get is
    oc import-image icescrum

__Collect all logs of a namespace:__

    #!/bin/bash

    namespace='openshift-monitoring'
    oc project $namespace
    mkdir -p $namespace-logs/pod

    for pod in $(oc get pods -o name -n $namespace); do 
        echo "$pod"
        echo "======$pod (tailed to 2000 lines=====" >> $namespace-logs/$pod-logs.txt
        oc logs $pod --all-containers --tail 2000 >> $namespace-logs/$pod-logs.txt 
    done

    tar -cvf $namespace-logs.tar $namespace-logs/



### Some Admin’s Day-2 Commands

__Garbage Collection__

Get rid of evicted pods:

    oc get pod --all-namespaces  | grep Evicted
    oc get pod --all-namespaces  | awk '{if ($4=="Evicted") print "oc delete pod " $2 " -n " $1;}' | sh 
    get rid of garbage by pruning: (as a cluster admin on a master node, so that registry is accessible)

    oc login -u admin

    oc adm prune builds --confirm 
    oc adm prune images --confirm 

List, describe and delete all resources with a label:

    oc get all --selector app=hallospark -o yaml
    oc describe all --selector app=hallospark
    oc delete all --selector app=hallospark 
    oc delete pvc --all 

e.g. delete (and restart) all Prometheus node-exporter and prometheus logs pods:

    oc project openshift-monitoring
    oc get -o name pods --selector app=node-exporter
    oc delete pods --selector app=node-exporter
    oc delete pods --selector app=prometheus

Look for Pods with DiskPressure

    oc describe node  | grep -i NodeHasDiskPressure

Drain and reboot node:

    oc adm manage-node ocp-app-1 --schedulable=false
    oc adm drain ocp-app-1
    systemctl restart atomic-openshift-node.service

__Adding NFS Provisioner__

In okd documentation, dynamic persistence volume provisioning for nfs is not supported. So we have to use following incubator

Clone the repo and go to the folder above.

    $ oc new-project nfs-provisioner

then replace the namespace with following code snippets:

    # Set the subject of the RBAC objects to the current namespace where the provisioner is being deployed
    $ NAMESPACE=`oc project -q`
    $ sed -i'' "s/namespace:.*/namespace: $NAMESPACE/g" ./deploy/rbac.yaml
    $ oc create -f deploy/rbac.yaml
    $ oadm policy add-scc-to-user hostmount-anyuid system:serviceaccount:$NAMESPACE:nfs-client-provisioner

in deployment.yaml, replace the values accordingly, e.g.:

    server: 10.0.0.14
    path: /var/nfs/general
    then apply the files.

    oc apply -f 
    - rbac.yaml
    - class.yaml
    - deployment.yaml 
    - 
the provisioner will run as a normal pod and create pv when they are claimed, including making directories on the server and deleting them.

To make the new storage class the default one, patch it with:

    $kubectl patch storageclass managed-nfs-storage -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'