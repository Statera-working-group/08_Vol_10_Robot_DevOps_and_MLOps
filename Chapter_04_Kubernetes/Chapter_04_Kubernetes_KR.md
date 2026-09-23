**Volume 10 Robot DevOps and MLOps**

# 4. Kubernetes

## 4.1. Kubernetes Architecture API Server etcd Scheduler Kubelet

![](images/image1.png){width="7.268055555555556in" height="7.268055555555556in"}

쿠버네티스(Kubernetes)는 여러 컴퓨팅 노드(Node)에 걸쳐 컨테이너화된 애플리케이션(Containerized Application)을 배포하고, 확장하며, 관리하기 위한 분산 오케스트레이션 계층(Distributed Orchestration Layer)을 제공한다. 쿠버네티스는 각각의 컨테이너(Container)나 머신(Machine)을 개별 자원으로 관리하는 대신, 원하는 시스템 상태(Desired System State)를 기술하는 선언적 객체(Declarative Object)를 통해 클러스터(Cluster)를 표현한다. 컨트롤러(Controller)는 이러한 원하는 상태와 실제 상태(Actual State)를 지속적으로 비교하고 차이가 발생하면 이를 수정하기 위한 동작을 수행한다.

쿠버네티스 클러스터(Kubernetes Cluster)는 논리적으로 컨트롤 플레인(Control Plane)과 워커 노드(Worker Node)로 구분된다. 컨트롤 플레인은 클러스터 전체의 구성(Configuration), 스케줄링 결정(Scheduling Decision), 오케스트레이션 로직(Orchestration Logic)을 관리하고, 워커 노드는 파드(Pod) 내부에서 실제 애플리케이션 워크로드(Application Workload)를 실행한다. 이러한 분리는 인프라 환경이 변경될 때 운영자가 개별 프로세스, 컨테이너 또는 서버를 직접 관리하지 않아도 애플리케이션이 여러 머신 사이에서 이동할 수 있도록 한다.

API 서버(API Server)는 쿠버네티스 컨트롤 플레인의 중앙 통신 게이트웨이(Central Communication Gateway)이다. 관리 도구(Administrative Tool), 컨트롤러, 스케줄러(Scheduler), 큐블릿(Kubelet), 외부 자동화 시스템(External Automation System)은 API를 통해 클러스터 자원과 상호작용한다. 운영자가 디플로이먼트 명세(Deployment Specification)를 제출하면 API 서버는 요청을 인증(Authentication) 및 인가(Authorization)하고, 자원 정의(Resource Definition)를 검증한 다음 승인된 클러스터 상태가 영속적으로 저장되도록 조정한다.

이러한 API 중심 아키텍처(API-Centered Architecture)가 중요한 이유는 쿠버네티스 구성요소들이 일반적으로 서로를 직접 조작하여 클러스터 상태를 변경하지 않기 때문이다. 대신 API 서버를 통해 자원 객체(Resource Object)를 관찰하고 갱신한다. 이를 통해 파드, 디플로이먼트(Deployment), 서비스(Service), 노드, 구성 객체(Configuration Object), 보안 정책(Security Policy), 사용자 정의 자원(Custom Resource)을 일관된 제어 인터페이스(Control Interface)로 관리할 수 있으며, 자동화 도구와 CI/CD 시스템 역시 동일한 관리 모델을 사용할 수 있다.

영속적인 클러스터 상태(Persistent Cluster State)는 쿠버네티스 컨트롤 플레인 정보를 위한 권위 있는 데이터 저장소(Authoritative Data Store) 역할을 하는 분산 키-값 저장소(Distributed Key-Value Store)인 etcd에 유지된다. 자원 정의, 구성 메타데이터(Configuration Metadata), 노드 정보 및 기타 영속 상태는 API 서버가 관리하고 etcd에 저장하는 데이터로 표현된다. 이러한 정보가 손실되거나 손상되면 클러스터 복구에 영향을 줄 수 있으므로 etcd의 가용성(Availability), 백업(Backup), 복원(Restoration)은 매우 중요한 운영 요소이다.

스케줄러(Scheduler)는 새롭게 생성된 파드가 어느 위치에서 실행될지를 결정한다. 특정 노드가 할당되지 않은 파드가 존재하면 스케줄러는 자원 요구사항(Resource Requirement)과 스케줄링 제약조건(Scheduling Constraint)을 기준으로 사용 가능한 워커 노드를 평가한다. CPU 및 메모리 요청, 노드 셀렉터(Node Selector), 어피니티(Affinity)와 안티 어피니티(Anti-Affinity), 테인트(Taint)와 톨러레이션(Toleration), 토폴로지 제약조건(Topology Constraint), GPU와 같은 특수 자원(Specialized Resource)이 최종 배치 결정에 영향을 줄 수 있다.

따라서 스케줄링(Scheduling)은 단순한 부하 분산(Load Distribution) 이상의 의미를 가진다. 로봇 워크로드(Robot Workload)는 NVIDIA GPU, 특정 CPU 아키텍처(CPU Architecture), 하드웨어 직접 접근(Direct Hardware Access), 또는 지정된 엣지 위치(Edge Location)에서의 실행을 요구할 수 있다. 쿠버네티스는 이러한 요구사항을 스케줄링 제약조건으로 표현하여 인지(Perception), 추론(Inference), 텔레메트리(Telemetry), 플릿 관리(Fleet Management), 지원 서비스(Supporting Service)가 해당 작업을 올바르게 실행할 수 있는 노드에만 배치되도록 할 수 있다.

스케줄링을 통해 파드가 워커 노드에 할당되면 해당 노드의 큐블릿(Kubelet)이 지정된 워크로드가 실제로 실행되도록 관리한다. 큐블릿은 자신의 노드에 연결된 파드 명세(Pod Specification)를 관찰하고 컨테이너 런타임(Container Runtime)과 통신하여 컨테이너를 생성하고 감독한다. 또한 노드와 워크로드의 상태를 쿠버네티스 컨트롤 플레인으로 보고하여 중앙 집중식 오케스트레이션(Centralized Orchestration)과 각 머신에서 이루어지는 실제 실행을 연결한다.

큐블릿은 컨테이너 실행 기능을 자체적으로 구현하는 대신 컨테이너 런타임 인터페이스(Container Runtime Interface, CRI)를 통해 컨테이너 런타임과 연동한다. 런타임(Runtime)은 컨테이너 이미지(Container Image)를 가져오고 필요한 실행 환경(Execution Environment)을 준비하며 쿠버네티스에서 전달된 요청에 따라 컨테이너를 시작하거나 중지한다. 이러한 분리를 통해 오케스트레이션 계층은 원하는 워크로드 상태 관리에 집중하고 런타임은 저수준 컨테이너 생명주기(Container Lifecycle)를 담당할 수 있다.

컨트롤러(Controller)는 쿠버네티스를 단순한 원격 실행 시스템(Remote Execution System)과 근본적으로 구분하는 지속적인 조정(Reconciliation) 동작을 제공한다. 컨트롤러는 특정 자원을 관찰하고 실제 상태가 선언된 원하는 상태와 일치하는지를 판단하며, 두 상태가 다르면 필요한 동작을 시작한다. 파드가 사라지거나 노드에 장애가 발생하거나 요청된 복제본 수(Replica Count)가 변경되면 컨트롤러는 쿠버네티스 자원을 통해 협력하여 시스템이 요청된 상태로 수렴하도록 한다.

예를 들어 디플로이먼트(Deployment)는 로봇 플릿 백엔드(Robot Fleet Backend)의 여러 복제본(Replica)이 지속적으로 사용 가능한 상태를 유지하도록 선언할 수 있다. 관련 컨트롤러가 필요한 복제 구조(Replica Structure)를 유지하는 동안 스케줄러는 적절한 노드를 선택하고 큐블릿은 생성된 파드를 실행한다. 하나의 파드에 장애가 발생하더라도 쿠버네티스는 관리자가 해당 프로세스를 직접 재시작하는 방식에 의존하지 않고 대체 워크로드 상태를 생성하여 선언된 구성을 복원하도록 한다.

이러한 구성요소 간의 통신은 하나의 제어 루프(Control Loop)를 형성한다. 사용자 또는 자동화 시스템이 원하는 상태를 API 서버에 제출하면 승인된 상태가 영속적으로 저장되고, 컨트롤러가 필요한 변경사항을 감지하며, 스케줄러가 아직 할당되지 않은 파드를 노드에 배치하고, 큐블릿이 워커 노드에서 해당 배치를 실제로 구현한다. 이후 상태 정보가 API를 통해 다시 전달되면서 컨트롤 플레인은 실행 중인 클러스터가 선언된 구성과 일치하는지를 지속적으로 평가한다.

이러한 아키텍처는 서로 다른 배포 생명주기(Deployment Lifecycle)를 가지는 다양한 지원 서비스를 포함하는 로보틱스 플랫폼(Robotics Platform)에서 특히 유용하다. 플릿 API(Fleet API), 텔레메트리 처리기(Telemetry Processor), 데이터베이스(Database), 모니터링 에이전트(Monitoring Agent), 시뮬레이션 서비스(Simulation Service), AI 추론 서버(AI Inference Server), 개발 인프라(Development Infrastructure)를 독립적으로 패키징하면서도 공통된 오케스트레이션 모델 아래에서 관리할 수 있다. 따라서 쿠버네티스는 데이터센터(Data Center), 클라우드 인프라(Cloud Infrastructure), 고성능 엣지 컴퓨터(Edge Computer)에 걸쳐 일관된 운영 계층을 제공할 수 있다.

그러나 쿠버네티스가 로봇 내부의 실시간 실행 메커니즘(Real-Time Execution Mechanism)을 자동으로 대체해야 하는 것은 아니다. 하드 실시간 모터 제어(Hard Real-Time Motor Control), 비상 정지(Emergency Stop), 결정론적 액추에이터 제어 루프(Deterministic Actuator Loop), 엄격한 시간 제약을 가지는 안전 기능(Safety Function)은 일반적으로 전용 실시간 소프트웨어(Real-Time Software) 또는 임베디드 제어 환경(Embedded Control Environment)을 필요로 한다. 쿠버네티스는 이러한 계층의 상위에서 분산 운영 인프라의 타이밍과 장애 특성에 적합한 컨테이너화 서비스를 오케스트레이션하는 역할에 더 적합하다.

엣지 로보틱스 아키텍처(Edge Robotics Architecture)에서는 이러한 역할 구분을 통해 실용적인 계층 구조(Hierarchy)를 구성할 수 있다. 결정론적 제어(Deterministic Control)는 마이크로컨트롤러(Microcontroller), 실시간 운영체제(Real-Time Operating System, RTOS), 실시간 리눅스(Real-Time Linux) 구성요소에 유지하고, 쿠버네티스는 상위 수준의 인지, AI 추론, 데이터 처리(Data Processing), 모니터링(Monitoring), 플릿 연계 애플리케이션(Fleet-Facing Application)을 관리한다. 경량 쿠버네티스 배포판(Lightweight Kubernetes Distribution)은 이러한 관리 모델을 자원이 제한된 엣지 시스템까지 확장할 수 있으며, 이는 이후 다루게 될 로봇 컴퓨팅을 위한 배포 패턴(Deployment Pattern)으로 자연스럽게 연결된다.

## 4.2. K3s Lightweight Kubernetes for Edge Robot Compute [w/Code]

![](images/image2.png){width="7.268055555555556in" height="7.268055555555556in"}

K3s는 기존 클러스터 설치보다 작은 운영 자원 사용량(Operational Footprint)으로 쿠버네티스 호환 오케스트레이션(Kubernetes-Compatible Orchestration)을 제공하도록 설계된 경량 쿠버네티스 배포판(Lightweight Kubernetes Distribution)이다. 필수 쿠버네티스 구성요소를 간결한 배포 형태로 패키징하면서 파드(Pod), 디플로이먼트(Deployment), 서비스(Service), 네임스페이스(Namespace), 선언적 구성(Declarative Configuration)과 같은 핵심 개념을 그대로 유지한다. 따라서 컴퓨팅, 메모리, 저장공간, 관리 자원이 제한될 수 있는 엣지 컴퓨팅(Edge Computing) 환경에 특히 적합하다.

전통적인 쿠버네티스(Kubernetes) 배포는 일반적으로 여러 컨트롤 플레인 서비스(Control Plane Service)를 실행하기에 충분한 자원을 갖춘 서버, 클라우드 인프라(Cloud Infrastructure), 클러스터(Cluster)를 전제로 한다. 반면 엣지 로봇(Edge Robot)은 인지(Perception), 위치추정(Localization), 내비게이션(Navigation), AI 추론(AI Inference), 텔레메트리(Telemetry), 하드웨어 인터페이스(Hardware Interface)를 동시에 실행할 수 있다. 따라서 오케스트레이션 플랫폼은 자체 자원 소비를 최소화하면서도 배포 자동화, 워크로드 격리(Workload Isolation), 복구, 생명주기 관리(Lifecycle Management)를 제공해야 한다.

K3s는 쿠버네티스 설치 및 패키징 과정을 단순화하여 이러한 요구사항에 대응한다. 정상적인 클러스터를 구성하는 데 필요한 많은 구성요소가 통합된 형태로 배포되므로 운영자가 관리해야 하는 개별 패키지와 구성 절차를 줄일 수 있다. 결과적으로 로봇 개발자는 대규모 데이터센터 중심 쿠버네티스 환경을 구축하고 유지할 때 발생하는 복잡성을 그대로 재현하지 않고도 쿠버네티스 호환 환경을 구축할 수 있다.

K3s 시스템은 쿠버네티스에서 사용되는 기본적인 컨트롤 플레인(Control Plane)과 워커 노드(Worker Node) 모델을 따른다. 서버 노드(Server Node)는 클러스터 관리 기능을 제공하고, 에이전트 노드(Agent Node)는 할당된 워크로드를 실행한다. 서버 측은 쿠버네티스 API를 제공하면서 클러스터 상태, 스케줄링(Scheduling), 컨트롤러(Controller)를 조정하고, 에이전트는 워크로드 정의를 전달받아 컨테이너(Container)를 실행하는 데 필요한 노드 측 서비스를 수행한다.

이러한 호환성(Compatibility)은 쿠버네티스용으로 개발된 애플리케이션을 완전히 다른 배포 모델을 도입하지 않고 K3s로 이전할 수 있다는 점에서 운영상 중요하다. 디플로이먼트 매니페스트(Deployment Manifest), 서비스, 컨피그맵(ConfigMap), 시크릿(Secret), 네임스페이스, 자원 요청(Resource Request) 및 기타 쿠버네티스 객체를 기존 소프트웨어 배포 과정에 유지할 수 있다. 따라서 개발팀은 클라우드 클러스터, 온프레미스 인프라(On-Premise Infrastructure), 연구실 컴퓨터, 로봇 엣지 시스템 전반에서 유사한 운영 방식을 유지할 수 있다.

단일 자율주행 로봇(Autonomous Robot)의 경우 K3s는 로봇의 엣지 컴퓨터(Edge Computer)에서 간결한 오케스트레이션 계층(Orchestration Layer)으로 동작할 수 있다. 인지 파이프라인(Perception Pipeline), AI 추론 서버, 텔레메트리 수집기(Telemetry Collector), 진단(Diagnostics), 로깅 에이전트(Logging Agent), 플릿 통신 클라이언트(Fleet Communication Client), 애플리케이션 수준 ROS 2 구성요소 등을 각각 독립된 워크로드로 관리할 수 있다. 이들의 배포 상태를 선언적으로 관리함으로써 여러 로봇에 걸쳐 소프트웨어 구성을 더욱 일관성 있게 재현할 수 있다.

