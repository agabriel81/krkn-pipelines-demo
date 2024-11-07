# krkn-pipelines-demo

This is a very basic OpenShift Pipelines example with the aim to automate Kraken scenarios and demonstrate OpenShift Pipelines capabilities.

## Requirements

To run the OpenShift Pipeline, you need to install OpenShift Pipelines from the OperatorHub following the instruction on the official documentation.
This example is based on the OpenShift Pipelines 1.16.

- `OpenShift Pipelines` Operator [official documentation](https://docs.openshift.com/pipelines/1.16/install_config/installing-pipelines.html)
- `tkn` CLI available [here](https://console.redhat.com/openshift/downloads)

## Create a sample app in a test namespace

~~~
$ oc new-project test
$ oc new-app --name nodejs-s2i https://github.com/nodeshift-starters/nodejs-rest-http -i nodejs:18-ubi8-minimal
$ oc expose svc nodejs-s2i
$ oc set volume deploy/nodejs-s2i --add -t pvc --name=test --claim-name=test --claim-class=gp3 --claim-size=1G --mount-path=/tes
~~~

## Cluster credentials

We will create a Kubernetes secret which contains the `kubeconfig` needed.
It will be referenced and mounted inside the TaskRun. 

~~~
$ oc create secret generic kubeconfig --from-file=config=<your kubeconfig file> -n test
~~~

## OpenShift Pipelines resources

Create the tasks and pipelines resources:

~~~
$ git clone https://
$ oc create -f task_pvc.yaml
$ oc create -f task_pod_network.yaml
$ oc create -f krkn-pipeline.yaml
~~~

Run the pipeline:

~~~
$ tkn pipeline start chaos-test-1 \
--param pod_namespace=test \
--param pod_label='deployment=nodejs-s2i' \
--param instance_count=1 \
--param traffic_type='[ingress]' \
--param ingress_port='[8080]' \
--param egress_port='[]' \
--param wait_duration=100 \
--param test_duration=40 \
--param pvc_name=test \
--param block_size=1024000 \
--param fill_percentage=99 \
--param pvc_duration=60 \
-n test
~~~

You can follow the progress from either the OpenShift Console - Pipelines or via `tkn` CLI:

~~~
$ tkn pipelinerun logs chaos-test-1-run-<random id> -f -n test
~~~

This pipeline can be further automated with EventListener and Triggers, and the kubeconfig credentatials can be stored in a more secure way, for example with the integration of [Secret Store CSI driver](https://docs.openshift.com/container-platform/4.17/storage/container_storage_interface/persistent-storage-csi-secrets-store.html) (Technical Preview) and external secret stores such as [AWS KMS, Azure Key Vault or Hashicorp Vault](https://docs.openshift.com/container-platform/4.17/nodes/pods/nodes-pods-secrets-store.html#mounting-secrets-external-secrets-store)

