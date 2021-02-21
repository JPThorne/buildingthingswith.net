---
layout: default
title: "Kubernetes Cheatsheet"
date: 2021-02-21
categories: devops kubernetes
tags: devops docker k8s
---

## Some k8s commands

- get all pods `kubectl get pods`

- get all pods in a namespace `kubectl get pods -n [namespace]`

- view details about a pod `kubectl describe pod [pod-name]`

- view details about a k8s resource `kubectl describe [resource-type] [resource-name]`

- view logs for a pod `kubectl logs [pod-name]`

- open a terminal in a running pod `kubectl exec --stdin --tty [pod-name] -n [namespace] -- /bin/bash`

[Home]( {{ site.url }})