K3s는 하나의 로봇이 여러 컴퓨팅 장치를 포함할 때 더욱 유용해질 수 있다. 고성능 x86 컴퓨터는 GPU 집약적인 인지 또는 추론 작업을 수행하고, ARM 기반 엣지 컴퓨터는 통신, 모니터링 또는 보조 애플리케이션을 처리할 수 있다. 이러한 컴퓨터를 엣지 클러스터(Edge Cluster)의 노드로 구성하면 쿠버네티스 스케줄링 메커니즘이 아키텍처, 자원 가용성, 레이블(Label), 애플리케이션 제약조건에 따라 호환 가능한 워크로드의 실행 위치를 결정할 수 있다.

이러한 환경에서는 자원 관리(Resource Management)가 특히 중요하다. 로봇 컴퓨터에서 오케스트레이션 워크로드가 내비게이션이나 인지에 필요한 자원을 과도하게 소비해서는 안 된다. 쿠버네티스의 자원 요청(Resource Request)과 제한(Resource Limit)을 이용하여 예상 CPU 및 메모리 사용량을 정의할 수 있으며, 노드 레이블(Node Label), 어피니티 규칙(Affinity Rule), 테인트(Taint), 톨러레이션(Toleration)을 통해 특정 애플리케이션을 적절한 하드웨어로 제한할 수 있다. GPU 워크로드 역시 지원되는 가속기 자원을 갖춘 노드로 배치할 수 있다.

컨테이너 이미지(Container Image)는 이기종 로봇 소프트웨어(Heterogeneous Robot Software)를 위한 유용한 배포 경계(Deployment Boundary)를 제공한다. 각각의 서비스는 서로 다른 버전의 라이브러리, 미들웨어(Middleware), 추론 프레임워크(Inference Framework), 시스템 의존성(System Dependency)을 요구할 수 있다. 이를 개별적으로 패키징하면 의존성 충돌을 줄이고 소프트웨어 버전을 더욱 쉽게 재현할 수 있다. K3s는 여러 머신별 시작 스크립트에 의존하는 대신 이러한 컨테이너를 시작, 중지, 교체, 모니터링하는 공통 메커니즘을 제공한다.

쿠버네티스로부터 상속받은 자가 치유(Self-Healing) 동작은 장시간 운영되는 자율 시스템(Autonomous System)에 유용하다. 관리 대상 애플리케이션 컨테이너가 예기치 않게 종료되면 오케스트레이션 계층은 선언된 워크로드 상태를 복원하려고 시도할 수 있다. 상태 점검(Health Check)을 통해 애플리케이션이 정상적으로 동작하는지 판단할 수 있으며, 컨트롤러는 요청된 배포 구성을 유지한다. 이러한 기능이 로봇의 안전 자체를 보장하는 것은 아니지만 적절하게 컨테이너화된 비실시간 서비스(Non-Real-Time Service)의 운영 복원력(Operational Resilience)을 향상시킨다.

엣지 네트워킹(Edge Networking)은 추가적인 설계 고려사항을 발생시킨다. 로봇은 여러 무선 네트워크 사이를 이동하거나 LTE 또는 5G 연결을 통해 동작할 수 있으며, 일시적으로 클라우드 연결을 잃거나 지연시간과 대역폭이 변화하는 환경을 경험할 수 있다. 따라서 로컬 K3s 환경은 중앙 클라우드 시스템에 항상 연결되어 있다고 가정해서는 안 된다. 핵심 엣지 애플리케이션은 외부 연결 품질이 저하되거나 일시적으로 단절되더라도 로컬 기능을 계속 수행할 수 있어야 한다.

이러한 특성은 중앙 인프라가 소프트웨어 아티팩트(Software Artifact), 구성(Configuration), 모니터링, 플릿 수준 정책(Fleet-Level Policy)을 관리하면서 각 로봇은 자율 운용에 필요한 충분한 로컬 컴퓨팅 능력을 유지하는 클라우드-엣지 아키텍처(Cloud-Edge Architecture)로 자연스럽게 연결된다. 컨테이너 이미지와 구성은 중앙에서 준비하여 통제된 배포 프로세스를 통해 엣지 시스템으로 전달할 수 있으며, 네트워크 연결이 가능한 경우 텔레메트리와 운영 상태 정보는 반대 방향으로 중앙 시스템에 전달될 수 있다.

K3s는 다중 로봇 운영 일관성(Multi-Robot Operational Consistency)을 지원하는 데에도 활용할 수 있다. 각 로봇을 개별적으로 수동 구성하는 대신 플릿(Fleet)은 표준화된 매니페스트(Standardized Manifest)와 버전 관리된 구성(Version-Controlled Configuration)을 사용하여 공통 애플리케이션 스택(Application Stack)을 정의할 수 있다. 하드웨어별 차이는 레이블, 구성 객체(Configuration Object), 배포 정책(Deployment Policy)을 통해 표현할 수 있다. 이러한 접근법은 구성 드리프트(Configuration Drift)를 줄이고 깃옵스(GitOps), 단계적 롤아웃(Staged Rollout), 롤백(Rollback), 관측 가능성(Observability), 플릿 전체 소프트웨어 생명주기 관리의 기반을 제공한다.

쿠버네티스 배포판이 경량화되어 있더라도 보안(Security)은 여전히 핵심 요소이다. API 접근, 자격증명(Credential), 인증서(Certificate), 시크릿, 컨테이너 권한(Container Privilege), 네트워크 노출(Network Exposure), 호스트 장치 접근(Host-Device Access)을 신중하게 관리해야 한다. 로봇 애플리케이션은 카메라, 라이다(LiDAR), GPU, 직렬 장치(Serial Device), CAN 인터페이스 또는 기타 하드웨어에 접근해야 하는 경우가 많지만, 단순한 편의성을 위해 제한 없는 호스트 권한을 부여하면 격리 수준이 약화되고 소프트웨어 침해 시 피해 범위가 확대될 수 있다.

따라서 K3s는 하드 실시간 로봇 기능(Hard Real-Time Robot Function)을 위한 주 제어기(Primary Controller)가 아니라 애플리케이션 오케스트레이션 계층으로 배치해야 한다. 모터 제어 루프(Motor Control Loop), 비상 정지 처리(Emergency-Stop Processing), 저수준 액추에이터 안전(Low-Level Actuator Safety), 기타 결정론적 기능(Deterministic Function)은 적절한 마이크로컨트롤러(Microcontroller), 실시간 운영체제(Real-Time Operating System, RTOS), 안전 제어기(Safety Controller), 실시간 리눅스(Real-Time Linux) 환경에 유지해야 한다. K3s는 이러한 하위 계층이 컨테이너 오케스트레이션과 독립적으로 결정론적 동작을 유지하는 동안 상위 수준 서비스를 관리할 수 있다.

따라서 엣지 로보틱스(Edge Robotics)에서 K3s의 핵심 아키텍처적 가치는 단순히 크기가 작은 쿠버네티스 배포판이라는 점에 있지 않다. K3s는 현대적인 클라우드 네이티브 소프트웨어 운영(Cloud-Native Software Operations)과 자원이 제한된 물리적 머신(Physical Machine)을 연결하는 가교 역할을 한다. 쿠버네티스 방식의 배포, 스케줄링, 복구, 구성, 생명주기 관리를 로봇 컴퓨터까지 확장함으로써 K3s는 안전 필수 실시간 제어(Safety-Critical Real-Time Control)를 오케스트레이션 계층에 강제로 포함시키지 않으면서 엣지 애플리케이션이 통합된 데브옵스 아키텍처(DevOps Architecture)에 참여할 수 있도록 한다.

## 4.3. Robot SW Deployment StatefulSet DaemonSet Patterns [w/Code]

![](images/image3.png){width="7.268055555555556in" height="7.268055555555556in"}

쿠버네티스(Kubernetes)는 로봇 소프트웨어(Robot Software)를 배포하기 위한 여러 워크로드 컨트롤러(Workload Controller)를 제공하며, 적절한 컨트롤러의 선택은 각 서비스의 운영 특성에 따라 달라진다. 디플로이먼트(Deployment)는 대부분 서로 교체 가능한 무상태 애플리케이션(Stateless Application)에 적합하고, 스테이트풀셋(StatefulSet)과 데몬셋(DaemonSet)은 영속적인 식별자(Persistent Identity), 순서가 있는 생명주기 동작(Ordered Lifecycle Behavior), 특정 노드에서의 실행이 필요한 워크로드를 담당한다. 로봇 플랫폼은 클라우드 형태의 서비스와 하드웨어 종속적인 엣지 프로세스(Edge Process)를 함께 포함하므로 이러한 세 가지 패턴을 모두 필요로 하는 경우가 많다.

스테이트풀셋(StatefulSet)은 각각의 애플리케이션 인스턴스(Application Instance)를 항상 서로 교체 가능한 복제본(Replica)으로 취급할 수 없는 경우를 위해 설계되었다. 각 파드(Pod)는 재스케줄링(Rescheduling)이나 재시작 이후에도 논리적으로 유지될 수 있는 안정적인 식별자(Stable Identity)를 부여받는다. 이러한 특성은 단순히 동일한 파드를 임의의 수만큼 실행하는 것이 아니라 예측 가능한 이름, 영속 스토리지(Persistent Storage), 순차적인 초기화, 애플리케이션 인스턴스 간의 안정적인 관계가 필요한 소프트웨어에 유용하다.

영속 스토리지는 스테이트풀셋을 사용하는 가장 중요한 이유 중 하나이다. 영속 볼륨 클레임(Persistent Volume Claim, PVC)을 통해 각각의 복제본은 컨테이너나 파드가 다시 생성되더라도 자신의 식별자와 연결된 스토리지 자원을 유지할 수 있다. 로보틱스 인프라(Robotics Infrastructure)에서는 로컬 데이터베이스(Local Database), 영속 메타데이터 서비스(Persistent Metadata Service), 지도 저장소(Map Repository), 임무 상태 저장소(Mission-State Store)와 같이 파드가 교체되더라도 운영 데이터가 자동으로 손실되어서는 안 되는 애플리케이션에 이러한 패턴을 적용할 수 있다.

안정적인 네트워크 식별자(Stable Network Identity) 역시 중요한 특징이다. 스테이트풀셋의 파드는 서비스 엔드포인트(Service Endpoint) 뒤에서 익명의 복제본으로만 취급되지 않고 예측 가능한 이름을 부여받는다. 따라서 필요한 경우 애플리케이션은 분산 시스템(Distributed System)의 특정 구성원을 식별할 수 있다. 이는 클러스터형 데이터베이스(Clustered Database), 복제 스토리지 서비스(Replicated Storage Service), 조정 시스템(Coordination System), 초기화와 복구 과정에서 안정적인 식별자가 필요한 로봇 인프라 구성요소에 유용할 수 있다.

스테이트풀셋은 배포 및 확장 과정에서 제어된 순서(Controlled Ordering)도 지원한다. 일부 분산 애플리케이션은 다른 인스턴스를 초기화하기 전에 특정 인스턴스가 먼저 사용 가능한 상태가 되어야 하거나, 축소(Scale-Down) 과정에서 예측 가능한 종료 동작이 필요할 수 있다. 쿠버네티스는 일반적인 디플로이먼트보다 이러한 생명주기를 더욱 세밀하게 관리할 수 있지만, 애플리케이션 수준의 일관성(Application-Level Consistency), 복제(Replication), 리더 선출(Leader Election), 데이터 복구(Data Recovery)는 여전히 스테이트풀셋을 사용하는 소프트웨어 자체에서 구현해야 한다.

데몬셋(DaemonSet)은 이와 다른 배포 문제를 해결한다. 지정된 수의 교체 가능한 복제본을 유지하는 대신 특정 파드가 모든 적격 노드(Eligible Node), 또는 지정된 스케줄링 조건(Scheduling Condition)에 일치하는 모든 노드에서 실행되도록 한다. 새로운 노드가 클러스터에 추가되면 해당 데몬셋 워크로드가 자동으로 배치될 수 있으며, 노드가 사라지면 해당 노드에 연결된 파드 역시 자연스럽게 제거된다.

이러한 동작은 노드 수준 로봇 인프라(Node-Level Robot Infrastructure)에 적합하다. 로깅 에이전트(Logging Agent), 메트릭 수집기(Metrics Collector), 하드웨어 모니터링 서비스(Hardware Monitoring Service), 보안 에이전트(Security Agent), 네트워크 구성요소(Network Component), 장치 탐색 프로세스(Device Discovery Process), 진단 수집기(Diagnostic Collector)는 로봇 컴퓨팅 노드가 존재하는 위치마다 실행되어야 하는 경우가 많다. 데몬셋을 사용하면 운영자가 각 엣지 컴퓨터에 하나씩 수동으로 구성하고 배포하지 않아도 이러한 기능이 클러스터 토폴로지(Cluster Topology)를 자동으로 따라갈 수 있다.

로봇 하드웨어 인터페이스(Robot Hardware Interface) 역시 적절한 제약조건을 적용한다면 데몬셋 패턴을 활용할 수 있다. 예를 들어 카메라, 라이다(LiDAR) 인터페이스, GPU, CAN 어댑터(Adapter), 특수 가속기(Specialized Accelerator)를 갖춘 노드에 해당 기능을 나타내는 레이블(Label)을 지정할 수 있다. 이후 노드 셀렉터(Node Selector), 어피니티 규칙(Affinity Rule), 테인트(Taint), 톨러레이션(Toleration)을 이용하면 데몬셋 파드를 전체 클러스터에 무차별적으로 실행하는 대신 해당 하드웨어를 가진 노드에서만 실행하도록 제한할 수 있다.

디플로이먼트, 스테이트풀셋, 데몬셋의 구분은 단순한 편의성이 아니라 소프트웨어의 역할(Software Responsibility)을 기준으로 결정해야 한다. 서로 교체 가능한 복제본으로 구성된 플릿 연계 REST 서비스(Fleet-Facing REST Service)는 디플로이먼트에 적합할 수 있고, 영속 데이터베이스(Persistent Database)는 스테이트풀셋의 특성이 필요할 수 있다. 적격 로봇 컴퓨터마다 하나씩 실행되어야 하는 텔레메트리 또는 하드웨어 모니터링 에이전트는 데몬셋에 적합할 수 있다. 워크로드의 동작 특성에 따라 컨트롤러를 선택하면 장애 처리(Failure Handling)와 생명주기 관리를 더욱 예측 가능하게 만들 수 있다.

컨테이너화된 ROS 2 소프트웨어(Containerized ROS 2 Software)는 프로세스가 네트워크 디스커버리(Network Discovery), 장치 접근(Device Access), 공유 메모리(Shared Memory), 타이밍 동작(Timing Behavior), 다른 노드와의 관계에 의존할 수 있기 때문에 추가적인 고려가 필요하다. 모든 ROS 2 프로세스를 독립적인 쿠버네티스 워크로드로 패키징한다고 해서 반드시 이점이 발생하는 것은 아니다. 배포 경계(Deployment Boundary)는 장애 격리(Failure Isolation), 자원 소유권(Resource Ownership), 업데이트 주기(Update Frequency), 하드웨어 의존성(Hardware Dependency), 통신 특성(Communication Characteristic), 개별 구성요소 재시작에 따른 운영상의 영향을 고려하여 결정해야 한다.

따라서 하나의 로봇 엣지 클러스터(Robot Edge Cluster)는 여러 배포 패턴을 동시에 조합할 수 있다. 인지 또는 AI 추론 서비스는 복제본을 서로 교체할 수 있다면 디플로이먼트로 실행하고, 노드 수준 모니터링은 데몬셋으로 실행하며, 영속적인 운영 서비스는 스테이트풀셋으로 실행할 수 있다. 스케줄링 제약조건을 이용하여 GPU 집약적 워크로드를 가속기 노드에 배치하고 하드웨어 인터페이스 서비스를 센서와 물리적으로 연결된 컴퓨터에 배치하는 동시에 일반적인 서비스는 호환 가능한 컴퓨팅 자원 사이에서 이동할 수 있도록 구성할 수 있다.

