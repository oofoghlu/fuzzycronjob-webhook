# fuzzycronjob-webhook
Boost your Kubernetes CronJobs with Jenkins-style hashed cron expressions.

fuzzycronjob-webhook provides a MutatingWebhookConfiguration along with a Deployment to handle the mutation,
effectively extending CronJobs to handle hashes in cron expressions.

The mutating webhook looks for a fuzzy-schedule annotation and, if present, evaluates the hash expression, mutates the
CronJob spec's schedule with the evaluated expression. These hashes allow for more uniform distribution of your workloads
across time. Like with Jenkins, the hashes are deterministic on a per-job basis, meaning changes to the Spec
won't affect your workloads already running.

## Description

A CronJob simply needs to specify the `github.com.oofoghlu/fuzzy-schedule` annotation.

Sample manifest:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
    annotations:
        github.com.oofoghlu/fuzzy-schedule: "H H * * *"
    name: fuzzycronjob-sample
spec:
    startingDeadlineSeconds: 60
    concurrencyPolicy: Allow
    jobTemplate:
        spec:
        template:
            spec:
            containers:
            - name: hello
                image: busybox
                args:
                - /bin/sh
                - -c
                - date; echo Hello from the Kubernetes cluster
            restartPolicy: OnFailure
```

Will be evaluated as:

```
$ kubectl get cj
NAME                  SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
fuzzycronjob-sample   46 21 * * *   False     0        119m            122m
```

The hash is based on `<namespace>-<name>` allowing for crons with the same name to be scheduled differently across
namespaces, or more simply for all crons that need to run on the same cadence are spread out. The hash is evaluated
differently per field index in the cron expression ensuring spread from field to field when evaluated for the same
range.

Some sample expressions supported (along with example evaluations):

```
H H H H H
-> 20 19 11 6 4

H H * H/2 *
-> 18 17 * 1/2 *

H(5-15)/20 H/5 * * *
-> 12/20 3/5 * * *

H(0-5) * * * *
-> 4 * * * *
```

For more info on cron hashes see: https://github.com/oofoghlu/fuzzycrontab

## Getting Started

### Prerequisites
- go version v1.20.0+
- docker version 17.03+.
- kubectl version v1.11.3+.
- Access to a Kubernetes v1.11.3+ cluster.

### To Deploy on the cluster
**Build and push your image to the location specified by `IMG`:**

```sh
make docker-build docker-push IMG=<some-registry>/fuzzycronjob-webhook:tag
```

**NOTE:** This image ought to be published in the personal registry you specified. 
And it is required to have access to pull the image from the working environment. 
Make sure you have the proper permission to the registry if the above commands don’t work.

**Install the CRDs into the cluster:**

```sh
make install
```

**Deploy the Manager to the cluster with the image specified by `IMG`:**

```sh
make deploy IMG=<some-registry>/fuzzycronjob-webhook:tag
```

> **NOTE**: If you encounter RBAC errors, you may need to grant yourself cluster-admin 
privileges or be logged in as admin.

**Create instances of your solution**
You can apply the samples (examples) from the config/sample:

```sh
kubectl apply -k config/samples/
```

>**NOTE**: Ensure that the samples has default values to test it out.

### To Uninstall
**Delete the instances (CRs) from the cluster:**

```sh
kubectl delete -k config/samples/
```

**Delete the APIs(CRDs) from the cluster:**

```sh
make uninstall
```

**UnDeploy the controller from the cluster:**

```sh
make undeploy
```

## Contributing
// TODO(user): Add detailed information on how you would like others to contribute to this project

**NOTE:** Run `make --help` for more information on all potential `make` targets

More information can be found via the [Kubebuilder Documentation](https://book.kubebuilder.io/introduction.html)

## License

Copyright 2024.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

