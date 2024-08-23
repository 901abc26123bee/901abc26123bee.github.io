---
title: Kubernetes Ingress and Services troubleshooting
date: 2023-09-10 00:52:24
categories: Kubernetes
tags: Kubernetes, debug
---

# Kubernetes Ingress and Services troubleshooting

- Traffic flow: Internet → Ingress controller rule (according to your Ingress YAML) → Service → Pods

- Debug flow: Pods → Service → Ingress → Ingress controller → Internet

## Check the Deployment & Pods
1. Makes sure that the Pod is up and running (Pod’s “Status” is “Running”). If not, checks the Deployment/Pod Resource Events and log to fix the problem.
2. If you are using HTTP GET livenessProbe, ensure your Service and Ingress are deployed beforehand.

```sh
$ kubectl get deployment -n <namespace>
# logs of deployment
$ kubectl logs deployment/<name-of-deployment> -n <namespace>
# follow logs of deployment, ctrl+c to quit
$ kubectl logs -f deployment/<name-of-deployment> -n <namespace>
# check "Events" at bottom of the output
$ kubectl describe deployment <name-of-deployment> -n <namespace>

$ kubectl get pods -n <namespace>
$ kubectl logs pod/<name-of-pod> -n <namespace>
$ kubectl describe pod <name-of-pod> -o wide -n <namespace>
```

## Check the port mapping
<img src="./1.png" width="80%" height="80%" alt="1">