컨트롤러 유형과 관계없이 상태 관리(Health Management)는 매우 중요하다. 쿠버네티스 프로브(Kubernetes Probe)를 사용하면 애플리케이션이 정상적으로 시작되었는지, 계속 실행되고 있는지, 종속 트래픽을 받을 준비가 되었는지를 판단하는 데 도움을 받을 수 있다. 비정상 컨테이너를 재시작하면 소프트웨어 서비스를 복구할 수 있지만 로보틱스에서는 상태의 의미를 신중하게 해석해야 한다. 예를 들어 인지 프로세스가 재시작되면 쿠버네티스가 컨테이너 자체를 성공적으로 복원하더라도 일정 시간 동안 하위 또는 후속 시스템에서 사용하는 데이터가 유효하지 않을 수 있다.

따라서 롤링 업데이트(Rolling Update)는 물리 시스템(Physical System)의 동작을 고려해야 한다. 웹 백엔드(Web Backend)를 업데이트하는 것과 실제 운행 중인 자율주행 로봇의 내비게이션 소프트웨어를 업데이트하는 것은 서로 다른 결과를 가져온다. 로봇 배포에서는 파드를 교체하기 전에 단계적 롤아웃(Staged Rollout), 유지보수 모드(Maintenance Mode), 임무 인식 업데이트 정책(Mission-Aware Update Policy), 준비 상태 게이트(Readiness Gate), 플릿 관리 시스템과의 조정이 필요할 수 있다. 쿠버네티스는 배포 메커니즘을 제공하지만 실제 물리 시스템에서 언제 소프트웨어를 안전하게 교체할 수 있는지는 상위 수준의 운영 로직(Operational Logic)이 결정해야 한다.

영속 워크로드(Persistent Workload) 역시 동일한 주의가 필요하다. 스테이트풀셋은 식별자와 스토리지의 관계를 유지하지만 그 자체로 애플리케이션 수준의 데이터 무결성(Data Integrity)을 보장하지는 않는다. 데이터베이스와 임무 상태 서비스에는 적절한 복제, 트랜잭션 처리(Transaction Handling), 백업, 복구, 데이터 손상 방지(Corruption Protection)가 여전히 필요하다. 마찬가지로 데몬셋은 배치 의도(Placement Intent)를 보장하지만 기능적 정상 상태(Functional Correctness)를 보장하지는 않는다. 하드웨어 에이전트가 파드로 정상 실행 중이더라도 관리 대상 물리 장치가 연결 해제되었거나 고장난 상태일 수 있다.

로봇 플릿(Robot Fleet)에서 이러한 워크로드 패턴은 표준화된 소프트웨어 배포(Standardized Software Deployment)를 위한 기본 구성요소가 된다. 공통 매니페스트(Common Manifest)를 사용하여 어떤 서비스가 무상태(Stateless), 상태 유지형(Stateful), 노드 종속형(Node-Bound)인지 정의하고, 구성 및 스케줄링 정책을 통해 서로 다른 로봇 모델에 맞게 조정할 수 있다. 이를 버전 관리(Version Control) 및 자동화된 배포 파이프라인(Automated Deployment Pipeline)과 결합하면 수동 구성을 줄이고 개발 시스템, 시험 로봇, 양산 플릿(Production Fleet), 엣지 클러스터 전반에서 반복 가능한 소프트웨어 구성을 구현할 수 있다.

아키텍처의 목표는 모든 로봇 프로세스를 쿠버네티스 아래에 배치하는 것이 아니라 각각의 소프트웨어 역할을 적절한 실행 환경(Execution Environment)에 매핑하는 것이다. 스테이트풀셋은 안정적인 식별자와 영속 상태(Persistent State)를 제공하고, 데몬셋은 토폴로지 인식 노드 수준 실행(Topology-Aware Node-Level Execution)을 제공하며, 디플로이먼트는 교체 가능한 애플리케이션 복제본을 제공한다. 이들을 함께 사용하면 쿠버네티스 또는 K3s가 상위 수준 로봇 소프트웨어를 관리하는 동안 결정론적 제어(Deterministic Control)와 안전 필수 기능(Safety-Critical Function)은 전용 실시간 계층(Dedicated Real-Time Layer)에 유지할 수 있다.

## 4.4. Kubernetes Networking CNI Service Ingress for Robots [w/Code]

![](images/image4.png){width="7.268055555555556in" height="7.268055555555556in"}

쿠버네티스 네트워킹(Kubernetes Networking)은 파드(Pod), 서비스(Service), 노드(Node), 외부 시스템(External System)이 클러스터(Cluster) 전체에서 데이터를 교환할 수 있도록 하는 통신 기반을 제공한다. 로보틱스(Robotics)에서는 인지(Perception), 텔레메트리(Telemetry), 플릿 API(Fleet API), ROS 2 서비스, 모니터링(Monitoring), 엣지-클라우드 통신(Edge-to-Cloud Communication)까지 지원해야 한다. 따라서 네트워크 계층(Network Layer)은 연결성, 성능, 격리, 이동성, 운영 신뢰성 사이의 균형을 고려해야 한다.

쿠버네티스 네트워크 모델(Kubernetes Network Model)은 각 파드가 고유한 네트워크 식별자(Network Identity)를 가지고, 애플리케이션 수준의 주소 변환(Address Translation) 없이 다른 접근 가능한 파드와 통신할 수 있다는 개념을 기반으로 한다. 이러한 모델은 컨테이너가 다시 생성되거나 다른 노드로 이동하더라도 일관된 네트워크 추상화(Network Abstraction)를 통해 워크로드 간 통신을 가능하게 한다. 실제 연결 기능은 클러스터 네트워킹 인프라(Cluster Networking Infrastructure)가 구현한다.

컨테이너 네트워크 인터페이스(Container Network Interface, CNI)는 쿠버네티스가 파드와 네트워크 구현을 연동하는 메커니즘을 정의한다. 파드가 생성되면 노드는 설정된 CNI 구현을 호출하여 네트워크 인터페이스(Network Interface)를 구성하고 주소 정보를 할당하며 파드를 클러스터 네트워크에 연결한다. 따라서 쿠버네티스는 네트워킹 모델을 정의하고, CNI 호환 플러그인(CNI-Compatible Plugin)이 실제 패킷 전달(Packet Forwarding)과 네트워크 동작을 구현한다.

서로 다른 CNI 구현은 라우팅(Routing), 오버레이 네트워크(Overlay Network), 네트워크 정책(Network Policy) 적용, 암호화(Encryption), 관측 가능성(Observability), 물리적 인프라 연동과 같은 기능을 제공할 수 있다. 로보틱스에 적합한 선택은 배포 환경에 따라 달라진다. 데이터센터 플릿 백엔드(Data-Center Fleet Backend), 로봇 내부 엣지 클러스터(Edge Cluster), 무선 네트워크로 연결된 여러 로봇 컴퓨터는 지연시간, 대역폭, 토폴로지(Topology), 장애 복구 측면에서 서로 다른 요구사항을 가질 수 있다.

파드는 정상적인 쿠버네티스 운영 과정에서도 제거되고 다시 생성될 수 있기 때문에 파드 주소(Pod Address)는 동적으로 변경될 수 있다. 따라서 애플리케이션은 특정 파드 IP 주소(Pod IP Address)에 직접 의존하지 않는 것이 바람직하다. 쿠버네티스 서비스(Kubernetes Service)는 변화하는 여러 파드 앞에 안정적인 논리적 엔드포인트(Logical Endpoint)를 제공한다. 이를 통해 클라이언트는 현재 어떤 개별 파드가 기능을 제공하는지 알지 못해도 애플리케이션과 통신할 수 있다.

클러스터IP 서비스(ClusterIP Service)는 주로 클러스터 내부에서 애플리케이션을 노출하며 내부 구성요소 간 통신에 유용하다. 예를 들어 인지 메타데이터 처리(Perception Metadata Processing), 임무 관리(Mission Management), 텔레메트리 집계(Telemetry Aggregation), 내부 API(Internal API)는 직접적인 파드 주소 대신 안정적인 서비스 이름(Service Name)을 통해 통신할 수 있다. 쿠버네티스 DNS(Kubernetes DNS)는 애플리케이션이 변화하는 네트워크 주소를 구성에 직접 기록하지 않고 예측 가능한 이름으로 서비스를 탐색하도록 지원한다.

다른 서비스 유형(Service Type)을 사용하면 내부 클러스터 외부로 워크로드를 노출할 수 있다. 노드포트(NodePort)는 클러스터 노드의 지정된 포트를 통해 서비스에 접근할 수 있도록 하며, 로드밸런서(LoadBalancer)는 외부 로드밸런싱 엔드포인트(Load-Balancing Endpoint)를 제공할 수 있는 인프라와 연동할 수 있다. 적절한 방식은 클러스터가 클라우드 환경, 온프레미스 서버 환경(On-Premise Server Environment), 또는 일반적인 클라우드 로드밸런서가 존재하지 않을 수 있는 로봇 엣지 네트워크(Robot Edge Network) 중 어디에서 운영되는지에 따라 달라진다.

인그레스(Ingress)는 외부에서 들어오는 HTTP 및 HTTPS 트래픽을 서비스로 전달하기 위한 애플리케이션 수준 메커니즘(Application-Level Mechanism)을 제공한다. 외부에 노출되는 각각의 애플리케이션을 독립적으로 공개하는 대신 인그레스 구성(Ingress Configuration)을 통해 호스트 이름(Hostname)이나 URL 경로(URL Path)에 따른 라우팅 규칙을 정의할 수 있다. 인그레스 컨트롤러(Ingress Controller)는 이러한 규칙을 해석하고 실제 트래픽 처리를 수행하여 대시보드(Dashboard), 플릿 API, 모니터링 인터페이스, 관리 애플리케이션이 통제된 외부 진입점(External Entry Point)을 공유하도록 한다.

인그레스를 모든 로봇 통신 프로토콜(Robot Communication Protocol)을 해결하는 범용 방법으로 이해해서는 안 된다. 많은 로보틱스 워크로드는 UDP, DDS, ROS 2 디스커버리 트래픽(Discovery Traffic), 스트리밍 프로토콜(Streaming Protocol), 사용자 정의 TCP 통신(Custom TCP Communication), 직접 장치 인터페이스(Direct Device Interface)를 사용하며, 이러한 통신은 HTTP 중심의 라우팅 구조에 자연스럽게 대응하지 않을 수 있다. 따라서 서비스, 호스트 네트워킹(Host Networking), 특수 게이트웨이(Specialized Gateway), CNI 구성 또는 기존 인그레스 모델 외부의 네트워크 설계가 필요할 수 있다.

ROS 2는 일반적으로 DDS 기반 디스커버리(DDS-Based Discovery)와 데이터 교환에 의존하므로 특히 중요한 네트워킹 고려사항을 가진다. 멀티캐스트(Multicast) 동작, 인터페이스 선택(Interface Selection), 서비스 품질(Quality of Service, QoS), 컨테이너 경계(Container Boundary), 서브넷(Subnet) 간 통신은 ROS 2 노드가 서로를 올바르게 발견할 수 있는지에 영향을 줄 수 있다. 따라서 일반적인 REST 마이크로서비스(REST Microservice)에 적합한 쿠버네티스 네트워크라도 ROS 2 통신을 투명하게 지원하기 위해서는 추가적인 설계와 검증이 필요할 수 있다.

호스트 네트워킹(Host Networking)은 파드가 노드의 네트워크 네임스페이스(Network Namespace)를 직접 사용하도록 하여 로봇 하드웨어 네트워크나 DDS 통신에 대한 접근을 단순화할 수 있다. 그러나 이러한 방식은 네트워크 격리(Network Isolation)를 감소시키고 포트 충돌(Port Conflict)을 발생시키거나 보안 노출(Security Exposure)을 증가시킬 수 있다. 따라서 모든 컨테이너화된 로봇 프로세스에 기본적으로 적용하기보다 명확한 요구사항이 있는 워크로드에 선택적으로 적용해야 한다.

네트워크 정책(NetworkPolicy)은 선택된 네트워킹 구현이 이를 지원하는 경우 워크로드 사이에서 어떤 네트워크 흐름(Network Flow)을 허용할 것인지 정의하는 중요한 보안 메커니즘이다. 로봇 플랫폼에서는 이러한 정책을 이용하여 인지 서비스, 플릿 클라이언트(Fleet Client), 데이터베이스(Database), 모니터링 시스템, 관리 API 사이의 통신을 제한할 수 있다. 이를 통해 불필요한 연결을 줄이고 컨테이너가 침해되거나 서비스가 잘못 구성되었을 때 발생할 수 있는 피해 범위를 축소할 수 있다.

로봇 엣지 네트워크는 불안정한 외부 연결(Unreliable External Connectivity)도 고려해야 한다. 자율이동로봇(Autonomous Mobile Robot)은 여러 Wi-Fi 액세스 포인트(Access Point) 사이를 이동하거나 통신 품질이 낮은 영역을 통과할 수 있으며, 변화하는 지연시간과 대역폭을 가진 LTE 또는 5G 연결을 사용할 수도 있다. 따라서 로컬 쿠버네티스 또는 K3s 네트워킹은 중앙 플릿 시스템이나 클라우드 플랫폼에 일시적으로 연결되지 않더라도 필수적인 온보드 서비스(Onboard Service)가 계속 통신할 수 있도록 설계해야 한다.

이러한 요구사항은 로컬 로봇 트래픽(Local Robot Traffic)과 외부 플릿 트래픽(External Fleet Traffic)의 분리를 권장한다. 고속 센서 스트림(High-Rate Sensor Stream), 로컬 인지 결과, 내비게이션 메시지, 제어 관련 데이터는 가능한 경우 로봇 또는 로컬 엣지 네트워크 내부에 유지하는 것이 바람직하다. 반면 플릿 상태, 임무 명령(Mission Command), 소프트웨어 관리, 선별된 텔레메트리, 집계 이벤트(Aggregated Event)는 통제된 인터페이스를 통해 외부 네트워크 경계를 통과하도록 하여 대역폭 소비와 원격 연결에 대한 의존성을 줄일 수 있다.

다중 컴퓨터 로봇(Multi-Computer Robot)은 추가적인 복잡성을 가진다. x86 GPU 컴퓨터, ARM 엣지 제어기(Edge Controller), 추가 센서 처리 컴퓨터가 동일한 K3s 클러스터에 참여하면서 이더넷(Ethernet) 또는 다른 온보드 네트워크(Onboard Network)를 통해 통신할 수 있다. 이러한 이기종 컴퓨팅 노드(Heterogeneous Computing Node) 사이에서 워크로드가 예측 가능하게 통신하려면 CNI 구성, 노드 주소 지정(Node Addressing), 라우팅, MTU 설정, 방화벽 규칙(Firewall Rule), 인터페이스 선택을 일관성 있게 관리해야 한다.

네트워크 설계에서는 쿠버네티스가 관리하는 애플리케이션과 결정론적 로봇 제어(Deterministic Robot Control)의 분리도 유지해야 한다. 비상 정지 신호(Emergency-Stop Signal), 안전 버스(Safety Bus), 엄격한 시간 제한을 가진 모터 제어 통신(Motor-Control Communication), 기타 안전 필수 통신 경로(Safety-Critical Communication Path)를 일반적인 쿠버네티스 네트워킹에만 의존하도록 구성해서는 안 된다. 이러한 기능은 일반적으로 전용 실시간 또는 안전 통신 계층(Dedicated Real-Time or Safety Communication Layer)에 배치하고, 쿠버네티스 네트워킹은 상위 수준 소프트웨어 통신, 텔레메트리, 모니터링, API, 애플리케이션 데이터를 관리하는 것이 적절하다.

