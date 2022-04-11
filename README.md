# fun-with-kubernetes-jobs

Some samples illustrating the use of Kubernetes jobs inlcuding some of the lesser known features

## Single execution jobs

The simplest form of job is one which has one single execution, that is to say which runs one pod to completion. Take a look at [./single-execution/single-execution.yml](./single-execution/single-execution.yml) for an example of such a job. You can run it with

```shell
kubectl apply -f ./single-execution
```

Check for completion of the job by looking at the job object itself

```shell
kubectl get job single-execution
```

or by watching the pods in the namespace

```shell
watch -n 0.5 "kubectl get pods"
```

Look at the log of job execution

```watch
kubectl logs $(kubectl get pod -l job-name=single-execution -o jsonpath={'.items[0].metadata.name'})
```
