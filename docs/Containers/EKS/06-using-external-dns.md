# Using External DNS

## Create the service account for External DNS

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2 3 4 5"
    CLUSTER_NAME="<cluster name>"
    POLICY_NAME="<policy name>"
    ROLE_NAME="<role name>"
    PROJECT_NAME="<project name>"
    REGION="<region>"

    curl -LO https://raw.githubusercontent.com/marcus16-kang/aws-resources-example/main/scripts/eks/external-dns-iam-policy.json

    POLICY_ARN=$(aws iam create-policy \
        --policy-name $POLICY_NAME \
        --policy-document file://external-dns-iam-policy.json \
        --query 'Policy.Arn' \
        --output text \
        #  --tags Key=project,Value=$PROJECT_NAME \ # AWS CLI v2
    )
    
    eksctl create iamserviceaccount \
        --name external-dns \
        --namespace external-dns \
        --cluster $CLUSTER_NAME \
        --attach-policy-arn $POLICY_ARN \
        --role-name $ROLE_NAME \
        --tags project=$PROJECT_NAME \
        --region $REGION \
        --override-existing-serviceaccounts \
        --approve
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2 3 4 5"
    $CLUSTER_NAME="<cluster name>"
    $POLICY_NAME="<policy name>"
    $ROLE_NAME="<role name>"
    $PROJECT_NAME="<project name>"
    $REGION="<region>"

    curl.exe -LO https://raw.githubusercontent.com/marcus16-kang/aws-resources-example/main/scripts/eks/external-dns-iam-policy.json

    $POLICY_ARN = aws iam create-policy `
        --policy-name $POLICY_NAME `
        --policy-document file://external-dns-iam-policy.json `
        --query 'Policy.Arn' `
        --output text `
        --tags Key=project,Value=$PROJECT_NAME
    
    eksctl create iamserviceaccount `
        --name external-dns `
        --namespace external-dns `
        --cluster $CLUSTER_NAME `
        --attach-policy-arn $POLICY_ARN `
        --role-name $ROLE_NAME `
        --tags project=$PROJECT_NAME `
        --region $REGION `
        --override-existing-serviceaccounts `
        --approve
    ```