따라서 로봇 플릿(Robot Fleet)에서 CNI, 서비스(Service), 인그레스(Ingress)는 서로 경쟁하는 메커니즘이 아니라 상호 보완적인 계층을 구성한다. CNI는 파드 연결성(Pod Connectivity)을 구축하고, 서비스는 변화하는 워크로드에 대한 안정적인 탐색과 접근을 제공하며, 인그레스는 클러스터로 들어오는 선택된 애플리케이션 수준 트래픽을 제어한다. 이를 DNS, 보안 정책(Security Policy), 엣지 인식 라우팅(Edge-Aware Routing), 실시간 기능의 적절한 분리와 결합하면 쿠버네티스 및 K3s 기반 로봇 소프트웨어 배포를 위한 확장 가능한 네트워킹 기반을 구축할 수 있다.

## 4.5. Persistent Storage PV PVC StorageClass for Robot Data [w/Code]

![](images/image5.png){width="7.268055555555556in" height="7.268055555555556in"}

영속 스토리지(Persistent Storage)는 쿠버네티스 기반 로봇 시스템(Kubernetes-Based Robot System)에서 필수적이다. 컨테이너(Container)와 파드(Pod)는 교체 가능한 구조로 설계되지만, 다양한 로봇 데이터(Robot Data)는 재시작, 재스케줄링(Rescheduling), 소프트웨어 업데이트, 노드 유지보수 이후에도 보존되어야 하기 때문이다. 지도(Map), 임무 기록(Mission Record), 캘리브레이션 파라미터(Calibration Parameter), 로그(Log), 데이터베이스(Database), 학습 모델(Learned Model), 진단 이력(Diagnostic History), 선별된 센서 데이터(Sensor Data)는 이를 생성하거나 사용하는 컨테이너보다 훨씬 긴 생명주기를 가질 수 있다. 쿠버네티스는 이러한 영속 데이터 요구사항을 애플리케이션 생명주기(Application Lifecycle)와 분리하여 관리한다.

영속 볼륨(PersistentVolume, PV)은 쿠버네티스 클러스터(Kubernetes Cluster)에 제공되는 스토리지 용량(Storage Capacity)을 나타낸다. 실제 스토리지는 로컬 디스크(Local Disk), 네트워크 연결 스토리지(Network-Attached Storage), 클라우드 스토리지 시스템(Cloud Storage System), 분산 스토리지 플랫폼(Distributed Storage Platform) 또는 기타 지원되는 스토리지 백엔드(Storage Backend)에서 제공될 수 있다. 쿠버네티스는 이러한 용량을 표준화된 자원 추상화(Resource Abstraction)를 통해 제공하므로 애플리케이션은 물리적인 스토리지 구현 세부사항을 컨테이너 구성에 직접 포함하지 않고 영속 스토리지를 사용할 수 있다.

영속 볼륨 클레임(PersistentVolumeClaim, PVC)은 애플리케이션이 요청하는 영속 스토리지 요구사항을 나타낸다. 파드가 특정 디스크나 스토리지 서버를 직접 지정하도록 하는 대신 워크로드(Workload)는 PVC를 통해 필요한 용량과 접근 요구사항(Access Requirement) 등의 특성을 지정한다. 쿠버네티스는 이 클레임(Claim)을 적절한 PV에 바인딩(Binding)하여 애플리케이션이 필요로 하는 스토리지와 인프라 관리자 또는 스토리지 시스템이 실제 용량을 제공하는 방식을 분리한다.

이러한 분리는 동일한 애플리케이션 소프트웨어가 서로 다른 하드웨어 구성에서 실행될 수 있는 로봇 플릿(Robot Fleet)에서 특히 유용하다. 개발용 로봇은 로컬 SSD에 데이터를 저장하고, 양산 로봇(Production Robot)은 이중화된 온보드 스토리지(Redundant Onboard Storage)를 사용할 수 있으며, 데이터센터 서비스(Data-Center Service)는 네트워크 또는 클라우드 스토리지를 사용할 수 있다. 물리적인 구현 방식이 배포 환경에 따라 변경되더라도 애플리케이션은 동일한 쿠버네티스 추상화를 통해 스토리지를 계속 요청할 수 있다.

스토리지클래스(StorageClass)는 스토리지의 종류를 정의하고 스토리지 환경에서 지원하는 경우 동적 프로비저닝(Dynamic Provisioning)을 가능하게 함으로써 이러한 모델을 확장한다. 관리자가 모든 PV를 사전에 수동으로 생성하는 대신 PVC가 특정 스토리지클래스를 요청하면 적절한 스토리지를 자동으로 생성하도록 할 수 있다. 서로 다른 클래스는 로컬 고성능 스토리지(Local High-Performance Storage), 네트워크 스토리지(Network Storage), 아카이브 스토리지(Archival Storage), 인프라별 스토리지 정책(Infrastructure-Specific Storage Policy) 등의 차이를 표현할 수 있다.

동적 프로비저닝은 각각의 애플리케이션 인스턴스(Application Instance)에 대해 스토리지 자원을 개별적으로 구성하는 대신 표준화된 정책(Standardized Policy)에 따라 생성할 수 있기 때문에 대규모 로봇 배포를 단순화할 수 있다. 그러나 엣지 로봇(Edge Robot)의 스토리지는 특정 컴퓨터에 물리적으로 연결되어 있는 경우가 많기 때문에 주의가 필요하다. 동적으로 프로비저닝된 로컬 볼륨(Local Volume)은 쿠버네티스가 공통 추상화로 표현한다고 해서 공유 네트워크 스토리지(Shared Network Storage)와 동일한 이동성을 자동으로 확보하는 것은 아니다.

접근 모드(Access Mode)는 볼륨(Volume)이 워크로드에 의해 어떤 방식으로 마운트(Mount)되는지를 정의한다. 스토리지 기술에 따라 하나의 볼륨은 단일 노드에서 접근하거나 정의된 읽기 및 쓰기 조건에 따라 여러 노드에서 접근할 수 있다. 선택된 접근 모드는 애플리케이션의 동작 방식과 스토리지 백엔드의 기능 모두에 적합해야 한다. 쿠버네티스 선언만으로는 실제 스토리지 기술이 지원하지 않는 동시 접근 특성을 제공할 수 없다.

로컬 영속 스토리지(Local Persistent Storage)는 높은 처리량(Throughput), 예측 가능한 접근 성능, 클라우드 연결 없이 지속적인 운영이 가능하다는 점에서 로보틱스에 유용하다. 고속 로그(High-Rate Log), 로컬 지도(Local Map), 추론 모델(Inference Model), 임시 센서 기록(Temporary Sensor Recording), 임무 데이터를 로봇의 컴퓨팅 자원 가까이에 유지할 수 있다. 반면 로컬 스토리지는 물리적 하드웨어에 종속되므로 워크로드가 다른 노드로 재스케줄링되었을 때 동일한 데이터를 자동으로 사용할 수 없다는 단점이 있다.

네트워크 연결 스토리지(Network-Attached Storage)는 이와 다른 장단점을 가진다. 여러 컴퓨팅 노드가 중앙에서 관리되는 데이터에 접근할 수 있어 공유, 백업(Backup), 플릿 수준 데이터 수집(Fleet-Level Data Collection)을 단순화할 수 있다. 그러나 네트워크 지연시간, 대역폭, 가용성(Availability), 연결 단절을 신중하게 고려해야 한다. 모바일 로봇(Mobile Robot)은 Wi-Fi, LTE, 5G 또는 외부 네트워크 연결이 사용할 수 없는 상황에서도 지속되어야 하는 기능을 원격 스토리지에 지속적으로 의존하도록 설계해서는 안 된다.

따라서 로봇 데이터는 운영 중요도(Operational Importance)와 접근 패턴(Access Pattern)에 따라 분류해야 한다. 현재 사용 중인 지도, 필수 구성(Essential Configuration), 로컬에서 필요한 모델과 같이 자율 운용(Autonomous Operation)에 즉시 필요한 데이터는 일반적으로 로봇 내부에서 항상 사용할 수 있어야 한다. 대용량 과거 로그(Historical Log), 학습 데이터셋(Training Dataset), 플릿 분석 데이터(Fleet Analytics Data), 장기 아카이브(Long-Term Archive)는 적절한 네트워크 연결을 사용할 수 있을 때 중앙 인프라로 동기화하거나 업로드할 수 있다.

스테이트풀셋(StatefulSet)은 각각의 복제본(Replica)이 안정적인 식별자(Stable Identity)와 연결된 영속 스토리지를 필요로 할 때 PVC와 함께 사용되는 경우가 많다. 예를 들어 데이터베이스 파드는 다시 생성되더라도 해당 논리적 인스턴스(Logical Instance)에 연결된 볼륨을 계속 유지할 수 있다. 이러한 관계는 영속적인 로봇 인프라 서비스에 유용하지만 스테이트풀셋이 데이터베이스 복제(Database Replication), 백업, 무결성 검사(Integrity Checking), 복구 절차를 대신하는 것은 아니다. 쿠버네티스는 자원을 관리하지만 애플리케이션 수준의 데이터 정확성(Data Correctness)까지 보장하지는 않는다.

스토리지 생명주기 정책(Storage Lifecycle Policy) 역시 신중하게 설계해야 한다. 스토리지 구성에 따라 클레임이나 애플리케이션을 삭제했을 때 실제 데이터가 유지되거나 함께 제거될 수 있다. 임무 기록, 진단 증거(Diagnostic Evidence), 캘리브레이션 데이터, 운영 로그가 자동으로 삭제되는 것은 로보틱스 환경에서 바람직하지 않을 수 있다. 반대로 모든 임시 데이터셋(Temporary Dataset)을 무기한 보관하면 제한된 온보드 스토리지가 고갈되어 결국 로봇 운영에 영향을 줄 수 있다.

따라서 엣지 컴퓨터(Edge Computer)에서는 용량 관리(Capacity Management)가 실질적으로 중요한 문제가 된다. 카메라, 라이다(LiDAR), 텔레메트리 시스템, 디버깅 도구(Debugging Tool), AI 파이프라인(AI Pipeline)은 데이터를 빠르게 생성할 수 있지만 로봇의 SSD 용량은 제한되어 있다. 스토리지 할당량(Storage Quota), 로그 순환(Log Rotation), 보존 기간(Retention Period), 압축(Compression), 이벤트 기반 기록(Event-Based Recording), 중앙 스토리지로의 제어된 동기화(Controlled Synchronization)를 쿠버네티스 볼륨 관리와 함께 적용해야 한다. 영속 스토리지는 의도하지 않은 데이터 손실을 방지하지만 데이터가 무제한으로 누적되는 것을 자동으로 방지하지는 않는다.

성능 요구사항(Performance Requirement) 역시 스토리지 배치(Storage Placement)에 영향을 주어야 한다. AI 모델과 빈번하게 접근하는 지도 데이터는 빠른 로컬 SSD 또는 NVMe 스토리지의 이점을 얻을 수 있는 반면 장기 로그는 상대적으로 느린 스토리지를 사용할 수 있다. 데이터베이스는 예측 가능한 지연시간과 내구성(Durability)을 요구할 수 있고, 임시 처리 데이터는 처리량을 우선할 수 있다. 스토리지클래스와 스케줄링 정책(Scheduling Policy)은 이러한 차이를 표현하는 데 도움을 줄 수 있지만 실제 성능은 여전히 물리적 스토리지 장치와 네트워크 아키텍처에 의해 결정된다.

보안(Security)은 실행 중인 컨테이너뿐만 아니라 영속 데이터에도 적용되어야 한다. 로봇 스토리지에는 운영 로그, 지도, 자격증명(Credential), 구성 정보, 기록된 센서 데이터, 독점 모델(Proprietary Model)이 포함될 수 있다. 따라서 데이터 접근은 실제로 필요한 워크로드로 제한해야 하며 저장 정보의 민감도와 운영 중요도에 따라 적절한 암호화(Encryption), 자격증명 관리(Credential Management), 파일시스템 권한(Filesystem Permission), 백업 보호(Backup Protection), 안전한 삭제 정책(Secure Deletion Policy)을 고려해야 한다.

백업과 동기화(Synchronization)는 영속성과 별도로 설계해야 한다. PV를 사용하면 파드가 재시작되어도 데이터를 유지할 수 있지만 SSD 고장, 노드 손상, 파일시스템 손상(Filesystem Corruption), 로봇 자체의 손실에는 여전히 취약할 수 있다. 따라서 중요한 로봇 데이터는 다른 장치, 네트워크 연결 스토리지(NAS), 데이터센터 또는 클라우드 환경으로 복제하거나 주기적으로 전송해야 할 수 있다. 영속성(Persistence)은 특정 수준의 장애에서 연속성을 제공하는 반면 백업은 더 광범위한 장애로부터 데이터를 복구할 수 있도록 한다.

따라서 쿠버네티스 또는 K3s 로봇 아키텍처에서 PV, PVC, 스토리지클래스는 계층화된 스토리지 모델(Layered Storage Model)을 구성한다. PV는 사용 가능한 영속 스토리지 용량을 나타내고, PVC는 워크로드가 요구하는 스토리지 조건을 표현하며, 스토리지클래스는 정책에 따라 적절한 스토리지를 어떻게 프로비저닝할지를 정의한다. 이를 로컬 스토리지, 중앙 집중식 동기화(Centralized Synchronization), 생명주기 제어(Lifecycle Control), 보안, 백업 전략과 결합하면 로봇 데이터를 교체 가능한 컨테이너와 파드의 생명주기로부터 독립적으로 유지할 수 있다.

## 4.6. GPU Resource Management nvidia device plugin in K8s [w/Code]

![](images/image6.png){width="7.268055555555556in" height="7.268055555555556in"}

GPU 가속(GPU Acceleration)은 인지(Perception), 딥러닝 추론(Deep Learning Inference), 시각 기반 위치추정(Visual Localization), 매핑(Mapping), 시뮬레이션(Simulation), 파운데이션 모델(Foundation Model) 워크로드가 CPU만으로 제공할 수 있는 것보다 훨씬 높은 병렬 연산 성능을 요구할 수 있기 때문에 로봇 컴퓨팅에서 점점 더 중요해지고 있다. 쿠버네티스(Kubernetes)에서는 GPU를 스케줄링 가능한 자원(Schedulable Resource)으로 표현하여 워크로드가 가속기를 명시적으로 요청하고, 스케줄러(Scheduler)가 적절한 GPU 자원이 존재하는 노드에만 파드(Pod)를 배치할 수 있도록 해야 한다.

쿠버네티스는 핵심 스케줄링 시스템(Core Scheduling System) 내부에서 특정 제조사의 GPU 관리를 직접 구현하지 않는다. 대신 하드웨어 공급업체가 디바이스 플러그인(Device Plugin)을 통해 특수 장치를 노출할 수 있도록 한다. NVIDIA GPU의 경우 NVIDIA 디바이스 플러그인(NVIDIA Device Plugin)이 GPU 자원을 쿠버네티스와 통합하고 감지된 장치를 큐블릿(Kubelet)에 제공한다. 등록이 완료되면 GPU 용량은 노드의 할당 가능한 자원(Allocatable Resource)에 포함되며 컨테이너화된 애플리케이션이 이를 요청할 수 있다.

