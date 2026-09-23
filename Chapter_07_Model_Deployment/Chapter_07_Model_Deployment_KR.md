**Volume 10 Robot DevOps and MLOps**

# 7. Model Deployment

## 7.1. Model Deployment Strategies Blue Green Shadow Canary

![](images/image1.png){width="7.268055555555556in" height="7.268055555555556in"}

머신러닝 모델(Machine Learning Model)을 로봇 시스템(Robotic System)에 배포하는 것은 학습된 모델 산출물(Model Artifact)을 엣지 컴퓨터(Edge Computer)에 단순히 복사하는 작업이 아니다. 모델 배포(Model Deployment)는 사람, 기계, 불확실한 환경과 상호작용하는 물리 시스템(Physical System)의 동작을 변화시킨다. 따라서 모델 배포에는 운영 위험(Operational Risk)을 제한하면서 새로운 모델이 실제 운영 환경에서 올바르게 동작한다는 것을 측정 가능한 증거로 확인할 수 있는 통제된 전환 전략(Controlled Transition Strategy)이 필요하다.

배포 전략(Deployment Strategy)은 기존 모델에서 새로운 모델로 트래픽(Traffic), 센서 데이터(Sensor Data), 추론 요청(Inference Request), 또는 로봇 집단(Robot Population)을 어떻게 전환할 것인지를 정의한다. 핵심 목적은 모델 릴리스(Model Release)와 즉각적인 전체 플릿 활성화(Fleet-wide Activation)를 분리하는 것이다. 블루-그린 배포(Blue-Green Deployment), 섀도 배포(Shadow Deployment), 카나리 배포(Canary Deployment)는 이러한 분리를 구현하는 서로 다른 방법을 제공하며, 엔지니어가 새로운 모델에 운영 책임을 부여하기 전에 단계적으로 검증할 수 있도록 한다.

블루-그린 배포(Blue-Green Deployment)는 두 개의 완전한 운영 환경(Production Environment)을 유지한다. 블루 환경(Blue Environment)은 일반적으로 현재 활성화되어 검증된 모델을 의미하며, 그린 환경(Green Environment)은 이를 대체할 후보 모델(Candidate Model)을 포함한다. 두 환경은 동일한 인터페이스(Interface), 전처리 로직(Preprocessing Logic), 런타임 의존성(Runtime Dependency), 하드웨어 가정(Hardware Assumption)을 사용해야 한다. 그린 환경이 검증을 통과하면 추론 트래픽(Inference Traffic)을 통제된 전환 메커니즘(Switching Mechanism)을 통해 블루에서 그린으로 전환할 수 있다.

블루-그린 배포의 주요 장점은 결정론적 롤백(Deterministic Rollback)이다. 기존 환경이 계속 사용 가능한 상태로 유지되므로, 활성화 이후 지연시간(Latency), 정확도(Accuracy), 자원 소비(Resource Consumption), 또는 동작 이상(Behavioral Abnormality)이 발생하면 운영자는 추론 트래픽을 다시 블루 모델로 전환할 수 있다. 로보틱스(Robotics)에서는 인지(Perception), 내비게이션(Navigation), 조작(Manipulation), 안전 지원 인공지능(Safety-supporting AI) 구성요소를 이전 소프트웨어 환경을 다시 구축하지 않고 신속하게 복구해야 하는 경우 특히 유용하다.

그러나 블루-그린 배포는 두 개의 모델 환경이 동시에 존재해야 할 수 있기 때문에 상당한 엣지 자원(Edge Resource)을 요구할 수 있다. 제한된 GPU, 가속기(Accelerator), 메모리 시스템(Memory Subsystem)을 탑재한 로봇은 두 모델을 동시에 메모리에 상주시킬 충분한 용량이 없을 수 있다. 플릿 아키텍처(Fleet Architecture)는 두 배포 패키지(Deployment Package)를 로컬에 저장하되 하나의 런타임(Runtime)만 활성화하거나, 플릿·서버·컨테이너 오케스트레이션(Container Orchestration) 수준에서 블루-그린 전환을 수행함으로써 이러한 한계를 해결할 수 있다.

섀도 배포(Shadow Deployment)는 근본적으로 다른 접근 방식을 사용한다. 운영 모델(Production Model)은 계속해서 로봇의 실제 동작을 담당하고, 후보 모델은 동일한 센서 입력(Sensor Input) 또는 추론 요청의 복사본을 전달받는다. 섀도 모델(Shadow Model)의 예측 결과는 기록되지만 액추에이터(Actuator)를 제어하거나 실제 운영 의사결정을 변경하는 데 사용되지 않는다. 이를 통해 검증되지 않은 모델 출력에 로봇을 직접 노출하지 않으면서 실제 운영 조건에서 새로운 모델을 평가할 수 있다.

예를 들어 로봇 인지 시스템(Robot Perception System)은 동일한 카메라 프레임(Camera Frame)을 현재 운영 중인 객체 탐지기(Object Detector)와 후보 객체 탐지기에 동시에 전달할 수 있다. 엔지니어는 실제 환경에서 탐지 결과, 신뢰도 분포(Confidence Distribution), 미탐지 객체(Missed Object), 추론 지연시간(Inference Latency), GPU 메모리 사용량, 모델 간 불일치 패턴(Disagreement Pattern)을 비교할 수 있다. 따라서 섀도 평가는 기존 모델의 제어 권한을 유지하면서 오프라인 데이터셋(Offline Dataset), 시뮬레이션 환경(Simulation Environment), 실험실 검증(Laboratory Validation)에서 발견되지 않았던 실패 모드(Failure Mode)를 찾아낼 수 있다.

섀도 배포는 병렬 추론(Parallel Inference)이 연산 자원, 메모리, 네트워크 대역폭(Bandwidth), 저장공간, 에너지를 소비하기 때문에 신중하게 설계해야 한다. 엣지 GPU에서 두 개의 대규모 인지 모델을 동시에 실행하면 프레임률(Frame Rate)이 감소하거나 다른 실시간 작업(Real-time Workload)을 방해할 수 있다. 따라서 실제 시스템에서는 대표성을 갖는 충분한 검증 데이터를 수집하면서도 선택된 프레임, 특정 로봇, 특정 운영 구역, 또는 사전에 정의된 시간 구간에 대해서만 섀도 모델을 실행할 수 있다.

카나리 배포(Canary Deployment)는 운영 환경의 일부에 후보 모델을 점진적으로 노출하는 방법이다. 전체 플릿(Fleet)의 모델을 한 번에 교체하는 대신 제한된 로봇 그룹이나 신중하게 선정된 임무(Mission)부터 배포를 시작한다. 카나리 집단(Canary Population)의 텔레메트리(Telemetry)는 기준 집단(Baseline Population)과 비교되며, 사전에 정의된 검증 게이트(Validation Gate)가 지속적으로 충족되는 경우에만 배포 범위를 확대한다. 이를 통해 플릿 배포를 점진적이고 증거 기반(Evidence-driven)의 과정으로 전환할 수 있다.

로보틱스의 카나리 배포는 단순한 비율이 아니라 운영 위험(Operational Risk)을 기준으로 정의해야 한다. 매우 어려운 환경에서 운용되는 플릿의 10%는 통제된 시설에서 운용되는 절반의 플릿보다 더 큰 위험을 나타낼 수 있다. 따라서 카나리 대상 선정에서는 후보 모델을 적용할 시스템을 결정하기 전에 로봇 하드웨어 버전(Hardware Version), 지리적 사이트(Geographic Site), 환경 조건(Environmental Condition), 임무 유형(Mission Type), 네트워크 품질(Network Quality), 안전 제약(Safety Constraint), 운영자 가용성(Operator Availability) 등을 고려할 수 있다.

점진적 롤아웃(Progressive Rollout)은 일반적으로 자동화된 게이트(Automated Gate)와 명시적인 롤백 정책(Rollback Policy)을 결합한다. 평가 지표에는 추론 지연시간, GPU 사용률(Utilization), 메모리 압력(Memory Pressure), 예측 신뢰도(Prediction Confidence), 모델 불일치율(Disagreement Rate), 인지 실패(Perception Failure), 내비게이션 개입(Navigation Intervention), 임무 완료율(Mission Completion), 안전 이벤트(Safety Event), 시스템 수준 오류 지표(System-level Fault Indicator) 등이 포함될 수 있다. 모니터링 값이 정의된 임계값을 초과하면 롤아웃을 자동으로 중단하고 영향을 받은 로봇을 이전에 검증된 모델로 복귀시켜야 한다.

블루-그린 배포, 섀도 배포, 카나리 배포는 서로 배타적인 방법이 아니라 상호 보완적인 전략이다. 후보 모델은 먼저 섀도 모드(Shadow Mode)에서 실제 운영 데이터를 기반으로 증거를 수집한 후, 실제 로봇 동작에 예측 결과가 반영되는 카나리 집단으로 이동하고, 마지막으로 보다 넓은 플릿 영역에 대해 블루-그린 활성화(Blue-Green Activation)를 수행할 수 있다. 이러한 단계적 절차는 모델이 실제 운영에 노출되는 수준을 점진적으로 높이면서 모델 수명주기(Model Lifecycle) 전체에 걸쳐 롤백 지점을 유지한다.

성숙한 로봇 MLOps 플랫폼(Robot MLOps Platform)은 모델 배포를 모델 식별 정보(Model Identity), 검증 증거(Validation Evidence), 플릿 구성(Fleet Configuration), 관측 가능성(Observability), 롤백 권한(Rollback Authority)에 의해 관리되는 상태 머신(State Machine)으로 다루어야 한다. 모든 배포 모델은 모델 레지스트리(Model Registry), 데이터셋과 학습 계보(Dataset and Training Lineage), 런타임 패키지(Runtime Package), 하드웨어 호환성(Hardware Compatibility), 검증 결과와 추적 가능해야 한다. 또한 디버깅(Debugging)이나 사고 분석(Incident Analysis) 과정에서 운영 동작을 재구성할 수 있도록 모든 배포 이벤트(Deployment Event)를 기록해야 한다.

궁극적인 목표는 로봇 지능(Robot Intelligence)을 통제된 방식으로 진화시키는 것이다. 새로운 모델이 오프라인 벤치마크(Offline Benchmark)에서 더 높은 점수를 기록했다는 이유만으로 신뢰해서는 안 된다. 신뢰는 점점 더 현실적인 배포 단계와 관측 가능한 운영 증거(Observable Production Evidence)를 통해 축적되어야 한다. 섀도 평가, 제한적인 카나리 노출, 블루-그린 전환, 지속적인 모니터링(Continuous Monitoring), 신속한 롤백을 결합함으로써 로봇 플릿은 모든 모델 업데이트를 통제되지 않은 운영 실험으로 만들지 않으면서 향상된 AI 모델을 안전하고 체계적으로 도입할 수 있다.

## 7.2. Model Serving Frameworks Triton TorchServe ONNX RT [w/Code]

![](images/image2.png){width="7.268055555555556in" height="7.268055555555556in"}

모델 서빙 프레임워크(Model Serving Framework)는 학습된 머신러닝 모델(Machine Learning Model)을 실제 운영 가능한 추론 서비스(Inference Service)로 변환하는 런타임 계층(Runtime Layer)을 제공한다. 로보틱스(Robotics)에서는 이 계층이 인지(Perception), 계획(Planning), 조작(Manipulation), 모니터링(Monitoring) 및 기타 AI 구성요소를 최적화된 실행 백엔드(Execution Backend)와 연결한다. 서빙 프레임워크는 예측 가능한 지연시간(Latency)을 유지하면서 모델 로딩(Model Loading), 추론 요청(Inference Request), 하드웨어 가속(Hardware Acceleration), 동시성(Concurrency), 메모리 사용량, 버전 관리(Versioning), 운영 인터페이스(Operational Interface)를 관리해야 한다.

로봇 AI 배포(Robot AI Deployment)는 추론이 실시간 물리 제어 루프(Real-time Physical Control Loop)에 포함되는 경우가 많다는 점에서 일반적인 클라우드 추론(Cloud Inference)과 다르다. 카메라 프레임(Camera Frame), 라이다 데이터(LiDAR Data), 오디오(Audio), 멀티모달 관측(Multimodal Observation)은 지속적으로 입력되며, 지연된 예측은 수학적으로 정확하더라도 운영 측면에서는 의미가 없어질 수 있다. 따라서 모델 서빙 아키텍처(Model Serving Architecture)는 처리량(Throughput)뿐만 아니라 지연시간 분포(Latency Distribution), 스케줄링 동작(Scheduling Behavior), 데이터 복사 오버헤드(Data-copy Overhead), GPU 자원 경합(GPU Contention), ROS2 노드 및 다른 로봇 프로세스와의 상호작용까지 고려해야 한다.

NVIDIA 트리톤 추론 서버(NVIDIA Triton Inference Server)는 여러 프레임워크(Framework)와 실행 백엔드(Execution Backend)의 모델을 호스팅할 수 있도록 설계된 고성능 서빙 플랫폼(High-performance Serving Platform)이다. 하나의 트리톤(Triton) 배포 환경에서 표준화된 추론 인터페이스(Standardized Inference Interface)를 통해 여러 모델을 제공하면서 GPU와 CPU 실행 자원을 관리할 수 있다. 이러한 아키텍처는 하나의 로봇 또는 엣지 서버(Edge Server)가 객체 탐지(Object Detection), 의미론적 분할(Semantic Segmentation), 자세 추정(Pose Estimation), 음성 처리(Speech Processing), 멀티모달 추론(Multimodal Inference) 등 여러 AI 작업을 공유 컴퓨팅 하드웨어에서 수행할 때 특히 유용하다.

트리톤(Triton)은 개별 모델을 구성 정보(Configuration Information) 및 버전별 모델 산출물(Versioned Model Artifact)과 함께 구성하는 모델 저장소(Model Repository)를 지원한다. 이러한 분리를 통해 서빙 인프라(Serving Infrastructure)는 애플리케이션 코드(Application Code)와 독립적으로 모델 수명주기(Model Lifecycle)를 관리할 수 있다. 로보틱스 소프트웨어는 안정적인 인터페이스를 통해 추론을 요청하면서 내부 모델 구현을 지속적으로 변경할 수 있으므로, 각각의 소비 애플리케이션(Consumer Application)에 모델별 로딩 로직을 포함하지 않고도 모델 교체, 롤백(Rollback), 테스트 및 플릿 수준 구성(Fleet-level Configuration)을 수행할 수 있다.

트리톤(Triton)의 중요한 기능 중 하나는 동적 배칭(Dynamic Batching)으로, 워크로드(Workload) 특성이 허용하는 경우 서로 호환되는 추론 요청을 더 큰 실행 배치(Execution Batch)로 결합한다. 배칭(Batching)은 특히 여러 로봇 또는 여러 추론 스트림(Inference Stream)을 처리하는 중앙 엣지 서버(Centralized Edge Server)에서 GPU 사용률과 처리량을 크게 향상시킬 수 있다. 그러나 지나치게 적극적인 배칭은 즉각적인 응답이 필요한 인지 기능의 지연시간과 지터(Jitter)를 증가시킬 수 있으므로 로보틱스 엔지니어는 성능 향상과 실시간성 사이의 균형을 고려해야 한다.

트리톤(Triton)은 여러 모델 인스턴스(Model Instance)와 동시 모델 실행(Concurrent Model Execution)도 지원하므로 사용 가능한 GPU 자원을 여러 워크로드에 분배할 수 있다. 여러 신경망(Neural Network)을 실행하는 로봇에서는 각 프로세스가 독립적으로 GPU 컨텍스트(GPU Context)를 관리하는 방식보다 가속기 활용률(Accelerator Utilization)을 향상시킬 수 있다. 그러나 특히 임베디드 플랫폼(Embedded Platform)에서는 여러 모델이 GPU 메모리, 연산 성능, 메모리 대역폭(Memory Bandwidth), 열 및 전력 예산(Thermal or Power Budget)을 두고 경쟁할 수 있으므로 자원 계획(Resource Planning)이 여전히 중요하다.

토치서브(TorchServe)는 파이토치(PyTorch) 모델을 중심으로 설계된 서빙 아키텍처(Serving Architecture)를 제공한다. 모델을 전처리(Preprocessing), 추론(Inference), 후처리(Postprocessing)에 필요한 로직과 함께 패키징할 수 있기 때문에 파이토치 애플리케이션은 모든 추론 파이프라인을 애플리케이션 프로세스 내부에 직접 포함하는 대신 학습된 모델을 서비스 인터페이스(Service Interface)를 통해 제공할 수 있다. 학습과 추론 생태계가 주로 파이토치를 기반으로 구성된 경우 이러한 접근 방식은 모델 개발에서 실제 운영 서빙으로의 전환을 단순화할 수 있다.

