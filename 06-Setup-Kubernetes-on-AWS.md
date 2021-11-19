# Setup Kubernetes on AWS

kOps를 사용하여 AWS에 Kubernetes Cluster 구축합니다.

## kubectl 설치

쿠버네티스 커맨드 라인 도구인 [kubectl](https://kubernetes.io/ko/docs/reference/kubectl/kubectl/)을 사용하면, 쿠버네티스 클러스터에 대해 명령을 실행할 수 있습니다. kubectl을 사용하여 애플리케이션을 배포하고, 클러스터 리소스를 검사 및 관리하며 로그를 볼 수 있습니다. kubectl 작업의 전체 목록에 대해서는, [kubectl 개요](https://kubernetes.io/ko/docs/reference/kubectl/overview/)를 참고하십시오.

Windows 시스템에서 설치는 [윈도우에 kubectl 설치](https://kubernetes.io/ko/docs/tasks/tools/install-kubectl/#%EC%9C%88%EB%8F%84%EC%9A%B0%EC%97%90-kubectl-%EC%84%A4%EC%B9%98) 문서를 참고하십시오.

### 리눅스에 kubectl 설치

#### 리눅스에서 curl을 사용하여 kubectl 바이너리 설치

1. 다음 명령으로 최신 릴리스를 다운로드합니다.

    ```bash
    curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
    ```

    특정 버전을 다운로드하려면, `$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)` 명령 부분을 특정 버전으로 바꿉니다.

    예를 들어, 리눅스에서 버전 v1.20.0을 다운로드하려면, 다음을 입력합니다.

    ```bash
    curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.20.0/bin/linux/amd64/kubectl
    ```

2. kubectl 바이너리를 실행 가능하게 만듭니다.

    ```bash
    chmod +x ./kubectl
    ````

3. 바이너리를 PATH가 설정된 디렉터리로 옮깁니다.

    ```bash
    sudo mv ./kubectl /usr/local/bin/kubectl
    ```

### macOS에 kubectl 설치

#### macOS에서 curl을 사용하여 kubectl 바이너리 설치

리눅스에서 설치하는 방식과 동일합니다.

#### Homebrew를 사용하여 kubectl 설치 또는 업그레이드

macOS에서 [Homebrew](https://brew.sh/) 패키지 관리자를 사용하는 경우, Homebrew로 kubectl을 설치할 수 있습니다.

```bash
brew install kubectl
```

```bash
brew upgrade kubectl
```

### kubectl 버전 확인

정상적으로 설치되었는지 버전을 확인합니다.

```bash
kubectl version --client
```

또는

```bash
kubectl version --short --client
```

## kubens 설치

<!-- kubectx는 kubectl 컨텍스트를 관리하고 전환하는 유틸리티 입니다.  -->

kubens는 Kubernetes 네임스페이스 사이를 전환하는 유틸리티 입니다.

### 리눅스에 kubens 설치

```bash
curl -sLo kubens.tar.gz https://github.com/ahmetb/kubectx/releases/download/v0.9.3/kubens_v0.9.3_linux_x86_64.tar.gz
tar -xvf kubens.tar.gz
chmod +x kubens
sudo mv kubens /usr/local/bin/kubens
```
<!--
kubectx도 비슷한 방식으로 설치하면 됩니다.
-->

### macOS에 kubens 설치 ( kubectx 에 포함 )

```bash
brew install kubectx
```

### kubens 사용법

```bash
kubens                    : list the namespaces
kubens <NAME>             : change the active namespace
kubens -                  : switch to the previous namespace
kubens -c, --current      : show the current namespace
```

## kops 설치

kOps(Kubernetes Operations)는 커맨드 라인에서 프로덕션 레벨의 고가용성 Kubernetes 클러스터를 생성, 파기(destroy), 업그레이드 및 유지 관리하는 데 도움이 되는 오픈소스 라이선스(Apache License 2.0) 도구입니다.
AWS(Amazon Web Services)는 현재 공식적으로 지원되며, GCE 및 OpenStack은 베타 지원, VMware vSphere는 알파이고, 기타 플랫폼이 계획되어 있습니다.

Windows 시스템에서 설치는 [윈도우에 kops 설치](https://kops.sigs.k8s.io/getting_started/install/#windows) 문서를 참고하십시오.

### 리눅스에 kops 설치

#### 리눅스에서 curl을 사용하여 kops 바이너리 설치

1. 다음 명령으로 최신 릴리스를 다운로드합니다.

    ```bash
    curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
    ```

    특정 버전의 kops를 다운로드하려면, `$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)` 명령 부분을 특정 버전으로 변경합니다.

    예를 들어, kops 버전을 v1.18.2을 다운로드 하려면 다음을 입력합니다.

    ```bash
    curl -LO  https://github.com/kubernetes/kops/releases/download/v1.18.2/kops-linux-amd64
    ```

2. kops 바이너리를 실행 가능하게 만듭니다.

    ```bash
    chmod +x kops-linux-amd64
    ````

3. 바이너리를 PATH가 설정된 디렉터리로 옮깁니다.

    ```bash
    sudo mv kops-linux-amd64 /usr/local/bin/kops
    ```

### macOS에 kops 설치

#### Homebrew를 사용하여 kops 설치

```bash
brew update && brew install kops
```

#### macOS에서 curl을 사용하여 kops 바이너리 설치

1. 다음 명령으로 최신 릴리스를 다운로드합니다.

    ```bash
    curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-darwin-amd64
    ```

    특정 버전의 kops를 다운로드하려면, `$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)` 명령 부분을 특정 버전으로 변경합니다.

    예를 들어, kops 버전을 v1.15.0을 다운로드 하려면 다음을 입력합니다.

    ```bash
    curl -LO  https://github.com/kubernetes/kops/releases/download/1.15.0/kops-darwin-amd64
    ```

2. kops 바이너리를 실행 가능하게 만듭니다.

    ```bash
    chmod +x kops-darwin-amd64
    ````

3. 바이너리를 PATH가 설정된 디렉터리로 옮깁니다.

    ```bash
    sudo mv kops-darwin-amd64 /usr/local/bin/kops
    ```

### kops 버전 확인

```bash
kops version
```

## AWS 환경 구성

kops에 대한 AWS 계정을 올바르게 준비하려면, AWS CLI 도구를 설치한 다음, 본인 IAM 계정에 대한 보안 자격 증명이 구성되어 있어야 합니다.  
이 계정은 kops에 대한 새 IAM 계정을 생성할 수 있는 권한(IAMFullAccess)이 있어야 합니다.

AWS CLI를 설치하지 않았다면 [AWS CLI 설치 및 구성](12.1-Setup-AWS-CLI.md) 문서를 참고하여 AWS CLI를 설치하고 보안 자격 증명을 구성합니다.

### IAM User 구성

kops 전용 User는 아래 권한(IAM permissions)이 필요합니다.

```text
AmazonEC2FullAccess
AmazonRoute53FullAccess
AmazonS3FullAccess
IAMFullAccess
AmazonVPCFullAccess
```

#### IAM Group 생성

```bash
aws iam create-group --group-name kops-group
```

#### kops-group 그룹에 관리형 정책(managed policies) 연결

```bash
aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess --group-name kops-group
aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonRoute53FullAccess --group-name kops-group
aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess --group-name kops-group
aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/IAMFullAccess --group-name kops-group
aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonVPCFullAccess --group-name kops-group
```

#### IAM User 생성

```bash
aws iam create-user --user-name kops
```

#### IAM 그룹에 사용자 추가

```bash
aws iam add-user-to-group --user-name kops --group-name kops-group
```

#### Access key 생성

Access key 생성 시 JSON 출력에서 AccessKeyID 및 SecretAccessKey를 기록합니다.

```bash
$ aws iam create-access-key --user-name kops
{
    "AccessKey": {
        "UserName": "kops",
        "AccessKeyId": "AKIAR2JHQKIN2EXMPLE",
        "Status": "Active",
        "SecretAccessKey": "siYaSx8RUvoQEXAMPLEReWr55dWZKqi4voS8xB0Iz",
        "CreateDate": "2021-09-09T01:40:40Z"
    }
}
```

### AWS CLI 자격 증명(credentials) 구성

기록해 놓은 kops 계정의 access key, secret access key와 AWS Region, output format를 입력합니다.
이미 등록한 자격 증명이 있으면 --profile 옵션을 사용하여 aws configure 명령을 실행합니다.

```bash
$ aws configure --profile kops
AWS Access Key ID [None]: AKIAR2JHQKIN2EXMPLE
AWS Secret Access Key [None]: siYaSx8RUvoQEXAMPLEReWr55dWZKqi4voS8xB0Iz
Default region name [None]: ap-northeast-2
Default output format [None]: json
```

### AWS CLI 자격 증명 구성 확인

aws configure로 구성된 두 개의 프로파일이 있는 credentials 및 config 파일은 다음과 유사합니다.

```bash
$ vi ~/.aws/credentials

[default]
aws_access_key_id = AKIAIOSFODNN7EXAMPLE
aws_secret_access_key = wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
[kops]
aws_access_key_id = AKIAR2JHQKIN2EXMPLE
aws_secret_access_key = siYaSx8RUvoQEXAMPLEReWr55dWZKqi4voS8xB0Iz
```

```bash
$ vi ~/.aws/config

[default]
region = ap-northeast-2
output = json
[profile kops]
region = ap-northeast-2
output = json
```

## DNS 구성

kops로 Kubernetes 클러스터를 구축하려면 필요한 DNS 레코드를 구축할 수 있는 곳을 준비해야 합니다.  
실습에서는 [Gossip 기반 DNS](https://github.com/kubernetes/kops/blob/master/docs/gossip.md)를 사용하여 클러스터를 구축합니다.

AWS를 통해 구매/호스팅 된 DNS를 사용하려면 [DNS 구성](https://github.com/kubernetes/kops/blob/master/docs/getting_started/aws.md#configure-dns) 문서를 참고하여 구성합니다.

## 클러스터 상태 저장용 S3 버킷 구성

클러스터의 상태와 클러스터 표현을 저장하려면 `kops`가 사용할 전용 S3 버킷을 만들어야 합니다. 이 버킷은 클러스터 구성의 정보 소스가 됩니다. 이 가이드에서는 이 버킷을 `example-com-state-store`라고 부르지만 버킷 이름은 고유해야 하므로 **사용자 지정 접두사**를 추가해야 합니다.

### API 수준(s3api) 명령을 사용하여 S3 버킷 생성

```bash
aws s3api create-bucket \
    --profile kops \
    ##--bucket devops-skcc-devops01-state-store \
    --bucket prefix-example-com-state-store \
    --create-bucket-configuration LocationConstraint=ap-northeast-2 \
    --region ap-northeast-2
```

> 참고 : `us-east-1` 이외의 리전에서는 `--create-bucket-configuration LocationConstraint=<region>` 옵션이 필요합니다.

AWS S3 버킷 이름은 전역적으로 고유해야 합니다. 이미 사용 중인 버킷 이름으로 create-bucket 명령을 실행하면 아래와 같은 오류가 발생합니다.

```bash
An error occurred (BucketAlreadyExists) when calling the CreateBucket operation: The requested bucket name is not available. The bucket namespace
is shared by all users of the system. Please select a different name and try again.
```

[head-bucket AWS CLI](https://docs.aws.amazon.com/cli/latest/reference/s3api/head-bucket.html) 명령을 실행하여 해당 이름의 버킷이 이미 있는지 확인할 수 있습니다.

### API 수준(s3api) 명령을 사용하여 S3 버킷 버전 관리(Versioning) 기능 활성화

```bash
aws s3api put-bucket-versioning \
    --profile kops \
    #--bucket devops-skcc-devops01-state-store \
    --bucket prefix-example-com-state-store \
    --versioning-configuration Status=Enabled
```

### S3 기본 버킷 암호화 사용

kops는 S3 버킷에서 상태를 암호화하기 위해 기본 버킷 암호화(default bucket encryption)를 지원합니다. 이렇게 하면, 버킷에 설정된 기본 서버 측 암호화(default server side encryption)가 kOps 상태에도 사용됩니다. 예를 들어, 기본적으로 작성된 모든 객체를 쉽게 암호화하거나 규정 준수를 위해 특정 암호화 키(KMS, CMK)를 사용해야 할 때, 이 AWS 기능을 사용할 수 있습니다.

S3 버킷에 기본 암호화가 설정되어 있으면 kOps가 이를 사용합니다.

```bash
aws s3api put-bucket-encryption \
    --profile kops \
    #--bucket devops-skcc-devops01-state-store \
    --bucket prefix-example-com-state-store \
    --server-side-encryption-configuration '{"Rules":[{"ApplyServerSideEncryptionByDefault":{"SSEAlgorithm":"AES256"}}]}'
```

기본 암호화가 설정되어 있지 않거나 확인할 수 없는 경우, kOps는 [Amazon S3 관리형 암호화 키(SSE-S3)](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingServerSideEncryption.html)와 함께 서버 측 AES256 버킷 암호화를 사용합니다.

## Kubernetes Cluster 생성

### KOPS 로컬 환경변수 추가

.bash_profile, .bashrc 또는 .zshrc에 환경변수를 추가합니다.

```bash
$ vi ~/.bashrc

#export KOPS_CLUSTER_NAME=devops.skcc-devopsXX.k8s.local
#export KOPS_STATE_STORE=s3://devops-skcc-devopsXX-state-store
export KOPS_CLUSTER_NAME=<your_cluster_name>.k8s.local
export KOPS_STATE_STORE=s3://devops-skcc-devops<내 번호>-state-store
export AWS_PROFILE=kops
```

```bash
source ~/.bashrc
```

### SSH Key Pair 생성

```bash
$ cd ~/.ssh
$ ssh-keygen -t rsa -b 2048
ubuntu@workspace:~/.ssh$ ssh-keygen -t rsa -b 2048
Generating public/private rsa key pair.
Enter file in which to save the key (/home/ubuntu/.ssh/id_rsa): id_rsa_k8s
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in id_rsa_k8s
Your public key has been saved in id_rsa_k8s.pub
The key fingerprint is:
SHA256:MEZ4DnviPKz9iuVlCFW+Y6wtUZ5ABZd/1YqZBaUKAbo ubuntu@workspace
The key's randomart image is:
+---[RSA 2048]----+
|    +=*.  .o..   |
|   oo=o.   .o .  |
|  . o=B.  .* .   |
|   oo*o*..= .    |
|  E+.oB S.       |
|   .=* .         |
|   o=.+          |
|  .+.+           |
|  . oo.          |
+----[SHA256]-----+
$ ls -al
total 28
drwx------  2 ubuntu ubuntu 4096 Sep  9 11:06 ./
drwxr-xr-x 32 ubuntu ubuntu 4096 Sep  9 11:04 ../
-rw-------  1 ubuntu ubuntu  790 Aug 25 15:06 authorized_keys
-rw-------  1 ubuntu ubuntu 1823 Sep  9 11:06 id_rsa_k8s
-rw-r--r--  1 ubuntu ubuntu  398 Sep  9 11:06 id_rsa_k8s.pub
-r--------  1 ubuntu ubuntu 1700 Sep  7 11:01 key-tools.pem
-rw-r--r--  1 ubuntu ubuntu  222 Sep  7 11:04 known_hosts
```

### Ubuntu 20.04 LTS (Focal Fossa) AMI 검색

```bash
aws ec2 describe-images --region ap-northeast-2 --output table \
    --owners 099720109477 \
    --filters "Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-*" "Name=state,Values=available" \
    --query "reverse(sort_by(Images, &CreationDate)[*].[CreationDate,Name,ImageId])"
```

실습에 사용할 이미지는 `ami-04876f29fd3a5e8ba` 선택합니다. 

|<img src="images/machine-image-id.png" width="700"/> |
| -------------------------------------------------------------------------- |

### Cluster 구성 만들기

아래 명령을 실행하여 사용할 수 있는 가용 영역을 확인합니다.

```bash
aws ec2 describe-availability-zones --region ap-northeast-2
```

아래 명령은 `create cluster` 명령으로, 클러스터 구성을 생성하지만 바로 클러스터를 만들지는 않습니다. 클러스터를 만들기 전에 SSH 키 쌍을 생성했는지 확인합니다.

```bash
kops create cluster \
    --cloud aws \
    --name ${KOPS_CLUSTER_NAME} \
    --state ${KOPS_STATE_STORE} \
    --zones ap-northeast-2a,ap-northeast-2c \
    --topology public \
    --networking calico \
    --ssh-public-key /home/ubuntu/.ssh/id_rsa_k8s.pub \
    --image ami-04876f29fd3a5e8ba \
    --master-count 1 \
    --master-size t3.medium \
    --master-zones ap-northeast-2a \
    --node-count 2 \
    --node-size t3.large
```

`kops`로 생성한 모든 인스턴스는 ASG(Auto Scaling Group) 내에서 구축됩니다. 즉, 장애가 발생할 경우 AWS에서 각 인스턴스를 자동으로 모니터링하고 재 구축합니다.

### SSH 액세스

SSH는 기본적으로 어디에서나 Master와 Node에 허용됩니다.

SSH (및 HTTPS)에 액세스 할 수 있는 CIDR을 변경하려면 클러스터 사양에서 AdminAccess를 설정합니다.

기본 이미지를 사용하는 경우 SSH username은 `admin`이고, SSH private key는 `kops get secrets --type sshpublickey admin`의 public key에 해당하는 private key가 됩니다. 새 클러스터를 만들 때 SSH 공개 키는 `--ssh-public-key` 옵션으로 지정할 수 있으며, 기본값은 `~/.ssh/id_rsa.pub` 입니다.

> 참고 : Flatcar에서 SSH 사용자 이름은 `core` 입니다.

새 SSH 공개 키를 만들고 상태 저장소에 키를 저장합니다.

```bash
kops create secret \
    --name ${KOPS_CLUSTER_NAME} \
    sshpublickey admin -i ~/.ssh/id_rsa_k8s.pub
```

기존 클러스터에서 SSH 공개 키를 변경하려면 :

```bash
kops delete secret --name ${KOPS_CLUSTER_NAME} sshpublickey admin
kops create secret --name ${KOPS_CLUSTER_NAME} sshpublickey admin -i ~/.ssh/newkey.pub
kops update cluster --yes # 자동 확장(auto-scaling) 그룹을 재구성하려면
kops rolling-update cluster --name ${KOPS_CLUSTER_NAME} --yes # 모든 머신를 즉시 롤링하여 새 키를 갖도록하려면 (선택 사항)
```

### Cluster 구성 편집

이제 클러스터 구성이 있으므로 명세을 편집하여 클러스터를 정의하는 모든 측면을 볼 수 있습니다.

```bash
kops edit cluster --name ${KOPS_CLUSTER_NAME}
```

### Cluster 구축

이제 실제로 클러스터를 구축하는 마지막 단계를 수행합니다. 시간이 좀 걸립니다. 완료되면 부팅된 인스턴스가 Kubernetes 구성 요소 다운로드를 완료하고 "준비" 상태에 도달하는 동안 더 오래 기다려야 합니다.

아래 명령을 실행하여 클러스터를 실제로 구축합니다.

```bash
kops update cluster --name ${KOPS_CLUSTER_NAME} --yes
```

kops는 클러스터가 예상대로 작동하는지 확인하기 위해 실행할 수 있는 편리한 유효성 검사 도구와 함께 제공됩니다.  
아래 명령을 실행하여 쿠버네티스 클러스터 생성이 완료되었는지 체크합니다.

```bash
kops validate cluster --wait 10m
```

### Cluster 사용

클러스터 생성이 완료되면 kubectl 명령을 사용해서 쿠버네티스 클러스터 정보를 확인합니다.

다음 명령으로 Node 목록을 확인할 수 있습니다.  
Master node의 이름을 기록해 둡니다.

```bash
kubectl get nodes
```

다음 명령으로 Pod 전체 목록을 확인할 수 있습니다.

```bash
kubectl get pod --all-namespaces
```

다음 명령으로 모든 시스템 구성 요소를 볼 수 있습니다.

```bash
kubectl -n kube-system get po
```

### Cluster 중지

instances group 목록을 조회합니다.

```bash
$ kops get ig
NAME			ROLE	MACHINETYPE	MIN	MAX	ZONES
master-ap-northeast-2a	Master	t3.medium	1	1	ap-northeast-2a
nodes			Node	t3.large	2	2	ap-northeast-2a,ap-northeast-2c
```

각 instances group의 maxSize와 minSize를 0으로 수정합니다.

```bash
kops edit ig master-ap-northeast-2a
kops edit ig nodes
```

아래 명령을 실행하여 Cluster를 업데이트 합니다.

```bash
kops update cluster --yes
```

다시 instances group 목록을 조회하면 maxSize와 minSize가 0으로 변경된 것을 확인할 수 있습니다.

```bash
$ kops get ig
NAME			ROLE	MACHINETYPE	MIN	MAX	ZONES
master-ap-northeast-2a	Master	t3.medium	0	0	ap-northeast-2a
nodes			Node	t3.large	0	0	ap-northeast-2a,ap-northeast-2c
```

### Cluster 시작

각 instances group의 maxSize와 minSize를 설정합니다.

```bash
kops edit ig master-ap-northeast-2a # maxSize: 1, minSize: 1
kops edit ig nodes # maxSize: 2, minSize: 2
```

아래 명령을 실행하여 Cluster를 업데이트 합니다.

```bash
kops update cluster --yes
```

다음을 실행하여 클러스터가 정상 동작하는지 확인합니다.

```bash
kops validate cluster --wait 10m
kubectl get nodes
kubectl -n kube-system get po
```

### Cluster 롤링 업데이트

K8S Cluster를 업그레이드하고 수정하려면 일반적으로 클라우드 인스턴스를 교체해야 합니다. 서비스 손실 및 기타 중단을 방지하기 위해 kOps는 클라우드 인스턴스를 롤링 업데이트로 점진적으로 대체합니다.

다운타임 없는(Zero-Downtime) 롤링 업데이트를 하려면 아래 명령을 실행합니다.

```bash
kops rolling-update cluster --yes
```

`--cloudonly` 옵션을 추가하면 클러스터의 유효성을 검사하지 않습니다.  
`--force` 옵션을 추가하면 클러스터를 롤링할 필요가 없다는 롤링 업데이트 보고가 있더라도 전체 클러스터를 강제로 롤링합니다.

```bash
kops rolling-update cluster --cloudonly --force --yes
```

### Cluster 삭제

AWS 내에서 Kubernetes 클러스터를 실행하는 것은 당연히 비용이 들기 때문에 실험 실행을 마친 경우 클러스터를 삭제하는 것이 좋습니다.

다음 명령을 실행하여 클러스터가 삭제 될 때 파기될 모든 AWS 리소스를 미리 볼 수 있습니다.

```bash
kops delete cluster --name ${KOPS_CLUSTER_NAME}
```

클러스터를 삭제하려면 --yes 플래그와 함께 delete 명령을 실행합니다. 이 명령은 매우 파괴적이며 클러스터와 그 안에 포함된 모든 것을 삭제합니다!

```bash
kops delete cluster --name ${KOPS_CLUSTER_NAME} --yes
```

## Lens 설정

Workspace EC2 인스턴스에서 아래 명령을 실행하여 kubeconfig 파일 내용을 출력합니다.

```bash
cat ~/.kube/config
```


참고로 만약 로컬 PC에 사용하려면 kubeconfig 파일을 생성하고 위에서 출력된 내용을 복사하여 붙여넣고 저장합니다.

```bash
vi kubeconfig-admin.yaml
```

다음과 같이 수행하여 Lens에 Kubernetes Cluster를 추가합니다.

* 우측하단에 있는 **`+`** 아이콘을 클릭합니다.
* 폴더 모양의 아이콘을 클릭하여 위에서 저장했던 kubeconfig 파일을 선택합니다.
* 콤보박스에서 Context를 선택합니다.

    ![Add Clusters from Kubeconfig](images/lens_add_cluster.png "Add Clusters from Kubeconfig")

* **Add cluster** 버튼을 클릭하면 Kubernetes Cluster를 Lens UI에서 확인할 수 있습니다.

    ![Cluster UI](images/lens_cluster_initial_ui.png "Cluster UI")

* 우측 상단 **`⋮`**(세로 줄임표 모양 메뉴) 클릭 후 **Settings**를 선택합니다.
* 아래로 스크롤하여 **Features** 섹션으로 이동하여 **Install** 버튼을 클릭하여 **Metrics Stack**을 설치합니다.

    ![Install Metrics Stack](images/lens_install_metrics_stack.png "Install Metrics Stack")

* 잠시 후 **Cluster** 메뉴를 클릭하면 **CPU, Memory** 등 Metrics 정보를 확인할 수 있습니다.

    ![Cluster Metrics](images/lens_cluster_metrics.png "Cluster Metrics")

## Cluster의 모든 Node(Worker)에 `insecure-registries` 설정

Cluster의 Pod가 Private Docker Registry에서 HTTP로 Docker 이미지를 Pull 할 수 있게 하려면, Cluster의 모든 Node의 Docker 데몬 설정에 `insecure-registries`을 추가해야 합니다.

아래 명령을 실행하고 **Role**이 `node`인 Node의 이름을 기록합니다.

```bash
kubectl get nodes
```

k8s nodes에 접속합니다. (퍼블릭 IPv4 주소 또는 퍼블릭 IPv4 DNS 이용)

```bash
# 퍼블릭 IPv4 주소
ssh -i ~/.ssh/id_rsa_k8s ubuntu@xx.xx.xx.xxx

또는
# 퍼블릭 IPv4 DNS
ssh -i ~/.ssh/id_rsa_k8s ubuntu@ec2-xx-xx-xx-xxx.ap-northeast-2.compute.amazonaws.com
```

`/etc/docker/daemon.json` 파일을 생성한 다음, 아래 내용을 붙여 넣고 저장합니다.

```bash
$ sudo vi /etc/docker/daemon.json

{
     "insecure-registries" : ["<Harbor_IP>:<Harbor_Port>"]
}
```

아래 명령을 실행하여 Docker daemon를 재시작합니다.

```bash
sudo service docker restart
```

`sudo systemctl status docker` 명령을 실행하여 Docker가 **active (running)**인지 확인합니다.  
`sudo docker info` 명령을 실행하여 적용되었는지 확인합니다.

다른 Node도 동일한 방식으로 설정합니다.

### Kubernetes에 Insecure Docker Registry를 설정하는 더 간단한 방법

`kops edit cluster` 명령을 실행하여 클러스터 구성 편집기 들어갑니다.

```bash
kops edit cluster --name ${KOPS_CLUSTER_NAME}
```

아래와 같이 `spec` 하위에 `docker.insecureRegistry`를 추가하고 저장합니다.

```bash
apiVersion: kops.k8s.io/v1alpha2
kind: Cluster
metadata:
  creationTimestamp: "2021-09-09T06:48:04Z"
  generation: 1
  name: xxx.xxx.k8s.local
spec:
  api:
    loadBalancer:
      type: Public

  ...(생략)

  docker:
    insecureRegistry: <Harbor_IP>:<Harbor_Port>

  ...(생략)
```

클러스터 구성 정보를 반영하려면 아래 명령을 실행합니다.

```bash
kops update cluster --name ${KOPS_CLUSTER_NAME} --yes
kops rolling-update cluster --name ${KOPS_CLUSTER_NAME} --yes
```

Node에 접속하여 아래 명령을 실행하여 확인하면 `insecure-registry`가 설정된 것을 알 수 있습니다.

```bash
sudo docker info
```

실제 설정된 곳은 `/etc/sysconfig/docker` 입니다.

```bash
$ cat /etc/sysconfig/docker
DOCKER_OPTS=--insecure-registry=xx.xx.xx.xx:8000 --ip-masq=false --iptables=false --log-driver=json-file --log-level=warn --log-opt=max-file=5 --log-opt=max-size=10m --storage-driver=overlay2
```

## 참고

[kubectl 설치 및 설정](https://kubernetes.io/ko/docs/tasks/tools/install-kubectl/)  
[kubectl 치트 시트](https://kubernetes.io/ko/docs/reference/kubectl/cheatsheet/)  
[kubectx + kubens: Power tools for kubectl](https://github.com/ahmetb/kubectx)  
[Kops로 쿠버네티스 설치하기](https://kubernetes.io/ko/docs/setup/production-environment/tools/kops/)  
[kops Repository](https://github.com/kubernetes/kops)  
[Installing kops](https://kops.sigs.k8s.io/getting_started/install/)  
[Getting Started with kOps on AWS](https://github.com/kubernetes/kops/blob/master/docs/getting_started/aws.md)  
[AWS CLI에서 Amazon S3 사용](https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/cli-services-s3.html)  
[AWS s3api create-bucket](https://docs.aws.amazon.com/cli/latest/reference/s3api/create-bucket.html)  
[Amazon S3 버킷의 기본 암호화 활성화](https://docs.aws.amazon.com/ko_kr/AmazonS3/latest/user-guide/default-bucket-encryption.html)  
[ssh-keygen - Generate a New SSH Key](https://www.ssh.com/ssh/keygen/)  
[Security Notes for Kubernetes](https://github.com/kubernetes/kops/blob/master/docs/security.md)  
[Kops create cluster CLI](https://kops.sigs.k8s.io/cli/kops_create_cluster/)  
[Kops create secret sshpublickey](https://kops.sigs.k8s.io/cli/kops_create_secret_sshpublickey/)  
[Bastion in kOps](https://github.com/kubernetes/kops/blob/master/docs/bastion.md)  
[Rolling Updates](https://kops.sigs.k8s.io/operations/rolling-update/)  
[kops rolling-update cluster](https://github.com/kubernetes/kops/blob/master/docs/cli/kops_rolling-update_cluster.md)  
[Cluster resource - Docker](https://github.com/kubernetes/kops/blob/master/docs/cluster_spec.md#docker)  
[kops apis - DockerConfig](https://pkg.go.dev/k8s.io/kops/pkg/apis/kops#DockerConfig)  