NVIDIA 디바이스 플러그인은 일반적으로 데몬셋(DaemonSet)으로 동작하여 GPU가 장착된 각 적격 노드(Eligible Node)에서 하나의 인스턴스가 실행되도록 한다. 각 플러그인 인스턴스는 로컬 머신에서 지원되는 NVIDIA 장치를 탐지하고 사용 가능 여부를 큐블릿에 전달한다. 이러한 노드 수준 배포 패턴(Node-Level Deployment Pattern)은 클러스터 토폴로지(Cluster Topology)를 자동으로 따라가므로 새롭게 추가된 GPU 노드가 각각의 애플리케이션 워크로드를 수동으로 구성하지 않고도 가속기 자원을 제공할 수 있다.

컨테이너는 쿠버네티스 자원 명세(Kubernetes Resource Specification)를 통해 NVIDIA GPU 자원을 요청하며 일반적으로 확장 자원 이름(Extended Resource Name)인 nvidia.com/gpu를 사용한다. 스케줄러는 노드를 선택할 때 이러한 요청을 고려하고 충분한 GPU 용량을 제공하지 않는 노드에는 해당 파드를 배치하지 않는다. 이를 통해 수동으로 특정 머신을 선택하는 대신 AI 워크로드 요구사항과 실제 가속기 가용성 사이에 선언적인 관계(Declarative Relationship)를 구성할 수 있다.

GPU 요청은 일반적인 CPU 및 메모리 자원 관리와 차이가 있다. CPU 자원은 일반적으로 부분적인 단위(Fractional Quantity)로 나누어 사용할 수 있지만 기존 쿠버네티스 GPU 할당은 노출된 GPU 장치를 개별적인 자원(Discrete Resource)으로 취급하는 경우가 일반적이다. 따라서 하나의 GPU를 요청한 워크로드에는 임의의 일부 GPU 연산 성능이 아니라 사용 가능한 GPU 자원이 할당된다. 보다 고급 GPU 공유(Advanced GPU Sharing)를 구현하려면 추가적인 GPU 전용 구성과 플랫폼 지원이 필요하다.

디바이스 플러그인 아래에서 동작하는 소프트웨어 스택(Software Stack) 역시 중요하다. GPU 노드에는 호환 가능한 NVIDIA 드라이버(NVIDIA Driver)와 GPU 장치를 컨테이너에 제공할 수 있는 컨테이너 런타임 환경(Container Runtime Environment)이 필요하다. CUDA 라이브러리와 애플리케이션 의존성(Application Dependency) 역시 호스트 드라이버 환경과 호환되어야 한다. 쿠버네티스 스케줄링이 GPU 워크로드를 올바른 노드에 할당하더라도 기반 드라이버, 런타임, CUDA 또는 프레임워크(Framework)의 버전이 호환되지 않으면 애플리케이션 실행에 실패할 수 있다.

로봇 클러스터(Robot Cluster)는 서로 다른 컴퓨팅 하드웨어를 포함하는 경우가 많다. 하나의 노드에는 NVIDIA RTX 계열 GPU가 탑재되고 다른 노드에는 통합 GPU를 사용하는 젯슨 플랫폼(Jetson Platform)이 사용되며, 추가 노드는 CPU 자원만 제공할 수 있다. 레이블(Label)과 스케줄링 제약조건(Scheduling Constraint)을 통해 이러한 차이를 표현하면 인지, 추론, 매핑 또는 시뮬레이션 워크로드를 적절한 하드웨어로 배치하면서 경량 서비스는 범용 노드(General-Purpose Node)에서 자유롭게 실행할 수 있다.

노드 레이블(Node Label)은 단순한 GPU 개수만으로 표현할 수 없는 특성을 나타낼 수 있다. 로봇 플랫폼은 GPU 아키텍처(GPU Architecture), 메모리 용량(Memory Capacity), 성능 등급(Performance Class), 전력 프로파일(Power Profile), 애플리케이션 역할(Application Role)을 구분할 수 있다. 이후 노드 셀렉터(Node Selector)와 노드 어피니티(Node Affinity)를 GPU 자원 요청과 결합할 수 있다. 이를 통해 단순히 GPU가 있다는 이유만으로 워크로드가 배치되는 것을 방지하고 실제 애플리케이션에 필요한 특정 가속기 기능을 만족하는 노드를 선택할 수 있다.

테인트(Taint)와 톨러레이션(Toleration)은 전용 가속기 노드(Dedicated Accelerator Node)를 제어하기 위한 또 다른 메커니즘을 제공한다. 고성능 GPU 컴퓨터에 테인트를 설정하면 일반적인 워크로드가 명시적으로 해당 제한을 허용하지 않는 한 그 노드에 배치되지 않도록 할 수 있다. GPU 집약적 애플리케이션(GPU-Intensive Application)은 필요한 가속기 자원과 특수 노드를 사용할 수 있는 권한을 함께 부여받을 수 있다. 이를 통해 실제로 GPU의 이점을 활용하는 워크로드를 위해 고가의 GPU 용량을 보존할 수 있다.

GPU 메모리(GPU Memory)는 GPU를 요청하는 것만으로 애플리케이션의 세부적인 메모리 사용 특성이 자동으로 표현되지 않기 때문에 자원 관리 측면에서 중요한 과제가 된다. 두 개의 추론 워크로드가 모두 GPU 접근을 요청하더라도 필요한 VRAM 용량은 크게 다를 수 있다. 모델 크기(Model Size), 배치 크기(Batch Size), 센서 해상도(Sensor Resolution), TensorRT 엔진, 중간 텐서(Intermediate Tensor), 동시 CUDA 워크로드 등이 모두 메모리 소비에 영향을 줄 수 있으므로 GPU 배포에는 쿠버네티스 자원 선언뿐만 아니라 애플리케이션 수준의 프로파일링(Application-Level Profiling)이 필요하다.

각 애플리케이션이 전체 GPU 성능을 사용하지 않는 경우 여러 워크로드가 하나의 GPU를 공유하면 활용률(Utilization)을 향상시킬 수 있다. NVIDIA가 지원하는 방식에는 시간 분할(Time-Slicing) 또는 호환 GPU에서 제공되는 하드웨어 파티셔닝(Hardware Partitioning) 기술 등이 포함될 수 있다. 이러한 메커니즘은 워크로드 밀도(Workload Density)를 높일 수 있지만 격리(Isolation), 메모리 사용량, 지연시간, 스케줄링 예측 가능성(Scheduling Predictability), 장애 동작(Failure Behavior)에 대한 추가적인 고려가 필요하다. 따라서 단순히 활용률을 최대화하기보다는 로봇 워크로드의 특성에 따라 선택해야 한다.

멀티 인스턴스 GPU(Multi-Instance GPU, MIG)는 호환되는 NVIDIA GPU에서 하드웨어 기반 파티셔닝(Hardware-Supported Partitioning)을 제공한다. 하나의 물리적 GPU를 정의된 연산 및 메모리 자원을 가진 서로 격리된 GPU 인스턴스(GPU Instance)로 분할하여 여러 워크로드가 개별 파티션을 사용할 수 있도록 한다. 필요한 구성을 지원하는 쿠버네티스 환경에서는 이러한 인스턴스를 스케줄링 가능한 자원으로 노출할 수 있으며, 단순한 시간 기반 공유(Time-Based Sharing)보다 강력한 자원 분리를 제공할 수 있다.

로보틱스는 가속기 워크로드가 지연시간에 민감한 처리 파이프라인(Latency-Sensitive Processing Pipeline)에 참여할 수 있기 때문에 GPU 스케줄링에 추가적인 제약조건을 부여한다. 카메라 프레임(Camera Frame)은 엄격한 시간 요구사항 아래에서 전처리(Preprocessing), 객체 탐지(Object Detection), 추적(Tracking), 위치추정(Localization), 경로 계획(Planning) 단계를 통과할 수 있다. 센서 데이터가 느린 네트워크 연결을 거쳐야 한다면 단순히 사용 가능한 임의의 GPU 노드로 워크로드를 이동하는 것은 바람직하지 않을 수 있다. 따라서 스케줄링은 GPU 가용성뿐만 아니라 데이터 지역성(Data Locality)과 로봇의 물리적 토폴로지(Physical Robot Topology)도 고려해야 한다.

GPU 장애 처리(GPU Failure Handling) 역시 단순히 컨테이너를 재시작하는 것 이상의 대응을 필요로 한다. GPU 메모리 고갈, 드라이버 오류(Driver Error), 장치 장애(Device Fault), 열적 상태(Thermal Condition), 애플리케이션 장애로 인해 프로세스가 비정상 상태가 될 수 있다. 쿠버네티스 상태 관리 메커니즘(Health Mechanism)은 워크로드를 재시작할 수 있고 모니터링 시스템은 애플리케이션과 노드 상태를 관찰할 수 있다. 그러나 장애가 반복되면 하드웨어 또는 드라이버 문제일 수 있으므로 노드 격리(Node Isolation), 워크로드 재배치(Workload Relocation), 유지보수(Maintenance), 로봇 수준의 폴백 동작(Robot-Level Fallback Behavior)이 필요할 수 있다.

GPU 활용률 모니터링(GPU Utilization Monitoring)은 로봇 클러스터를 효율적으로 설계하고 운영하기 위해 필수적이다. GPU 활용률, 메모리 사용량, 온도(Temperature), 전력 소비(Power Consumption), 워크로드 동작과 같은 메트릭(Metric)을 통해 가속기가 과부하 상태인지, 충분히 활용되지 않는지, 또는 열적 한계(Thermal Limit)에 접근하고 있는지를 파악할 수 있다. 특히 모바일 로봇에서는 전력과 냉각 용량(Cooling Capacity)이 제한되고 GPU 부하가 에너지 소비와 열 설계(Thermal Design)에 직접적인 영향을 주기 때문에 이러한 정보가 중요하다.

GPU 자원 관리는 소프트웨어 배포(Software Deployment) 및 모델 생명주기 관리(Model Lifecycle Management)와도 연계되어야 한다. 새로운 AI 모델은 이전 버전보다 더 많은 VRAM, 다른 CUDA 라이브러리 또는 다른 가속기 아키텍처를 요구할 수 있다. 따라서 CI/CD 및 MLOps 파이프라인은 롤아웃(Rollout) 이전에 하드웨어 호환성(Hardware Compatibility)을 검증해야 한다. 스케줄링 정책, 컨테이너 이미지(Container Image), 모델 아티팩트(Model Artifact), 노드 기능(Node Capability)은 서로 독립적으로 관리하기보다 함께 변화하고 관리되어야 한다.

따라서 쿠버네티스 또는 K3s 기반 로봇 아키텍처에서 NVIDIA 디바이스 플러그인은 물리적인 NVIDIA 가속기와 쿠버네티스 자원 스케줄링(Resource Scheduling)을 연결하는 가교 역할을 한다. 장치 탐색(Device Discovery)을 통해 GPU를 클러스터에 노출하고, 자원 요청(Resource Request)을 통해 워크로드를 사용 가능한 가속기와 연결하며, 레이블, 어피니티(Affinity), 테인트, 파티셔닝(Partitioning), 모니터링을 통해 배치와 활용을 세밀하게 조정한다. 이러한 메커니즘을 함께 사용하면 로봇 고유의 지연시간, 전력, 열 관리, 안전 요구사항을 유지하면서 확장 가능한 GPU 오케스트레이션(GPU Orchestration)을 구현할 수 있다.

## 4.7. Helm Charts for Robot Application Packaging [w/Code]

![](images/image7.png){width="7.268055555555556in" height="7.268055555555556in"}

헬름(Helm)은 복잡한 애플리케이션의 정의, 설치, 구성, 업그레이드를 단순화하는 쿠버네티스(Kubernetes)용 패키지 관리자(Package Manager)이다. 각각의 로봇 서비스를 위해 여러 개의 독립적인 쿠버네티스 매니페스트(Kubernetes Manifest)를 관리하는 대신, 헬름은 관련 자원을 차트(Chart)라고 하는 재사용 가능한 패키지로 묶는다. 로보틱스(Robotics)에서는 이를 통해 인지(Perception), 내비게이션(Navigation), 텔레메트리(Telemetry), 모니터링(Monitoring), AI 추론(AI Inference), 지원 서비스를 버전 관리가 가능한 배포 단위(Deployment Unit)로 패키징할 수 있다.

헬름 차트(Helm Chart)는 쿠버네티스 자원을 생성하는 데 필요한 템플릿(Template)과 구성 정보(Configuration Information)를 포함한다. 일반적인 자원에는 디플로이먼트(Deployment), 스테이트풀셋(StatefulSet), 데몬셋(DaemonSet), 서비스(Service), 컨피그맵(ConfigMap), 시크릿 참조(Secret Reference), 영속 볼륨 클레임(PersistentVolumeClaim), 인그레스(Ingress) 정의 등이 포함될 수 있다. 서로 다른 로봇이나 환경을 위해 거의 동일한 YAML 파일을 반복해서 작성하는 대신 템플릿으로 공통 구조를 정의하고 구성 가능한 값(Configurable Value)을 통해 배포 간 차이를 표현한다.

values.yaml 파일은 이러한 구성 모델(Configuration Model)의 중심적인 역할을 한다. 이 파일은 쿠버네티스 매니페스트를 생성할 때 헬름 템플릿이 참조할 수 있는 기본 파라미터(Default Parameter)를 제공한다. 로봇 애플리케이션은 컨테이너 이미지(Container Image) 버전, CPU와 메모리 요구량, GPU 요청(GPU Request), 네트워크 포트(Network Port), 스토리지 용량(Storage Capacity), 로깅 수준(Logging Level), 기능 스위치(Feature Switch), 노드 선택 규칙(Node-Selection Rule) 등의 설정을 외부에 노출할 수 있다. 이러한 파라미터는 기본 자원 템플릿을 수정하지 않고 변경할 수 있다.

템플릿과 값의 이러한 분리는 하나의 로봇 소프트웨어 스택(Robot Software Stack)이 여러 하드웨어 변형(Hardware Variant)을 지원해야 할 때 유용하다. NVIDIA RTX GPU를 탑재한 x86 로봇은 ARM 기반 젯슨 로봇(Jetson Robot)과 서로 다른 자원 제한(Resource Limit)과 스케줄링 규칙(Scheduling Rule)을 요구할 수 있다. 동일한 차트 구조를 두 시스템에 사용하면서 별도의 값 파일(Values File)을 통해 하드웨어별 컨테이너 이미지, GPU 요구사항, 스토리지 경로(Storage Path), 노드 레이블(Node Label), 애플리케이션 구성을 정의할 수 있다.

헬름 템플릿은 파라미터 치환(Parameter Substitution)과 템플릿 로직(Template Logic)을 사용하여 최종 쿠버네티스 자원 정의를 생성한다. 조건부 섹션(Conditional Section)은 구성에 따라 특정 자원을 포함하거나 제외할 수 있으며, 필요한 경우 반복문(Loop)을 이용하여 반복되는 구조를 생성할 수 있다. 따라서 하나의 차트에서 GPU 추론, 모니터링, 외부 스토리지(External Storage), 플릿 연결(Fleet Connectivity)과 같은 선택적 구성요소(Optional Component)를 지원하면서 각각의 조합마다 완전히 별도의 매니페스트 집합을 유지할 필요를 줄일 수 있다.

차트는 일반적으로 패키지 자체를 설명하는 메타데이터(Metadata)를 포함하며 차트 버전(Chart Version)과 실제 배포되는 애플리케이션 버전(Application Version)을 구분할 수 있다. 이러한 구분은 배포 로직(Deployment Logic)과 애플리케이션 소프트웨어가 서로 독립적으로 발전할 수 있기 때문에 중요하다. 새로운 컨테이너 이미지가 로봇 애플리케이션을 업데이트할 수 있는 반면 차트 개정(Chart Revision)은 애플리케이션 코드를 변경하지 않고 쿠버네티스 구성, 자원 정책(Resource Policy), 프로브(Probe), 스토리지 정의, 스케줄링 동작을 변경할 수 있다.