그러나 서빙 계층(Serving Layer)은 단순히 학습 프레임워크와 일치한다는 이유만으로 선택해서는 안 되며 목표 로봇 아키텍처(Target Robot Architecture)를 기준으로 평가해야 한다. 편리한 서버 추상화(Server Abstraction)는 프로세스 경계(Process Boundary), 직렬화(Serialization), 통신 오버헤드(Communication Overhead), 추가적인 메모리 소비를 발생시킬 수 있다. 따라서 엄격한 제약을 가진 제어 루프(Control Loop)에서는 직접적인 런타임 통합(Direct Runtime Integration)이 더 적합할 수 있으며, 연산량이 큰 인지, 언어, 플릿 또는 비동기 AI 워크로드에는 서비스 지향 서빙(Service-oriented Serving)이 적합할 수 있다.

ONNX 런타임(ONNX Runtime)은 개방형 신경망 교환 형식(Open Neural Network Exchange Format)으로 표현된 모델을 위한 크로스 플랫폼 추론 런타임(Cross-platform Inference Runtime)을 제공한다는 점에서 다른 접근 방식을 사용한다. 파이토치와 같은 프레임워크에서 학습한 모델을 ONNX 형식으로 내보내고 사용 가능한 실행 제공자(Execution Provider)를 통해 ONNX 런타임에서 실행할 수 있다. 이를 통해 학습 프레임워크와 배포 하드웨어 사이에 유용한 추상화 계층(Abstract Layer)을 형성하고 원래 모델을 개발했던 환경에 대한 의존성을 줄일 수 있다.

실행 제공자(Execution Provider)는 ONNX 런타임이 추론 워크로드를 다양한 하드웨어 가속 기술(Hardware Acceleration Technology)에 매핑할 수 있도록 한다. 배포 플랫폼에 따라 CPU, GPU 또는 특수 가속 백엔드(Specialized Acceleration Backend)를 이용하여 추론을 실행할 수 있다. 이러한 유연성은 동일한 논리적 모델 인터페이스(Logical Model Interface)를 개발용 워크스테이션(Development Workstation), x86 엣지 컴퓨터, 임베디드 장치(Embedded Device), 기타 대상 시스템에 배포하면서 하드웨어별 최적화를 런타임 추상화 계층 아래에서 처리할 수 있다는 점에서 이기종 로봇 플릿(Heterogeneous Robot Fleet)에 유용하다.

ONNX 기반 배포는 명시적인 변환 및 검증 단계(Conversion and Validation Stage)도 필요로 한다. 내보낸 그래프(Exported Graph)는 지원되지 않는 연산자(Unsupported Operator), 수치 정밀도 변화(Numerical Precision Change), 그래프 변환(Graph Transformation), 동적 입력 처리(Dynamic Input Handling), 백엔드별 구현 차이로 인해 원래 모델과 다르게 동작할 수 있다. 따라서 원래 학습 프레임워크에서 정확도 검증을 통과한 모델이라도 변환된 산출물을 운영 모델 레지스트리(Production Model Registry)에 등록하기 전에 대표적인 로봇 데이터를 사용하여 ONNX 내보내기 이후 다시 검증해야 한다.

트리톤(Triton)과 ONNX 런타임(ONNX Runtime)은 반드시 서로 경쟁하는 대안 관계는 아니다. 트리톤을 서빙 계층으로 사용하면서 최적화된 런타임 백엔드(Optimized Runtime Backend)가 실제 모델 연산을 수행하도록 구성할 수 있다. 이러한 계층형 아키텍처(Layered Architecture)는 외부 추론 관리(External Inference Management)와 내부 실행 최적화(Internal Execution Optimization)를 분리한다. 예를 들어 플릿 서버(Fleet Server)는 트리톤을 통해 모델 버전, 요청 및 동시성을 관리하면서 각 모델은 배포 요구사항에 적합한 GPU 지향 또는 런타임별 백엔드를 통해 실행할 수 있다.

ROS2 통합(ROS2 Integration)에서는 추론을 ROS2 노드 내부에서 실행할 것인지 외부 서빙 프로세스(External Serving Process)를 통해 실행할 것인지에 대한 또 다른 아키텍처 결정이 필요하다. 임베디드 추론(Embedded Inference)은 통신 경계를 줄이고 지연시간을 최소화할 수 있는 반면, 외부 모델 서버(External Model Server)는 시스템 격리(Isolation)를 향상시키고 독립적인 모델 수명주기 관리를 가능하게 한다. 대규모 카메라 이미지나 포인트 클라우드(Point Cloud)가 이러한 경계를 통과하는 경우 공유 메모리(Shared Memory), 제로 카피 전송(Zero-copy Transport), 프로세스 내부 통신(Intra-process Communication), 효율적인 메시지 설계(Message Design)가 중요해진다.

모델 서빙(Model Serving)은 관측 가능성(Observability)도 포함해야 한다. 평균 추론 지연시간만으로는 충분하지 않은데, 평균 성능이 정상적으로 보이더라도 드물게 발생하는 지연시간 급증(Latency Spike)이 내비게이션이나 조작 기능을 방해할 수 있기 때문이다. 운영 시스템은 지연시간 분포, 요청률(Request Rate), 대기열 깊이(Queue Depth), GPU 및 CPU 사용률, 메모리 소비, 추론 실패(Inference Failure), 모델 로딩 상태(Model Loading Status), 필요한 경우 하드웨어 온도(Hardware Temperature)까지 관측해야 한다. 이러한 지표는 모델 서빙을 전체 로봇 관측 가능성 및 MLOps 인프라와 직접 연결한다.

따라서 적절한 서빙 아키텍처는 워크로드 특성과 시스템 경계(System Boundary)에 따라 결정된다. 트리톤(Triton)은 동시성과 운영 모델 관리가 중요한 중앙집중형 또는 다중 모델 GPU 서빙(Centralized or Multi-model GPU Serving)에 적합하다. 파이토치 중심 서빙(PyTorch-oriented Serving)은 파이토치 산출물과 긴밀하게 연결된 워크플로를 단순화할 수 있으며, ONNX 런타임은 이기종 대상 시스템 전반에서 이식 가능한 추론 계층(Portable Inference Layer)을 제공한다. 많은 로봇 시스템에서는 모든 컴퓨팅 계층에 하나의 프레임워크를 강제하기보다 이러한 기술을 함께 사용할 수 있다.

성숙한 로보틱스 배포(Robotics Deployment)는 궁극적으로 모델 개발(Model Development), 모델 표현(Model Representation), 추론 실행(Inference Execution), 서비스 관리(Service Management)를 명확하게 통제되는 계층으로 분리한다. 학습 과정은 검증된 산출물을 생성하고, 변환 과정은 배포 호환 표현(Deployment-compatible Representation)을 만들며, 최적화된 런타임은 실제 연산을 실행하고, 서빙 인프라는 운영 접근과 수명주기를 관리한다. 이러한 분리를 통해 로봇 AI 시스템은 재현성(Reproducibility), 관측 가능성, 롤백 기능, 예측 가능한 추론 동작을 유지하면서 GPU, 엣지 컴퓨터, 플릿 서버 전반으로 지속적으로 발전할 수 있다.

## 7.3. TensorRT Model Optimization and Engine Build Pipeline [w/Code]

![](images/image3.png){width="7.268055555555556in" height="7.268055555555556in"}

TensorRT는 학습된 신경망(Neural Network)을 NVIDIA GPU에서 사용할 수 있는 고도로 최적화된 실행 엔진(Execution Engine)으로 변환하기 위한 NVIDIA의 추론 최적화 및 런타임 기술(Inference Optimization and Runtime Technology)이다. 로보틱스(Robotics)에서 TensorRT는 일반적으로 모델 개발(Model Development)과 운영 추론(Production Inference) 사이에 위치하며, 이 환경에서는 지연시간(Latency), 처리량(Throughput), 메모리 사용량(Memory Consumption), 결정론적 실행 동작(Deterministic Execution Behavior)이 인지(Perception)와 자율 동작(Autonomous Operation)에 직접적인 영향을 준다. 최적화 파이프라인(Optimization Pipeline)은 검증된 모델을 실제 배포에 적합한 하드웨어 인식 런타임 산출물(Hardware-aware Runtime Artifact)로 변환한다.

일반적인 TensorRT 워크플로(Workflow)는 PyTorch와 같은 프레임워크(Framework)에서 생성된 학습 완료 모델(Trained Model)로부터 시작한다. 학습 과정에서 사용된 모델 표현을 최종 배포 산출물(Deployment Artifact)로 직접 사용하는 대신, 모델을 ONNX와 같은 상호운용 가능한 표현(Interoperable Representation)으로 내보낸다. 이를 통해 학습 환경(Training Environment)과 추론 환경(Inference Environment) 사이에 명확한 경계를 형성하고, 최적화를 시작하기 전에 입력 형상(Input Shape), 출력 텐서(Output Tensor), 연산자(Operator), 동적 차원(Dynamic Dimension), 호환성(Compatibility)을 검사할 수 있다.

내보낸 모델(Exported Model)은 엔진 생성(Engine Generation) 전에 반드시 검증해야 한다. 변환 과정에서 지원되지 않는 연산자(Unsupported Operator), 그래프 차이(Graph Difference), 동적 형상 문제(Dynamic-shape Problem), 수치적 불일치(Numerical Discrepancy)가 발생할 수 있기 때문이다. 대표적인 로봇 데이터(Representative Robot Data)를 원본 모델과 내보낸 모델 모두에서 처리하고, 정의된 허용 오차(Tolerance)를 기준으로 출력을 비교해야 한다. 이 단계에서 변환 문제를 발견하면 최적화 오류가 이후 배포 파이프라인에서 진단하기 어려운 장애로 발전하는 것을 방지할 수 있다.

TensorRT는 지원되는 네트워크 표현(Network Representation)을 파싱(Parsing)하고 대상 GPU에 최적화할 수 있는 내부 네트워크 정의(Internal Network Definition)를 구성한다. 엔진 빌드(Engine Build) 과정에서 TensorRT는 계층(Layer), 텐서 차원(Tensor Dimension), 데이터 유형(Data Type), 사용 가능한 커널(Kernel), 하드웨어 특성(Hardware Characteristic)을 분석한다. 이후 설정된 제약조건을 만족하면서 추론 비용(Inference Cost)을 줄일 수 있는 실행 전략(Execution Strategy)을 탐색한다. 따라서 엔진 빌드는 단순한 파일 형식 변환(File-format Conversion)이 아니라 최적화 과정이다.

계층 및 연산 융합(Layer and Operation Fusion)은 이러한 과정에서 중요한 최적화 메커니즘(Optimization Mechanism) 중 하나이다. 별도의 커널 실행(Kernel Launch)과 중간 메모리 전송(Intermediate Memory Transfer)이 필요했던 연산들을 더욱 효율적인 실행 경로(Execution Path)로 결합할 수 있다. TensorRT는 또한 텐서 레이아웃(Tensor Layout), 커널 선택(Kernel Selection), 메모리 재사용(Memory Reuse), 실행 스케줄링(Execution Scheduling)을 최적화할 수 있다. 이러한 변환은 로봇 애플리케이션이 각각의 최적화를 직접 구현하지 않아도 GPU 연산 자원의 활용도를 높이고 오버헤드(Overhead)를 감소시킨다.

정밀도 선택(Precision Selection)은 성능에 큰 영향을 미친다. FP32는 일반적인 수치 기준선(Numerical Baseline)을 제공하며, FP16은 메모리 전송량을 줄이고 낮은 정밀도 연산에 최적화된 GPU 하드웨어를 활용할 수 있다. INT8은 모델과 대상 하드웨어가 지원하는 경우 추가적인 성능 및 메모리상의 이점을 제공할 수 있지만, 낮아진 정밀도는 탐지 신뢰도(Detection Confidence), 위치 추정 정확도(Localization Accuracy), 분할 경계(Segmentation Boundary), 기타 후속 로봇 동작에 영향을 줄 수 있으므로 세심한 검증이 필요하다.

INT8 배포에서는 학습 후 양자화(Post-training Quantization)를 사용하는 경우 전통적으로 대표성 있는 보정 데이터(Calibration Data)가 필요하다. 보정 데이터는 임의의 샘플에 의존하기보다 로봇이 실제로 경험할 것으로 예상되는 센서 분포(Sensor Distribution)와 운영 환경(Operational Environment)을 반영해야 한다. 카메라 노출(Camera Exposure), 환경 조명(Environmental Lighting), 객체 분포(Object Distribution), 센서 특성(Sensor Characteristic), 운영 도메인(Operating Domain)은 활성화 통계(Activation Statistics)에 영향을 줄 수 있다. 부적절한 보정 데이터는 빠르게 동작하지만 실제 배포에 적합하지 않은 추론 품질을 가진 엔진을 생성할 수 있다.

동적 입력 형상(Dynamic Input Shape)은 또 다른 설계 고려사항이다. 로봇 인지 모델(Robot Perception Model)은 서로 다른 이미지 해상도, 배치 크기(Batch Size), 시퀀스 길이(Sequence Length), 가변 형상 입력(Variable-shaped Input)을 처리할 수 있다. TensorRT는 동적 입력에 대해 지원되는 최소(Minimum), 최적(Optimum), 최대(Maximum) 차원을 정의하는 최적화 프로파일(Optimization Profile)을 사용할 수 있다. 이를 통해 예상되는 운영 범위에서 엔진을 최적화할 수 있지만, 지나치게 넓은 범위는 최적화 복잡성을 증가시키고 실제 워크로드를 중심으로 설계한 프로파일보다 실행 효율을 낮출 수 있다.

엔진 생성은 작업 공간(Workspace)과 메모리 제약(Memory Constraint)의 영향도 받는다. 최적화 과정에서 TensorRT는 서로 다른 임시 메모리 용량과 실행 자원을 요구할 수 있는 구현 전략(Implementation Strategy)을 평가한다. 따라서 배포 엔지니어는 다른 로봇 워크로드와 함께 실제 GPU 메모리 예산(GPU Memory Budget)을 고려해야 한다. 독립적인 환경에서는 높은 성능을 제공하는 인지 엔진이라도 위치 추정(Localization), 매핑(Mapping), 계획(Planning), 시각화(Visualization), 추가 AI 모델이 동시에 GPU 메모리를 사용하면 시스템 수준의 문제를 발생시킬 수 있다.

생성된 직렬화 TensorRT 엔진(Serialized TensorRT Engine)은 특정 배포 환경에 최적화된 실행 계획(Execution Plan)을 나타낸다. 따라서 이를 모든 환경에서 사용할 수 있는 범용 모델 산출물(Universally Portable Model Artifact)로 취급해서는 안 된다. 호환성은 TensorRT 버전, GPU 아키텍처(GPU Architecture), 정밀도 처리 능력(Precision Capability), 플러그인(Plugin), 기타 런타임 특성(Runtime Characteristic)에 따라 달라질 수 있다. 따라서 이기종 로봇 플릿(Heterogeneous Robot Fleet)의 MLOps 파이프라인에서는 이식 가능한 원본 모델(Portable Source Model)을 유지하면서 서로 다른 GPU 등급과 소프트웨어 환경을 위한 대상별 엔진(Target-specific Engine)을 생성할 수 있다.

내보낸 그래프(Exported Graph)에 TensorRT의 기본 계층(Built-in Layer)으로 직접 구현할 수 없는 기능이 포함되어 있다면 사용자 정의 연산자(Custom Operator)에 대한 추가적인 고려가 필요하다. TensorRT 플러그인(TensorRT Plugin)을 사용하면 이러한 연산에 최적화된 구현을 제공할 수 있지만, 플러그인은 배포 의존성 체인(Deployment Dependency Chain)의 일부가 된다. 따라서 재현성(Reproducibility)을 유지하려면 플러그인 버전, 소스 코드(Source Code), 컴파일된 바이너리(Compiled Binary), 대상 아키텍처(Target Architecture), 호환성 정보를 모델 및 엔진 빌드 구성과 함께 버전 관리해야 한다.

따라서 엔진 빌드는 엔지니어의 워크스테이션에서 수동으로 수행하기보다 모델 배포 파이프라인(Model Deployment Pipeline)의 일부로 자동화해야 한다. 재현 가능한 빌드 프로세스(Reproducible Build Process)는 모델 레지스트리(Model Registry)에서 검증된 모델을 가져오고 메타데이터(Metadata)를 확인한 다음, ONNX 표현을 내보내거나 불러오고, 최적화 프로파일을 구성하며, 정밀도 정책(Precision Policy)을 선택하고, TensorRT 엔진을 빌드한 후 검증 및 벤치마킹(Validation and Benchmarking)을 수행하여 최종 산출물을 등록할 수 있다.

엔진 생성 이후의 검증(Validation)은 필수적이다. 최적화에 성공했다는 사실이 애플리케이션 수준의 정확성(Application-level Correctness)을 보장하지 않기 때문이다. 대표적인 데이터셋을 사용하여 TensorRT 출력과 검증된 기준 모델(Reference Model)을 비교하고 해당 작업에 적합한 평가 지표(Task Metric)를 적용해야 한다. 객체 탐지 모델(Object Detection Model)은 정확도와 신뢰도를 비교할 수 있으며, 분할(Segmentation), 자세 추정(Pose Estimation), 깊이 추정(Depth Estimation), 제어 관련 네트워크(Control-related Network)는 각각 출력 특성에 적합한 지표를 사용해야 한다. 단순한 수치적 동등성(Numerical Equivalence)만으로 실제 운영 적합성을 완전히 판단할 수는 없다.

