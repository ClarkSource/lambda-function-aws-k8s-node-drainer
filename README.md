<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Amazon EKS Node Drainer](#amazon-eks-node-drainer)
  - [Unit Tests](#unit-tests)
- [Appendix](#appendix)
  - [Limitations](#limitations)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Amazon EKS Node Drainer

Gracefully terminate nodes of an Amazon Elastic Container Service for Kubernetes
(Amazon EKS) cluster when managed as part of an Amazon EC2 Auto Scaling Group.

The code provides an AWS Lambda function that integrates as an [Amazon EC2 Auto
Scaling Lifecycle Hook](https://docs.aws.amazon.com/autoscaling/ec2/userguide/lifecycle-hooks.html).
When called, the Lambda function calls the Kubernetes API to cordon and evict all evictable pods from the node being
terminated. It will then wait until all pods have been evicted before the Auto Scaling group continues to terminate the
EC2 instance. The lambda may be killed by the function timeout before all evictions complete successfully, in which case
the lifecycle hook may re-execute the lambda to try again. If the lifecycle heartbeat expires then termination of the EC2
instance will continue regardless of whether or not draining was successful.

Using this approach can minimise disruption to the services running in EKS cluster by allowing Kubernetes to
reschedule the pod prior to the instance being terminated enters the TERMINATING state. It works by using
[Amazon EC2 Auto Scaling Lifecycle Hooks](https://docs.aws.amazon.com/autoscaling/ec2/userguide/lifecycle-hooks.html)
to trigger an AWS Lambda function that uses the Kubernetes API to cordon the node and evict the pods.

## Unit Tests

To run the unit tests, install the test dependencies and run `pytest` against the `tests` directory:

```bash
pipenv install --dev --ignore-pipfile
pipenv run py.test --cov=drainer
```

# Appendix

## Limitations

This solution works on a per cluster per autoscaling group basis, multiple autoscaling groups will require a separate
deployment for each group.

Certain types of pod cannot be evicted from a node, so this lambda will not attempt to evict DaemonSets or mirror pods.