헬름 릴리스(Helm Release)는 설치된 차트의 인스턴스(Instance)를 나타낸다. 동일한 차트를 서로 다른 릴리스 이름(Release Name)과 구성 값으로 여러 번 설치할 수 있으므로 개발(Development), 시뮬레이션(Simulation), 시험(Testing), 운영(Production) 환경이 공통된 패키징 구조를 공유할 수 있다. 로봇 플릿(Robot Fleet)에서는 릴리스 구성을 이용하여 로봇 모델, 하드웨어 세대(Hardware Generation), 연구실 시스템, 현장 시험 장비(Field-Test Unit), 서로 다른 소프트웨어 정책을 사용하는 그룹을 구분할 수도 있다.

업그레이드 관리(Upgrade Management)는 헬름이 제공하는 가장 중요한 운영 기능 중 하나이다. 컨테이너 버전, 구성 파라미터, 자원 정의 또는 배포 정책(Deployment Policy)이 변경되면 헬름은 수정된 차트와 값을 기반으로 업데이트된 릴리스를 적용할 수 있다. 이는 개별 쿠버네티스 자원을 수동으로 수정하는 방식에 대한 구조화된 대안을 제공하며, 운영자가 어떤 패키지 구성이 의도된 애플리케이션 상태를 나타내는지 파악하는 데 도움을 준다.

헬름은 업데이트를 되돌려야 할 때 롤백(Rollback)을 지원할 수 있도록 릴리스 정보(Release Information)를 유지한다. 새로운 애플리케이션 구성으로 인해 허용할 수 없는 동작이 발생하면 운영자는 이전 릴리스 리비전(Release Revision)을 복원할 수 있다. 그러나 쿠버네티스 자원의 롤백이 외부 데이터베이스 변경, 모델 형식(Model Format), 펌웨어 업데이트(Firmware Update), 물리적인 로봇 상태까지 자동으로 이전 상태로 되돌리는 것은 아니다. 따라서 로봇 배포 절차는 헬름 릴리스 자체를 넘어 애플리케이션 호환성(Application Compatibility)을 고려해야 한다.

의존성(Dependency)을 이용하면 하나의 차트가 애플리케이션에 필요한 다른 차트를 포함할 수 있다. 로봇 관리 플랫폼(Robot Management Platform)은 별도로 유지관리되는 모니터링, 데이터베이스, 메시지 처리(Message Processing), 관측 가능성(Observability) 구성요소에 의존할 수 있다. 의존성 관리를 통해 모듈성(Modularity)을 유지하면서 이러한 구성요소를 더 큰 애플리케이션 패키지로 조합할 수 있다. 다만 버전을 통제하고 자원이 제한된 로봇 컴퓨터에 불필요한 인프라가 포함되지 않도록 주의해야 한다.

헬름 차트는 하나의 애플리케이션 패키지 내부에서 서로 다른 쿠버네티스 워크로드 패턴(Workload Pattern)을 표현할 수 있다. 무상태 플릿 통신 서비스(Stateless Fleet Communication Service)는 디플로이먼트로 생성하고, 노드 수준 모니터링(Node-Level Monitoring)은 데몬셋으로 구성하며, 영속 데이터베이스(Persistent Database)는 PVC를 사용하는 스테이트풀셋으로 구성할 수 있다. 서비스는 안정적인 네트워킹을 제공하고, GPU 요청과 스케줄링 제약조건은 AI 워크로드를 NVIDIA 가속기 자원을 제공하는 노드로 배치할 수 있다.

이러한 특성은 헬름을 다중 컴퓨터 로봇 아키텍처(Multi-Computer Robot Architecture)에 특히 유용하게 만든다. 하나의 차트에서 어떤 워크로드를 GPU 노드, ARM 엣지 컴퓨터(ARM Edge Computer), 센서 연결 머신(Sensor-Connected Machine), 범용 노드(General-Purpose Node)에 배치할 것인지 정의할 수 있다. 노드 셀렉터(Node Selector), 어피니티(Affinity), 테인트(Taint)와 톨러레이션(Toleration), 자원 요청(Resource Request), 구성 가능한 레이블을 템플릿과 값으로 표현하여 배포 정책을 로봇 애플리케이션 구성과 함께 버전 관리할 수 있다.

그러나 구성 정보(Configuration)는 신중하게 구분해야 한다. 값 파일은 배포 파라미터에 적합하지만 민감한 자격증명(Sensitive Credential)을 단순한 평문(Plain Text)으로 소스 저장소(Source Repository)에 저장해서는 안 된다. 필요한 경우 쿠버네티스 시크릿(Kubernetes Secret)이나 외부 시크릿 관리 메커니즘(External Secret-Management Mechanism)을 연동해야 한다. 마찬가지로 로봇별 캘리브레이션 데이터(Robot-Specific Calibration Data)나 고유 하드웨어 식별자(Unique Hardware Identifier)는 일반적인 차트 내부에 영구적으로 포함하기보다 별도의 생명주기 관리가 필요할 수 있다.

헬름은 CI/CD 및 깃옵스(GitOps) 워크플로(Workflow)와 자연스럽게 결합된다. 파이프라인(Pipeline)은 운영 환경으로 승격(Promotion)하기 전에 차트 문법을 검증하고, 템플릿을 렌더링(Rendering)하며, 생성된 매니페스트를 검사하고, 컨테이너 버전을 확인하며, 배포 동작을 시험할 수 있다. 승인된 차트 버전과 값은 로봇 소프트웨어 스택을 재현할 수 있는 설명이 될 수 있다. 이를 통해 소스 변경, 컨테이너 아티팩트(Container Artifact), 배포 구성, 실제 로봇 클러스터에 설치된 소프트웨어 사이의 추적성(Traceability)을 확보할 수 있다.

문법적으로 올바른 차트라도 운영상 잘못된 로봇 배포를 생성할 수 있기 때문에 테스트(Testing)는 특히 중요하다. 서로 다른 하드웨어 프로파일(Hardware Profile)을 대표하는 값으로 템플릿을 검증하고 생성된 자원의 스케줄링, 네트워킹, 스토리지, 권한(Permission), 자원 할당(Resource Allocation)을 점검해야 한다. 통합 테스트(Integration Testing)를 통해 서비스가 서로를 정상적으로 탐색하고 하드웨어 종속 워크로드가 올바른 장치에 접근하는지도 추가로 검증할 수 있다.

로봇 소프트웨어 업데이트는 물리적 운영 상태(Physical Operating State)도 고려해야 한다. 헬름은 업그레이드를 실행할 수 있지만 로봇이 이동 중인지, 충전 중인지, 화물을 운반하고 있는지, 또는 안전에 민감한 임무(Safety-Sensitive Mission)를 수행하고 있는지를 본질적으로 인식하지는 않는다. 따라서 플릿 관리 시스템(Fleet-Management System) 또는 배포 시스템이 언제 업그레이드를 허용할 것인지 결정해야 한다. 헬름은 패키징 및 릴리스 메커니즘을 제공하고, 로봇 수준 운영 로직(Robot-Level Operational Logic)은 안전한 배포 시점을 제어한다.

대규모 플릿에서 재사용 가능한 차트(Reusable Chart)는 구성의 중복을 줄이고 개별 로봇 사이에서 발생하는 구성 드리프트(Configuration Drift)를 방지하는 데 도움을 준다. 공통 기본값(Common Default)을 통해 표준 소프트웨어 스택을 정의하고, 통제된 값 파일을 통해 하드웨어 모델, 환경 또는 배포 그룹 사이의 차이를 표현할 수 있다. 이를 버전 관리, 자동화된 검증(Automated Validation), 단계적 롤아웃(Staged Rollout), 관측 가능성과 결합하면 개발 클러스터에서 운영 엣지 로봇(Production Edge Robot)까지 반복 가능한 소프트웨어 전달(Software Delivery)을 지원할 수 있다.

따라서 헬름은 쿠버네티스 또는 K3s 자원 위에서 동작하는 패키징 및 구성 계층(Packaging and Configuration Layer)의 역할을 한다. 차트는 관련 매니페스트를 구성하고, 템플릿은 재사용 가능한 정의를 환경별 자원으로 변환하며, 값은 배포 환경의 차이를 표현하고, 릴리스는 설치된 애플리케이션 구성을 추적한다. 로보틱스에서 이러한 메커니즘은 하드웨어 인식(Hardware Awareness), 재현성(Reproducibility), 통제된 업데이트(Controlled Update), 플릿 전체 생명주기 관리(Fleet-Wide Lifecycle Management)를 유지하면서 복잡한 소프트웨어 스택을 패키징하기 위한 확장 가능한 기반을 제공한다.

## 4.8. GitOps with ArgoCD for Robot Fleet Config Mgmt [w/Code]

![](images/image8.png){width="7.268055555555556in" height="7.268055555555556in"}

깃옵스(GitOps)는 깃 저장소(Git Repository)를 소프트웨어 인프라와 애플리케이션 구성의 목표 상태(Desired State)를 관리하는 권위 있는 기준(Source of Truth)으로 사용하는 운영 모델(Operational Model)이다. 운영자가 개별 로봇의 쿠버네티스(Kubernetes) 자원을 직접 변경하는 대신 구성 변경사항을 깃(Git)에 커밋하고 실행 중인 클러스터와 자동으로 조정(Reconciliation)한다. 로봇 플릿(Robot Fleet)에서는 이를 통해 다수의 분산 시스템에 대한 소프트웨어 구성을 통제 가능하고 추적 가능한 방식으로 관리할 수 있다.

아르고 CD(Argo CD)는 쿠버네티스를 위해 설계된 선언적 지속적 전달(Declarative Continuous Delivery) 도구이며 깃옵스 워크플로(GitOps Workflow)를 구현하는 데 널리 사용된다. 아르고 CD는 깃에 저장된 애플리케이션의 목표 상태와 쿠버네티스 또는 K3s 클러스터에서 실행 중인 실제 상태(Actual State)를 지속적으로 비교한다. 차이가 감지되면 구성 드리프트(Configuration Drift)를 보고하거나 클러스터를 동기화(Synchronize)하여 배포된 자원을 저장소에 정의된 승인된 구성으로 복원할 수 있다.

이 모델은 깃의 역할을 단순한 소스 코드 저장소(Source-Code Repository)에서 구성 제어 계층(Configuration Control Plane)으로 확장한다. 쿠버네티스 매니페스트(Kubernetes Manifest), 헬름 차트(Helm Chart), 커스터마이즈(Kustomize) 정의, 환경별 구성(Environment-Specific Configuration)을 변경 이력과 함께 버전 관리할 수 있다. 따라서 하나의 커밋(Commit)은 단순한 문서화 이벤트가 아니라 로봇 소프트웨어, 자원 할당, 네트워킹, 스토리지, 모니터링 또는 배포 정책에 대한 의도된 운영 변경을 나타낼 수 있다.

로봇 플릿은 일반적으로 여러 수준의 구성을 필요로 한다. 일부 설정은 모든 로봇에 공통적으로 적용되지만 다른 설정은 로봇 모델, 하드웨어 세대(Hardware Generation), 운영 사이트(Operational Site), 개발 단계(Development Stage), 임무 역할(Mission Role)에 따라 달라질 수 있다. 깃 저장소는 이러한 계층을 구성하여 공유 설정을 중앙에서 유지하면서 오버레이(Overlay) 또는 값 파일(Values File)을 통해 전체 배포 정의를 중복하지 않고 그룹별 차이를 통제된 방식으로 표현할 수 있다.

아르고 CD는 애플리케이션(Application) 자원을 통해 관리 대상 배포(Managed Deployment)를 표현한다. 애플리케이션은 깃에 존재하는 목표 소스(Desired Source), 관련 경로 또는 패키지, 대상 쿠버네티스 클러스터와 네임스페이스(Namespace), 동기화 동작(Synchronization Behavior)을 지정한다. 따라서 플릿 관리 아키텍처(Fleet-Management Architecture)는 공통적인 깃 기반 관리 모델을 유지하면서 개발 클러스터, 시뮬레이션 환경, 현장 로봇 또는 지역별 로봇 그룹과 애플리케이션을 연결할 수 있다.

동기화(Synchronization)는 운영 요구사항에 따라 수동 또는 자동으로 수행할 수 있다. 수동 동기화(Manual Synchronization)는 승인된 깃 변경사항을 적용하기 전에 운영자가 검토할 수 있도록 하고, 자동 동기화(Automated Synchronization)는 선택된 환경을 지속적으로 조정할 수 있도록 한다. 로봇 시스템에서는 연구실 클러스터에는 빠른 자동화를 허용하는 반면 현장 로봇에는 소프트웨어 변경이 활성화되기 전에 통제된 유지보수 시간(Maintenance Window)이나 추가적인 안전 검증(Safety Check)을 요구할 수 있으므로 두 방식을 조합하는 것이 유용하다.

구성 드리프트(Configuration Drift)는 클러스터의 실제 상태가 깃에 저장된 목표 상태와 달라지는 경우 발생한다. 수동 수정, 긴급 문제 해결(Emergency Troubleshooting), 실패한 업데이트 또는 의도하지 않은 구성 변경으로 이러한 차이가 발생할 수 있다. 아르고 CD는 실행 중인 자원(Live Resource)을 저장소 정의와 비교하여 이러한 상태를 탐지한다. 운영자는 차이를 조사하거나 동기화를 사용해 선언된 구성을 복원함으로써 플릿 구성원 사이의 장기적인 구성 차이를 줄일 수 있다.

자가 복구(Self-Healing)는 특정한 비인가 또는 우발적인 변경을 아르고 CD가 자동으로 수정할 수 있도록 함으로써 조정 개념을 확장한다. 관리 대상 쿠버네티스 자원이 직접 수정되면 컨트롤러(Controller)는 깃에 정의된 상태로 이를 복원할 수 있다. 이러한 동작은 일관성을 높이지만 로보틱스에서는 일부 런타임 변경(Runtime Change)이 하드웨어 장애, 성능 저하 상태(Degraded Operation), 현장 유지보수에 대한 의도적인 대응일 수 있으므로 항상 자동으로 덮어쓰지 않도록 정책을 신중하게 설계해야 한다.

깃 이력(Git History)은 로봇 플릿 구성에 중요한 감사 추적(Audit Trail)을 제공한다. 변경사항을 커밋, 작성자, 리뷰(Review), 브랜치(Branch), 풀 리퀘스트(Pull Request)와 연결할 수 있으므로 배포 정의가 어떻게 변화했는지를 확인할 수 있다. 업데이트 이후 현장에서 문제가 발생하면 엔지니어는 수동으로 관리되는 배포 기록에만 의존하는 대신 리비전(Revision)을 비교하여 해당 이벤트와 관련된 구성 변경사항을 식별할 수 있다.

깃옵스 워크플로에서 롤백(Rollback)은 일반적으로 깃에서 목표 구성을 이전 상태로 복원하거나 되돌린 뒤 조정 과정을 통해 이전 상태를 적용하는 방식으로 수행된다. 이를 통해 정상적인 배포에 사용하는 동일한 기준 정보(Source of Truth) 메커니즘을 복구에도 일관되게 적용할 수 있다. 그러나 헬름 롤백(Helm Rollback)과 마찬가지로 쿠버네티스 구성을 되돌린다고 해서 데이터베이스 마이그레이션(Database Migration), 펌웨어 변경, 모델 형식(Model Format), 캘리브레이션 업데이트(Calibration Update), 물리적 상태 변화까지 자동으로 되돌아가는 것은 아니다.

