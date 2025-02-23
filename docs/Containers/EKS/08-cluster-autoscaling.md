# Cluster Autoscaling

## Using Cluster Autoscaler(CA)

### Create the service account

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2 3 4 5"
    POLICY_NAME="<policy name>"
    ROLE_NAME="<role name>"
    CLUSTER_NAME="<cluster name>"
    PROJECT_NAME="<project name>"
    REGION="<region code>"

    curl -LO https://raw.githubusercontent.com/marcus16-kang/aws-resources-example/main/scripts/eks/cluster-autoscaler-policy.json

    POLICY_ARN=$(aws iam create-policy \
        --policy-name $POLICY_NAME \
        --policy-document file://cluster-autoscaler-policy.json \
        --query 'Policy.Arn' \
        --output text \
    #    --tags Key=project,Value=$PROJECT_NAME \ # AWS CLI v2
    )

    eksctl create iamserviceaccount \
        --cluster=$CLUSTER_NAME \
        --namespace=kube-system \
        --name=cluster-autoscaler \
        --role-name=$ROLE_NAME \
        --attach-policy-arn=$POLICY_ARN \
        --region $REGION \
        --override-existing-serviceaccounts \
        --approve
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2 3 4 5"
    $POLICY_NAME="<policy name>"
    $ROLE_NAME="<role name>"
    $CLUSTER_NAME="<cluster name>"
    $PROJECT_NAME="<project name>"
    $REGION="<region code>"

    curl.exe -LO https://raw.githubusercontent.com/marcus16-kang/aws-resources-example/main/scripts/eks/cluster-autoscaler-policy.json

    POLICY_ARN = aws iam create-policy `
        --policy-name $POLICY_NAME `
        --policy-document file://cluster-autoscaler-policy.json `
        --query 'Policy.Arn' `
        --output text `
       --tags Key=project,Value=$PROJECT_NAME `

    eksctl create iamserviceaccount `
        --cluster=$CLUSTER_NAME `
        --namespace=kube-system `
        --name=cluster-autoscaler `
        --role-name=$ROLE_NAME `
        --attach-policy-arn=$POLICY_ARN `
        --region $REGION `
        --override-existing-serviceaccounts `
        --approve
    ```