성능 벤치마킹(Performance Benchmarking)은 평균 추론 지연시간만 평가해서는 안 된다. 로보틱스 시스템에서는 지연시간 분포(Latency Distribution), 처리량, 워밍업 동작(Warm-up Behavior), GPU 사용률, 메모리 소비량, 동시 워크로드(Concurrent Workload) 환경에서의 성능을 함께 측정하는 것이 중요하다. 최종 배포 결정은 전체 로봇 소프트웨어 스택(Robot Software Stack)을 반영해야 한다. 독립적으로 최대 처리량에 최적화된 엔진이라도 다른 실시간 구성요소와 통합되었을 때 허용할 수 없는 지연시간이나 자원 경합(Resource Contention)을 발생시킬 수 있기 때문이다.

TensorRT 엔진 산출물은 MLOps 수명주기 전체에서 출처 추적성(Provenance)과 연결된 상태를 유지해야 한다. 모델 레지스트리에는 원본 모델 버전(Source Model Version), ONNX 산출물, 학습 구성(Training Configuration), 정밀도 모드(Precision Mode), 최적화 프로파일, TensorRT 및 CUDA 환경, 대상 하드웨어(Target Hardware), 검증 결과(Validation Result), 벤치마크 결과(Benchmark Result)가 식별 가능하도록 기록되어야 한다. 이러한 추적 가능성(Traceability)을 통해 엔진을 재현하고, 현장 장애(Field Failure)를 진단하며, 최적화 변경 사항을 비교하고, 이전에 검증된 산출물로 안전하게 롤백(Rollback)할 수 있다.

플릿 규모(Fleet Scale)에서 엔진 빌드 파이프라인은 하드웨어 독립적인 모델 개발(Hardware-independent Model Development)과 하드웨어별 추론 배포(Hardware-specific Inference Deployment)를 연결하는 다리 역할을 한다. 하나의 검증된 모델에서 서로 다른 NVIDIA GPU 대상에 최적화된 여러 TensorRT 엔진을 생성할 수 있으며, 배포 정책(Deployment Policy)은 각 로봇에 적합한 산출물을 선택한다. 모델 레지스트리, 자동화된 검증(Automated Validation), 관측 가능성(Observability), 롤백 메커니즘을 결합하면 TensorRT 기반 최적화 추론을 개별적인 성능 튜닝 작업이 아니라 통제되고 재현 가능한 로봇 MLOps 구성요소로 운영할 수 있다.

## 7.4. Edge Model Deployment Jetson Hailo OpenVINO [w/Code]

![](images/image4.png){width="7.268055555555556in" height="7.268055555555556in"}

엣지 모델 배포(Edge Model Deployment)는 인공지능 추론(AI Inference)을 중앙집중형 클라우드 인프라(Centralized Cloud Infrastructure)에서 데이터가 생성되는 로봇, 기계 또는 센서 시스템 가까이로 이동시킨다. 로보틱스(Robotics)에서는 이를 통해 네트워크 의존성(Network Dependency)을 줄이고 연결이 불안정한 상황에서도 인지(Perception)와 의사결정 지원(Decision-support) 워크로드를 지속적으로 수행할 수 있다. 엣지 배포에서는 추론 성능과 함께 전력 소비(Power Consumption), 열 한계(Thermal Limit), 메모리 용량, 하드웨어 인터페이스(Hardware Interface), 물리적 크기, 로봇 소프트웨어 스택(Robot Software Stack)의 실시간 요구사항을 균형 있게 고려해야 한다.

엣지 AI 배포 파이프라인(Edge AI Deployment Pipeline)은 일반적으로 하드웨어별 최적화(Hardware-specific Optimization)를 바로 수행하기보다 검증된 모델(Validated Model)에서 시작한다. 원본 모델(Source Model)은 PyTorch 또는 다른 학습 프레임워크(Training Framework)에서 생성된 후 ONNX와 같은 상호운용 가능한 표현(Interoperable Representation)으로 내보낼 수 있다. 이러한 이식 가능한 산출물(Portable Artifact)을 유지하면 모델 개발과 대상별 컴파일(Target-specific Compilation)을 분리할 수 있으며, 하나의 검증된 모델 계보(Model Lineage)에서 여러 종류의 엣지 하드웨어에 최적화된 배포 산출물을 생성할 수 있다.

NVIDIA Jetson 플랫폼은 CPU와 NVIDIA GPU 자원을 소형의 전력 제약 시스템(Power-constrained System)에 결합한 이기종 엣지 컴퓨팅 아키텍처(Heterogeneous Edge Computing Architecture)를 제공한다. 로봇 애플리케이션에서 Jetson 장치는 센서 가까이에서 카메라 인지(Camera Perception), 객체 탐지(Object Detection), 분할(Segmentation), 자세 추정(Pose Estimation), 깊이 처리(Depth Processing), 위치 추정 지원(Localization Support), 멀티모달 AI(Multimodal AI)를 실행할 수 있다. CUDA 중심의 소프트웨어 생태계(Software Ecosystem)를 통해 대형 NVIDIA GPU 시스템에서 사용하는 기술을 배포 파이프라인에서 재사용할 수도 있다.

Jetson 배포는 CUDA, TensorRT 및 플랫폼별 시스템 소프트웨어(Platform-specific System Software)를 결합하는 경우가 많다. 학습된 모델을 ONNX로 내보내고 검증한 후 최적화된 TensorRT 엔진으로 변환하여 ROS2 노드(Node) 또는 추론 서비스(Inference Service)에 통합할 수 있다. FP16 또는 INT8 정밀도(Precision)는 적절하게 검증될 경우 연산량과 메모리 요구량을 줄일 수 있다. 데스크톱 GPU의 성능은 임베디드 환경의 열, 전력, 메모리 제약을 정확하게 나타내지 못하므로 생성된 엔진은 실제 Jetson 대상 장치에서 벤치마킹해야 한다.

Jetson 기반 로봇에서는 자원 공유(Resource Sharing)가 특히 중요하다. AI 추론은 영상 처리(Image Processing), 위치 추정(Localization), 매핑(Mapping), 내비게이션(Navigation), 시각화(Visualization), 기타 GPU 또는 CPU 워크로드와 자원을 공유한다. 따라서 최대 신경망 처리량(Maximum Neural-network Throughput)이 항상 올바른 최적화 목표가 되는 것은 아니다. 실제 동시 워크로드(Concurrent Workload) 환경에서 종단 간 지연시간(End-to-end Latency), 프레임 일관성(Frame Consistency), GPU 메모리 압력(Memory Pressure), CPU 사용률, 전력 모드(Power Mode), 온도, 전체 시스템 동작을 평가해야 한다.

Hailo는 범용 GPU 실행(General-purpose GPU Execution)이 아니라 전용 신경망 가속(Dedicated Neural-network Acceleration)을 기반으로 하는 또 다른 엣지 AI 접근 방식을 제공한다. 이 아키텍처에서는 지원되는 신경망 워크로드를 특수 가속기 하드웨어(Specialized Accelerator Hardware)에 맞게 컴파일하고 관련 런타임 스택(Runtime Stack)을 통해 실행한다. 호스트 프로세서(Host Processor)는 ROS2, 센서 통신, 제어 로직(Control Logic), 시스템 관리를 계속 담당하면서 선택된 추론 워크로드를 가속기로 오프로딩(Offloading)할 수 있다.

이러한 전용 가속(Dedicated Acceleration)은 로봇이 엄격한 전력 및 열 제약을 갖거나 AI 추론이 호스트 컴퓨팅 자원을 적게 사용해야 하는 경우 유용할 수 있다. 그러나 실제 배포 가능성은 가속기 툴체인(Accelerator Toolchain)과 모델의 호환성에 크게 좌우된다. 따라서 가속기를 운영 모델의 대상으로 선정하기 전에 네트워크 연산자(Network Operator), 텐서 차원(Tensor Dimension), 전처리 요구사항(Preprocessing Requirement), 양자화 동작(Quantization Behavior), 컴파일러 지원(Compiler Support), 런타임 인터페이스(Runtime Interface)를 평가해야 한다.

따라서 Hailo 배포 프로세스에는 모델 변환 또는 파싱(Model Translation or Parsing), 최적화, 필요한 경우 양자화(Quantization), 하드웨어별 컴파일(Hardware-specific Compilation), 대상 가속기를 위한 실행 가능 산출물(Executable Artifact) 생성이 포함된다. 대표성 있는 보정 또는 최적화 데이터(Calibration or Optimization Data)는 실제 로봇의 센서 조건을 반영해야 한다. 컴파일 이후에는 하드웨어별 변환과 낮은 정밀도가 추론 동작을 변화시킬 수 있으므로 검증된 기준 모델(Validated Reference Model)과 정확도를 비교해야 한다.

OpenVINO는 지원되는 컴퓨팅 대상에서 AI 추론을 실행하기 위한 배포 및 최적화 툴킷(Deployment and Optimization Toolkit)을 제공하며, 특히 Intel 중심 환경(Intel-oriented Environment)에서 활용할 수 있다. 지원되는 모델 표현(Model Representation)을 변환하거나 직접 입력받아 효율적인 런타임 실행(Runtime Execution)을 위해 준비할 수 있다. 이기종 x86 및 엣지 시스템을 운영하는 로보틱스 팀에서는 애플리케이션 수준의 추론 인터페이스(Application-level Inference Interface)를 개별 컴퓨팅 장치의 세부사항으로부터 분리하는 소프트웨어 추상화(Software Abstraction)를 제공할 수 있다.

OpenVINO 중심 파이프라인은 일반적으로 학습된 모델을 런타임에서 지원되는 표현으로 변환하거나 가져온 후 적절한 최적화를 적용하고 사용 가능한 실행 장치(Execution Device)를 선택한다. 이후 로봇 애플리케이션은 일관된 런타임 인터페이스를 통해 추론을 호출하고, 배포 구성(Deployment Configuration)에 따라 실제 연산이 수행될 위치를 결정할 수 있다. 다른 런타임과 마찬가지로 모델 변환에 성공했다는 사실만으로 호환성 및 수치적 동작(Numerical Behavior)을 가정해서는 안 되며, 대표적인 운영 데이터를 사용하여 검증해야 한다.

따라서 Jetson, Hailo, OpenVINO는 완전히 동일한 대안이라기보다 서로 다른 배포 계층(Deployment Layer)과 하드웨어 전략(Hardware Strategy)을 나타낸다. Jetson은 범용 임베디드 컴퓨팅(General-purpose Embedded Computing)과 NVIDIA GPU 가속을 결합하고, Hailo는 전용 AI 가속에 중점을 두며, OpenVINO는 지원되는 하드웨어 환경을 대상으로 하는 추론 소프트웨어 스택(Inference Software Stack)을 제공한다. 서로 다른 워크로드가 각기 다른 지연시간, 전력, 유연성 요구사항을 갖는 경우 이기종 로봇 플랫폼(Heterogeneous Robot Platform)에서 두 가지 이상의 접근 방식을 함께 사용할 수도 있다.

이러한 이기종 시스템에서는 모델 분할(Model Partitioning)이 유용할 수 있다. 높은 대역폭(High-bandwidth)의 카메라 인지 작업은 전용 가속기에서 실행하고, 유연한 CUDA 연산이 필요한 모델은 Jetson GPU 자원에서 실행하며, 일반적인 로봇 소프트웨어는 CPU에서 실행할 수 있다. 프로세서 사이에서 대형 텐서를 불필요하게 이동하면 데이터 변환(Data Conversion), 메모리 복사(Memory Copy), 장치 간 통신(Inter-device Communication)으로 인해 가속을 통해 확보한 지연시간 및 에너지상의 이점이 사라질 수 있으므로 아키텍처에서는 이러한 데이터 이동을 최소화해야 한다.

ROS2 통합(ROS2 Integration)은 하드웨어별 추론 기능을 안정적인 소프트웨어 경계(Software Boundary)를 통해 제공해야 한다. 기반 가속기가 변경될 때마다 인지 노드(Perception Node)를 대규모로 재설계해야 하는 구조는 피하는 것이 바람직하다. 추론 추상화(Inference Abstraction)는 일관된 ROS2 토픽(Topic), 서비스(Service), 액션(Action)을 유지하면서 전처리, 런타임 호출(Runtime Invocation), 후처리(Postprocessing)를 격리할 수 있다. 이를 통해 하드웨어 마이그레이션(Hardware Migration)을 쉽게 하고 전체 로봇 애플리케이션을 다시 작성하지 않고도 다양한 런타임을 벤치마킹할 수 있다.

하드웨어 인식 패키징(Hardware-aware Packaging)도 엣지 배포의 핵심 요소이다. 하나의 모델 패키지(Model Package)에는 이식 가능한 원본 산출물, 대상별 최적화 바이너리(Target-specific Optimized Binary), 런타임 라이브러리(Runtime Library), 전처리 구성, 레이블 메타데이터(Label Metadata), 보정 정보(Calibration Information), 호환성 제약조건(Compatibility Constraint)이 포함될 수 있다. 배포 시스템은 Jetson TensorRT 엔진, 전용 가속기 바이너리, OpenVINO 호환 모델이 적절한 대상에만 전달되도록 산출물을 선택하기 전에 로봇 하드웨어를 식별해야 한다.

엣지 배포 검증(Edge Deployment Validation)은 모델 정확도만이 아니라 시스템 전체의 동작을 측정해야 한다. 주요 관측 항목에는 추론 지연시간 분포(Inference Latency Distribution), 지속 프레임률(Sustained Frame Rate), 메모리 소비, CPU 및 가속기 사용률, 전력 소비, 온도, 스로틀링(Throttling), 모델 초기화 시간(Model Initialization Time), 장애 복구(Failure Recovery)가 포함된다. 특히 여러 시간 동안 연속으로 운용되는 로봇에서는 짧은 실험실 벤치마크에서 드러나지 않는 열 또는 자원 영향을 확인할 수 있도록 충분한 시간 동안 테스트해야 한다.

플릿 관리(Fleet Management)는 이러한 과정을 개별 장치에서 전체 로봇 집단으로 확장한다. 이기종 플릿(Heterogeneous Fleet)은 여러 세대의 Jetson, 다양한 가속기 구성(Accelerator Configuration), x86 엣지 컴퓨터를 포함할 수 있다. 따라서 모델 레지스트리(Model Registry)는 각 모델 버전을 호환 가능한 대상 산출물(Target Artifact), 런타임 버전, 정밀도 설정(Precision Setting), 검증 결과, 하드웨어 프로파일(Hardware Profile)과 연결해야 한다. 이후 배포 정책(Deployment Policy)은 모델 계보와 롤백 기능(Rollback Capability)을 유지하면서 각 로봇에 적합한 패키지를 선택할 수 있다.

엣지 모델 배포의 더 큰 목적은 단순히 가속기 성능을 극대화하는 것이 아니라 제한된 물리적 하드웨어에서 예측 가능한 로봇 지능(Predictable Robot Intelligence)을 구현하는 것이다. Jetson, Hailo, OpenVINO는 추론을 센서와 액추에이터(Actuator) 가까이에서 수행하기 위한 서로 다른 메커니즘을 제공한다. 이식 가능한 모델 표현, 하드웨어별 최적화, ROS2 추상화, 검증, 관측 가능성(Observability), 플릿 인식 패키징(Fleet-aware Packaging), 롤백을 결합하면 학습된 모델에서 신뢰할 수 있는 엣지 AI 운영(Edge AI Operation)으로 이어지는 통제된 MLOps 경로를 구축할 수 있다.

## 7.5. Model Packaging and Container Image for Robot AI [w/Code]

![](images/image5.png){width="7.268055555555556in" height="7.268055555555556in"}

모델 패키징(Model Packaging)은 검증된 AI 모델을 일관되게 추론(Inference)하는 데 필요한 모든 요소를 포함하는 배포 가능한 단위(Deployable Unit)로 변환하는 과정이다. 로봇 AI 시스템에서는 모델 파일만으로 충분한 경우가 드물다. 배포에는 전처리 및 후처리 로직(Preprocessing and Postprocessing Logic), 런타임 라이브러리(Runtime Library), 구성 파일(Configuration File), 레이블 정의(Label Definition), 하드웨어별 엔진(Hardware-specific Engine), 보정 데이터(Calibration Data), 메타데이터(Metadata), 호환성 정보(Compatibility Information) 등이 추가로 필요할 수 있으며, 이러한 요소들이 함께 모델의 실제 운영 동작을 정의한다.