아르고 CD는 헬름 기반 로봇 애플리케이션 패키징(Helm-Based Robot Application Packaging)과 자연스럽게 통합된다. 헬름은 재사용 가능한 애플리케이션 템플릿과 구성 가능한 값을 정의하고, 깃은 승인된 차트 참조(Chart Reference)와 환경별 구성을 저장한다. 아르고 CD는 이러한 정의를 관찰하고 렌더링된 자원(Rendered Resource)을 대상 클러스터에 배포한다. 이를 통해 애플리케이션 패키징과 플릿 구성 관리를 분리하면서도 두 영역을 하나의 통합된 선언적 전달 프로세스(Declarative Delivery Process)로 연결할 수 있다.

실제 플릿 저장소(Fleet Repository)는 기본 애플리케이션 정의(Base Application Definition)와 로봇 그룹별 오버레이를 분리할 수 있다. 개발 로봇은 실험적인 컨테이너 버전, 시험 기능 또는 추가적인 디버깅 서비스(Debugging Service)를 사용할 수 있고, 운영 로봇은 검증된 이미지와 더욱 엄격한 자원 정책을 사용할 수 있다. 사이트별 오버레이(Site-Specific Overlay)는 전체 플릿의 공통 애플리케이션 정의를 유지하면서 스토리지, 네트워킹, 엔드포인트(Endpoint), 하드웨어 관련 값을 정의할 수 있다.

점진적 배포(Progressive Deployment)는 시뮬레이션에서 정상적으로 동작하는 구성이 실제 물리 하드웨어에서는 다르게 동작할 수 있기 때문에 로봇 플릿에서 특히 중요하다. 변경사항을 먼저 개발 시스템에 적용하고, 이후 소규모 검증 그룹(Validation Group)에 배포한 다음 관찰 결과를 확인한 후 더 넓은 운영 그룹으로 확대할 수 있다. 깃 브랜치, 디렉터리, 버전 참조(Version Reference), 외부 롤아웃 메커니즘(External Rollout Mechanism)은 배포 단계 사이의 감사 가능한 경로를 유지하면서 이러한 단계적 승격(Staged Promotion)을 지원할 수 있다.

로봇의 연결성(Connectivity)은 일반적인 데이터센터에서 상대적으로 덜 두드러지는 문제를 발생시킨다. 모바일 로봇은 일시적으로 네트워크 연결을 잃거나 셀룰러 연결(Cellular Connection)을 통해 동작하거나 장시간 오프라인 상태에 있을 수 있다. 따라서 깃옵스 아키텍처는 지속적인 연결을 전제로 해서는 안 된다. 로컬 쿠버네티스 워크로드는 연결이 끊어진 상태에서도 안전하게 계속 동작해야 하며 관리 인프라와의 통신이 다시 가능해지면 동기화를 재개할 수 있어야 한다.

보안(Security)은 깃옵스 저장소가 전체 플릿에서 실행되는 소프트웨어를 간접적으로 제어할 수 있기 때문에 매우 중요하다. 저장소 권한(Repository Permission), 브랜치 보호(Branch Protection), 리뷰 정책(Review Policy), 배포 자격증명(Deployment Credential), 쿠버네티스 역할 기반 접근 제어(Role-Based Access Control, RBAC), 아르고 CD 접근 제어를 통해 구성을 수정하거나 적용할 수 있는 사용자를 제한해야 한다. 민감한 자격증명(Sensitive Credential)은 일반적인 깃 콘텐츠와 분리하고 평문으로 커밋하는 대신 적절한 시크릿 관리 메커니즘(Secret-Management Mechanism)을 통해 처리해야 한다.

관측 가능성(Observability)은 구성 동기화 성공과 로봇의 실제 운영 성공을 혼동하지 않도록 동기화 과정과 함께 구성되어야 한다. 아르고 CD는 쿠버네티스 자원이 목표 상태와 일치하는지를 나타낼 수 있지만 이것만으로 인지 정확도(Perception Accuracy), 내비게이션, 센서, 액추에이터(Actuator), 임무 동작(Mission Behavior)이 정상적으로 작동한다는 것을 보장하지 않는다. 따라서 플릿 텔레메트리(Fleet Telemetry)와 애플리케이션 모니터링(Application Monitoring)을 통해 각 배포가 물리적 및 기능적으로 어떤 결과를 발생시키는지 검증해야 한다.

깃옵스는 CI/CD 및 MLOps 통합도 강화한다. CI 파이프라인은 컨테이너를 빌드하고 테스트하며 매니페스트 또는 헬름 차트를 검증하고 승인된 아티팩트(Artifact)를 게시할 수 있다. 이후 깃 변경을 통해 검증된 이미지 또는 모델 버전을 참조하면 아르고 CD가 배포 조정(Deployment Reconciliation)을 수행한다. 이를 통해 아티팩트 생성(Artifact Creation)과 배포 승인(Deployment Authorization)을 분리하고 소스 코드와 모델에서 실제 로봇에서 실행되는 구성까지의 추적성을 확보할 수 있다.

따라서 로봇 플릿에서 아르고 CD를 활용한 깃옵스(GitOps with Argo CD)는 쿠버네티스 또는 K3s 위에서 확장 가능한 구성 관리 계층(Configuration-Management Layer)을 제공한다. 깃은 목표 상태를 정의하고, 아르고 CD는 이를 배포된 클러스터와 지속적으로 비교하며, 동기화는 승인된 변경사항이 어떻게 전파되는지를 제어한다. 헬름, CI/CD, 단계적 롤아웃, 보안 제어(Security Control), 텔레메트리, 로봇 인식 안전 정책(Robot-Aware Safety Policy)과 결합하면 개별 로봇을 수동으로 업데이트하지 않고도 재현 가능하고 감사 가능한 플릿 구성 관리를 구현할 수 있다.

## 4.9. Kubernetes RBAC and Security Policy for Robot Services [w/Code]

![](images/image9.png){width="7.268055555555556in" height="7.268055555555556in"}

쿠버네티스 역할 기반 접근 제어(Kubernetes Role-Based Access Control, RBAC)는 어떤 사용자, 서비스, 자동화 프로세스가 클러스터 자원에 대해 특정 작업을 수행할 수 있는지를 결정하는 구조화된 메커니즘을 제공한다. 로봇 시스템에서는 내비게이션(Navigation), 인지(Perception), 텔레메트리(Telemetry), 플릿 관리(Fleet Management), 모니터링(Monitoring), 유지보수(Maintenance) 서비스가 모두 동일한 권한을 자동으로 부여받아서는 안 되기 때문에 중요하다. 접근 권한은 최소 권한 원칙(Principle of Least Privilege)에 따라 부여해야 한다.

쿠버네티스 인증(Authentication)은 누가 요청을 수행하는지를 식별하고, 인가(Authorization)는 해당 신원이 요청한 작업을 수행할 권한이 있는지를 결정한다. RBAC는 사용자(User), 그룹(Group), 서비스 어카운트(ServiceAccount)에 연결된 규칙을 평가하여 인가 단계에 참여한다. 이러한 분리를 통해 로봇 플랫폼은 사람 운영자(Human Operator)와 소프트웨어 워크로드(Software Workload)를 구분하고 운영 책임에 따라 서로 다른 권한을 할당할 수 있다.

롤(Role)은 특정 네임스페이스(Namespace) 내에서의 권한을 정의한다. 규칙에는 API 그룹(API Group), 자원 유형(Resource Type), 그리고 get, list, watch, create, update, patch, delete와 같은 허용 동작(Allowed Verb)을 지정할 수 있다. 예를 들어 로봇 텔레메트리 서비스는 특정 구성 자원을 읽을 권한이 필요할 수 있지만 디플로이먼트(Deployment)나 시크릿(Secret)을 수정할 권한까지 필요하지는 않다. 권한을 제한하면 손상되거나 오작동하는 소프트웨어가 시스템에 미칠 수 있는 영향을 줄일 수 있다.

클러스터롤(ClusterRole)은 롤과 유사한 규칙 구조를 사용하지만 여러 네임스페이스에 적용되거나 클러스터 범위 자원(Cluster-Scoped Resource)에 대한 권한을 표현할 수 있다. 클러스터롤은 더 넓은 가시성이 정당하게 필요한 모니터링 에이전트(Monitoring Agent), 인프라 컨트롤러(Infrastructure Controller), 플릿 관리 구성요소 등에 유용하다. 이러한 권한은 시스템의 더 넓은 영역에 영향을 줄 수 있으므로 신중하게 설계하고 클러스터 전체 접근이 운영상 필요한 경우에만 부여해야 한다.

롤과 클러스터롤은 그 자체만으로 권한을 부여하지 않는다. 롤바인딩(RoleBinding)과 클러스터롤바인딩(ClusterRoleBinding)은 이러한 권한 정의를 사용자, 그룹 또는 서비스 어카운트와 같은 신원(Identity)에 연결한다. 롤바인딩은 일반적으로 특정 네임스페이스 내에서 권한을 부여하는 반면 클러스터롤바인딩은 클러스터 전체에 대한 접근 권한을 제공할 수 있다. 이러한 권한 정의와 할당의 분리는 인가 정책(Authorization Policy)을 재사용 가능하고 감사하기 쉽게 만든다.

서비스 어카운트는 파드(Pod)가 이를 사용하여 쿠버네티스 API에 인증하기 때문에 로봇 서비스에서 특히 중요하다. 모든 워크로드가 광범위한 권한을 가진 기본 신원(Default Identity)을 통해 동작하도록 하는 대신 내비게이션, 인지, 모니터링, 배포, 유지보수 구성요소에 각각 별도의 서비스 어카운트를 할당할 수 있다. 이후 각 계정에는 해당 서비스가 실제로 필요로 하는 API 권한만 부여할 수 있다.

네임스페이스 설계(Namespace Design)는 로봇 워크로드를 구성하기 위한 또 하나의 보안 경계(Security Boundary)를 제공한다. 개발(Development), 시뮬레이션(Simulation), 모니터링, 플릿 서비스, 운영 애플리케이션(Production Application)을 서로 다른 네임스페이스로 분리하고 각각 다른 RBAC 정책을 적용할 수 있다. 네임스페이스 자체만으로 완전한 보안 격리(Security Isolation)를 제공하지는 않지만 추가적인 보안 제어와 결합하면 권한, 할당량(Quota), 구성, 운영 책임을 관리하는 유용한 범위를 제공한다.

최소 권한 원칙은 로봇 소프트웨어 생명주기(Robot Software Lifecycle) 전체의 RBAC 설계를 이끌어야 한다. 권한은 서비스에 필요한 최소한의 작업에서 시작하고 검증된 요구사항이 존재할 때만 확장해야 한다. 광범위한 와일드카드 권한(Wildcard Permission)이나 불필요한 클러스터 관리자 권한(Cluster-Admin Access)은 초기 개발을 단순화할 수 있지만 자격증명이 노출되거나 애플리케이션 취약점을 통해 공격자가 쿠버네티스 API에 접근할 경우 위험을 크게 증가시킨다.

RBAC는 쿠버네티스 API 자원에 대한 접근을 보호하지만 로봇 서비스 사이의 모든 통신 형태를 제어하는 것은 아니다. 네트워크 정책(NetworkPolicy)은 어떤 파드 또는 네임스페이스가 네트워크를 통해 통신할 수 있는지를 제한하여 RBAC를 보완할 수 있다. 인지 서비스는 위치추정(Localization)과 경로 계획(Planning) 구성요소와 통신해야 할 수 있지만 관련 없는 워크로드의 연결을 허용할 이유는 없을 수 있다. 네트워크 분할(Network Segmentation)은 손상된 서비스에서 중요 기능으로 연결되는 불필요한 경로를 줄인다.

파드 수준 보안(Pod-Level Security)은 또 다른 중요한 계층이다. 하드웨어 접근에 실제로 필요한 경우가 아니라면 컨테이너의 특권 실행(Privileged Execution)을 피해야 하며 불필요한 리눅스 기능(Linux Capability)은 제거해야 한다. 프로세스를 비루트 사용자(Non-Root User)로 실행하고, 권한 상승(Privilege Escalation)을 제한하며, 가능한 경우 읽기 전용 파일시스템(Read-Only Filesystem)을 사용하고, 호스트패스 볼륨(hostPath Volume)이나 호스트 네임스페이스(Host Namespace)를 신중하게 제어하면 손상된 컨테이너가 기반 로봇 컴퓨터에 영향을 미칠 가능성을 줄일 수 있다.

로봇 워크로드는 카메라(Camera), GPU, 라이다(LiDAR) 인터페이스, 직렬 장치(Serial Device), CAN 어댑터, 특수 가속기(Specialized Accelerator)와 같은 하드웨어에 직접 접근해야 하는 경우가 많다. 광범위한 호스트 권한을 사용하여 이러한 접근을 허용하면 일반적인 컨테이너 격리(Container Isolation)가 약화될 수 있다. 따라서 무제한 호스트 접근을 부여하기보다는 디바이스 플러그인(Device Plugin), 통제된 마운트(Controlled Mount), 런타임 구성(Runtime Configuration), 전용 노드(Dedicated Node) 등의 메커니즘을 사용하여 실제로 필요한 워크로드에만 장치 접근을 제한해야 한다.

로봇 시스템에는 API 자격증명(API Credential), 인증서(Certificate), 레지스트리 인증 정보(Registry Authentication Data), VPN 정보, 서비스 토큰(Service Token) 등이 포함될 수 있으므로 시크릿(Secret)은 특히 신중하게 관리해야 한다. RBAC를 사용하여 어떤 서비스 어카운트가 쿠버네티스 시크릿을 읽을 수 있는지 제한해야 하며, 민감한 정보를 컨테이너 이미지나 일반적인 구성 저장소에 직접 포함해서는 안 된다. 더욱 강력한 생명주기 제어(Lifecycle Control)가 필요한 경우 외부 시크릿 관리 시스템(External Secret-Management System)을 통합할 수 있다.

보안 정책(Security Policy)은 소프트웨어 공급망(Software Supply Chain)도 보호해야 한다. 컨테이너 이미지는 CI/CD 파이프라인에서 빌드하고 알려진 취약점(Known Vulnerability)을 검사하며 통제된 레지스트리(Controlled Registry)에 저장하고 검증된 버전 또는 변경 불가능한 다이제스트(Immutable Digest)를 사용하여 참조할 수 있다. 어드미션 정책(Admission Policy)은 워크로드가 클러스터에 배포되기 전에 배포 요구사항을 강제할 수 있다. 이러한 제어를 깃옵스(GitOps)와 결합하면 승인된 구성 변경이 임의의 컨테이너가 아니라 검증된 소프트웨어를 배포하도록 지원할 수 있다.

구성 및 인가 변경은 동시에 많은 로봇에 영향을 미칠 수 있기 때문에 플릿 보안(Fleet Security)에서는 감사 가능성(Auditability)이 필수적이다. 쿠버네티스 감사 정보(Kubernetes Audit Information), 깃 이력(Git History), 아르고 CD(Argo CD) 배포 기록, CI/CD 로그, 보안 모니터링(Security Monitoring)은 누가 정책을 변경했는지, 어떤 소프트웨어가 배포되었는지, 언제 자원이 수정되었는지에 대한 상호 보완적인 증거를 제공할 수 있다. 중앙 집중식 분석(Centralized Analysis)을 통해 분산된 로봇 시스템 전체에서 비정상적인 접근 패턴이나 승인되지 않은 변경을 식별할 수 있다.

보안 정책은 클라우드 인프라(Cloud Infrastructure)와 물리적 로봇(Physical Robot)의 차이도 고려해야 한다. 손상된 클라우드 서비스는 데이터나 컴퓨팅 자원을 노출할 수 있지만 손상된 로봇 서비스는 잠재적으로 센서, 이동(Motion), 액추에이터(Actuator), 사람과의 상호작용에 영향을 미칠 수 있다. 따라서 쿠버네티스 보안 제어는 로봇 수준 안전 메커니즘(Robot-Level Safety Mechanism)과 결합되어야 하며 소프트웨어 인가만으로 독립적인 이동 제한(Motion Limit), 비상 정지(Emergency Stop), 안전 컨트롤러(Safety Controller)를 우회할 수 없도록 해야 한다.

