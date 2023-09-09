---
title: How to deploy a Kubernate Job by client-go in cluster
date: 2023-09-07 23:47:47
categories: Kubernate
tags:
- Kubernate
- ServiceAccount
---

# **Kubernets API Server**

The API server is a component of the Kubernetes control plane that exposes the Kubernetes API via HTTPS. It is the main management point of the entire cluster. To invoke this API, the client requests needs implement the correct auth(authentication and authorization). API server is also responsible for the authentication and authorization mechanism, processes REST operations, validates them, and updates the corresponding objects in Etcd. There are two main authentication mechanisms we can use handle the API Server auth, TLS/SSL Certificate-based auth(X509) and Token-based auth.

# **Role-Based Access Control**

In the Kubernetes cluster , the mechanism responsible for completing the authorization work is RBAC (Role-Based Access Control). RBAC defines policies for restricting and controlling user access to resources of a system by using roles attached to users. However, RBAC policies can also govern the behavior of software resources. A *service account* provides an identity for processes that run in a Pod, and maps to a ServiceAccount object. A secret with access token is generated when ServiceAccount object is deployed. After specify `serviceAccountName` with in a pod, the secret will mount into pod at `/var/run/secrets/kubernetes.io/serviceaccount` . The pod can access API Server with permissions defined in `serviceAccountName` .

RBAC authorizations can define permissions/policies that can be limited within a namespace(Role) or the entire cluster(ClusterRule).

<img src="./1.png" width="100%" height="100%" alt="1">

# **ServiceAccountToken**

Kubelet automatically mount a default ServiceAccount’s API credentials (ServiceAccountToken) in the same namespace as the pod is create without specifying a ServiceAccount. This allow pod to access API server incluster by ServiceAccountToken. You can opt out of automounting API credentials on `/var/run/secrets/kubernetes.io/serviceaccount/token` for a service account by setting `automountServiceAccountToken: false` on the ServiceAccount.

serviceAccount, Role, RoleBinding

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: service-account-jobs
  namespace: default
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: job-operation
  namespace: default
