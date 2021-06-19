---
layout: default
title: "Kubernetes Cheatsheet"
date: 2021-02-21
categories: devops kubernetes
tags: devops docker k8s
---

## Terms

- [resource-type]: one of `deployment`, `pod`, `service` etc.

## Some k8s commands

- get all k8s info `kubectl get all`

- get all pods `kubectl get pods`

- get all pods in a namespace `kubectl get pods -n [namespace]`

- view details about a pod `kubectl describe pod [pod-name]`

- view details about a k8s resource `kubectl describe [resource-type] [resource-name]`

- view logs for a pod `kubectl logs [pod-name]`

- open a terminal in a running pod `kubectl exec --stdin --tty [pod-name] -n [namespace] -- /bin/bash`

- delete a resource `kubectl delete [resource-type] [deployment-name]`

## k8s local setup

- Using docker desktop, enable k8s. Wait for docker-desktop to restart.

- `kubectl` command now available.

- Run the following:

```sh
kubectl apply -f <https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml>
```

```sh
   kubectl describe secret -n kube-system
```

- Copy 1st token.

- Run the following:

```sh
   kubectl proxy
```

- Navigate to: <http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/overview?namespace=default>

[Home]( {{ site.url }})
