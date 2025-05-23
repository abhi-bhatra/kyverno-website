---
title: "Protect Node Taints"
category: Other
version: 1.6.0
subject: Node
policyType: "validate"
description: >
    Node taints are often used as a control in multi-tenant use cases. If users can alter them, they may be able to affect scheduling of Pods which may impact other workloads. This sample prohibits altering of node taints unless by a user holding the `cluster-admin` ClusterRole. Use of this policy requires removal of the Node resource filter in the Kyverno ConfigMap ([Node,*,*]). Due to Kubernetes CVE-2021-25735, this policy requires, at minimum, one of the following versions of Kubernetes: v1.18.18, v1.19.10, v1.20.6, or v1.21.0.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/protect-node-taints/protect-node-taints.yaml" target="-blank">/other/protect-node-taints/protect-node-taints.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: protect-node-taints
  annotations:
    policies.kyverno.io/title: Protect Node Taints
    policies.kyverno.io/category: Other
    policies.kyverno.io/subject: Node
    policies.kyverno.io/minversion: 1.6.0
    policies.kyverno.io/description: >-
      Node taints are often used as a control in multi-tenant use cases.
      If users can alter them, they may be able to affect scheduling of
      Pods which may impact other workloads. This sample prohibits
      altering of node taints unless by a user holding the `cluster-admin`
      ClusterRole. Use of this policy requires removal of the Node resource filter
      in the Kyverno ConfigMap ([Node,*,*]). Due to Kubernetes CVE-2021-25735, this policy
      requires, at minimum, one of the following versions of Kubernetes:
      v1.18.18, v1.19.10, v1.20.6, or v1.21.0.
spec:
  validationFailureAction: Enforce
  background: false
  rules:
  - name: protect-node-taints
    match:
      any:
      - resources:
          kinds:
          - Node
    exclude:
      clusterRoles:
      - cluster-admin
    preconditions:
      all:
      - key: "{{request.operation || 'BACKGROUND'}}"
        operator: Equals
        value: UPDATE
    validate:
      message: "Node taints may not be altered."
      deny:
        conditions:
          any:
          - key: "{{request.object.spec.taints}}"
            operator: NotEquals
            value: ""
          - key: "{{request.oldObject.spec.taints}}"
            operator: NotEquals
            value: "{{request.object.spec.taints}}"

```
