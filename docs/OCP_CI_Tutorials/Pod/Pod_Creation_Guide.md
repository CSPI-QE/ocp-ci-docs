# OpenShift CI Pod Creation Guide<!-- omit from toc -->

- [Introduction](#introduction)
- [Creating Pod](#creating-pod)
- [Running Pod](#running-pod)
- [Pod Example](#pod-example)
  
## Introduction

Before we start, we want to make sure private test image is built and push into a private registry. The image includes installation of the product and test operators, test framework and other dependencies.

## Creating Pod

There are several important parameters we need to define on a pod spec:

1. Containers image: this is the quay.io registry image along with tag.

2. Containers command and args: this is the command you want to run inside the container.

3. Containers env: multiple environmental variables could be used in a pod. One of the them is KUBECONFIG, which defines where is cluster kubeconfig located.

4. volumeMount: defines the mount path for kubeconfig.

5. imagePullSecrets: defines the secret name to access private registry.

## Running Pod

A pod is started with a bash command by 'oc create -f ..'. It automatically pulls test images and runs the tests command with kubeconfig and environemtal variables configred. Once the tests finish, the artifacts are saved before the pod is deleted.

## Pod Example

Below is a pod we are using to run our tests:

`Pod`

```Pod
apiVersion: v1
kind: Pod
metadata:
  name: insights-pod
spec:
  restartPolicy: Never
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: insights-container
    image: quay.io/cloudservices/iqe-tests:cost-management
    command: ["/bin/sh", "-c"]
    args: ["iqe tests plugin cost_management -m cost_interop -vv --junitxml=test_run.xml && sleep 120"]
    env:
    - name: KUBECONFIG
      value: "/kubeconfig/kubeconfig"
    - name: DYNACONF_IBUTSU_URL
      value: https://ibutsu-api.apps.ocp4.prod.psi.redhat.com/
    - name: DYNACONF_IQE_VAULT_LOADER_ENABLED
      value: "true"
    - name: DYNACONF_IQE_VAULT_MOUNT_POINT
      valueFrom:
        secretKeyRef:
          key: mountPoint
          name: iqe-vault
    - name: DYNACONF_IQE_VAULT_URL
      valueFrom:
        secretKeyRef:
          key: url
          name: iqe-vault
    - name: DYNACONF_IQE_VAULT_ROLE_ID
      valueFrom:
        secretKeyRef:
          key: roleId
          name: iqe-vault
    - name: DYNACONF_IQE_VAULT_SECRET_ID
      valueFrom:
        secretKeyRef:
          key: secretId
          name: iqe-vault
    imagePullPolicy: Always
    resources:
      limits:
        cpu: 500m
        memory: 1000Mi
      requests:
        cpu: 500m
        memory: 500Mi
    securityContext:
        allowPrivilegeEscalation: false
        runAsNonRoot: true
        capabilities:
          drop:
            - ALL
    volumeMounts:
    - name: kubeconfig
      mountPath: /kubeconfig
  volumes:
  - name: kubeconfig
    secret:
      secretName: kubeconfig-secret
  imagePullSecrets:
  - name: cspi-pull-secret
```