신뢰할 수 있는 패키징 전략(Packaging Strategy)은 이식 가능한 모델 산출물(Portable Model Artifact)과 배포별 산출물(Deployment-specific Artifact)을 분리하는 것에서 시작한다. 이식 가능한 표현에는 PyTorch 체크포인트(Checkpoint) 또는 ONNX 모델이 사용될 수 있으며, 대상별 산출물에는 TensorRT 엔진이나 특수 가속기(Specialized Accelerator)를 위해 생성된 바이너리(Binary)가 포함될 수 있다. 이러한 분리를 유지하면 각각의 최적화 바이너리를 독립적인 모델로 취급하지 않고도 하나의 검증된 모델 계보(Model Lineage)를 여러 로봇 하드웨어 구성에서 사용할 수 있다.

메타데이터(Metadata)는 모델과 실제 운영 환경(Operational Context)을 연결하는 역할을 한다. 패키지는 모델 버전(Model Version), 예상 입력과 출력(Expected Input and Output), 전처리 매개변수(Preprocessing Parameter), 정밀도 모드(Precision Mode), 런타임 요구사항(Runtime Requirement), 호환 가능한 하드웨어, 관련 검증 정보(Validation Information)를 식별할 수 있어야 한다. 이러한 메타데이터가 없다면 기술적으로 정상적인 모델 파일도 호환되지 않는 환경에 배포될 수 있으며, 추론 런타임이 모델을 정상적으로 로딩하더라도 잘못된 동작을 생성할 수 있다.

전처리(Preprocessing)와 후처리(Postprocessing)는 실질적인 추론 파이프라인(Inference Pipeline)의 일부이므로 모델과 함께 버전 관리해야 한다. 이미지 크기 조정(Image Resizing), 정규화(Normalization), 색 공간 변환(Color-space Conversion), 텐서 레이아웃(Tensor Layout), 신뢰도 임계값(Confidence Threshold), 비최대 억제(Non-maximum Suppression), 좌표 변환(Coordinate Transformation), 클래스 매핑(Class Mapping)은 모델 출력에 상당한 영향을 줄 수 있다. 이러한 요소를 함께 패키징하면 서로 다른 버전이 플릿(Fleet)에 배포되는 과정에서 애플리케이션 코드와 모델 동작이 인지되지 않은 채 서로 달라지는 것을 방지할 수 있다.

컨테이너 이미지(Container Image)는 모델 실행에 필요한 소프트웨어 환경을 캡슐화(Encapsulation)함으로써 모델 패키징을 확장한다. 하나의 컨테이너에는 추론 애플리케이션(Inference Application), 런타임 라이브러리, ROS2 의존성(Dependency), Python 또는 C++ 구성요소, 시스템 패키지(System Package), AI 서비스를 실행하는 데 필요한 설정을 포함할 수 있다. 이를 통해 호스트 운영체제(Host Operating System)와 모델 서빙 환경(Model-serving Environment) 사이에 재현 가능한 경계(Reproducible Boundary)를 형성하여 개발, 검증, 운영 시스템 사이의 배포 차이를 줄일 수 있다.

로봇 AI 컨테이너(Robot AI Container)는 전체 개발 환경을 그대로 복제하기보다 운영에 필요한 런타임 요구사항에 집중해야 한다. 학습 데이터셋(Training Dataset), 컴파일러 캐시(Compiler Cache), 임시 파일, 디버깅 도구(Debugging Utility), 패키지 관리자(Package Manager), 불필요한 개발 의존성(Development Dependency)은 이미지 크기를 증가시키고 공격 표면(Attack Surface)을 확대한다. 다단계 컨테이너 빌드(Multi-stage Container Build)를 사용하면 컴파일과 패키징 과정을 최종 런타임 이미지와 분리하여 운영 실행에 필요한 구성요소만 남길 수 있다.

GPU 지원 컨테이너(GPU-enabled Container)는 컨테이너와 호스트 시스템 사이에 추가적인 의존성을 갖는다. NVIDIA 기반 배포에서는 일반적으로 호환 가능한 GPU 드라이버와 컨테이너 런타임 통합(Container Runtime Integration)이 필요하며, CUDA, TensorRT 및 관련 사용자 공간 라이브러리(User-space Library)는 대상 환경의 호환성 제약조건을 따라야 한다. 따라서 컨테이너 이미지는 모든 하드웨어 및 드라이버 의존성을 자동으로 제거하는 기술이 아니라 재현 가능한 소프트웨어 패키징 방법으로 이해해야 한다.

하드웨어별 모델 산출물(Hardware-specific Model Artifact)은 컨테이너 워크플로(Container Workflow) 내부에서 신중하게 관리해야 한다. 예를 들어 직렬화된 TensorRT 엔진(Serialized TensorRT Engine)은 GPU 아키텍처, TensorRT 환경, 정밀도 설정, 엔진 생성 과정에서 사용된 플러그인(Plugin)에 의존할 수 있다. 따라서 하나의 컨테이너 이미지에 모든 환경에서 사용할 수 있는 범용 엔진을 포함한다고 가정하기보다, 정의된 하드웨어 프로파일(Hardware Profile)에 맞는 별도의 대상 변형(Target Variant)을 관리하거나 최종 패키지를 게시하기 전에 최적화 엔진을 생성하는 방법을 사용할 수 있다.

컨테이너 레지스트리(Container Registry)는 로봇 AI 이미지를 통제된 방식으로 배포하기 위한 중앙 배포 지점(Distribution Point)을 제공한다. 이미지는 변경 불가능한 다이제스트(Immutable Digest)와 의미 있는 버전 태그(Version Tag)를 사용하여 식별할 수 있으며, 배포 시스템은 로컬에서 임의로 빌드된 컨테이너가 아니라 승인된 산출물을 참조할 수 있다. 레지스트리는 소프트웨어 공급망(Software Supply Chain)의 일부가 되어 CI/CD 파이프라인, 모델 레지스트리(Model Registry), 보안 스캐닝(Security Scanning), 배포 정책(Deployment Policy), 플릿 관리 시스템(Fleet Management System)을 각 로봇에 설치된 정확한 소프트웨어 이미지와 연결한다.

모델 레지스트리(Model Registry)와 컨테이너 레지스트리(Container Registry)는 서로 연관되어 있지만 서로 다른 목적을 가진다. 모델 레지스트리는 AI 산출물, 모델 버전, 평가 결과(Evaluation Result), 계보(Lineage), 배포 상태(Deployment Status)를 추적하고, 컨테이너 레지스트리는 실행 가능한 소프트웨어 이미지(Executable Software Image)를 저장한다. 성숙한 MLOps 워크플로에서는 특정 컨테이너 이미지를 해당 이미지를 생성하는 데 사용된 정확한 모델 산출물, 런타임 구성(Runtime Configuration), 전처리 구현(Preprocessing Implementation), 검증 증거(Validation Evidence)까지 추적할 수 있도록 두 레지스트리를 연결한다.

따라서 버전 관리(Versioning)는 여러 계층에서 동시에 이루어져야 한다. 모델에는 하나의 의미론적 버전(Semantic Version) 또는 레지스트리 버전(Registry Version)이 존재할 수 있고, 런타임 애플리케이션에는 별도의 소프트웨어 버전이 있으며, 컨테이너 이미지에는 고유한 변경 불가능 다이제스트가 존재할 수 있다. 배포 메타데이터는 이러한 관계를 보존해야 한다. 현장 장애(Field Failure)가 발생하면 엔지니어는 영향을 받은 로봇에서 정확히 어떤 모델, 런타임, 구성, 컨테이너, 하드웨어 프로파일이 활성화되어 있었는지 확인할 수 있어야 한다.

ROS2 애플리케이션은 AI 노드가 복잡하거나 서로 충돌하는 의존성을 갖는 경우 컨테이너 경계(Container Boundary)를 활용할 수 있다. 인지 컨테이너(Perception Container)는 자체 런타임 환경을 패키징하면서 정의된 ROS2 인터페이스를 통해 내비게이션(Navigation), 매핑(Mapping), 플릿 구성요소와 통신할 수 있다. 그러나 컨테이너화(Containerization)가 통신 비용을 제거하는 것은 아니다. 대용량 이미지와 포인트 클라우드(Point Cloud)를 처리할 때는 불필요한 직렬화(Serialization)와 복사 오버헤드를 줄이기 위해 공유 메모리(Shared Memory), 호스트 네트워킹(Host Networking), 최적화된 DDS 구성 또는 기타 전송 전략(Transport Strategy)이 필요할 수 있다.

운영 값이 로봇이나 배포 사이트마다 다른 경우 구성(Configuration)은 일반적으로 변경 불가능한 컨테이너 내용(Immutable Container Content)과 분리하여 유지하는 것이 적절하다. 모델 선택(Model Selection), 센서 식별자(Sensor Identifier), 임계값, 토픽 이름(Topic Name), 장치 할당(Device Assignment), 자원 제한(Resource Limit)은 구성 파일, 환경 설정(Environment Setting), 마운트된 자원(Mounted Resource), 오케스트레이션 메커니즘(Orchestration Mechanism)을 통해 제공할 수 있다. 이를 통해 하나의 검증된 컨테이너 이미지를 각 로봇마다 소프트웨어를 다시 빌드하지 않고 통제된 여러 구성에서 사용할 수 있다.

보안(Security)은 모델 패키징의 필수적인 부분이다. 컨테이너 이미지는 신뢰할 수 있는 베이스 이미지(Trusted Base Image)를 사용하고, 불필요한 패키지를 최소화하며, 내장된 비밀정보(Embedded Secret)를 포함하지 않고, 릴리스 전에 취약점 스캐닝(Vulnerability Scanning)을 수행해야 한다. 모델 산출물과 컨테이너에는 체크섬(Checksum), 서명(Signature), 출처 기록(Provenance Record), 소프트웨어 자재명세서(Software Bill of Materials, SBOM)를 연결할 수도 있다. 이러한 통제를 통해 로봇에 배포된 산출물이 조직의 검증 및 릴리스 파이프라인을 통과한 것과 동일한 산출물임을 확인할 수 있다.

컨테이너 빌드 파이프라인(Container Build Pipeline)은 자동화되고 재현 가능해야 한다. 일반적인 프로세스는 승인된 모델 산출물을 가져오고, 필요한 런타임과 하드웨어 프로파일을 선택하고, 애플리케이션 의존성과 구성을 조립하고, 컨테이너 이미지를 빌드하고, 테스트와 보안 검사를 수행한 다음 이미지를 레지스트리에 게시한다. 모델과 완전한 런타임 패키지가 정의된 요구사항을 만족한다는 것을 검증 게이트(Validation Gate)가 확인한 이후에만 배포를 진행해야 한다.

검증은 원본 모델뿐만 아니라 조립이 완료된 컨테이너(Assembled Container)까지 포함해야 한다. 통합 테스트(Integration Test)를 통해 모델 로딩, 센서 입력 처리, 전처리, 추론, 후처리, ROS2 통신, 가속기 접근(Accelerator Access), 종료 또는 복구 동작(Shutdown or Recovery Behavior)을 확인해야 한다. 또한 컨테이너 수준의 구성이 모델 정확도를 변화시키지 않더라도 시스템 동작에 영향을 줄 수 있으므로 성능 테스트에서는 추론 지연시간, 메모리 사용량, 시작 시간(Startup Time), CPU 및 가속기 사용률, 지속적인 운영 성능을 측정해야 한다.

플릿 배포(Fleet Deployment)에서는 변경 불가능한 패키징(Immutable Packaging)이 특히 큰 가치를 갖는다. 각 로봇은 자신의 컨테이너 다이제스트(Container Digest), 모델 버전, 구성 버전(Configuration Version), 하드웨어 프로파일을 플릿 관리 인프라에 보고할 수 있다. 운영자는 이를 통해 배포 차이를 식별하고, 단계적 롤아웃(Staged Rollout)을 수행하고, 운영 지표(Operational Metric)를 비교하며, 영향을 받은 로봇을 이전에 검증된 이미지로 복귀시킬 수 있다. 롤백(Rollback)은 이전 소프트웨어 환경을 수동으로 재구성하는 작업이 아니라 통제된 산출물 전환(Controlled Artifact Transition)이 된다.

모델 패키징과 컨테이너화는 궁극적으로 AI 개발과 로봇 운영 사이에 재현 가능한 경계(Reproducible Boundary)를 형성한다. 모델은 학습된 동작(Learned Behavior)을 정의하고, 패키지는 특정 소프트웨어 및 하드웨어 환경에서 그 동작이 어떻게 실행되는지를 정의한다. 모델 산출물, 런타임 의존성, 구성, 컨테이너 이미지, 레지스트리, 검증 증거, 보안 메타데이터(Security Metadata), 관측 가능성(Observability), 롤백을 연결함으로써 로봇 MLOps는 AI 소프트웨어를 느슨하게 관리되는 파일들의 집합이 아니라 추적 가능하고 반복적으로 배포할 수 있는 운영 단위(Production Unit)로 제공할 수 있다.

## 7.6. A B Model Testing in Production Robot Fleet [w/Code]

![](images/image6.png){width="7.268055555555556in" height="7.268055555555556in"}

운영 로봇 플릿(Production Robot Fleet)에서의 A/B 모델 테스트(A/B Model Testing)는 실제 운영 조건에서 두 가지 모델 변형(Model Variant)을 통제된 방식으로 비교하는 방법이다. 모델 A(Model A)는 일반적으로 기존 운영 기준 모델(Production Baseline)을 의미하며, 모델 B(Model B)는 정확도, 지연시간, 강건성(Robustness), 효율성 또는 기타 측정 가능한 성능을 개선하기 위한 후보 모델을 의미한다. 전체 플릿을 동시에 교체하는 대신 선택된 로봇이나 임무(Mission)에 각각의 모델을 적용하여 비교 가능한 운영 증거(Operational Evidence)를 생성한다.

A/B 테스트의 목적은 일반적인 오프라인 모델 평가(Offline Model Evaluation)와 다르다. 오프라인 데이터셋은 반복 가능한 벤치마크(Benchmark)를 제공하지만 변화하는 조명, 센서 성능 저하(Sensor Degradation), 바닥 상태, 교통 패턴(Traffic Pattern), 사람과의 상호작용, 네트워크 품질, 임무 복잡성을 완전히 재현할 수는 없다. 운영 A/B 테스트는 실제 운영 분포(Operating Distribution)에 모델을 노출하고, 개발 과정에서 관측된 성능 개선이 완전한 로봇 시스템에 통합된 이후에도 의미 있게 유지되는지를 엔지니어링 팀이 판단할 수 있도록 한다.

운영 실험(Production Experiment)은 명확하게 정의된 가설(Hypothesis)과 측정 가능한 성공 기준(Success Criteria)에서 시작한다. 예를 들어 모델 B가 허용 가능한 임계값 이상으로 추론 지연시간(Inference Latency)을 증가시키지 않으면서 미탐지(Missed Detection)를 감소시킬 것으로 예상할 수 있다. 실험에서는 배포 전에 주요 지표(Primary Metric), 보조 지표(Supporting Metric), 안전 제약조건(Safety Constraint), 종료 조건(Termination Condition)을 정의해야 한다. 사전에 정의된 기준이 없다면 실험 이후 텔레메트리(Telemetry)를 선택적으로 해석하여 재현하기 어려운 결론에 도달할 수 있다.

로봇을 A 그룹과 B 그룹에 할당하려면 신중한 실험 설계(Experimental Design)가 필요하다. 로봇과 환경이 충분히 유사하다면 단순 무작위 할당(Random Assignment)을 통해 체계적 편향(Systematic Bias)을 줄일 수 있지만, 실제 플릿에는 서로 다른 하드웨어 세대(Hardware Generation), 사이트(Site), 임무 프로파일(Mission Profile), 센서, 운영 일정이 존재하는 경우가 많다. 층화 할당(Stratified Assignment)을 사용하면 이러한 특성을 두 그룹에 분산시켜 측정된 차이가 관련 없는 플릿 구성보다 모델 변경에서 발생했을 가능성을 높일 수 있다.

실험 단위(Experimental Unit) 역시 올바르게 정의해야 한다. 애플리케이션 특성에 따라 A/B 할당은 로봇, 사이트, 임무, 시간 구간(Time Window), 추론 요청(Inference Request) 수준에서 이루어질 수 있다. 모델 출력이 상태 기반 내비게이션(Stateful Navigation)이나 시간적 인지(Temporal Perception)에 영향을 미치는 경우 개별 추론 요청마다 모델을 전환하는 방식은 적절하지 않을 수 있다. 로봇 수준 또는 임무 수준 할당은 하나의 모델이 서로 연관된 전체 의사결정 시퀀스(Decision Sequence)를 담당하기 때문에 보다 명확한 운영 경계를 제공하는 경우가 많다.

안전 요구사항(Safety Requirement)은 운영 실험에 중요한 제한을 부여한다. 후보 모델은 실제 로봇에 대한 제어 권한(Control Authority)을 부여받기 전에 오프라인 평가, 시뮬레이션(Simulation), 통합 테스트(Integration Testing), 적절한 단계별 검증(Staged Validation)을 통과해야 한다. A/B 테스트를 기본적인 안전 실패(Safety Failure)를 처음 발견하기 위한 수단으로 사용해서는 안 된다. 실험 전체 기간 동안 안전 모니터(Safety Monitor), 운영 제한(Operational Limit), 폴백 동작(Fallback Behavior), 사람의 개입 절차(Human Intervention Procedure), 자동 롤백 조건(Automatic Rollback Condition)이 유지되어야 한다.

