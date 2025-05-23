---
title: "Require Pod priorityClassName in CEL expressions"
category: Multi-Tenancy, EKS Best Practices in CEL
version: 
subject: Pod
policyType: "validate"
description: >
    A Pod may optionally specify a priorityClassName which indicates the scheduling priority relative to others. This requires creation of a PriorityClass object in advance. With this created, a Pod may set this field to that value. In a multi-tenant environment, it is often desired to require this priorityClassName be set to make certain tenant scheduling guarantees. This policy requires that a Pod defines the priorityClassName field with some value.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other-cel/require-pod-priorityclassname/require-pod-priorityclassname.yaml" target="-blank">/other-cel/require-pod-priorityclassname/require-pod-priorityclassname.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-pod-priorityclassname
  annotations:
    policies.kyverno.io/title: Require Pod priorityClassName in CEL expressions
    policies.kyverno.io/category: Multi-Tenancy, EKS Best Practices in CEL 
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Pod
    kyverno.io/kyverno-version: 1.11.0
    kyverno.io/kubernetes-version: "1.26-1.27"
    policies.kyverno.io/description: >-
      A Pod may optionally specify a priorityClassName which indicates the scheduling
      priority relative to others. This requires creation of a PriorityClass object in advance.
      With this created, a Pod may set this field to that value. In a multi-tenant environment,
      it is often desired to require this priorityClassName be set to make certain tenant
      scheduling guarantees. This policy requires that a Pod defines the priorityClassName field
      with some value.
spec:
  validationFailureAction: Audit
  background: true
  rules:
  - name: check-priorityclassname
    match:
      any:
      - resources:
          kinds:
          - Pod
          operations:
          - CREATE
          - UPDATE
    validate:
      cel:
        expressions:
          - expression: "object.spec.?priorityClassName.orValue('') != ''"
            message: "Pods must define the priorityClassName field."


```
