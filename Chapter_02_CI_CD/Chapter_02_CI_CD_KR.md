**Volume 10 Robot DevOps and MLOps**

# 2. CI/CD

## 2.1. CI CD Principles and Pipeline Architecture for Robotics

![](images/image1.png){width="7.268055555555556in" height="7.268055555555556in"}

지속적 통합(Continuous Integration)과 지속적 전달(Continuous Delivery)은 빈번하게 발생하는 소프트웨어 변경 사항을 검증되고 배포 가능한 로봇 소프트웨어로 전환하기 위한 체계적인 방법을 제공한다. 로보틱스(Robotics)에서 파이프라인(Pipeline)은 애플리케이션 코드뿐만 아니라 ROS 2 패키지, 임베디드 펌웨어(Embedded Firmware), 디바이스 드라이버(Device Driver), AI 모델(AI Model), 구성 파일(Configuration File), 컨테이너(Container), 하드웨어별 종속성(Hardware-specific Dependency)까지 함께 처리해야 한다.

지속적 통합(Continuous Integration, CI)은 개발자가 변경 사항을 공유 저장소(Shared Repository)에 정기적으로 병합하면서 시작된다. 각 변경은 종속성 해결(Dependency Resolution), 컴파일(Compilation), 정적 분석(Static Analysis), 단위 테스트(Unit Test), 통합 테스트(Integration Test), 아티팩트 생성(Artifact Generation) 등의 작업을 자동으로 실행한다. 목적은 호환되지 않는 변경 사항이 시스템 통합 단계까지 누적되기 전에 결함을 조기에 발견하는 것이다.

지속적 전달(Continuous Delivery)은 필요한 품질 게이트(Quality Gate)를 통과한 소프트웨어가 항상 배포 가능한 상태로 유지되도록 이 과정을 확장한다. 지속적 배포(Continuous Deployment)는 여기서 더 나아가 승인된 아티팩트(Artifact)를 대상 환경에 자동으로 배포한다. 로보틱스에서는 물리적 로봇에 소프트웨어를 배포하면 모션(Motion), 액추에이터(Actuator), 센서(Sensor), 통신 인터페이스(Communication Interface), 물리적 안전(Physical Safety)에 영향을 줄 수 있으므로 보다 통제된 방식으로 적용해야 한다.

로보틱스 CI/CD 아키텍처(CI/CD Architecture)는 점차 현실성이 높아지는 일련의 검증 환경(Verification Environment)으로 이해할 수 있다. 소스 변경은 먼저 포매팅(Formatting), 린팅(Linting), 종속성 검증(Dependency Validation), 컴파일과 같은 비교적 저비용 검사를 통과한다. 성공적인 빌드(Build)는 단위 및 통합 테스트를 거쳐 시뮬레이션(Simulation), 시스템 테스트(System Test), 하드웨어 인 더 루프(Hardware-in-the-Loop, HIL) 검증, 스테이징 환경(Staging Environment), 최종적으로 실제 로봇 또는 로봇 플릿(Robot Fleet) 배포 단계로 진행된다.

파이프라인을 통과하는 동안에는 가능한 한 동일한 소프트웨어 아티팩트(Software Artifact)를 유지해야 한다. 테스트용과 운영용 애플리케이션을 별도로 다시 빌드하면 실제 배포된 바이너리(Binary)가 검증된 바이너리와 달라질 가능성이 생긴다. 불변 바이너리(Immutable Binary), 패키지(Package), 펌웨어 이미지(Firmware Image), 컨테이너 이미지(Container Image), AI 모델 아티팩트를 사용하면 체크섬(Checksum), 메타데이터(Metadata), 종속성 정보, 테스트 증거를 유지하면서 검증된 버전을 승격(Promotion)할 수 있다.

로보틱스 파이프라인은 이기종 컴퓨팅 대상(Heterogeneous Computing Target)도 처리해야 한다. 개발 서버는 x86-64 프로세서를 사용하는 반면 실제 로봇은 ARM64 프로세서, Jetson 플랫폼, 마이크로컨트롤러(Microcontroller), 특수 AI 가속기(AI Accelerator)를 사용할 수 있다. 따라서 CI 인프라(Infrastructure)는 각 대상에 재현 가능한 아티팩트를 생성하면서 컴파일러(Compiler), SDK, 라이브러리(Library), 빌드 옵션(Build Option)의 일관성을 유지할 수 있는 네이티브 빌더(Native Builder) 또는 교차 컴파일 환경(Cross-compilation Environment)을 제공해야 한다.

파이프라인 단계는 처리 비용과 피드백 속도에 따라 분리하는 것이 적절하다. 빠른 검사는 단순한 결함을 수분 내에 제거할 수 있으므로 파이프라인 앞부분에 배치한다. 비용이 높은 시뮬레이션, GPU 기반 추론 평가(Inference Evaluation), 하드웨어 테스트(Hardware Test), 장시간 회귀 테스트(Regression Test)는 기본적인 품질 게이트가 통과된 이후 실행할 수 있다. 이러한 순서는 불필요한 인프라 사용을 줄이면서 실제 장비에 도달하기 전에 점진적으로 강한 검증 근거를 확보하게 한다.

로보틱스 테스트(Robotics Testing)는 일반적인 애플리케이션 파이프라인에서는 나타나지 않을 수 있는 상호작용까지 검증해야 한다. ROS 2 노드는 정상적으로 컴파일되더라도 잘못된 메시지 주기(Message Rate)를 사용하거나 시간 제약(Timing Constraint)을 위반하고, 호환되지 않는 서비스 품질(Quality of Service, QoS) 정책을 사용하거나 다른 노드와 연결될 때 불안정한 동작을 보일 수 있다. 따라서 CI는 소스 수준의 정확성뿐만 아니라 인터페이스, 실행 구성(Launch Configuration), 메시지 계약(Message Contract), 시간 조건, 자원 사용량(Resource Consumption), 시스템 수준 동작을 함께 검증해야 한다.

시뮬레이션(Simulation)은 소프트웨어 통합(Software Integration)과 실제 물리 시스템 배포 사이를 연결하는 중요한 단계이다. 파이프라인은 대표적인 로봇 모델과 환경을 자동으로 실행하여 반복 가능한 시나리오에서 내비게이션(Navigation), 인식(Perception), 조작(Manipulation), 제어(Control) 기능을 시험할 수 있다. 시뮬레이션이 모든 하드웨어 특성을 완벽하게 재현할 수는 없지만 확장 가능한 회귀 테스트를 제공하고 제한된 실제 로봇을 테스트에 투입하기 전에 행동 수준의 오류를 발견할 수 있게 한다.

하드웨어 인 더 루프(Hardware-in-the-Loop, HIL)와 로봇 인 더 루프(Robot-in-the-Loop) 테스트는 실제 센서, 제어기(Controller), 액추에이터, 컴퓨팅 플랫폼 또는 완전한 로봇을 파이프라인 후반부 검증 과정에 포함한다. 이러한 테스트는 타이밍(Timing), 전기적 인터페이스(Electrical Interface), 디바이스 드라이버, 가속기 동작, 통신 지연(Communication Latency), 물리 동역학(Physical Dynamics)에서 발생하는 문제를 발견할 수 있다. 하드웨어 자원은 제한적이므로 일반적으로 저비용 자동 검증을 통과한 소프트웨어에 대해서만 CI 시스템이 해당 자원을 할당한다.

배포 아키텍처(Deployment Architecture)는 소프트웨어 릴리스(Software Release)와 소프트웨어 활성화(Software Activation)를 구분해야 한다. 패키지를 로봇에 전송하더라도 즉시 현재 실행 버전으로 활성화할 필요는 없다. 두 작업을 분리하면 유지보수 시간(Maintenance Window), 준비 상태 확인(Readiness Check), 운영자 승인(Operator Approval), 단계적 롤아웃(Staged Rollout), 상호 의존적인 구성 요소의 동기화된 활성화를 지원할 수 있다. 또한 네트워크 연결이 불안정한 로봇에서도 업데이트를 보다 안전하게 준비할 수 있다.

플릿 배포(Fleet Deployment)는 추가적인 제어 계층을 요구한다. 모든 로봇을 동시에 업데이트하는 대신 CD 파이프라인은 개발용 로봇, 내부 시험 장비, 소규모 운영 로봇 그룹, 그리고 점차 더 큰 그룹의 순서로 릴리스를 배포할 수 있다. 각 단계에서 수집된 텔레메트리(Telemetry)와 운영 상태 정보(Operational Health Information)를 기반으로 다음 단계로의 승격 여부를 결정할 수 있으며, 이를 통해 사전 검증 과정에서 발견되지 않은 결함의 운영 영향을 제한할 수 있다.

따라서 롤백 기능(Rollback Capability)은 선택적인 복구 기능이 아니라 파이프라인의 기본 요구사항이다. 로봇은 이전에 검증된 소프트웨어 버전, 구성 세트(Configuration Set), 펌웨어 패키지 또는 컨테이너 이미지를 유지하거나 안정적으로 다시 가져올 수 있어야 한다. 배포 검사에서 시작 실패(Startup Failure), 비정상적인 자원 사용, 동작 성능 저하, 통신 문제가 감지되면 시스템은 추가 배포를 중단하고 검증된 운영 기준 상태(Known Operational Baseline)로 복원할 수 있어야 한다.

구성(Configuration)은 로봇의 동작이 실행 코드만큼이나 파라미터(Parameter)에 크게 좌우되므로 소프트웨어와 함께 버전 관리되어야 한다. 내비게이션 제한값, 제어기 게인(Controller Gain), 센서 보정 참조값(Sensor Calibration Reference), 기능 플래그(Feature Flag), 네트워크 설정, 하드웨어 프로파일(Hardware Profile)은 특정 릴리스와 추적 가능하게 연결되어야 한다. 문서화되지 않은 수동 변경과 구성을 분리하면 재현성이 향상되고 현장 장애가 발생했을 때 정확한 운영 상태를 재구성할 수 있다.

보안(Security)은 CI/CD 아키텍처 전체에서 지속적으로 적용되어야 한다. 소스 저장소(Source Repository), 빌드 러너(Build Runner), 패키지 레지스트리(Package Registry), 서명 키(Signing Key), 자격 증명(Credential), 배포 서비스(Deployment Service)는 모두 로봇 소프트웨어 공급망(Software Supply Chain)의 일부이다. 접근 제어(Access Control), 격리된 러너(Isolated Runner), 보호 브랜치(Protected Branch), 종속성 스캐닝(Dependency Scanning), 아티팩트 서명(Artifact Signing), 비밀정보 관리(Secret Management), 소프트웨어 자재 명세서(Software Bill of Materials, SBOM) 생성 등을 통해 승인되지 않거나 손상된 구성 요소가 실제 로봇까지 전달되는 것을 방지할 수 있다.

추적성(Traceability)은 소스 코드와 실제 운영 동작을 연결한다. 운영 환경의 아티팩트는 소스 리비전(Source Revision), 빌드 환경(Build Environment), 종속성 버전, 테스트 결과, 승인 이력(Approval History), 대상 플랫폼(Target Platform), 배포 기록(Deployment Record)과 연결되어야 한다. 플릿에서 장애가 보고되면 엔지니어는 당시 활성화되어 있던 정확한 소프트웨어와 구성을 확인하고 해당 빌드를 재현하며 검증 결과를 분석하고 동일한 문제가 영향을 줄 수 있는 다른 로봇을 식별할 수 있다.

성숙한 로보틱스 CI/CD 파이프라인은 단순히 코드를 빌드하고 파일을 로봇으로 복사하는 자동화 스크립트(Automation Script)가 아니다. 이는 소스 변경을 신뢰할 수 있는 운영 아티팩트(Operational Artifact)로 단계적으로 전환하는 엔지니어링 제어 시스템(Engineering Control System)이다. 재현 가능한 빌드(Reproducible Build), 계층적 검증(Layered Verification), 시뮬레이션, 하드웨어 테스트, 통제된 플릿 롤아웃, 롤백, 보안, 추적성을 결합함으로써 CI/CD는 대규모 로봇 소프트웨어를 안정적으로 지속 발전시키기 위한 핵심 기반이 된다.

## 2.2. GitHub Actions CI Pipeline for ROS2 Packages [w/Code]

![](images/image2.png){width="7.268055555555556in" height="7.268055555555556in"}

GitHub Actions는 ROS 2 패키지를 위한 지속적 통합(Continuous Integration, CI) 파이프라인을 구현할 수 있는 저장소 통합형 자동화 플랫폼(Repository-integrated Automation Platform)을 제공한다. 로보틱스 프로젝트에서는 각각의 푸시(Push)나 풀 리퀘스트(Pull Request)가 컴파일(Compilation), 종속성 검증(Dependency Validation), 정적 분석(Static Analysis), 테스트를 자동으로 실행할 수 있다. 이를 통해 저장소의 변경 활동을 반복 가능한 검증 프로세스로 전환하고 공유 개발 브랜치에 병합되기 전에 개발자에게 빠른 피드백을 제공한다.

