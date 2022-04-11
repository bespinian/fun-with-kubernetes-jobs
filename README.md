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

Clean up the job by executing

```shell
kubectl delete -f ./single-execution
```

## Multiple execution jobs

Jobs can also have multiple executions. This can be achieved by setting the `completions` attribute to the desired number of executions.

### Serial executions

By default all executions of a job run one after the other. Take a look at [./multiple-executions/multiple-executions-serial.yml](./multiple-executions/multiple-executions-serial.yml) for an example of such a job. You can run it with

```shell
kubectl apply -f ./multiple-executions/multiple-executions-serial.yml
```

and observe the three pods running to completion in sequence

```shell
watch -n 0.5 "kubectl get pods"
```

### Indexed execution

Sometimes each individual execution needs to know it's index in the sequence of all executions. This can be achieved by setting the `completionMode` attribute to `Indexed`. With this setting the job's containers get an environment variable named `JOB_COMPLETION_INDEX` from which they can read their index. Indexed execution is useful, for example, for determining the segment of data which a particular execution needs to operate on. Take a look at [./multiple-executions/multiple-executions-serial-indexed.yml](./multiple-executions/multiple-executions-serial-indexed.yml) for an example of such a job. You can run it with

```shell
kubectl apply -f ./multiple-executions/multiple-executions-serial-indexed.yml
```

and observe the three pods running to completion in sequence

```shell
watch -n 0.5 "kubectl get pods"
```

Now check the logs of each execution's pod

```shell
kubectl logs $(kubectl get pod -l job-name=multiple-execution-serial-indexed -o jsonpath={'.items[0].metadata.name'})
kubectl logs $(kubectl get pod -l job-name=multiple-execution-serial-indexed -o jsonpath={'.items[1].metadata.name'})
kubectl logs $(kubectl get pod -l job-name=multiple-execution-serial-indexed -o jsonpath={'.items[2].metadata.name'})
```

### Parallel executions

Job can also run their executions in parallel. This is achieved by setting the `parallelism` attribute to a number less than or equal to the number of `completions`. Using these two attributes, jobs can be configured to run all of their executions in parallel or only some. Take a look at [./multiple-executions/multiple-executions-parallel-full.yml](./multiple-executions/multiple-executions-parallel-full.yml) for an example of a job where all executions run in parallel. Take a look at [./multiple-executions/multiple-executions-parallel-partial.yml](./multiple-executions/multiple-executions-parallel-partial.yml) for a job where some but not all executions run in parallel. Run them with

```shell
kubectl apply -f ./multiple-executions/multiple-executions-parallel-full.yml
kubectl apply -f ./multiple-executions/multiple-executions-parallel-partial.yml
```

and observe the pods being scheduled for both jobs.

Clean up the example jobs after you are finished

```shell
kubectl delete -f ./multiple-executions
```

## Retries

Jobs do not need to implement retry logic themselves. Instead, they should just ensure that a non-zero exit code is returned of they run into a problem and should be restarted again at a later time. The Job resource will ensure that such retries are performed a configurable number of times and with a suitable back-off time. Take a look at [./retries/target-deployment.yml](./retries/target-deployment.yml) for a dummy application which takes a long time to get ready. The job in [./retries/retries.yml](./retries/retries.yml) will fail while the application is not yet ready and will be automatically restarted until it eventually succeeds. Run the setup with

```shell
kubectl apply -f ./retries
```

and observe the retry behavior by looking at the job's pods

```shell
watch -n 0.5 "kubectl get pods"
```

Clean up the example jobs after you are finished

```shell
kubectl delete -f ./retries
```

## Scripting

Often, jobs will be made up of a shell script running in a general purpose container. Such scripts can be included inline in the job resource using a multi-line string for the script. The script can use environment variables which are set at runtime of the container. Take a look at [./scripting/inline-script.yml](./scripting/inline-script.yml) for an example of such a job. Sometimes, especially if a large script is involved, it is cleaner to mount the script as a file from a ConfigMap. Take a look at [./scripting/file-script.yml](./scripting/file-script.yml) for an example of such a job.

## Immutability

Jobs are defined to run to completion in a pre-defined number of executions. Due to these semantics, changing a jobs definition after it has been created does not make sense, since it would not be clear what to do about already completed executions. For this reason, jobs are immutable. You can test this by creating an example job

```shell
kubectl apply -f ./immutability/update-attempt.yml
```

and then trying to change a line in the script of the job. You will see an error of the form

```
The Job "update-attempt" is invalid: spec.template: Invalid value: core.PodTemplateSpec{ ... }: field is immutable
```

due to the immutability of the job.
