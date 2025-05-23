---
title: "Disallow hostPath in CEL expressions"
category: Pod Security Standards (Baseline) in CEL
version: 1.11.0
subject: Pod,Volume
policyType: "validate"
description: >
    HostPath volumes let Pods use host directories and volumes in containers. Using host resources can be used to access shared data or escalate privileges and should not be allowed. This policy ensures no hostPath volumes are in use.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//pod-security-cel/baseline/disallow-host-path/disallow-host-path.yaml" target="-blank">/pod-security-cel/baseline/disallow-host-path/disallow-host-path.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: disallow-host-path
  annotations:
    policies.kyverno.io/title: Disallow hostPath in CEL expressions
    policies.kyverno.io/category: Pod Security Standards (Baseline) in CEL
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Pod,Volume
    policies.kyverno.io/minversion: 1.11.0
    kyverno.io/kubernetes-version: "1.26-1.27"
    policies.kyverno.io/description: >-
      HostPath volumes let Pods use host directories and volumes in containers.
      Using host resources can be used to access shared data or escalate privileges
      and should not be allowed. This policy ensures no hostPath volumes are in use.
spec:
  validationFailureAction: Audit
  background: true
  rules:
    - name: host-path
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
            - expression: "object.spec.?volumes.orValue([]).all(volume, size(volume) == 0 || !has(volume.hostPath))"
              message: "HostPath volumes are forbidden. The field spec.volumes[*].hostPath must be unset"

```
