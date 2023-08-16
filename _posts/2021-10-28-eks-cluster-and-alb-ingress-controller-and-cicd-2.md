---
layout: single
title: "EKS Cluster 생성 + ALB Ingress Controller + CI/CD(2)"
categories: AWS
tag: [EKS]
author_profile: false
sidebar:
    nav: "counts"
search: true
use_math: false
---

## 2. ALB Ingress Controller

### 1) eksctl로 EKS 클러스터에 대한 OIDC 자격 증명 공급자 생성

1. 클러스터에 대한 기존 IAM OIDC 공급자가 있는지 확인한다. 클러스터의 OIDC 공급자 URL을 본다.

    ```shell
    aws eks describe-cluster --name <cluster_name> --query "cluster.identity.oidc.issuer" --output text
    ```

    출력 예)

    ```shell
    https://oidc.eks.us-west-2.amazonaws.com/id/EXAMPLED539D4633E53DE1B716D3041E
    ```

    계정의 IAM OIDC 공급자를 나열한다. &lt;EXAMPLED539D4633E53DE1B716D3041E&gt; (포함 <>)을 이전 명령에서 반환된 값으로 바꿉니다.

    ```shell
    aws iam list-open-id-connect-providers | grep <EXAMPLED539D4633E53DE1B716D3041E>
    ```

    출력 예)

    ```shell
    "Arn": "arn:aws:iam::111122223333:oidc-provider/oidc.eks.us-west-2.amazonaws.com/id/EXAMPLED539D4633E53DE1B716D3041E"
    ```

    이전 명령에서 출력이 반환되면 클러스터에 대한 공급자가 이미 있는 것이다. 출력이 반환되지 않으면 IAM OIDC 공급자를 생성해야 합니다.

2. 다음 명령을 사용하여 클러스터에 대한 IAM OIDC 자격 증명 공급자를 생성한다. &lt;cluster_name&gt;(포함 <>)을 자신의 값으로 바꿉니다.

    ```shell
    eksctl utils associate-iam-oidc-provider --cluster <cluster_name> --approve
    ```

### 2) AWS 로드 밸런서 컨트롤러에 대한 IAM 정책 생성

1. 생성한 Amazon EKS 정책은 AWS 로드 밸런서 컨트롤러가 사용자를 대신하여 AWS API를 호출할 수 있도록 허용한다.

    ```shell
    curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json
    ```

2. 작업자 노드 인스턴스 프로파일에 대해 AWSLoadBalancerControllerIAMPolicy라는 이름의 IAM 정책을 생성하려면 다음 명령을 실행한다.

    ```shell
    aws iam create-policy \
        --policy-name AWSLoadBalancerControllerIAMPolicy \
        --policy-document file://iam-policy.json
    ```

3. eksctl을 사용하여 AWS 로드 밸런서 컨트롤러에 대해 새 IAM Role을 생성한다.

    ```shell
    eksctl create iamserviceaccount \
    --cluster EKS-Cluster \
    --namespace kube-system \
    --name aws-load-balancer-controller \
    --attach-policy-arn arn:aws:iam::${ACCOUNT_ID}:policy/AWSLoadBalancerControllerIAMPolicy \
    --override-existing-serviceaccounts \
    --approve
    ```

### 3) AWS AWSLoadBalancer Controller 생성

