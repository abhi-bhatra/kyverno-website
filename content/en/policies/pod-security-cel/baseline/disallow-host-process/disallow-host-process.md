---
title: "Disallow hostProcess in CEL expressions"
category: Pod Security Standards (Baseline) in CEL
version: 1.11.0
subject: Pod
policyType: "validate"
description: >
    Windows pods offer the ability to run HostProcess containers which enables privileged access to the Windows node. Privileged access to the host is disallowed in the baseline policy. HostProcess pods are an alpha feature as of Kubernetes v1.22. This policy ensures the `hostProcess` field, if present, is set to `false`.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//pod-security-cel/baseline/disallow-host-process/disallow-host-process.yaml" target="-blank">/pod-security-cel/baseline/disallow-host-process/disallow-host-process.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: disallow-host-process
  annotations:
    policies.kyverno.io/title: Disallow hostProcess in CEL expressions
    policies.kyverno.io/category: Pod Security Standards (Baseline) in CEL
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/minversion: 1.11.0
    kyverno.io/kubernetes-version: "1.26-1.27"
    policies.kyverno.io/description: >-
      Windows pods offer the ability to run HostProcess containers which enables privileged
      access to the Windows node. Privileged access to the host is disallowed in the baseline
      policy. HostProcess pods are an alpha feature as of Kubernetes v1.22. This policy ensures
      the `hostProcess` field, if present, is set to `false`.
spec:
  validationFailureAction: Audit
  background: true
  rules:
    - name: host-process-containers
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
          variables:
            - name: allContainers
              expression: "(object.spec.containers + (has(object.spec.initContainers) ? object.spec.initContainers : []) + (has(object.spec.ephemeralContainers) ? object.spec.ephemeralContainers : []))"
          expressions:
            - expression: >-
                variables.allContainers.all(container,
                container.?securityContext.?windowsOptions.?hostProcess.orValue(false) == false)
              message: >-
                HostProcess containers are disallowed. The field spec.containers[*].securityContext.windowsOptions.hostProcess,
                spec.initContainers[*].securityContext.windowsOptions.hostProcess, and
                spec.ephemeralContainers[*].securityContext.windowsOptions.hostProcess
                must either be undefined or set to `false`.    

```