A/B 테스트는 카나리 배포(Canary Deployment)와 밀접하게 관련되어 있지만 주요 목적은 서로 다르다. 카나리 배포는 새로운 버전을 소규모 집단에 먼저 노출한 후 배포 범위를 확대함으로써 릴리스 위험(Release Risk)을 통제하는 데 중점을 둔다. 반면 A/B 테스트는 서로 다른 대안 사이에서 비교 가능한 증거를 확보하는 데 중점을 둔다. 동일한 인프라를 두 방식 모두에 사용할 수 있지만, 충분한 비교 데이터가 수집되기 전에 성공적으로 동작하는 모든 B 그룹 로봇을 후보 버전으로 자동 전환하기보다 의미 있는 기준 그룹(Baseline Group)을 유지해야 한다.

텔레메트리는 각각의 관측 결과를 해당 결과를 생성한 모델 변형과 연결해야 한다. 로그(Log)에는 모델 버전, 컨테이너 또는 패키지 식별자(Container or Package Identifier), 로봇 하드웨어 프로파일(Hardware Profile), 소프트웨어 구성(Software Configuration), 임무 상황(Mission Context), 사이트 정보, 관련 센서 조건, 타임스탬프(Timestamp)가 포함되어야 한다. 이러한 출처 추적성(Provenance)을 통해 분석가는 실제 모델 효과와 런타임 구성 또는 운영 환경 변화의 영향을 구분할 수 있으며, 이후 조사 과정에서 비정상적인 동작을 재현할 수 있다.

평가 지표(Metric)는 모델 수준과 시스템 수준의 동작을 모두 포함해야 한다. 모델 지표에는 정밀도(Precision), 재현율(Recall), 신뢰도 분포(Confidence Distribution), 불일치율(Disagreement Rate), 작업별 예측 오류(Task-specific Prediction Error)가 포함될 수 있다. 로봇 수준 지표에는 임무 완료(Mission Completion), 개입 빈도(Intervention Frequency), 위치 추정 실패(Localization Failure), 내비게이션 복구(Navigation Recovery), 사이클 시간(Cycle Time), 에너지 소비(Energy Consumption), 안전 이벤트(Safety Event)가 포함될 수 있다. 추론 지연시간, 메모리 소비, GPU 사용률, 온도, 통신 실패와 같은 인프라 지표(Infrastructure Metric)는 향상된 모델 품질이 허용할 수 없는 연산 비용을 발생시키는지 판단하는 데 사용된다.

통계적 해석(Statistical Interpretation)에서는 표본 크기(Sample Size)와 관측치 사이의 의존성(Dependence)에 주의해야 한다. 하나의 로봇에서 수집된 수천 개의 카메라 프레임이 반드시 수천 개의 독립적인 운영 실험을 의미하는 것은 아니다. 연속된 관측 결과 사이에는 강한 상관관계(Correlation)가 존재하기 때문이다. 따라서 실험에서는 데이터에 포함된 로봇 수, 임무 수, 사이트 수, 운영 시간, 환경 조건을 함께 고려해야 한다. 실험 기간을 연장하면 데이터 범위를 확대할 수 있지만, 기간 자체만으로 대표성이 부족한 실험 집단(Experimental Population)의 문제를 해결할 수는 없다.

환경적 불균형(Environmental Imbalance)은 잘못된 결론을 만들어내는 주요 원인 중 하나이다. 모델 A가 주로 주간에 운용되고 모델 B가 야간 임무를 더 많이 수행했다면 직접적인 성능 비교 결과는 모델 품질보다 조명 조건을 반영할 수 있다. 날씨, 사이트 구조(Site Geometry), 페이로드(Payload), 로봇 노후도(Robot Age), 센서 오염(Sensor Contamination), 운영자 행동(Operator Behavior), 교통 밀도(Traffic Density)에서도 유사한 교란 요인(Confounding Factor)이 발생할 수 있다. 따라서 실험 설계와 텔레메트리에서는 분석 과정에서 중요한 상황 변수를 식별할 수 있도록 해야 한다.

운영 실험에는 주요 최적화 목표(Primary Optimization Objective)뿐만 아니라 가드레일 지표(Guardrail Metric)도 포함해야 한다. 인지 모델이 탐지 재현율(Detection Recall)을 향상시키면서 지연시간을 증가시켜 후속 계획(Downstream Planning)을 불안정하게 만들 수 있다. 다른 모델은 임무 시간을 단축하는 대신 훨씬 많은 전력을 소비하거나 추가적인 열 부하(Thermal Load)를 발생시킬 수 있다. 가드레일은 하나의 지표에서 나타난 개선이 로봇 시스템의 다른 영역에서 발생한 성능 저하를 감추지 못하도록 한다.

자동화된 모니터링(Automated Monitoring)은 A와 B 모델이 운용되는 동안 실험 상태(Experiment Health)를 지속적으로 평가할 수 있다. 안전 이벤트, 추론 실패(Inference Failure), 지연시간 급증(Latency Spike), 자원 고갈(Resource Exhaustion), 비정상적인 로봇 동작과 관련된 임계값 위반(Threshold Violation)이 발생하면 실험을 일시 중단하거나 모델 B를 실제 운영에서 제거할 수 있다. 롤백(Rollback)은 사고 발생 시 엔지니어가 이전 환경을 다시 구축하는 방식이 아니라 사전에 검증된 산출물과 구성으로 복귀하는 방식으로 수행해야 한다.

실험 분석(Experiment Analysis)은 결과를 하나의 평균값으로 축소하기보다 불확실성(Uncertainty)을 유지해야 한다. 성능 분포(Performance Distribution), 신뢰구간(Confidence Interval), 하위 그룹 동작(Subgroup Behavior), 희귀 실패(Rare Failure), 사이트별 차이(Site-specific Difference)는 집계 지표(Aggregate Metric)에 의해 감춰진 영향을 보여줄 수 있다. 평균적으로 우수한 후보 모델이라도 특정 환경에서 불균형하게 실패한다면 즉각적인 플릿 전체 적용보다 추가 학습, 표적 검증(Targeted Validation), 제한적인 배포 정책(Restricted Deployment Policy)이 필요할 수 있다.

성공적인 A/B 테스트는 재현 가능한 실험 구성(Reproducible Experiment Configuration)에 의존한다. 할당 정책(Assignment Policy), 모델 버전, 컨테이너 다이제스트(Container Digest), 하드웨어 프로파일, 지표 정의(Metric Definition), 분석 규칙(Analysis Rule), 시작 및 종료 조건, 롤백 임계값(Rollback Threshold)을 실험 정보와 함께 기록해야 한다. 이러한 정보를 모델 레지스트리(Model Registry)와 배포 이력(Deployment History)에 연결하면 개발 단계의 검증 증거, 실제 운영 테스트, 이후의 릴리스 결정 사이에 감사 가능한 관계(Auditable Relationship)를 구축할 수 있다.

플릿 규모(Fleet Scale)에서 A/B 테스트는 지속적인 모델 개선 루프(Continuous Model Improvement Loop)의 일부가 된다. 운영 과정에서 수집된 관측 결과를 통해 어려운 환경과 실패 사례(Failure Case)를 식별하고, 선택된 데이터를 다시 레이블링(Labeling) 및 학습 파이프라인(Training Pipeline)으로 전달할 수 있다. 이후 후보 모델은 오프라인 및 시뮬레이션 검증을 거쳐 자격을 충족하면 다시 통제된 플릿 실험에 투입된다. 이를 통해 로봇 동작에 통제되지 않은 온라인 변경을 허용하지 않으면서 운영 환경을 데이터셋 진화(Dataset Evolution) 및 모델 개발과 연결할 수 있다.

운영 A/B 테스트의 더 큰 목적은 증거 기반 로봇 지능의 진화(Evidence-based Evolution of Robot Intelligence)를 실현하는 것이다. 모델 A는 안정적인 운영 기준(Operational Reference)을 제공하고, 모델 B는 실제 임무, 하드웨어, 환경 전반에서 효과를 측정할 수 있는 통제된 변화를 제공한다. 실험 설계, 안전 게이트(Safety Gate), 텔레메트리, 통계 분석(Statistical Analysis), 관측 가능성(Observability), 출처 추적성, 롤백이 통합되면 로봇 플릿은 운영 통제(Operational Control)를 유지하면서 배포된 AI를 지속적으로 개선할 수 있는 체계적인 검증 환경(Disciplined Validation Environment)이 된다.

## 7.7. Model Rollback Mechanism and Fallback Policy [w/Code]

![](images/image7.png){width="7.268055555555556in" height="7.268055555555556in"}

모델 롤백(Model Rollback)은 새롭게 배포된 AI 모델이 허용할 수 없는 동작을 생성할 때 이를 이전에 검증된 버전(Previously Validated Version)으로 교체하는 통제된 과정이다. 로봇 시스템에서 운영 동작은 전처리(Preprocessing), 런타임 라이브러리(Runtime Library), 구성(Configuration), 컨테이너 이미지(Container Image), 하드웨어별 엔진(Hardware-specific Engine), 인터페이스(Interface)에 의존할 수 있으므로 롤백은 단순히 모델 파일만 복원해서는 안 된다. 따라서 신뢰할 수 있는 롤백 메커니즘(Rollback Mechanism)은 완전히 검증된 배포 상태(Validated Deployment State)를 복원해야 한다.

롤백 기능(Rollback Capability)은 사고가 발생한 이후 추가하는 것이 아니라 새로운 모델이 운영 환경(Production Environment)에 도달하기 전에 설계해야 한다. 모든 배포는 이미 모델 검증(Model Validation), 통합 테스트(Integration Testing), 운영 적격성 검증(Operational Qualification)을 통과한 정상 기준 버전(Known-good Baseline)을 유지해야 한다. 배포 시스템은 모델 버전, 컨테이너 다이제스트(Container Digest), 구성 버전(Configuration Version), 런타임 의존성(Runtime Dependency), 호환 가능한 하드웨어 프로파일(Hardware Profile)을 포함하여 기준 버전을 신속하게 식별하고 복원하는 데 필요한 정보를 유지해야 한다.

롤백 트리거(Rollback Trigger)는 자동화된 모니터링(Automated Monitoring) 또는 운영자의 판단(Human Operational Judgment)에서 발생할 수 있다. 자동 트리거에는 추론 실패(Inference Failure), 과도한 지연시간(Latency), 메모리 고갈(Memory Exhaustion), 비정상적인 GPU 사용률, 작업 지표(Task Metric)의 저하, 반복적인 내비게이션 복구(Navigation Recovery), 안전 관련 이벤트(Safety-related Event) 등이 포함될 수 있다. 자동 지표가 충분히 표현하지 못하는 현장 동작이 발견되면 운영자가 직접 롤백을 시작할 수도 있다. 두 방식 모두 동일하게 통제된 롤백 절차로 연결되어야 한다.

롤백 임계값(Rollback Threshold)은 운영 중 즉흥적으로 결정하는 것이 아니라 배포 정책(Deployment Policy)의 일부로 정의해야 한다. 시스템은 제한적인 지연시간 증가나 일시적인 모델 오류를 허용할 수 있지만, 반복적인 추론 실패 또는 안전 위반(Safety Violation)은 즉각적인 롤백 조건으로 처리할 수 있다. 임계값은 각 지표가 운영에서 가지는 중요도를 반영해야 하며, 단순 경고 조건(Warning Condition)과 배포 중단 또는 이전 버전 자동 복원이 필요한 조건을 구분해야 한다.

폴백 정책(Fallback Policy)은 선호되는 AI 기능을 사용할 수 없거나 신뢰할 수 없을 때 로봇이 어떻게 동작해야 하는지를 정의함으로써 버전 롤백보다 더 넓은 범위를 다룬다. 폴백은 이전에 검증된 모델, 더 단순한 모델, 결정론적 알고리즘(Deterministic Algorithm), 기능 축소(Reduced Functionality), 원격 지원(Remote Assistance), 또는 통제된 안전 상태(Controlled Safe State)를 사용할 수 있다. 적절한 대응 방식은 문제가 발생한 AI 구성요소가 인지(Perception), 계획(Planning), 조작(Manipulation), 언어 상호작용(Language Interaction) 또는 기타 어떤 기능을 지원하는지와 각각의 운영 결과에 따라 달라진다.

로봇이 축소된 기능으로 안전하게 계속 운용될 수 있다면 점진적 성능 저하(Graceful Degradation)가 바람직하다. 예를 들어 연산량이 많은 인지 모델이 실패하면 정확도는 낮지만 예측 가능한 지연시간을 제공하는 더 단순한 탐지기(Detector)로 전환할 수 있다. 반면 성능이 저하된 상태로 계속 운용하는 것이 적절하지 않은 상황에서는 해당 임무를 중단하거나, 정의된 안전 위치(Safe Location)로 이동하거나, 사람의 개입(Human Intervention)을 요청해야 한다. 따라서 폴백 정책은 시스템 수준의 안전 요구사항(System-level Safety Requirement)과 연결되어야 한다.

롤백과 폴백은 서로 분리되어 있지만 상호 연계되는 메커니즘으로 다루어야 한다. 롤백은 배포된 소프트웨어 또는 모델 상태를 이전에 검증된 버전으로 변경하는 것이며, 폴백은 정상적인 운용을 신뢰성 있게 지속할 수 없을 때 런타임 동작(Runtime Behavior)을 변경하는 것이다. 로봇은 심각한 문제를 감지한 직후 폴백 상태로 진입하고, 플릿 관리 시스템(Fleet Management System)이 롤백을 수행하는 동안 해당 상태를 유지할 수 있다. 복원된 배포가 필요한 상태 점검(Health Check)을 통과한 이후에만 정상 운용을 재개해야 한다.

상태 점검(Health Check)은 롤백 이전과 이후 모두에서 필수적이다. 배포 전에는 대상 로봇이 호환 가능한 하드웨어, 충분한 자원, 올바른 의존성, 필요한 산출물에 대한 접근 권한을 가지고 있는지 확인한다. 롤백 이후에는 모델이 정상적으로 로딩되고, 추론 요청이 성공하며, 필요한 센서를 사용할 수 있고, ROS2 인터페이스가 정상적으로 동작하며, 주요 성능 지표(Key Performance Indicator)가 허용 가능한 범위로 복귀했는지 확인한 이후 완전한 임무 수행 권한(Full Mission Authority)을 복원해야 한다.

산출물 불변성(Artifact Immutability)은 롤백의 신뢰성을 크게 높인다. 이전 릴리스(Previous Release)는 사고 발생 중 소스 코드에서 이전 버전을 다시 빌드하는 대신 정확한 모델 산출물(Model Artifact), 컨테이너 다이제스트, 구성 스냅샷(Configuration Snapshot), 하드웨어별 런타임 패키지(Runtime Package)를 사용하여 복원해야 한다. 다시 빌드하면 변경된 의존성이나 툴체인(Toolchain)이 포함될 수 있지만, 변경 불가능한 산출물(Immutable Artifact)을 사용하면 이전에 검증된 정확한 소프트웨어 상태로 복귀할 수 있다.

모델 레지스트리(Model Registry)와 컨테이너 레지스트리(Container Registry)는 이러한 변경 불가능한 배포 상태를 유지하기 위한 기반을 제공한다. 승인된 각 릴리스는 모델 버전을 ONNX 표현, TensorRT 엔진 또는 기타 최적화 산출물(Optimized Artifact), 컨테이너 이미지, 전처리 구성(Preprocessing Configuration), 검증 결과(Validation Result), 대상 하드웨어 프로파일과 연결할 수 있다. 이를 통해 롤백은 긴급한 파일 교체 작업이 아니라 레지스트리를 통해 통제되는 알려진 배포 상태 간 전환이 된다.

하드웨어별 엔진은 추가적인 롤백 제약조건(Rollback Constraint)을 발생시킨다. 특정 GPU 환경을 대상으로 빌드된 TensorRT 엔진은 다른 로봇 구성에 적합하지 않을 수 있으며, 전용 가속기 바이너리(Dedicated Accelerator Binary)는 특정 컴파일러 또는 런타임 버전에 의존할 수 있다. 따라서 롤백 시스템은 하나의 이전 모델 패키지가 이기종 플릿(Heterogeneous Fleet) 전체에서 공통으로 유효하다고 가정하지 않고 각각의 로봇과 호환되는 정상 산출물(Known-good Artifact)을 선택해야 한다.

플릿 수준 롤백(Fleet-level Rollback)에서는 통제된 대상 선정(Controlled Targeting)이 필요하다. 장애는 특정 모델 버전을 사용하는 모든 로봇에 영향을 줄 수도 있지만, 하나의 하드웨어 세대(Hardware Generation), 특정 사이트, 또는 특정 환경 조건에서 운용되는 일부 로봇에만 영향을 줄 수도 있다. 배포 인프라는 모델 버전, 로봇 그룹(Robot Group), 하드웨어 프로파일, 사이트 또는 배포 코호트(Deployment Cohort)를 기준으로 롤백을 수행할 수 있어야 한다. 이를 통해 특정 구성에만 문제가 존재할 때 불필요한 플릿 전체 변경을 방지할 수 있다.