GitHub Actions 파이프라인은 일반적으로 저장소의 \`.github/workflows\` 디렉터리에 저장되는 워크플로 파일(Workflow File)을 통해 정의된다. 워크플로 정의가 ROS 2 소스 코드와 함께 버전 관리되므로 빌드 및 테스트 절차의 변경도 동일한 풀 리퀘스트 프로세스를 통해 검토할 수 있다. 따라서 CI 구성 자체도 수동으로 설정된 빌드 서버에 의존하지 않고 추적 가능하고 재현 가능한 형태로 관리할 수 있다.

워크플로 실행(Workflow Execution)은 \`push\`, \`pull_request\`, 수동 실행(Manual Dispatch), 예약 실행(Scheduled Execution)과 같은 이벤트로 시작된다. 로보틱스 개발팀은 검증 수준에 따라 서로 다른 트리거(Trigger)를 사용할 수 있다. 풀 리퀘스트에서는 빠른 컴파일과 단위 테스트를 실행하고, 메인 브랜치(Main Branch)에 병합된 변경 사항에는 더 광범위한 통합 테스트, 패키지 생성, 컨테이너 빌드(Container Build), 이후 배포 및 하드웨어 검증을 위한 아티팩트(Artifact) 준비 과정을 실행할 수 있다.

각 워크플로는 GitHub 호스팅 러너(GitHub-hosted Runner) 또는 자체 호스팅 러너(Self-hosted Runner)에서 실행되는 하나 이상의 작업(Job)으로 구성된다. 작업에는 소스 코드 체크아웃(Checkout), 운영체제 준비, ROS 2와 프로젝트 종속성 설치, 패키지 빌드, 테스트 실행, 결과 수집 등의 순차적인 단계(Step)가 포함된다. 독립적인 작업은 병렬로 실행할 수 있으며 작업 간 종속성을 설정하면 이전 품질 게이트(Quality Gate)를 통과한 경우에만 비용이 높은 검증 단계를 실행할 수 있다.

ROS 2 프로젝트는 일반적으로 명시적인 패키지 종속성(Package Dependency)을 가진 여러 패키지가 포함된 워크스페이스(Workspace)로 구성된다. 따라서 CI 작업은 컴파일을 시작하기 전에 예상되는 ROS 2 환경을 재현해야 한다. 러너는 필요한 ROS 배포판(ROS Distribution)을 준비하고 외부 종속성을 확보하며 적절한 설정 환경(Setup Environment)을 적용하고 워크스페이스를 구성하여 패키지 검색과 종속성 관계가 개발 및 대상 환경과 일관되게 동작하도록 해야 한다.

종속성 관리(Dependency Management)는 ROS 2 애플리케이션이 운영체제 패키지, ROS 저장소, 서드파티 라이브러리(Third-party Library), 프로젝트별 소스 저장소를 함께 사용하는 경우가 많기 때문에 특히 중요하다. 자동화된 종속성 해결은 개발자 컴퓨터와 CI 러너 사이의 환경 차이를 줄여준다. 주요 버전을 고정하고 빌드 환경을 기록하면 외부 종속성이 예상하지 못하게 변경되어 파이프라인 결과가 달라지는 문제도 줄일 수 있다.

컴파일은 일반적으로 ROS 2 빌드 시스템(Build System)과 워크스페이스 내부의 패키지 메타데이터(Package Metadata)를 사용하여 수행된다. CI 파이프라인은 컴파일러 오류(Compiler Error)와 해결되지 않은 종속성을 즉각적인 실패 조건으로 처리해야 한다. 많은 패키지를 포함하는 저장소에서는 선택적 빌드(Selective Build)로 피드백 시간을 줄이고, 주기적인 전체 워크스페이스 빌드를 통해 서로 독립적으로 보이는 변경 사항이 전체 로봇 소프트웨어 스택에 호환성 문제를 발생시키지 않았는지 확인할 수 있다.

테스트는 성공적인 빌드 직후 수행되어야 한다. 단위 테스트(Unit Test)는 개별 알고리즘과 구성 요소를 검증하고, 통합 테스트(Integration Test)는 ROS 2 노드(Node), 라이브러리, 인터페이스, 서비스 간의 상호작용을 검증한다. 테스트 결과는 기계 판독 가능한 아티팩트(Machine-readable Artifact)로 수집하여 러너 실행이 종료된 이후에도 실패 정보를 확인할 수 있어야 한다. 이를 통해 어떤 패키지, 테스트 사례(Test Case), 통합 경계에서 파이프라인이 실패했는지 개발자가 확인할 수 있다.

ROS 2 CI는 소스 코드의 컴파일 성공 여부보다 더 많은 요소를 검증해야 한다. 정적 분석(Static Analysis), 포매팅 검사(Formatting Check), 린팅(Linting), 패키지 메타데이터 검증, 인터페이스 일관성(Interface Consistency)을 명시적인 품질 게이트로 적용할 수 있다. 이러한 저비용 검사는 계산 비용이 높은 시뮬레이션이나 시스템 수준 검증 자원을 할당하기 전에 기본적인 결함을 제거할 수 있으므로 워크플로 초기에 배치하는 것이 효과적이다.

매트릭스 전략(Matrix Strategy)을 사용하면 전체 파이프라인 정의를 중복하지 않고 동일한 워크플로를 여러 환경에서 검증할 수 있다. 프로젝트는 지원하는 운영체제 버전, ROS 2 배포판, 컴파일러 구성(Compiler Configuration), 프로세서별 빌드 설정을 별도의 매트릭스 작업(Matrix Job)으로 시험할 수 있다. 이는 로봇 소프트웨어가 개발 워크스테이션, 엣지 컴퓨터(Edge Computer), 임베디드 대상(Embedded Target), 여러 세대의 배포 하드웨어와 호환되어야 할 때 유용하다.

캐싱(Caching)은 CI 실행 시간을 크게 단축할 수 있지만 신중하게 사용해야 한다. 변경되지 않은 종속성을 반복적으로 다운로드하거나 다시 빌드하면 러너 자원이 낭비되지만 잘못된 캐시는 종속성 문제를 숨기거나 재현 불가능한 동작을 발생시킬 수 있다. 따라서 캐시 키(Cache Key)는 운영체제, ROS 배포판, 종속성, 빌드 구성의 주요 변경 사항을 반영하여 성능 향상이 검증 신뢰성을 훼손하지 않도록 설계해야 한다.

아티팩트(Artifact)는 CI 검증과 이후 전달 단계(Delivery Stage)를 연결한다. 성공적인 작업은 바이너리(Binary), ROS 패키지, 테스트 보고서(Test Report), 코드 커버리지(Code Coverage) 정보, 로그(Log), 펌웨어 구성 요소, 컨테이너 관련 결과물을 보존할 수 있다. 가능한 경우 후속 작업은 동일한 소프트웨어를 다시 빌드하는 대신 검증된 아티팩트를 사용해야 하며, 이를 통해 파이프라인에서 테스트된 아티팩트가 이후 배포 단계에서도 식별 가능한 상태로 유지되도록 한다.

컨테이너화된 빌드 환경(Containerized Build Environment)은 ROS 2 프로젝트가 시스템 라이브러리, CUDA 버전, 디바이스 SDK, 특수 미들웨어(Specialized Middleware)의 복잡한 조합에 의존할 때 재현성을 더욱 향상시킬 수 있다. 통제된 컨테이너 이미지(Container Image)는 러너에 예측 가능한 환경을 제공하고 구성 드리프트(Configuration Drift)를 줄여준다. 생성된 이미지나 패키지 애플리케이션은 이후 컨테이너 레지스트리(Container Registry), 엣지 배포(Edge Deployment), 플릿 업데이트(Fleet Update) 워크플로의 입력으로 사용할 수 있다.

자체 호스팅 러너(Self-hosted Runner)는 표준 호스팅 환경으로 로봇 전용 하드웨어 조건을 재현할 수 없을 때 중요해진다. 개발 인프라 내부의 러너는 ARM64 프로세서, NVIDIA GPU, CAN 인터페이스, 센서 또는 전용 시험 장비에 대한 접근을 제공할 수 있다. CI 작업은 저장소에서 제공되는 명령을 실행하고 중요한 하드웨어나 내부 네트워크 자원과 상호작용할 수 있으므로 이러한 러너는 격리되고 엄격하게 통제되어야 한다.

GitHub Actions는 실제 하드웨어를 사용하기 전에 시뮬레이션 기반 검증(Simulation-based Verification)을 조정할 수도 있다. 컴파일과 기본 테스트가 성공하면 워크플로가 사전에 정의된 시뮬레이션 환경을 실행하고 대표적인 ROS 2 노드를 구동하여 예상 동작을 평가할 수 있다. 이를 통해 내비게이션(Navigation), 인식(Perception), 조작(Manipulation), 통신(Communication), 수명주기 동작(Lifecycle Behavior)을 소프트웨어 리비전이 하드웨어 인 더 루프(Hardware-in-the-Loop, HIL) 또는 로봇 인 더 루프(Robot-in-the-Loop) 테스트로 진행되기 전에 자동으로 회귀 검증할 수 있다.

보안 제어(Security Control)는 배포 시점에만 추가하는 것이 아니라 워크플로 전체에 통합되어야 한다. 저장소 권한(Repository Permission), 보호 브랜치(Protected Branch), 워크플로 권한(Workflow Permission), 비밀정보(Secret), 종속성 스캐닝(Dependency Scanning), 아티팩트 무결성(Artifact Integrity), 서드파티 액션(Third-party Action)은 모두 소프트웨어 공급망(Software Supply Chain)에 영향을 준다. 비밀정보를 워크플로 파일에 직접 포함해서는 안 되며 외부 액션은 신뢰할 수 있는 빌드 및 검증 환경의 일부로 실행되므로 신중하게 선택하고 버전을 관리해야 한다.

풀 리퀘스트(Pull Request)는 ROS 2 개발에서 자연스러운 품질 경계(Quality Boundary)를 제공한다. 필수 상태 검사(Required Status Check)를 설정하면 지정된 워크플로가 성공적으로 완료될 때까지 병합을 방지할 수 있으며, 코드 리뷰(Code Review)는 자동 테스트에서 발견하기 어려운 아키텍처 및 동작상의 문제를 검토한다. 자동화된 CI와 사람에 의한 검토를 결합하면 개별 개발자의 변경 사항에서 로봇 개발 조직이 사용하는 공유 소프트웨어 기준선(Software Baseline)까지 통제된 변경 경로를 구축할 수 있다.

성숙한 GitHub Actions 파이프라인은 단순한 원격 ROS 2 빌드 서비스(Remote Build Service) 이상의 역할을 수행한다. 저장소 이벤트, 재현 가능한 환경, 종속성 관리, 컴파일, 자동 테스트, 정적 분석, 시뮬레이션, 아티팩트, 보안 제어, 병합 정책(Merge Policy)을 하나의 추적 가능한 워크플로로 연결한다. 이러한 기반은 이후의 컨테이너 전달(Container Delivery), 단계적 플릿 배포(Staged Fleet Deployment), 하드웨어 인 더 루프 테스트, 지속적인 로봇 소프트웨어 진화(Continuous Robot Software Evolution)를 가능하게 한다.

## 2.3. GitLab CI Multi Stage Pipeline for Embedded SW [w/Code]

![](images/image3.png){width="7.268055555555556in" height="7.268055555555556in"}

GitLab CI는 로봇 시스템에 사용되는 임베디드 소프트웨어(Embedded Software)의 복잡성을 관리할 수 있도록 다단계 파이프라인(Multi-stage Pipeline)을 구축하는 통합 자동화 프레임워크(Integrated Automation Framework)를 제공한다. 일반적인 애플리케이션 소프트웨어와 달리 임베디드 로봇 소프트웨어에는 마이크로컨트롤러 펌웨어(Microcontroller Firmware), 실시간 제어 코드(Real-time Control Code), 하드웨어 추상화 계층(Hardware Abstraction Layer), 통신 드라이버(Communication Driver), 부트로더(Bootloader), 보드별 구성(Board-specific Configuration)이 포함될 수 있다. 구조화된 파이프라인을 통해 이러한 구성 요소가 실제 하드웨어에 적용되기 전에 체계적으로 검증할 수 있다.

GitLab CI 파이프라인은 일반적으로 프로젝트 소스 코드와 함께 버전 관리되는 \`.gitlab-ci.yml\` 파일에 정의된다. 이 파일은 파이프라인 단계(Stage), 작업(Job), 실행 규칙(Execution Rule), 종속성(Dependency), 변수(Variable), 아티팩트(Artifact), 러너 요구사항(Runner Requirement)을 기술한다. 파이프라인 정의를 임베디드 소프트웨어와 함께 관리하면 빌드 절차의 변경도 소스 코드와 함께 검토하고 버전 관리할 수 있어 소프트웨어 리비전(Software Revision)과 검증 프로세스 사이에 재현 가능한 관계를 구축할 수 있다.

다단계 설계(Multi-stage Design)는 검증 과정을 준비(Preparation), 빌드(Build), 정적 분석(Static Analysis), 단위 테스트(Unit Testing), 통합 테스트(Integration Testing), 패키징(Packaging), 배포 준비(Deployment Preparation)와 같은 순차적인 계층으로 구분한다. 동일한 단계에서 서로 독립적인 작업은 병렬로 실행할 수 있으며 이후 단계는 필요한 선행 작업이 성공한 경우에만 실행된다. 이를 통해 기본적인 컴파일이나 소프트웨어 품질 검사에서 이미 실패한 경우 고비용 하드웨어 및 시스템 수준 테스트에 자원이 투입되는 것을 방지할 수 있다.

준비 단계(Preparation Stage)는 결정론적인 빌드 환경(Deterministic Build Environment)을 구축한다. 대상 플랫폼에 필요한 툴체인(Toolchain), SDK, 라이브러리(Library), 서브모듈(Submodule), 보드 지원 패키지(Board Support Package), 프로젝트 종속성을 확보할 수 있다. 임베디드 개발은 정확한 컴파일러 및 공급업체 SDK 버전에 크게 의존하므로 개발자 워크스테이션의 환경을 암묵적으로 사용하는 대신 이러한 종속성을 통제해야 한다. 컨테이너화된 러너(Containerized Runner)를 사용하면 로컬 개발 환경과 CI 실행 환경 사이의 일관성을 더욱 향상시킬 수 있다.

빌드 단계(Build Stage)는 소스 코드를 대상별 바이너리(Target-specific Binary)로 변환한다. 로보틱스 저장소에서는 ARM 기반 리눅스 컴퓨터, Cortex-M 마이크로컨트롤러, 실시간 프로세서(Real-time Processor), 기타 임베디드 대상을 위한 별도의 작업이 필요할 수 있다. 교차 컴파일(Cross-compilation)을 사용하면 고성능 x86-64 CI 서버에서 이러한 아키텍처용 소프트웨어를 생성할 수 있다. 여러 보드가 대부분의 빌드 절차를 공유하면서 서로 다른 툴체인이나 컴파일 옵션을 요구한다면 재사용 가능한 템플릿(Reusable Template)을 통해 중복을 줄일 수 있다.

컴파일(Compilation)은 단순한 아티팩트 생성 과정이 아니라 품질 게이트(Quality Gate)로 취급해야 한다. 중요한 모듈에서는 컴파일러 경고(Compiler Warning)를 오류로 처리할 수 있으며 구성 오류, 해결되지 않은 심볼(Unresolved Symbol), 호환되지 않는 인터페이스, 과도한 바이너리 크기는 즉각적인 파이프라인 실패 조건이 될 수 있다. 빌드 메타데이터(Build Metadata)에는 소스 리비전, 컴파일러 버전, 대상 보드(Target Board), 빌드 유형(Build Type), 주요 옵션을 기록하여 생성된 펌웨어를 이후에도 재현하고 분석할 수 있도록 해야 한다.

정적 분석(Static Analysis)은 실제 하드웨어가 사용되기 전에 수행할 수 있는 또 다른 초기 검증 계층이다. 린터(Linter), 코딩 규칙 검사기(Coding-rule Checker), 컴파일러 진단(Compiler Diagnostic), 전문 분석 도구를 통해 의심스러운 메모리 연산, 도달 불가능 코드(Unreachable Code), 안전하지 않은 형 변환(Unsafe Conversion), 동시성 문제(Concurrency Problem), 프로젝트 코딩 표준 위반을 탐지할 수 있다. 이러한 검사는 상대적으로 적은 하드웨어 인프라만 필요하므로 병합 요청(Merge Request)마다 자동으로 실행하여 결함이 이후 테스트 단계로 전달되는 것을 방지할 수 있다.

단위 테스트(Unit Testing)는 가능한 경우 임베디드 로직(Embedded Logic)을 물리적 장치에서 분리하여 검증한다. 제어 알고리즘(Control Algorithm), 상태 머신(State Machine), 프로토콜 파서(Protocol Parser), 진단 기능(Diagnostic Function), 수학 루틴(Mathematical Routine), 하드웨어 독립 라이브러리는 호스트 러너(Host Runner)에서 실행할 수 있다. 호스트 기반 테스트는 빠른 피드백과 높은 테스트 처리량을 제공하며 이후 대상 기반 테스트(Target-based Test)를 통해 프로세서 아키텍처, 타이밍 동작, 메모리 배치, 임베디드 런타임 특성에 의존하는 조건을 검증할 수 있다.

통합 테스트(Integration Testing)는 펌웨어 모듈과 외부 인터페이스 사이의 상호작용을 검증한다. CAN, UART, SPI, I2C, Ethernet 등의 통신은 시뮬레이터(Simulator), 가상 장치(Virtual Device), 전용 테스트 픽스처(Test Fixture)를 필요로 할 수 있다. 로봇 시스템에서는 저수준 제어기(Low-level Controller)와 ROS 2 노드와 같은 상위 소프트웨어 사이의 데이터 교환도 검증할 수 있다. 목적은 완전한 로봇 수준 검증이 필요해지기 전에 인터페이스와 프로토콜 오류를 발견하는 것이다.

GitLab Runner는 파이프라인 작업을 실제로 수행하는 실행 계층(Execution Layer)을 제공한다. 공유 러너(Shared Runner)는 일반적인 컴파일과 분석을 담당하고 자체 호스팅 러너(Self-hosted Runner)는 특수 툴체인, ARM 컴퓨터, GPU, 디버깅 프로브(Debugging Probe), CAN 인터페이스, 프로그래머블 전원공급장치(Programmable Power Supply), 테스트 벤치(Test Bench)에 대한 접근을 제공할 수 있다. 러너 태그(Runner Tag)를 사용하면 작업이 필요한 인프라를 지정하여 하드웨어 의존 작업이 적합한 시스템에서만 안전하게 실행되도록 할 수 있다.

아티팩트(Artifact)는 검증된 결과물을 파이프라인 단계 사이에서 전달한다. 펌웨어 바이너리(Firmware Binary), ELF 파일, 심볼 정보(Symbol Information), 맵 파일(Map File), 테스트 보고서(Test Report), 커버리지 결과(Coverage Result), 로그(Log), 구성 패키지(Configuration Package), 체크섬(Checksum)을 개별 작업이 종료된 이후에도 보존할 수 있다. 가능한 경우 이후 단계에서는 소프트웨어를 독립적으로 다시 빌드하지 않고 이러한 아티팩트를 사용하여 검증을 통과한 소프트웨어와 최종 배포 준비 바이너리 사이의 직접적인 관계를 유지해야 한다.

캐싱(Caching)은 아티팩트와 다른 목적으로 사용된다. 캐시는 재사용 가능한 종속성, 컴파일러 다운로드, 중간 자원을 보존하여 반복되는 작업을 가속하는 반면 아티팩트는 특정 파이프라인 실행과 연결되어 유지되어야 하는 결과물이다. 두 개념을 혼동하면 재현성이 저하될 수 있다. 따라서 캐시 키(Cache Key)는 관련 툴체인 및 종속성 버전을 반영하고 릴리스와 관련된 바이너리는 불변 파이프라인 아티팩트(Immutable Pipeline Artifact)로 관리하는 것이 적절하다.

하드웨어 인 더 루프(Hardware-in-the-Loop, HIL) 테스트는 소프트웨어 전용 단계가 성공한 이후에 적용할 수 있다. 전용 러너는 임베디드 제어기에 펌웨어를 플래싱(Flashing)하고 대상을 리셋하며 디지털 또는 통신 인터페이스에 입력을 제공하고 텔레메트리(Telemetry)를 수집하여 예상 응답을 평가할 수 있다. 더욱 발전된 테스트 벤치는 실제 모터 제어기, 센서, 액추에이터(Actuator), 로봇 서브시스템을 연결하여 완성된 로봇에 배포하기 전에 타이밍과 하드웨어 상호작용 문제를 자동으로 탐지할 수 있다.

임베디드 파이프라인은 일반적인 서버 애플리케이션에서는 상대적으로 중요하지 않은 자원 제약(Resource Constraint)도 고려해야 한다. 플래시 사용량(Flash Usage), RAM 사용량, 스택 요구량(Stack Requirement), 부팅 시간(Boot Time), 제어 루프 타이밍(Control-loop Timing), 통신 지연(Communication Latency), CPU 사용률은 모두 빌드의 허용 여부를 결정할 수 있다. CI는 이러한 지표를 사전에 정의된 한계값 또는 이전 기준선(Baseline)과 비교하여 기능적으로 정상인 변경이라도 과도한 자원 증가나 실시간 성능 저하를 발생시키면 거부할 수 있다.

릴리스 패키징(Release Packaging)은 검증된 결과물을 실제 배포 가능한 단위로 변환한다. 대상에 따라 펌웨어 이미지, 부트로더 호환 패키지(Bootloader-compatible Package), 업데이트 매니페스트(Update Manifest), 구성 파일, 암호화 서명(Cryptographic Signature), 버전 메타데이터(Version Metadata)가 포함될 수 있다. 패키지는 해당 결과물을 생성한 소스 리비전과 파이프라인을 명확하게 연결하는 식별 정보를 유지해야 하며, 이러한 관계는 현장에 배포된 로봇을 진단하거나 업데이트하고 롤백(Rollback)할 때 매우 중요하다.

보안 제어(Security Control)는 CI 인프라와 임베디드 소프트웨어 공급망(Embedded Software Supply Chain)을 모두 보호해야 한다. 보호 브랜치(Protected Branch), 제한된 러너(Restricted Runner), 통제된 변수, 비밀정보 관리(Secret Management), 종속성 스캐닝(Dependency Scanning), 아티팩트 서명(Artifact Signing), 접근 정책(Access Policy)은 승인되지 않은 코드나 자격 증명이 빌드 프로세스에 진입할 가능성을 줄인다. 하드웨어 연결 러너는 잘못되거나 악의적인 작업이 장치를 재프로그래밍하거나 시험실 장비에 영향을 줄 수 있으므로 특히 강력하게 보호해야 한다.

병합 요청(Merge Request)은 자동화된 파이프라인 결과와 사람의 엔지니어링 검토(Human Engineering Review)를 결합할 수 있다. 필수 작업(Required Job)은 컴파일, 테스트, 분석 또는 대상 검증이 실패하면 병합을 차단할 수 있으며 검토자는 자동화된 검사만으로 완전히 판단하기 어려운 아키텍처 변경과 하드웨어 영향을 평가한다. 이를 통해 개별 펌웨어 변경 사항이 시스템 통합과 로봇 테스트에 적합한 공유 임베디드 소프트웨어 기준선(Embedded Software Baseline)으로 전환되는 과정을 통제할 수 있다.

성숙한 GitLab CI 다단계 파이프라인(Multi-stage Pipeline)은 단순한 원격 컴파일러(Remote Compiler)가 아니라 임베디드 로봇 소프트웨어를 위한 엔지니어링 기반(Engineering Backbone)이 된다. 소스 제어(Source Control), 재현 가능한 툴체인, 교차 컴파일, 정적 분석, 자동 테스트, 하드웨어 러너, 아티팩트, 자원 검사(Resource Check), 보안, 릴리스 패키징을 하나의 추적 가능한 과정으로 연결한다. 이러한 기반은 임베디드 제어기를 더 광범위한 ROS 2, 엣지 컴퓨팅(Edge Computing), 로봇 배포 아키텍처(Robot Deployment Architecture)와 안정적으로 통합할 수 있도록 지원한다.

## 2.4. Jenkins Pipeline for Large Scale Robot SW Projects [w/Code]

![](images/image4.png){width="7.268055555555556in" height="7.268055555555556in"}

Jenkins는 대규모 로봇 소프트웨어 프로젝트를 위한 지속적 통합(Continuous Integration, CI)과 지속적 전달(Continuous Delivery, CD) 파이프라인을 구축할 수 있는 높은 확장성의 자동화 플랫폼(Automation Platform)을 제공한다. 이러한 프로젝트에는 ROS 2 애플리케이션, 임베디드 펌웨어(Embedded Firmware), 인식(Perception) 및 내비게이션(Navigation) 모듈, AI 모델(AI Model), 시뮬레이션 자산(Simulation Asset), 컨테이너(Container), 클라우드 서비스(Cloud Service), 하드웨어별 구성 요소가 함께 포함될 수 있다. Jenkins는 반복 가능한 워크플로를 통해 이러한 이기종 요소를 조정하고 여러 개발팀과 컴퓨팅 환경으로 확장할 수 있도록 한다.

Jenkins 파이프라인은 일반적으로 버전 관리되는 \`Jenkinsfile\`에 코드 형태로 정의되며, 이를 통해 빌드 및 검증 프로세스를 로봇 소프트웨어와 함께 발전시킬 수 있다. 선언형 파이프라인(Declarative Pipeline) 또는 스크립트형 파이프라인(Scripted Pipeline)을 사용하여 단계(Stage), 에이전트(Agent), 환경 변수(Environment Variable), 자격 증명(Credential), 실행 조건(Condition), 병렬 실행(Parallel Execution), 빌드 후 작업(Post-build Action)을 정의할 수 있다. 코드형 파이프라인(Pipeline-as-Code)은 변경 사항을 검토 가능하게 하고 특정 소프트웨어 리비전과 연결된 자동화 로직을 재구성하는 데 도움을 준다.

대규모 로보틱스 저장소에서는 모든 구성 요소가 동일한 실행 환경을 요구하는 경우가 드물다. Jenkins는 컨트롤러-에이전트 아키텍처(Controller-Agent Architecture)를 통해 이를 처리하며, 컨트롤러(Controller)는 작업을 조정하고 에이전트는 실제 컴퓨팅 자원을 제공한다. 서로 다른 에이전트를 Linux 빌드, Windows 도구, ARM 대상, GPU 워크로드, 임베디드 툴체인(Embedded Toolchain), 시뮬레이션 서버(Simulation Server), 물리적 로봇 시험실에 맞게 구성하여 각 작업을 적절한 인프라에서 실행할 수 있다.

일반적인 파이프라인은 소스 체크아웃(Source Checkout)과 환경 준비(Environment Preparation)에서 시작하여 컴파일, 분석, 테스트, 패키징(Packaging), 배포 준비(Deployment Preparation) 단계로 진행된다. 초기 단계에서는 저비용 검사를 빠르게 수행하여 명백한 결함이 고비용 자원을 사용하기 전에 제거되도록 해야 한다. 이후 단계에서는 시뮬레이션, 하드웨어 인 더 루프(Hardware-in-the-Loop, HIL) 테스트, 로봇 인 더 루프(Robot-in-the-Loop) 검증, 스테이징 플릿(Staging Fleet)에 대한 통제된 배포 등 점차 실제 환경에 가까운 검증을 수행할 수 있다.

병렬 실행(Parallel Execution)은 대규모 로봇 소프트웨어 스택에서 특히 중요하다. 서로 독립적인 ROS 2 패키지, 펌웨어 대상, 운영체제 구성, 시뮬레이션 시나리오, AI 추론 테스트(Inference Test)를 서로 다른 에이전트에서 동시에 실행할 수 있다. 이를 통해 순차 처리에 비해 전체 피드백 시간을 단축할 수 있으며, Jenkins는 작업 간 종속성을 조정하여 필요한 상위 빌드와 검증 작업이 성공적으로 완료된 경우에만 후속 단계를 시작하도록 구성할 수 있다.

매트릭스 방식 검증(Matrix-style Validation)은 이러한 구조를 다양한 이기종 대상까지 확장할 수 있다. 프로젝트에서는 x86-64 개발 시스템, ARM64 엣지 컴퓨터(Edge Computer), Jetson 플랫폼, 마이크로컨트롤러(Microcontroller), 여러 ROS 2 배포판(ROS 2 Distribution) 및 컴파일러 구성을 검증해야 할 수 있다. Jenkins는 이러한 조합을 적절한 레이블(Label)이 지정된 에이전트에 분산하여 하나의 논리적 파이프라인으로 로봇 제품군 전체에서 사용하는 다양한 하드웨어 및 소프트웨어 환경의 호환성을 검증할 수 있다.

수백 개의 소프트웨어 구성 요소와 외부 종속성을 사용하는 환경에서는 빌드 재현성(Build Reproducibility)이 필수적이다. 따라서 Jenkins 에이전트는 수동으로 구성된 시스템 환경에 의존하기보다 통제된 툴체인, 컨테이너, 종속성 버전, 환경 정의를 사용해야 한다. 일시적 컨테이너 기반 에이전트(Ephemeral Container-based Agent)는 일반적인 빌드에 깨끗한 환경을 제공할 수 있으며 독점 SDK, GPU, 디버깅 인터페이스, 물리적 시험 장비에는 지속적으로 유지되는 전문 에이전트가 필요할 수 있다.

ROS 2 통합(ROS 2 Integration)은 단순한 컴파일 성공 이상의 검증을 요구한다. Jenkins는 워크스페이스 빌드(Workspace Build), 패키지 테스트, 실행 테스트(Launch Test), 린팅(Linting), 인터페이스 검사(Interface Check), 시스템 수준 검증(System-level Validation)을 실행하고 결과 보고서를 보존할 수 있다. 실패가 발생하면 단순히 파이프라인 전체의 실패만 표시하는 것이 아니라 문제가 발생한 패키지나 서브시스템을 식별할 수 있어야 하며, 이는 내비게이션, 인식, 제어, 통신, 플릿 소프트웨어를 서로 다른 팀이 개발하는 환경에서 특히 중요하다.

대규모 프로젝트에서는 재사용 가능한 파이프라인 로직(Reusable Pipeline Logic)과 프로젝트별 구성을 분리하는 것이 효과적이다. Jenkins 공유 라이브러리(Shared Library)는 소스 체크아웃, ROS 2 환경 준비, 컴파일, 테스트, 아티팩트 게시(Artifact Publication), 컨테이너 생성, 배포 절차와 같은 공통 기능을 캡슐화할 수 있다. 각 팀은 저장소마다 대규모 스크립트를 복사하지 않고 표준화된 파이프라인 동작을 재사용할 수 있으며, 중앙에서 관리되는 라이브러리를 통해 조직 전체의 개선 사항을 체계적으로 전파할 수 있다.

아티팩트(Artifact)는 분산된 Jenkins 파이프라인의 여러 단계를 연결한다. 성공적인 빌드는 바이너리(Binary), ROS 패키지, 펌웨어 이미지(Firmware Image), 컨테이너 이미지(Container Image), AI 모델 패키지, 테스트 보고서, 로그(Log), 체크섬(Checksum), 메타데이터(Metadata)를 보존할 수 있다. 가능한 경우 이후 단계에서는 동일한 소프트웨어를 다시 빌드하지 않고 검증된 아티팩트를 승격(Promotion)해야 한다. 이를 통해 검증을 통과한 정확한 결과물과 실제 로봇 또는 플릿에 전달되는 버전 사이의 추적성을 유지할 수 있다.

시뮬레이션(Simulation)은 상당한 CPU 및 GPU 자원을 소비할 수 있으므로 스케줄링(Scheduling)이 중요한 아키텍처 요소가 된다. Jenkins는 시뮬레이션 작업을 전용 에이전트에 할당하고 인프라가 허용하는 경우 여러 시나리오를 병렬로 실행할 수 있다. 내비게이션 회귀(Regression), 인식 파이프라인, 조작(Manipulation) 작업, 통신 장애, 환경 변화 등을 실제 로봇을 사용하기 전에 검증할 수 있으며, 시뮬레이션 실패 시 물리적 하드웨어 검증 단계로 불필요하게 진행되는 것을 방지할 수 있다.

하드웨어 인 더 루프(Hardware-in-the-Loop, HIL) 테스트에서는 임베디드 제어기, 센서, 액추에이터(Actuator), 네트워크 장비 또는 완전한 로봇 서브시스템에 연결된 전문 Jenkins 에이전트를 사용한다. 파이프라인 작업은 펌웨어 플래싱(Flashing), ROS 2 노드 실행, 인터페이스 자극, 텔레메트리(Telemetry) 수집, 예상 동작 평가를 수행할 수 있다. 여러 작업이 동일한 로봇이나 테스트 벤치(Test Bench)를 동시에 제어하지 않도록 자원 잠금(Resource Locking)을 적용하는 것이 중요하며, 특히 실제 움직임을 발생시키는 하드웨어에서는 이러한 통제가 필수적이다.

로봇 인 더 루프(Robot-in-the-Loop) 단계는 소프트웨어와 서브시스템 테스트를 통과한 이후 완전한 물리적 플랫폼까지 검증 범위를 확장한다. 전용 테스트 로봇은 사전에 정의된 내비게이션, 도킹(Docking), 조작, 통신, 안전 시나리오를 실행하고 Jenkins는 그 결과를 기록할 수 있다. 물리적 테스트는 소프트웨어 테스트보다 느리고 비용이 높기 때문에 컴파일, 정적 분석, 단위 테스트, 통합 테스트, 시뮬레이션을 통과한 리비전에 대해서만 이러한 자원을 할당하는 것이 적절하다.

Jenkins는 로봇 엣지 컴퓨터(Robot Edge Computer)를 위한 컨테이너 기반 전달(Container-oriented Delivery)도 조정할 수 있다. 검증된 소프트웨어를 버전이 지정된 컨테이너 이미지로 구성하고 레지스트리(Registry)에 전송할 수 있으며, 메타데이터를 통해 각 이미지와 소스 리비전 및 테스트 이력을 연결할 수 있다. 이후 배포 단계에서는 동일한 이미지를 개발, 스테이징(Staging), 운영 환경(Production Environment)으로 순차적으로 승격하여 테스트된 소프트웨어와 실제 로봇에서 실행되는 소프트웨어 사이의 차이를 줄일 수 있다.

Jenkins가 저장소, 레지스트리, 클라우드 인프라, 서명 키(Signing Key), 내부 네트워크, 물리적 하드웨어에 접근하게 되면서 보안(Security)의 중요성도 증가한다. 자격 증명은 파이프라인 스크립트에 직접 포함하지 않고 통제된 자격 증명 관리 메커니즘(Credential Management Mechanism)을 통해 관리해야 한다. 에이전트 권한, 플러그인 거버넌스(Plugin Governance), 접근 제어(Access Control), 아티팩트 서명(Artifact Signing), 종속성 스캐닝(Dependency Scanning), 신뢰할 수 없는 작업의 격리(Isolation)를 통해 자동화 인프라와 로봇 소프트웨어 공급망(Software Supply Chain)을 보호할 수 있다.

대규모 환경에서는 CI/CD 인프라 자체에 대한 관측 가능성(Observability)도 필요하다. 개발팀은 대기 시간(Queue Time), 빌드 시간(Build Duration), 실패율(Failure Rate), 에이전트 사용률(Agent Utilization), 불안정 테스트(Flaky Test), 시뮬레이션 처리 용량, 하드웨어 테스트 가용성을 추적할 수 있다. 이러한 지표를 통해 느린 피드백의 원인이 소프트웨어 복잡성인지 인프라 부족인지 파악할 수 있으며, 로그와 과거 빌드 기록은 반복적으로 발생하는 실패를 분석하고 파이프라인 효율성을 개선하는 근거를 제공한다.

파이프라인 게이트(Pipeline Gate)는 점차 중요도가 높아지는 환경 사이의 전환을 통제한다. 테스트가 실패하면 종속된 후속 단계가 중단되어야 하며, 특정 릴리스는 하드웨어 테스트나 플릿 배포 전에 엔지니어링 승인(Engineering Approval)을 요구할 수 있다. 보호된 소스 브랜치(Protected Source Branch)와 코드 리뷰(Code Review)를 이러한 게이트와 결합하면 개별 변경 사항이 검증 프로세스를 우회하는 것을 방지하고 개발자의 커밋에서 승인된 로봇 소프트웨어 기준선(Robot Software Baseline)까지 명확한 경로를 제공할 수 있다.

대규모 로봇 소프트웨어에서 Jenkins는 분산 저장소(Distributed Repository), 이기종 빌드 에이전트(Heterogeneous Build Agent), ROS 2 테스트, 임베디드 대상, 시뮬레이션 인프라, GPU 자원, 하드웨어 시험실, 아티팩트, 보안 제어, 배포 시스템을 연결하는 오케스트레이션 계층(Orchestration Layer)으로 기능한다. Jenkins의 핵심 가치는 개별 작업을 단순히 자동화하는 것이 아니라 복잡한 로봇 소프트웨어를 안전하고 재현 가능하며 조직 규모로 지속 발전시키는 데 필요한 전체 검증 경로를 통합적으로 조정하는 데 있다.

## 2.5. Cross Compilation CI ARM x86 RISC V Targets [w/Code]

![](images/image5.png){width="7.268055555555556in" height="7.268055555555556in"}

교차 컴파일(Cross-compilation)은 하나의 프로세서 아키텍처(Processor Architecture)에서 소스 코드를 컴파일하면서 생성된 실행 파일은 다른 아키텍처에서 실행되도록 하는 소프트웨어 개발 기법이다. 로보틱스에서는 고성능 x86-64 워크스테이션이나 CI 서버에서 ARM64 엣지 컴퓨터(Edge Computer), ARM 마이크로컨트롤러(Microcontroller), RISC-V 프로세서용 소프트웨어를 빌드할 수 있다. 이를 통해 각각의 대상 장치가 직접 컴파일을 수행하지 않아도 중앙화되고 자동화된 빌드 환경을 구축할 수 있다.

교차 컴파일 환경(Cross-compilation Environment)은 단순히 컴파일러만으로 구성되지 않는다. 일반적으로 대상별 컴파일러(Target-specific Compiler), 링커(Linker), 어셈블러(Assembler), 시스템 헤더(System Header), 런타임 라이브러리(Runtime Library), 시스템 루트(Sysroot), 빌드 시스템 구성(Build-system Configuration), 아키텍처별 종속성(Architecture-specific Dependency)이 함께 필요하다. 이러한 구성 요소는 대상 환경을 정확하게 반영해야 하며, 빌드 시스템이 실수로 호스트 라이브러리나 헤더를 사용하면 컴파일에는 성공하더라도 실제 로봇 하드웨어에서 정상적으로 실행되지 않는 바이너리가 생성될 수 있다.

따라서 호스트(Host), 빌드(Build), 대상(Target)의 개념을 명확하게 구분해야 한다. 호스트 시스템은 컴파일러와 CI 작업을 실행하고 대상 아키텍처는 생성된 소프트웨어를 실행한다. 많은 로보틱스 환경에서는 x86-64 Linux 서버가 호스트 역할을 수행하고 ARM64 또는 RISC-V Linux 컴퓨터가 로봇 내부에서 동작한다. 마이크로컨트롤러 펌웨어는 범용 운영체제나 일반적인 Linux 런타임을 사용하지 않는 또 다른 형태의 대상을 구성한다.

ARM 대상(Target)은 임베디드 마이크로컨트롤러부터 고성능 엣지 컴퓨터까지 폭넓게 사용되기 때문에 로보틱스에서 일반적이다. CI 파이프라인은 Cortex-M 장치를 위한 펌웨어를 빌드하면서 별도로 ARM64 프로세서 또는 Jetson 계열 플랫폼을 위한 Linux 애플리케이션을 생성할 수 있다. 이러한 대상은 모두 광범위한 ARM 생태계에 포함되지만 실행 환경, 응용 프로그램 바이너리 인터페이스(Application Binary Interface, ABI), 운영체제, 부동소수점 옵션(Floating-point Option), 툴체인이 크게 다를 수 있으므로 별도의 빌드 구성으로 관리해야 한다.

x86-64는 개발 아키텍처(Development Architecture)이면서 동시에 배포 대상(Deployment Target)으로서 중요한 역할을 한다. 엔지니어링 워크스테이션, 시뮬레이션 서버, 산업용 PC, 일부 로봇 엣지 컴퓨터는 x86-64 프로세서를 사용한다. 교차 컴파일된 ARM 또는 RISC-V 빌드와 함께 네이티브 x86 빌드(Native x86 Build)를 유지하면 많은 단위 테스트와 통합 테스트를 CI 러너에서 직접 실행할 수 있으므로 빠른 피드백을 제공하고 이후 대상별 바이너리를 에뮬레이션(Emulation), 하드웨어 테스트, 실제 로봇 검증 단계로 진행할 수 있다.

RISC-V는 임베디드 및 특수 목적 컴퓨팅에서 점차 중요성이 높아지고 있는 개방형 명령어 집합 아키텍처(Open Instruction Set Architecture)이다. RISC-V를 대상으로 하는 CI 파이프라인은 필요한 명령어 집합 확장(Instruction-set Extension), 응용 프로그램 바이너리 인터페이스, 컴파일러 구성, 런타임 환경을 명확하게 정의해야 한다. 단순히 RISC-V라는 일반적인 명칭만으로는 충분하지 않으며 구현에 따라 정수, 곱셈, 원자적 연산(Atomic Operation), 부동소수점, 압축 명령어(Compressed Instruction), 벡터(Vector) 등 서로 다른 확장 조합을 지원할 수 있다.

툴체인 버전 관리(Toolchain Version Control)는 재현 가능한 교차 컴파일을 위해 필수적이다. 컴파일러 릴리스가 달라지면 최적화 동작, 경고, 생성 명령어, 바이너리 크기, 라이브러리 호환성이 변경될 수 있다. 따라서 CI 환경은 컴파일러 버전, 바이너리 유틸리티(Binutils), C/C++ 표준 라이브러리, SDK, 대상 런타임 구성 요소를 식별하고 통제해야 한다. 컨테이너 이미지(Container Image)나 불변 빌드 환경(Immutable Build Environment)을 사용하면 이러한 종속성을 패키징하여 개발자와 자동화 러너가 일관된 툴체인을 사용하도록 구성할 수 있다.

시스템 루트(Sysroot)는 대상 Linux 환경을 교차 컴파일러가 참조할 수 있도록 필요한 파일 시스템 환경을 제공한다. 여기에는 대상 시스템의 헤더, 공유 라이브러리(Shared Library), 시작 파일(Startup File), 기타 개발 자원이 포함될 수 있다. 시스템 루트는 실제 배포되는 운영체제 이미지와 최대한 일치해야 한다. 오래되었거나 일관성이 없는 시스템 루트를 사용하면 CI에서는 정상적으로 링크되지만 실제 로봇에서는 필요한 라이브러리 버전이나 심볼(Symbol)이 없어 실행되지 않는 바이너리가 생성될 수 있다.

빌드 시스템(Build System) 역시 다른 아키텍처를 대상으로 컴파일하고 있다는 사실을 명확하게 인식해야 한다. CMake 툴체인 파일(CMake Toolchain File)은 대상 시스템, 프로세서, 컴파일러, 시스템 루트, 검색 경로(Search Path), 관련 옵션을 정의할 수 있다. ROS 2 프로젝트에서는 이러한 설정이 워크스페이스 빌드 프로세스 및 패키지 종속성 구조와 함께 동작해야 한다. 개별 패키지 내부에 특정 아키텍처에 대한 가정이 숨겨져 있다면 x86-64에서 정상적으로 동작한 소프트웨어를 ARM64 또는 RISC-V로 교차 컴파일할 때 문제가 발생할 수 있다.

CI 파이프라인은 다중 아키텍처 빌드(Multi-architecture Build)를 병렬 작업(Parallel Job) 또는 빌드 매트릭스(Build Matrix) 형태로 구성할 수 있다. 하나의 소스 리비전(Source Revision)이 네이티브 x86-64 컴파일과 함께 ARM64, 임베디드 ARM, RISC-V 교차 빌드를 동시에 실행하도록 구성할 수 있다. 이를 통해 아키텍처별 오류를 변경 사항이 병합되기 전에 발견할 수 있으며, 서로 다른 대상이 독립적인 툴체인, SDK, 운영체제 이미지, 컴파일러 옵션을 요구할 때 병렬 실행을 통해 전체 검증 시간을 줄일 수 있다.

모든 테스트를 최종 대상 아키텍처에서 실행할 필요는 없다. 하드웨어 독립적인 알고리즘, 파서(Parser), 수학 함수, 상태 머신(State Machine), 많은 ROS 2 구성 요소는 먼저 x86-64 러너에서 네이티브 방식으로 테스트할 수 있다. 이후 대상 바이너리는 에뮬레이션 또는 실제 하드웨어를 이용하여 아키텍처별 검증을 수행할 수 있다. 이처럼 이식 가능한 기능 테스트(Portable Functional Test)와 대상 의존 테스트(Target-dependent Test)를 분리하면 빠른 피드백을 유지하면서 아키텍처별 동작도 검증할 수 있다.

에뮬레이션(Emulation)은 교차 컴파일과 물리적 하드웨어 테스트 사이의 중간 검증 계층을 제공한다. 사용자 모드(User-mode) 또는 시스템 수준 에뮬레이터(System-level Emulator)를 사용하면 일부 ARM 또는 RISC-V 바이너리를 x86-64 인프라에서 실행할 수 있으며, 이를 통해 CI에서 명령어 집합, 동적 링크(Dynamic Linking), 시작 과정, 런타임 문제를 탐지할 수 있다. 그러나 에뮬레이션은 모든 타이밍, 주변장치(Peripheral), 가속기(Accelerator), 하드웨어 동작을 재현하지 못하므로 에뮬레이션 성공을 실제 대상 하드웨어 검증의 완전한 대체 수단으로 간주해서는 안 된다.

대상 기반 테스트(Target-based Testing)는 교차 컴파일만으로 검증할 수 없는 가정을 확인한다. 자체 호스팅 러너(Self-hosted Runner) 또는 시험실 제어 시스템(Laboratory Controller)은 아티팩트(Artifact)를 ARM이나 RISC-V 장치로 전송하고 애플리케이션을 실행하며 로그를 수집하고 테스트를 수행한 뒤 결과를 CI 시스템으로 반환할 수 있다. 임베디드 펌웨어도 같은 방식으로 개발 보드에 플래싱(Flashing)할 수 있으며, 이를 통해 생성된 아티팩트가 의도한 프로세서와 런타임 환경에서 실제로 시작되고 동작하는지 확인할 수 있다.

하드웨어 전용 가속(Hardware-specific Acceleration)은 추가적인 고려가 필요하다. 로봇 컴퓨터는 ARM 또는 x86 CPU와 함께 GPU, NPU, DSP 등의 가속기를 사용할 수 있으며, 이들의 SDK와 런타임 라이브러리는 아키텍처에 따라 달라질 수 있다. 따라서 인식(Perception)이나 AI 애플리케이션을 교차 컴파일하려면 CPU 아키텍처와 가속기 스택(Accelerator Stack)에 모두 호환되는 헤더와 라이브러리가 필요하다. CI에서는 컴파일 시점의 가용성과 실제 가속기가 장착된 하드웨어에서의 런타임 검증을 구분해야 한다.

바이너리 호환성(Binary Compatibility)은 명시적인 품질 요구사항(Quality Requirement)으로 관리해야 한다. 아키텍처, ABI, 엔디언(Endianness), 명령어 확장, 동적 링커 경로(Dynamic-linker Path), 라이브러리 버전은 실행 파일이 대상에서 동작할 수 있는지를 결정한다. CI 작업은 배포 전에 생성된 바이너리와 메타데이터를 검사하여 실수로 생성된 호스트 빌드나 지원되지 않는 명령어를 탐지할 수 있다. 이러한 검사는 호환되지 않는 바이너리가 실제 로봇에 배포된 이후 문제를 분석하는 것보다 훨씬 적은 비용으로 수행할 수 있다.

아티팩트(Artifact)는 아키텍처와 대상 프로파일(Target Profile)에 따라 명확하게 분리되어야 한다. 패키지 이름, 디렉터리 구조, 컨테이너 태그(Container Tag), 펌웨어 메타데이터, 릴리스 매니페스트(Release Manifest)에 대상 아키텍처, 보드, 운영체제 버전, 소프트웨어 리비전을 포함할 수 있다. 이를 통해 ARM64 패키지를 x86-64 빌드로 잘못 인식하거나 RISC-V 바이너리를 이기종 로봇 플릿(Heterogeneous Robot Fleet)의 호환되지 않는 프로세서 변형에 배포하는 문제를 방지할 수 있다.

보안(Security)과 공급망 제어(Supply-chain Control)는 교차 컴파일에도 적용된다. 툴체인, 시스템 루트, SDK, 컨테이너 이미지, 외부 라이브러리는 신뢰할 수 있는 빌드 환경의 일부가 되므로 통제된 출처에서 확보하고 가능한 경우 버전을 관리해야 한다. 생성된 아티팩트에는 체크섬(Checksum)이나 서명(Signature)을 적용할 수 있으며, CI 기록에는 각 빌드와 연결된 소스 리비전 및 툴체인 식별 정보를 보존할 수 있다. 이를 통해 소프트웨어 무결성을 높이고 이후 보안 사고 분석에도 활용할 수 있다.

성숙한 교차 컴파일 CI 아키텍처(Cross-compilation CI Architecture)는 네이티브 빌드(Native Build), 통제된 툴체인, 시스템 루트, 아키텍처 인식형 빌드 구성(Architecture-aware Build Configuration), 병렬 다중 대상 작업(Parallel Multi-target Job), 에뮬레이션, 대상 하드웨어 테스트를 통합한다. x86-64는 고속 개발 및 CI 실행 환경을 제공하고 ARM과 RISC-V 아티팩트는 점차 대상 환경에 가까워지는 단계를 통해 생성되고 검증된다. 이를 통해 하나의 로봇 소프트웨어 코드베이스(Codebase)를 점점 다양해지는 이기종 컴퓨팅 플랫폼 전반에서 안정적으로 발전시킬 수 있다.

## 2.6. Automated Unit and Integration Test in CI [w/Code]

![](images/image6.png){width="7.268055555555556in" height="7.268055555555556in"}

자동화된 단위 테스트(Automated Unit Test)와 통합 테스트(Integration Test)는 로보틱스 소프트웨어의 지속적 통합(Continuous Integration, CI)에서 핵심적인 품질 관리 메커니즘이다. 관련 소스가 변경될 때마다 반복 가능한 테스트를 자동으로 실행하여 코드가 병합되거나 릴리스되기 전에 개별 기능과 소프트웨어 구성 요소 간 상호작용을 검증할 수 있다. ROS 2, 임베디드 제어기(Embedded Controller), 인식(Perception), 내비게이션(Navigation), AI 모듈을 결합하는 로봇 시스템에서 자동화 테스트는 결함이 비용이 높은 시뮬레이션이나 실제 하드웨어 단계까지 전달될 위험을 줄인다.

단위 테스트(Unit Test)는 실용적으로 분리할 수 있는 가장 작은 소프트웨어 구성 요소를 독립적으로 검증하는 데 초점을 둔다. 수학 함수, 상태 머신(State Machine), 경로 처리 알고리즘(Path-processing Algorithm), 프로토콜 파서(Protocol Parser), 좌표 변환(Coordinate Transformation), 진단 로직(Diagnostic Logic), 유틸리티 라이브러리(Utility Library)를 사전에 정의된 입력과 예상 출력으로 테스트할 수 있다. 이러한 테스트는 일반적으로 외부 인프라가 거의 필요하지 않기 때문에 빠르게 실행되며, 로컬 코드 변경으로 예상 동작이 달라졌을 때 개발자에게 즉각적인 피드백을 제공해야 한다.

우수한 단위 테스트는 결정론적(Deterministic)이고 독립적이며 반복 가능해야 한다. 동일한 소프트웨어 리비전(Software Revision)과 테스트 입력은 실행 순서나 관련 없는 시스템 상태와 관계없이 동일한 결과를 생성해야 한다. 센서, 데이터베이스, 네트워크 서비스, 시계(Clock), 하드웨어 인터페이스 등의 종속성은 모의 객체(Mock), 스텁(Stub), 통제된 테스트 대역(Test Double)으로 대체할 수 있다. 이를 통해 테스트 대상 구성 요소를 격리하고 CI 환경에서 실패를 보다 쉽게 재현하고 진단할 수 있다.

통합 테스트(Integration Test)는 여러 구성 요소가 실제 인터페이스를 통해 연결되었을 때 올바르게 동작하는지를 검증한다. ROS 2 시스템에서는 퍼블리셔(Publisher), 서브스크라이버(Subscriber), 서비스(Service), 액션(Action), 수명주기 노드(Lifecycle Node), 파라미터(Parameter), 실행 구성(Launch Configuration)이 포함될 수 있다. 하나의 구성 요소가 모든 단위 테스트를 통과하더라도 상호작용하는 노드 사이에서 메시지 유형, 네임스페이스(Namespace), 서비스 품질(Quality of Service, QoS) 정책, 초기화 순서, 타이밍 조건이 서로 다르면 시스템 수준에서는 실패할 수 있다.

CI 파이프라인은 일반적으로 자원 소모가 큰 통합 테스트보다 비용이 낮은 단위 테스트를 먼저 실행해야 한다. 소스 체크아웃(Source Checkout), 종속성 준비(Dependency Preparation), 컴파일, 정적 검사(Static Check), 단위 테스트가 초기 검증 계층을 구성한다. 이 단계를 성공적으로 통과한 리비전만 여러 프로세스, 미들웨어(Middleware) 통신, 데이터베이스, 컨테이너(Container), 시뮬레이터(Simulator), 특수 하드웨어가 필요한 통합 환경으로 진행해야 한다. 이러한 순서는 피드백 시간을 단축하고 공유 인프라의 불필요한 사용을 줄인다.

ROS 2는 패키지 빌드 및 테스트 워크플로에 직접 통합할 수 있는 테스트 메커니즘을 제공한다. 테스트를 개별 패키지와 연결하고 컴파일 이후 자동으로 실행할 수 있으며, 실행 기반 테스트(Launch-based Test)를 통해 여러 노드를 시작하고 상호작용을 평가할 수 있다. CI 시스템은 구조화된 테스트 결과(Structured Test Result)와 로그를 수집하여 개발자가 전체 로봇 환경을 수동으로 재현하지 않고도 문제가 발생한 패키지, 노드, 인터페이스 또는 검증 조건(Assertion)을 식별할 수 있도록 해야 한다.

인터페이스 테스트(Interface Testing)는 분산형 로봇 소프트웨어(Distributed Robot Software)에서 특히 중요하다. 메시지 정의(Message Definition), 서비스 계약(Service Contract), 액션 인터페이스(Action Interface), 토픽 이름(Topic Name), 파라미터 스키마(Parameter Schema), 직렬화 가정(Serialization Assumption)은 서로 다른 저장소나 개발팀에서 독립적으로 변경될 수 있다. 자동화 테스트는 데이터 생산자와 소비자 사이의 호환성을 검증할 수 있으며, 계약 지향 테스트(Contract-oriented Testing)는 파괴적 변경(Breaking Change)이 인식, 계획(Planning), 제어(Control), 플릿 관리(Fleet Management), 클라우드 연동 서비스로 확산되기 전에 탐지하도록 지원한다.

타이밍(Timing)과 동시성(Concurrency)은 일반적인 애플리케이션 테스트에서 드러나지 않을 수 있는 추가적인 문제를 발생시킨다. 로봇 소프트웨어에는 비동기 콜백(Asynchronous Callback), 멀티스레드 실행기(Multi-threaded Executor), 센서 스트림(Sensor Stream), 타이머(Timer), 실시간 또는 준실시간 제어 루프(Control Loop)가 빈번하게 포함된다. 따라서 통합 테스트에서는 시작 순서, 타임아웃 처리(Timeout Handling), 경쟁 상태(Race Condition), 동기화(Synchronization), 메시지 주기(Message Frequency), 복구 동작(Recovery Behavior)을 검증해야 한다. 반복 실행을 통해 특정 스케줄링 조건에서만 나타나는 간헐적인 오류도 발견할 수 있다.

테스트 픽스처(Test Fixture)는 반복 가능한 통합 시나리오를 실행할 수 있는 통제된 환경을 제공한다. 픽스처에는 사전에 정의된 구성 파일, 기록된 센서 데이터, 시뮬레이션 메시지, 임시 데이터베이스, 가상 네트워크 엔드포인트(Virtual Network Endpoint), 모의 하드웨어 인터페이스(Mock Hardware Interface)가 포함될 수 있다. 각 테스트 전에 이러한 자원을 초기화하면 하나의 테스트가 다른 테스트에 영향을 미치는 것을 방지할 수 있다. 재현 가능한 픽스처는 복잡한 로봇 상태에 의존하는 오류를 진단할 때 특히 유용하다.

기록된 데이터셋(Recorded Dataset)은 인식 및 센서 처리 소프트웨어를 결정론적으로 테스트하는 데 활용할 수 있다. 모든 CI 실행에서 실제 카메라, LiDAR, IMU, GNSS 수신기를 사용할 필요 없이 선별된 기록 데이터를 이용해 알려진 입력 시퀀스(Input Sequence)를 제공할 수 있다. 알고리즘은 예상 출력, 허용 오차(Tolerance), 성능 지표(Performance Metric)를 기준으로 평가할 수 있다. 이러한 테스트가 실제 센서 검증을 대체할 수는 없지만 물리적 시험실 자원을 사용하지 않고도 빈번한 회귀 검사(Regression Check)를 수행할 수 있다.

결함 주입(Fault Injection)은 정상적인 운영 조건을 넘어 통합 테스트의 범위를 확장한다. 테스트에서 의도적으로 메시지 지연, 센서 데이터 누락, 비정상 패킷(Malformed Packet), 네트워크 단절, 서비스 사용 불가, 잘못된 파라미터, 구성 요소 장애를 발생시킬 수 있다. 로봇 소프트웨어는 재시도(Retry), 성능 저하 운전(Degraded Operation), 안전 종료(Safe Shutdown), 오류 보고(Error Reporting) 등의 방식으로 예측 가능하게 대응해야 한다. 이러한 시나리오를 자동화하면 이상적인 실행 경로뿐만 아니라 시스템의 회복탄력성(Resilience)도 검증할 수 있다.

테스트 커버리지(Test Coverage)는 자동화 테스트가 소프트웨어의 어느 부분을 실행하는지 보여주지만 커버리지 비율만으로 정확성을 입증할 수는 없다. 높은 커버리지를 확보하더라도 검증 조건이 약하거나 중요한 동작 시나리오가 누락될 수 있다. 따라서 CI는 커버리지를 유일한 품질 목표가 아니라 진단 근거(Diagnostic Evidence)로 활용해야 한다. 안전 핵심(Safety-critical), 제어, 통신, 복구 로직의 변경에는 전체 커버리지가 이미 높더라도 해당 기능에 집중된 테스트가 필요할 수 있다.

불안정 테스트(Flaky Test)는 관련 소프트웨어 변경이 없는데도 때로는 성공하고 때로는 실패하기 때문에 CI의 신뢰성을 크게 저하시킨다. 주요 원인에는 타이밍 가정, 통제되지 않은 동시성, 외부 서비스, 공유 상태(Shared State), 자원 고갈(Resource Exhaustion), 비결정적 데이터(Nondeterministic Data)가 포함된다. 이러한 실패를 반복적으로 무시하면 전체 파이프라인에 대한 신뢰가 약화된다. 따라서 불안정 테스트를 단순히 성공할 때까지 재실행하기보다 식별하고 격리하여 원인을 조사하고 수정해야 한다.

병렬 테스트 실행(Parallel Test Execution)은 대규모 로보틱스 저장소의 피드백 시간을 단축할 수 있다. 독립적인 패키지 테스트나 통합 테스트 스위트(Test Suite)를 여러 CI 워커(Worker)에서 동시에 실행할 수 있으며, 종속성에 따라 일부 테스트는 순차적으로 실행하도록 구성할 수 있다. 병렬화 과정에서 공유 포트, 파일, 장치, 데이터베이스, ROS 도메인(ROS Domain)을 통한 숨겨진 간섭이 발생해서는 안 된다. 따라서 CI 인프라가 확장될수록 테스트 격리(Test Isolation)와 명시적인 자원 할당(Resource Allocation)이 중요해진다.

테스트 실패는 분석에 유용한 진단 정보를 생성해야 한다. CI 시스템은 테스트 보고서, 콘솔 로그(Console Log), ROS 2 로그, 코어 덤프(Core Dump), 트레이스(Trace), 커버리지 데이터, 구성 스냅샷(Configuration Snapshot), 선택된 런타임 아티팩트(Runtime Artifact)를 보존할 수 있다. 통합 테스트에서는 실행 당시 사용된 소프트웨어 버전과 환경을 기록하는 것도 중요하다. 정확한 구성과 연결하거나 재현할 수 없는 실패는 파이프라인이 정상적으로 실패를 표시하더라도 엔지니어링 관점에서 활용 가치가 제한적이다.

자동화 테스트는 명시적인 품질 게이트(Quality Gate)로 동작해야 한다. 필수 단위 테스트나 통합 테스트가 실패하면 승인된 절차를 통해 의도적으로 게이트를 우회하지 않는 한 해당 리비전이 릴리스 단계로 진행되어서는 안 된다. 풀 리퀘스트(Pull Request)나 병합 요청(Merge Request)은 보호 브랜치(Protected Branch)에 통합되기 전에 성공적인 테스트 상태를 요구하도록 설정할 수 있다. 이를 통해 자동화 테스트를 선택적인 개발 지원 도구가 아니라 소프트웨어 엔지니어링 프로세스의 강제 가능한 요소로 전환할 수 있다.

단위 테스트와 통합 테스트는 이후의 검증 계층과도 자연스럽게 연결되어야 한다. 호스트 기반 테스트(Host-based Test)를 통과한 소프트웨어는 시뮬레이션(Simulation), 에뮬레이션(Emulation), 하드웨어 인 더 루프(Hardware-in-the-Loop, HIL), 로봇 인 더 루프(Robot-in-the-Loop) 검증으로 진행할 수 있다. 각 계층은 서로 다른 유형의 실패를 다루므로 하나의 단계만으로는 충분하지 않다. 빠른 소프트웨어 테스트는 광범위하고 빈번한 피드백을 제공하고, 점차 현실성이 높아지는 환경은 독립적인 소프트웨어 실행만으로 재현하기 어려운 가정을 검증한다.

성숙한 CI 테스트 아키텍처(CI Testing Architecture)는 결정론적 단위 테스트, 인터페이스 인식형 통합 테스트(Interface-aware Integration Test), 통제된 테스트 픽스처, 기록 데이터, 결함 주입, 커버리지 분석(Coverage Analysis), 실패 진단(Failure Diagnostics), 강제 가능한 품질 게이트를 통합한다. 로봇 소프트웨어가 변경될 때마다 이러한 검증 메커니즘을 자동으로 실행하면 개발팀은 회귀 오류(Regression)를 발생 지점 가까이에서 발견하고, 이미 강력한 소프트웨어 수준의 기준선(Software-level Baseline)을 확보한 리비전에 대해서만 비용이 높은 시뮬레이션 및 실제 로봇 자원을 투입할 수 있다.

## 2.7. Static Analysis and Linting Gates in CI [w/Code]

![](images/image7.png){width="7.268055555555556in" height="7.268055555555556in"}

정적 분석(Static Analysis)과 린팅(Linting)은 전체 로봇 시스템을 실행하지 않고 소스 코드를 검사함으로써 지속적 통합(Continuous Integration, CI)의 초기 품질 관리 계층(Quality-control Layer)을 제공한다. 로보틱스 프로젝트에서는 이러한 검사를 통해 비용이 높은 시뮬레이션이나 하드웨어 테스트가 시작되기 전에 프로그래밍 결함, 안전하지 않은 코드 구조, 일관되지 않은 인터페이스, 스타일 위반, 유지보수성 문제를 발견할 수 있다. CI 게이트(CI Gate)는 이러한 검사를 선택적인 개발 도구가 아니라 모든 관련 소프트웨어 변경에 적용되는 반복 가능한 요구사항으로 전환한다.

린팅(Linting)은 주로 소스 코드의 일관성, 코딩 규칙(Coding Convention), 일반적인 프로그래밍 오류를 검사하는 데 초점을 둔다. 린터(Linter)는 사용되지 않는 변수, 의심스러운 표현식, 일관되지 않은 포매팅(Formatting), 명명 규칙 위반, 불필요한 임포트(Import), 결함 가능성이 높은 코드 패턴을 식별할 수 있다. 개별적으로는 사소해 보이는 문제라도 일관된 규칙을 적용하면 코드 리뷰(Code Review) 과정의 모호성을 줄이고 대규모 ROS 2, 임베디드, 인식(Perception), 제어(Control) 코드베이스에 저수준 문제가 누적되는 것을 방지할 수 있다.

정적 분석(Static Analysis)은 일반적인 런타임 조건에서 애플리케이션을 실행하지 않고 프로그램의 구조와 동작을 더욱 깊이 분석한다. 분석 도구에 따라 널 포인터 역참조(Null-pointer Dereference), 초기화되지 않은 값, 메모리 오용(Memory Misuse), 자원 누수(Resource Leak), 도달 불가능 코드(Unreachable Code), 잘못된 형 변환(Type Conversion), 정수 연산 문제, 동시성 위험(Concurrency Hazard), 정의된 프로그래밍 규칙 위반 등을 탐지할 수 있다. 이러한 결함은 오류가 실제 물리적 동작으로 이어질 수 있는 로봇 소프트웨어에서 특히 중요하다.

CI 품질 게이트(Quality Gate)는 분석 결과를 기준으로 소프트웨어 리비전(Software Revision)이 파이프라인의 다음 단계로 진행할 수 있는지를 결정한다. 컴파일 경고, 린트 위반(Lint Violation), 심각한 분석 결과 또는 정책 위반이 정의된 기준을 초과하면 변경 사항을 거부하도록 구성할 수 있다. 이러한 게이트를 단위 테스트(Unit Test), 통합 테스트(Integration Test), 시뮬레이션(Simulation), 하드웨어 인 더 루프(Hardware-in-the-Loop, HIL) 검증보다 앞에 배치하면 비용이 적게 드는 단계에서 발견할 수 있는 결함을 먼저 제거할 수 있다.

서로 다른 프로그래밍 언어에는 서로 다른 분석 전략이 필요하다. C와 C++는 ROS 2 노드(Node), 실시간 제어(Real-time Control), 디바이스 드라이버(Device Driver), 임베디드 펌웨어(Embedded Firmware)에 널리 사용되며, Python은 오케스트레이션(Orchestration), AI 처리, 개발 도구, 실험에 자주 사용된다. 따라서 CI 파이프라인은 각 언어에 적합한 분석기(Analyzer)와 린터를 선택하면서 결과는 일관된 품질 관리 프로세스를 통해 처리해야 한다. 목적은 분석 도구 자체를 통일하는 것이 아니라 엔지니어링 요구사항을 일관되게 적용하는 것이다.

컴파일러 진단(Compiler Diagnostic)은 첫 번째 정적 품질 계층을 구성할 수 있다. 최신 컴파일러는 의심스러운 형 변환, 사용되지 않는 값, 문제가 될 수 있는 제어 흐름(Control Flow), 더 이상 권장되지 않는 기능(Deprecated Feature), 기타 잠재적으로 안전하지 않은 코드 구조를 탐지한다. 프로젝트에서는 더 엄격한 경고 수준(Warning Level)을 활성화하고 중요한 경고를 선택적으로 오류로 처리할 수 있다. 그러나 대상 툴체인을 고려하지 않고 가능한 모든 경고를 활성화하면 특히 x86-64, ARM, RISC-V, 임베디드 환경을 함께 지원하는 로봇 소프트웨어에서 과도한 노이즈(Noise)가 발생할 수 있다.

ROS 2 프로젝트는 린팅을 패키지 수준 테스트(Package-level Testing)와 워크스페이스 검증(Workspace Validation)에 직접 통합할 수 있다. 소스 포매팅, 저작권 검사(Copyright Check), CMake 규칙, 패키지 메타데이터(Package Metadata), Python 스타일, C++ 코딩 규칙을 각 패키지에 대해 자동으로 평가할 수 있다. 이러한 검사를 풀 리퀘스트(Pull Request) 또는 병합 요청(Merge Request) 과정에서 실행하면 통합 전에 개발자에게 피드백을 제공할 수 있으며, 서로 다른 팀이 관리하는 패키지가 하나의 저장소에 존재하더라도 일관된 엔지니어링 방식을 유지하는 데 도움이 된다.

정적 분석은 실제 빌드 구성(Build Configuration)을 이해할 때 더욱 높은 가치를 제공한다. 컴파일 명령(Compile Command), 전처리기 정의(Preprocessor Definition), 인클루드 경로(Include Path), 언어 표준(Language Standard), 대상 아키텍처 설정, 생성된 헤더(Generated Header)는 C와 C++ 코드가 해석되는 방식에 영향을 준다. 불완전한 빌드 정보를 사용하는 분석기는 거짓 양성(False Positive)을 발생시키거나 실제 결함을 놓칠 수 있다. 따라서 CI는 분석 도구를 실제 소프트웨어 아티팩트(Software Artifact)를 생성하는 데 사용되는 동일한 통제된 빌드 구성과 연결해야 한다.

임베디드 소프트웨어(Embedded Software)는 메모리, 타이밍, 하드웨어 접근이 엄격하게 제한되므로 추가적인 고려가 필요하다. 분석을 통해 위험한 포인터 연산, 암시적 형 변환(Implicit Conversion), 산술 오버플로 위험(Arithmetic Overflow Risk), 재귀(Recursion), 도달 불가능 분기, 프로젝트별 코딩 규칙 위반을 식별할 수 있다. 코딩 표준(Coding Standard)은 검증하기 어렵거나 결정론적 제어 소프트웨어에 적합하지 않은 언어 기능을 제한할 수도 있다. 자동화된 CI 적용을 통해 이러한 제한을 개발자와 대상 플랫폼 전반에 일관되게 적용할 수 있다.

보안 중심 정적 분석(Security-oriented Static Analysis)은 일반적인 코드 품질 검사(Code-quality Check)를 보완할 수 있다. 소스 스캐닝(Source Scanning)은 안전하지 않은 API 사용, 명령 구성 문제, 안전하지 않은 임시 파일, 노출된 자격 증명(Credential), 취약한 입력 검증(Input Validation), 기타 취약점 패턴(Vulnerability Pattern)을 탐지할 수 있다. 이러한 검사가 보안 테스트(Security Testing)를 대체하지는 않지만 소프트웨어가 컨테이너, 로봇 엣지 컴퓨터(Edge Computer), 플릿 인프라(Fleet Infrastructure), 외부 연결 서비스에 도달하기 전에 의심스러운 코드를 저비용으로 발견할 기회를 제공한다.

모든 경고가 자동으로 개발을 차단해야 하는 것은 아니다. 정적 분석 도구는 템플릿(Template)을 많이 사용하는 C++, 생성 코드(Generated Code), 하드웨어 추상화(Hardware Abstraction), 플랫폼별 구현에서 거짓 양성을 발생시킬 수 있다. 따라서 실용적인 게이트는 심각도 수준(Severity Level)을 구분하고 엔지니어가 특정 결과를 검토하여 허용 가능하다고 판단한 경우 문서화된 억제(Suppression)를 허용할 수 있어야 한다. 억제 규칙은 분석 범주 전체를 비활성화하기보다 최소 범위로 제한하고 추적 가능하며 버전 관리되는 형태로 유지해야 한다.

기준선 관리(Baseline Management)는 기존에 많은 정적 분석 결과가 존재하는 코드베이스에 분석 체계를 새롭게 도입할 때 유용하다. 모든 기존 경고를 즉시 수정하도록 요구하면 개발팀이 품질 게이트 자체를 도입하기 어려울 수 있다. 기준선(Baseline)은 기존에 허용된 분석 결과를 기록하면서 새로 추가되거나 수정된 코드에서는 추가적인 위반이 발생하지 않도록 요구할 수 있다. 이후 기술 부채(Technical Debt)를 수정하면서 기준선을 점진적으로 줄여 코드 품질이 더 이상 악화되지 않도록 관리할 수 있다.

변경 코드 분석(Changed-code Analysis)은 매우 큰 저장소에서 피드백 주기를 단축할 수 있다. 풀 리퀘스트에서는 현재 변경의 영향을 받는 파일이나 패키지를 우선 분석하고, 예약 파이프라인(Scheduled Pipeline)이나 메인 브랜치(Main Branch) 파이프라인에서는 전체 저장소 검사를 수행할 수 있다. 이러한 계층적 접근은 개발자에게 빠른 피드백을 제공하면서 광범위한 회귀 탐지(Regression Detection)를 유지한다. 다만 로컬 소스 변경이 직접 수정되지 않은 종속 구성 요소에도 영향을 줄 수 있다는 점을 고려해야 한다.

기계 판독 가능한 보고서(Machine-readable Report)는 개별 CI 작업의 콘솔 출력 이상으로 분석 결과를 활용할 수 있도록 한다. 분석 결과를 소스 파일, 줄 번호(Line Number), 심각도, 규칙 식별자(Rule Identifier), 설명 메시지와 연결할 수 있다. CI 플랫폼은 이러한 결과를 풀 리퀘스트나 병합 요청에 직접 표시할 수 있으며, 보관된 보고서는 이후 조사에 필요한 근거를 제공한다. 명확한 보고는 개발자가 문제 위치를 찾는 시간을 줄이고 검토자가 품질 게이트가 실패한 이유를 이해하도록 지원한다.

추세 모니터링(Trend Monitoring)은 개별 파이프라인 결과보다 넓은 관점을 제공한다. 개발팀은 릴리스에 따른 경고 개수, 결함 범주, 중복 코드(Duplicated Code), 복잡도 지표(Complexity Indicator), 억제 규칙 증가, 반복적인 위반을 추적할 수 있다. 목적은 임의의 수치 점수 자체를 최적화하는 것이 아니라 유지보수성(Maintainability)이나 결함 위험이 악화되는 영역을 식별하는 것이다. 특정 지표가 갑자기 증가하면 개별 코드 리뷰만으로 발견하기 어려운 아키텍처 문제를 나타낼 수 있다.

정적 분석은 동적 테스트(Dynamic Testing)를 대체하는 것이 아니라 함께 사용되어야 한다. 소스 분석기는 잠재적인 메모리 오류를 발견할 수 있지만 런타임 타이밍, ROS 2 통신 동작, 센서 상호작용, 실제 로봇의 물리적 동역학(Physical Dynamics)을 완전히 검증할 수 없다. 반대로 단위 테스트와 통합 테스트는 선택된 시나리오만 실행하기 때문에 숨겨진 위험 코드 경로(Unsafe Code Path)를 실행하지 못할 수 있다. 정적 검사와 자동 실행 테스트를 결합하면 어느 한 방식만 사용할 때보다 더 강력한 검증 근거를 확보할 수 있다.

품질 게이트는 개발 생산성(Developer Productivity)도 유지해야 한다. 빠른 린팅과 대상화된 정적 검사(Targeted Static Check)는 파이프라인 초기에 실행하고, 더 깊은 전체 프로그램 분석(Whole-program Analysis)은 이후 단계 또는 병렬로 수행할 수 있다. 캐싱(Caching), 증분 분석(Incremental Analysis), 분산 CI 워커(Distributed CI Worker), 신중하게 범위를 설정한 규칙을 통해 피드백 시간을 줄일 수 있다. 지나치게 오래 걸리거나 관련성이 낮은 경고를 대량으로 생성하는 게이트는 개발자가 분석 결과를 신뢰하지 않거나 우회하게 만들어 본래의 엔지니어링 통제 기능을 약화시킬 수 있다.

성숙한 CI 아키텍처(CI Architecture)는 정적 분석과 린팅을 강제 가능하고 추적 가능하며 지속적으로 관리되는 품질 게이트로 취급한다. 컴파일러 진단, 언어별 린터(Language-specific Linter), 심층 분석기(Analyzer), 보안 스캐닝(Security Scanning), 통제된 억제, 기준선, 보고 체계, 추세 모니터링이 비용이 높은 검증 단계 이전에 함께 동작한다. 이러한 접근을 통해 로보틱스 개발팀은 방지 가능한 결함이 공유 소프트웨어 기준선(Shared Software Baseline)에 포함되는 것을 차단하면서 이기종 로봇 소프트웨어 및 하드웨어 대상 전반에서 일관된 품질을 유지할 수 있다.

## 2.8. Container Image Build and Registry Push in CD [w/Code]

![](images/image8.png){width="7.268055555555556in" height="7.268055555555556in"}

컨테이너 이미지 생성(Container Image Build)은 검증된 소스 코드와 종속성을 이식 가능한 배포 단위(Portable Deployment Unit)로 변환하기 때문에 현대 로보틱스 소프트웨어의 지속적 전달(Continuous Delivery, CD)에서 핵심적인 단계이다. CI가 컴파일, 정적 분석(Static Analysis), 단위 테스트(Unit Test), 통합 테스트(Integration Test)를 완료하면 CD 파이프라인은 검증된 애플리케이션을 컨테이너 이미지(Container Image)로 패키징할 수 있다. 이 이미지는 로봇 컴퓨터, 엣지 서버(Edge Server), 시뮬레이션 시스템, 클라우드 인프라에 일관되게 배포할 수 있는 통제된 런타임 환경(Runtime Environment)을 제공한다.

컨테이너 이미지에는 애플리케이션과 함께 실행에 필요한 런타임 라이브러리(Runtime Library), 시스템 패키지(System Package), 구성 요소(Configuration Component), 지원 소프트웨어가 포함된다. 이미지는 일반적으로 Dockerfile과 같이 버전 관리되는 빌드 명세(Build Specification)를 통해 정의된다. 이러한 명세를 소스 코드처럼 관리하면 런타임 환경의 변경에도 애플리케이션 코드와 동일한 검토 및 추적성(Traceability) 절차를 적용할 수 있으며 개발, 테스트, 운영 환경 사이의 차이를 줄일 수 있다.

베이스 이미지(Base Image)는 컨테이너의 초기 운영 환경을 구성한다. 로보틱스 애플리케이션에는 Ubuntu, ROS 2, CUDA, 공급업체 소프트웨어 개발 키트(Vendor SDK), 통신 라이브러리 또는 특수 런타임 구성 요소가 필요할 수 있다. 적절한 베이스 이미지 선택은 호환성, 보안, 이미지 크기, 빌드 시간에 영향을 준다. 운영체제 버전과 미들웨어(Middleware) 환경은 대상 로봇 플랫폼과 호환되도록 유지하여 배포 과정에서 예상하지 못한 런타임 차이가 발생하지 않도록 해야 한다.

컨테이너 빌드(Container Build)는 통제되지 않은 외부 상태에 의존하기보다 재현 가능(Reproducible)해야 한다. 패키지 버전, 언어 종속성(Language Dependency), 시스템 라이브러리, 주요 빌드 도구의 버전은 가능한 범위에서 제한해야 한다. 파이프라인이 버전이 지정되지 않은 최신 패키지를 반복적으로 다운로드하면 동일한 소스 리비전(Source Revision)에서도 시점에 따라 서로 다른 이미지가 생성될 수 있다. 통제된 종속성 정의와 안정적인 베이스 이미지 참조는 배포된 로봇 소프트웨어의 재현, 진단, 롤백(Rollback) 능력을 향상시킨다.

다단계 컨테이너 빌드(Multi-stage Container Build)는 컴파일에 필요한 자원과 최종 런타임 환경을 분리할 수 있다. 초기 빌드 단계에는 컴파일러, 개발용 헤더(Development Header), ROS 2 빌드 도구 및 기타 대규모 종속성을 포함하고, 최종 단계에는 실행에 필요한 바이너리(Binary)와 런타임 구성 요소만 포함할 수 있다. 이러한 방식은 복잡한 로보틱스 애플리케이션에 대해 반복 가능한 빌드 프로세스를 유지하면서 이미지 크기와 불필요한 소프트웨어 노출을 줄일 수 있다.

CD 파이프라인은 필수 CI 품질 게이트(Quality Gate)를 통과한 소프트웨어 리비전만 이미지로 빌드해야 한다. 단위 테스트, 통합 테스트, 정적 분석 검사 또는 보안 정책(Security Policy)이 실패하면 해당 리비전이 릴리스 이미지(Release Image)로 생성되는 것을 차단해야 한다. 이러한 연계는 컨테이너화(Containerization)가 소프트웨어 검증과 분리된 독립적인 패키징 과정이 되는 것을 방지한다. 생성된 이미지는 알려져 있고 검증된 소프트웨어 기준선(Software Baseline)을 나타내야 한다.

이미지 태깅(Image Tagging)은 생성된 각 컨테이너 아티팩트(Container Artifact)에 식별 정보를 제공한다. 태그(Tag)는 소프트웨어 버전, 릴리스 채널(Release Channel), 브랜치(Branch), 커밋 식별자(Commit Identifier), 배포 환경 등을 나타낼 수 있다. 사람이 읽을 수 있는 릴리스 버전 태그는 운영 측면에서 유용하고, 불변 식별자(Immutable Identifier)는 더 강력한 추적성을 제공한다. 배포 시스템은 특정 로봇에서 실행되는 소프트웨어가 어떤 소스 리비전, CI 실행, 컨테이너 이미지와 연결되는지 정확하게 판단할 수 있어야 한다.

변경 가능한 태그(Mutable Tag)는 동일한 태그가 나중에 서로 다른 이미지 내용을 가리킬 수 있으므로 주의해서 관리해야 한다. \`latest\`와 같은 태그는 개발 과정에서는 편리하지만 운영 환경에 실제 배포된 정확한 아티팩트를 식별하는 근거로는 부족하다. 이미지 다이제스트(Image Digest)는 콘텐츠 주소 기반 식별자(Content-addressable Identifier)를 제공하며 릴리스 메타데이터(Release Metadata)와 함께 기록할 수 있다. 통제된 배포에서 불변 참조(Immutable Reference)를 사용하면 재현성을 높이고 롤백이나 장애 조사(Incident Investigation)를 더욱 신뢰성 있게 수행할 수 있다.

이미지가 게시되기 전에 자동화 검증(Automated Validation)을 통해 기능과 구성 내용을 모두 검사할 수 있다. 파이프라인은 컨테이너를 시작하고 스모크 테스트(Smoke Test)를 실행하며 필수 실행 파일을 검증하고 환경 구성을 검사하며 필요한 ROS 2 노드가 정상적으로 초기화되는지 확인할 수 있다. 이미지 스캐닝(Image Scanning)을 추가하면 아티팩트가 이후 배포 단계에서 사용되기 전에 알려진 취약점, 부적절한 패키지, 노출된 비밀정보(Secret), 구성상의 취약점을 탐지할 수 있다.

로보틱스 컨테이너는 개발 서버와 실제 배포 로봇이 서로 다른 프로세서를 사용할 수 있기 때문에 아키텍처별 빌드(Architecture-specific Build)가 필요한 경우가 많다. 하나의 소프트웨어 리비전에서 x86-64 시뮬레이션 시스템과 ARM64 엣지 컴퓨터용 이미지를 각각 생성할 수 있다. 멀티플랫폼 빌드(Multi-platform Build) 메커니즘은 이러한 변형을 조정하면서 각 이미지의 식별성을 명확하게 유지할 수 있다. 이미지가 성공적으로 생성되었다고 해서 서로 다른 대상에서 동일한 런타임 동작이 보장되는 것은 아니므로 각 아키텍처에 적합한 테스트가 필요하다.

GPU를 사용하는 로봇 애플리케이션에는 추가적인 호환성 계층(Compatibility Layer)이 존재한다. 인식(Perception), AI 추론(AI Inference), 매핑(Mapping), 시뮬레이션 컨테이너는 NVIDIA 런타임 구성 요소, CUDA 라이브러리, 아키텍처별 가속 소프트웨어에 의존할 수 있다. 컨테이너에는 호환되는 사용자 공간 종속성(User-space Dependency)을 포함하면서 적절한 장치 접근(Device Access)과 드라이버 통합(Driver Integration)은 배포 플랫폼에 의존하도록 구성할 수 있다. 따라서 CI/CD 검증은 일반적인 컨테이너 실행과 하드웨어 가속 런타임 검증을 구분해야 한다.

이미지 생성과 검증이 성공하면 이미지는 컨테이너 레지스트리(Container Registry)로 푸시(Push)된다. 레지스트리는 컨테이너 아티팩트의 중앙 집중식 저장, 버전 관리, 배포, 접근 제어 기능을 제공한다. 조직의 인프라 구성에 따라 별도의 저장소(Repository)나 네임스페이스(Namespace)를 사용하여 로봇 애플리케이션, 시뮬레이션 도구, 개발 이미지, 운영 릴리스를 구분할 수 있다. 레지스트리는 소프트웨어 생성 과정과 통제된 배포 과정 사이의 중요한 경계가 된다.

레지스트리 인증(Registry Authentication)에는 소스 저장소나 빌드 스크립트에 직접 포함된 비밀번호 또는 토큰 대신 CI에서 관리되는 자격 증명(Credential)을 사용해야 한다. 비밀정보는 필요한 작업(Job)에만 주입하고 저장소, 브랜치, 환경, 러너(Runner)의 권한에 따라 접근을 제한할 수 있다. 수명이 짧은 자격 증명(Short-lived Credential)과 최소 범위의 접근 권한을 사용하면 우발적인 노출의 영향을 줄이고 승인되지 않은 이미지 게시로부터 소프트웨어 전달 인프라를 보호하는 데 도움이 된다.

레지스트리 푸시(Registry Push)는 모든 필수 게이트가 성공한 이후에만 수행되어야 한다. 일반적인 파이프라인은 소스 검증에서 시작하여 빌드, 테스트, 이미지 생성, 이미지 스캐닝, 레지스트리 게시(Registry Publication) 순서로 진행된다. 운영 릴리스 이전에는 추가적인 승인 게이트(Approval Gate)를 배치할 수도 있다. 이러한 단계적 프로세스는 검증되지 않은 이미지가 신뢰할 수 있는 운영 네임스페이스에 들어가는 것을 방지하고 일반 개발 아티팩트와 로봇 배포 승인을 받은 소프트웨어를 구분한다.

컨테이너 레지스트리는 태그, 다이제스트, 생성 시간, 아키텍처, 관련 출처 정보(Provenance Information)를 포함하여 각 릴리스와 연결된 메타데이터를 보존할 수 있다. CD 시스템은 이미지와 연결된 소스 커밋(Source Commit), 빌드 파이프라인(Build Pipeline), 종속성 버전, 테스트 상태를 추가로 기록할 수 있다. 이러한 기록을 결합하면 소스 코드에서 CI 검증, 컨테이너 생성, 실제 배포 시스템으로 전달되는 정확한 아티팩트까지 전체 과정에 대한 추적성을 확보할 수 있다.

소프트웨어 공급망 보안(Software Supply-chain Security)은 컨테이너 이미지에 서명(Signing)하고 배포 전에 서명을 검증함으로써 강화할 수 있다. 서명을 통해 하위 시스템은 이미지가 승인된 릴리스 프로세스에서 생성되었는지, 이미지의 식별 정보가 예상된 아티팩트와 일치하는지를 확인할 수 있다. 소프트웨어 자재 명세서(Software Bill of Materials, SBOM)는 이미지에 포함된 패키지와 종속성을 추가로 설명하여 취약점 관리, 규정 준수 분석(Compliance Analysis), 이후의 보안 조사에 활용할 수 있다.

프로모션(Promotion)은 각 환경에서 애플리케이션을 다시 빌드하기보다 이미 검증된 이미지를 환경 사이에서 이동시키는 방식이 바람직하다. 동일한 불변 아티팩트(Immutable Artifact)는 필요한 승인과 검증을 통과하면서 개발 테스트 환경에서 스테이징(Staging)을 거쳐 최종적으로 운영 환경으로 진행할 수 있다. 환경 경계마다 다시 빌드하면 미세한 차이가 발생할 위험이 있지만 아티팩트 프로모션(Artifact Promotion)은 테스트한 대상과 실제 배포되는 대상 사이의 관계를 그대로 유지한다.

컨테이너 릴리스가 불변으로 관리되고 이전 이미지 버전이 유지되면 롤백(Rollback)이 단순해진다. 새로 배포한 로봇 소프트웨어 버전에서 예상하지 못한 동작이 발생하면 변경된 빌드 환경에서 과거 소스 코드를 다시 생성하는 대신 이전에 검증된 이미지를 선택할 수 있다. 그러나 효과적인 롤백을 위해서는 구성(Configuration), 영구 데이터(Persistent Data), 펌웨어(Firmware), 외부 인터페이스와의 호환성도 필요하므로 이러한 종속성 역시 명시적으로 관리해야 한다.

성숙한 컨테이너 중심 CD 파이프라인(Container-oriented CD Pipeline)은 검증된 소스 리비전, 재현 가능한 이미지 빌드, 멀티아키텍처 패키징(Multi-architecture Packaging), 보안 스캐닝, 불변 식별, 레지스트리 인증, 아티팩트 서명(Artifact Signing), 게시, 프로모션, 롤백을 하나의 과정으로 연결한다. 로보틱스 시스템에서 이러한 프로세스는 CI 검증과 이기종 로봇, 엣지 컴퓨터, 시뮬레이션 플랫폼, 클라우드 서비스로의 배포 사이에 통제된 연결을 구축하고 전체 소프트웨어 전달 수명주기(Software Delivery Lifecycle)에 걸쳐 추적성을 유지한다.

## 2.9. CD to Fleet Staged Rollout Canary Deployment [w/Code]

![](images/image9.png){width="7.268055555555556in" height="7.268055555555556in"}

로봇 플릿(Robot Fleet)에 대한 지속적 전달(Continuous Delivery, CD)은 소프트웨어 배포를 단일 장비를 넘어 다수의 로봇으로 확장한다. 검증된 릴리스(Release)는 서로 다른 하드웨어 구성, 임무, 연결 상태, 운영 가능 시간을 가진 수십 대에서 수천 대의 로봇에 전달되어야 할 수 있다. 단계적 롤아웃(Staged Rollout)은 전체 플릿을 동시에 업데이트하는 대신 소프트웨어를 점진적으로 배포하여 이전의 CI, 시뮬레이션, 하드웨어 검증 단계에서 발견되지 않은 결함이 운영 환경에 미치는 영향을 줄인다.

배포 프로세스는 필요한 CI/CD 품질 게이트(Quality Gate)를 이미 통과한 불변 릴리스 아티팩트(Immutable Release Artifact)에서 시작해야 한다. 컨테이너 이미지(Container Image), 패키지 번들(Package Bundle), 펌웨어 이미지(Firmware Image), 서명된 릴리스(Signed Release)는 소스 리비전(Source Revision), 구성(Configuration), 종속성(Dependency), 테스트 증거(Test Evidence), 배포 메타데이터(Deployment Metadata)와 연결할 수 있다. 플릿 배포에서는 소프트웨어를 다시 빌드하지 않고 검증된 바로 그 아티팩트를 프로모션(Promotion)하여 테스트된 대상과 각 로봇에서 실제 실행되는 소프트웨어 사이의 추적성(Traceability)을 유지해야 한다.

플릿 관리 시스템(Fleet Management System)은 로봇 모델, 하드웨어 리비전(Hardware Revision), 프로세서 아키텍처, 센서 구성, 운영 사이트, 고객, 임무 유형 또는 소프트웨어 호환성에 따라 로봇을 배포 그룹(Deployment Group)으로 구성할 수 있다. 이러한 그룹은 통제된 롤아웃의 기본 단위가 된다. 이후 배포 정책(Deployment Policy)을 통해 어떤 로봇이 먼저 릴리스를 적용받고, 어떤 로봇이 이전 버전을 유지하며, 배포 범위를 확대하기 전에 어떤 조건을 만족해야 하는지를 정의할 수 있다.

단계적 롤아웃(Staged Rollout)은 플릿 배포를 점차 규모가 커지는 여러 단계로 나눈다. 릴리스는 먼저 내부 테스트 로봇에 적용한 후 소규모 운영 그룹으로 확장하고, 이후 적격 플릿(Eligible Fleet)의 더 큰 비율로 확대할 수 있다. 각 단계에는 텔레메트리(Telemetry), 로그, 장애, 임무 완료율, 자원 사용량, 안전 관련 이벤트를 평가하는 관찰 기간(Observation Period)이 포함된다. 정의된 승인 기준(Acceptance Criteria)이 계속 충족되는 경우에만 다음 단계로 배포 범위를 확대한다.

카나리 배포(Canary Deployment)는 전체 플릿의 대부분에 새로운 소프트웨어를 적용하기 전에 소수의 대표적인 로봇에 먼저 배포하는 특수한 형태의 단계적 롤아웃이다. 카나리 로봇(Canary Robot)은 실제 운영 조건에서 동작하면서 실험실 환경에서 재현하기 어려운 작업 부하에 릴리스를 노출한다. 목적은 단순히 설치 성공 여부를 확인하는 것이 아니라 새로운 버전이 실제 로봇 운영 중 허용 가능한 수준으로 동작하는지를 확인한 후 더 넓은 범위로 배포하는 것이다.

적절한 카나리 로봇을 선정하는 것은 중요하다. 편리하지만 대표성이 부족한 표본은 잘못된 검증 결과를 제공할 수 있기 때문이다. 카나리 그룹(Canary Group)은 관련된 하드웨어, 센서, 환경, 작업 부하, 통신 조건의 조합을 반영해야 한다. 이기종 플릿(Heterogeneous Fleet)에서는 서로 다른 로봇 모델이나 하드웨어 세대마다 별도의 카나리가 필요할 수 있다. 위험도가 높거나 임무 중요도가 높은 배포에서는 고객이 운영하는 로봇을 포함하기 전에 전용 내부 시스템에서 먼저 검증할 수도 있다.

배포 오케스트레이션(Deployment Orchestration)은 로봇의 운영 상태를 고려해야 한다. 일반적인 서버와 달리 로봇은 주행, 충전, 물체 조작, 화물 운반을 수행하거나 작업 중단이 안전하지 않은 장소에서 운영될 수 있다. 따라서 업데이트는 주차 상태, 도킹(Docking) 상태, 충분한 배터리 충전량, 승인된 네트워크 연결, 활성 임무가 없는 상태 등 정의된 사전 조건(Precondition)이 충족될 때만 수행해야 한다. 배포 제어기(Deployment Controller)는 이러한 조건을 단순한 가정이 아니라 명시적인 게이트(Gate)로 처리해야 한다.

로봇이 무선 또는 셀룰러 네트워크를 통해 운영되는 경우에는 신뢰할 수 있는 패키지 전송(Package Transfer)도 필요하다. 소프트웨어 아티팩트는 용량이 클 수 있고 연결이 간헐적으로 끊길 수 있으며 사이트마다 대역폭(Bandwidth)이 크게 달라질 수 있다. 업데이트 메커니즘은 중단된 다운로드를 처리하고 아티팩트 무결성(Artifact Integrity)을 검증하며 불완전한 소프트웨어가 활성화되는 것을 방지해야 한다. 로컬 캐싱(Local Caching), 재개 가능한 전송(Resumable Transfer), 예약된 다운로드 시간, 대역폭 인식형 배포(Bandwidth-aware Distribution)를 통해 대규모 플릿의 네트워크 부하를 줄일 수 있다.

활성화 전에 로봇은 다운로드된 아티팩트를 검증해야 한다. 암호학적 서명(Cryptographic Signature), 해시(Hash), 릴리스 메타데이터, 아키텍처 호환성, 사용 가능한 저장 공간, 필수 런타임 버전, 구성 사전 조건을 로컬에서 검사할 수 있다. 로봇은 손상되었거나 승인되지 않았거나 호환되지 않거나 불완전한 아티팩트를 거부해야 한다. 이러한 검사는 플릿 전달 인프라(Fleet Delivery Infrastructure)와 실제 물리 시스템을 제어하게 될 소프트웨어 사이에 최종 신뢰 경계(Trust Boundary)를 형성한다.

가능한 경우 활성화(Activation)는 다운로드(Download)와 분리해야 한다. 로봇은 유휴 상태이거나 고대역폭 네트워크에 연결되어 있을 때 업데이트를 다운로드하고, 안전한 유지보수 상태(Safe Maintenance State)에 도달할 때까지 활성화를 연기할 수 있다. 이러한 분리는 운영 중단을 줄이고 플릿 관리자가 롤아웃 시점을 더욱 정밀하게 제어할 수 있도록 한다. 일부 시스템에서는 활성화 이후 컨테이너, 서비스, 미들웨어 또는 전체 로봇 컴퓨터를 통제된 방식으로 재시작해야 할 수도 있다.

배포 후 검증(Post-deployment Verification)은 업데이트된 로봇이 실제로 정상 상태인지 판단한다. 설치가 성공했다는 사실만으로는 충분하지 않다. 시스템은 프로세스 시작, ROS 2 노드 가용성(Node Availability), 예상 토픽(Topic)과 서비스(Service), 센서 연결 상태, 위치 추정(Localization) 상태, 제어기 준비 상태, CPU 및 GPU 사용률, 메모리 사용량, 네트워크 동작, 진단 상태(Diagnostic State)를 검증할 수 있다. 짧은 자동 기능 검사(Automated Functional Check)를 통해 로봇이 정상 운영으로 복귀할 준비가 되었는지 확인할 수 있다.

관측 가능성(Observability)은 카나리 배포와 단계적 배포 과정에서 필수적이다. 플릿 텔레메트리는 운영상 의미 있는 지표를 이용하여 새로운 릴리스와 이전 기준선(Baseline)을 비교해야 한다. 이러한 지표에는 소프트웨어 충돌(Crash), 재시작 빈도, 통신 장애, 임무 성공률, 위치 추정 손실, 인식 오류, 지연 시간(Latency), 자원 소비, 배터리 동작, 안전 이벤트 등이 포함될 수 있다. 배포 결정은 비공식적인 관찰만이 아니라 사전에 정의된 측정 지표를 기반으로 이루어져야 한다.

자동화된 롤아웃 게이트(Automated Rollout Gate)는 다음 배포 단계를 허용하기 전에 이러한 지표를 평가할 수 있다. 정의된 관찰 기간 동안 카나리 그룹이 허용 가능한 임계값(Threshold)을 유지하면 배포 제어기는 다음 그룹으로 릴리스를 확대할 수 있다. 오류율이나 상태 지표(Health Indicator)가 정책을 위반하면 롤아웃을 자동으로 일시 중지할 수 있다. 특히 업데이트가 안전 핵심 기능(Safety-critical Function)이나 고객 운영에 영향을 미치는 경우 특정 단계 전환에서 사람의 승인(Human Approval)을 요구할 수도 있다.

롤백(Rollback)은 배포로 인해 허용할 수 없는 동작이 발생했을 때 사용하는 주요 복구 경로(Recovery Path)를 제공한다. 로봇은 이전에 검증된 소프트웨어 버전이나 알려진 정상 릴리스(Known-good Release)에 접근할 수 있어야 한다. 플릿 시스템은 추가 롤아웃을 중단하고 영향을 받은 로봇을 해당 버전으로 복원할 수 있다. 롤백 설계에서는 구성 파일, 영구 데이터(Persistent Data), 데이터베이스 스키마(Database Schema), 펌웨어, 통신 프로토콜을 고려해야 한다. 애플리케이션 바이너리만 이전 버전으로 되돌린다고 해서 전체 호환성이 복원되는 것은 아니기 때문이다.

모든 장애가 즉각적인 플릿 전체 롤백을 요구하는 것은 아니다. 단계적 배포 시스템은 영향을 받은 그룹을 격리하면서 이전 릴리스를 실행하는 다른 로봇이 정상적으로 운영되도록 유지할 수 있다. 이를 통해 결함의 영향 범위(Blast Radius)를 제한하고 엔지니어가 로그, 텔레메트리, 트레이스(Trace), 배포 이력(Deployment History)을 조사할 시간을 확보할 수 있다. 따라서 물리적 환경에서 동작하는 자율 시스템에서는 빠르게 배포하는 능력만큼 롤아웃 진행을 일시 중지하는 능력도 중요하다.

구성 관리(Configuration Management)는 소프트웨어 롤아웃과 동기화되어야 한다. 새로운 로봇 애플리케이션에는 수정된 파라미터(Parameter), 기능 플래그(Feature Flag), 지도(Map), 모델 파일(Model File), 보정 데이터(Calibration Data), 서비스 엔드포인트(Service Endpoint)가 필요할 수 있다. 이러한 종속성은 개별 로봇에서 수동으로 변경하는 대신 버전 관리하고 릴리스와 연결해야 한다. 기능 플래그를 사용하면 소프트웨어 배포와 기능 활성화(Feature Activation)를 분리하여 운영 검증이 완료될 때까지 새로운 기능을 비활성 상태로 유지할 수도 있다.

플릿 배포에서는 소프트웨어 상태에 대한 정확한 인벤토리(Inventory)를 유지해야 한다. 운영자는 각 로봇에서 어떤 릴리스, 컨테이너 다이제스트(Container Digest), 구성 버전, 펌웨어 수준, 배포 단계가 활성화되어 있는지 확인할 수 있어야 한다. 단계적 롤아웃에서는 서로 다른 버전이 동시에 운영되는 혼합 버전 운영(Mixed-version Operation)이 일반적이므로 백엔드 서비스와 로봇 간 인터페이스는 일정 기간 하위 호환성(Backward Compatibility)을 유지해야 할 수 있다. 정확한 인벤토리는 장애 조사, 유지보수 계획, 이후의 규정 준수 감사(Compliance Audit)도 지원한다.

성숙한 플릿 CD 아키텍처(Fleet CD Architecture)는 불변 아티팩트, 플릿 세분화(Fleet Segmentation), 카나리 선정, 안전한 업데이트 조건, 보안 전송(Secure Transfer), 로컬 검증(Local Verification), 단계적 활성화(Staged Activation), 관측 가능성, 자동화 게이트, 롤백, 소프트웨어 인벤토리 관리를 하나의 체계로 결합한다. 배포를 모든 로봇에 대한 단일 푸시(Push)로 처리하는 대신 소규모 운영 집단에서 얻은 증거를 바탕으로 릴리스가 더 넓은 플릿으로 안전하게 진행할 수 있는지를 결정하는 통제된 피드백 프로세스(Controlled Feedback Process)로 관리한다.

## 2.10. CI CD Pipeline Security Secret Management SBOM [w/Code]

![](images/image10.png){width="7.268055555555556in" height="7.268055555555556in"}

CI/CD 파이프라인 보안(CI/CD Pipeline Security)은 전달 시스템이 소스 코드를 실제 물리적 장비를 제어하는 소프트웨어로 변환할 수 있는 권한을 가지므로 로보틱스에서 매우 중요하다. 저장소(Repository), 빌드 러너(Build Runner), 자격 증명(Credential), 종속성(Dependency), 릴리스 아티팩트(Release Artifact) 중 하나라도 침해되면 악의적이거나 의도하지 않은 동작이 전체 로봇 플릿(Robot Fleet)으로 확산될 수 있다. 따라서 보안은 애플리케이션 코드뿐 아니라 개발과 테스트에서 패키징, 레지스트리 저장, 배포, 운영에 이르는 전체 경로를 보호해야 한다.

CI/CD 시스템은 단순한 개발 편의 도구가 아니라 운영 인프라(Production Infrastructure)로 취급해야 한다. 빌드 서버(Build Server), 러너(Runner), 아티팩트 저장소(Artifact Repository), 컨테이너 레지스트리(Container Registry), 배포 제어기(Deployment Controller), 자동화 계정(Automation Account)에는 명시적인 접근 정책과 지속적인 유지보수가 필요하다. 관리자 권한을 제한하고 소프트웨어를 최신 상태로 유지하며 네트워크 노출을 최소화해야 한다. 또한 보안 관련 작업에 대한 감사 기록(Audit Record)을 생성하여 누가 또는 어떤 시스템이 변경을 시작했는지 추적할 수 있어야 한다.

신원 및 접근 관리(Identity and Access Management, IAM)는 파이프라인 보안의 기반을 제공한다. 개발자, 서비스, CI 러너, 배포 에이전트(Deployment Agent)에는 각자의 책임을 수행하는 데 필요한 최소한의 권한만 부여해야 한다. 보호 브랜치(Protected Branch), 필수 검토(Mandatory Review), 역할 기반 접근 제어(Role-based Access Control, RBAC), 환경별 권한을 활용하면 하나의 계정이 침해되더라도 소스 코드를 수정하고 즉시 로봇에 배포하는 것을 방지할 수 있다. 운영 환경의 특권 작업(Privileged Production Action)에는 더욱 강력한 인증이나 추가 승인을 요구할 수 있다.

비밀정보(Secret)는 패키지 저장소, 클라우드 서비스, 컨테이너 레지스트리, 서명 시스템(Signing System), 배포 API, 로봇 플릿 인프라에 접근하기 위해 자주 필요하다. 이러한 자격 증명을 소스 코드, Dockerfile, 구성 파일(Configuration File), 저장소에 커밋되는 CI 정의에 직접 포함해서는 안 된다. 비밀정보가 버전 이력(Version History)에 한번 포함되면 보이는 문자열을 삭제하는 것만으로는 충분하지 않을 수 있다. 이전 리비전, 캐시(Cache), 로그 또는 복제된 저장소에 해당 자격 증명이 계속 남아 있을 수 있기 때문이다.

전용 비밀정보 관리(Secret Management) 메커니즘을 사용하면 CI 작업이 필요한 시점에만 자격 증명을 얻도록 할 수 있다. 비밀정보는 보호된 CI 변수(Protected CI Variable), 외부 비밀정보 저장소(External Secret Store), 플랫폼별 자격 증명 시스템에 저장하고 승인된 작업에만 런타임(Runtime) 시점에 주입할 수 있다. 접근 권한은 저장소, 브랜치, 환경, 작업 또는 신원에 따라 제한해야 한다. 운영 배포용 자격 증명이 일반적인 빌드나 풀 리퀘스트(Pull Request) 파이프라인에 자동으로 제공되어서는 안 된다.

인프라가 지원하는 경우 장기간 유지되는 정적 비밀정보보다 단기 자격 증명(Short-lived Credential)을 사용하는 것이 바람직하다. CI 워크로드(Workload)는 자신의 워크로드 신원(Workload Identity)을 사용해 인증하고 특정 레지스트리, 클라우드 서비스 또는 배포 작업에 필요한 임시 토큰(Temporary Token)을 발급받을 수 있다. 자격 증명은 제한된 시간이 지나면 만료되며 매우 좁은 범위의 권한만 갖도록 설정할 수 있다. 이를 통해 유출된 자격 증명의 가치를 낮추고 영구적인 비밀정보를 배포하고 관리하는 운영 부담을 줄일 수 있다.

파이프라인 로그(Pipeline Log)는 비밀정보가 올바르게 저장되어 있더라도 민감한 정보가 유출될 수 있으므로 신중하게 관리해야 한다. 명령이 토큰, 환경 변수(Environment Variable), 자격 증명이 포함된 URL 또는 구성 값을 실수로 출력할 수 있다. CI 시스템은 알려진 비밀정보를 마스킹(Masking)해야 하며 스크립트는 인증 정보를 노출할 수 있는 과도한 출력(Verbose Output)을 피해야 한다. 전체 환경 정보를 출력하는 디버깅 기능(Debugging Feature)은 특히 운영 권한을 가진 작업에서 비활성화하거나 엄격하게 통제해야 한다.

빌드 러너(Build Runner)는 또 하나의 중요한 신뢰 경계(Trust Boundary)를 형성한다. 공유 러너(Shared Runner)는 여러 저장소나 개발자가 제공한 코드를 실행할 수 있으며, 자체 호스팅 러너(Self-hosted Runner)는 내부 네트워크, 하드웨어 장치, 서명 키(Signing Key), 배포 시스템에 접근할 수 있다. 작업은 적절하게 격리하고 임시 작업 공간(Temporary Workspace)을 정리하며 신뢰할 수 없는 코드가 불필요한 권한으로 실행되지 않도록 해야 한다. 영구 러너(Persistent Runner)는 악성 상태가 파이프라인 실행 사이에 남아 있을 가능성이 있으므로 특별한 주의가 필요하다.

서드파티 종속성(Third-party Dependency)은 프로젝트 자체의 소스 코드가 안전하더라도 소프트웨어 공급망 위험(Software Supply-chain Risk)을 발생시킨다. 로보틱스 애플리케이션은 운영체제 패키지, ROS 2 패키지, Python 모듈, C/C++ 라이브러리, GPU 런타임, 공급업체 SDK, 펌웨어 도구, 컨테이너 베이스 이미지(Container Base Image)에 의존할 수 있다. 따라서 종속성의 버전과 출처를 통제하고 추적할 수 있어야 한다. 자동화 스캐닝(Automated Scanning)을 통해 알려진 취약점, 예상하지 못한 변경, 엔지니어링 검토가 필요한 패키지를 식별할 수 있다.

소프트웨어 자재 명세서(Software Bill of Materials, SBOM)는 릴리스 아티팩트에 포함된 소프트웨어 구성 요소에 대한 구조화된 인벤토리(Structured Inventory)를 제공한다. SBOM은 애플리케이션이나 컨테이너 이미지에 포함된 패키지, 라이브러리, 버전, 공급자(Supplier), 종속 관계를 기술할 수 있다. CI/CD 과정에서 SBOM을 생성하면 실제로 전달된 소프트웨어의 기계 판독 가능한 기록(Machine-readable Record)을 확보할 수 있으며 취약점 분석, 라이선스 검토(License Review), 사고 대응(Incident Response), 소프트웨어 공급망 추적성을 지원할 수 있다.

SBOM은 별도의 관련 없는 문서로 관리하기보다 정확한 릴리스 아티팩트와 연결해야 한다. 여러 아키텍처 또는 컨테이너 변형(Container Variant)이 생성되는 경우 각 아티팩트에는 서로 다른 구성 요소가 포함될 수 있으므로 각각에 대응하는 인벤토리 정보가 필요할 수 있다. SBOM을 이미지 다이제스트(Image Digest), 패키지 체크섬(Package Checksum) 또는 다른 불변 아티팩트 식별자(Immutable Artifact Identifier)와 연결하면 이후의 분석이 실제로 빌드되고 배포된 소프트웨어를 대상으로 수행되도록 할 수 있다.

취약점 스캐닝(Vulnerability Scanning)은 SBOM 데이터를 사용하거나 아티팩트를 직접 검사하여 알려진 보안 문제와 관련된 구성 요소를 식별할 수 있다. 발견된 문제는 단순한 취약점 개수만으로 판단하기보다 심각도(Severity), 악용 가능성(Exploitability), 배포 환경(Deployment Context), 사용 가능한 대응 방법을 기준으로 평가해야 한다. 파이프라인은 정의된 정책에 따라 치명적인 취약점 발견 시 진행을 차단하고, 즉각적인 수정이 불가능하면서 운영 위험이 검토된 경우에는 문서화된 예외(Documented Exception)를 허용할 수 있다.

빌드가 완료된 이후에도 아티팩트 무결성(Artifact Integrity)을 유지해야 한다. 체크섬과 콘텐츠 주소 기반 식별자(Content-addressable Identifier)는 변경 여부를 탐지할 수 있으며, 디지털 서명(Digital Signature)은 아티팩트가 승인된 프로세스에서 생성되거나 승인되었다는 근거를 제공할 수 있다. 배포 시스템은 설치 전에 이러한 속성을 검증할 수 있다. 로봇 플릿에서는 대상 장치에서 직접 검증함으로써 손상되거나 대체되었거나 승인되지 않은 소프트웨어가 실제 물리 플랫폼에 도달하는 것을 막는 최종 통제 계층을 구축할 수 있다.

서명 키(Signing Key)가 침해되면 공격자가 정상적인 아티팩트처럼 보이는 소프트웨어를 생성할 수 있으므로 일반적인 애플리케이션 비밀정보보다 더욱 강력하게 보호해야 한다. 키는 범용 빌드 작업에서 격리하고 통제된 서명 작업(Signing Operation)에만 노출해야 한다. 하드웨어 기반(Hardware-backed) 또는 관리형 서명 서비스(Managed Signing Service)를 사용하면 키의 직접적인 노출을 더욱 줄일 수 있다. 또한 자격 증명이 침해된 것으로 의심될 때 신뢰 체계를 복원할 수 있도록 키 교체(Key Rotation), 폐기(Revocation), 감사 가능성(Auditability)을 계획해야 한다.

컨테이너 보안(Container Security)은 Dockerfile에서 실제 배포된 런타임까지 확장되어야 한다. 파이프라인은 베이스 이미지와 설치된 패키지를 스캔하고, 포함된 자격 증명을 탐지하며, 불필요한 소프트웨어를 최소화하고, 애플리케이션이 과도한 권한으로 실행되지 않도록 할 수 있다. 이미지는 불변 다이제스트(Immutable Digest)로 식별하고 접근이 통제되는 레지스트리에 저장해야 한다. 운영 저장소는 승인된 파이프라인에서 생성된 아티팩트만 허용하여 개발자가 임의로 생성한 이미지가 정식 릴리스 절차를 우회하지 못하도록 해야 한다.

파이프라인 구성(Pipeline Configuration) 자체도 보안에 민감한 코드이다. 워크플로 파일(Workflow File), Jenkinsfile, GitLab CI 정의, 배포 매니페스트(Deployment Manifest), 자동화 스크립트는 어떤 명령을 실행하고 어떤 비밀정보 또는 인프라에 접근할 수 있는지를 결정한다. 따라서 이러한 파일의 변경에도 적절한 검토와 브랜치 보호(Branch Protection)를 적용해야 한다. 겉보기에는 작은 파이프라인 변경이라도 권한을 변경하거나 보안 검사를 비활성화하고 자격 증명을 노출하거나 릴리스 아티팩트를 승인되지 않은 위치로 전달할 수 있다.

보안 게이트(Security Gate)는 하나의 최종 스캔 단계에 집중하기보다 CI/CD 전체에 배치해야 한다. 초기 단계에서는 소스 코드, 자격 증명, 종속성, 구성을 검사하고 이후 단계에서는 바이너리(Binary), 컨테이너, SBOM, 서명, 배포 메타데이터를 검사할 수 있다. 심각도가 높은 문제가 발견되면 진행을 자동으로 중단할 수 있다. 이러한 계층형 통제(Layered Control)는 하나의 보안 도구에 대한 의존도를 줄이고 단일 단계에서 놓친 결함이 운영 로봇까지 전달될 가능성을 낮춘다.

추적성(Traceability)과 감사 가능성(Auditability)은 이러한 통제 요소를 하나의 운영 보안 시스템으로 연결한다. 릴리스 기록(Release Record)을 통해 소스 리비전, 파이프라인 실행, 빌드 환경, 종속성, 테스트 결과, SBOM, 취약점 상태, 아티팩트 다이제스트, 서명, 승인 이력(Approval History), 배포 대상을 식별할 수 있어야 한다. 이후 취약점이나 보안 사고가 발견되면 운영자는 영향을 받는 로봇 버전을 정확하게 파악하고 훨씬 높은 정밀도로 수정 배포(Remediation) 또는 롤백(Rollback)을 수행할 수 있다.

성숙한 로보틱스 CI/CD 보안 아키텍처(CI/CD Security Architecture)는 최소 권한 접근(Least-privilege Access), 보호된 파이프라인(Protected Pipeline), 격리된 러너(Isolated Runner), 안전한 비밀정보 관리, 통제된 종속성, SBOM 생성, 취약점 스캐닝, 불변 아티팩트(Immutable Artifact), 서명, 검증, 완전한 감사 기록을 하나의 체계로 결합한다. 이러한 메커니즘은 소스 코드에서 실제 배포된 로봇까지 이어지는 신뢰 사슬(Chain of Trust)을 구축하여 플릿 규모로 전달되는 소프트웨어가 식별 가능하고 승인되었으며 검사 가능하고 공급망 침해(Supply-chain Compromise)에 대한 저항성을 갖도록 지원한다.