> [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
    - An Ingress may be configured to give Services externally-reachable URLs, load balance traffic, terminate SSL / TLS, and offer name-based virtual hosting. An Ingress controller is responsible for fulfilling the Ingress, usually with a load balancer, though it may also configure your edge router or additional frontends to help handle the traffic.
    - An Ingress does not expose arbitrary ports or protocols. Exposing services other than HTTP and HTTPS to the internet typically uses a service of type Service.Type=NodePort or Service.Type=LoadBalancer.

## Check the Service
1. Checks the “Endpoints” field in Service, which should match the “IP” of Pods.
2. If you are using public cloud like GKE, AKS…, you can modify the Service Types to LoadBalancer to expose your Service publicly without ingress. If this work for you, meaning that your Pods and Service work correctly, problems are cause by others.
```sh
$ kubectl get service -n <namespace>
$ kubectl describe service <name-of-pod> -n <namespace>
```

## Check the Ingress & Ingress Controller Logs and Resource Events
> [Ingress Controllers](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)
    - In order for the Ingress resource to work, the cluster must have an ingress controller running.
    - Unlike other types of controllers which run as part of the kube-controller-manager binary, Ingress controllers are not started automatically with a cluster in public cloud like GKE, AKS…. You need to choose the ingress controller implementation that best fits your cluster.
    - Ingress Controller in AKS is AKS Application Gateway Ingress Controller ,which is ingress-appgw-deployment below. The ingress controller runs as a pod within the AKS cluster. It consumes Kubernetes Ingress Resources and converts them to an Azure Application Gateway configuration which allows the gateway to load-balance traffic to Kubernetes pods.

1. Checks your Ingress for any Event or error log
    ```sh
    $ kubectl get ingress -n <namespace>
    $ kubectl describe ingress <name-of-ingress> -n <namespace>
    ```
2. Checks your ingress controller configuration to see if it’s rule match the ingress you just apply.
    ```sh
    λ kubectl get deployment  -n kube-system
    NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
    ingress-appgw-deployment   1/1     1            1           48d
    ...
    $ kubectl get pods -n kube-system
    NAME                                        READY   STATUS    RESTARTS   AGE
    azure-cni-networkmonitor-2fmfk              1/1     Running   0          7d5h
    ...
    azure-ip-masq-agent-6k4rm                   1/1     Running   0          3d5h
    ...
    coredns-84d976c568-pjt8q                    1/1     Running   1          86d
    ...
    ingress-appgw-deployment-7b8b687b46-scvs7   1/1     Running   315        3d5h
    kube-proxy-4c6qw                            1/1     Running   0          7d5h
    ...
    metrics-server-569f6547dd-j2wjz             1/1     Running   5          86d
    ...
    $ kubectl describe pod ingress-appgw-deployment-7b8b687b46-scvs7 -n kube-system
    ```

    Finally figure out that the problem is caused by Ingress controller fail to convert Ingress YAML to Azure Application Gateway configuration while you update your Ingress YAML or Pod Service endpoint, including livenessProbe.

    ```sh
    $ kubectl logs -f -n kube-system ingress-appgw-deployment-7b8b687b46-scvs7 | grep --color=always -i error
    E0110 03:19:02.123462       1 requestroutingrules.go:386] A path-rule with path '/merchant/*' already exists in config for BackendPool '/subscriptions/49fc9d19-f517-4ca5-a93e-76ed0fbd0ab1/resourceGroups/xxx/providers/Microsoft.Network/applicationGateways/aks-gw/backendAddressPools/defaultaddresspool'. Duplicate path-rule with BackendPool '/subscriptions/49fc9d19-f517-4ca5-a93e-76ed0fbd0ab1/resourceGroups/xxx/providers/Microsoft.Network/applicationGateways/aks-gw/backendAddressPools/pool-payment-payment-svc-80-bp-80' will not be applied
    ```

3. Restart Ingress Controller
    ```sh
    $ kubectl rollout restart deployment <your-ingress-controller-deployment>
    ```

> For AKS Application Gateway, check:
    - [Application Gateway health monitoring overview](https://learn.microsoft.com/en-us/azure/application-gateway/application-gateway-probe-overview)
    - [Create a custom probe for Application Gateway by using the portal](https://learn.microsoft.com/en-us/azure/application-gateway/application-gateway-create-probe-portal)

## Reason for Application Gateway failure
The reasons that may cause Application Gateway fail to monitor and apply Ingress configuration.

1. Using HTTPS without a Secret that contains a TLS private key and certificate. If you are using let's encrypt to automatically produce secret file with tls.crt and tls.key with kubernates.io/tls type, make sure it hasn’t block by your your AKS firewall, which will result in wrong type of Secret(Opaque).

    ```yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: testsecret-tls
      namespace: default
    data:
      tls.crt: base64 encoded cert
      tls.key: base64 encoded key
    type: kubernetes.io/tls
    ```

2. redirecting problem

    [Ingress gives 502 error](https://stackoverflow.com/questions/42101808/ingress-gives-502-error)

    noticing that the application must return a 200 status code at ‘/’. if your application was returning a 302 (redirect to login), which would cause the health fail. When the health check fails, the ingress resource returns 502.

3. Backend path rule conflict.

    If you specify the backend-path-prefix in Ingress, make sure it did not conflict your backend resource Deployment livenessProbe path prefix.

    Deployment
    ```yaml
    ...
            livenessProbe:
              httpGet:
                path: /payment_resource/healthcheck.jsp
                port: 8080
              initialDelaySeconds: 180
              periodSeconds: 10
              timeoutSeconds: 3
              failureThreshold: 3
              successThreshold: 1
    ```

    Ingress
    ```yaml
    kind: Ingress
    apiVersion: networking.k8s.io/v1
    metadata:
      name: ingress-payment
      namespace: payment
      annotations:
        appgw.ingress.kubernetes.io/backend-path-prefix: /payment_resource/
        appgw.ingress.kubernetes.io/connection-draining: 'true'
        appgw.ingress.kubernetes.io/connection-draining-timeout: '30'
        appgw.ingress.kubernetes.io/cookie-based-affinity: 'true'
        appgw.ingress.kubernetes.io/ssl-redirect: 'true'
        cert-manager.io/cluster-issuer: letsencrypt-production
        kubernetes.io/ingress.allow-http: 'false'
        kubernetes.io/ingress.class: azure/application-gateway
    spec:
      tls:
        - hosts:
            - xxx.xxx.xxx.azure.com
          secretName: aks-ingress-cert
      rules:
        - host: xxx.xxx.xxx.azure.com
          http:
            paths:
              - path: /payment/*
                pathType: ImplementationSpecific
                backend:
                  service:
                    name: payment-svc
                    port:
                      number: 80
    ```
    Visit https://xxx.xxx.xxx.azure.com/payment/healthcheck.jsp , which would redirect to backend resource endpoint https://xxx.xxx.xxx.azure.com/payment_resource/healthcheck.jsp .

> [Backend Path Prefix](https://azure.github.io/application-gateway-kubernetes-ingress/annotations/)
This annotation allows the backend path specified in an ingress resource to be re-written with prefix specified in this annotation. This allows users to expose services whose endpoints are different than endpoint names used to expose a service in an ingress resource.

[Ingress — Context Path Based Routing](https://stacksimplify.com/azure-aks/azure-kubernetes-service-ingress-context-path-routing/)

<img src="./2.png" width="80%" height="80%" alt="2">