[AWS Documentation](https://aws.amazon.com/ko/premiumsupport/knowledge-center/eks-set-up-externaldns/)

## Install External DNS using `helm`

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2"
    DOMAIN_NAME="<domain name (ex. my-org.com)>"
    ZONE_TYPE="<zone type (public or private)>"

    helm repo add external-dns https://kubernetes-sigs.github.io/external-dns/

    helm install external-dns external-dns/external-dns \
        --namespace external-dns \
        --set serviceAccount.create=false \
        --set serviceAccount.name=external-dns \
        --set interval=10s \
        --set policy=sync \
        --set registry=txt \
        --set txtOwnerId=my-hostedzone-identifier \
        --set domainFilters[0]=$DOMAIN_NAME \
        --set provider=aws \
        --set extraArgs[0]="--aws-zone-type=$ZONE_TYPE"
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2"
    $DOMAIN_NAME="<domain name (ex. my-org.com)>"
    $ZONE_TYPE="<zone type (public or private)>"

    helm repo add external-dns https://kubernetes-sigs.github.io/external-dns/

    helm install external-dns external-dns/external-dns `
        --namespace external-dns `
        --set serviceAccount.create=false `
        --set serviceAccount.name=external-dns `
        --set interval=10s `
        --set policy=sync `
        --set registry=txt `
        --set txtOwnerId=my-hostedzone-identifier `
        --set domainFilters[0]=$DOMAIN_NAME `
        --set provider=aws `
        --set extraArgs[0]="--aws-zone-type=$ZONE_TYPE"
    ```

[External DNS Helm Chart Documentation](https://github.com/kubernetes-sigs/external-dns/tree/master/charts/external-dns)

### External DNS policy references

- `upsert-only` : CREATE, UPDATE
- `sync` : CREATE, UPDATE, DELETE
- `create-only` : CREATE

[AWS Documentation](https://aws.amazon.com/ko/premiumsupport/knowledge-center/eks-set-up-externaldns/)

[External DNS for AWS Documentation](https://kubernetes-sigs.github.io/external-dns/v0.13.5/tutorials/aws/)

## Using External DNS with same domain for public and private Route53 zones

### Install two AWS load balancer controller

**For public zone**

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2 3"
    VPC_ID="<vpc id>"
    CLUSTER_NAME="<cluster name>"
    REGION="<region>"

    helm repo add eks https://aws.github.io/eks-charts
    helm repo update

    helm install aws-load-balancer-controller-public eks/aws-load-balancer-controller \
        -n kube-system \
        --set clusterName=$CLUSTER_NAME \
        --set serviceAccount.create=false \
        --set serviceAccount.name=aws-load-balancer-controller \
        --set region=$REGION \
        --set vpcId=$VPC_ID \
        --set ingressClass=alb-public \
        --set nameOverride=alb-public

    kubectl get deployment aws-load-balancer-controller-public \
        -n kube-system \
        -w
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2 3"
    $VPC_ID="<vpc id>"
    $CLUSTER_NAME="<cluster name>"
    $REGION="<region>"

    helm repo add eks https://aws.github.io/eks-charts
    helm repo update

    helm install aws-load-balancer-controller-public eks/aws-load-balancer-controller `
        -n kube-system `
        --set clusterName=$CLUSTER_NAME `
        --set serviceAccount.create=false `
        --set serviceAccount.name=aws-load-balancer-controller `
        --set region=$REGION `
        --set vpcId=$VPC_ID `
        --set ingressClass=alb-public `
        --set nameOverride=alb-public

    kubectl get deployment aws-load-balancer-controller-public `
        -n kube-system `
        -w
    ```

!!! note

    If you create a ingress for public zone, you should define ingress class like this.

    ``` yaml
    ingressClassName: alb-public
    ```

**For private zone**

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2 3"
    VPC_ID="<vpc id>"
    CLUSTER_NAME="<cluster name>"
    REGION="<region>"

    helm repo add eks https://aws.github.io/eks-charts
    helm repo update

    helm install aws-load-balancer-controller-private eks/aws-load-balancer-controller \
        -n kube-system \
        --set clusterName=$CLUSTER_NAME \
        --set serviceAccount.create=false \
        --set serviceAccount.name=aws-load-balancer-controller \
        --set region=$REGION \
        --set vpcId=$VPC_ID \
        --set ingressClass=alb-private \
        --set nameOverride=alb-private

    kubectl get deployment aws-load-balancer-controller-private \
        -n kube-system \
        -w
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2 3"
    $VPC_ID="<vpc id>"
    $CLUSTER_NAME="<cluster name>"
    $REGION="<region>"

    helm repo add eks https://aws.github.io/eks-charts
    helm repo update

    helm install aws-load-balancer-controller-private eks/aws-load-balancer-controller `
        -n kube-system `
        --set clusterName=$CLUSTER_NAME `
        --set serviceAccount.create=false `
        --set serviceAccount.name=aws-load-balancer-controller `
        --set region=$REGION `
        --set vpcId=$VPC_ID `
        --set ingressClass=alb-private `
        --set nameOverride=alb-private

    kubectl get deployment aws-load-balancer-controller-private `
        -n kube-system `
        -w
    ```

!!! note

    If you create a ingress for private zone, you should define ingress class like this.

    ``` yaml
    ingressClassName: alb-private
    ```

### Install two External DNS

**For public zone**

=== ":simple-linux: Linux"

    ``` bash hl_lines="1"
    DOMAIN_NAME="<domain name (ex. my-org.com)>"
    ZONE_TYPE="public"

    helm repo add external-dns https://kubernetes-sigs.github.io/external-dns/

    helm install external-dns-public external-dns/external-dns \
        --namespace external-dns \
        --set serviceAccount.create=false \
        --set serviceAccount.name=external-dns \
        --set interval=10s \
        --set policy=sync \
        --set registry=txt \
        --set txtOwnerId=my-hostedzone-identifier \
        --set domainFilters[0]=$DOMAIN_NAME \
        --set provider=aws \
        --set extraArgs[0]="--aws-zone-type=$ZONE_TYPE" \
        --set extraArgs[1]="--ingress-class=alb-public"
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1"
    $DOMAIN_NAME="<domain name (ex. my-org.com)>"
    $ZONE_TYPE="public"

    helm repo add external-dns https://kubernetes-sigs.github.io/external-dns/

    helm install external-dns-public external-dns/external-dns `
        --namespace external-dns `
        --set serviceAccount.create=false `
        --set serviceAccount.name=external-dns `
        --set interval=10s `
        --set policy=sync `
        --set registry=txt `
        --set txtOwnerId=my-hostedzone-identifier `
        --set domainFilters[0]=$DOMAIN_NAME `
        --set provider=aws `
        --set extraArgs[0]="--aws-zone-type=$ZONE_TYPE" `
        --set extraArgs[1]="--ingress-class=alb-public"
    ```

**For private zone**

=== ":simple-linux: Linux"

    ``` bash hl_lines="1"
    DOMAIN_NAME="<domain name (ex. my-org.com)>"
    ZONE_TYPE="private"

    helm repo add external-dns https://kubernetes-sigs.github.io/external-dns/

    helm install external-dns-private external-dns/external-dns \
        --namespace external-dns \
        --set serviceAccount.create=false \
        --set serviceAccount.name=external-dns \
        --set interval=10s \
        --set policy=sync \
        --set registry=txt \
        --set txtOwnerId=my-hostedzone-identifier \
        --set domainFilters[0]=$DOMAIN_NAME \
        --set provider=aws \
        --set extraArgs[0]="--aws-zone-type=$ZONE_TYPE" \
        --set extraArgs[1]="--ingress-class=alb-private"
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1"
    $DOMAIN_NAME="<domain name (ex. my-org.com)>"
    $ZONE_TYPE="private"

    helm repo add external-dns https://kubernetes-sigs.github.io/external-dns/

    helm install external-dns-private external-dns/external-dns `
        --namespace external-dns `
        --set serviceAccount.create=false `
        --set serviceAccount.name=external-dns `
        --set interval=10s `
        --set policy=sync `
        --set registry=txt `
        --set txtOwnerId=my-hostedzone-identifier `
        --set domainFilters[0]=$DOMAIN_NAME `
        --set provider=aws `
        --set extraArgs[0]="--aws-zone-type=$ZONE_TYPE" `
        --set extraArgs[1]="--ingress-class=alb-private"
    ```

## Using External DNS with ALB (in Ingress)

``` yaml title="ingress.yaml" hl_lines="12" linenums="1"
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  namespace: <namespace>
  name: <ingress name>
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
spec:
  ingressClassName: alb
  rules:
    - host: <hostname (ex. echoserver.mycluster.example.org)>
      http:
        path: /
        pathType: Prefix
        backend:
          service:
            name: <service name>
            port:
              number: <port number> # like 80
```

### Dualstack ALB

``` yaml title="ingress.yaml" hl_lines="13" linenums="1"
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  namespace: <namespace>
  name: <ingress name>
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/ip-address-type: dualstack
    alb.ingress.kubernetes.io/target-type: ip
spec:
  ingressClassName: alb
  rules:
    - host: <hostname (ex. echoserver.example.org)>
      http:
        path: /
        pathType: Prefix
        backend:
          service:
            name: <service name>
            port:
              number: <port number> # like 80
```

[External DNS Documentation](https://kubernetes-sigs.github.io/external-dns/v0.12.2/tutorials/alb-ingress/)

## Using External DNS with NLB

``` yaml title="service.yaml" hl_lines="7" linenums="1"
apiVersion: v1
kind: Service
metadata:
  namespace: <namespace>
  name: <ingress name>
  annotations:
    external-dns.alpha.kubernetes.io/hostname: <hostname (ex. echoserver.example.org)>
    service.beta.kubernetes.io/aws-load-balancer-type: external
    service.beta.kubernetes.io/aws-load-balancer-nlb-target-type: ip
    service.beta.kubernetes.io/aws-load-balancer-scheme: internet-facing
spec:
  type: LoadBalancer
  selector:
    app: <app label>
  ports:
    - port: <port number> # like 80
      targetPort: <port number> # like 80
      protocol: <protocol> # like TCP
```

[External DNS Documentation](https://kubernetes-sigs.github.io/external-dns/v0.12.2/tutorials/aws/)

## Using External DNS with NodePort Service

``` yaml title="service.yaml" hl_lines="7" linenums="1"
apiVersion: v1
kind: Service
metadata:
  name: <name>
  namespace: <namespace>
  annotations:
    external-dns.alpha.kubernetes.io/hostname: <hostname (ex. echoserver.example.org)>
spec:
  selector:
    app: <app label>
  type: NodePort
  ports:
    - port: <port number> # like 80
      targetPort: <port number> # like 80
      protocol: <protocol> # like TCP
```

!!! warning

    If you use `ClusterIP`, you **CANNOT** use External DNS.