플릿 환경에서는 보안 정책의 통제된 차이(Controlled Variation)가 필요하다. 개발 로봇에는 운영 시스템에는 절대로 존재해서는 안 되는 디버깅 권한(Debugging Permission)이 필요할 수 있으며 유지보수 담당자는 진단을 위해 일시적으로 상승된 권한(Elevated Access)이 필요할 수 있다. 헬름 값(Helm Values), 깃옵스 저장소, 네임스페이스, 정책 정의를 통해 이러한 차이를 표현할 수 있지만 예외 사항은 명시적이고 검토 가능해야 하며 가능한 경우 시간 제한(Time-Limited)을 적용하고 운영상의 필요성과 연결하여 추적할 수 있어야 한다.

RBAC와 보안 정책은 배포 이후에만 검증하는 것이 아니라 CI/CD 과정의 일부로 테스트해야 한다. 자동화된 검사(Automated Check)를 통해 지나치게 광범위한 권한, 특권 컨테이너(Privileged Container), 안전하지 않은 호스트 마운트(Host Mount), 누락된 자원 제한 또는 정책 위반을 현장 로봇에 변경사항이 전달되기 전에 탐지할 수 있다. 통합 테스트(Integration Testing)를 통해 정상적인 서비스가 필요한 접근 권한을 유지하면서 승인되지 않은 작업은 거부되는지 확인하여 보안 강화(Security Hardening)가 예상치 못하게 로봇 기능을 중단시키는 것을 방지할 수 있다.

따라서 로봇 서비스를 위한 쿠버네티스 보안(Kubernetes Security)은 하나의 RBAC 구성만으로 이루어지는 것이 아니라 계층화된 아키텍처(Layered Architecture)로 구성된다. 인증은 신원을 확립하고, RBAC는 API 인가를 제어하며, 서비스 어카운트는 워크로드 권한을 분리하고, 네트워크 정책은 통신을 제한하며, 파드 보안(Pod Security)은 컨테이너 수준의 노출을 줄인다. 이를 시크릿 관리, 공급망 제어(Supply-Chain Control), 감사(Auditing), 깃옵스, 독립적인 로봇 안전 메커니즘과 결합하면 분산된 로봇 플릿을 위한 확장 가능한 보안 체계를 구축할 수 있다.

## 4.10. Multi Cluster Federation for Geo Distributed Robot Fleets

![](images/image10.png){width="7.268055555555556in" height="7.268055555555556in"}

멀티 클러스터 페더레이션(Multi-Cluster Federation)은 여러 개의 독립적인 클러스터에 걸쳐 워크로드, 정책, 서비스, 구성을 조정함으로써 쿠버네티스(Kubernetes) 관리를 단일 클러스터의 범위를 넘어 확장한다. 지리적으로 분산된 로봇 플릿(Geo-Distributed Robot Fleet)에서는 각각의 공장, 물류창고, 캠퍼스, 도시 또는 운영 지역이 자체 쿠버네티스 또는 K3s 클러스터를 유지하면서 더 큰 플릿 아키텍처(Fleet Architecture)에 참여할 수 있다. 이를 통해 로컬 자율성(Local Autonomy)을 유지하면서 중앙 집중식 운영 거버넌스(Centralized Operational Governance)를 구현할 수 있다.

하나의 쿠버네티스 클러스터는 많은 노드를 관리할 수 있지만 지리적으로 분산된 로봇은 네트워크 지연시간(Network Latency), 간헐적인 연결(Intermittent Connectivity), 관리 경계(Administrative Boundary), 서로 다른 장애 도메인(Failure Domain)이라는 문제를 발생시킨다. 모든 로봇을 하나의 원격 클러스터 아래에 배치하면 광역 통신(Wide-Area Communication)에 대한 불필요한 의존성이 발생할 수 있다. 멀티 클러스터 설계는 대신 로봇 그룹이 로컬 컨트롤 플레인(Local Control Plane)을 통해 동작하도록 하면서 상위 시스템이 구성과 플릿 전체 서비스를 조정하도록 한다.

각 클러스터는 중앙 인프라와의 통신을 사용할 수 없는 상황에서도 필수적인 로컬 로봇 동작을 지원할 수 있어야 한다. 내비게이션(Navigation), 인지(Perception), 안전 관련 애플리케이션 로직(Safety-Related Application Logic), 센서 처리(Sensor Processing), 로컬 임무 수행(Local Mission Execution)은 원격 컨트롤 플레인에 대한 지속적인 접근에 의존해서는 안 된다. 따라서 페더레이션은 로컬 운영 연속성(Local Operational Continuity)을 전역 관리 및 조정(Global Management and Coordination)과 분리함으로써 엣지 자율성(Edge Autonomy)을 보완한다.

지리적으로 분산된 아키텍처(Geo-Distributed Architecture)는 물리적 사이트 또는 운영 지역을 기준으로 클러스터를 구성할 수 있다. 물류창고는 하나의 로컬 K3s 클러스터를 운영하고, 제조 공장은 다른 클러스터를 운영하며, 원격 실외 로봇은 별도의 클러스터를 사용할 수 있다. 지역 또는 클라우드 시스템(Regional or Cloud System)은 지연시간에 민감한 모든 로봇 상호작용을 지리적 네트워크 경계를 넘어 전송하지 않고도 플릿 분석(Fleet Analytics), 소프트웨어 배포, 모델 관리(Model Management), 모니터링, 구성 거버넌스(Configuration Governance)를 제공할 수 있다.

멀티 클러스터 관리(Multi-Cluster Management)가 반드시 독립적인 클러스터를 하나의 논리적인 쿠버네티스 컨트롤 플레인(Logical Kubernetes Control Plane)으로 통합하는 것을 의미하지는 않는다. 현대적인 아키텍처에서는 각각의 클러스터 컨트롤 플레인을 독립적으로 유지하면서 관리 플랫폼(Management Platform), 깃옵스(GitOps), 서비스 디스커버리(Service Discovery), 정책 배포(Policy Distribution), 전용 멀티 클러스터 기술을 통해 조정하는 경우가 많다. 이러한 격리는 장애의 영향을 제한하고 각 사이트가 로컬 로봇 환경에 적합한 인프라를 유지할 수 있도록 한다.

클러스터 등록(Cluster Registration)과 신원(Identity)은 플릿 전체 관리의 기본 요소이다. 중앙 관리 계층(Central Management Layer)은 어떤 클러스터가 존재하고, 어떤 로봇 그룹을 나타내며, 어떠한 하드웨어 기능을 보유하고 있고, 어떤 정책이 적용되는지를 파악해야 한다. 메타데이터(Metadata)를 통해 위치 분류(Location Class), 로봇 모델, GPU 기능, 환경, 소프트웨어 채널(Software Channel), 운영 역할(Operational Role)을 표현하면 개별 로봇을 수동으로 지정하지 않고도 관리 시스템이 구성을 대상별로 적용할 수 있다.

깃옵스는 이러한 아키텍처를 위한 실용적인 구성 모델(Configuration Model)을 제공한다. 중앙 깃 저장소(Git Repository)는 공통 플릿 구성과 함께 클러스터별 또는 지역별 오버레이(Regional Overlay)를 정의할 수 있다. 아르고 CD(Argo CD) 또는 관련 배포 컨트롤러(Deployment Controller)는 승인된 정의를 지정된 클러스터에 동기화할 수 있다. 공통 로봇 서비스는 표준화된 상태를 유지하면서 사이트별 네트워킹, 스토리지, 하드웨어 구성, 운영 정책을 통제된 차이(Controlled Variation)로 표현할 수 있다.

헬름(Helm)을 이용한 애플리케이션 패키징(Application Packaging)은 클러스터 사이의 중복을 더욱 줄인다. 공통 헬름 차트(Helm Chart)는 인지, 텔레메트리(Telemetry), 모니터링 또는 플릿 에이전트(Fleet Agent) 서비스를 정의하고, 별도의 값 파일(Values File)은 지역별 엔드포인트(Regional Endpoint), 하드웨어 프로파일(Hardware Profile), 스토리지 클래스(Storage Class), 자원 제한(Resource Limit), 기능 설정을 정의할 수 있다. 따라서 모든 지역에 동일한 런타임 파라미터를 강제하지 않으면서 동일한 애플리케이션 패키지를 이기종 클러스터(Heterogeneous Cluster)에 일관되게 배포할 수 있다.

클러스터 간 워크로드 배치(Workload Placement)는 하나의 클러스터 내부에서 파드(Pod)를 스케줄링하는 것과 다른 의사결정 계층을 필요로 한다. 쿠버네티스 스케줄러(Kubernetes Scheduler)는 자신의 클러스터 내부에서 노드를 선택하지만 플릿 수준 오케스트레이션(Fleet-Level Orchestration)은 어떤 클러스터가 워크로드나 구성을 받아야 하는지를 결정해야 한다. 이러한 결정에는 지리적 위치, 로봇 수, 하드웨어 가용성, 규제 경계(Regulatory Boundary), 네트워크 품질, 데이터 지역성(Data Locality), 운영 역할 또는 소프트웨어 릴리스 단계(Software Release Stage)가 고려될 수 있다.

애플리케이션이 여러 클러스터에 걸쳐 동작하면 서비스 통신(Service Communication) 역시 더욱 복잡해진다. 로봇 서비스는 일반적으로 지연시간에 민감한 통신을 로컬에 유지하고, 플릿 텔레메트리, 모델 배포(Model Distribution), 보고(Reporting), 관리 트래픽(Management Traffic)은 지역 경계를 넘어 전달할 수 있다. 멀티 클러스터 네트워킹(Multi-Cluster Networking)이나 서비스 디스커버리 메커니즘을 이용하여 선택된 서비스를 연결할 수 있지만 모든 클러스터의 모든 파드가 투명한 전역 통신(Transparent Global Communication)을 필요로 한다고 가정하기보다 필요한 연결만 의도적으로 구성해야 한다.

로봇 플릿에서는 센서가 대량의 이미지, 비디오, 라이다(LiDAR), 로그, AI 데이터를 생성할 수 있기 때문에 데이터 지역성이 특히 중요하다. 모든 원시 정보(Raw Information)를 중앙 클라우드로 지속적으로 전송하면 과도한 대역폭을 소비하고 지연시간을 증가시킬 수 있다. 로컬 클러스터는 로봇 가까이에서 데이터를 처리하고 필터링하며, 선택된 이벤트(Event), 메트릭(Metric), 압축 결과(Compressed Result), 학습 샘플(Training Sample), 보관 데이터셋(Archived Dataset)만 지역 또는 중앙 인프라로 전송할 수 있다.

장애 도메인 격리(Failure-Domain Isolation)는 멀티 클러스터 아키텍처의 주요 장점 중 하나이다. 한 사이트에서 발생한 컨트롤 플레인 장애(Control-Plane Failure), 네트워크 문제, 구성 오류 또는 서비스 과부하가 다른 지역에서 동작하는 로봇을 자동으로 중단시켜서는 안 된다. 독립적인 클러스터는 장애를 특정 영역에 제한할 수 있는 경계를 형성한다. 중앙 관리 시스템은 하나의 로컬 사고를 전체 플릿 장애로 확대하지 않으면서 영향을 받은 지역을 관찰하고 복구를 조정할 수 있다.

소프트웨어 롤아웃(Software Rollout) 역시 이러한 장애 도메인을 고려해야 한다. 새로운 로봇 애플리케이션이나 AI 모델은 먼저 개발 클러스터(Development Cluster)에 배포하고, 이후 선택된 사이트 또는 카나리 그룹(Canary Group)에 적용한 다음 추가적인 지역으로 확대할 수 있다. 비정상적인 동작이 발견되면 전체 플릿이 영향을 받기 전에 롤아웃을 중단할 수 있다. 깃옵스, 버전 관리된 헬름 패키지(Versioned Helm Package), 텔레메트리, 배포 정책은 통제된 지리적 승격(Controlled Geographic Promotion)을 위한 기반을 함께 제공한다.

여러 클러스터와 네트워크에 걸쳐 신뢰 관계(Trust)를 관리해야 하므로 보안(Security)은 더욱 복잡해진다. 각 클러스터에는 자체 인증(Authentication), 인가(Authorization), 서비스 어카운트(ServiceAccount), 시크릿(Secret), 인증서(Certificate), RBAC 정책이 필요하며 중앙 관리 시스템은 원격 환경에 대한 접근을 신중하게 통제해야 한다. 하나의 사이트가 침해되더라도 다른 클러스터에 대한 무제한 접근 권한이 자동으로 제공되어서는 안 되므로 신원 경계(Identity Boundary)와 최소 권한 페더레이션(Least-Privilege Federation)이 필수적이다.

관측 가능성(Observability)은 로컬 수준의 상세 정보와 플릿 전체 수준의 가시성(Fleet-Wide Visibility)을 모두 제공해야 한다. 각 클러스터는 애플리케이션 로그, 자원 메트릭(Resource Metric), GPU 상태, 로봇 텔레메트리, 쿠버네티스 이벤트(Kubernetes Event)를 로컬에서 수집하고 선택된 정보를 지역 또는 중앙 대시보드(Dashboard)로 집계할 수 있다. 운영자는 모든 운영 데이터를 중앙에서 실시간으로 처리하지 않고도 전체 플릿 상태를 확인하고 특정 클러스터 또는 로봇 수준까지 세부적으로 분석할 수 있다.

연결 단절 상태(Disconnected Operation)는 예외적인 장애가 아니라 정상적인 설계 조건으로 다루어야 한다. 원격 로봇 사이트는 인터넷 또는 광역망(Wide Area Network, WAN) 연결을 잃더라도 로컬 클러스터와 로봇 자체는 정상적으로 동작할 수 있다. 따라서 로컬 구성, 컨테이너 이미지(Container Image), 필수 서비스, 안전 로직(Safety Logic)은 연결이 끊어진 상태에서도 지속적인 운영을 지원해야 한다. 연결이 복구되면 텔레메트리, 구성 동기화(Configuration Synchronization), 소프트웨어 관리를 통제된 방식으로 재개할 수 있다.

멀티 클러스터 플릿의 확장성(Scalability)은 수동 관리가 아니라 자동화(Automation)에 의존한다. 클러스터 프로비저닝(Cluster Provisioning), 등록, 정책 할당, 애플리케이션 배포, 인증서 관리(Certificate Management), 모니터링, 생명주기 업데이트(Lifecycle Update)는 반복 가능한 프로세스를 사용해야 한다. 표준 클러스터 프로파일(Standard Cluster Profile)을 통해 공통 기준을 설정하고 통제된 오버레이를 사용하여 각 사이트별로 독립적인 구성을 관리하지 않으면서 지역별 하드웨어 또는 운영 차이를 수용할 수 있다.

따라서 지리적으로 분산된 로봇 플릿을 위한 멀티 클러스터 페더레이션은 로컬 자율성과 중앙 집중식 거버넌스를 결합한다. 독립적인 쿠버네티스 또는 K3s 클러스터는 지리적 장애 격리(Geographic Fault Isolation)와 저지연 로봇 운영(Low-Latency Robot Operation)을 제공하고, 깃옵스, 헬름, 보안 정책, 관측 가능성, 플릿 수준 오케스트레이션이 이들을 하나의 더 큰 시스템으로 조정한다. 이러한 아키텍처를 통해 안전한 로컬 운영을 지속적으로 사용 가능한 중앙 클러스터에 의존시키지 않으면서 여러 사이트와 지역으로 로봇 플릿을 확장할 수 있다.
