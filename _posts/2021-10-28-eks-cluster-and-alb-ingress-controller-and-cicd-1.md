---
layout: single
title: "EKS Cluster 생성 + ALB Ingress Controller + CI/CD(1)"
categories: AWS
tag: [EKS]
author_profile: false
sidebar:
    nav: "counts"
search: true
use_math: false
---

# EKS Cluster 생성  + ALB Ingress Controller + CI/CD(1)

## 1. EKS Cluster 생성

### 1) EKS 클러스터 IAM User 생성

![EKS-1]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-1/images_jsj3282_post_79d55b84-99de-40de-a64d-9671178bdc09_사진1.png)

클러스터를 생성할 IAM User를 생성한다.

![EKS-2]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-1/images_jsj3282_post_ac764815-5b5e-4499-8399-d153bb8fba9c_사진2.png)

나의 경우에는 Admin 권한으로 IAM User를 생성해줬다.

그 후에는 위에서 생성한 IAM User로 로그인한다.

### 2) EKS 클러스터 VPC 세팅

#### 1. VPC 생성

![EKS-3]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-1/images_jsj3282_post_4e1fe36f-2244-4bbf-96b2-8b6467d69432_사진5.png)

#### 2. Subnet 생성

![EKS-4]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-1/images_jsj3282_post_665c645d-08d6-4853-a4be-527080e14dc1_사진7.png)

![EKS-5]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-1/images_jsj3282_post_ca8e5588-6e17-40ad-8bfa-9c175018ac01_사진8.png)

![EKS-6]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-1/images_jsj3282_post_29e6845f-50c1-4f51-80fd-07ea8cbcace3_사진9.png)

Public Subnet 2개, Private Subnet 2개를 만들어준다.

![EKS-7]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-1/images_jsj3282_post_b61ed124-551b-4f5b-a0af-6168cab104c3_사진10.png)

![EKS-8]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-1/images_jsj3282_post_3d224949-f193-4ce3-b6cf-3918c1421d62_사진11.png)

Public Subnet에는 퍼블릭 IPv4 주소를 자동 할당 활성화 시켜준다.

#### 3. Internet Gateway Attach

![EKS-9]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-1/images_jsj3282_post_1ba69cb8-a083-496c-a95a-dea0d20ef816_사진12.png)

![EKS-10]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-1/images_jsj3282_post_e9987779-e39f-4550-a491-617565b10bd5_사진13.png)

위에서 생성한 VPC에 연결해준다.

#### 4. NAT Gateway Attach

![EKS-11]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-1/images_jsj3282_post_c903b883-cba6-440e-b653-9359c3af2d38_사진14.png)

NAT Gateway를 생성할 때는 위에서 생성한 퍼블릭 서브넷 중 하나를 선택한다.

#### 5. Routing Table 설정

**Public Routing Table**

![EKS-12]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-1/images_jsj3282_post_08b2bf41-9e86-48aa-a801-e66d475a31c0_사진15.png)

![EKS-13]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-1/images_jsj3282_post_8d0b6408-4923-4ef1-9763-76671aeb2b1e_사진16.png)

서브넷 연결 편집을 눌러 위에서 생성한 퍼블릭 서브넷 2개를 연결시켜준다.

![EKS-14]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-1/images_jsj3282_post_195d847b-de4b-4bad-b602-fcc8e308d954_사진17.png)

위에서 생성한 Internet Gateway 0.0.0.0/0으로 라우팅 추가해준다.

**Private Routing Table**

![EKS-15]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-1/images_jsj3282_post_5acd94e9-29b4-4038-aa18-0f3b0c909ec3_사진18.png)

![EKS-16]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-1/images_jsj3282_post_307e55f7-fe36-4bd6-8937-da4694b090a2_사진19.png)

서브넷 연결 편집을 눌러 위에서 생성한 프라이빗 서브넷 2개를 연결시켜준다.

![EKS-17]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-1/images_jsj3282_post_9fa93892-64e0-4db9-a84e-1a031bc9199e_사진20.png)

위에서 생성한 NAT Gateway 0.0.0.0/0으로 라우팅 추가해준다.

### 3) EKS 클러스터 및 노드그룹 IAM Role 생성

#### 1. EKS Cluster IAM Role 생성

![EKS-18]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-1/images_jsj3282_post_a979c6e7-2179-4516-b5a0-0cdbd503ce6e_사진26.png)

Permissions
- AmazonEKSClusterPolicy

Trust relationships

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "eks.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

#### 2. EKS 노드그룹 IAM Role 생성

![EKS-19]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-1/images_jsj3282_post_f1ae4f01-a272-4844-80b7-ff4eeee35ad3_사진25.png)

Permissions
- AmazonEKSWorkerNodePolicy
- AmazonEC2ContainerRegistryReadOnly
- AmazonEKS_CNI_Policy

Trust relationships

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "eks.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

### 4) EKS 클러스터 생성

![EKS-20]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-1/images_jsj3282_post_aba59171-2b74-4d21-88ae-60323b035ffb_사진27.png)

