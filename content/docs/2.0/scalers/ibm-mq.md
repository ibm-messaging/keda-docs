+++
title = "IBM MQ"
layout = "scaler"
availability = "v2.0+"
maintainer = "Community"
description = "Scale applications based on IBM MQ Queue"
go_file = "ibmmq_scaler"
+++

### Trigger Specification

This specification describes the `ibmmq` trigger for IBM MQ Queue.

```yaml
triggers:
    - type: ibmmq
      metadata:
        queueLength: '5' # Queue length target for HPA. Default: 5 messages
        host: IBMMQ_HOST # IBM MQ Queue Manager Admin REST Endpoint
        queueName: QUEUE_NAME
      authenticationRef:
        name: ibmmq-consumer-trigger
```

**Parameter list:**

- `queueLength`: OPTIONAL - Queue Length Target for HPA. Will be set to Default Value of 5 if not Provided.
- `host`: REQUIRED - IBM MQ Queue Manager Admin REST Endpoint. Example URI endpoint structure on IBM cloud `https://example.mq.appdomain.cloud/ibmmq/rest/v2/admin/action/qmgr/QM/mqsc`
- `queueName`: REQUIRED - Name of the Queue within the Queue Manager defined from which messages will be consumed

### Authentication Parameters

TriggerAuthentication CRD is used to connect and authenticate to IBM MQ:

### Example

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: keda-ibmmq-secret
data:
  ADMIN_USER: <encoded-username>
  ADMIN_PASSWORD: <encoded-password>
---
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: ibmmq-scaledobject
  namespace: default
  labels:
    deploymentName: ibmmq-deployment
spec:
  scaleTargetRef:
    name: ibmmq-deployment
  pollingInterval: 5 # OPTIONAL - Default: 30 seconds
  cooldownPeriod: 30 # OPTIONAL - Default: 300 seconds
  maxReplicaCount: 18 # OPTIONAL - Default: 100
  triggers:
    - type: ibmmq
      metadata:
        queueLength: '5'
        host: IBMMQ_HOST
        queueName: QUEUE_NAME
      authenticationRef:
        name: keda-ibmmq-trigger-auth
---
apiVersion: keda.sh/v1alpha1
kind: TriggerAuthentication
metadata:
  name: keda-ibmmq-trigger-auth
  namespace: default
spec:
  secretTargetRef:
    - parameter: username
      name: keda-ibmmq-secret
      key: ADMIN_USER
    - parameter: password
      name: keda-ibmmq-secret
      key: ADMIN_PASSWORD
```