롤백 자체도 많은 로봇의 소프트웨어가 동시에 변경될 경우 운영 위험(Operational Risk)을 발생시킬 수 있다. 따라서 장애가 즉각적으로 안전에 치명적인 문제가 아니라면 단계적 롤백(Staged Rollback)을 적용할 수 있다. 먼저 소규모 그룹에서 정상 버전을 복원하고 상태 검증을 완료한 후 영향을 받은 다른 로봇으로 복원 범위를 확대할 수 있다. 반면 심각한 장애에서는 정책에 따라 즉각적인 폴백을 수행한 후 영향을 받은 전체 배포 그룹에 신속한 롤백을 적용할 수 있다.

모델이 시간적 또는 상태 기반 프로세스(Temporal or Stateful Process)에 참여하는 경우 상태 관리(State Management)가 중요해진다. 내비게이션, 추적(Tracking), 조작 또는 시퀀스 기반 추론(Sequence-based Inference) 도중 모델을 교체하면 내부 상태가 복원된 모델과 일치하지 않을 수 있다. 따라서 롤백 절차에서는 추론 상태(Inference State)를 초기화하고, 관련 ROS2 노드를 재시작하며, 캐시(Cache)를 삭제하고, 추적기(Tracker)를 다시 초기화하거나 임무를 안전하게 재시작해야 할 수 있다. 따라서 모델 교체는 애플리케이션 수명주기 관리(Application Lifecycle Management)와 조정되어야 한다.

관측 가능성(Observability)은 롤백이 성공했는지를 판단하는 데 필요한 증거를 제공한다. 모니터링 시스템은 추론 지연시간, 오류율(Error Rate), 자원 소비(Resource Consumption), 작업 성능(Task Performance), 안전 이벤트, 로봇 수준 운영 지표를 사용하여 장애 이전, 장애 발생 시점, 롤백 이후의 상태를 비교해야 한다. 로그에는 트리거, 영향을 받은 모델, 롤백 산출물, 타임스탬프(Timestamp), 로봇 식별 정보, 구성 변경, 상태 점검 결과, 최종 복구 상태(Recovery Status)를 기록하여 이후 분석에 활용해야 한다.

롤백 이벤트(Rollback Event)는 더 광범위한 MLOps 학습 과정에도 반영되어야 한다. 실패한 배포는 데이터셋 공백(Dataset Gap), 분포 변화(Distribution Shift), 최적화 문제(Optimization Problem), 런타임 비호환성(Runtime Incompatibility), 부족한 검증 시나리오 또는 배포 정책의 약점을 드러낼 수 있다. 따라서 사고 데이터(Incident Data)는 단순한 운영 중단으로 처리하는 것이 아니라 근본 원인 분석(Root-cause Analysis), 데이터셋 큐레이션(Dataset Curation), 재학습(Retraining), 시뮬레이션, 회귀 테스트(Regression Testing), 향후 릴리스 게이트(Release Gate)와 연결해야 한다.

폴백 정책은 실제로 필요해지기 전에 테스트해야 한다. 시뮬레이션(Simulation), 소프트웨어 인 더 루프(Software-in-the-loop, SIL), 하드웨어 인 더 루프(Hardware-in-the-loop, HIL), 통제된 실제 로봇 테스트를 통해 장애 감지(Failure Detection), 성능 저하 모드(Degraded Mode), 모델 전환(Model Switching), 노드 재시작(Node Restart), 안전 정지(Safe Stopping), 복구 절차(Recovery Procedure)가 의도한 대로 동작하는지 검증할 수 있다. 실제 장애는 항상 명확한 모델 실패 형태로 발생하지 않으므로 부분 장애(Partial Failure)와 통신 손실(Communication Loss)도 테스트에 포함해야 한다.

성숙한 로봇 배포 아키텍처(Robot Deployment Architecture)는 롤백을 예외적인 비상 절차가 아니라 정상적인 릴리스 엔지니어링(Release Engineering)의 일부로 다룬다. 모든 모델 릴리스는 정상 상태가 검증된 이전 버전(Known-good Predecessor), 변경 불가능한 산출물, 정의된 트리거, 호환 가능한 폴백 동작, 상태 점검, 관측 가능성, 복구 증거(Recovery Evidence)를 갖추어야 한다. 이러한 구조를 통해 새로운 AI 기능을 도입하면서도 이전에 검증된 운영 상태로 결정론적으로 복귀할 수 있는 경로를 유지할 수 있다.

롤백 및 폴백 정책의 더 큰 목적은 운영 복원력(Operational Resilience)을 확보하는 것이다. 로봇 지능(Robot Intelligence)은 반복적인 모델 업데이트를 통해 지속적으로 발전하지만, 모든 업데이트는 현실 세계와 상호작용하는 물리 시스템에 새로운 불확실성을 도입한다. 장애 감지, 폴백, 변경 불가능한 배포 상태, 자동 롤백(Automated Rollback), 검증, 플릿 대상 선정(Fleet Targeting), 관측 가능성, 사고 학습(Incident Learning)을 통합함으로써 로봇 MLOps는 통제 가능하고 복구 가능한 운영을 유지하면서 지속적인 AI 개선을 지원할 수 있다.

## 7.8. Multi Model Ensemble Deployment on Robot Edge [w/Code]

![](images/image8.png){width="7.268055555555556in" height="7.268055555555556in"}

다중 모델 앙상블 배포(Multi-model Ensemble Deployment)는 로봇 엣지 컴퓨팅 환경(Robot Edge Computing Environment)에서 여러 AI 모델의 예측 또는 표현(Representation)을 결합하는 방식이다. 모든 운영 조건에서 하나의 신경망에만 의존하는 대신, 서로 다른 센서, 환경, 객체 클래스(Object Class), 불확실성 영역(Uncertainty Regime)에 특화된 모델들의 상호 보완적인 강점을 활용할 수 있다. 목적은 단순히 더 많은 모델을 실행하는 것이 아니라 엄격한 지연시간(Latency), 메모리, 전력, 열 제약(Thermal Constraint) 안에서 강건성(Robustness)과 의사결정 품질(Decision Quality)을 향상시키는 것이다.

앙상블 아키텍처(Ensemble Architecture)는 개별 모델이 최종 결과에 어떻게 기여할 것인지를 정의하는 것에서 시작한다. 여러 모델이 동일한 센서 입력(Sensor Input)을 독립적으로 처리한 후 예측 결과를 결합할 수도 있고, 서로 다른 모델이 RGB 카메라, 깊이 센서(Depth Sensor), 라이다(LiDAR), 레이더(Radar), 오디오(Audio)와 같은 개별 모달리티(Modality)를 처리할 수도 있다. 출력은 작업 아키텍처와 엣지에서 허용 가능한 연산 결합도(Computational Coupling)에 따라 특징 수준(Feature Level), 예측 수준(Prediction Level), 의사결정 수준(Decision Level)에서 융합할 수 있다.

예측 수준 앙상블(Prediction-level Ensemble)은 각 모델이 독립적인 출력을 생성하고 이를 평균화(Averaging), 투표(Voting), 신뢰도 가중(Confidence Weighting), 학습 기반 융합(Learned Fusion)을 통해 결합할 수 있기 때문에 비교적 단순하다. 분류(Classification)에서는 여러 모델의 확률을 통합한 후 최종 클래스를 선택할 수 있다. 객체 탐지(Object Detection)에서는 중첩된 탐지 결과에 대해 신뢰도를 조정하고 공간적 매칭(Spatial Matching)을 수행해야 할 수 있다. 융합 방법은 배포된 추론 파이프라인(Inference Pipeline)의 일부이므로 모델과 함께 검증하고 버전 관리해야 한다.

특징 수준 융합(Feature-level Fusion)은 여러 네트워크 또는 센서 모달리티의 중간 표현(Intermediate Representation)을 결합함으로써 더욱 풍부한 관계를 포착할 수 있다. 카메라 모델은 의미론적 외형 정보(Semantic Appearance Information)를 제공하고 라이다 네트워크는 기하학적 구조(Geometric Structure)를 제공할 수 있다. 이러한 표현은 후속 예측(Downstream Prediction) 이전에 융합될 수 있다. 이 방식은 멀티모달 추론(Multimodal Reasoning)을 향상시킬 수 있지만 독립적인 예측 수준 앙상블보다 모델 아키텍처, 텐서 차원(Tensor Dimension), 동기화(Synchronization), 전처리(Preprocessing), 런타임 실행(Runtime Execution) 사이에 더 강한 의존성을 형성한다.

의사결정 수준 융합(Decision-level Fusion)은 실제 로봇 동작에 더 가까운 단계에서 이루어진다. 서로 분리된 인지 또는 추론 모듈(Reasoning Module)이 독립적으로 가설(Hypothesis)을 생성하고 이를 상위 의사결정 계층(Supervisory Decision Layer)에서 조정할 수 있다. 예를 들어 시각 인지(Visual Perception), 깊이 추정(Depth Estimation), 장애물 탐지(Obstacle Detection)는 하나의 신경망으로 통합되지 않더라도 내비게이션 의사결정에 각각 증거를 제공할 수 있다. 이러한 아키텍처는 모듈성(Modularity)을 유지하고 장애를 쉽게 격리할 수 있지만, 의사결정 정책은 모델 간 불일치와 출력 누락을 명시적으로 처리해야 한다.

모델 다양성(Model Diversity)은 유용한 앙상블을 구성하는 핵심 요소이다. 거의 동일한 여러 모델을 실행하면 의미 있는 강건성 향상 없이 연산 비용만 증가할 수 있다. 다양성은 서로 다른 아키텍처, 학습 데이터셋, 초기화(Initialization), 센서 모달리티, 최적화 목표(Optimization Objective), 특화된 운영 도메인(Operating Domain)에서 발생할 수 있다. 플릿 모델(Fleet Model)은 일반 모델이 광범위한 환경에서 안정적인 기준을 제공하는 동시에 실내, 실외, 저조도(Low-light), 혼잡 환경 또는 비정상적인 환경에 특화된 전문 모델(Specialist Model)을 포함할 수도 있다.

정적 앙상블(Static Ensemble)은 모든 관련 입력에 대해 사전에 정의된 모델 집합을 실행한다. 추론 경로(Inference Path)가 일정하게 유지되기 때문에 동작을 비교적 예측하기 쉽고 검증도 단순화할 수 있다. 그러나 여러 신경망을 지속적으로 실행하면 상당한 GPU 메모리, 가속기 용량(Accelerator Capacity), 에너지, 열 예산(Thermal Budget)을 소비할 수 있다. 따라서 정적 앙상블은 결합된 워크로드가 짧은 벤치마크 조건에서만 동작하는 것이 아니라 로봇의 지속 가능한 연산 범위(Sustained Compute Envelope) 안에 충분히 들어오는 경우에 가장 실용적이다.

동적 앙상블(Dynamic Ensemble)은 런타임 상황(Runtime Context)에 따라 모델을 활성화한다. 일반적인 장면에서는 경량 모델(Lightweight Model)을 실행하고, 불확실성이 증가하거나 특정 환경 조건이 감지될 때 특화 모델 또는 연산량이 많은 모델을 호출할 수 있다. 상황 정보에는 신뢰도(Confidence), 조명, 로봇 위치, 임무 유형(Mission Type), 센서 상태(Sensor Health), 감지된 장면 복잡도(Scene Complexity)가 포함될 수 있다. 조건부 실행(Conditional Execution)은 평균 연산 비용을 줄일 수 있지만 라우팅 로직(Routing Logic) 자체가 테스트와 모니터링이 필요한 핵심 구성요소가 된다.

계단식 추론(Cascaded Inference)은 유용한 동적 배포 방식 중 하나이다. 연산 비용이 낮은 1단계 모델(First-stage Model)이 쉬운 사례를 처리하고, 모호한 입력은 더 강력한 2단계 모델(Second-stage Model)로 전달한다. 이러한 아키텍처는 어려운 관측에 대해 높은 성능을 유지하면서 평균 지연시간과 전력 소비를 줄일 수 있다. 계단식 구조의 효과는 신뢰할 수 있는 신뢰도 추정(Confidence Estimation)에 달려 있는데, 1단계 모델이 잘못된 결과에 과도하게 높은 신뢰도를 부여하면 어렵거나 잘못된 예측이 더 강력한 모델로 전달되지 않을 수 있기 때문이다.

모델 수가 증가할수록 엣지 자원 스케줄링(Edge Resource Scheduling)의 중요성도 증가한다. 여러 신경망은 GPU 연산, 가속기 메모리(Accelerator Memory), CPU 전처리, 메모리 대역폭(Memory Bandwidth), 센서 버퍼(Sensor Buffer)를 두고 경쟁할 수 있다. 각각의 모델을 개별적으로 벤치마킹했을 때 문제가 없더라도 모든 모델을 동시에 실행하면 지연시간 급증(Latency Spike)이 발생할 수 있다. 따라서 배포 설계에서는 각 로봇 기능의 시간 요구사항에 따라 순차 실행(Sequential Execution), 통제된 동시성(Controlled Concurrency), 모델 우선순위(Model Priority), 가속기 분할(Accelerator Partitioning), 자원 예약(Resource Reservation)을 고려해야 한다.

메모리 관리(Memory Management)는 순수한 연산 성능만큼 중요해질 수 있다. 여러 최적화 엔진(Optimized Engine)이 각각 장치 메모리에 들어갈 수 있더라도 ROS2, 매핑(Mapping), 내비게이션(Navigation), 시각화(Visualization) 워크로드와 동시에 로딩하면 전체 용량을 초과할 수 있다. 배포 시스템은 핵심 모델을 메모리에 상주시킨 상태에서 전문 모델을 필요할 때만 로딩하거나, 서로 다른 모델을 별도의 가속기에 할당할 수 있다. 이 경우 모델 로딩 지연시간(Model Loading Latency)도 성능 측정에서 제외하지 않고 시스템 수준의 시간 분석(System-level Timing Analysis)에 포함해야 한다.

이기종 로봇 엣지 플랫폼(Heterogeneous Robot Edge Platform)은 앙상블 구성요소를 서로 다른 컴퓨팅 장치에 분산할 수 있다. Jetson GPU는 유연한 CUDA 또는 TensorRT 워크로드를 실행하고, 전용 신경망 가속기(Dedicated Neural Accelerator)는 호환 가능한 고주파 인지 모델(High-frequency Perception Model)을 처리하며, CPU는 경량의 결정론적 로직(Deterministic Logic)을 실행할 수 있다. 이러한 분할은 효율성을 향상시킬 수 있지만 장치 사이에서 지나치게 많은 텐서를 이동하면 통신, 동기화, 메모리 복사 오버헤드로 인해 병렬 가속(Parallel Acceleration)의 장점이 사라질 수 있다.

센서 동기화(Sensor Synchronization)는 멀티모달 앙상블(Multimodal Ensemble)에서 특히 중요하다. 카메라, 라이다, 레이더 및 기타 센서의 관측값은 서로 다른 주기와 타임스탬프(Timestamp)로 입력될 수 있다. 시간적으로 제대로 정렬되지 않은 데이터를 융합하면 각각의 모델이 정확하게 동작하더라도 잘못된 결론을 생성할 수 있다. 따라서 배포 파이프라인은 타임스탬프 처리, 버퍼링(Buffering), 동기화 허용 오차(Synchronization Tolerance), 데이터 누락 시 동작(Missing-data Behavior), 오래된 입력 정책(Stale-input Policy)을 앙상블 운영 사양(Operational Specification)의 일부로 정의해야 한다.

불확실성(Uncertainty)과 불일치(Disagreement)는 앙상블 제어에 유용한 신호를 제공한다. 여러 모델이 강하게 일치하면 시스템은 더 높은 신뢰도로 예측을 수용하거나 추가적인 연산 호출을 생략할 수 있다. 반대로 상당한 불일치가 발생하면 다른 모델을 추가로 실행하거나, 추가 센서 증거를 요청하거나, 로봇 속도를 낮추거나, 보수적인 의사결정 정책(Conservative Decision Policy)으로 전환할 수 있다. 불일치를 자동으로 장애로 해석해서는 안 되지만, 현재 입력이 모델의 성능 한계(Capability Boundary)에 근접했음을 나타내는 관측 가능한 지표로 활용할 수 있다.

장애 격리(Failure Isolation)는 모듈형 앙상블(Modular Ensemble)의 또 다른 장점이다. 런타임 장애(Runtime Failure), 센서 손실(Sensor Loss), 가속기 오류(Accelerator Error), 호환되지 않는 입력으로 인해 하나의 모델을 사용할 수 없게 되더라도 안전 요구사항이 허용한다면 축소된 모델 집합으로 시스템을 계속 운영할 수 있다. 앙상블은 어떤 구성요소가 필수(Mandatory)이고 어떤 구성요소가 선택적(Optional)이며 어떤 폴백 조합(Fallback Combination)이 검증되어 있는지를 명시적으로 정의해야 한다. 따라서 점진적 성능 저하(Graceful Degradation)는 장애 발생 시 우연히 이루어지는 것이 아니라 사전에 설계하고 검증해야 한다.

