---
title: "Check deprecated APIs in CEL expressions"
category: Best Practices in CEL
version: 
subject: Kubernetes APIs
policyType: "validate"
description: >
    Kubernetes APIs are sometimes deprecated and removed after a few releases. As a best practice, older API versions should be replaced with newer versions. This policy validates for APIs that are deprecated or scheduled for removal. Note that checking for some of these resources may require modifying the Kyverno ConfigMap to remove filters. PodSecurityPolicy is removed in v1.25 so therefore the validate-v1-25-removals rule may not completely work on 1.25+.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//best-practices-cel/check-deprecated-apis/check-deprecated-apis.yaml" target="-blank">/best-practices-cel/check-deprecated-apis/check-deprecated-apis.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: check-deprecated-apis
  annotations:
    policies.kyverno.io/title: Check deprecated APIs in CEL expressions
    policies.kyverno.io/category: Best Practices in CEL 
    policies.kyverno.io/subject: Kubernetes APIs
    kyverno.io/kyverno-version: 1.12.1
    kyverno.io/kubernetes-version: "1.26-1.27"
    policies.kyverno.io/description: >-
      Kubernetes APIs are sometimes deprecated and removed after a few releases.
      As a best practice, older API versions should be replaced with newer versions.
      This policy validates for APIs that are deprecated or scheduled for removal.
      Note that checking for some of these resources may require modifying the Kyverno
      ConfigMap to remove filters. PodSecurityPolicy is removed in v1.25
      so therefore the validate-v1-25-removals rule may not completely work on 1.25+.
spec:
  validationFailureAction: Audit
  background: true
  rules:
  - name: validate-v1-25-removals
    match:
      any:
      - resources:
          # NOTE: PodSecurityPolicy is completely removed in 1.25.
          kinds:
          - batch/*/CronJob
          - discovery.k8s.io/*/EndpointSlice
          - events.k8s.io/*/Event
          - policy/*/PodDisruptionBudget
          - policy/*/PodSecurityPolicy
          - node.k8s.io/*/RuntimeClass
    celPreconditions:
      - name: "allowed-api-versions"
        expression: "object.apiVersion in ['batch/v1beta1', 'discovery.k8s.io/v1beta1', 'events.k8s.io/v1beta1', 'policy/v1beta1', 'node.k8s.io/v1beta1']"
    validate:
      cel:
        expressions:
          - expression: "false"
            messageExpression: >-
              object.apiVersion + '/' + object.kind + ' is deprecated and will be removed in v1.25.
              See: https://kubernetes.io/docs/reference/using-api/deprecation-guide/'
  - name: validate-v1-26-removals
    match:
      any:
      - resources:
          kinds:
          - flowcontrol.apiserver.k8s.io/*/FlowSchema
          - flowcontrol.apiserver.k8s.io/*/PriorityLevelConfiguration
          - autoscaling/*/HorizontalPodAutoscaler
    celPreconditions:
      - name: "allowed-api-versions"
        expression: "object.apiVersion in ['flowcontrol.apiserver.k8s.io/v1beta1', 'autoscaling/v2beta2']"
    validate:
      cel:
        expressions:
          - expression: "false"
            messageExpression: >-
              object.apiVersion + '/' + object.kind + ' is deprecated and will be removed in v1.26.
              See: https://kubernetes.io/docs/reference/using-api/deprecation-guide/'
  - name: validate-v1-27-removals
    match:
      any:
      - resources:
          kinds:
          - storage.k8s.io/*/CSIStorageCapacity
    celPreconditions:
      - name: "allowed-api-versions"
        expression: "object.apiVersion in ['storage.k8s.io/v1beta1']"
    validate:
      cel:
        expressions:
          - expression: "false"
            messageExpression: >-
              object.apiVersion + '/' + object.kind + ' is deprecated and will be removed in v1.27.
              See: https://kubernetes.io/docs/reference/using-api/deprecation-guide/'
  - name: validate-v1-29-removals
    match:
      any:
      - resources:
          kinds:
          - flowcontrol.apiserver.k8s.io/*/FlowSchema
          - flowcontrol.apiserver.k8s.io/*/PriorityLevelConfiguration
    celPreconditions:
      - name: "object.apiVersion"
        expression: "object.apiVersion in ['flowcontrol.apiserver.k8s.io/v1beta2']"
    validate:
      cel:
        expressions:
          - expression: "false"
            messageExpression: >-
              object.apiVersion + '/' + object.kind + ' is deprecated and will be removed in v1.29.
              See: https://kubernetes.io/docs/reference/using-api/deprecation-guide/'
  

```