rules:
  - apiGroups: ["", "batch"]
    resources: ["jobs"]
    verbs: ["get", "list", "watch", "create", "delete"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: job-operation
  namespace: default
subjects:
  - kind: ServiceAccount
    name: service-account-jobs
roleRef:
  kind: Role
  name: job-operation
  apiGroup: rbac.authorization.k8s.io
```

deployment.yaml (program running in pod which want to access Kubernate API Server within cluster to deploy a Job)

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: program
spec:
  selector:
    matchLabels:
      app: program
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: program
    spec:
      serviceAccountName: service-account-jobs
      containers:
      - name: program
        image: {{ .Values.program.image }}
        command:
          - "bin/sh"
          - "-c"
          - "/go/bin/server -sql=/etc/sql/postgres.txt -worker-job=/etc/jobs/worker-job.yaml"
        ports:
        - containerPort: 8081
        volumeMounts:
          - name: postgre-sql
            mountPath: /etc/sql
          - name: worker-job
            mountPath: /etc/jobs
      volumes:
        - name: postgre-sql
          configMap:
            name: configs
            items:
              - key: postgres.txt
                path: postgres.txt
        - name: worker-job
          configMap:
            name: configs
            items:
              - key: worker-job.yaml
                path: worker-job.yaml
```

configmap.yaml (render by helm, place to store Job template)

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: configs
data:
  {{- .Files.Get .Values.configmapStrings | nindent 2 }}
  {{- (.Files.Glob .Values.configmapFiles).AsConfig | nindent 2 }}
  {{- (.Files.Glob .Values.configmapJobsFiles).AsConfig | nindent 2 }}
```

local-values.yaml (render by helm)

```
configmapFiles: "configmaps/files/local/**"
configmapStrings: "configmaps/strings/local.yaml"
configmapJobsFiles: "configmaps/jobs/local/**"
```

Job template

```
apiVersion: batch/v1
kind: Job
metadata:
  name: worker
  namespace: default
  labels:
    app: worker
spec:
  ttlSecondsAfterFinished: 100
  template:
    spec:
      containers:
      - name: worker
        image: worker:target
        imagePullPolicy: Never
        command:
          - "bin/sh"
          - "-c"
          - "/go/bin/image_worker -sql=/etc/sql/postgres.txt -custom=${CUSTOM_ID}"
        volumeMounts:
          - name: postgre-sql
            mountPath: /etc/sql
      restartPolicy: Never
      volumes:
        - name: postgre-sql
          configMap:
            name: configs
            items:
              - key: postgres.txt
                path: postgres.txt
  backoffLimit: 0
```

Golang program read in Job template YAML as an parameter by golang flag and parse YAML to client-go Job spec. You can then customize Job setting. Beware that the InClusterConfig only work when running program within cluster.

```
package k8sutil

import (
 "context"
 "fmt"
 "os"
 "strings"

 batchv1 "k8s.io/api/batch/v1"
 metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
 "k8s.io/client-go/kubernetes"
 "k8s.io/client-go/kubernetes/scheme"
 "k8s.io/client-go/rest"
)

func connectToK8s() (*kubernetes.Clientset, error) {
 // creates the in-cluster config
 config, err := rest.InClusterConfig()
 if err != nil {
  return nil, fmt.Errorf("failed to create in-cluster cofig: %v", err)
 }

 // creates the clientset
 clientset, err := kubernetes.NewForConfig(config)
 if err != nil {
  return nil, fmt.Errorf("failed to create k8s client set: %v", err)
 }

 return clientset, nil
}

func resolveJobTemplate(imageWorkerJobYamlPath string) (*batchv1.Job, error) {
 // read in k8s yaml file
 b, err := os.ReadFile(imageWorkerJobYamlPath)
 if err != nil {
  return nil, fmt.Errorf("failed to read job yaml file: %v", err)
 }

 // deserialize k8s yaml to k8s object
 decode := scheme.Codecs.UniversalDeserializer().Decode
 obj, groupVersionKind, err := decode([]byte(b), nil, nil)
 if err != nil {
  return nil, fmt.Errorf("failed to decode yaml file to object: %v", err)
 }

 var job *batchv1.Job
 if groupVersionKind.Kind == "Job" {
  job = obj.(*batchv1.Job)
 }

 return job, nil
}

// LaunchImageWorkerK8sJob luach a worker job with specific job name and customID
func LaunchImageWorkerK8sJob(ctx context.Context, jobName string, customID string, imageWorkerJobYamlPath string) error {
 // resolve kubernate job template
 jobSpec, err := resolveJobTemplate(imageWorkerJobYamlPath)
 if err != nil {
  return fmt.Errorf("failed to resolve kubernate job template: %v", err)
 }

 // connect to k8s
 clientset, err := connectToK8s()
 if err != nil {
  return fmt.Errorf("failed to connect to kubernate: %v", err)
 }

 // customize Job setting
 // job name setting
 jobSpec.ObjectMeta.Name = jobName
 // replace customiD flag with target customID
 commandWithFlag := jobSpec.Spec.Template.Spec.Containers[0].Command[2]
 replaceCustomIDFlag := strings.Replace(commandWithFlag, "${CUSTOM_ID}", customID, 1)
 jobSpec.Spec.Template.Spec.Containers[0].Command[2] = replaceCustomIDFlag

 namespace := jobSpec.Namespace

 // apply k8s job
 jobs := clientset.BatchV1().Jobs(namespace)
 jobDetail, err := jobs.Create(ctx, jobSpec, metav1.CreateOptions{})
 if err != nil {
  return fmt.Errorf("failed to create kubernete job: %v \n Job detail: %v", err, jobDetail)
 }

 return nil
}
```

**Reference**

- [Creating a Job with Kubernetes client-go](https://stackoverflow.com/questions/50510381/creating-a-job-with-kubernetes-client-go)
- [client-go/examples/in-cluster-client-configuration/main.go](https://github.com/kubernetes/client-go/blob/master/examples/in-cluster-client-configuration/main.go)
- [How to run kubectl within a job in a namespace?](https://stackoverflow.com/questions/60885448/how-to-run-kubectl-within-a-job-in-a-namespace)
- [Support for parsing K8s yaml spec into client-go data structures #193](https://github.com/kubernetes/client-go/issues/193)
- [Authenticating](https://kubernetes.io/docs/reference/access-authn-authz/authentication/)
- [How to enter an optional flag with no parameters in Go CLI program](https://stackoverflow.com/questions/34388593/how-to-enter-an-optional-flag-with-no-parameters-in-go-cli-program)
- [Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
- [Configure Service Accounts for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)
- [Kubernetes HTTP API with Authentication and Authorization](https://medium.com/rahasak/kubernetes-http-api-with-authentication-and-authorization-c0cb7051114f)
- [Kubelet authentication/authorization](https://kubernetes.io/docs/reference/access-authn-authz/kubelet-authn-authz/)
- [How To Call Kubernetes API using Simple HTTP Client](https://iximiuz.com/en/posts/kubernetes-api-call-simple-http-client/)