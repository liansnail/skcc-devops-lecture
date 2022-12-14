**[목차]**
- [용어](#용어)
- [추가 정보](#추가-정보)
	- [환경의 변화](#환경의-변화)
	- [보안](#보안)
### 용어
**도커의 아키텍처**  
클라이언트/서버 모델을 따르며, 서버인 도커 데몬이 클라이언트인 도커 커맨드로부터 요청을 받아 동작한다.  

**컨테이너**  
하나의 프로세스. 리눅스의 네임스페이스나 컨트롤 그룹(Cgroups)을 통해 다른 프로세스들과 완전히 분리되어 실행되는 프로세스. 하지만 컨테이너는 정지된 상태로도 관리되기 때문에 정확히는 '실행 가능한 이미지의 인스턴스'이다. docker run 명령어를 통해 이미지는 컨테이너로 변환되어 하나의 인스턴스가 된다. 이 실행 상태의 컨테이너는 IP주소를 가지는 하나의 독립되 서버처럼 동작한다. 컨테이너의 IP주소와 포트번호로 오는 요청은 컨테이너 내의 프로세스로 연결된다.

**이미지**  
읽기 전용인 컨테이너의 템플릿. 컨테이너를 기동하기 위한 실행 파일과 설정 파일의 묶음. 컨테이너를 실행하면 이미지에 담긴 미들웨어나 애플리케이션이 설정에 따라 기동한다.  

**도커 컴포즈(Docerk compose)**  
컨테이너를 이용한 서비스의 개발과 CI를 위해 여러 개의 컨테이너의 옵션과 환경을 정의한 파일을 읽어 여러 컨테이너를 하나의 프로젝트로서 다룰 수 있는 작업환경을 제공한다.

**도커 레지스트리**  
컨테이너의 이미지가 보관되는 곳. 도커는 기본적으로 도커 허브(Docker Hub)에 있는 이미지를 찾도록 설정되어 있다. 참고로 리포지터리는 하나의 이미지에 대해 태그를 사용여 다양한 출시 버전을 함께 보관하는 곳이고, 레지스트리는 리포지터리를 여러 개 가지는 보관 서비스다. 본 교육에서는 비공개 레지스트리 중 하나인 Harbor를 사용한다.  

**레지스트리와 쿠버네티스의 관계**  
docker build로 이미지 빌드 -> docker push로 이미지를 레지스트리에 등록 -> kubectl 커맨드로 매니페스트에 기재한 오브젝트들의 생성을 요청 -> 매니페스트에 기재된 리포지터리로부터 컨테이너의 이미지를 다운로드 -> 컨테이너를 파드 위에서 기동 

[**쿠버네티스(Kubernetes)**](https://kubernetes.io/ko/docs/concepts/overview/what-is-kubernetes/) 

[The Illustrated Children’s Guide to Kubernetes](https://www.cncf.io/phippy/the-childrens-illustrated-guide-to-kubernetes/) 

[쿠버네티스(Kubernetes)란? 개‭념, 성‭능, 사‭용 방‭법 및 차‭이‭점](https://www.redhat.com/ko/topics/containers/what-is-kubernetes)  
구글의 사내 운영 시스템인 Borg를 CNCF(Cloud Native Computing Foundation)에 기증되어 오픈소스 프로젝트로 운영 중이다. 컨테이너화된 애플리케이션을 효율적으로 배포하고 운영하기 위해 설계된 오픈소스 플랫폼. 그리스어로 조타수, 캡틴 등의 의미가 있어 배의 타륜을 그린 로고가 사용된다. 개발자들이 일반적으로 오픈소스르 사용하여 짧은 시간에 고품질의 애플리케이션을 개발하는데, 빠르게 변화하는 오픈소스으 버전 차이로 개발 생산성과 안정성이 떨어지는 문제가 발생하여 애플리케이션 실행에 필요한 라이브러리나 운영체제 패키지 등을 모두 담아서 불변의 실행 환경(Immutable Infrastructure)가 필요해서 탄생하게 된다. 아키텍처는 마스터와 노드로 구성되며, 마스터가 노드를 제어하고, 노드에서 컨테이너가 돌아간다. 쿠버네티스 스택은 주로 도커를 런타임 환경으로 사용했는데, CRI(Container Runtime Interface)로 컨테이너 실행 환경오 연동하는 식으로 발전하고 있다.  

+ 배포 계획에 맞춰 애플리케이션을 신속 배포 가능 (컨테이너 개수, CPU사용률, 메모리 사용량, 저장 공간, 네트워크 접근제어, 로드밸런싱 설정 가능)
+ 가동 중인 애플리케이션을 Scale Up/Down 가능 (컨테이너 수를 늘려 처리능력을 높이거나, 줄여서 자원점유율이나 요금을 줄임)
+ 새로운 버전의 애플리케이션을 무정지로 업그레이드 가능
+ 하드웨어 가동률을 높여 자원 낭비를 줄임  

**쿠버네티스 설치도구**  
+ Minikube : 간편하게 로컬에서 1개의 노드에 쿠버네티스 기본 기능 설치 및 사용, 일부 기능 제한됨.  
+ GKE, EKS 등 완전 관리형 서비스 : 클라우드 플랫폼에 종속적인 기능(로드밸런싱/오토스케일링/퍼시스턴트 스토리지 등)도 사용 가능. 클라우드 사용 비용 및 의존성 증가.  
+ kubespray, kubeadm : 온프레미스, 클라우드 인프라 다 설치 가능. 서버 인프라 및 K8S 관리가 다소 어려울 수 있음.  
+ kops : 특정 클라우드 플랫폼(AWS/GCE/DigitalOcean/OpenStack)에서 쉽게 쿠버네티스 설치 가능. 서버/네트워크 등 각종 인프라도 자동으로 프로비저닝.  

kubeadm은 쿠버네티스를 설치할 서버 인프라를 직접 준비해야 하지만, kops는 서버 인스턴스와 네트워크 리소스 등을 클라우드에서 자동으로 생성해 쿠버네티스를 설치한다.  

**파드(Pod)**  
컨테이너 애플리케이션를 다루는 기본 단위. 포드는 1개 이상의 컨테이너로 구성된 컨테이너의 집합. 참고로 도커 엔진에서는 기본 단위가 도커 컨테이너. 쿠버네티스가 도커 컨테이너가 아닌 포드를 사용하는 이유는 컨테이너 런타임의 인터페이스 제공 등 여러 가지가 있지만, 그 이유 중 하나는 여러 리눅스 네임스페이스(namespace)를 공유하는 여러 컨테이너들을 추상화된 집합으로 사용하기 위해서이다. 즉, 여러 개의 컨테이너가 동일한 네트워크 환경을 가지게 됨(네임스페이스는 컨테이너의 고유한 네트워크 환경을 제공해 주는 역할을 담당)  

**레플리카셋(Replica Set)**
일정 개수의 포드를 유지하는 컨트롤러. 쿠버네티스의 기본 단위인 포드는 여러 개의 컨테이너를 추상화해 하나의 애플리케이션으로 동작하도록 만드는 컨테이너 묶음이지만, YAML에 포드만 정의해 생성하게 되면 이 포드의 생애주기(Lifecycle)은 오직 쿠버네티스 사용자에 의해 관리된다. 단순히 포드의 기능을 테스트하는 등의 간단한 용도로는 괜찮지만, 실제 외부 사용자의 요청을 처리해야 하는 마이크로서비스 구조의 포드라면 적합하지 않다.(마이크로서비스에서는 여러 개의 동일한 컨테이너를 생성한 뒤 외부 요청이 각 컨테이너에 적절히 분해되어야 하기 때문) 따라서 레플리카셋을 통해 정해진 개수의 동일한 포드가 항상 실행되도록 관리하고, 노드 장애 등의 이유로 포드를 사용할 수 없다면 다른 노드에서 포드를 다시 생성하도록 한다. (레플리카셋은 실제로 포드와 연결되어 있지는 않고 느슨한 연결(라벨 셀렉터)를 통해 연결을 유지한다) 

**디플로이먼트(Deployment)**  
레플리카셋, 포드의 배포를 관리. 실제 쿠버네티스 운영 환경에서 레플리카셋을 YAML파일에서 사용하는 경우는 거의 없고, 대부분 레플리카셋과 포드의 정보를 정의하는 디플로이먼트라는 오브젝트를 YAML파일에 정의해 사용한다. 디플로이먼트는 레플리카셋의 상위 오브젝트이기 때문에 디플로이먼트를 생성하면 해당 디플로이먼트에 해당하는 레플리카셋도 함께 생성된다. 따라서 디플로이먼트를 사용하면 포드와 레플리카셋을 직접 생성할 필요없다. 레플리카셋만으로 동일한 개수의 포드를 유지할 수 있지만 디플로이먼트를 통해 간접적으로 레플리카셋을 생성하는 이유는 애플리케이션의 업데이트와 배포를 편하게 만들기 위해서이다. 예를 들어 애플리케이션을 업데이트할 때 레플리카셋의 변경사항을 저장하는 리비전(Revision)을 남겨 롤백을 가능하게 하고, 무중단 서비스를 위해 포드의 롤링 업데이트의 전략을 지정할 수 있다. 즉, 레플리카셋의 리비전 관리 뿐만 아니라 다양한 포드의 롤링 업데이트 정책을 사용할 수도 있다.  

**서비스(Service)**  
포드를 연결하고 외부에 노출. 포드의 IP는 영속적이지 않고 항상 변할 수 있으므로 여러 개의 디플로이먼트를 하나의 애플리케이션으로 연동하려면 포드IP가 아닌, 서로를 발견(Discovery)할 수 있는 방법이 필요하다. 도커와 달리, 쿠버네티스는 디플로이먼트를 생성할 때 포드를 외부로 노출하지 않으며, 디플로이먼트의 YAML파일에는 단지 포드의 애플리케이션이 사용할 내부 포트만 정의하고, 서비스라고 부르는 별도의 쿠버네티스 오브젝트를 생성해 접근한다. 즉, 서비스는 여러 개의 포드에 쉽게 접근할 수 있도록 고유한 도메인 이름을 부여하고, 여러 개의 포드에 접근할 때 요청을 분산하는 로드밸런서 기능을 수행하며, 클라우드 플랫폼의 로드밸런서, 클러스터 노드의 포트 등을 통해 포드를 외부에 노출한다.  
서비스의 종류로는 쿠버네티스 내부에서만 포드들에 접근할 때 사용하는 ClusterIP, 포드에 접근할 수 있는 포트를 클러스터의 모든 노드에 동일하게 개방하여 외부에서 포드에 접근할 수 있는 NodePort, 노드포트 타입과 마찬가지로 외부에서 포드에 접근할 수 있지만 일반적으로 AWS 등 클라우드 플랫폼 환경에서만 사용할 수 있는 LOadBalancer 타입 등이 있다.(로드밸런서 타입은 동적으로 프로비저닝해 포드에 연결됨)  

**네임스페이스(Namespace)**  
리소스를 논리적으로 구분하는 오브젝트. 포드/레플리카셋/디플로이먼트/서비스 등과 같이 쿠버네티스 리소스들이 묶여있는 하나의 그룹. 논리적으로 구분되었으나 물리적으로 격리된 것은 아니여서 서로 다른 네임스페이스에서 생성된 포드가 같은 노드에 존재할 수도 있다. 쿠버네티스 클러스터를 여러 명이 동시에 사용할 때나 특정 목적(모니터링/로드밸런싱 인그레스 등)으로 네임스페이스별로 구분할 때 사용할 수 있다. 리눅스 네임스페이스와는 완전 다르다. 리눅스 네임스페이스는 컨테이너의 격리된 공간을 생성하기 위해 리눅스 커널의 자체 기능을 활용하는 것으로 일반적으로 네트워크나 마운트, 프로세스 네임스페이스 등을 의미한다. 모든 리소스가 네임스페이스에 의해 구분되는 것은 아니다. 노드(nodes) 또한 쿠버네티스의 오브젝트 중 하나이지만, 네임스페이스에 속하지 않는다. nodes는 쿠버네티스 클러스터에서 사용되는 저수준의 오브젝트이며, 네임스페이스에 의해 구분되지 않는다.  

**컨피그맵(ConfigMap)**  
일반적인 설정값을 담아 저장할 수 있는 쿠버네티스 오브젝트. 네임스페이스에 속하기 때문에 네임스페이스별로 컨피그맵이 존재한다.  

**시크릿(Secret)**  
SSH 키, 비밀번호 등과 같이 민감한 정보를 저장하기 위한 용도로 사용되며, 네임스페이스에 종속되는 쿠버네티스 오브젝트.

**kubectx**  
cluster 전환을 쉽게 해주는 쿠버네티스 플러그인.

**kubens**  
namespace 전환을 쉽게 해주는 쿠버네티스 플러그인.

**인그레스(Ingress)**  
일반적으로 외부에서 내부로 향하는 것을 지칭. 외부 요청을 어떻게 처리할 것인지 네트워크 7계층 레벨에서 정의하는 쿠버네티스 오브젝트. SSL/TLS 보안 연결, 접근 도메인 및 클라이언트 상태에 기반한 라우팅 정의 등과 같은 역할을 각 서비스와 디플로이먼트가 아닌 인그레스에서 담당.  

**HELM(헬름)**  
K8S Package로 해서 관리하는 도구. helm chart는 helm의 패키지 포맷이다. 차트는 K8S를 설명하는 파일들의 집합. 만든 애플리케이션을 도커 이미지로 만들어서 가져올 때 하나하나 설정(도메인 이름, 클러스터마다 변동되는 환경변수 등)을 바꿔줘야 하는데, 커스터마이징 가능.

### 추가 정보

#### 환경의 변화
[당황하지 마세요. 쿠버네티스와 도커](https://kubernetes.io/ko/blog/2020/12/02/dont-panic-kubernetes-and-docker/)

#### 보안

toktok 상단 메뉴 '업무지원' > 정보보안 Portal > 정책/가이드 하단
+ AWS 보안 Guide Pack (V2.0)
+ Cloud 보안 Basic 체크리스트
+ Cloud 보안점검 자동화 Tool
