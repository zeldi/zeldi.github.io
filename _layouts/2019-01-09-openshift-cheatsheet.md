---
layout: post
title: Openshift Cheetsheet
permalink: openshift-cheatsheet
date: 2019-01-09
---


At the moment I am working on OpenShift, here I am writing stuff that is exciting and I keep forgetting. The list has (almost) no order and is very random. By the way: there is also an official cheat sheet

### Basic Openshift Commands

**Create and switch project**

```
oc new-project thespark
oc projects
oc project thespark
oc delete project thespark
```

__Expose applications__
```
oc expose dc/hallogo --port=8080
oc sd
```