1. Public Subnet에 태그 지정

    이전 포스팅에서 생성한 Public Subnet 2개에 다음과 같은 태그를 달아준다.

    ![](https://images.velog.io/images/jsj3282/post/22ef980e-26af-453f-b4cc-b673e85f6508/%EC%82%AC%EC%A7%8442.png)

2. 인증서 구성을 웹훅에 삽입할 수 있도록 cert-manager를 설치

    ```shell
    kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/$VERSION/cert-manager.yaml
    ```

    참고: $VERSION을 배포하려는 cert-manager의 버전(Jetstack GitHub 사이트 참조)으로 바꾼다.

3. AWS GitHub에서 AWS 로드 밸런서 컨트롤러에 대해 다운로드한 매니페스트 파일에서 다음 명령을 실행한다.

    ```shell
    curl -o ingress-controller.yaml https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/$VERSION/docs/install/v2_1_0_full.yaml
    ```

    참고: $VERSION을 배포하려는 AWS 로드 밸런서 컨트롤러의 버전(Kubernetes SIGs GitHub 사이트 참조)으로 바꾼다.

4. 클러스터의 cluster-name을 편집한다. 예를 들면 다음과 같다.

    ```yaml
    spec:
        containers:
        - args:
            - --cluster-name=your-cluster-name # edit the cluster name
            - --ingress-class=alb
    ```

5. 위에서 eksctl로 이미 ServiceAccount를 생성했기 때문에 파일의 ServiceAccount 부분을 삭제한다.

    ```yaml
    apiVersion: v1
    kind: ServiceAccount
    metadata:
    labels:
        app.kubernetes.io/component: controller
        app.kubernetes.io/name: aws-load-balancer-controller
    name: aws-load-balancer-controller
    namespace: kube-system
    ```

    이 부분을 파일에서 삭제한다.

6. AWS 로드 밸런서 컨트롤러를 배포하려면 다음 명령을 실행한다.

    ```shell
    kubectl apply -f ingress-controller.yaml
    ```

### 5) EKS Ingress 생성

1. NodePort, Deployment를 배포한다.

    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
    name: ingress-test
    spec:
    selector:
        app: nginx
    ports:
        - protocol: TCP
        port: 80
        targetPort: 80
        nodePort: 30010
    type: NodePort

    ---
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    name: ingress-test
    labels:
        app: nginx
    spec:
    replicas: 3
    selector:
        matchLabels:
        app: nginx
    template:
        metadata:
        labels:
            app: nginx
        spec:
        containers:
        - name: nginx
            image: nginx
            ports:
            - containerPort: 80
    ```

    여기서 NodePort가 ALB TargetGroup의 역할을 하고 Deployment는 배포되는 애플리케이션 이미지를 지정해주면 된다.

    이를 배포하려면 다음 명령을 실행한다.

    ```shell
    kubectl apply -f nginx.yaml
    ```

2. Ingress를 배포한다.

    ALB를 생성하기 위해 Ingress를 배포한다. ingress.yaml은 다음과 같다.

    ```yaml
    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
    name: "ingress"
    annotations:
        kubernetes.io/ingress.class: alb # the value we set in alb-ingress-controller
        alb.ingress.kubernetes.io/scheme: internet-facing
        alb.ingress.kubernetes.io/subnets: subnet-04bdd6a509acee4bc,subnet-05bab6ef226b681c6
    spec:
    rules:
        - http:
            paths:
            - path: /*
                backend:
                serviceName: "ingress-test"
                servicePort: 80
    ```

    이를 배포하려면 다음 명령을 실행한다.

    ```shell
    kubectl apply -f ingress.yaml
    ```

    여기까지 완료하면 다음과 같은 화면이 보인다.

    ![EKS-32]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-2/images_jsj3282_post_a2bf89e6-fe02-4d7a-a338-5efb9af207b8_사진44.png)

    ![EKS-33]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-2/images_jsj3282_post_b17b3495-dced-4cef-a10f-4e05f296c142_사진45.png)

참고
- [https://aws.amazon.com/ko/premiumsupport/knowledge-center/eks-alb-ingress-controller-setup/](https://aws.amazon.com/ko/premiumsupport/knowledge-center/eks-alb-ingress-controller-setup/)
- [https://docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html](https://docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html)
- [https://www.eksworkshop.com/beginner/130_exposing-service/ingress_controller_alb/](https://www.eksworkshop.com/beginner/130_exposing-service/ingress_controller_alb/)
- [https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/alb-ingress.html](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/alb-ingress.html)