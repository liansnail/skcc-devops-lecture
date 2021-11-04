# Jenkins로 CD Pipeline 구축

## Jenkins와 Kubernetes 연동

### Kubernetes plugin 설치

Jenkins와 Kubernetes를 연동하고 Jenkins Pipeline에서 Kubernetes cluster에 리소스 구성(Manifest)을 배포하기 위해 [Kubernetes plugin(v1.30.1)](https://plugins.jenkins.io/kubernetes/), [Kubernetes Continuous Deploy plugin(v.2.3.1)](https://plugins.jenkins.io/kubernetes-cd)과 [Kubernetes CLI(v1.10.3)](https://plugins.jenkins.io/kubernetes-cli/) 설치합니다.

>[주의] 현재 환경에 맞게 버전을 확인하여 설치하여야합니다.

<details markdown="1">
<summary>Jenkins 플러그인 직접 설치 방법</summary>

1. **젠킨스 관리** ► **플러그인 관리** 메뉴로 이동
2. `설치 가능` 탭 클릭
3. 조건검색으로 플러그인 검색
4. 해당 플러그인 이름을 클릭
5. Plugin Information 메뉴에서 `Release` 항목으로 이동
6. 원하는 버전을 바로 눌러 다운로드(확장자: `hpi`)
7. 플러그인 관리 메뉴 고급탭으로 이동
8. 플러그인 올리기 항목에서 다운로드 받은 파일(`hpi`)파일을 선택하여 업로드
9. 설치 후 재시작

</details><br>

> Jenkins를 재시작후 설치가 필요한 경우 다음과 같이 합니다.
> http://xx.xx.xx.xx:8080/restart

* **Dashboard dropdown menu > Jenkins 관리 > System Configuration > 플러그인 관리**을 선택합니다.
* **설치 가능** 탭을 클릭하고 검색 필드에 `kube`를 입력합니다.
* **Kubernetes**, **Kubernetes Continuous Deploy**와 **Kubernetes CLI** 플러그인을 체크하고 설치합니다.
<img src="images/jenkins_plugin_kube.png" width="400"/>

### Kubernetes credentials(자격 증명) 생성

* **Dashboard dropdown menu > Jenkins 관리 > Security > Manage Credentials**을 선택하여 **Credentials** 페이지로 이동합니다.
* **Domain > (global)** 드롭다운 메뉴에서 **Add Credentials**을 클릭합니다.
* 다음 항목을 입력하거나 선택한 후 **OK** 버튼을 클릭합니다.
  * **Kind** : `Secret file` 선택
  * **Scope** : `Global (Jenkins, nodes, items, all child items, etc)` 선택
  * **File** : kube config 파일 선택 (예 : `/home/ubuntu/.kube/config`)
  * **ID** : 원하는 고유한 ID 입력 (예 : `kubernetes-credentials`)
  * **Description** : (선택사항) 유사한 자격 증명(credentials)을 구분하는 데 도움이 되는 설명 입력 (예: `kubernetes-credentials-admin`)

### Kubernetes 연동 설정

* **Dashboard dropdown menu > Jenkins 관리 > System Configuration > Configurate System**을 선택합니다.
* 스크롤해서 맨아래로 이동하면 **Cloud** 섹션이 있습니다.
* **a separate configuration page** 링크를 클릭합니다.
* **Configure Clouds** 페이지에서 **Add a new cloud** 드롭다운 메뉴에서 `Kubernetes`를 선택합니다.
* **Kubernetes Cloud details...** 버튼을 클릭한 다음, 아래 항목을 입력합니다.
  * **Name** : `kubernetes` (기본값)
  * **Credentials** : 앞 단계에서 생성한 Kubernetes credentials 선택
  * **Test Connection** 버튼을 클릭하여 Jenkins에서 Kubernetes Cluster로 연결되는지 확인합니다. (에러가 발생할 경우, K8s master, nodes가 정상적으로 동작하는지 확인)
  * **Jenkins URL** : `http://<Jenkins_IP>:<Jenkins_Port>` 입력
* **Pod Templates...** 버튼을 클릭합니다.
* **Add Pod Template** 버튼을 클릭하고 **Pod Template details...** 버튼을 클릭한 다음, 아래 항목을 입력합니다.
  * **Name** : `jenkins-slave`(원하는 이름) 입력
  * **Labels** : `jenkins-slave`(원하는 이름) 입력
  * **Usage** : `Use this node as much as possible` 선택
* **Containers** 필드의 **Add Container** 드롭다운 메뉴에서 **Container Template**을 클릭한 다음, 다음 항목을 입력합니다.
  * **Name** : `jenkins-slave`(원하는 이름) 입력
  * **Docker image** : `jenkinsci/jnlp-slave` 입력
  * **Working directory** : `/home/jenkins/agent` 입력
  * **Command to run** : `/bin/sh -c` 입력
  * **Arguments to pass to the command** : `cat` 입력
* **Save** 버튼을 클릭합니다.

### Kubernetes 연동 Job 구성 및 실행

다음을 수행하여 Kubernetes Cluster의 Pod에서 Jenkins agent(`jenkins-slave`)에 의해서 실행되는 Jenkins Job을 생성하고 실행합니다.

* 사이드 바에서 **새로운 Item (New Item)** 메뉴를 클릭 합니다.
* Job 이름(`Kubernetes-Integration-Pipeline`)을 입력하고 `Pipeline`를 선택한 다음 **OK** 버튼을 클릭합니다.
* **Pipeline** 탭을 클릭하면 아래로 스크롤 되어 **Pipeline** 섹션으로 이동합니다.
* **Definition** 필드에 `Pipeline script`(기본값)을 선택합니다.
* **Script** 필드에 아래 내용을 복사하여 붙여 넣습니다.

    ```bash
    pipeline {
      agent {
        kubernetes {
          yaml """\
            apiVersion: v1
            kind: Pod
            metadata:
              labels:
                some-label: some-label-value
            spec:
              containers:
              - name: maven
                image: maven:alpine
                command:
                - cat
                tty: true
              - name: busybox
                image: busybox
                command:
                - cat
                tty: true
          """.stripIndent()
        }
      }
      stages {
        stage('Run maven') {
          steps {
            container('maven') {
              sh 'mvn -version'
            }
            container('busybox') {
              sh '/bin/busybox'
            }
          }
        }
      }
    }
    ```

* **저장** 버튼을 클릭합니다.
* 사이드 바에서 **Build Now**를 클릭하여 Job을 실행합니다.
* **Build History**에서 **#일련번호**을 선택한 다음, **Console Output**을 선택하여 빌드 로그를 확인합니다.
(앞에서 petclinic-dev을 지웠을 경우, 에러가 발생하므로 07-Exposing-Apps-using-Kubernetes-Services를 통해 복구 필요)

## 기본 CD Pipeline 구성

### Kubernetes Namespace 생성

Workspace EC2 인스턴스에서 아래 명령을 실행하여 `petclinic-dev`라는 이름의 Namespace를 생성합니다.

```bash
kubectl create namespace petclinic-dev
```

Namespace가 생성되었는지 확인합니다.

```bash
$ kubectl get namespaces
NAME              STATUS   AGE
default           Active   44h
petclinic-dev   Active   26s
kube-node-lease   Active   44h
kube-public       Active   44h
kube-system       Active   44h
lens-metrics      Active   43h
```

현재 Namespace를 변경합니다.

```bash
kubens petclinic-dev
```

### Service Account 생성

`petclinic-sa`라는 이름의 Service Account를 생성합니다.

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: petclinic-sa
  namespace: petclinic-dev
EOF
```

Service Account 정보를 확인합니다.

```bash
kubectl get serviceaccounts petclinic-sa -o yaml
```

### Role 생성

`petclinic-role`라는 이름의 Role를 생성합니다.

```bash
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: petclinic-dev
  name: petclinic-role
rules:
- apiGroups: ["extensions", "apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: [""]
  resources: ["pods","services"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: ["networking.k8s.io"]
  resources: ["ingresses"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
EOF
```

Role 정보를 확인합니다.

```bash
kubectl get roles petclinic-role -o yaml
```

### Role Binding 생성

`petclinic-rb`라는 이름의 RoleBinding를 생성하여 `petclinic-sa` Service Account와 `petclinic-role` Role를 Binding 합니다.

```bash
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  namespace: petclinic-dev
  name: petclinic-rb
subjects:
- kind: ServiceAccount
  name: petclinic-sa
  namespace: petclinic-dev
roleRef:
  kind: Role 
  name: petclinic-role
  apiGroup: rbac.authorization.k8s.io
EOF
```

Role Binding 정보를 확인합니다.

```bash
kubectl get rolebindings petclinic-rb -o yaml
```

### Harbor User 생성

* **Administration > Users**에서 **NEW USER** 버튼을 클릭한 후 User를 생성합니다.
* **Projects** 페이지로 이동한 후, `petclinic` 프로젝트를 선택합니다.
* **Members** 탭을 클릭합니다.
* **USER** 버튼을 클릭한 후, **Name** 필드에서 계정을 검색하여 선택합니다.
* **Role** 필드에서 `Developer`를 선택하고 **OK** 버튼을 클릭합니다.

### Docker Registry 시크릿(Secret) 생성

Private Docker Registry에서 이미지를 Pull 하기 위한 `docker-registry` 타입의 시크릿이 생성합니다. (시크릿명 : `harbor-credentials`)

--------------------------------------------------
참고.신규가 아닌 07-Exposing-Apps-using-Kubernetes-Services 기존의 Docker 자격 증명을 기반으로 시크릿 생성하기 이용함
```
kubectl create secret generic harbor-credentials --from-file=.dockerconfigjson=./docker-config.json --type=kubernetes.io/dockerconfigjson
```
--------------------------------------------------

```bash
kubectl create secret docker-registry harbor-credentials \
    --docker-server=<Harbor_IP>:<Harbor_Port> \
    --docker-username=<your-name> \
    --docker-password=<your-password> --docker-email=<your-email>
```

생성되었는지 확인합니다.

```bash
kubectl get secrets harbor-credentials
```

또는

```bash
kubectl describe secrets/harbor-credentials
```

### Service Account에 ImagePullSecrets 추가

Service Account을 수정하여 Harbor 시크릿을 imagePullSecret으로 사용하도록 합니다.

```bash
kubectl patch serviceaccount petclinic-sa -p '{"imagePullSecrets": [{"name": "harbor-credentials"}]}'
```

추가되었는지 확인합니다.

```bash
kubectl get serviceaccounts petclinic-sa -o yaml
```

### Service Account Token 확인

```bash
$ kubectl describe secret $(kubectl get secret | grep petclinic-sa | awk '{print $1}')
Name:         petclinic-sa-token-zr4ff
Namespace:    petclinic-dev
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: petclinic-sa
              kubernetes.io/service-account.uid: 0adc4d27-e95a-4b6d-843b-94398bec3c38

Type:  kubernetes.io/service-account-token

Data
====
token:      eyJhbGciOiJSUzI1NiIs...(생략)
ca.crt:     1042 bytes
namespace:  15 bytes
```

### Service Account Token으로 인증하는 kubeconfig 파일 생성

로컬 PC에서 `/home/ubuntu/.kube/config` 파일을 `kubeconfig-sa-token.yaml`로 복사한 후, `kubeconfig-sa-token.yaml` 파일에서 `user:`의 `client-certificate-data`, `client-key-data` 부분을 제거합니다.

```bash
$ cd /home/ubuntu/.kube/
$ cp config kubeconfig-sa-token.yaml

(ubuntu@workspace:~/.kube)$ ls -l
total 28
drwxr-x--- 4 ubuntu ubuntu 4096 Sep  9 16:06 cache
-rw------- 1 ubuntu ubuntu 5694 Sep 13 09:32 config
drwxr-x--- 3 ubuntu ubuntu 4096 Sep  9 16:31 http-cache
-rw------- 1 ubuntu ubuntu 5694 Sep 13 09:59 kubeconfig-sa-token.yaml
drwxr-xr-x 2 ubuntu ubuntu 4096 Sep 10 09:58 kubens


$ vi kubeconfig-sa-token.yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0t...(생략)
    server: https://api-xxxxx.ap-northeast-2.elb.amazonaws.com
  name: devops.skcc-devops01.k8s.local
contexts:
- context:
    cluster: devops.skcc-devops01.k8s.local
    user: devops.skcc-devops01.k8s.local
  name: devops.skcc-devops01.k8s.local
current-context: devops.skcc-devops01.k8s.local
kind: Config
preferences: {}
users:
- name: devops.skcc-devops01.k8s.local
  user:
    client-certificate-data: LS0tL...(생략)
    client-key-data: LS0t...(생략)
```

`user:` 아래에 앞 단계에서 확인한 Service Account Token를 추가합니다.

```bash
$ vi kubeconfig-sa-token.yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0t...(생략)
    server: https://api-xxxxx.ap-northeast-2.elb.amazonaws.com
  name: devops.skcc-devops01.k8s.local
contexts:
- context:
    cluster: devops.skcc-devops01.k8s.local
    user: devops.skcc-devops01.k8s.local
  name: devops.skcc-devops01.k8s.local
current-context: devops.skcc-devops01.k8s.local
kind: Config
preferences: {}
users:
- name: devops.skcc-devops01.k8s.local
  user:
    token: eyJhbGciOiJSUzI1NiIs...(생략)
```

### kubeconfig 파일로 권한 확인

로컬 PC에 kubectl일 설치되어 있다면, 다음을 수행하여 권한을 확인합니다. (설치가 안된 경우, Workspace EC2에 복사하여 수행)

* `petclinic-dev` Namespace의 Pod 객체 조회 (접근 가능)

```bash
$ kubectl --kubeconfig ./kubeconfig-sa-token.yaml -n petclinic-dev get pod
No resources found in petclinic-dev namespace.
```

* `default` Namespace의 Pod 객체 조회 (접근 불가능)

```bash
$ kubectl --kubeconfig ./kubeconfig-sa-token.yaml -n default get pod
Error from server (Forbidden): pods is forbidden: User "system:serviceaccount:petclinic-dev:petclinic-sa" cannot list resource "pods" in API group "" in the namespace "default"
```

### `kubeconfig-sa-token` Kubernetes credentials(자격 증명) 생성

* **Dashboard dropdown menu > Jenkins 관리 > Security > Manage Credentials**을 선택하여 **Credentials** 페이지로 이동합니다.
* **Domain > (global)** 드롭다운 메뉴에서 **Add Credentials**을 클릭합니다.
* 다음 항목을 입력하거나 선택한 후 **OK** 버튼을 클릭합니다.
  * **Kind** : `Kubernetes configuration (kubeconfig)` 선택
  * **Scope** : `Global (Jenkins, nodes, items, all child items, etc)` 선택
  * **ID** : 원하는 고유한 ID 입력 (예 : `kubeconfig-sa-token`)
  * **Description** : (선택사항) 유사한 자격 증명(credentials)을 구분하는 데 도움이 되는 설명 입력 (예: `petclinic-dev Namespace Service Account Token`)
  * **Kubeconfig** : `Enter directly`를 선택하고 `kubeconfig-sa-token.yaml` 파일의 내용 입력
    > [주의] **cat kubeconfig-sa-token.yaml** 를 이용하여 복사/붙여넣기 합니다.

### (앞서 생성한 경우 생략)Docker Registry 자격 증명 생성 

* **Dashboard dropdown menu > Jenkins 관리 > Security > Manage Credentials**을 선택하여 **Credentials** 페이지로 이동합니다.
* **Domain > (global)** 드롭다운 메뉴에서 **Add Credentials**을 클릭합니다.
* 다음 항목을 입력하거나 선택한 후 **OK** 버튼을 클릭합니다.
  * **Kind** : `Username and password` 선택
  * **Scope** : `Global (Jenkins, nodes, items, all child items, etc)` 선택
  * **Username** : Harbor의 프로젝트 계정 Username (시크릿 생성 시 사용했던 계정)
  * **Password** : Harbor의 프로젝트 계정 패스워드 입력
  * **ID** : 원하는 고유한 ID 입력 (예: `harbor-docker-registry`)
  * **Description** : (선택사항) 유사한 자격 증명(credentials)을 구분하는 데 도움이 되는 설명 입력

### `petclinic` App 배포 매니페스트(Manifest) 파일 준비

Kubernetes Cluster에 애플리케이션을 배포하고 서비스를 통해 노출하기 위해서는 Deployment, Service, Ingress와 같은 리소스가 필요합니다.  
이러한 쿠버네티스 오브젝트는 구성 파일를 정의하고 `kubectl apply` 명령을 통해 오브젝트를 생성합니다.  
즉, [Manifest](https://kubernetes.io/docs/reference/glossary/?all=true#term-manifest) 파일이 필요한데, 각 오브젝트 별로 생성하거나 동일한 파일에 그룹화하여 단순화할 수 있습니다. (YAML에서 `---` 로 구분)

다음을 수행하여 `spring-petclinic-rest.yaml` 파일을 작성하고 GitHub에 Push 합니다.

* `spring-petclinic-rest` Workspace에서 `spring-petclinic-rest.yaml` 파일을 생성하고 아래 내용을 붙여 넣습니다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: petclinic-deployment
  namespace: petclinic-dev
  labels:
    app: petclinic
spec:
  replicas: 2
  selector:
    matchLabels:
      app: petclinic
  template:
    metadata:
      labels:
        app: petclinic
    spec:
      containers:
      - name: petclinic
        image: ${REGISTRY_URL}/${IMAGE_REPO}/${IMAGE_NAME}:${IMAGE_TAG}
        ports:
        - containerPort: 9966
      imagePullSecrets:
      - name: harbor-credentials
---
apiVersion: v1
kind: Service
metadata:
  name: petclinic-service
  namespace: petclinic-dev
  labels:
    app: petclinic
spec:
  type: ClusterIP
  selector:
    app: petclinic
  ports:
    - protocol: TCP
      port: 9966
      targetPort: 9966
---
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: petclinic-ingress
  namespace: petclinic-dev
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
  - http:
      paths:
      - path: /
        backend:
          serviceName: petclinic-service
          servicePort: 9966
```

* 아래 명령을 수행하여 GitHub에 Push 합니다.

    ```bash
    git add spring-petclinic-rest.yaml
    git commit -m "Add K8S Manifest file"
    git push
    ```

### 기본 CD Jenkinsfile 생성

다음을 수행하여 Kubernetes agent pod yaml 파일과 Jenkinsfile 파일을 생성하고 GitHub에 Push 합니다.

* `jenkins-agent-pod.yaml` 파일을 생성하고 아래 내용을 붙여 넣습니다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  namespace: petclinic-dev
  labels:
    component: jenkins-agent
spec:
  containers:
  - name: maven
    image: 'maven:3.6.3-adoptopenjdk-11'
    command:
    - cat
    tty: true
  - name: docker
    image: docker:latest
    command:
    - cat
    tty: true
    volumeMounts:
    - mountPath: /var/run/docker.sock
      name: docker-sock
  - name: kubectl
    image: bitnami/kubectl:latest
    command:
    - cat
    tty: true
  serviceAccountName: petclinic-sa
  securityContext:
    runAsUser: 0
  volumes:
    - name: docker-sock
      hostPath:
        path: /var/run/docker.sock
```

* 아래 명령을 수행하여 GitHub에 Push 합니다.

    ```bash
    git add jenkins-agent-pod.yaml
    git commit -m "Add Kubernetes agent Pod Manifest file"
    git push
    ```

* `Jenkinsfile-Basic-CD` 파일을 생성하고 아래 내용을 붙여 넣고 `REGISTRY_URL`, `REGISTRY_CREDENTIALS`를 수정합니다. (ex. REGISTRY_CREDENTIALS='harbor-docker-register')

```groovy
pipeline {
  agent {
    kubernetes {
      label 'petclinic-cd'
      yamlFile 'jenkins-agent-pod.yaml'
    }
  }

  environment {
    REGISTRY_URL = '<Harbor_IP>:<Harbor_Port>'
    REGISTRY_CREDENTIALS = '<Docker_Registry_Credentials>'
    APP_IMAGE = null
    IMAGE_REPO = 'petclinic'
    IMAGE_NAME = 'spring-petclinic-rest'
    IMAGE_TAG = sh(returnStdout: true, script: '(git rev-parse --short HEAD && echo "_$BUILD_NUMBER") | tr -d "\n"').trim()
  }

  stages {
    stage('Package') {
      steps {
        container('maven') {
          sh 'mvn clean package -DskipTests'
          archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
        }
      }
    }
    stage('Build Docker image') {
      steps {
        container('docker') {
          script {
            APP_IMAGE = docker.build("${IMAGE_REPO}/${IMAGE_NAME}:${IMAGE_TAG}")
          }
        }
      }
    }
    stage('Push Docker image') {
      steps {
        container('docker') {
          script {
            docker.withRegistry('http://' + REGISTRY_URL, REGISTRY_CREDENTIALS) {
              APP_IMAGE.push()
              APP_IMAGE.push('latest')
            }
          }
        }
      }
    }
    stage( 'Deploy to Cluster' ) {
      steps {
        script {
          kubernetesDeploy(kubeconfigId: 'kubeconfig-sa-token', configs: 'spring-petclinic-rest.yaml')
        }
        sh 'sleep 20'
      }
    }
  }
}
```

* 아래 명령을 수행하여 GitHub에 Push 합니다.

    ```bash
    git add .
    git commit -m "Add Jenkinsfile-Basic-CD"
    git push
    ```

### 기본 CD Pipeline Job 생성

* 사이드 바에서 **새로운 Item (New Item)** 메뉴를 클릭 합니다.
* Job 이름(예: `Petclinic-Basic-CD-Pipeline`)을 입력하고 `Pipeline`를 선택한 다음 **OK** 버튼을 클릭합니다.
* **Pipeline** 탭을 클릭하면 아래로 스크롤 되어 **Pipeline** 섹션으로 이동합니다.
* **Definition** 필드에 `Pipeline script from SCM`을 선택합니다.
* **SCM** 필드에 `Git`을 선택합니다.
* **Repository URL** 필드에 GitHub Repository의 Clone HTTPS URL을 복사하여 붙여 넣습니다.
* **Credentials** 필드에 GitHub 자격 증명을 선택합니다. (ex. skcc-devopsXX-github credential)
* **Script Path** 필드에 `Jenkinsfile-Basic-CD`을 입력합니다.
* **Branch Specifier** 필드에 `*/master`으로 수정(확인)한 다음 **저장** 버튼을 클릭합니다.

### 기본 CD Pipeline Job 실행

* **Dashboard**에서 Job 선택한 다음, 사이드 바에서 **Build Now**를 클릭하여 Job이 실행합니다.
* **Build History**에서 **#일련번호**을 선택한 다음, **Console Output**을 선택하여 빌드 로그를 확인합니다.

<!--
참고)
아래와 같은 에러 발생함. https://stackoverflow.com/questions/60790118/java-kubernetes-client-sslhandshakeexception-extension-5-should-not-be-present
참고해 JVM 옵션을 추가하는 방법이 필요해 보임(-Djdk.tls.client.protocols=TLSv1.2).
https://issueexplorer.com/issue/jenkinsci/kubernetes-cd-plugin/133
https://segmentfault.com/a/1190000040469278/en
https://velog.io/@freddiey/kubernetes-client-TLS-%EC%97%90%EB%9F%AC
```
Starting Kubernetes deployment
Loading configuration: /home/jenkins/agent/workspace/Petclinic-Basic-CD-Pipeline/spring-petclinic-rest.yaml
ERROR: ERROR: java.lang.RuntimeException: io.kubernetes.client.openapi.ApiException: javax.net.ssl.SSLHandshakeException: extension (5) should not be presented in certificate_request
hudson.remoting.ProxyException: java.lang.RuntimeException: io.kubernetes.client.openapi.ApiException: javax.net.ssl.SSLHandshakeException: extension (5) should not be presented in certificate_request
	at com.microsoft.jenkins.kubernetes.wrapper.ResourceManager.handleApiExceptionExceptNotFound(ResourceManager.java:180)
 ...
	Suppressed: hudson.remoting.Channel$CallSiteStackTrace: Remote call to JNLP4-connect connection from ec2-13-125-103-105.ap-northeast-2.compute.amazonaws.com/13.125.103.105:47612
		at hudson.remoting.Channel.attachCallSiteStackTrace(Channel.java:1800)
 ...
ERROR: Kubernetes deployment ended with HasError
```
-->

### Maven 성능 확인

Jenkins Job을 다시 실행하여 `Package` stage 단계에서 Maven 의존성 라이브러리를 다시 다운로드 받는 것을 확인합니다.

### Maven Cache 설정

#### PersistentVolumeClaim 생성

StorageClass 리소스를 통행 사용 가능한 스토리지 유형을 정의합니다.

```bash
cat <<EOF | kubectl apply -f -
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: maven-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
EOF
```

PersistentVolumeClaim를 생성합니다.

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: "maven-pvc"
  namespace: "petclinic-dev"
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: "maven-ssd"
  resources:
    requests:
      storage: 2Gi
EOF
```

PersistentVolumeClaim가 Bound(STATUS) 되었는지 확인합니다.

```bash
$ kubectl get pv,pvc
NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                            STORAGECLASS    REASON   AGE
persistentvolume/pvc-8194e87e-2235-49d4-8a48-0476ae9662a7   19Gi       RWO            Delete           Bound    lens-metrics/data-prometheus-0   kops-ssd-1-17            3d18h
persistentvolume/pvc-88a142d8-b72e-46b0-9c8c-6cea84a9e34b   2Gi        RWO            Delete           Bound    petclinic-dev/maven-pvc          maven-ssd                6s

NAME                              STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/maven-pvc   Bound    pvc-88a142d8-b72e-46b0-9c8c-6cea84a9e34b   2Gi        RWO            maven-ssd      12s
```

**EC2 콘솔 > Elastic Block Store > 볼륨**에서 `*-dynamic-pvc-xxxx`이라는 PersistentVolumeClaim용 EBS 볼륨이 생성된 것을 확인할 수 있습니다.

#### `jenkins-agent-pod.yaml` 파일 수정

* `jenkins-agent-pod.yaml` 파일에서 Maven Cache를 위해 PersistentVolumeClaim 설정을 추가합니다.

```yaml

...(생략)

spec:
  containers:
  - name: maven
    image: 'maven:3.6.3-adoptopenjdk-11'
    command:
    - cat
    tty: true
    volumeMounts:
    - mountPath: '/root/.m2'
      name: m2

...(생략)

  volumes:
    - name: m2
      persistentVolumeClaim:
        claimName: maven-pvc
    - name: docker-sock
      hostPath:
        path: /var/run/docker.sock
```

* 아래 명령을 수행하여 GitHub에 Push 합니다.

    ```bash
    git add .
    git commit -m "Add Maven PersistentVolumeClaim"
    git push
    ```

Job을 두 번 이상 실행하면 Maven .m2 폴더가 Cache되어 의존성 라이브러리를 다운로드 하지 않는 것을 확인할 수 있습니다.

## API Test

### Collection 파일 준비

[Postman 구성 및 Test scripts 작성](08.1-Setup-Postman.md) 가이드에 따라 API 테스트 스크립트를 작성하고 Postman Collection를 Export하여, `spring-petclinic-rest` Workspace에 저장합니다.

```bash
cd ~/devops/workspace
git clone https://github.com/skcc-devops-pipeline-study/skcc-devops-lecture.git
cd skcc-devops-lecture
```
ubuntu 서랍장을 이용하여 필요한 파일 또는 디렉토리를 spring-petclinic-rest 폴더로 붙여 넣습니다.

아래 명령을 수행하여 GitHub에 Push 합니다.

```bash
git add api_test.json
git commit -m "Add Postman Collection"
git push
```

### `postman/newman` Container 추가

* `jenkins-agent-pod.yaml` 파일에 `postman/newman` Container을 추가합니다.

```yaml

...(생략)

spec:
  containers:
  
  ...(생략)

  - name: newman
    image: postman/newman
    command:
    - cat
    tty: true
  serviceAccountName: petclinic-sa

...(생략)

```

* 아래 명령을 수행하여 GitHub에 Push 합니다.

    ```bash
    git add .
    git commit -m "Add 'postman/newman' Container"
    git push
    ```

### 네트워크 로드 밸런서(NLB)의 DNS 이름 확인

**EC2 콘솔 > 로드 밸런싱 > 로드밸런서**에서 네트워크 로드 밸런서(`network` 유형)을 클릭한 다음, **설명** 탭에서 **DNS 이름**을 복사하여 기록해 둡니다. (`petclinic-ingress`의 Hostname)

### Jenkinsfile에 'API Test' stage 추가

* `Jenkinsfile-Basic-CD` 파일을 열고 아래 내용을  추가하여 수정합니다.

```groovy
pipeline {
  
  ...(생략)

  environment {

    ...(생략)

    APP_URL = 'http://<NLB_DNS_Name>/petclinic'
  }

  stages {
    
    ...(생략)

    stage('API Test') {
      steps {
        container('newman') {
          sh """
            newman run api_test.json \
                  --env-var 'baseUrl=${APP_URL}' \
                  --reporters cli,junit \
                  --reporter-junit-export 'target/newman/petclinic-report.xml'
          """
        }
      }
      post {
        always {
          junit 'target/newman/*.xml'
        }
      }
    }
  }
}
```

* 아래 명령을 수행하여 GitHub에 Push 합니다.

    ```bash
    git add Jenkinsfile-Basic-CD
    git commit -m "Add 'API Test' stage to Jenkinsfile"
    git push
    ```

### 기본 CD Pipeline Job 실행 (API Test)

* **Dashboard**에서 Job 선택한 다음, 사이드 바에서 **Build Now**를 클릭하여 Job이 실행합니다.
* **Build History**에서 **#일련번호**을 선택한 다음, **Console Output**을 선택하여 빌드 로그를 확인합니다.
```bash
newman

REST Petclinic backend Api Customized2

❏ api / pettypes
↳ getAllPetTypes
  GET http://a625d83985d844a76905bcfcd73e43b2-f79a950ae81364e7.elb.ap-northeast-2.amazonaws.com/petclinic/api/pettypes [200 OK, 554B, 1052ms]
  ✓  Body에 cat이 포함되어 있는지 테스트
  ┌
  │ 'request : [{"id":1,"name":"cat"},{"id":2,"name":"dog"
  │ },{"id":3,"name":"lizard"},{"id":4,"name":"snake"},{"i
  │ d":5,"name":"bird"},{"id":6,"name":"hamster"}]'
  └

↳ addPetType
  POST http://a625d83985d844a76905bcfcd73e43b2-f79a950ae81364e7.elb.ap-northeast-2.amazonaws.com/petclinic/api/pettypes?petTypeId [201 Created, 568B, 494ms]
  ┌
  │ 'addPetType 결과 id : 7'
  │ 'addPetType 결과 name : elephant'
  │ '환경변수 설정된 petTypeId : 7'
  └

┌─────────────────────────┬─────────────────────┬─────────────────────┐
│                         │            executed │              failed │
├─────────────────────────┼─────────────────────┼─────────────────────┤
│              iterations │                   1 │                   0 │
├─────────────────────────┼─────────────────────┼─────────────────────┤
│                requests │                   2 │                   0 │
├─────────────────────────┼─────────────────────┼─────────────────────┤
│            test-scripts │                   4 │                   0 │
├─────────────────────────┼─────────────────────┼─────────────────────┤
│      prerequest-scripts │                   3 │                   0 │
├─────────────────────────┼─────────────────────┼─────────────────────┤
│              assertions │                   1 │                   0 │
├─────────────────────────┴─────────────────────┴─────────────────────┤
│ total run duration: 1669ms                                          │
├─────────────────────────────────────────────────────────────────────┤
│ total data received: 169B (approx)                                  │
├─────────────────────────────────────────────────────────────────────┤
│ average response time: 773ms [min: 494ms, max: 1052ms, s.d.: 279ms] │
└─────────────────────────────────────────────────────────────────────┘
[Pipeline] }
[Pipeline] // container
Post stage
[Pipeline] junit
Recording test results
```


* **Test Result** 메뉴에서 **API Test** 결과를 확인합니다.



## Performance Test

### JMeter Test Plan 파일 준비

[Apache JMeter 구성 및 Test Plan 작성](08.2-Setup-JMeter.md) 가이드에 따라 Performance Test Plan를 작성하고 JMX(ex: `PerformanceTest/petclinic-test-plan.jmx`) 파일을 담은 디렉토리 전체를 `spring-petclinic-rest` Workspace에 저장합니다.

아래 명령을 수행하여 GitHub에 Push 합니다.

```bash
git add .
git commit -m "Add JMeter Test Plan"
git push
```

### Performance plugin 설치

JMeter의 성능 테스트 결과를 레포트하기 위해 [Performance plugin](https://plugins.jenkins.io/performance)을 설치합니다.

* **Dashboard dropdown menu > Jenkins 관리 > System Configuration > 플러그인 관리**을 선택합니다.
* **설치 가능** 탭을 클릭하고 검색 필드에 `performance`를 입력합니다.
* **Performance** 플러그인을 체크하고 설치합니다.

### `jmeter` Container 추가

* `jenkins-agent-pod.yaml` 파일에 `jmeter` Container을 추가합니다.

JMeter의 Docker official image가 없으므로  `silver790310/jmeter:v2` 이미지를 사용합니다.

```yaml

...(생략)

spec:
  containers:
  
  ...(생략)

  - name: jmeter
    image: silver790310/jmeter:v2
    command:
    - cat
    tty: true
  serviceAccountName: petclinic-sa

...(생략)

```

* 아래 명령을 수행하여 GitHub에 Push 합니다.

    ```bash
    git add .
    git commit -m "Add 'JMeter' Container"
    git push
    ```

### Jenkinsfile에 'Performance Test' stage 추가

* `Jenkinsfile-Basic-CD` 파일을 열고 아래 내용(PerfURL, Performance Test stage)을 추가하여 수정합니다.

```groovy
pipeline {
  
  ...(생략)

  environment {
    ...(생략)
    PerfURL='a4cf4fcb1fa4144d59a3f627fbacc317-799325xxxxxxxxx.elb.ap-northeast-2.amazonaws.com'
  }

  stages {
    
    ...(생략)

    stage('PerforfmanceTest') {
     steps {
       withCredentials([usernamePassword(credentialsId: 'skcc-devops01-github', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PWD')]) {
          container('jmeter') {
              sh "git checkout origin/master"
              sh "git config remote.origin.url https://${GIT_USER}:${GIT_PWD}@github.com/skcc-devops01/spring-petclinic-rest.git"
              sh "git config --global user.email 'skcc.devops01@sk.com'"
              sh "git config --global user.name 'skcc-devops01'"
              echo 'jmeter 명령어 실행'
              sh """
                 JVM_ARGS="-Xms1G -Xmx1G" && export JVM_ARGS && /usr/local/jmeter/apache-jmeter-5.4.1/bin/jmeter.sh \
                 -n -f -t PerformanceTest/petclinic-test-plan.jmx -Jurl=${PerfURL} \
                 -l PerformanceTest/TestResult/Result_${BUILD_NUMBER}.jtl \
                 -e -o PerformanceTest/TestResult/Result_html_${BUILD_NUMBER}
                 mv jmeter.log PerformanceTest/TestResult/Result_html_${BUILD_NUMBER}/jmeter.log
                 git add . && git commit -m 'Publish Jmeter result' && git push origin HEAD:master
              """
          }
       }
     }
      post {
        always {
        perfReport 'PerformanceTest/TestResult/Result_${BUILD_NUMBER}.jtl'
        }
      }
    }     

  }
}
```

* 아래 명령을 수행하여 GitHub에 Push 합니다.

    ```bash
    git add Jenkinsfile-Basic-CD
    git commit -m "Add 'Performance Test' stage to Jenkinsfile"
    git push
    ```

### 기본 CD Pipeline Job 실행 (Performance Test)

* **Dashboard**에서 Job 선택한 다음, 사이드 바에서 **Build Now**를 클릭하여 Job이 실행합니다.
* **Build History**에서 **#일련번호**을 선택한 다음, **Console Output**을 선택하여 빌드 로그를 확인합니다.
* **Performance Trend**와 **Performance Report** 결과를 확인합니다.

### 터미널에서 Performance Test 결과 가져오기

```bash
$ cd ~/PATH/workspace/spring-petclinic-rest/
$ git pull
$ cd PerformanceTest/TestResult
$ ls -al
total 112
drwxrwxr-x 7 ubuntu ubuntu  4096 Sep 27 14:17 .
drwxrwxr-x 5 ubuntu ubuntu  4096 Sep 14 10:41 ..
-rw-rw-r-- 1 ubuntu ubuntu  7326 Sep 27 14:17 Result_xx.jtl
drwxrwxr-x 4 ubuntu ubuntu  4096 Sep 27 14:17 Result_html_xx
-rw-rw-r-- 1 ubuntu ubuntu 36028 Jul 14 15:37 TS01_TC01_AWS.jmx
$ cd Result_html_xx
$ ls -al
total 56
drwxrwxr-x 4 ubuntu ubuntu  4096 Sep 27 14:17 ./
drwxrwxr-x 7 ubuntu ubuntu  4096 Sep 27 14:17 ../
drwxrwxr-x 5 ubuntu ubuntu  4096 Sep 27 14:17 content/
-rw-rw-r-- 1 ubuntu ubuntu  9467 Sep 27 14:17 index.html
-rw-rw-r-- 1 ubuntu ubuntu 21375 Sep 27 14:17 jmeter.log
drwxrwxr-x 5 ubuntu ubuntu  4096 Sep 27 14:17 sbadmin2-1.0.7/
-rw-rw-r-- 1 ubuntu ubuntu  1917 Sep 27 14:17 statistics.json
```

* (크롬)Browser에서 index.html을 열어 결과를 확인합니다.
<img src="images/jmeter_index_result.png" width=""/>

## 참고

[Jenkins JNLP Agent Docker image](https://hub.docker.com/r/jenkinsci/jnlp-slave)  
[Maven Docker Official Images](https://hub.docker.com/_/maven)  
[BusyBox Docker Official Images](https://hub.docker.com/_/busybox)  
[프라이빗 레지스트리 사용](https://kubernetes.io/ko/docs/concepts/containers/images/#%ED%94%84%EB%9D%BC%EC%9D%B4%EB%B9%97-%EB%A0%88%EC%A7%80%EC%8A%A4%ED%8A%B8%EB%A6%AC-%EC%82%AC%EC%9A%A9)  
[Kubernetes 리소스 관리](https://kubernetes.io/ko/docs/concepts/cluster-administration/manage-deployment/)  
[Kubernetes Manifest](https://kubernetes.io/docs/reference/glossary/?all=true#term-manifest)  
[구성 파일을 이용한 쿠버네티스 오브젝트의 선언형 관리](https://kubernetes.io/ko/docs/tasks/manage-kubernetes-objects/declarative-config/)  
[Kubernetes 디플로이먼트](https://kubernetes.io/ko/docs/concepts/workloads/controllers/deployment/)  
[Kubernetes 서비스](https://kubernetes.io/ko/docs/concepts/services-networking/service/)  
[Performance Plugin](https://www.jenkins.io/doc/pipeline/steps/performance/)  
[Performance Trend Reporting](https://github.com/jenkinsci/performance-plugin/blob/master/docs/Reporting.md)  
