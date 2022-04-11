# fun-with-kubernetes-jobs

Some samples illustrating the use of Kubernetes jobs inlcuding some of the lesser known features

## Single execution jobs

The simplest form of job is one which has one single execution, that is to say which runs one pod to completion. Take a look at [](./single-execution/single-execution.yml) for an example of such a job. You can run it with

```shell
kubectl apply -f ./single-execution
```