앙상블 패키징(Ensemble Packaging)은 단일 모델 패키징보다 더 강력한 의존성 관리(Dependency Management)를 요구한다. 각 구성요소에는 모델 버전, 최적화 산출물(Optimized Artifact), 전처리 구성, 런타임 요구사항, 하드웨어 호환성 프로파일(Hardware Compatibility Profile), 검증 이력(Validation History)이 존재한다. 또한 앙상블에는 융합 로직(Fusion Logic), 라우팅 규칙(Routing Rule), 동기화 정책, 자원 할당(Resource Allocation), 폴백 구성이 필요하다. 전체 운영 상태를 재현할 수 있도록 이러한 관계를 하나의 버전 관리된 배포 매니페스트(Versioned Deployment Manifest)로 표현해야 한다.

관측 가능성(Observability)은 개별 모델의 동작과 앙상블 수준의 동작을 모두 포착해야 한다. 측정 지표에는 모델별 지연시간, 메모리 사용량, 신뢰도, 호출 빈도(Invocation Frequency), 불일치율(Disagreement Rate), 라우팅 결정(Routing Decision), 융합 결과(Fusion Outcome), 가속기 사용률, 최종 작업 성능(Task Performance)이 포함될 수 있다. 이러한 측정을 통해 이론적으로 더 강력한 앙상블이 실제 로봇 운영을 향상시키는지, 아니면 의미 있는 시스템 수준의 이점 없이 연산 복잡성과 자원 경합(Resource Contention)만 증가시키는지를 판단할 수 있다.

따라서 검증(Validation)은 개별 모델에서 완전히 통합된 앙상블로 단계적으로 확장되어야 한다. 오프라인 테스트(Offline Testing)는 모델 품질과 융합 동작을 검증하고, 시뮬레이션은 더 광범위한 시나리오에서 상호작용을 평가하며, 하드웨어 테스트(Hardware Testing)는 대상 엣지 플랫폼에서 시간 및 자원 영향을 측정한다. 이후 운영 배포에서는 전체 플릿 활성화 이전에 섀도(Shadow), 카나리(Canary), 통제된 A/B 전략을 사용할 수 있다. 롤백(Rollback)은 하나의 구성 모델만 복원하는 것이 아니라 전체 앙상블 구성을 복원해야 한다.

다중 모델 앙상블 배포는 궁극적으로 로봇 엣지를 단일 모델 추론 장치(Single-model Inference Device)에서 협력형 지능 플랫폼(Coordinated Intelligence Platform)으로 전환한다. 여러 특화 모델, 센서, 런타임(Runtime), 가속기가 협력하여 강건성과 적응성(Adaptability)을 향상시킬 수 있지만, 구성요소가 추가될 때마다 오케스트레이션 복잡성(Orchestration Complexity)도 증가한다. 따라서 효과적인 로봇 MLOps는 앙상블 다양성과 자원 스케줄링, 동기화, 불확실성 처리, 관측 가능성, 재현 가능한 패키징(Reproducible Packaging), 검증, 폴백 사이의 균형을 유지하여 추가된 지능이 운영 측면에서도 예측 가능한 상태로 유지되도록 해야 한다.

## 7.9. ROS2 AI Node Hot Swap and Runtime Model Update [w/Code]

![](images/image9.png){width="7.268055555555556in" height="7.268055555555556in"}

ROS2 AI 노드 핫스왑(ROS2 AI Node Hot-swap)은 로봇 소프트웨어 스택(Robotic Software Stack)이 계속 운영되는 동안 추론 모델(Inference Model)을 교체하거나 업데이트할 수 있도록 한다. 새로운 모델을 도입할 때마다 전체 시스템을 중지하는 대신 AI 구성요소를 격리하여 내비게이션(Navigation), 센싱(Sensing), 제어(Control), 임무 관리(Mission Management)와 독립적으로 모델 상태를 변경할 수 있도록 한다. 이러한 기능은 지속적인 가용성(Continuous Availability)을 유지하면서 빈번한 AI 개선을 적용해야 하는 로봇에서 특히 중요하다.

런타임 모델 업데이트(Runtime Model Update)는 단순히 가중치 파일(Weight File)을 교체하는 것보다 더 넓은 개념이다. AI 노드는 전처리 파라미터(Preprocessing Parameter), 라벨 정의(Label Definition), 텐서 형상(Tensor Shape), 추론 엔진(Inference Engine), 보정 데이터(Calibration Data), 신뢰도 임계값(Confidence Threshold), 후처리 로직(Postprocessing Logic), 하드웨어별 최적화 산출물(Hardware-specific Optimization Artifact)에 의존할 수 있다. 따라서 안전한 업데이트 메커니즘은 모델을 버전 관리된 배포 단위(Versioned Deployment Unit)로 취급하고 활성화하기 전에 관련된 모든 산출물이 실행 중인 ROS2 노드와 호환되는지 검증해야 한다.

ROS2 수명주기 관리(ROS2 Lifecycle Management)는 통제된 모델 교체를 위한 유용한 아키텍처 기반을 제공한다. 수명주기를 인식하는 노드(Lifecycle-aware Node)는 지속적으로 실행되는 단일 프로세스로만 동작하는 대신 미구성(Unconfigured), 비활성(Inactive), 활성(Active), 종료(Finalized)와 같은 상태 사이를 전환할 수 있다. 노드가 비활성 상태일 때 모델 자원을 준비하고 활성화 전에 검증하며 정리(Cleanup) 과정에서 자원을 해제할 수 있다. 이러한 명시적인 상태 전환(State Transition)은 런타임 업데이트를 조정하기 위한 예측 가능한 제어 지점을 제공한다.

핫스왑 설계(Hot-swap Design)는 모델 로딩(Model Loading)과 모델 활성화(Model Activation)를 분리해야 한다. 후보 모델(Candidate Model)은 운영 추론 트래픽(Production Inference Traffic)을 즉시 처리하지 않은 상태에서 먼저 다운로드하고 검증한 후 메모리에 로딩하여 초기화할 수 있다. 초기화가 성공하면 시스템은 워밍업 추론(Warm-up Inference)과 호환성 검사(Compatibility Check)를 수행할 수 있다. 이러한 검사를 통과한 이후에만 노드는 활성 추론 참조(Active Inference Reference)를 현재 모델에서 후보 모델로 전환하여 모델 변경 과정에서 발생하는 중단을 줄일 수 있다.

이중 버퍼 모델 아키텍처(Double-buffered Model Architecture)는 모델 전환 지연시간(Switching Latency)을 최소화할 수 있다. 하나의 모델 인스턴스(Model Instance)가 활성 상태를 유지하는 동안 두 번째 슬롯에서 후보 모델을 준비한다. 후보 모델이 준비될 때까지 센서 요청은 기존 활성 인스턴스를 통해 계속 처리되며, 준비가 완료되면 원자적 전환(Atomic Switch) 또는 신중하게 동기화된 참조 전환(Synchronized Reference Switch)을 통해 새로운 요청을 후보 모델로 전달한다. 이전 모델은 GPU 메모리와 기타 자원을 해제하기 전에 신속한 롤백(Rollback)을 위해 일정 시간 유지할 수 있다.

메모리 제약(Memory Constraint)은 임베디드 로봇 컴퓨터(Embedded Robot Computer)에서 이중 버퍼링(Double Buffering)을 어렵게 만들 수 있다. TensorRT 엔진, 신경망 가중치(Neural Network Weight), 활성화 버퍼(Activation Buffer), CUDA 컨텍스트(CUDA Context), 센서 처리 워크로드가 이미 사용 가능한 메모리의 상당 부분을 소비할 수 있다. 두 개의 완전한 모델을 동시에 유지할 수 없다면 업데이트 절차에는 통제된 추론 일시정지(Controlled Inference Pause), 기존 엔진 언로딩(Unloading), 대체 모델 로딩, 상태 검증(Health Verification), 서비스 재개가 필요할 수 있다. 허용 가능한 중단 시간은 로봇 동작과 안전 요구사항에 따라 정의해야 한다.

모델 전환이 시작될 때에도 추론 요청이 실행 중일 수 있으므로 동시성 제어(Concurrency Control)는 필수적이다. 다른 콜백(Callback)이나 작업 스레드(Worker Thread)가 엔진을 사용하고 있는 동안 해당 엔진을 해제하면 장애 또는 상태 손상(State Corruption)이 발생할 수 있다. 따라서 AI 노드는 활성 요청(Active Request)을 추적하고 필요한 경우 새로운 요청의 수신을 중지하거나 드레이닝(Draining)하며, 진행 중인 추론(In-flight Inference)이 완료될 때까지 기다린 후 동기화된 실행 경계(Synchronized Execution Boundary)를 통해 모델 참조를 변경하고 정상적인 요청 처리를 재개해야 한다.

ROS2 콜백 그룹(Callback Group)과 실행기(Executor)는 런타임 업데이트를 얼마나 안전하게 구현할 수 있는지에 영향을 미친다. 실행기 구성(Executor Configuration)에 따라 센서 콜백, 추론 작업자(Inference Worker), 파라미터 변경(Parameter Change), 서비스 요청(Service Request), 모델 관리 작업(Model-management Operation)이 동시에 실행될 수 있다. 아키텍처는 모델 수명주기 작업과 추론 콜백 사이의 경쟁 상태(Race Condition)를 방지하면서 관련 없는 로봇 기능이 불필요하게 차단되지 않도록 해야 한다. 우연한 콜백 실행 순서에 의존하는 것보다 명시적인 동시성 설계(Explicit Concurrency Design)를 적용하는 것이 바람직하다.

업데이트 인터페이스(Update Interface)는 ROS2 서비스(Service), 액션(Action), 파라미터(Parameter) 또는 전용 배포 관리자(Dedicated Deployment Manager)를 통해 제공할 수 있다. 모델 관리 요청(Model-management Request)은 모델 식별자(Model Identifier), 산출물 위치(Artifact Location), 버전, 하드웨어 프로파일(Hardware Profile), 활성화 정책(Activation Policy)을 지정할 수 있다. 파라미터는 경량 구성 변경에 적합하지만, 규모가 큰 모델 전환에서는 준비, 검증, 활성화, 실패, 롤백 상태를 보고할 수 있는 명시적인 트랜잭션 인터페이스(Transactional Interface)를 사용하는 것이 일반적으로 더 적합하다.

산출물 검증(Artifact Verification)은 후보 모델이 활성 상태로 진입하기 전에 수행해야 한다. 노드 또는 배포 관리자는 체크섬(Checksum), 서명(Signature), 매니페스트 정보(Manifest Information), 런타임 호환성(Runtime Compatibility), 예상 텐서 인터페이스(Expected Tensor Interface), 사용 가능한 메모리, 가속기 지원(Accelerator Support), 필요한 전처리 구성을 검증할 수 있다. TensorRT 엔진과 같은 하드웨어별 산출물은 의도된 배포 환경과 일치해야 한다. 다운로드가 성공했다는 사실만으로 배포가 성공했다고 판단해서는 안 된다.

워밍업(Warm-up)은 모델을 로딩한 직후의 최초 추론 요청에서 정상 상태(Steady-state)의 동작을 대표하지 않는 초기화 오버헤드(Initialization Overhead)가 발생할 수 있기 때문에 중요하다. GPU 커널(GPU Kernel), 메모리 할당(Memory Allocation), 런타임 캐시(Runtime Cache), 실행 컨텍스트(Execution Context)는 예측 가능한 성능에 도달하기 전에 초기화가 필요할 수 있다. 트래픽을 전환하기 전에 통제된 워밍업 입력(Controlled Warm-up Input)을 실행하면 기본적인 추론 동작을 검증할 수 있으며 초기화 지연시간이 활성 로봇 제어 루프(Robot Control Loop)에 영향을 미칠 가능성을 줄일 수 있다.

상태 기반 AI 모델(Stateful AI Model)은 핫스왑 과정에서 추가적인 조정이 필요하다. 추적 시스템(Tracking System), 순환 신경망(Recurrent Network), 시간적 월드 모델(Temporal World Model), 시퀀스 기반 정책(Sequence-based Policy), 멀티모달 이력 버퍼(Multimodal History Buffer)는 새로운 모델과 호환되지 않는 상태를 포함할 수 있다. 배포 사양(Deployment Specification)은 이러한 상태를 이전(Migration), 변환(Transformation), 유지(Preservation), 폐기(Discard)할 수 있는지를 결정해야 한다. 호환성을 보장할 수 없다면 정의된 운영 경계(Operational Boundary)에서 영향을 받는 상태를 초기화하는 것이 일반적으로 더 안전하다.

업데이트 경계(Update Boundary)를 선택할 때는 로봇의 물리적 상태(Physical State)도 고려해야 한다. 중요한 기동(Critical Maneuver)을 수행하는 도중 내비게이션 또는 조작 모델(Manipulation Model)을 교체하는 것은 로봇이 정지해 있거나 임무 사이에 있을 때 업데이트하는 것보다 더 위험할 수 있다. 따라서 배포 관리자는 소프트웨어 준비 상태(Software Readiness)와 운영 조건(Operational Condition)을 결합하여 모델 준비는 언제든 수행하되 로봇이 승인된 안전 업데이트 지점(Safe Update Point)에 도달할 때까지 활성화를 지연시킬 수 있다.

런타임 검증(Runtime Validation)은 모델 활성화 직후에도 계속되어야 한다. 시스템은 추론 성공 여부, 지연시간 분포(Latency Distribution), GPU 메모리, 신뢰도 동작(Confidence Behavior), 출력 유효성(Output Validity), 센서 연결 상태(Sensor Connectivity), ROS2 토픽 상태(Topic Health), 애플리케이션 수준 성능(Application-level Performance)을 모니터링할 수 있다. 새롭게 활성화된 모델이 정상적으로 로딩되더라도 실제 입력에서는 잘못 동작할 수 있다. 따라서 더욱 엄격한 상태 기준(Health Criteria)을 적용하는 짧은 관찰 구간(Observation Window)을 활성화 이후 검증 단계로 사용할 수 있다.

롤백(Rollback)은 별도의 비상 절차로 구현하기보다 핫스왑 메커니즘에 직접 통합해야 한다. 활성화 이후의 검사(Post-activation Check)가 실패하면 노드는 이전의 검증된 정상 모델(Known-good Model)을 다시 활성화하거나 검증된 폴백 구성(Fallback Configuration)으로 전환할 수 있어야 한다. 후보 모델이 안정적으로 동작한다는 사실이 확인될 때까지 이전 산출물과 호환 가능한 구성을 유지하면 복구 시간(Recovery Time)을 크게 단축할 수 있다.

분산 로봇 시스템(Distributed Robot System)에는 모델을 일관성 있게 업데이트해야 하는 여러 AI 노드가 포함될 수 있다. 인지(Perception), 추적(Tracking), 위치추정(Localization), 계획(Planning), 조작(Manipulation) 모델은 인터페이스 또는 의미론적 출력(Semantic Output)을 통해 서로 호환성 의존성(Compatibility Dependency)을 가질 수 있다. 각각의 모델이 개별적으로 정상적으로 동작하더라도 하나의 구성요소만 업데이트하면 유효하지 않은 조합이 만들어질 수 있다. 배포 매니페스트(Deployment Manifest)는 호환 가능한 모델 집합을 정의하고 조정기(Coordinator)가 순차적 또는 동기화된 다중 노드 전환(Multi-node Transition)을 수행하도록 할 수 있다.

플릿 규모의 런타임 업데이트(Fleet-scale Runtime Update)는 추가적인 오케스트레이션 계층(Orchestration Layer)을 필요로 한다. 배포 서비스(Deployment Service)는 선택된 로봇에 후보 패키지를 배포하고 로컬에서 준비한 다음 소규모 코호트(Cohort)에서 활성화하고 운영 텔레메트리(Operational Telemetry)를 관찰한 후 배포 범위를 확대할 수 있다. 각 로봇은 활성 모델 버전, 준비 상태, 업데이트 결과, 하드웨어 프로파일, 상태 정보(Health Status), 롤백 결과를 보고하여 플릿 제어기(Fleet Controller)가 정확한 배포 상태를 유지할 수 있도록 해야 한다.

관측 가능성(Observability)과 감사 가능성(Auditability)은 런타임 모델 관리(Runtime Model Management)의 핵심 요소이다. 로그에는 이전 모델과 새로운 모델의 식별자, 산출물 버전, 업데이트 요청자(Update Requester), 준비 결과, 활성화 타임스탬프(Activation Timestamp), 수명주기 상태 전환(Lifecycle Transition), 검증 지표(Validation Metric), 장애, 롤백 이벤트를 기록해야 한다. 이러한 기록은 런타임 동작을 실제 로봇에서 실행된 정확한 AI 구성과 연결하고 디버깅(Debugging), 사고 분석(Incident Analysis), 향후 모델 개선을 위한 근거를 제공한다.