[AWS Documentation](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/autoscaling.html#ca-create-policy)

[Autoscaler Documentation](https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/README.md#iam-policy)

### Deploy the Cluster Autoscaler using `helm`

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2 3"
    CLUSTER_NAME="<cluster name>"
    IMAGE_TAG="<image tag (ex. 1.27.2)>"
    REGION="<region code>"

    helm repo add autoscaler https://kubernetes.github.io/autoscaler

    helm install cluster-autoscaler autoscaler/cluster-autoscaler \
        --namespace kube-system \
        --set autoDiscovery.clusterName=$CLUSTER_NAME \
        --set awsRegion=$REGION \
        --set cloudProvider=aws \
        --set extraArgs.logtostderr=true \
        --set extraArgs.stderrthreshold=info \
        --set extraArgs.v=4 \
        --set extraArgs.skip-nodes-with-local-storage=false \
        --set extraArgs.expander=least-waste \
        --set extraArgs.balance-similar-node-groups=true \
        --set extraArgs.skip-nodes-with-system-pods=false \
        --set image.tag=v$IMAGE_TAG \
        --set rbac.serviceAccount.create=false \
        --set rbac.serviceAccount.name=cluster-autoscaler
    ```

=== ":simple-windows: Windows"

    ``` bash hl_lines="1 2 3"
    $CLUSTER_NAME="<cluster name>"
    $IMAGE_TAG="<image tag (ex. 1.27.2)>"
    $REGION="<region code>"

    helm repo add autoscaler https://kubernetes.github.io/autoscaler

    helm install cluster-autoscaler autoscaler/cluster-autoscaler `
        --namespace kube-system `
        --set autoDiscovery.clusterName=$CLUSTER_NAME `
        --set awsRegion=$REGION `
        --set cloudProvider=aws `
        --set extraArgs.logtostderr=true `
        --set extraArgs.stderrthreshold=info `
        --set extraArgs.v=4 `
        --set extraArgs.skip-nodes-with-local-storage=false `
        --set extraArgs.expander=least-waste `
        --set extraArgs.balance-similar-node-groups=true `
        --set extraArgs.skip-nodes-with-system-pods=false `
        --set image.tag=v$IMAGE_TAG `
        --set rbac.serviceAccount.create=false `
        --set rbac.serviceAccount.name=cluster-autoscaler
    ```

| Kubernetes | CA     | Default |
|------------|--------|---------|
| 1.32       | 1.32.0 | default |
| 1.31       | 1.31.1 |         |
| 1.30       | 1.30.2 |         |
| 1.29       | 1.29.5 |         |

> Go to [here](https://github.com/kubernetes/autoscaler/releases) and please check the new version of your kubernetes version. (2025-02-23)

[AWS Documentation](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/autoscaling.html#ca-deploy)

## Using Karpenter

### Add tags to subnets

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 3 4"
    REGION=""

    CLUSTER_NAME=""
    SUBNET_LIST=(
        "subnet-xxxxxxxxxxxxxxxxx"
        "subnet-xxxxxxxxxxxxxxxxx"
    )

    for subnet in "${SUBNET_LIST[@]}"
    do
        aws ec2 create-tags \
            --tags "Key=karpenter.sh/discovery,Value=${CLUSTER_NAME}" \
            --resources ${subnet} \
            --region $REGION \
            --no-cli-pager
    done
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 3 4"
    $REGION=""

    $CLUSTER_NAME=""
    $SUBNET_LIST  = @(
        "subnet-xxxxxxxxxxxxxxxxx",
        "subnet-xxxxxxxxxxxxxxxxx"
    )

    foreach ($subnet in $SUBNET_LIST) {
        aws ec2 create-tags `
            --tags "Key=karpenter.sh/discovery,Value=${CLUSTER_NAME}" `
            --resources ${subnet} `
            --region $REGION `
            --no-cli-pager
    }
    ```

### Add tags to security groups

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 3 4"
    REGION=""

    CLUSTER_NAME=""
    SG_LIST=(
        "sg-xxxxxxxxxxxxxxxxx"
        "sg-xxxxxxxxxxxxxxxxx"
    )

    for sg in "${SG_LIST[@]}"
    do
        aws ec2 create-tags \
            --tags "Key=karpenter.sh/discovery,Value=${CLUSTER_NAME}" \
            --resources ${sg} \
            --region $REGION \
            --no-cli-pager
    done
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 3 4"
    $REGION=""

    $CLUSTER_NAME=""
    $SG_LIST  = @(
        "sg-xxxxxxxxxxxxxxxxx",
        "sg-xxxxxxxxxxxxxxxxx"
    )

    foreach ($sg in $SG_LIST) {
        aws ec2 create-tags `
            --tags "Key=karpenter.sh/discovery,Value=${CLUSTER_NAME}" `
            --resources ${sg} `
            --region $REGION `
            --no-cli-pager
    }
    ```

### Create the Karpenter Controller policy

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2 3 6 9 10 13"
    STACK_NAME=""
    PROJECT_NAME=""
    REGION=""

    ### Policy Configuration
    PolicyName=""     # [REQUIRED] The name of service account's IAM policy.

    ### EKS Configuration
    ClusterName=""    # [REQUIRED] The name of EKS cluster.
    NodeRoleName=""   # [REQUIRED] The name of EKS node's role.

    ### SQS Configuration
    QueueName=""      # [REQUIRED] The name of SQS queue for NTH.

    curl -LO https://raw.githubusercontent.com/marcus16-kang/aws-resources-example/main/scripts/eks/karpenter-controller-policy.yaml

    aws cloudformation deploy \
        --template-file ./karpenter-controller-policy.yaml \
        --stack-name $STACK_NAME \
        --parameter-overrides \
            PolicyName=$PolicyName \
            ClusterName=$ClusterName \
            NodeRoleName=$NodeRoleName \
            QueueName=$QueueName \
        --tags project=$PROJECT_NAME \
        --capabilities CAPABILITY_NAMED_IAM \
        --disable-rollback \
        --region $REGION
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2 3 6 9 10 13"
    $STACK_NAME=""
    $PROJECT_NAME=""
    $REGION=""

    ### Policy Configuration
    $PolicyName=""    # [REQUIRED] The name of service account's IAM policy.

    ### EKS Configuration
    $ClusterName=""   # [REQUIRED] The name of EKS cluster.
    $NodeRoleName=""  # [REQUIRED] The name of EKS node's role.

    ### SQS Configuration
    $QueueName=""      # [REQUIRED] The name of SQS queue for NTH

    curl.exe -LO https://raw.githubusercontent.com/marcus16-kang/aws-resources-example/main/scripts/eks/karpenter-controller-policy.yaml

    aws cloudformation deploy `
        --template-file ./karpenter-controller-policy.yaml `
        --stack-name $STACK_NAME `
        --parameter-overrides `
            PolicyName=$PolicyName `
            ClusterName=$ClusterName `
            NodeRoleName=$NodeRoleName `
            QueueName=$QueueName `
        --tags project=$PROJECT_NAME `
        --capabilities CAPABILITY_NAMED_IAM `
        --disable-rollback `
        --region $REGION
    ```

[Karpenter Documentation](https://karpenter.sh/preview/getting-started/migrating-from-cas/#create-iam-roles)

### Create the service account

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2 3 4 5"
    POLICY_NAME="<policy name>"
    ROLE_NAME="<role name>"
    CLUSTER_NAME="<cluster name>"
    PROJECT_NAME="<project name>"
    REGION="<region code>"

    POLICY_ARN=$(aws iam list-policies \
        --query "Policies[?PolicyName=='$POLICY_NAME'].Arn" --output text)

    eksctl create iamserviceaccount \
        --cluster=$CLUSTER_NAME \
        --namespace=karpenter \
        --name=karpenter \
        --role-name=$ROLE_NAME \
        --attach-policy-arn=$POLICY_ARN \
        --region $REGION \
        --approve
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2 3 4 5"
    $POLICY_NAME="<policy name>"
    $ROLE_NAME="<role name>"
    $CLUSTER_NAME="<cluster name>"
    $PROJECT_NAME="<project name>"
    $REGION="<region code>"

    $POLICY_ARN = aws iam list-policies `
        --query "Policies[?PolicyName=='$POLICY_NAME'].Arn" --output text

    eksctl create iamserviceaccount `
        --cluster=$CLUSTER_NAME `
        --namespace=karpenter `
        --name=karpenter `
        --role-name=$ROLE_NAME `
        --attach-policy-arn=$POLICY_ARN `
        --region $REGION `
        --approve
    ```

!!! note

    If you use CMK in SQS queue, you should add below policy.

    ``` json title="KMS Key Policy" hl_lines="6"
    {
        "Sid": "AllowConsumersToReceiveFromTheQueue - Karpenter",
        "Effect": "Allow",
        "Principal": {
            "AWS": [
                "<KARPENTER SERVICE ACCOUNT ROLE ARN>"
            ]
        },
        "Action": [
            "kms:Decrypt"
        ],
        "Resource": "*"
    }
    ```

### Install Karpenter using `helm`

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2 3"
    CLUSTER_NAME=""
    NODEGROUP_ROLE_NAME=""
    QUEUE_NAME=""

    helm repo add karpenter https://charts.karpenter.sh/
    helm repo update

    helm install karpenter oci://public.ecr.aws/karpenter/karpenter \
        --create-namespace \
        --namespace karpenter \
        --version v0.37.7 \
        --set serviceAccount.create=false \
        --set serviceAccount.name=karpenter \
        --set settings.aws.clusterName=$CLUSTER_NAME \
        --set settings.aws.defaultInstanceProfile=$NODEGROUP_ROLE_NAME \
        --set settings.aws.interruptionQueueName=$QUEUE_NAME
    ```

=== ":simple-windows: Windows"

    ``` bash hl_lines="1 2 3"
    $CLUSTER_NAME=""
    $NODEGROUP_ROLE_NAME=""
    $QUEUE_NAME=""

    helm repo add karpenter https://charts.karpenter.sh/
    helm repo update

    helm install karpenter oci://public.ecr.aws/karpenter/karpenter `
        --create-namespace `
        --namespace karpenter `
        --version v0.37.7 `
        --set serviceAccount.create=false `
        --set serviceAccount.name=karpenter `
        --set settings.aws.clusterName=$CLUSTER_NAME `
        --set settings.aws.defaultInstanceProfile=$NODEGROUP_ROLE_NAME `
        --set settings.aws.interruptionQueueName=$QUEUE_NAME
    ```

> Go to [here](https://github.com/aws/karpenter/releases) and please check the new version. (2025-02-23)

### Create a node template

``` yaml title="node-template.yaml" linenums="1"
apiVersion: karpenter.k8s.aws/v1alpha1
kind: AWSNodeTemplate
metadata:
  name: <node template name>
spec:
  subnetSelector: { ... }        # required, discovers tagged subnets to attach to instances
  securityGroupSelector: { ... } # required, discovers tagged security groups to attach to instances
  instanceProfile: "..."         # optional, overrides the node's identity from global settings
  amiFamily: "..."               # optional, resolves a default ami and userdata
  amiSelector: { ... }           # optional, discovers tagged amis to override the amiFamily's default
  userData: "..."                # optional, overrides autogenerated userdata with a merge semantic
  tags: { ... }                  # optional, propagates tags to underlying EC2 resources
  metadataOptions: { ... }       # optional, configures IMDS for the instance
  blockDeviceMappings: [ ... ]   # optional, configures storage devices for the instance
  detailedMonitoring: "..."      # optional, configures detailed monitoring for the instance
```

??? note "Amazon Linux 2 Example"

    ``` yaml title="node-template.yaml" linenums="1"
    apiVersion: karpenter.k8s.aws/v1alpha1
    kind: AWSNodeTemplate
    metadata:
      name: <node-template-name>
    spec:
      subnetSelector:
        aws-ids: "subnet-xxxxxxxxxxxxxxxxx,subnet-xxxxxxxxxxxxxxxxx"
      securityGroupSelector:
        aws-ids: "sg-xxxxxxxxxxxxxxxxx,sg-xxxxxxxxxxxxxxxxx"
      instanceProfile: <instance-profile-name>
      amiFamily: AL2
      tags:
        Name: <instance, volume, lt name>
        project: <project name>
      metadataOptions:
        httpEndpoint: enabled
        httpProtocolIPv6: disabled
        httpPutResponseHopLimit: 2
        httpTokens: required
      blockDeviceMappings:
        - deviceName: /dev/xvda
          ebs:
            volumeSize: 50Gi
            volumeType: gp3
            encrypted: true
    ```

??? note "Bottlerocket Example"

    ``` yaml title="node-template.yaml" linenums="1"
    apiVersion: karpenter.k8s.aws/v1alpha1
    kind: AWSNodeTemplate
    metadata:
      name: <node-template-name>
    spec:
      subnetSelector:
        aws-ids: "subnet-xxxxxxxxxxxxxxxxx,subnet-xxxxxxxxxxxxxxxxx"
      securityGroupSelector:
        aws-ids: "sg-xxxxxxxxxxxxxxxxx,sg-xxxxxxxxxxxxxxxxx"
      instanceProfile: <instance-profile-name>
      amiFamily: Bottlerocket
      tags:
        Name: <instance, volume, lt name>
        project: <project name>
      metadataOptions:
        httpEndpoint: enabled
        httpProtocolIPv6: disabled
        httpPutResponseHopLimit: 2
        httpTokens: required
      blockDeviceMappings:
        # Root device
        - deviceName: /dev/xvda
          ebs:
            volumeSize: 4Gi
            volumeType: gp3
            encrypted: true
        # Data device: Container resources such as images and logs
        - deviceName: /dev/xvdb
          ebs:
            volumeSize: 50Gi
            volumeType: gp3
            encrypted: true
    ```

[Karpenter Documentation](https://karpenter.sh/v0.28/concepts/node-templates/)

### Create a provisioner

``` yaml title="provisioner" linenums="1"
apiVersion: karpenter.sh/v1alpha5
kind: Provisioner
metadata:
  name: default
spec:
  # References cloud provider-specific custom resource, see your cloud provider specific documentation
  providerRef:
    name: default # Node Template Name

  # Provisioned nodes will have these taints
  # Taints may prevent pods from scheduling if they are not tolerated by the pod.
  taints:
    - key: key1
      value: "value1"
      effect: NoSchedule

  # Provisioned nodes will have these taints, but pods do not need to tolerate these taints to be provisioned by this
  # provisioner. These taints are expected to be temporary and some other entity (e.g. a DaemonSet) is responsible for
  # removing the taint after it has finished initializing the node.
  startupTaints:
    - key: key1
      effect: NoSchedule

  # Labels are arbitrary key-values that are applied to all nodes
  labels:
    key1: value1

  # Annotations are arbitrary key-values that are applied to all nodes
  annotations:
    key1: "value1"

  # Requirements that constrain the parameters of provisioned nodes.
  # These requirements are combined with pod.spec.affinity.nodeAffinity rules.
  # Operators { In, NotIn } are supported to enable including or excluding values
  requirements:
    - key: "karpenter.k8s.aws/instance-category"
      operator: In
      values: ["c", "m", "r"]
    - key: "karpenter.k8s.aws/instance-cpu"
      operator: In
      values: ["4", "8", "16", "32"]
    - key: "karpenter.k8s.aws/instance-hypervisor"
      operator: In
      values: ["nitro"]
    - key: karpenter.k8s.aws/instance-generation
      operator: Gt
      values: ["2"]
    - key: "topology.kubernetes.io/zone"
      operator: In
      values: ["us-west-2a", "us-west-2b"]
    - key: "kubernetes.io/arch"
      operator: In
      values: ["arm64", "amd64"]
    - key: "karpenter.sh/capacity-type" # If not included, the webhook for the AWS cloud provider will default to on-demand
      operator: In
      values: ["spot", "on-demand"]

  # Karpenter provides the ability to specify a few additional Kubelet args.
  # These are all optional and provide support for additional customization and use cases.
  kubeletConfiguration:
    clusterDNS: ["10.0.1.100"]
    containerRuntime: containerd
    systemReserved:
      cpu: 100m
      memory: 100Mi
      ephemeral-storage: 1Gi
    kubeReserved:
      cpu: 200m
      memory: 100Mi
      ephemeral-storage: 3Gi
    evictionHard:
      memory.available: 5%
      nodefs.available: 10%
      nodefs.inodesFree: 10%
    evictionSoft:
      memory.available: 500Mi
      nodefs.available: 15%
      nodefs.inodesFree: 15%
    evictionSoftGracePeriod:
      memory.available: 1m
      nodefs.available: 1m30s
      nodefs.inodesFree: 2m
    evictionMaxPodGracePeriod: 60
    imageGCHighThresholdPercent: 85
    imageGCLowThresholdPercent: 80
    cpuCFSQuota: true
    podsPerCore: 2
    maxPods: 20
    

  # Resource limits constrain the total size of the cluster.
  # Limits prevent Karpenter from creating new instances once the limit is exceeded.
  limits:
    resources:
      cpu: "1000"
      memory: 1000Gi

  # Enables consolidation which attempts to reduce cluster cost by both removing un-needed nodes and down-sizing those
  # that can't be removed.  Mutually exclusive with the ttlSecondsAfterEmpty parameter.
  consolidation:
    enabled: true

  # If omitted, the feature is disabled and nodes will never expire.  If set to less time than it requires for a node
  # to become ready, the node may expire before any pods successfully start.
  ttlSecondsUntilExpired: 2592000 # 30 Days = 60 * 60 * 24 * 30 Seconds;

  # If omitted, the feature is disabled, nodes will never scale down due to low utilization
  ttlSecondsAfterEmpty: 30

  # Priority given to the provisioner when the scheduler considers which provisioner
  # to select. Higher weights indicate higher priority when comparing provisioners.
  # Specifying no weight is equivalent to specifying a weight of 0.
  weight: 10
```

[Karpenter Documentation](https://karpenter.sh/v0.28/concepts/provisioners/)

## Deploy Overprovisioning Pod

``` yaml title="overprovisioning.yaml"
---
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: overprovisioning
value: -1
globalDefault: false
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: overprovisioning
  namespace: kube-system
spec:
  replicas: 3
  selector:
    matchLabels:
      run: overprovisioning
  template:
    metadata:
      labels:
        run: overprovisioning
    spec:
      priorityClassName: overprovisioning
      containers:
      - name: reserve-resources
        image: k8s.gcr.io/pause
        resources:
          requests:
            cpu: 820m
            memory: 2000Mi
      # tolerations:
      #   - key: "key1"         # taint key
      #     value: "value1"     # taint value
      #     operator: "Equal"
      #     effect: "NoSchedule"
      # nodeSelector:
      #   key: value            # node label key and value
```