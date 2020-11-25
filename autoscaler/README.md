How to install cluster autoscaler:

https://docs.aws.amazon.com/eks/latest/userguide/cluster-autoscaler.html
kubectl apply -f https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml
kubectl -n kube-system annotate deployment.apps/cluster-autoscaler cluster-autoscaler.kubernetes.io/safe-to-evict="false"

# add balance-similar-node-groups and skip-nodes-with-system-pods=false
kubectl -n kube-system edit deployment.apps/cluster-autoscaler
    spec:
      containers:
      - command:
        - ./cluster-autoscaler
        - --v=4
        - --stderrthreshold=info
        - --cloud-provider=aws
        - --skip-nodes-with-local-storage=false
        - --expander=least-waste
        - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/<<YOUR CLUSTER NAME>>
        - --balance-similar-node-groups
        - --skip-nodes-with-system-pods=false
#replace version
kubectl -n kube-system set image deployment.apps/cluster-autoscaler cluster-autoscaler=us.gcr.io/k8s-artifacts-prod/autoscaling/cluster-autoscaler:v1.17.4

#check it out
kubectl -n kube-system logs -f deployment.apps/cluster-autoscaler

OR
next time try helm
$ helm repo add autoscaler https://kubernetes.github.io/autoscaler

# Method 1 - Using Autodiscovery
$ helm install my-release autoscaler/cluster-autoscaler \
--set 'autoDiscovery.clusterName'=<CLUSTER NAME> \
--set 'image.tag'=<VERSION> #v1.17.4 \