런타임 모델 교체는 전체 소프트웨어 스택을 다시 설치하지 않고도 로봇 지능을 변경할 수 있는 메커니즘을 제공하므로 업데이트 경로(Update Path)에 보안(Security)도 포함해야 한다. 업데이트 권한 부여(Update Authorization), 산출물 무결성 검증(Artifact Integrity Verification), 신뢰할 수 있는 레지스트리(Trusted Registry), 보안 전송(Secure Transport), 서명된 패키지(Signed Package), 제한된 모델 관리 인터페이스를 통해 승인되지 않았거나 손상된 모델이 활성화되는 것을 방지할 수 있다. 신속한 업데이트를 가능하게 하는 편의성이 배포 거버넌스(Deployment Governance)를 우회하는 수단이 되어서는 안 된다.

성숙한 ROS2 핫스왑 아키텍처(ROS2 Hot-swap Architecture)는 모델 교체를 단순한 파일 복사 작업이 아니라 통제된 상태 전환(Controlled State Transition)으로 다룬다. 산출물 준비(Artifact Preparation), 수명주기 조정(Lifecycle Coordination), 동시성 제어, 워밍업, 호환성 검증, 안전한 활성화 경계(Safe Activation Boundary), 상태 모니터링(Health Monitoring), 롤백, 보안, 플릿 관측 가능성이 함께 작동하여 지속적인 AI 발전(Continuous AI Evolution)을 가능하게 한다. 그 결과 로봇 플랫폼은 예측 가능하고 복구 가능한 운영(Predictable and Recoverable Operation)을 유지하면서 런타임에서 향상된 지능을 적용할 수 있다.

## 7.10. Model Deployment Latency and Throughput Benchmarking

![](images/image10.png){width="7.268055555555556in" height="7.268055555555556in"}

모델 배포 벤치마킹(Model Deployment Benchmarking)은 AI 모델이 변환, 최적화, 패키징되고 대상 런타임(Target Runtime)에 통합된 이후 로봇의 시간적 요구사항과 연산 요구사항을 충족할 수 있는지를 판단한다. 오프라인 정확도(Offline Accuracy)만으로는 배포 준비 상태(Deployment Readiness)를 판단할 수 없다. 동일한 모델이라도 GPU, 엣지 가속기(Edge Accelerator), 정밀도 모드(Precision Mode), 런타임 엔진(Runtime Engine), 시스템 부하(System Load)에 따라 실행 특성이 크게 달라질 수 있기 때문이다. 따라서 벤치마킹은 모델 품질(Model Quality)과 운영 가능성(Operational Feasibility)을 연결한다.

지연시간(Latency)은 추론 요청(Inference Request)이 사용 가능한 모델 결과로 변환되는 데 필요한 경과 시간을 의미한다. 단순한 벤치마크에서는 신경망 실행 시간만 측정할 수 있지만, 로봇 배포에서는 일반적으로 더 넓은 정의가 필요하다. 센서 데이터 획득(Sensor Acquisition), 전처리(Preprocessing), 메모리 전송(Memory Transfer), 추론(Inference), 후처리(Postprocessing), 미들웨어 통신(Middleware Communication), 동기화(Synchronization)가 모두 제어 시스템이 경험하는 지연에 영향을 줄 수 있다. 따라서 지연시간 결과를 보고할 때는 측정 경계(Measurement Boundary)를 명확하게 정의해야 한다.

모델 단위 지연시간(Model-only Latency)은 최적화 대안을 비교하는 데 여전히 유용하다. 이는 추론 엔진을 독립적으로 측정하여 아키텍처, 정밀도, 커널 선택(Kernel Selection), 입력 해상도(Input Resolution), 가속기 구성(Accelerator Configuration)의 영향을 확인할 수 있도록 한다. 그러나 수 밀리초 안에 실행되는 모델이라도 이미지 변환, 텐서 준비(Tensor Preparation), GPU 전송, 후처리, ROS2 통신이 포함되면 애플리케이션 지연시간(Application Latency)은 크게 증가할 수 있다. 따라서 모델 수준과 종단간(End-to-end) 측정이 모두 필요하다.

평균 지연시간(Average Latency)은 편리한 요약 지표이지만 시간적 불안정성(Timing Instability)을 감출 수 있다. 로봇 시스템에서는 간헐적으로 발생하는 긴 지연이 인지 업데이트(Perception Update), 위치추정(Localization), 계획(Planning), 제어(Control)를 방해할 수 있기 때문에 지연시간 분포(Latency Distribution)가 더 중요할 때가 많다. 벤치마크에서는 중앙값(Median)과 함께 필요에 따라 p95 또는 p99와 같은 상위 백분위 지연시간(Percentile Latency)을 확인해야 한다. 지속적인 테스트에서는 최대 관측 지연시간(Maximum Observed Latency)도 유용하지만, 그 해석은 테스트 지속시간과 워크로드 범위에 따라 달라진다.

처리량(Throughput)은 배포 시스템이 단위 시간 동안 완료할 수 있는 추론 작업량을 나타내며 일반적으로 초당 프레임(Frames per Second), 초당 요청(Requests per Second), 초당 샘플(Samples per Second)로 표현한다. 높은 처리량은 카메라 스트림(Camera Stream)과 다중 센서 워크로드(Multi-sensor Workload)에서 중요하지만 반드시 낮은 지연시간을 의미하지는 않는다. 배칭(Batching)과 비동기 실행(Asynchronous Execution)은 처리량을 높이는 동시에 개별 요청의 대기시간을 증가시킬 수 있으므로 연산 효율성과 응답시간 사이에 상충관계(Trade-off)가 발생한다.

따라서 배치 크기(Batch Size)는 로봇 벤치마크에서 신중하게 다루어야 한다. 클라우드 추론 시스템(Cloud Inference System)은 가속기 활용률(Accelerator Utilization)을 향상시키기 위해 여러 요청을 더 큰 배치로 축적할 수 있지만, 물리적 로봇은 각각의 센서 관측에 즉각적으로 응답해야 할 수 있다. 따라서 배치 크기 1(Batch Size One)은 실시간 로봇 추론의 중요한 기준선(Baseline)이다. 더 큰 배치는 여러 카메라, 독립적인 로봇, 기록 데이터 또는 즉각적인 응답보다 처리량이 더 중요한 비핵심 워크로드에서 유용할 수 있다.

워밍업 동작(Warm-up Behavior)은 정상 상태 성능(Steady-state Performance)과 분리하여 평가해야 한다. 최초의 추론 호출에는 GPU 컨텍스트 생성(GPU Context Creation), 커널 초기화(Kernel Initialization), 메모리 할당(Memory Allocation), 그래프 최적화(Graph Optimization), 캐시 채우기(Cache Population), 지연 런타임 설정(Lazy Runtime Setup)이 포함될 수 있다. 이러한 요청을 정상 상태 평균에 포함하면 일반적인 성능을 왜곡할 수 있지만 완전히 무시하면 모델 핫스왑(Model Hot-swap)이나 복구 과정에서 중요한 시작 지연(Startup Delay)을 놓칠 수 있다. 따라서 벤치마크 보고서는 초기화, 워밍업, 지속 추론(Sustained Inference) 단계를 구분해야 한다.

동기화(Synchronization)는 특히 GPU 실행에서 측정 오류(Measurement Error)를 발생시키는 주요 원인이다. 가속기 연산은 CPU에 대해 비동기적으로 실행되는 경우가 많기 때문에 호스트 측 함수 반환 시간(Host-side Function Return Time)만 측정하면 실제 추론 시간을 과소평가할 수 있다. 따라서 벤치마크 계측(Benchmark Instrumentation)은 측정 구간이 의도된 장치 워크로드(Device Workload)의 완료 시점까지 포함하도록 해야 한다. 여러 스트림(Stream), 추론 엔진 또는 가속기가 동시에 실행되는 경우에도 동일한 원칙이 적용된다.

전처리와 후처리는 전체 파이프라인 안에서뿐만 아니라 독립적으로도 벤치마킹해야 한다. 이미지 크기 조정(Image Resizing), 정규화(Normalization), 색상 변환(Color Conversion), 포인트 클라우드 변환(Point-cloud Transformation), 비최대 억제(Non-maximum Suppression), 디코딩(Decoding), 좌표 변환(Coordinate Conversion), 추적(Tracking)은 상당한 자원을 소비할 수 있다. 주변 연산을 그대로 둔 채 신경망만 최적화하면 실제 로봇 응답시간은 거의 개선되지 않을 수 있다. 파이프라인 프로파일링(Pipeline Profiling)을 통해 추론이 항상 병목이라고 가정하지 않고 실제 병목(Bottleneck)을 식별해야 한다.

메모리 동작(Memory Behavior)은 지연시간과 처리량과 함께 측정해야 한다. GPU 메모리 소비량, 호스트 메모리(Host Memory), 활성화 버퍼(Activation Buffer), 엔진 작업공간(Engine Workspace), 고정 메모리(Pinned Memory), 임시 메모리 할당(Temporary Allocation)은 원시 추론 속도가 충분하더라도 배포를 제한할 수 있다. 특히 여러 AI 모델이 위치추정, 매핑(Mapping), 계획, 시각화(Visualization), 센서 처리와 함께 동작하는 경우 최대 메모리 사용량(Peak Memory)이 중요하다. 따라서 가능하면 전체 엣지 워크로드(Complete Edge Workload)를 반영하여 벤치마킹해야 한다.

짧은 테스트에서는 열 및 전력 효과(Thermal and Power Effect)가 드러나지 않을 수 있으므로 지속 벤치마킹(Sustained Benchmarking)이 필요하다. 임베디드 컴퓨터(Embedded Computer)는 초기에는 높은 성능을 제공하다가 열 또는 전력 제한에 도달하면 클럭 주파수(Clock Frequency)를 낮출 수 있다. 따라서 로봇 엣지 플랫폼은 충분히 긴 시간 동안 워크로드를 실행하면서 온도, 주파수, 전력 모드(Power Mode), 활용률(Utilization), 스로틀링(Throttling) 동작을 관찰해야 한다. 짧은 순간의 최고 성능보다 안정적으로 지속 가능한 성능이 운영 측면에서 일반적으로 더 중요하다.

동시 워크로드(Concurrent Workload)는 실험실 모델 벤치마크와 실제 로봇 성능을 구분하는 또 다른 핵심 요소이다. 운영 중 하나의 AI 모델이 전체 프로세서 또는 GPU를 독점하는 경우는 드물다. ROS2 노드, 이미지 파이프라인(Image Pipeline), 위치추정, SLAM, 내비게이션, 로깅(Logging), 시각화, 추가 모델이 CPU 사이클, GPU 실행시간, 메모리 대역폭(Memory Bandwidth), 저장장치 접근(Storage Access)을 두고 경쟁할 수 있다. 대표적인 동시 실행 환경에서 벤치마킹하면 독립적인 모델 테스트로는 확인할 수 없는 자원 경합(Resource Contention)을 발견할 수 있다.

하드웨어별 최적화(Hardware-specific Optimization)는 동일한 측정 정의를 사용하여 평가해야 한다. FP32, FP16, INT8, TensorRT 엔진, ONNX Runtime 구성, 가속기별 바이너리(Accelerator-specific Binary)는 입력 형상(Input Shape), 전처리, 워밍업, 배치 크기, 타이밍 경계(Timing Boundary), 테스트 데이터가 통제된 경우에만 올바르게 비교할 수 있다. 또한 더 빠른 배포가 허용할 수 없는 수치적 또는 작업 수준 성능 저하(Task-level Degradation)를 발생시킨다면 의미가 없으므로 성능 향상은 정확도 검증(Accuracy Validation)과 함께 평가해야 한다.

입력 특성(Input Characteristics)은 런타임 성능에 영향을 줄 수 있다. 동적 텐서 형상(Dynamic Tensor Shape), 다양한 이미지 해상도, 포인트 클라우드 밀도(Point-cloud Density), 탐지된 객체 수, 시퀀스 길이(Sequence Length), 후처리 복잡도(Postprocessing Complexity)에 따라 장면별 실행시간이 달라질 수 있다. 따라서 하나의 합성 텐서(Synthetic Tensor)에 기반한 벤치마크는 실제 운영 환경의 변동성을 과소평가할 수 있다. 대표적인 로봇 데이터를 사용하여 쉬운 상황, 일반적인 상황, 연산 요구량이 높은 상황을 모두 포함해야 한다.

주기적 인지 파이프라인(Periodic Perception Pipeline)의 경우 벤치마크는 추론 시간을 센서 및 애플리케이션 업데이트 주기(Update Rate)와 연결해야 한다. 초당 30프레임으로 동작하는 카메라는 프레임 사이에 약 33밀리초의 시간을 제공하지만, 전체 처리 파이프라인은 이 시간 안에서 데이터 획득, 전처리, 융합(Fusion), 후속 연산(Downstream Computation)을 함께 수행해야 할 수 있다. 명목상의 프레임 주기(Frame Period)를 한 번 충족하는 것만으로는 충분하지 않으며, 시스템은 지속적으로 큐(Queue)가 누적되지 않으면서 요구되는 타이밍을 유지해야 한다.

큐 동작(Queue Behavior)은 배포된 파이프라인이 연속적인 입력 환경에서 안정성을 유지할 수 있는지를 보여준다. 요청이 처리 속도보다 빠르게 들어오면 큐 깊이(Queue Depth)가 증가하고 모든 추론이 결국 완료되더라도 관측 데이터는 점점 오래된 정보가 된다. 큐 길이, 드롭된 프레임(Dropped Frame), 입력 데이터의 경과 시간(Input Age), 처리 속도(Processing Rate)를 모니터링하면 일시적인 지연시간 급증과 지속적인 과부하(Persistent Overload)를 구분할 수 있다. 많은 로봇 애플리케이션에서는 계속 지연되는 백로그(Backlog)를 처리하는 것보다 오래된 관측값을 폐기하는 것이 더 적절할 수 있다.

다중 모델 로봇(Multi-model Robot)은 개별 모델 측정만이 아니라 시스템 수준 벤치마킹(System-level Benchmarking)이 필요하다. 탐지(Detection), 분할(Segmentation), 깊이 추정(Depth Estimation), 추적, 언어 모델(Language Model), 계획 네트워크(Planning Network)는 공유 하드웨어에서 순차적으로 또는 동시에 실행될 수 있다. 이들의 결합된 스케줄링(Combined Scheduling)이 실제 응답시간을 결정한다. 따라서 하나의 모델을 최적화하는 과정에서 더 높은 시간적 중요도를 가진 기능의 성능이 의도하지 않게 저하되지 않도록 대표적인 실행 그래프(Execution Graph)와 우선순위를 평가해야 한다.

ROS2 통합(ROS2 Integration)은 배포 성능 측정에 포함되어야 하는 통신 및 스케줄링 효과를 추가한다. 토픽 전송(Topic Transport), 직렬화(Serialization), 콜백 스케줄링(Callback Scheduling), 실행기 동작(Executor Behavior), 프로세스 경계(Process Boundary), 동기화는 빠른 추론 엔진 주변에서도 추가적인 지연시간 또는 지터(Jitter)를 발생시킬 수 있다. 센서 입력, AI 노드 처리, 결과 발행(Publication), 후속 소비(Downstream Consumption) 단계에 걸쳐 타임스탬프를 측정하면 전체 로봇 애플리케이션이 실제로 경험하는 지연을 더 명확하게 파악할 수 있다.

벤치마크 재현성(Benchmark Reproducibility)을 확보하려면 상세한 구성 기록(Configuration Record)이 필요하다. 결과에는 모델 버전, 최적화 산출물(Optimized Artifact), 정밀도, 입력 크기, 배치 크기, 런타임 및 드라이버 버전, 하드웨어 플랫폼, 전력 모드, 클럭 구성(Clock Configuration), 소프트웨어 환경, 워크로드 구성, 워밍업 정책, 측정 시간, 관련 시스템 조건을 기록해야 한다. 이러한 정보가 없다면 서로 다른 실험이나 배포 대상 사이의 지연시간 또는 처리량 결과를 신뢰성 있게 비교할 수 없다.

궁극적으로 벤치마킹은 독립적인 성능 실험이 아니라 배포 게이트(Deployment Gate)로 발전해야 한다. 후보 모델(Candidate Model)은 플릿 배포(Fleet Release) 이전에 대상 하드웨어에서 정의된 지연시간, 처리량, 메모리, 열 특성, 작업 품질(Task Quality) 기준을 충족하도록 요구할 수 있다. 동일한 지표를 배포 이후에도 지속적으로 모니터링하여 성능 드리프트(Performance Drift) 또는 자원 경합을 감지할 수 있다. 이를 통해 실험실 최적화(Laboratory Optimization), 엣지 검증(Edge Validation), 운영 모니터링(Production Monitoring), 지속적인 모델 개선(Continuous Model Improvement)이 하나의 폐루프(Closed Loop)로 연결된다.