위에서 생성한 Cluster IAM Role을 선택한다.

![EKS-21]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-1/images_jsj3282_post_edbb4491-d7d8-4bc7-9322-b53837c075f4_사진28.png)

위에서 생성한 VPC와 프라이빗 서브넷들을 선택하고 보안그룹은 그대로 두면 기본 보안 그룹이 생성된다. 또한 클러스터 엔드포인트 액세스는 퍼블릭 및 프라이빗으로 설정한다.

### 5) EKS Node Group 생성

#### 1. EKS 노드그룹 보안그룹 생성

[https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/sec-group-reqs.html](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/sec-group-reqs.html)

다음 설명서에 나와있는 것처럼 EKS 노드 보안 그룹을 생성한다.

![EKS-22]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-1/images_jsj3282_post_1f39853b-1dca-4d08-ae55-a033a8e93642_사진31.png)

#### 2. EKS 노드그룹 생성을 위한 Launch templates 생성

![EKS-23]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-1/images_jsj3282_post_5875c289-1f5d-4fe5-9100-cd172c492d45_사진30.png)

해당 이미지는 Public AMI이고 EKS 버전에 맞는 기본 이미지를 선택하면 된다.

![EKS-24]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-1/images_jsj3282_post_afdc7d05-6364-4899-ab9e-f49d8eb22d39_사진32.png)

키 페어를 선택하고 위에서 생성한 보안그룹을 선택해준다.

```shell
MIME-Version: 1.0
Content-Type: multipart/mixed; boundary="//"

--//
Content-Type: text/x-shellscript; charset="us-ascii"
#!/bin/bash
set -ex
B64_CLUSTER_CA=[EKS Cluster Certificate authority]
API_SERVER_URL=[EKS Cluster API server endpoint]
K8S_CLUSTER_DNS_IP=[EKS Cluster DNS IP]
/etc/eks/bootstrap.sh [EKS Cluster Name] --kubelet-extra-args '--node-labels=eks.amazonaws.com/nodegroup-image=[EKS WorkerNode Base Image ID],eks.amazonaws.com/capacityType=ON_DEMAND,eks.amazonaws.com/nodegroup=[생성할 NodeGroup Name]' --b64-cluster-ca $B64_CLUSTER_CA --apiserver-endpoint $API_SERVER_URL --dns-cluster-ip $K8S_CLUSTER_DNS_IP

--//--

```

사용자 데이터에 다음과 같은 내용을 입력하고 시작 템플릿 생성을 완료한다.

#### 3. EKS 노드그룹 생성

![EKS-25]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-1/images_jsj3282_post_06d259f8-e13d-4cb4-b39c-708c7e587419_사진34.png)

EKS Cluster에서 구성 -> 컴퓨팅 -> 노드 그룹 추가를 선택합니다.

![EKS-26]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-1/images_jsj3282_post_889a0a88-5a85-45b4-a748-ffdf3c40f666_사진33.png)

위에서 생성한 NodeGroup IAM Role을 선택합니다.

![EKS-27]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-1/images_jsj3282_post_1d5891ee-81fa-4f92-b151-8bb13b5b071c_사진35.png)

위에서 생성한 NodeGroup Template을 선택합니다.

![EKS-28]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-1/images_jsj3282_post_84d75e05-de99-4905-a500-e8d2e0ace8d9_사진36.png)

상황에 맞게 크기를 선택합니다.

![EKS-29]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-1/images_jsj3282_post_f38a46a2-99eb-4b93-8cf8-fc2b58520891_사진37.png)

위에서 생성한 프라이빗 서브넷 2개를 선택합니다.

### 5) EC2 MGMT 서버 생성

![EKS-30]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-1/images_jsj3282_post_209652d8-475a-4b43-a6ab-5ad3ed834e80_사진38.png)

퍼블릭 서브넷 중 하나를 선택해서 EC2를 생성합니다.

[https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/sec-group-reqs.html](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/sec-group-reqs.html) 에서 볼 수 있듯이 EKS 클러스터 보안그룹에 MGMT EC2에서 접근 가능하도록 다음 인바운드 규칙을 추가해줍니다.

![EKS-31]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-1/images_jsj3282_post_e1fb8619-ee25-4bd2-8fa9-be0f64e5343d_사진40.png)

EC2 서버에 접속해 다음과 같은 설정을 해줍니다.

```shell
# 1. kubectl 설치
$ aws configure   # 위에서 생성한 IAM USER Access_Key, Secret_Key 설정
$ curl –o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
$ chmod +x ./kubectl
$ mkdir –p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin
$ kubectl version –short –client

# 2. kubeconfig 설정
$ aws sts get-caller-identity
$ aws eks –region [region code] update-kubeconfig –name [eks cluster name]
$ kubectl get svc
```

참고문서
- [https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/create-cluster.html](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/create-cluster.html)
- [https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/sec-group-reqs.html](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/sec-group-reqs.html)