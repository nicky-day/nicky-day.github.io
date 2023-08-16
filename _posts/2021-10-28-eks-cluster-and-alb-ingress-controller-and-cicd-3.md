---
layout: single
title: "EKS Cluster 생성 + ALB Ingress Controller + CI/CD(3)"
categories: AWS
tag: [EKS]
author_profile: false
sidebar:
    nav: "counts"
search: true
use_math: false
---

## 3. EKS CI/CD

### 1) ECR 생성

![EKS-35]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-3/images_jsj3282_post_b25a8f4e-a986-46d1-a1eb-722df5083115_사진46.png)

도커 이미지 저장소용 ECR을 하나 만든다.

### 2) CodeBuild IAM Role 생성

Permissions
- AmazonEc2ContainerRegistryPowerUser(AWS managed policy)
- EKS-ACCESS(Inline Policy)
  
```json
{    
    "Version": "2012-10-17",    
    "Statement": [        
        {            
            "Sid": "VisualEditor0",            
            "Effect": "Allow",            
            "Action": "eks:DescribeCluster",            
            "Resource": "*"        
        }    
    ]
}
```

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

### 3) EKS 배포 권한 설정

위에서 생성한 CodeBuild Role 권한으로 EKS에 접근 가능해야 한다.

다음 aws-auth.yaml 파일에 CodeBuild 역할을 추가하여 EKS에 권한을 추가한다.

```yaml
apiVersion: v1
data:
  mapRoles: |
    - groups:
      - system:bootstrappers
      - system:nodes
      rolearn: arn:aws:iam::${ACCOUNT_ID}:role/myAmazonEKSNodeRole
      username: system:node:{{EC2PrivateDNSName}}
    - groups:
        - system:masters
      rolearn: arn:aws:iam::${ACCOUNT_ID}:role/codebuild-EKS-TEST-DEPLOY-service-role
      username: codebuild-EKS-TEST-DEPLOY-service-role
kind: ConfigMap
metadata:
  annotations:
  name: aws-auth
  namespace: kube-system
  uid: 13ec8417-861a-46c8-bef7-76cf39d9142b
```

**주의사항** : CodeBuildRole은 Service Role이지만, aws-auth.yaml을 수정할 때 rolearn에 arn:aws:iam::{ACCOUNT_ID}:role/service-role/codebuild-EKS-TEST-DEPLOY-service-role이 아닌 arn:aws:iam::{ACCOUNT_ID}:role/codebuild-EKS-TEST-DEPLOY-service-role로 입력해야 제대로 인식된다.

### 4) AWS CodeCommit 생성 및 파일 업로드

![EKS-36]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-3/images_jsj3282_post_07d44888-b6e8-4aa8-80f7-bf8c8049c717_사진47.png)

 buildspec.yml

```yaml
version: 0.2

phases:
  install:
    runtime-versions:
      docker: 19
    commands:
      - echo Install Kubectl
      - echo ---------------------------------
      - curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
      - chmod +x ./kubectl
      - mv ./kubectl /usr/local/bin/kubectl
      - mkdir ~/.kube
      - aws sts get-caller-identity
      - aws eks --region ap-northeast-2 update-kubeconfig --name EKS-Cluster
      - echo ---------------------------------
  pre_build:
    commands:
      - echo ENV Values
      - echo ---------------------------------
      - echo $AWS_DEFAULT_REGION
      - echo $AWS_ACCOUNT_ID
      - echo $IMAGE_REPO_NAME
      - echo $IMAGE_TAG
      - echo $CODEBUILD_BUILD_NUMBER
      - echo ---------------------------------
      - echo Logging in to Amazon ECR...
      - aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com
  build:
    commands:
      - echo Build started on `date`
      - echo Application Build
      - echo ---------------------------------
      - echo GO Build Start
      - echo Go Build Code In here
      - echo GO Build Stop
      - echo ---------------------------------
      - echo Building the Docker image...   
      - echo ---------------------------------    
      - docker build -t $IMAGE_REPO_NAME:$IMAGE_TAG-$CODEBUILD_BUILD_NUMBER .
      - docker tag $IMAGE_REPO_NAME:$IMAGE_TAG-$CODEBUILD_BUILD_NUMBER $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG-$CODEBUILD_BUILD_NUMBER
      - echo ---------------------------------
      - echo Pushing the Docker image...
      - echo ---------------------------------
      - docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG-$CODEBUILD_BUILD_NUMBER
  post_build:
    commands:
      - AWS_ECR_URI=$AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG-$CODEBUILD_BUILD_NUMBER
      - DATE=`date`
      - echo Build completed on $DATE
      - sed -i 's#AWS_ECR_URI#'"$AWS_ECR_URI"'#' ./eks-app-deploy.yml
      - kubectl apply -f ./eks-app-deploy.yml
```

install : 실행 환경 지정 및 kubectl 설치, .kubeconfig 구성
pre_build : Docker ECR 저장소 로그인
build : 빌드 작업 수행, Docker Build(이미지 생성) 및 태그 지정, ECR Push(이미지 업로드)
post_build : EKS 배포 수행


Dockerfile

```docker
FROM nginx:latest

COPY ./index.html /usr/share/nginx/html/index.html
```

eks-app-deploy.yml

```yaml
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
        image: AWS_ECR_URI
        ports:
        - containerPort: 80
```

index.html

```html
Nginx Test<br>
version : 1
```

### 5) AWS CodeCommit Branch 생성

![EKS-37]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-3/images_jsj3282_post_03a9941b-bbbc-41a0-a6ba-53d3686e5fce_사진48.png)

dev Branch를 생성한다.

### 6) AWS CodeBuild 프로젝트 생성

![EKS-38]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-3/images_jsj3282_post_1edf78bf-9dcf-4a9e-8358-a99eab48453a_사진49.png)

![EKS-39]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-3/images_jsj3282_post_474cc68f-ddfd-4cd8-bb40-85b1006aafd6_사진50.png)

![EKS-40]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-3/images_jsj3282_post_5afac1eb-d460-4a08-96ec-a992da38feb0_사진52.png)

위에서 생성한 CodeBuild Role을 선택한다.

![EKS-41]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-3/images_jsj3282_post_820419d6-b1aa-49e6-8c72-405fac789606_사진53.png)

다음과 같이 환경 변수를 설정한다.

### 6) AWS CodePipeline 생성

![EKS-42]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-3/images_jsj3282_post_332fb8c2-b971-4172-bb33-c8f8f647db62_사진54.png)

![EKS-43]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-3/images_jsj3282_post_e0e4dc3a-0440-4454-be4f-f81600f2e4ce_사진55.png)

![EKS-44]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-3/images_jsj3282_post_daa60916-e25f-4c72-bbaf-ae34dc4b4412_사진56.png)

![EKS-45]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-3/images_jsj3282_post_85be46e4-f087-4e94-883d-e0483413d81b_사진57.png)

이미 CodeBuild에서 배포를 수행하므로 배포 스테이지는 건너뛴다.

![EKS-46]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-3/images_jsj3282_post_2dcf8af9-1f64-4434-aadd-53db273e2952_사진58.png)

파이프라인이 잘 실행되면 다음과 같다.

![EKS-46]({{site.url}}/images/2021-10-28-eks-cluster-and-alb-ingress-controller-and-cicd-3/images_jsj3282_post_81c8b568-9f86-47d2-bc55-cd6986eb37cf_사진60.png)

또한 ALB DNS 주소로 접속하면 다음과 같은 화면을 볼 수 있다.