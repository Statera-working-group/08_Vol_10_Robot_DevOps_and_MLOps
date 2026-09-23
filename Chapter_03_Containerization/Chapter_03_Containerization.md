**Volume 10 Robot DevOps and MLOps**


# 3. Containerization

##  

## 3.1. Container Fundamentals Namespaces Cgroups Union FS

![](images/image1.png){width="7.268055555555556in" height="7.268055555555556in"}

Containerization provides a lightweight method for packaging software together with its runtime dependencies while isolating it from other processes on the same operating system. Unlike a virtual machine, a container normally does not include a complete guest operating system. Multiple containers share the host kernel while maintaining logically separated execution environments.

This model is particularly useful for robotics because a robot software stack often combines ROS 2 packages, perception libraries, AI frameworks, device interfaces, middleware, and application-specific dependencies. Packaging these components into containers reduces differences between development, testing, simulation, edge deployment, and fleet operation environments while improving reproducibility.

A container should not be understood as a single Linux kernel feature. Linux container isolation emerges from several mechanisms working together, especially namespaces, control groups, filesystem isolation, capabilities, and security policies. Container runtimes coordinate these mechanisms to create an environment that appears to an application like an independent system even though it remains part of the host.

Linux namespaces provide the fundamental mechanism for separating the resources visible to processes. When a process is placed inside a namespace, its view of a particular system resource can differ from the view available to processes outside that namespace. Containers commonly combine several namespace types so that processes, networking, mount points, host names, users, and interprocess communication can be isolated independently.

The PID namespace isolates process identifiers and process trees. A process may appear as PID 1 inside a container while having a completely different PID on the host. This creates an independent process hierarchy for the container and prevents ordinary container processes from seeing every host process. PID 1 also has special responsibilities for signal handling and orphaned child processes.

Network namespaces isolate networking resources such as interfaces, routing tables, firewall rules, and sockets. A container can therefore possess its own virtual network interface and IP configuration while sharing the same physical host. Container runtimes can connect network namespaces through virtual Ethernet devices, software bridges, host networking, or more specialized networking mechanisms.

Mount namespaces provide independent views of filesystem mount points. They allow a container to see a filesystem hierarchy that differs from the host hierarchy even though both use the same kernel. Combined with layered container filesystems and bind mounts, mount namespaces make it possible to expose only the files and directories required by an application while attaching persistent or hardware-related paths when necessary.

UTS namespaces isolate host and domain names, while IPC namespaces separate mechanisms such as System V IPC and POSIX message queues. User namespaces can map user and group identifiers inside a container to different identifiers on the host. This is especially valuable for reducing privileges because a process that appears to run as root inside a container does not necessarily need equivalent root privileges on the host.

Namespaces primarily determine what a container can see, but they do not by themselves control how much hardware capacity it can consume. Linux control groups, commonly called cgroups, provide resource accounting, limitation, and prioritization. They can regulate resources such as CPU time, memory, process counts, and I/O behavior, allowing multiple workloads to share a host without unrestricted competition.

CPU controls can constrain the processor capacity available to a container or influence scheduling priority among workloads. Memory controls can establish limits and track consumption so that one application cannot freely consume all available system memory. These mechanisms become important on robot computers where localization, perception, planning, logging, and AI inference may execute concurrently under strict resource constraints.

Modern Linux systems increasingly use cgroup v2, which provides a unified hierarchy and more consistent resource-control model than the multiple hierarchies associated with earlier cgroup implementations. Container engines and orchestration platforms translate higher-level resource configurations into cgroup controls, allowing deployment policies to specify resource boundaries without applications directly managing kernel interfaces.

Filesystem layering addresses another requirement of containerization: efficiently distributing application environments without copying a complete filesystem for every container. Container images are commonly constructed from multiple read-only layers. Each layer represents filesystem changes, and the runtime combines these layers into a unified view that appears to the application as a normal directory hierarchy.

This concept is often described through union or layered filesystem behavior. Technologies such as OverlayFS can combine lower read-only layers with an upper writable layer. When a container modifies a file originating from a lower layer, copy-on-write behavior allows the modification to be represented in the writable layer rather than altering the original image layer. Unchanged data can therefore remain shared.

Layering makes container images efficient to build, distribute, cache, and reuse. For example, several robot applications may share an Ubuntu base layer, common ROS 2 libraries, CUDA components, and organization-wide middleware packages while differing only in their application layers. A registry and local runtime can reuse existing layers instead of repeatedly transferring identical content.

The writable container layer should generally be considered ephemeral rather than permanent application storage. When a container is deleted and recreated, data stored only in that layer may disappear. Persistent robot configuration, maps, calibration information, recorded sensor data, databases, and operational logs therefore require deliberate storage strategies such as volumes, bind mounts, or external storage services.

Container images and running containers represent related but different concepts. An image is an immutable package describing the filesystem and execution metadata required to start an application. A container is a runtime instance created from that image with additional writable state, namespaces, cgroups, networking, and other runtime configuration. Many containers can consequently be instantiated from the same image.

The container runtime is responsible for converting an image and its configuration into an executing process environment. Modern container ecosystems commonly separate high-level image management and user interfaces from lower-level runtime functions. Open Container Initiative specifications help standardize image formats and runtime behavior, reducing dependence on a single implementation and improving portability.

For robotics, container boundaries should follow operational requirements rather than simply placing the entire robot software stack into one large container. Perception, navigation, fleet connectivity, diagnostics, or AI inference may require different dependencies and update cycles. Separating appropriate services can reduce dependency conflicts and enable independent deployment, although excessive fragmentation increases communication and management complexity.

Hardware access requires special consideration because container isolation can hide devices that robot applications need. Cameras, LiDAR interfaces, serial devices, CAN adapters, GPUs, and other accelerators must be deliberately exposed to the container. Device permissions, users, kernel drivers, runtime configuration, and host libraries must therefore be coordinated instead of assuming that containerization completely abstracts hardware.

Containers also do not provide the same isolation boundary as conventional virtual machines. Because containers share the host kernel, kernel vulnerabilities, excessive privileges, exposed devices, or unsafe runtime configurations can weaken isolation. Production robot systems should consequently combine namespaces and cgroups with Linux capabilities, seccomp filtering, mandatory access controls, non-root execution, and restricted filesystem permissions.

Together, namespaces, cgroups, and layered filesystems explain the core container model. Namespaces define isolated views of system resources, cgroups regulate resource consumption, and layered filesystems provide efficient application packaging and copy-on-write storage. Container runtimes integrate these mechanisms into a repeatable deployment unit suitable for development computers, edge devices, robot platforms, and fleet infrastructure.

In a DevOps-oriented robotics architecture, this foundation enables the same versioned software artifact to move through CI testing, simulation, hardware-in-the-loop validation, registry storage, and robot deployment with fewer environmental differences. Containerization therefore becomes not merely an application packaging technique, but a fundamental building block for reproducible robot software delivery and lifecycle management.

컨테이너화(Containerization)는 소프트웨어를 실행에 필요한 의존성(Runtime Dependencies)과 함께 패키징하면서도 동일한 운영체제(Operating System)의 다른 프로세스로부터 격리하는 경량화된 방법을 제공한다. 가상 머신(Virtual Machine)과 달리 컨테이너(Container)는 일반적으로 완전한 게스트 운영체제(Guest Operating System)를 포함하지 않는다. 여러 컨테이너가 호스트 커널(Host Kernel)을 공유하면서 논리적으로 분리된 실행 환경을 유지한다.

이러한 모델은 로봇 소프트웨어 스택(Robot Software Stack)이 ROS 2 패키지, 인식 라이브러리(Perception Library), 인공지능 프레임워크(AI Framework), 장치 인터페이스(Device Interface), 미들웨어(Middleware), 응용 프로그램별 의존성(Application-specific Dependency)을 함께 사용하는 경우가 많기 때문에 로보틱스(Robotics)에 특히 유용하다. 이러한 구성요소를 컨테이너로 패키징하면 개발, 시험, 시뮬레이션, 엣지 배포(Edge Deployment), 플릿 운영(Fleet Operation) 환경 간 차이를 줄이고 재현성(Reproducibility)을 향상시킬 수 있다.

컨테이너는 단일 리눅스 커널 기능(Linux Kernel Feature)으로 이해해서는 안 된다. 리눅스 컨테이너 격리(Linux Container Isolation)는 여러 메커니즘이 함께 동작하면서 구현되며, 대표적으로 네임스페이스(Namespaces), 제어 그룹(Control Groups), 파일 시스템 격리(Filesystem Isolation), 권한 기능(Capabilities), 보안 정책(Security Policies)이 사용된다. 컨테이너 런타임(Container Runtime)은 이러한 메커니즘을 조정하여 응용 프로그램에서 독립적인 시스템처럼 보이는 환경을 생성한다.

리눅스 네임스페이스(Linux Namespaces)는 프로세스에서 볼 수 있는 자원을 분리하는 기본 메커니즘을 제공한다. 프로세스를 특정 네임스페이스에 배치하면 특정 시스템 자원에 대한 해당 프로세스의 관점이 네임스페이스 외부 프로세스의 관점과 달라질 수 있다. 컨테이너는 일반적으로 여러 종류의 네임스페이스를 조합하여 프로세스, 네트워크, 마운트 지점(Mount Point), 호스트 이름(Host Name), 사용자, 프로세스 간 통신(Interprocess Communication)을 각각 격리한다.

프로세스 ID 네임스페이스(PID Namespace)는 프로세스 식별자(Process Identifier)와 프로세스 트리(Process Tree)를 격리한다. 하나의 프로세스가 컨테이너 내부에서는 PID 1로 보이면서 호스트에서는 완전히 다른 PID를 가질 수 있다. 이를 통해 컨테이너마다 독립적인 프로세스 계층(Process Hierarchy)을 구성하며, 일반적인 컨테이너 프로세스가 호스트의 모든 프로세스를 볼 수 없도록 한다. PID 1은 신호 처리(Signal Handling)와 고아 자식 프로세스(Orphaned Child Process) 처리에서도 특별한 역할을 담당한다.

네트워크 네임스페이스(Network Namespace)는 네트워크 인터페이스(Network Interface), 라우팅 테이블(Routing Table), 방화벽 규칙(Firewall Rule), 소켓(Socket) 등의 네트워크 자원을 격리한다. 따라서 동일한 물리적 호스트를 공유하면서도 컨테이너마다 자체 가상 네트워크 인터페이스(Virtual Network Interface)와 IP 구성을 가질 수 있다. 컨테이너 런타임은 가상 이더넷(Virtual Ethernet), 소프트웨어 브리지(Software Bridge), 호스트 네트워킹(Host Networking) 등을 통해 네트워크 네임스페이스를 연결할 수 있다.

마운트 네임스페이스(Mount Namespace)는 파일 시스템 마운트 지점(Filesystem Mount Point)에 대해 독립적인 관점을 제공한다. 이를 통해 컨테이너와 호스트가 동일한 커널을 사용하면서도 서로 다른 파일 시스템 계층을 볼 수 있다. 계층형 컨테이너 파일 시스템(Layered Container Filesystem) 및 바인드 마운트(Bind Mount)와 결합하면 응용 프로그램에 필요한 파일과 디렉터리만 노출하면서 필요에 따라 영구 저장소나 하드웨어 관련 경로를 연결할 수 있다.

UTS 네임스페이스(UTS Namespace)는 호스트 이름과 도메인 이름을 격리하고, IPC 네임스페이스(IPC Namespace)는 System V IPC 및 POSIX 메시지 큐(Message Queue)와 같은 메커니즘을 분리한다. 사용자 네임스페이스(User Namespace)는 컨테이너 내부의 사용자 및 그룹 식별자(User and Group Identifier)를 호스트의 다른 식별자로 매핑할 수 있다. 이를 통해 컨테이너 내부에서 루트(Root)로 보이는 프로세스라도 호스트에서 동일한 루트 권한을 가질 필요가 없도록 구성할 수 있다.

네임스페이스는 주로 컨테이너가 무엇을 볼 수 있는지를 결정하지만, 컨테이너가 사용할 수 있는 하드웨어 자원의 양까지 직접 제어하지는 않는다. 리눅스 제어 그룹(Linux Control Groups), 즉 Cgroups는 자원의 사용량 측정(Resource Accounting), 제한(Limitation), 우선순위 지정(Prioritization)을 제공한다. CPU 시간, 메모리, 프로세스 수, 입출력(I/O) 동작 등을 제어하여 여러 워크로드가 하나의 호스트 자원을 무제한으로 경쟁하지 않도록 한다.

CPU 제어(CPU Control)를 사용하면 컨테이너에서 사용할 수 있는 프로세서 용량을 제한하거나 워크로드 사이의 스케줄링 우선순위(Scheduling Priority)를 조정할 수 있다. 메모리 제어(Memory Control)는 사용량을 추적하고 제한을 설정하여 하나의 응용 프로그램이 시스템 메모리를 과도하게 사용하는 것을 방지한다. 이러한 기능은 위치 추정(Localization), 인식(Perception), 경로 계획(Planning), 로깅(Logging), AI 추론(AI Inference)이 동시에 실행되는 로봇 컴퓨터에서 특히 중요하다.

현대적인 리눅스 시스템에서는 이전 Cgroup 구현의 여러 계층 구조보다 통합되고 일관된 자원 제어 모델을 제공하는 Cgroup v2가 점차 일반적으로 사용되고 있다. 컨테이너 엔진(Container Engine)과 오케스트레이션 플랫폼(Orchestration Platform)은 상위 수준의 자원 설정을 Cgroup 제어로 변환한다. 따라서 응용 프로그램이 커널 인터페이스를 직접 관리하지 않고도 배포 정책을 통해 자원 경계(Resource Boundary)를 지정할 수 있다.

파일 시스템 계층화(Filesystem Layering)는 모든 컨테이너마다 완전한 파일 시스템을 복사하지 않고 응용 프로그램 환경을 효율적으로 배포하기 위한 메커니즘이다. 컨테이너 이미지(Container Image)는 일반적으로 여러 개의 읽기 전용 계층(Read-only Layer)으로 구성된다. 각 계층은 파일 시스템의 변경 사항을 나타내며, 런타임은 이 계층들을 통합하여 응용 프로그램에서는 일반적인 디렉터리 구조처럼 보이도록 한다.

이러한 개념은 일반적으로 유니온 또는 계층형 파일 시스템(Union or Layered Filesystem) 동작으로 설명된다. OverlayFS와 같은 기술은 하위 읽기 전용 계층(Lower Read-only Layer)과 상위 쓰기 가능 계층(Upper Writable Layer)을 결합할 수 있다. 컨테이너가 하위 계층의 파일을 수정하면 쓰기 시 복사(Copy-on-Write) 방식으로 원본 이미지 계층을 변경하지 않고 쓰기 가능 계층에 변경 내용을 기록할 수 있다.

계층화(Layering)는 컨테이너 이미지를 효율적으로 빌드하고 배포하며 캐시하고 재사용할 수 있게 한다. 예를 들어 여러 로봇 응용 프로그램이 동일한 Ubuntu 기본 계층(Base Layer), 공통 ROS 2 라이브러리, CUDA 구성요소, 조직 공통 미들웨어 패키지를 공유하면서 응용 프로그램 계층만 서로 다르게 구성할 수 있다. 레지스트리(Registry)와 로컬 런타임(Local Runtime)은 동일한 데이터를 반복적으로 전송하지 않고 기존 계층을 재사용할 수 있다.

쓰기 가능한 컨테이너 계층(Writable Container Layer)은 일반적으로 영구적인 응용 프로그램 저장소가 아니라 일시적인 저장 영역(Ephemeral Storage)으로 간주해야 한다. 컨테이너가 삭제되고 다시 생성되면 해당 계층에만 저장된 데이터는 사라질 수 있다. 따라서 로봇 설정, 지도, 보정 정보(Calibration Information), 센서 기록 데이터, 데이터베이스, 운영 로그 등은 볼륨(Volume), 바인드 마운트(Bind Mount), 외부 저장 서비스(External Storage Service)와 같은 별도의 저장 전략이 필요하다.

컨테이너 이미지(Container Image)와 실행 중인 컨테이너(Running Container)는 서로 관련되어 있지만 다른 개념이다. 이미지는 응용 프로그램을 시작하는 데 필요한 파일 시스템과 실행 메타데이터(Runtime Metadata)를 정의하는 불변 패키지(Immutable Package)이다. 컨테이너는 이미지로부터 생성된 실행 인스턴스(Runtime Instance)로, 쓰기 가능 상태와 네임스페이스, Cgroups, 네트워킹 및 기타 런타임 구성이 추가된다. 따라서 하나의 이미지에서 여러 컨테이너를 생성할 수 있다.

컨테이너 런타임(Container Runtime)은 이미지와 해당 설정을 실제 실행 프로세스 환경으로 변환한다. 현대적인 컨테이너 생태계(Container Ecosystem)는 상위 수준의 이미지 관리 및 사용자 인터페이스와 하위 수준의 런타임 기능을 분리하는 경우가 많다. 오픈 컨테이너 이니셔티브(Open Container Initiative, OCI)의 명세는 이미지 형식과 런타임 동작을 표준화하여 특정 구현에 대한 의존성을 줄이고 이식성(Portability)을 향상시킨다.

로보틱스에서 컨테이너 경계(Container Boundary)는 전체 로봇 소프트웨어 스택을 하나의 대형 컨테이너에 단순히 배치하기보다 운영 요구사항(Operational Requirements)을 기준으로 설계해야 한다. 인식, 내비게이션(Navigation), 플릿 연결(Fleet Connectivity), 진단(Diagnostics), AI 추론 등은 서로 다른 의존성과 업데이트 주기를 가질 수 있다. 적절한 서비스 분리는 의존성 충돌을 줄이고 독립적인 배포를 가능하게 하지만 지나친 세분화는 통신 및 관리 복잡성을 증가시킨다.

하드웨어 접근(Hardware Access)은 컨테이너 격리로 인해 로봇 응용 프로그램에 필요한 장치가 숨겨질 수 있기 때문에 특별한 고려가 필요하다. 카메라, LiDAR 인터페이스, 직렬 장치(Serial Device), CAN 어댑터, GPU 및 기타 가속기(Accelerator)는 컨테이너에 의도적으로 노출해야 한다. 따라서 장치 권한(Device Permission), 사용자, 커널 드라이버(Kernel Driver), 런타임 설정, 호스트 라이브러리(Host Library)를 함께 조정해야 한다.

컨테이너는 기존 가상 머신과 동일한 수준의 격리 경계(Isolation Boundary)를 제공하지 않는다. 컨테이너가 호스트 커널을 공유하기 때문에 커널 취약점(Kernel Vulnerability), 과도한 권한(Excessive Privilege), 노출된 장치 또는 안전하지 않은 런타임 설정이 격리 수준을 약화시킬 수 있다. 따라서 실제 로봇 시스템에서는 네임스페이스와 Cgroups를 리눅스 권한 기능(Linux Capabilities), Seccomp 필터링(Seccomp Filtering), 강제 접근 제어(Mandatory Access Control), 비루트 실행(Non-root Execution), 제한된 파일 시스템 권한과 함께 적용해야 한다.

네임스페이스(Namespaces), Cgroups, 계층형 파일 시스템(Layered Filesystems)은 함께 컨테이너의 핵심 동작 모델을 구성한다. 네임스페이스는 시스템 자원에 대한 격리된 관점을 정의하고, Cgroups는 자원 사용량을 제어하며, 계층형 파일 시스템은 효율적인 응용 프로그램 패키징과 쓰기 시 복사(Copy-on-Write) 저장 방식을 제공한다. 컨테이너 런타임은 이러한 메커니즘을 통합하여 개발 컴퓨터, 엣지 장치(Edge Device), 로봇 플랫폼, 플릿 인프라(Fleet Infrastructure)에서 반복적으로 사용할 수 있는 배포 단위(Deployment Unit)를 만든다.

데브옵스(DevOps) 중심의 로보틱스 아키텍처에서 이러한 기반 기술은 동일한 버전의 소프트웨어 산출물(Software Artifact)이 지속적 통합 시험(CI Testing), 시뮬레이션(Simulation), 하드웨어 인 더 루프 검증(Hardware-in-the-Loop Validation), 레지스트리 저장(Registry Storage), 로봇 배포(Robot Deployment) 과정을 환경 차이를 최소화하면서 이동할 수 있도록 한다. 따라서 컨테이너화(Containerization)는 단순한 응용 프로그램 패키징 기술을 넘어 재현 가능한 로봇 소프트웨어 배포와 수명주기 관리(Lifecycle Management)를 위한 핵심 기반 기술이 된다.

##  

## 3.2. Dockerfile Best Practices for Robot SW Stacks [w/Code]

![](images/image2.png){width="7.268055555555556in" height="7.268055555555556in"}

A Dockerfile defines how a container image is assembled from a reproducible sequence of instructions. For robot software stacks, it serves as executable documentation of the runtime environment, including the operating-system base, ROS 2 distribution, system libraries, middleware, AI frameworks, application dependencies, configuration, and startup behavior required by the robot application.

Choosing the base image is one of the most important Dockerfile decisions because every subsequent layer inherits its operating-system libraries and compatibility constraints. Robot projects should prefer trusted, maintained, and appropriately minimal base images while explicitly matching Ubuntu, ROS 2, CUDA, Python, and hardware-runtime requirements. Smaller images reduce transfer time and unnecessary attack surface.

Image versions should be controlled explicitly rather than relying on floating tags whenever reproducibility matters. A tag such as latest may resolve to different content at different times, causing identical Dockerfiles to produce different environments. Robot deployments benefit from fixed distribution versions, package versions where practical, and immutable image digests for releases that require precise reconstruction.

Dockerfile instructions should be organized with the image layer model in mind. Instructions such as RUN, COPY, and ADD can create filesystem changes that become part of image layers. Stable operations, including installation of rarely changing system dependencies, should generally appear before frequently changing application source files so that Docker\'s build cache can reuse expensive earlier layers.

Package installation should minimize unnecessary files and intermediate artifacts. On Debian- and Ubuntu-based robot images, package index updates and package installation should normally occur within the same RUN instruction, followed by removal of cached package lists. Recommended packages that are not actually required by the robot application should be avoided when possible to reduce image size and dependency complexity.

Robot software frequently depends on ROS 2 packages distributed through both operating-system repositories and workspace source trees. The Dockerfile should clearly separate installation of external dependencies from compilation of the robot workspace. Copying package manifests or dependency metadata before application source code can improve caching because dependency installation does not need to repeat after every source-code modification.

ROS 2 workspace builds require careful management of environment initialization. The required ROS distribution must be sourced before invoking tools such as rosdep or colcon when their behavior depends on the environment. Runtime containers must likewise initialize the ROS environment and application workspace before launching nodes, otherwise package discovery, shared libraries, message definitions, and middleware configuration may fail.

A Dockerfile should distinguish build-time dependencies from runtime dependencies. Compilers, header packages, source trees, debugging utilities, and build systems may be required to compile a ROS 2 or native C++ application but are often unnecessary during operation. Keeping these tools out of production images reduces storage requirements, startup distribution cost, maintenance burden, and potential security exposure.

This separation becomes particularly important for robot AI software because development environments can contain CUDA toolchains, Python build utilities, model conversion tools, and large intermediate artifacts. The production container may require only the resulting binaries, Python environment, inference libraries, models, and compatible GPU runtime components. Later multi-stage build techniques can formalize this separation more efficiently.

The COPY instruction should normally be preferred when files only need to be transferred from the build context into the image. ADD provides additional behaviors that are unnecessary for many ordinary robot software builds. Explicit file-copy operations make Dockerfiles easier to understand and reduce unexpected behavior, while a carefully designed .dockerignore file prevents irrelevant data from entering the build context.

A robotics repository may contain recorded ROS bags, simulation assets, neural-network checkpoints, datasets, build directories, logs, and Git metadata that should not automatically become part of every container build. Excluding these resources can substantially reduce build-context size and accidental image growth. Large models should be incorporated deliberately according to the deployment and model-versioning strategy.

Dockerfile commands should avoid embedding credentials, access tokens, private registry passwords, SSH keys, or other secrets into image layers. Removing a secret in a later instruction does not guarantee that it disappears from previous layers. Build-time secret mechanisms, controlled CI/CD credentials, and external runtime secret management should be used instead of permanently storing sensitive information inside robot images.

Container processes should run with the minimum privileges required by the application. Creating a dedicated non-root user in the Dockerfile reduces the consequences of application compromise. Robotics introduces additional complexity because access to serial ports, CAN interfaces, cameras, LiDAR devices, and other hardware may require specific users, groups, device mappings, or runtime permissions rather than unrestricted root execution.

WORKDIR should define a predictable working directory instead of repeatedly relying on shell directory changes. ENV can establish stable runtime environment variables, while ARG is useful for build-time parameters that do not need to remain runtime configuration. These mechanisms should be used deliberately because excessive build arguments and environment variables can make image behavior difficult to reproduce and diagnose.

CMD and ENTRYPOINT define how the container starts and should reflect the intended operational model. ENTRYPOINT is useful for establishing initialization behavior, such as sourcing ROS 2 environments before execution, while CMD can provide the default application or arguments. Exec-form commands are generally preferable because they preserve clearer process and signal behavior inside the container.

Signal handling is especially important in robotics because containers may host ROS 2 nodes controlling sensors, navigation processes, or hardware interfaces. The primary process should correctly receive termination signals and perform orderly shutdown when possible. Poor entrypoint design can interfere with signal propagation, leave child processes behind, or delay controlled termination during deployment, restart, and OTA operations.

Health checking should distinguish whether a process merely exists from whether the robot service is operational. A container may remain running while a ROS 2 node has lost communication, a sensor stream has stopped, or an inference service has become unresponsive. Docker-level health mechanisms can provide basic checks, while application-level observability should evaluate richer robot-specific operational conditions.

Image contents should remain deterministic and application-focused. Configuration that differs between robots, sites, fleets, or deployment stages should generally be injected at runtime rather than creating a separate image for every configuration. The same tested image can then move through development, simulation, hardware-in-the-loop testing, staging, and production while configuration remains independently controlled.

Logging should also respect the container execution model. Applications should preferably expose operational logs through standard output and standard error when appropriate, allowing the container platform to collect them consistently. High-volume sensor recordings, maps, diagnostic archives, and persistent robot data should be directed to explicitly managed storage rather than accumulating invisibly in the container\'s writable layer.

Build reproducibility should be integrated with CI/CD. A Dockerfile committed with robot source code allows automated pipelines to build the same environment used for testing and deployment. Images can be tagged using software versions, release identifiers, or source revisions, scanned for vulnerabilities, stored in a registry, and promoted through deployment stages without rebuilding application contents unnecessarily.

Robot software images must also account for target architecture. Development servers may use AMD64 processors while deployed edge computers use ARM64 platforms such as NVIDIA Jetson systems. Dockerfile design should therefore avoid unnecessary architecture assumptions, while architecture-specific packages, binary dependencies, CUDA components, and hardware libraries must be selected deliberately when multi-platform deployment is required.

GPU-enabled robot containers require particularly strict compatibility management. The image must provide appropriate user-space CUDA, inference, and AI framework components while remaining compatible with the host GPU driver and container runtime. Bundling arbitrary host driver components into the image can create conflicts, so the boundary between host-managed hardware support and container-managed application libraries must remain clear.

A well-designed Dockerfile ultimately represents more than a list of installation commands. It defines a controlled software supply boundary for the robot application. Minimal dependencies, deterministic versions, efficient layers, non-root execution, clean runtime configuration, proper signal handling, and CI/CD integration collectively make containerized robot software easier to reproduce, test, secure, distribute, update, and operate across a fleet.

Dockerfile은 컨테이너 이미지(Container Image)가 재현 가능한 일련의 명령어를 통해 어떻게 구성되는지를 정의한다. 로봇 소프트웨어 스택(Robot Software Stack)에서 Dockerfile은 운영체제 기반(Base), ROS 2 배포판(Distribution), 시스템 라이브러리(System Library), 미들웨어(Middleware), AI 프레임워크(AI Framework), 응용 프로그램 의존성(Application Dependency), 설정(Configuration), 시작 동작(Startup Behavior)을 포함하는 실행 환경의 실행 가능한 문서 역할을 한다.

기본 이미지(Base Image)의 선택은 이후의 모든 계층(Layer)이 운영체제 라이브러리와 호환성 제약을 상속하기 때문에 Dockerfile 설계에서 가장 중요한 결정 중 하나이다. 로봇 프로젝트에서는 신뢰할 수 있고 유지보수되며 적절하게 최소화된 이미지를 우선 사용하면서 Ubuntu, ROS 2, CUDA, Python 및 하드웨어 런타임(Hardware Runtime) 요구사항을 명확하게 일치시켜야 한다. 작은 이미지는 전송 시간과 불필요한 공격 표면(Attack Surface)을 줄여준다.

재현성(Reproducibility)이 중요한 경우에는 유동적인 태그(Floating Tag)에 의존하기보다 이미지 버전을 명시적으로 제어해야 한다. latest와 같은 태그는 시점에 따라 서로 다른 콘텐츠를 가리킬 수 있으므로 동일한 Dockerfile에서도 서로 다른 환경이 생성될 수 있다. 로봇 배포에서는 고정된 배포판 버전, 가능한 경우 명시적인 패키지 버전, 정확한 재구성이 필요한 릴리스에서는 불변 이미지 다이제스트(Immutable Image Digest)를 사용하는 것이 유리하다.

Dockerfile 명령어는 이미지 계층 모델(Image Layer Model)을 고려하여 구성해야 한다. RUN, COPY, ADD와 같은 명령은 이미지 계층의 일부가 되는 파일 시스템 변경 사항을 생성할 수 있다. 거의 변경되지 않는 시스템 의존성 설치와 같은 안정적인 작업은 자주 변경되는 응용 프로그램 소스 파일보다 앞에 배치하는 것이 일반적으로 바람직하며, 이를 통해 Docker 빌드 캐시(Build Cache)가 비용이 큰 이전 계층을 재사용할 수 있다.

패키지 설치 과정에서는 불필요한 파일과 중간 산출물(Intermediate Artifact)을 최소화해야 한다. Debian 및 Ubuntu 기반 로봇 이미지에서는 일반적으로 패키지 인덱스 업데이트와 패키지 설치를 동일한 RUN 명령 안에서 수행하고 이후 캐시된 패키지 목록을 제거해야 한다. 실제 로봇 응용 프로그램에서 필요하지 않은 권장 패키지(Recommended Package)는 가능한 한 제외하여 이미지 크기와 의존성 복잡도를 줄이는 것이 좋다.

로봇 소프트웨어는 운영체제 저장소와 워크스페이스 소스 트리(Workspace Source Tree)를 통해 배포되는 ROS 2 패키지에 동시에 의존하는 경우가 많다. Dockerfile에서는 외부 의존성 설치와 로봇 워크스페이스 빌드 과정을 명확하게 분리해야 한다. 응용 프로그램 소스 코드보다 패키지 매니페스트(Package Manifest)나 의존성 메타데이터(Dependency Metadata)를 먼저 복사하면 소스 코드가 수정될 때마다 의존성을 다시 설치하지 않아도 되므로 캐시 효율을 높일 수 있다.

ROS 2 워크스페이스 빌드에서는 환경 초기화(Environment Initialization)를 신중하게 관리해야 한다. rosdep이나 colcon과 같은 도구의 동작이 환경에 의존하는 경우 해당 도구를 실행하기 전에 필요한 ROS 배포판 환경을 소싱(Sourcing)해야 한다. 런타임 컨테이너 역시 노드를 실행하기 전에 ROS 환경과 응용 프로그램 워크스페이스를 초기화해야 하며, 그렇지 않으면 패키지 탐색, 공유 라이브러리, 메시지 정의 및 미들웨어 설정에서 문제가 발생할 수 있다.

Dockerfile에서는 빌드 시 의존성(Build-time Dependency)과 런타임 의존성(Runtime Dependency)을 구분해야 한다. 컴파일러, 헤더 패키지(Header Package), 소스 트리, 디버깅 도구 및 빌드 시스템은 ROS 2 또는 네이티브 C++ 응용 프로그램을 컴파일하는 데 필요할 수 있지만 실제 운영 시에는 필요하지 않은 경우가 많다. 이러한 도구를 운영 이미지에서 제외하면 저장 공간, 배포 비용, 유지보수 부담 및 잠재적인 보안 노출을 줄일 수 있다.

이러한 분리는 개발 환경에 CUDA 툴체인(CUDA Toolchain), Python 빌드 유틸리티(Build Utility), 모델 변환 도구(Model Conversion Tool), 대규모 중간 산출물이 포함될 수 있는 로봇 AI 소프트웨어에서 특히 중요하다. 운영 컨테이너에는 최종 바이너리(Binary), Python 환경, 추론 라이브러리(Inference Library), 모델 및 호환되는 GPU 런타임 구성요소만 필요할 수 있다. 이후 다단계 빌드(Multi-stage Build) 기법을 사용하면 이러한 분리를 더욱 효율적으로 구현할 수 있다.

파일을 빌드 컨텍스트(Build Context)에서 이미지로 단순히 복사해야 하는 경우에는 일반적으로 COPY 명령을 사용하는 것이 좋다. ADD는 많은 일반적인 로봇 소프트웨어 빌드에서는 필요하지 않은 추가 동작을 제공한다. 명시적인 파일 복사 작업은 Dockerfile을 이해하기 쉽게 만들며, 신중하게 설계된 .dockerignore 파일은 불필요한 데이터가 빌드 컨텍스트에 포함되는 것을 방지한다.

로보틱스 저장소(Robotics Repository)에는 기록된 ROS Bag, 시뮬레이션 자산(Simulation Asset), 신경망 체크포인트(Neural-network Checkpoint), 데이터셋(Dataset), 빌드 디렉터리, 로그 및 Git 메타데이터가 포함될 수 있으며, 이러한 데이터가 모든 컨테이너 빌드에 자동으로 포함되어서는 안 된다. 이를 제외하면 빌드 컨텍스트 크기와 의도하지 않은 이미지 크기 증가를 크게 줄일 수 있다. 대형 모델은 배포 및 모델 버전 관리 전략에 따라 의도적으로 포함해야 한다.

Dockerfile 명령어에는 인증 정보(Credential), 접근 토큰(Access Token), 비공개 레지스트리 암호(Private Registry Password), SSH 키 또는 기타 비밀 정보(Secret)를 직접 포함해서는 안 된다. 이후 명령에서 비밀 정보를 삭제하더라도 이전 이미지 계층에서 완전히 제거된다는 보장은 없다. 민감한 정보를 로봇 이미지에 영구 저장하는 대신 빌드 시 비밀 관리(Build-time Secret), 통제된 CI/CD 인증 정보 및 외부 런타임 비밀 관리(Runtime Secret Management)를 사용해야 한다.

컨테이너 프로세스(Container Process)는 응용 프로그램에 필요한 최소 권한(Least Privilege)으로 실행해야 한다. Dockerfile에서 전용 비루트 사용자(Non-root User)를 생성하면 응용 프로그램이 침해되었을 때 발생할 수 있는 영향을 줄일 수 있다. 로보틱스에서는 직렬 포트, CAN 인터페이스, 카메라, LiDAR 및 기타 하드웨어 접근에 특정 사용자, 그룹, 장치 매핑(Device Mapping), 런타임 권한이 필요할 수 있으므로 무제한 루트 실행 대신 필요한 권한만 제공하도록 설계해야 한다.

WORKDIR은 셸의 디렉터리 변경 명령에 반복적으로 의존하지 않고 예측 가능한 작업 디렉터리(Working Directory)를 정의하는 데 사용해야 한다. ENV는 안정적인 런타임 환경 변수(Runtime Environment Variable)를 설정할 수 있으며, ARG는 런타임 설정으로 유지할 필요가 없는 빌드 시 매개변수(Build-time Parameter)에 유용하다. 과도한 빌드 인수와 환경 변수는 이미지 동작의 재현과 진단을 어렵게 만들 수 있으므로 신중하게 사용해야 한다.

CMD와 ENTRYPOINT는 컨테이너가 시작되는 방식을 정의하며 의도된 운영 모델(Operational Model)을 반영해야 한다. ENTRYPOINT는 실행 전에 ROS 2 환경을 소싱하는 것과 같은 초기화 동작을 설정하는 데 유용하고, CMD는 기본 응용 프로그램이나 인수를 제공할 수 있다. 일반적으로 실행 형식(Exec Form)의 명령을 사용하는 것이 컨테이너 내부의 프로세스 및 신호 동작을 보다 명확하게 유지하는 데 유리하다.

신호 처리(Signal Handling)는 컨테이너가 센서, 내비게이션 프로세스 또는 하드웨어 인터페이스를 제어하는 ROS 2 노드를 실행할 수 있기 때문에 로보틱스에서 특히 중요하다. 주 프로세스(Primary Process)는 종료 신호(Termination Signal)를 올바르게 수신하고 가능한 경우 순차적인 종료(Orderly Shutdown)를 수행해야 한다. 잘못 설계된 엔트리포인트(Entrypoint)는 신호 전달을 방해하거나 자식 프로세스를 남기고 배포, 재시작 및 OTA 작업 중 제어된 종료를 지연시킬 수 있다.

상태 확인(Health Checking)은 단순히 프로세스가 존재하는지와 로봇 서비스가 실제로 정상 동작하는지를 구분해야 한다. 컨테이너가 실행 중이더라도 ROS 2 노드가 통신을 상실하거나 센서 스트림이 중단되거나 추론 서비스가 응답하지 않을 수 있다. Docker 수준의 상태 확인 메커니즘은 기본적인 검사를 제공할 수 있으며, 응용 프로그램 수준의 관측성(Observability)을 통해 보다 구체적인 로봇 운영 상태를 평가해야 한다.

이미지의 내용은 결정적(Deterministic)이며 응용 프로그램 중심으로 유지해야 한다. 로봇, 사이트, 플릿 또는 배포 단계에 따라 달라지는 설정은 각각 별도의 이미지를 생성하기보다 일반적으로 런타임에 주입(Runtime Injection)하는 것이 좋다. 이를 통해 동일하게 검증된 이미지를 개발, 시뮬레이션, 하드웨어 인 더 루프 시험(Hardware-in-the-Loop Testing), 스테이징(Staging), 운영 환경으로 이동시키면서 설정은 독립적으로 관리할 수 있다.

로깅(Logging) 역시 컨테이너 실행 모델을 고려해야 한다. 응용 프로그램은 적절한 경우 표준 출력(Standard Output)과 표준 오류(Standard Error)를 통해 운영 로그를 제공하여 컨테이너 플랫폼이 일관된 방식으로 수집할 수 있도록 하는 것이 좋다. 대용량 센서 기록, 지도, 진단 아카이브(Diagnostic Archive) 및 영구 로봇 데이터는 컨테이너의 쓰기 가능 계층에 누적시키지 않고 명시적으로 관리되는 저장소로 전달해야 한다.

빌드 재현성(Build Reproducibility)은 CI/CD와 통합되어야 한다. 로봇 소스 코드와 함께 저장된 Dockerfile을 사용하면 자동화된 파이프라인이 시험과 배포에 사용되는 동일한 환경을 빌드할 수 있다. 이미지는 소프트웨어 버전, 릴리스 식별자 또는 소스 리비전(Source Revision)을 사용해 태깅하고, 취약점 검사(Vulnerability Scanning)를 수행한 뒤 레지스트리에 저장하여 응용 프로그램을 불필요하게 다시 빌드하지 않고 여러 배포 단계로 승격(Promotion)할 수 있다.

로봇 소프트웨어 이미지는 대상 아키텍처(Target Architecture)도 고려해야 한다. 개발 서버에서는 AMD64 프로세서를 사용할 수 있지만 배포되는 엣지 컴퓨터에서는 NVIDIA Jetson 시스템과 같은 ARM64 플랫폼을 사용할 수 있다. 따라서 Dockerfile 설계에서는 불필요한 아키텍처 가정을 피해야 하며, 다중 플랫폼 배포(Multi-platform Deployment)가 필요한 경우 아키텍처별 패키지, 바이너리 의존성, CUDA 구성요소 및 하드웨어 라이브러리를 명확하게 선택해야 한다.

GPU를 사용하는 로봇 컨테이너는 특히 엄격한 호환성 관리(Compatibility Management)가 필요하다. 이미지는 적절한 사용자 공간 CUDA(User-space CUDA), 추론 및 AI 프레임워크 구성요소를 제공하면서 호스트 GPU 드라이버와 컨테이너 런타임의 호환성을 유지해야 한다. 임의의 호스트 드라이버 구성요소를 이미지 내부에 포함하면 충돌이 발생할 수 있으므로 호스트가 관리하는 하드웨어 지원과 컨테이너가 관리하는 응용 프로그램 라이브러리 사이의 경계를 명확하게 유지해야 한다.

잘 설계된 Dockerfile은 궁극적으로 단순한 설치 명령어의 목록 이상의 의미를 가진다. 이는 로봇 응용 프로그램을 위한 통제된 소프트웨어 공급 경계(Software Supply Boundary)를 정의한다. 최소화된 의존성, 결정적인 버전 관리, 효율적인 계층, 비루트 실행, 명확한 런타임 설정, 적절한 신호 처리 및 CI/CD 통합은 컨테이너화된 로봇 소프트웨어의 재현, 시험, 보안, 배포, 업데이트 및 플릿 운영을 더욱 효율적으로 만든다.

##  

## 3.3. Multi Stage Build for Embedded and ROS2 Images [w/Code]

![](images/image3.png){width="7.268055555555556in" height="7.268055555555556in"}

Multi-stage builds allow a Dockerfile to contain multiple build environments while producing a final image that contains only the components required for execution. Each stage begins from its own base image and can perform a specialized task such as dependency resolution, compilation, testing, artifact generation, or runtime packaging. Selected artifacts are then copied between stages without carrying the complete build environment into production.

This approach is especially valuable for embedded and ROS 2 software because development environments are usually much larger than runtime environments. Building C++ packages may require compilers, CMake, colcon, header files, debugging utilities, rosdep, and source repositories. The deployed robot normally needs only compiled executables, shared libraries, ROS interfaces, configuration files, and selected runtime dependencies.

A typical multi-stage Dockerfile starts with a builder stage containing the complete toolchain. Source code and dependency metadata are copied into this stage, required packages are installed, and the ROS 2 workspace or embedded application is compiled. A later runtime stage starts from a cleaner base image and copies only the generated installation artifacts and other files required to operate the application.

Docker identifies stages through FROM instructions, and stages can be assigned descriptive names using the AS syntax. Naming stages such as builder, tester, and runtime makes the Dockerfile easier to understand and prevents fragile references based only on numerical stage order. The COPY \--from mechanism then transfers selected files or directories from an earlier stage into a later stage.

For ROS 2 projects, the builder stage commonly contains the source workspace and all packages required by rosdep and colcon. After dependencies are resolved, colcon builds the workspace and generates install artifacts. The runtime stage can then receive the install directory rather than the entire source and build trees, substantially reducing the amount of development material included in the deployed image.

ROS 2 environment handling remains important across stages. The builder must initialize the appropriate ROS distribution before compilation, and the final runtime environment must source both the underlying ROS installation and the copied workspace overlay. If this initialization is omitted, package discovery, shared-library resolution, interface definitions, plugins, and executable lookup may behave differently from the original build environment.

Multi-stage builds also improve dependency separation. Build dependencies such as gcc, g++, make, CMake, development headers, Python build tools, and source-control clients can remain exclusively in the builder stage. Runtime packages can be installed independently in the final stage. This creates a clearer distinction between software needed to manufacture an artifact and software needed to execute that artifact.

The same principle applies to embedded Linux applications. Cross-compilation may require architecture-specific toolchains, sysroots, board support packages, SDKs, code generators, and development libraries that can occupy substantial storage. These components can remain in a dedicated build stage while only target binaries, required shared objects, configuration files, and runtime assets are transferred into the deployment image.

Cross-compilation introduces an important architectural distinction between the machine performing the build and the machine executing the resulting software. An AMD64 CI server may build software intended for an ARM64 robot computer. Multi-stage design can isolate host-side compilation tools from target-side runtime components, making architecture assumptions more visible and simplifying later multi-architecture container strategies.

AI-enabled ROS 2 images can benefit even more because their build environments may contain CUDA development packages, TensorRT development components, model converters, Python compilers, and temporary model artifacts. If the deployed application only performs inference, the final stage can contain the appropriate runtime libraries and optimized model artifacts without retaining the complete development toolchain.

Image size reduction is an important consequence, but it is not the only objective of multi-stage builds. Removing compilers, package managers, source code, temporary files, and debugging utilities from production images reduces unnecessary software inventory and potential attack surface. The final image becomes easier to scan, distribute, cache, audit, and maintain across a large robot fleet.

However, a smaller final image must still contain every runtime dependency required by the copied artifacts. A compiled executable may depend on shared libraries that existed automatically in the builder image but are absent from the runtime image. Native ROS 2 packages, plugins, DDS implementations, GPU libraries, Python modules, and dynamically loaded components must therefore be checked carefully when constructing the runtime stage.

Using the same or compatible operating-system family across builder and runtime stages can reduce binary compatibility problems. Differences in C libraries, compiler runtime libraries, ABI versions, ROS distributions, or middleware implementations may prevent an apparently successful build artifact from running correctly. Multi-stage optimization should therefore preserve compatibility rather than pursuing minimal image size without considering the runtime dependency chain.

Docker build caching remains highly relevant in multi-stage designs. Dependency metadata should generally be copied before frequently changing application source so that dependency installation can remain cached. Expensive operations such as rosdep installation, package downloads, and compilation of stable libraries should be structured to avoid unnecessary repetition whenever only a small portion of the robot application changes.

Separate stages can also be used for testing. A builder stage may compile the software, while an intermediate test stage executes unit tests, static checks, or ROS 2 package tests before artifacts are admitted to the runtime stage. In CI/CD, failure of this stage prevents an invalid image from progressing further, allowing the Dockerfile itself to represent part of the software quality gate.

Artifact ownership and file permissions must be considered when copying data between stages. Files generated as root in the builder may later be used by a non-root runtime process. The final image should establish appropriate ownership, executable permissions, directory access, and device-related group configuration so that the deployed application does not require unnecessary privilege simply because of how artifacts were created.

Runtime configuration should remain separate from build artifacts wherever practical. Robot-specific identifiers, network parameters, DDS configuration, map selection, sensor calibration, fleet endpoints, and site settings often vary independently of the software version. Multi-stage builds should create a reusable application image while runtime configuration is injected through environment variables, mounted files, volumes, or deployment configuration mechanisms.

Large datasets, ROS bag files, simulation resources, and temporary build outputs should not pass automatically from one stage to another. COPY \--from should be deliberately scoped so that only required artifacts enter the final image. This explicit transfer boundary provides a useful opportunity to inspect what the production robot actually receives and prevents accidental inclusion of development data.

For embedded robots, the final runtime stage may also require controlled access to physical devices such as serial ports, CAN interfaces, cameras, LiDAR sensors, and GPU accelerators. These devices are generally exposed when the container starts rather than embedded during image construction. Multi-stage builds optimize software contents, while runtime device mapping and permissions remain deployment responsibilities.

Multi-stage Dockerfiles can support several deliverables from a shared build process. One target may provide a development environment with debugging tools, another may support automated testing, and another may produce a minimal production runtime. Explicit build targets allow CI pipelines and developers to select the appropriate stage without maintaining completely separate Dockerfiles that gradually diverge from one another.

This pattern improves traceability when combined with version-controlled source and automated CI/CD. The same Dockerfile can compile a defined source revision, execute validation, create runtime artifacts, generate a production image, and associate that image with a release tag or immutable digest. The resulting container can then be promoted through simulation, hardware-in-the-loop testing, staging, and robot fleet deployment.

Multi-stage builds therefore establish a controlled boundary between software construction and software execution. For embedded and ROS 2 systems, they help isolate complex toolchains from deployed workloads, reduce image size and attack surface, preserve reproducible build procedures, and support architecture-specific packaging. When dependency compatibility and runtime configuration are managed carefully, they provide a strong foundation for reliable containerized robot software delivery.

다단계 빌드(Multi-stage Build)는 하나의 Dockerfile 안에 여러 빌드 환경(Build Environment)을 구성하면서 최종적으로는 실행에 필요한 구성요소만 포함하는 이미지를 생성할 수 있도록 한다. 각 단계(Stage)는 자체 기본 이미지(Base Image)에서 시작하며 의존성 해결, 컴파일, 테스트, 산출물 생성 또는 런타임 패키징(Runtime Packaging)과 같은 특정 작업을 수행한다. 이후 필요한 산출물만 단계 사이에서 복사하므로 전체 빌드 환경을 운영 이미지로 가져갈 필요가 없다.

이러한 접근 방식은 개발 환경이 런타임 환경(Runtime Environment)보다 훨씬 큰 경우가 많은 임베디드(Embedded) 및 ROS 2 소프트웨어에서 특히 유용하다. C++ 패키지를 빌드하려면 컴파일러, CMake, colcon, 헤더 파일, 디버깅 도구, rosdep 및 소스 저장소가 필요할 수 있다. 그러나 실제 로봇에는 일반적으로 컴파일된 실행 파일, 공유 라이브러리, ROS 인터페이스, 설정 파일 및 선택된 런타임 의존성만 필요하다.

일반적인 다단계 Dockerfile은 완전한 툴체인(Toolchain)을 포함하는 빌더 단계(Builder Stage)에서 시작한다. 소스 코드와 의존성 메타데이터(Dependency Metadata)를 이 단계로 복사하고 필요한 패키지를 설치한 후 ROS 2 워크스페이스(Workspace) 또는 임베디드 응용 프로그램을 컴파일한다. 이후 런타임 단계(Runtime Stage)는 보다 정리된 기본 이미지에서 시작하여 생성된 설치 산출물과 응용 프로그램 실행에 필요한 파일만 복사한다.

Docker는 FROM 명령을 통해 단계를 구분하며 AS 구문을 사용하여 각 단계에 설명적인 이름을 지정할 수 있다. builder, tester, runtime과 같이 단계 이름을 지정하면 Dockerfile을 이해하기 쉬워지고 숫자로 된 단계 순서에 의존하는 불안정한 참조를 방지할 수 있다. 이후 COPY \--from 메커니즘을 사용하여 이전 단계의 특정 파일이나 디렉터리를 다음 단계로 전달한다.

ROS 2 프로젝트에서 빌더 단계는 일반적으로 소스 워크스페이스와 rosdep 및 colcon에 필요한 모든 패키지를 포함한다. 의존성을 해결한 후 colcon으로 워크스페이스를 빌드하여 설치 산출물(Install Artifact)을 생성한다. 런타임 단계에서는 전체 소스 및 빌드 트리 대신 install 디렉터리만 전달할 수 있으므로 배포 이미지에 포함되는 개발 관련 자료를 크게 줄일 수 있다.

ROS 2 환경 처리(Environment Handling)는 각 단계에서도 중요하다. 빌더는 컴파일 전에 적절한 ROS 배포판(ROS Distribution)을 초기화해야 하며, 최종 런타임 환경에서는 기본 ROS 설치와 복사된 워크스페이스 오버레이(Workspace Overlay)를 모두 소싱(Sourcing)해야 한다. 이러한 초기화가 누락되면 패키지 탐색, 공유 라이브러리 해결, 인터페이스 정의, 플러그인 및 실행 파일 검색이 원래 빌드 환경과 다르게 동작할 수 있다.

다단계 빌드는 의존성 분리(Dependency Separation)도 향상시킨다. gcc, g++, make, CMake, 개발용 헤더, Python 빌드 도구 및 소스 제어 클라이언트(Source-control Client)와 같은 빌드 의존성은 빌더 단계에만 유지할 수 있다. 런타임 패키지는 최종 단계에서 별도로 설치할 수 있으며, 이를 통해 산출물을 생성하는 데 필요한 소프트웨어와 실제 산출물을 실행하는 데 필요한 소프트웨어를 명확하게 구분할 수 있다.

동일한 원칙은 임베디드 리눅스(Embedded Linux) 응용 프로그램에도 적용된다. 교차 컴파일(Cross-compilation)에는 상당한 저장 공간을 차지할 수 있는 아키텍처별 툴체인, 시스템 루트(Sysroot), 보드 지원 패키지(Board Support Package), SDK, 코드 생성기 및 개발 라이브러리가 필요할 수 있다. 이러한 구성요소는 전용 빌드 단계에 유지하고 대상 바이너리, 필요한 공유 객체(Shared Object), 설정 파일 및 런타임 자산(Runtime Asset)만 배포 이미지로 전달할 수 있다.

교차 컴파일은 빌드를 수행하는 시스템과 결과 소프트웨어를 실행하는 시스템 사이의 중요한 아키텍처 차이를 발생시킨다. AMD64 기반 CI 서버에서 ARM64 로봇 컴퓨터용 소프트웨어를 빌드할 수 있다. 다단계 설계는 호스트 측 컴파일 도구(Host-side Compilation Tool)와 대상 측 런타임 구성요소(Target-side Runtime Component)를 분리하여 아키텍처에 대한 가정을 명확하게 만들고 이후의 다중 아키텍처 컨테이너(Multi-architecture Container) 전략을 단순화할 수 있다.

AI 기능을 포함하는 ROS 2 이미지는 빌드 환경에 CUDA 개발 패키지, TensorRT 개발 구성요소, 모델 변환기(Model Converter), Python 컴파일러 및 임시 모델 산출물이 포함될 수 있기 때문에 더 큰 효과를 얻을 수 있다. 배포된 응용 프로그램이 추론(Inference)만 수행한다면 최종 단계에는 전체 개발 툴체인을 유지하지 않고 적절한 런타임 라이브러리와 최적화된 모델 산출물만 포함할 수 있다.

이미지 크기 감소는 중요한 효과이지만 다단계 빌드의 유일한 목적은 아니다. 운영 이미지에서 컴파일러, 패키지 관리자, 소스 코드, 임시 파일 및 디버깅 도구를 제거하면 불필요한 소프트웨어 구성요소와 잠재적인 공격 표면(Attack Surface)을 줄일 수 있다. 최종 이미지는 대규모 로봇 플릿(Robot Fleet)에서 검사, 배포, 캐싱, 감사 및 유지보수하기 쉬워진다.

그러나 크기가 작은 최종 이미지라도 복사된 산출물을 실행하는 데 필요한 모든 런타임 의존성(Runtime Dependency)을 포함해야 한다. 컴파일된 실행 파일은 빌더 이미지에는 기본적으로 존재하지만 런타임 이미지에는 없는 공유 라이브러리에 의존할 수 있다. 따라서 네이티브 ROS 2 패키지, 플러그인, DDS 구현, GPU 라이브러리, Python 모듈 및 동적으로 로드되는 구성요소를 런타임 단계 구성 시 주의 깊게 확인해야 한다.

빌더와 런타임 단계에서 동일하거나 호환되는 운영체제 계열(Operating-system Family)을 사용하면 바이너리 호환성(Binary Compatibility) 문제를 줄일 수 있다. C 라이브러리, 컴파일러 런타임 라이브러리, ABI 버전, ROS 배포판 또는 미들웨어 구현의 차이로 인해 정상적으로 빌드된 산출물이 실행되지 않을 수 있다. 따라서 다단계 최적화는 단순히 최소 이미지 크기를 추구하기보다 런타임 의존성 체인의 호환성을 유지해야 한다.

Docker 빌드 캐싱(Build Caching)은 다단계 설계에서도 매우 중요하다. 의존성 메타데이터는 일반적으로 자주 변경되는 응용 프로그램 소스보다 먼저 복사하여 의존성 설치 결과를 캐시에 유지할 수 있도록 해야 한다. rosdep 설치, 패키지 다운로드 및 안정적인 라이브러리 컴파일과 같이 비용이 큰 작업은 로봇 응용 프로그램의 작은 부분만 변경되었을 때 불필요하게 반복되지 않도록 구성해야 한다.

별도의 단계는 테스트(Test)를 수행하는 데에도 사용할 수 있다. 빌더 단계에서 소프트웨어를 컴파일한 후 중간 테스트 단계(Test Stage)에서 단위 테스트(Unit Test), 정적 검사(Static Check) 또는 ROS 2 패키지 테스트를 수행하고, 검증된 산출물만 런타임 단계로 전달할 수 있다. CI/CD에서는 이 단계가 실패하면 잘못된 이미지가 다음 과정으로 진행되는 것을 차단하므로 Dockerfile 자체가 소프트웨어 품질 게이트(Quality Gate)의 일부가 될 수 있다.

단계 사이에서 데이터를 복사할 때는 산출물 소유권(Artifact Ownership)과 파일 권한(File Permission)을 고려해야 한다. 빌더에서 루트(Root) 권한으로 생성된 파일을 이후 비루트 런타임 프로세스(Non-root Runtime Process)가 사용할 수 있다. 최종 이미지에서는 적절한 소유권, 실행 권한, 디렉터리 접근 권한 및 장치 관련 그룹 설정을 구성하여 산출물이 생성된 방식 때문에 배포 응용 프로그램이 불필요한 권한을 요구하지 않도록 해야 한다.

런타임 설정(Runtime Configuration)은 가능한 경우 빌드 산출물과 분리하여 유지해야 한다. 로봇 식별자, 네트워크 매개변수, DDS 설정, 지도 선택, 센서 보정(Sensor Calibration), 플릿 엔드포인트(Fleet Endpoint), 사이트 설정 등은 소프트웨어 버전과 독립적으로 변경되는 경우가 많다. 다단계 빌드는 재사용 가능한 응용 프로그램 이미지를 생성하고 런타임 설정은 환경 변수, 마운트된 파일, 볼륨 또는 배포 설정 메커니즘을 통해 주입하도록 구성하는 것이 적절하다.

대규모 데이터셋, ROS Bag 파일, 시뮬레이션 자원 및 임시 빌드 출력은 한 단계에서 다음 단계로 자동 전달되어서는 안 된다. COPY \--from은 필요한 산출물만 최종 이미지로 들어가도록 명시적인 범위로 제한해야 한다. 이러한 명확한 전달 경계(Transfer Boundary)는 실제 운영 로봇에 무엇이 배포되는지 검사할 수 있는 기회를 제공하고 개발 데이터가 실수로 포함되는 것을 방지한다.

임베디드 로봇의 최종 런타임 단계에서는 직렬 포트, CAN 인터페이스, 카메라, LiDAR 센서 및 GPU 가속기와 같은 물리적 장치에 대한 제어된 접근이 필요할 수도 있다. 이러한 장치는 일반적으로 이미지 빌드 과정에서 포함되는 것이 아니라 컨테이너 시작 시 노출된다. 다단계 빌드는 소프트웨어 내용을 최적화하고 런타임 장치 매핑(Runtime Device Mapping)과 권한 관리는 배포 단계에서 담당하도록 역할을 분리한다.

다단계 Dockerfile은 하나의 공통 빌드 프로세스에서 여러 종류의 결과물을 지원할 수도 있다. 하나의 대상(Target)은 디버깅 도구를 포함한 개발 환경을 제공하고, 다른 대상은 자동화 테스트를 지원하며, 또 다른 대상은 최소화된 운영 런타임을 생성할 수 있다. 명시적인 빌드 대상(Build Target)을 사용하면 CI 파이프라인과 개발자가 서로 다른 Dockerfile을 별도로 유지하여 점차 내용이 달라지는 문제 없이 필요한 단계를 선택할 수 있다.

이러한 패턴은 버전 관리된 소스와 자동화된 CI/CD를 결합할 때 추적성(Traceability)을 향상시킨다. 동일한 Dockerfile을 사용하여 특정 소스 리비전(Source Revision)을 컴파일하고 검증을 수행하며 런타임 산출물을 생성한 뒤 운영 이미지를 만들고 이를 릴리스 태그(Release Tag) 또는 불변 다이제스트(Immutable Digest)와 연결할 수 있다. 생성된 컨테이너는 이후 시뮬레이션, 하드웨어 인 더 루프 시험(Hardware-in-the-Loop Testing), 스테이징(Staging), 로봇 플릿 배포 단계로 순차적으로 승격할 수 있다.

따라서 다단계 빌드(Multi-stage Build)는 소프트웨어 생성(Software Construction)과 소프트웨어 실행(Software Execution) 사이에 통제된 경계를 설정한다. 임베디드 및 ROS 2 시스템에서는 복잡한 툴체인을 배포 워크로드에서 분리하고 이미지 크기와 공격 표면을 줄이며 재현 가능한 빌드 절차와 아키텍처별 패키징을 지원한다. 의존성 호환성과 런타임 설정을 신중하게 관리하면 신뢰성 높은 컨테이너 기반 로봇 소프트웨어 배포를 위한 강력한 기반을 제공할 수 있다.

##  

## 3.4. Multi Architecture Image Build ARM64 AMD64 [w/Code]

![](images/image4.png){width="7.268055555555556in" height="7.268055555555556in"}

Multi-architecture container images allow the same application release to run across different processor architectures while preserving a common image name and deployment workflow. In robotics, this capability is particularly important because development workstations and cloud CI servers commonly use AMD64 processors, while embedded robot computers frequently use ARM64 platforms such as NVIDIA Jetson systems.

AMD64, also known as x86-64, dominates conventional desktop computers, workstations, servers, and many cloud environments. ARM64, also called AArch64, is widely used in power-efficient embedded and edge computing platforms. Although application source code may be identical, compiled binaries, operating-system packages, native libraries, and hardware-specific components must match the instruction set architecture of the target processor.

A conventional container image normally contains binaries for one architecture. Attempting to execute an AMD64 binary directly on an ARM64 processor, or the reverse, generally fails unless an emulation mechanism is available. Multi-architecture image design solves the distribution problem by associating architecture-specific images with a common image reference that allows a container runtime to select the appropriate variant.

This mechanism is commonly implemented through an image index or manifest list. Instead of a single tag identifying only one architecture-specific image, the tag references metadata describing several platform variants. When a client pulls the image, the container runtime evaluates the host operating system and processor architecture and retrieves the matching image automatically.

For example, a robotics release can publish both linux/amd64 and linux/arm64 variants under the same version tag. An AMD64 development workstation receives the AMD64 image, while an ARM64 edge computer receives the ARM64 image. This provides a consistent deployment interface even though the underlying binaries and some dependencies differ between the two platforms.

Docker Buildx provides a practical mechanism for creating multi-platform images using the BuildKit build engine. A build can specify several target platforms and generate architecture-specific outputs from one Dockerfile. The resulting images can then be pushed to a registry together with a multi-platform manifest, allowing downstream systems to use a single logical image reference.

Multi-architecture builds require a clear distinction between the build platform and the target platform. The build platform identifies the architecture executing the Docker build process, while the target platform identifies the architecture for which a particular image stage is being produced. Dockerfile variables and BuildKit platform metadata can be used to make architecture-dependent behavior explicit when necessary.

There are several ways to build images for architectures different from the host. Emulation can execute foreign-architecture binaries through mechanisms such as QEMU and Linux binary-format registration. This approach is convenient for CI and relatively simple builds, but emulated compilation can be significantly slower than native execution, particularly for large C++, ROS 2, computer vision, and AI workloads.

Native multi-node builders provide another strategy. An AMD64 builder can produce AMD64 artifacts while an ARM64 builder produces ARM64 artifacts, with BuildKit coordinating the outputs. Native builders avoid much of the performance penalty associated with emulation and are attractive for large robotics projects where ROS 2 compilation, CUDA components, or extensive native dependencies make build times significant.

Cross-compilation is a third approach and can be particularly effective for embedded software. A compiler running on an AMD64 machine can generate ARM64 binaries using an appropriate cross-toolchain and target sysroot. This method can provide excellent performance, but the build system, libraries, package discovery, and architecture-specific dependencies must all be designed carefully to prevent host components from contaminating target artifacts.

Multi-stage builds combine naturally with multi-architecture workflows. A builder stage can contain architecture-specific compilers and development dependencies, while the runtime stage receives only the resulting target binaries and required libraries. This separation keeps production images smaller and makes it easier to reason about which components belong to the build host and which belong to the deployed robot.

ROS 2 adds another compatibility layer because many packages include native C or C++ code. Pure Python ROS 2 nodes may require fewer architecture-specific changes, but native packages must be compiled for each target. DDS implementations, message-support libraries, plugins, computer-vision packages, and vendor drivers must also be available and compatible with the selected architecture and ROS 2 distribution.

Architecture-independent Dockerfile design should be preferred whenever possible. Hard-coded downloads containing amd64 or arm64 in filenames can break multi-platform builds unless the architecture is selected dynamically. Package repositories, installation scripts, and externally downloaded binaries should therefore be examined to ensure that the correct artifact is selected for the active target platform.

The base image must also support every intended architecture. Many official Linux and ROS-related images publish variants for multiple platforms, but specialized images may support only a subset. A Dockerfile cannot produce a valid ARM64 runtime merely by requesting linux/arm64 if the chosen base image or a required binary dependency is unavailable for that architecture.

GPU-enabled robotics introduces additional constraints. An AMD64 workstation with a discrete NVIDIA GPU and an ARM64 NVIDIA Jetson platform may both use CUDA-based applications, but their software stacks are not necessarily interchangeable. CUDA versions, TensorRT packages, platform libraries, device drivers, and Jetson-specific components must be aligned with the corresponding hardware and operating environment.

For this reason, a single Dockerfile does not necessarily imply that every instruction and dependency must be identical across architectures. Shared logic should remain common, while legitimate platform-specific differences can be handled explicitly. The objective is to minimize unnecessary divergence while acknowledging hardware-specific runtime requirements rather than forcing incompatible platforms into an artificial uniform configuration.

CI/CD pipelines should build and validate each supported architecture independently. Successful compilation on AMD64 does not demonstrate that an ARM64 image is functional. Each variant should undergo appropriate dependency checks, automated tests, vulnerability scanning, startup validation, and where necessary hardware-in-the-loop testing on representative target devices before fleet deployment.

Testing becomes particularly important when emulation is used during the build. QEMU can confirm many installation and execution paths, but it cannot fully reproduce physical robot hardware, GPU behavior, device drivers, timing characteristics, or architecture-specific performance. Native ARM64 validation remains necessary for software that interacts closely with sensors, accelerators, real-time workloads, or vendor-specific hardware.

Image tagging should separate application version identity from platform selection. Instead of requiring operators to manually choose unrelated tags for every processor architecture, a common release tag can reference a manifest containing the supported variants. Architecture-specific tags may still be retained for debugging or traceability, while normal deployment uses the common multi-platform reference.

Registries store the architecture-specific images and the manifest metadata that connects them. This makes distribution transparent to most deployment tools. A fleet management system can request the same application version for heterogeneous robots, while each robot\'s container runtime retrieves the variant appropriate for its processor architecture, assuming the registry manifest contains that supported platform.

Software supply-chain controls should apply to every architecture variant. Each image can contain different binaries and potentially different package versions, so vulnerability scanning and software bill of materials generation should not assume that the AMD64 result represents ARM64. Release records should preserve the image digests, provenance, dependencies, and validation status associated with each platform variant.

Reproducibility also requires controlling external dependencies. If an AMD64 build and ARM64 build independently download packages without version constraints, their contents may diverge beyond unavoidable architectural differences. Fixed package versions where practical, controlled repositories, deterministic build procedures, and immutable release references help ensure that both variants represent the same logical software release.

In robotics, multi-architecture images support a broader software-defined platform strategy. Developers can work on AMD64 laptops or workstations, CI systems can build on scalable server infrastructure, simulation can run on high-performance x86 machines, and production software can be deployed to ARM64 edge computers without maintaining completely separate application repositories and release processes.

A robust ARM64 and AMD64 image strategy therefore combines common source code, architecture-aware Dockerfiles, BuildKit or equivalent build infrastructure, controlled dependencies, architecture-specific validation, and registry manifests. The result is not one universal binary, but one coordinated software release composed of verified platform variants that can move consistently from development and CI through simulation, edge deployment, and heterogeneous robot fleets.

다중 아키텍처 컨테이너 이미지(Multi-architecture Container Image)는 동일한 이미지 이름과 배포 워크플로(Deployment Workflow)를 유지하면서 동일한 응용 프로그램 릴리스(Application Release)를 서로 다른 프로세서 아키텍처에서 실행할 수 있도록 한다. 로보틱스에서는 개발 워크스테이션과 클라우드 CI 서버가 일반적으로 AMD64 프로세서를 사용하는 반면, 임베디드 로봇 컴퓨터는 NVIDIA Jetson과 같은 ARM64 플랫폼을 사용하는 경우가 많기 때문에 이러한 기능이 특히 중요하다.

x86-64라고도 하는 AMD64는 일반적인 데스크톱 컴퓨터, 워크스테이션, 서버 및 많은 클라우드 환경에서 널리 사용된다. AArch64라고도 하는 ARM64는 전력 효율적인 임베디드 및 엣지 컴퓨팅(Edge Computing) 플랫폼에서 폭넓게 사용된다. 응용 프로그램 소스 코드가 동일하더라도 컴파일된 바이너리, 운영체제 패키지, 네이티브 라이브러리(Native Library) 및 하드웨어별 구성요소는 대상 프로세서의 명령어 집합 아키텍처(Instruction Set Architecture)에 맞아야 한다.

일반적인 컨테이너 이미지(Container Image)는 하나의 아키텍처에 해당하는 바이너리를 포함한다. 에뮬레이션(Emulation) 메커니즘이 없다면 AMD64 바이너리를 ARM64 프로세서에서 직접 실행하거나 그 반대로 실행하는 것은 일반적으로 실패한다. 다중 아키텍처 이미지 설계는 아키텍처별 이미지를 공통 이미지 참조(Common Image Reference)와 연결하여 컨테이너 런타임(Container Runtime)이 적절한 변형을 선택하도록 함으로써 이러한 배포 문제를 해결한다.

이러한 메커니즘은 일반적으로 이미지 인덱스(Image Index) 또는 매니페스트 목록(Manifest List)을 통해 구현된다. 하나의 태그(Tag)가 단일 아키텍처 이미지만 가리키는 대신 여러 플랫폼 변형(Platform Variant)을 설명하는 메타데이터를 참조한다. 클라이언트가 이미지를 가져오면 컨테이너 런타임은 호스트 운영체제와 프로세서 아키텍처를 확인하고 이에 맞는 이미지를 자동으로 가져온다.

예를 들어 하나의 로보틱스 릴리스에서 동일한 버전 태그 아래에 linux/amd64와 linux/arm64 변형을 함께 배포할 수 있다. AMD64 개발 워크스테이션에서는 AMD64 이미지를 받고 ARM64 엣지 컴퓨터에서는 ARM64 이미지를 받는다. 기본 바이너리와 일부 의존성이 서로 다르더라도 이를 통해 일관된 배포 인터페이스(Deployment Interface)를 제공할 수 있다.

Docker Buildx는 BuildKit 빌드 엔진(Build Engine)을 이용하여 다중 플랫폼 이미지(Multi-platform Image)를 생성하는 실용적인 방법을 제공한다. 빌드 과정에서 여러 대상 플랫폼(Target Platform)을 지정하고 하나의 Dockerfile로 아키텍처별 결과물을 생성할 수 있다. 생성된 이미지는 다중 플랫폼 매니페스트(Multi-platform Manifest)와 함께 레지스트리에 푸시하여 이후 시스템에서 하나의 논리적인 이미지 참조를 사용하도록 할 수 있다.

다중 아키텍처 빌드에서는 빌드 플랫폼(Build Platform)과 대상 플랫폼(Target Platform)을 명확하게 구분해야 한다. 빌드 플랫폼은 Docker 빌드 프로세스를 실행하는 아키텍처를 의미하고, 대상 플랫폼은 특정 이미지 단계가 생성되는 대상 아키텍처를 의미한다. 필요한 경우 Dockerfile 변수와 BuildKit 플랫폼 메타데이터를 사용하여 아키텍처에 따라 달라지는 동작을 명시적으로 정의할 수 있다.

호스트와 다른 아키텍처의 이미지를 빌드하는 방법에는 여러 가지가 있다. 에뮬레이션(Emulation)은 QEMU 및 리눅스 바이너리 형식 등록(Linux Binary-format Registration)과 같은 메커니즘을 통해 다른 아키텍처의 바이너리를 실행한다. 이 방식은 CI와 비교적 단순한 빌드에서 편리하지만 대규모 C++, ROS 2, 컴퓨터 비전 및 AI 워크로드에서는 에뮬레이션 기반 컴파일이 네이티브 실행보다 상당히 느릴 수 있다.

네이티브 다중 노드 빌더(Native Multi-node Builder)는 또 다른 전략을 제공한다. AMD64 빌더에서 AMD64 산출물을 생성하고 ARM64 빌더에서 ARM64 산출물을 생성하면서 BuildKit이 각각의 결과를 통합할 수 있다. 네이티브 빌더는 에뮬레이션에 따른 성능 저하를 상당 부분 피할 수 있으므로 ROS 2 컴파일, CUDA 구성요소 또는 대규모 네이티브 의존성으로 인해 빌드 시간이 길어지는 로보틱스 프로젝트에 유용하다.

교차 컴파일(Cross-compilation)은 세 번째 접근 방식으로 임베디드 소프트웨어에서 특히 효과적일 수 있다. AMD64 시스템에서 실행되는 컴파일러가 적절한 교차 툴체인(Cross-toolchain)과 대상 시스템 루트(Target Sysroot)를 사용하여 ARM64 바이너리를 생성할 수 있다. 높은 성능을 제공할 수 있지만 호스트 구성요소가 대상 산출물에 혼입되지 않도록 빌드 시스템, 라이브러리, 패키지 검색 및 아키텍처별 의존성을 신중하게 설계해야 한다.

다단계 빌드(Multi-stage Build)는 다중 아키텍처 워크플로와 자연스럽게 결합된다. 빌더 단계(Builder Stage)는 아키텍처별 컴파일러와 개발 의존성을 포함하고, 런타임 단계(Runtime Stage)는 생성된 대상 바이너리와 필요한 라이브러리만 전달받을 수 있다. 이러한 분리를 통해 운영 이미지를 작게 유지하면서 빌드 호스트에 필요한 구성요소와 실제 로봇에 배포되는 구성요소를 명확하게 구분할 수 있다.

ROS 2는 많은 패키지가 네이티브 C 또는 C++ 코드를 포함하기 때문에 추가적인 호환성 계층(Compatibility Layer)을 발생시킨다. 순수 Python 기반 ROS 2 노드는 아키텍처별 변경이 상대적으로 적을 수 있지만 네이티브 패키지는 각각의 대상에 맞게 컴파일해야 한다. DDS 구현, 메시지 지원 라이브러리, 플러그인, 컴퓨터 비전 패키지 및 제조사 드라이버 역시 선택한 아키텍처와 ROS 2 배포판에 맞게 제공되고 호환되어야 한다.

가능하면 아키텍처에 독립적인 Dockerfile 설계(Architecture-independent Dockerfile Design)를 우선해야 한다. 파일 이름에 amd64 또는 arm64가 고정된 다운로드 주소를 사용하면 아키텍처를 동적으로 선택하지 않는 한 다중 플랫폼 빌드가 실패할 수 있다. 따라서 패키지 저장소, 설치 스크립트 및 외부에서 다운로드하는 바이너리가 현재 대상 플랫폼에 맞는 산출물을 선택하는지 확인해야 한다.

기본 이미지(Base Image) 역시 모든 대상 아키텍처를 지원해야 한다. 많은 공식 Linux 및 ROS 관련 이미지는 여러 플랫폼에 대한 변형을 제공하지만 일부 특수 이미지는 제한된 아키텍처만 지원할 수 있다. 선택한 기본 이미지나 필수 바이너리 의존성이 해당 아키텍처에서 제공되지 않는다면 단순히 linux/arm64를 지정하는 것만으로 유효한 ARM64 런타임 이미지를 생성할 수 없다.

GPU 기반 로보틱스(GPU-enabled Robotics)는 추가적인 제약을 발생시킨다. 개별 NVIDIA GPU를 장착한 AMD64 워크스테이션과 ARM64 기반 NVIDIA Jetson 플랫폼 모두 CUDA 기반 응용 프로그램을 사용할 수 있지만 두 소프트웨어 스택이 반드시 서로 호환되는 것은 아니다. CUDA 버전, TensorRT 패키지, 플랫폼 라이브러리, 장치 드라이버 및 Jetson 전용 구성요소를 각각의 하드웨어 및 운영 환경과 일치시켜야 한다.

따라서 하나의 Dockerfile을 사용한다고 해서 모든 명령과 의존성이 모든 아키텍처에서 완전히 동일해야 하는 것은 아니다. 공통 로직(Common Logic)은 가능한 한 공유하되 정당한 플랫폼별 차이는 명시적으로 처리할 수 있다. 목표는 서로 호환되지 않는 플랫폼을 인위적으로 동일한 구성으로 강제하는 것이 아니라 하드웨어별 런타임 요구사항을 인정하면서 불필요한 차이를 최소화하는 것이다.

CI/CD 파이프라인은 지원되는 각각의 아키텍처를 독립적으로 빌드하고 검증해야 한다. AMD64에서 컴파일이 성공했다고 해서 ARM64 이미지가 정상적으로 동작한다는 것을 의미하지 않는다. 각 변형은 플릿에 배포하기 전에 적절한 의존성 검사, 자동화 테스트, 취약점 검사(Vulnerability Scanning), 시작 검증(Startup Validation), 그리고 필요한 경우 대표적인 대상 장치를 이용한 하드웨어 인 더 루프 시험(Hardware-in-the-Loop Testing)을 거쳐야 한다.

빌드 과정에서 에뮬레이션을 사용하는 경우 테스트는 특히 중요하다. QEMU는 많은 설치 및 실행 경로를 확인할 수 있지만 실제 로봇 하드웨어, GPU 동작, 장치 드라이버, 타이밍 특성 또는 아키텍처별 성능을 완전히 재현할 수 없다. 따라서 센서, 가속기, 실시간 워크로드 또는 제조사 전용 하드웨어와 밀접하게 상호작용하는 소프트웨어는 실제 ARM64 환경에서 네이티브 검증(Native Validation)을 수행해야 한다.

이미지 태깅(Image Tagging)은 응용 프로그램 버전 식별과 플랫폼 선택을 분리해야 한다. 운영자가 각 프로세서 아키텍처마다 서로 다른 태그를 수동으로 선택하도록 하는 대신 공통 릴리스 태그(Common Release Tag)가 지원되는 변형을 포함하는 하나의 매니페스트를 참조하도록 구성할 수 있다. 아키텍처별 태그는 디버깅이나 추적성을 위해 유지할 수 있지만 일반적인 배포에서는 공통 다중 플랫폼 참조를 사용할 수 있다.

레지스트리(Registry)는 아키텍처별 이미지와 이를 연결하는 매니페스트 메타데이터를 저장한다. 따라서 대부분의 배포 도구에서는 이러한 배포 과정이 투명하게 처리된다. 플릿 관리 시스템(Fleet Management System)은 서로 다른 로봇에 동일한 응용 프로그램 버전을 요청할 수 있으며, 레지스트리 매니페스트에 해당 플랫폼이 존재한다면 각 로봇의 컨테이너 런타임이 자신의 프로세서 아키텍처에 맞는 변형을 가져온다.

소프트웨어 공급망 제어(Software Supply-chain Control)는 모든 아키텍처 변형에 적용되어야 한다. 각 이미지에는 서로 다른 바이너리와 경우에 따라 서로 다른 패키지 버전이 포함될 수 있으므로 AMD64 이미지의 검사 결과가 ARM64를 대표한다고 가정해서는 안 된다. 릴리스 기록에는 각 플랫폼 변형에 해당하는 이미지 다이제스트(Image Digest), 출처 정보(Provenance), 의존성 및 검증 상태를 보존해야 한다.

재현성(Reproducibility)을 확보하려면 외부 의존성도 제어해야 한다. AMD64 빌드와 ARM64 빌드가 버전 제약 없이 각각 패키지를 다운로드하면 불가피한 아키텍처 차이를 넘어 이미지 내용 자체가 달라질 수 있다. 가능한 경우 고정된 패키지 버전, 통제된 저장소, 결정적인 빌드 절차(Deterministic Build Procedure) 및 불변 릴리스 참조(Immutable Release Reference)를 사용하면 두 변형이 동일한 논리적 소프트웨어 릴리스를 나타내도록 할 수 있다.

로보틱스에서 다중 아키텍처 이미지는 보다 광범위한 소프트웨어 정의 플랫폼 전략(Software-defined Platform Strategy)을 지원한다. 개발자는 AMD64 노트북이나 워크스테이션에서 작업하고, CI 시스템은 확장 가능한 서버 인프라에서 빌드를 수행하며, 시뮬레이션은 고성능 x86 시스템에서 실행하고, 운영 소프트웨어는 ARM64 엣지 컴퓨터에 배포할 수 있다. 이를 위해 응용 프로그램 저장소와 릴리스 프로세스를 완전히 별도로 유지할 필요가 없다.

따라서 견고한 ARM64 및 AMD64 이미지 전략은 공통 소스 코드, 아키텍처 인식 Dockerfile(Architecture-aware Dockerfile), BuildKit 또는 이에 상응하는 빌드 인프라, 통제된 의존성, 아키텍처별 검증 및 레지스트리 매니페스트를 결합한다. 그 결과 하나의 범용 바이너리가 만들어지는 것이 아니라 검증된 플랫폼별 변형으로 구성된 하나의 통합 소프트웨어 릴리스가 만들어지며, 이를 개발과 CI에서 시뮬레이션, 엣지 배포 및 이기종 로봇 플릿(Heterogeneous Robot Fleet)까지 일관되게 전달할 수 있다.

##  

## 3.5. Docker Compose for Robot SW Local Dev Environment [w/Code]

![](images/image5.png){width="7.268055555555556in" height="7.268055555555556in"}

Docker Compose provides a declarative way to define and run multiple containers as one coordinated application environment. In robot software development, this is useful because a complete local system rarely consists of a single process. ROS 2 nodes, perception services, databases, simulation tools, monitoring components, AI inference servers, and supporting middleware can be represented as separate but connected services.

A Compose configuration is normally described in a YAML file that defines services and their runtime relationships. Each service can specify an image or Dockerfile, command, environment variables, volumes, networks, device mappings, resource settings, and other parameters. Instead of manually starting many containers with long command lines, developers can describe the intended local robot environment once and recreate it consistently.

The service is the central abstraction in Docker Compose. A service represents a containerized application role such as navigation, perception, localization, simulation, database, or fleet communication. Multiple services can use different images and dependency sets while participating in the same local application. This separation helps robot developers isolate software responsibilities without requiring every component to share one oversized environment.

For ROS 2 development, Compose can organize nodes or groups of nodes according to their dependency and deployment boundaries. A perception container may contain camera and vision libraries, while a navigation container contains mapping and planning packages. Another service can provide visualization or diagnostics. ROS 2 communication can then connect these services through the networking environment created for the Compose project.

Container networking requires careful attention because ROS 2 relies on DDS-based discovery and communication. Containers attached to the same Compose network can communicate through service names and container networking, but DDS discovery behavior may vary according to middleware implementation and network configuration. Multicast, host networking, interface selection, and DDS configuration must therefore be considered when communication crosses container boundaries.

Docker Compose automatically creates a project network in typical configurations, giving services a convenient private communication environment. Service names can function as resolvable network identities, reducing dependence on manually assigned IP addresses. For conventional client-server components such as databases, APIs, dashboards, and inference services, this model provides a straightforward way to connect robot software components during local development.

Volumes and bind mounts are especially important for iterative robot software development. Instead of rebuilding an image whenever a source file changes, a developer can mount the local ROS 2 workspace into a development container. Source code can then be edited using host tools while compilation and execution occur inside the controlled container environment, combining fast iteration with consistent dependencies.

Persistent volumes can be used for information that must survive container recreation. Databases, caches, maps, configuration repositories, or selected development artifacts may require persistent storage. In contrast, temporary build outputs or generated files can remain ephemeral. Separating persistent data from disposable container state makes local environments easier to reset without accidentally deleting valuable development information.

Robot-specific configuration should be externalized whenever practical. ROS parameters, DDS settings, simulation options, model paths, network endpoints, and feature switches can be provided through mounted configuration files or environment variables. This allows developers to reuse the same container images while changing the behavior of the local system without modifying or rebuilding the image itself.

Compose also provides a convenient way to express startup relationships between services. For example, an application service may depend on a database, simulator, or supporting backend. However, container startup order should not be confused with application readiness. A container may have started while its internal service is still initializing, so robust robot software should combine dependency declarations with health checks and retry-capable application behavior.

Health checks can improve local development by identifying whether a service is actually ready rather than merely running. This is useful for databases, web APIs, inference servers, and other components with clear readiness endpoints. ROS 2 services may require application-specific checks because process existence alone does not prove that topics are being published, sensors are available, or required nodes are communicating correctly.

Profiles can be used to activate optional groups of services for different development scenarios. A lightweight developer session might start only core ROS 2 services, while a simulation profile can add a simulator and visualization tools. Another profile may enable AI inference or diagnostics. This reduces the need to maintain several nearly identical Compose files for every local development combination.

Environment-specific overrides can further separate common definitions from machine-specific or workflow-specific settings. The shared Compose configuration can describe the standard robot software topology, while local adjustments handle paths, ports, hardware devices, debugging options, or development-only behavior. Careful separation prevents personal workstation settings from becoming embedded in the common deployment definition.

Hardware access introduces challenges that ordinary web application Compose environments rarely encounter. Robot developers may need cameras, serial interfaces, CAN adapters, LiDAR devices, USB peripherals, or GPUs inside containers. Compose can express device mappings and related runtime settings, but the host must still provide compatible drivers, permissions, device nodes, and hardware-specific runtime support.

GPU workloads require coordination between the container image and host GPU environment. A perception or AI inference service may need CUDA, TensorRT, or another acceleration stack while other services do not. Keeping GPU requirements isolated to the services that actually require them avoids unnecessarily enlarging every container and allows developers to represent heterogeneous computational roles within the same local environment.

Graphical robotics tools add another consideration. Applications such as RViz or simulation interfaces may require access to the host display system, GPU acceleration, or remote visualization mechanisms. A local Compose environment should expose only the resources required by those tools and should avoid making broad host access the default simply for convenience, particularly when the same configuration may later be reused by multiple developers.

Compose can also coordinate simulation-oriented workflows. A simulator service can generate virtual sensor data, while localization, perception, planning, and control services consume the same ROS 2 interfaces expected on a physical robot. This enables developers to replace selected hardware-facing services with simulated equivalents while preserving much of the surrounding software topology and communication structure.

A useful local environment should support debugging without becoming fundamentally different from production. Development containers may include debuggers, editors, source mounts, or additional diagnostic utilities, while production images remain minimal. Multi-stage Dockerfiles can provide separate development and runtime targets, and Compose can select the appropriate target or image according to the intended workflow.

Logging becomes easier to coordinate when services are managed together. Developers can inspect logs from multiple containers through the Compose project rather than opening separate terminal sessions for every process. Applications should still produce structured and meaningful output so that failures in perception, middleware, databases, or backend services can be correlated across the local robot software stack.

Resource management is useful when a development workstation runs many computationally intensive services simultaneously. Perception, simulation, AI inference, mapping, and visualization may compete for CPU, memory, and GPU resources. Appropriate limits and service separation can prevent one component from consuming the entire workstation, although local resource behavior should not automatically be assumed to represent embedded target performance.

Secrets should not be permanently written into the Compose file or container images. Registry credentials, API tokens, cloud access information, and private certificates require controlled handling. Environment files can improve convenience but are not inherently secure if committed to source control. Development workflows should separate non-sensitive configuration from credentials and use appropriate secret-management mechanisms where available.

Docker Compose is primarily valuable as a development and integration tool rather than as a complete robot fleet orchestrator. It can reproduce a multi-container robot software environment on a workstation, test machine, or individual edge system, but large-scale scheduling, fleet health management, staged rollout, failure recovery, and distributed orchestration generally require additional infrastructure beyond a local Compose project.

For CI workflows, Compose can create repeatable integration-test environments containing several dependent services. A pipeline can start the required containers, execute ROS 2 integration tests or API tests, collect logs and results, and then destroy the environment. This helps reproduce software interactions that cannot be validated by testing an individual container independently and brings local development closer to automated verification.

The greatest benefit of Docker Compose for robot development is the ability to describe the local software system as version-controlled configuration. New developers can recreate the intended environment, CI can instantiate similar service relationships, and teams can modify individual components without manually reconstructing the entire stack. This reduces workstation-specific configuration drift and improves reproducibility across the development organization.

A well-designed Compose environment therefore becomes a bridge between individual container images and the complete robot software topology. By coordinating services, networks, volumes, configuration, hardware access, health checks, and development profiles, it provides a practical foundation for local ROS 2 integration. Used together with disciplined Dockerfiles and CI/CD, it enables faster development while preserving a clear path toward production deployment.

Docker Compose는 여러 컨테이너(Container)를 하나의 통합된 응용 프로그램 환경(Application Environment)으로 정의하고 실행할 수 있도록 하는 선언적 방식(Declarative Method)을 제공한다. 로봇 소프트웨어 개발에서는 전체 로컬 시스템이 하나의 프로세스로만 구성되는 경우가 드물기 때문에 특히 유용하다. ROS 2 노드, 인식 서비스(Perception Service), 데이터베이스, 시뮬레이션 도구, 모니터링 구성요소, AI 추론 서버(Inference Server), 지원 미들웨어(Middleware)를 서로 분리되면서 연결된 서비스로 구성할 수 있다.

Compose 설정(Configuration)은 일반적으로 YAML 파일로 작성되며 서비스(Service)와 서비스 간 런타임 관계(Runtime Relationship)를 정의한다. 각 서비스에는 이미지 또는 Dockerfile, 명령(Command), 환경 변수(Environment Variable), 볼륨(Volume), 네트워크(Network), 장치 매핑(Device Mapping), 자원 설정(Resource Setting) 등의 매개변수를 지정할 수 있다. 개발자는 긴 명령어를 사용해 여러 컨테이너를 수동으로 실행하는 대신 원하는 로컬 로봇 환경을 한 번 정의하고 일관되게 재생성할 수 있다.

서비스(Service)는 Docker Compose의 핵심 추상화 개념이다. 하나의 서비스는 내비게이션(Navigation), 인식(Perception), 위치 추정(Localization), 시뮬레이션(Simulation), 데이터베이스 또는 플릿 통신(Fleet Communication)과 같은 컨테이너화된 응용 프로그램 역할을 나타낸다. 여러 서비스는 서로 다른 이미지와 의존성 집합을 사용하면서 동일한 로컬 응용 프로그램에 참여할 수 있으며, 모든 구성요소를 하나의 거대한 환경에 포함하지 않고 소프트웨어 역할을 분리할 수 있다.

ROS 2 개발에서 Compose는 노드(Node) 또는 노드 그룹을 의존성과 배포 경계(Deployment Boundary)에 따라 구성할 수 있다. 인식 컨테이너에는 카메라 및 비전 라이브러리를 포함하고 내비게이션 컨테이너에는 지도 작성과 경로 계획 패키지를 포함할 수 있다. 또 다른 서비스는 시각화(Visualization) 또는 진단(Diagnostics)을 제공할 수 있으며, ROS 2 통신은 Compose 프로젝트에서 생성한 네트워크 환경을 통해 이러한 서비스를 연결할 수 있다.

ROS 2는 DDS 기반 탐색(Discovery)과 통신에 의존하기 때문에 컨테이너 네트워킹(Container Networking)을 신중하게 구성해야 한다. 동일한 Compose 네트워크에 연결된 컨테이너는 서비스 이름과 컨테이너 네트워크를 통해 통신할 수 있지만 DDS 탐색 동작은 미들웨어 구현과 네트워크 설정에 따라 달라질 수 있다. 따라서 컨테이너 경계를 넘어 통신할 때는 멀티캐스트(Multicast), 호스트 네트워킹(Host Networking), 인터페이스 선택 및 DDS 설정을 함께 고려해야 한다.

Docker Compose는 일반적인 구성에서 프로젝트 네트워크(Project Network)를 자동으로 생성하여 서비스에 편리한 사설 통신 환경(Private Communication Environment)을 제공한다. 서비스 이름은 네트워크에서 해석 가능한 식별자(Network Identity)로 사용할 수 있으므로 수동으로 IP 주소를 할당해야 하는 필요성을 줄여준다. 데이터베이스, API, 대시보드(Dashboard), 추론 서비스와 같은 일반적인 클라이언트-서버 구성요소에서는 이러한 모델을 통해 로컬 개발 중 로봇 소프트웨어 구성요소를 쉽게 연결할 수 있다.

볼륨(Volume)과 바인드 마운트(Bind Mount)는 반복적인 로봇 소프트웨어 개발에서 특히 중요하다. 소스 파일이 변경될 때마다 이미지를 다시 빌드하는 대신 로컬 ROS 2 워크스페이스를 개발 컨테이너에 마운트할 수 있다. 그러면 호스트의 개발 도구를 이용해 소스 코드를 수정하면서 컴파일과 실행은 통제된 컨테이너 환경에서 수행할 수 있어 빠른 반복 개발과 일관된 의존성 환경을 동시에 확보할 수 있다.

영구 볼륨(Persistent Volume)은 컨테이너를 다시 생성한 후에도 유지해야 하는 정보에 사용할 수 있다. 데이터베이스, 캐시(Cache), 지도, 설정 저장소 또는 일부 개발 산출물은 영구 저장이 필요할 수 있다. 반면 임시 빌드 출력이나 생성 파일은 일시적인 상태로 유지할 수 있다. 영구 데이터와 폐기 가능한 컨테이너 상태를 분리하면 중요한 개발 정보를 실수로 삭제하지 않으면서 로컬 환경을 쉽게 초기화할 수 있다.

로봇별 설정(Robot-specific Configuration)은 가능한 경우 외부화(Externalization)해야 한다. ROS 매개변수(Parameter), DDS 설정, 시뮬레이션 옵션, 모델 경로, 네트워크 엔드포인트(Network Endpoint), 기능 스위치(Feature Switch)를 마운트된 설정 파일이나 환경 변수를 통해 제공할 수 있다. 이를 통해 동일한 컨테이너 이미지를 재사용하면서 이미지 자체를 수정하거나 다시 빌드하지 않고 로컬 시스템의 동작을 변경할 수 있다.

Compose는 서비스 사이의 시작 관계(Startup Relationship)를 표현하는 편리한 방법도 제공한다. 예를 들어 응용 프로그램 서비스는 데이터베이스, 시뮬레이터 또는 지원 백엔드(Backend)에 의존할 수 있다. 그러나 컨테이너 시작 순서와 응용 프로그램 준비 상태(Application Readiness)는 동일한 의미가 아니다. 컨테이너가 시작되었더라도 내부 서비스가 초기화 중일 수 있으므로 견고한 로봇 소프트웨어는 의존성 선언과 상태 확인(Health Check), 재시도 가능한 응용 프로그램 동작을 함께 사용해야 한다.

상태 확인(Health Check)은 서비스가 단순히 실행 중인지가 아니라 실제로 준비되었는지를 확인하여 로컬 개발 환경을 개선할 수 있다. 이는 명확한 준비 상태 엔드포인트(Readiness Endpoint)를 제공하는 데이터베이스, 웹 API, 추론 서버 등에 유용하다. ROS 2 서비스에서는 프로세스가 존재하는 것만으로 토픽(Topic)이 정상적으로 발행되고 센서를 사용할 수 있으며 필요한 노드가 통신하고 있다는 것을 보장하지 않기 때문에 응용 프로그램별 상태 확인이 필요할 수 있다.

프로파일(Profile)을 사용하면 서로 다른 개발 시나리오에 따라 선택적인 서비스 그룹을 활성화할 수 있다. 가벼운 개발 세션에서는 핵심 ROS 2 서비스만 시작하고 시뮬레이션 프로파일에서는 시뮬레이터와 시각화 도구를 추가할 수 있다. 또 다른 프로파일에서는 AI 추론이나 진단 기능을 활성화할 수 있다. 이를 통해 각각의 로컬 개발 조합을 위해 거의 동일한 여러 Compose 파일을 별도로 유지해야 하는 필요성을 줄일 수 있다.

환경별 오버라이드(Environment-specific Override)를 사용하면 공통 정의와 시스템 또는 워크플로별 설정을 추가로 분리할 수 있다. 공유 Compose 설정에서는 표준 로봇 소프트웨어 토폴로지(Software Topology)를 정의하고 로컬 조정에서는 경로, 포트, 하드웨어 장치, 디버깅 옵션 또는 개발 전용 동작을 처리할 수 있다. 이러한 분리를 통해 개인 워크스테이션의 설정이 공통 배포 정의에 포함되는 것을 방지할 수 있다.

하드웨어 접근(Hardware Access)은 일반적인 웹 응용 프로그램의 Compose 환경에서는 거의 발생하지 않는 문제를 로봇 개발 환경에 추가한다. 개발자는 카메라, 직렬 인터페이스(Serial Interface), CAN 어댑터, LiDAR 장치, USB 주변장치 또는 GPU를 컨테이너 내부에서 사용해야 할 수 있다. Compose는 장치 매핑과 관련 런타임 설정을 정의할 수 있지만 호스트는 여전히 호환되는 드라이버, 권한, 장치 노드(Device Node), 하드웨어별 런타임 지원을 제공해야 한다.

GPU 워크로드는 컨테이너 이미지와 호스트 GPU 환경 사이의 조정이 필요하다. 인식 또는 AI 추론 서비스에는 CUDA, TensorRT 또는 다른 가속 스택(Acceleration Stack)이 필요할 수 있지만 다른 서비스에는 필요하지 않을 수 있다. GPU 요구사항을 실제로 필요한 서비스에만 분리하면 모든 컨테이너가 불필요하게 커지는 것을 방지하고 동일한 로컬 환경에서 서로 다른 계산 역할(Computational Role)을 표현할 수 있다.

그래픽 기반 로보틱스 도구(Graphical Robotics Tool)는 추가적인 고려사항을 발생시킨다. RViz 또는 시뮬레이션 인터페이스와 같은 응용 프로그램은 호스트 디스플레이 시스템, GPU 가속 또는 원격 시각화(Remote Visualization) 메커니즘에 접근해야 할 수 있다. 로컬 Compose 환경에서는 이러한 도구에 필요한 자원만 노출해야 하며, 특히 동일한 설정을 여러 개발자가 재사용할 수 있는 경우 편의를 위해 광범위한 호스트 접근 권한을 기본값으로 제공하는 것은 피해야 한다.

Compose는 시뮬레이션 중심 워크플로(Simulation-oriented Workflow)를 조정하는 데에도 사용할 수 있다. 시뮬레이터 서비스가 가상 센서 데이터를 생성하고 위치 추정, 인식, 경로 계획 및 제어 서비스가 실제 로봇에서 사용하는 것과 동일한 ROS 2 인터페이스를 사용할 수 있다. 이를 통해 주변 소프트웨어 토폴로지와 통신 구조를 대부분 유지하면서 일부 하드웨어 연결 서비스를 시뮬레이션된 구성요소로 대체할 수 있다.

효과적인 로컬 환경은 운영 환경과 근본적으로 달라지지 않으면서 디버깅(Debugging)을 지원해야 한다. 개발 컨테이너에는 디버거, 편집기, 소스 마운트 또는 추가 진단 유틸리티를 포함할 수 있지만 운영 이미지는 최소한으로 유지할 수 있다. 다단계 Dockerfile(Multi-stage Dockerfile)을 통해 개발용 및 런타임용 대상을 별도로 제공하고 Compose에서는 워크플로 목적에 따라 적절한 대상이나 이미지를 선택할 수 있다.

서비스를 함께 관리하면 로깅(Logging)도 보다 쉽게 조정할 수 있다. 개발자는 각 프로세스마다 별도의 터미널 세션을 열지 않고 Compose 프로젝트를 통해 여러 컨테이너의 로그를 확인할 수 있다. 응용 프로그램은 여전히 구조화되고 의미 있는 출력을 생성해야 하며 이를 통해 인식, 미들웨어, 데이터베이스 또는 백엔드 서비스에서 발생하는 오류를 로컬 로봇 소프트웨어 스택 전체에서 연관 지어 분석할 수 있어야 한다.

하나의 개발 워크스테이션에서 계산 집약적인 여러 서비스를 동시에 실행할 때는 자원 관리(Resource Management)가 유용하다. 인식, 시뮬레이션, AI 추론, 지도 작성 및 시각화가 CPU, 메모리 및 GPU 자원을 서로 경쟁할 수 있다. 적절한 자원 제한과 서비스 분리를 적용하면 하나의 구성요소가 전체 워크스테이션 자원을 독점하는 것을 방지할 수 있지만 로컬 환경의 자원 동작이 임베디드 대상의 성능을 그대로 나타낸다고 가정해서는 안 된다.

비밀 정보(Secret)는 Compose 파일이나 컨테이너 이미지에 영구적으로 기록해서는 안 된다. 레지스트리 인증 정보, API 토큰, 클라우드 접근 정보 및 비공개 인증서(Private Certificate)는 통제된 방식으로 관리해야 한다. 환경 파일(Environment File)은 편의성을 높일 수 있지만 소스 제어 시스템에 커밋하면 본질적으로 안전하지 않다. 개발 워크플로에서는 민감하지 않은 설정과 인증 정보를 분리하고 가능한 경우 적절한 비밀 관리 메커니즘(Secret-management Mechanism)을 사용해야 한다.

Docker Compose는 완전한 로봇 플릿 오케스트레이터(Robot Fleet Orchestrator)라기보다 개발 및 통합 도구(Development and Integration Tool)로서 특히 유용하다. 워크스테이션, 테스트 시스템 또는 개별 엣지 시스템에서 다중 컨테이너 로봇 소프트웨어 환경을 재현할 수 있지만 대규모 스케줄링, 플릿 상태 관리, 단계적 배포(Staged Rollout), 장애 복구 및 분산 오케스트레이션(Distributed Orchestration)에는 일반적으로 로컬 Compose 프로젝트 이상의 추가 인프라가 필요하다.

CI 워크플로에서 Compose는 서로 의존하는 여러 서비스를 포함하는 반복 가능한 통합 테스트 환경(Integration-test Environment)을 생성할 수 있다. 파이프라인은 필요한 컨테이너를 시작하고 ROS 2 통합 테스트 또는 API 테스트를 수행하며 로그와 결과를 수집한 다음 환경을 제거할 수 있다. 이를 통해 개별 컨테이너 테스트만으로 검증할 수 없는 소프트웨어 상호작용을 재현하고 로컬 개발 환경을 자동화 검증 환경과 더욱 가깝게 만들 수 있다.

로봇 개발에서 Docker Compose의 가장 큰 장점은 로컬 소프트웨어 시스템을 버전 관리되는 설정(Version-controlled Configuration)으로 표현할 수 있다는 점이다. 새로운 개발자는 의도된 환경을 재현할 수 있고 CI는 유사한 서비스 관계를 생성할 수 있으며 팀은 전체 스택을 수동으로 다시 구성하지 않고 개별 구성요소를 변경할 수 있다. 이를 통해 개발 조직 전체에서 워크스테이션별 설정 편차(Configuration Drift)를 줄이고 재현성(Reproducibility)을 향상시킬 수 있다.

따라서 잘 설계된 Compose 환경은 개별 컨테이너 이미지와 전체 로봇 소프트웨어 토폴로지 사이를 연결하는 역할을 한다. 서비스, 네트워크, 볼륨, 설정, 하드웨어 접근, 상태 확인 및 개발 프로파일을 통합적으로 조정함으로써 로컬 ROS 2 통합을 위한 실용적인 기반을 제공한다. 체계적인 Dockerfile 및 CI/CD와 함께 사용하면 운영 배포(Production Deployment)로 이어지는 명확한 경로를 유지하면서 더욱 빠르고 일관된 로봇 소프트웨어 개발을 가능하게 한다.

##  

## 3.6. NVIDIA Container Toolkit GPU Access in Containers [w/Code]

![](images/image6.png){width="7.268055555555556in" height="7.268055555555556in"}

The NVIDIA Container Toolkit enables containerized applications to access NVIDIA GPUs while preserving the isolation and reproducibility benefits of container deployment. This capability is especially important in robotics, where perception, deep-learning inference, visual SLAM, simulation, sensor processing, and foundation-model workloads increasingly depend on GPU acceleration at development, edge, and deployment stages.

A standard container does not automatically receive direct access to an NVIDIA GPU. Containers share the host kernel, but GPU devices, driver libraries, and supporting runtime components must be deliberately exposed. The NVIDIA Container Toolkit connects the container runtime with the NVIDIA driver environment so that GPU-enabled applications can execute without installing an independent kernel-level GPU driver inside every container.

The architecture separates responsibilities between the host and container. The host operating system provides the physical GPU and compatible NVIDIA kernel driver, while the container contains application-level components such as CUDA runtime libraries, TensorRT, PyTorch, computer-vision libraries, and robot software. This boundary allows multiple containers to use host GPU resources while maintaining independently versioned application environments.

The NVIDIA Container Toolkit integrates with container engines such as Docker through NVIDIA\'s container runtime infrastructure. When a GPU-enabled container is launched, the runtime identifies the requested GPU resources and injects the necessary device interfaces and compatible driver-side libraries into the container environment. Applications inside the container can then communicate with the GPU through the host driver.

This architecture explains why installing a complete NVIDIA kernel driver inside the container is generally unnecessary and undesirable. Kernel drivers operate as part of the host operating system and must correspond to the installed hardware and kernel environment. Container images should instead contain the user-space GPU software required by the application while relying on the host to provide the actual device driver.

CUDA compatibility is therefore a central design consideration. A container may package a particular CUDA user-space environment, but the host NVIDIA driver must support the CUDA requirements of that container. Selecting arbitrary combinations of driver, CUDA, deep-learning framework, and inference runtime versions can cause initialization failures or missing functionality even when the container itself builds successfully.

GPU access can be requested when starting a Docker container rather than permanently embedding a physical GPU into the image. This distinction is fundamental because container images describe portable software environments, while device assignment belongs to runtime configuration. The same image can consequently run with different available GPUs or without GPU access when the application supports an appropriate fallback mode.

GPU visibility can also be restricted to selected devices. On a multi-GPU workstation or robot computer, one container may use one GPU while another service uses a different GPU. Explicit device assignment helps isolate workloads such as perception, mapping, large-model inference, or simulation and prevents applications from automatically assuming unrestricted access to every accelerator installed in the system.

Environment variables and runtime configuration can further control which GPUs and driver capabilities are exposed. This is useful when containers require only compute functionality or when multiple services share the same host. However, GPU visibility is not equivalent to complete resource isolation. Applications sharing one physical GPU may still compete for memory capacity, compute cycles, bandwidth, and scheduling opportunities.

This distinction is important for robotics because GPU memory exhaustion can affect several safety-relevant software functions simultaneously. A large perception model or temporary tensor allocation may consume memory required by another inference process. GPU workload architecture should therefore consider memory budgets, process separation, model scheduling, monitoring, and platform-specific resource controls rather than relying only on container boundaries.

Docker Compose can represent GPU requirements alongside other robot services. A perception service may request GPU access while localization, database, communication, and diagnostics services remain CPU-only. This allows the local development topology to express computational roles explicitly and avoids adding CUDA or AI dependencies to containers that do not need GPU acceleration.

GPU-enabled images should be based on carefully selected CUDA or application framework images rather than assembled from unrelated binary components without compatibility planning. The operating-system version, CUDA runtime, cuDNN where required, TensorRT, Python packages, and ROS 2 dependencies form a compatibility chain. A change in one layer can influence whether the final application loads libraries and initializes the GPU correctly.

Multi-stage builds are useful for reducing the size of GPU-enabled robot images. A builder stage may include CUDA development tools, compilers, headers, model conversion utilities, and other large dependencies. The runtime stage can then contain only compiled artifacts, required CUDA runtime libraries, inference engines, optimized models, and the ROS 2 components needed during actual robot operation.

Development and production GPU images may consequently have different purposes. A development image can include compilers, profilers, debuggers, visualization tools, and source code, while a production image should normally contain a smaller validated runtime environment. Maintaining these roles through related build stages reduces duplication while preventing development utilities from unnecessarily expanding fleet deployment images.

NVIDIA Jetson platforms require additional attention because their software architecture differs from a conventional AMD64 workstation equipped with a discrete NVIDIA GPU. Jetson combines ARM64 processors with integrated NVIDIA acceleration and platform-specific software components. Container images intended for Jetson must therefore align with the supported Jetson software stack rather than assuming that an ordinary desktop CUDA image is directly interchangeable.

This difference becomes important in multi-architecture robot fleets. An AMD64 development workstation and an ARM64 Jetson may execute logically equivalent perception software, but their container variants can require different base images, CUDA components, TensorRT packages, and platform libraries. Multi-architecture manifests can provide a common release identity while retaining valid GPU software stacks for each target architecture.

ROS 2 GPU applications should keep hardware acceleration boundaries clear. Camera acquisition, preprocessing, neural-network inference, point-cloud processing, and visualization may have different GPU requirements. Placing every ROS 2 node into one GPU-enabled container can simplify initial development but may create excessive coupling, larger images, and more difficult resource management as the software stack grows.

A more modular architecture can expose GPU access only to services that need it. For example, an AI perception container can process camera frames and publish detection results through ROS 2, while navigation services consume those results without requiring CUDA libraries. This approach reduces dependency propagation and makes GPU-specific components easier to update, test, monitor, and replace independently.

Containerized GPU applications also require appropriate access to other robot hardware. A perception service may need both an NVIDIA GPU and one or more cameras, while a LiDAR processing service may require network or USB access. GPU enablement does not automatically provide access to these devices, so Docker or Compose runtime configuration must separately define the required device mappings, permissions, networking, and mounted resources.

Verification should occur at several levels. The host should first confirm that the NVIDIA driver recognizes the GPU correctly. A simple GPU-enabled container can then verify that the runtime exposes the accelerator, after which the actual CUDA, TensorRT, PyTorch, or ROS 2 application should be tested. Separating these checks helps distinguish host-driver failures from container-runtime problems and application dependency errors.

Monitoring is equally important after deployment. GPU utilization, memory consumption, temperature, power behavior, and application latency can reveal conditions that container health checks alone cannot detect. For edge robots operating continuously, GPU telemetry can be integrated with system diagnostics and fleet monitoring so that degraded acceleration performance or repeated memory pressure can be detected before broader application failure.

Security principles still apply when GPU devices are exposed. Containers should receive only the devices and capabilities they require, run as non-root users where practical, and avoid unnecessary privileged execution. Granting broad host access simply to make GPU software work weakens container isolation and can obscure the actual permissions required by the robot application.

CI/CD pipelines should distinguish builds that merely create a GPU image from tests that genuinely execute on GPU hardware. Many syntax, dependency, and unit tests can run without physical accelerators, but CUDA initialization, TensorRT engine execution, performance validation, and hardware-specific integration require GPU-capable runners. Production releases should therefore include appropriate native hardware validation before fleet deployment.

The NVIDIA Container Toolkit ultimately forms the bridge between portable containerized robot software and host-managed NVIDIA acceleration. By separating host drivers from application-level GPU libraries, controlling device exposure at runtime, and integrating with Docker, Compose, and CI/CD workflows, it enables reproducible GPU-enabled environments without abandoning the container model.

For modern robot software stacks, this capability supports a consistent path from GPU development workstations to AI-enabled edge computers and heterogeneous robot platforms. Reliable deployment depends on treating the GPU as an explicitly managed runtime resource, maintaining driver and CUDA compatibility, separating development from production images, validating target hardware, and monitoring accelerator behavior throughout the robot software lifecycle.

NVIDIA Container Toolkit은 컨테이너화된 응용 프로그램(Containerized Application)이 컨테이너 배포의 격리성(Isolation)과 재현성(Reproducibility)을 유지하면서 NVIDIA GPU에 접근할 수 있도록 한다. 이러한 기능은 인식(Perception), 딥러닝 추론(Deep-learning Inference), 비주얼 SLAM(Visual SLAM), 시뮬레이션, 센서 처리 및 파운데이션 모델(Foundation Model) 워크로드가 개발, 엣지 및 실제 배포 단계에서 GPU 가속에 점점 더 의존하는 로보틱스 분야에서 특히 중요하다.

일반적인 컨테이너(Container)는 NVIDIA GPU에 대한 직접적인 접근 권한을 자동으로 제공받지 않는다. 컨테이너는 호스트 커널(Host Kernel)을 공유하지만 GPU 장치, 드라이버 라이브러리(Driver Library) 및 관련 런타임 구성요소(Runtime Component)는 명시적으로 노출되어야 한다. NVIDIA Container Toolkit은 컨테이너 런타임(Container Runtime)과 NVIDIA 드라이버 환경을 연결하여 각 컨테이너 내부에 독립적인 커널 수준 GPU 드라이버를 설치하지 않고도 GPU 응용 프로그램을 실행할 수 있도록 한다.

이 아키텍처(Architecture)는 호스트와 컨테이너의 책임을 분리한다. 호스트 운영체제는 물리적 GPU와 호환되는 NVIDIA 커널 드라이버(Kernel Driver)를 제공하고, 컨테이너는 CUDA 런타임 라이브러리, TensorRT, PyTorch, 컴퓨터 비전 라이브러리 및 로봇 소프트웨어와 같은 응용 프로그램 수준 구성요소를 포함한다. 이러한 경계를 통해 여러 컨테이너가 호스트 GPU 자원을 사용하면서도 독립적으로 버전이 관리되는 응용 프로그램 환경을 유지할 수 있다.

NVIDIA Container Toolkit은 NVIDIA의 컨테이너 런타임 인프라(Container Runtime Infrastructure)를 통해 Docker와 같은 컨테이너 엔진(Container Engine)과 통합된다. GPU가 활성화된 컨테이너가 실행되면 런타임은 요청된 GPU 자원을 식별하고 필요한 장치 인터페이스(Device Interface)와 호환 가능한 드라이버 측 라이브러리를 컨테이너 환경에 제공한다. 이후 컨테이너 내부의 응용 프로그램은 호스트 드라이버를 통해 GPU와 통신할 수 있다.

이러한 아키텍처는 컨테이너 내부에 완전한 NVIDIA 커널 드라이버를 설치하는 것이 일반적으로 불필요하며 바람직하지 않은 이유를 설명한다. 커널 드라이버는 호스트 운영체제의 일부로 동작하며 설치된 하드웨어 및 커널 환경과 일치해야 한다. 따라서 컨테이너 이미지는 응용 프로그램에 필요한 사용자 공간 GPU 소프트웨어(User-space GPU Software)를 포함하고 실제 장치 드라이버는 호스트가 제공하도록 구성해야 한다.

따라서 CUDA 호환성(Compatibility)은 핵심적인 설계 고려사항이다. 컨테이너는 특정 CUDA 사용자 공간 환경(User-space Environment)을 포함할 수 있지만 호스트 NVIDIA 드라이버는 해당 컨테이너가 요구하는 CUDA 조건을 지원해야 한다. 드라이버, CUDA, 딥러닝 프레임워크(Deep-learning Framework), 추론 런타임(Inference Runtime)의 버전을 호환성 검토 없이 조합하면 컨테이너 자체의 빌드가 성공하더라도 초기화 실패나 기능 누락이 발생할 수 있다.

GPU 접근은 물리적 GPU를 이미지에 영구적으로 포함하는 것이 아니라 Docker 컨테이너를 시작할 때 요청할 수 있다. 이러한 구분은 컨테이너 이미지가 이식 가능한 소프트웨어 환경(Portable Software Environment)을 정의하는 반면 장치 할당(Device Assignment)은 런타임 설정에 속한다는 점에서 중요하다. 따라서 동일한 이미지를 서로 다른 GPU 구성에서 실행할 수 있으며 응용 프로그램이 적절한 대체 모드(Fallback Mode)를 지원한다면 GPU 접근 없이 실행하는 것도 가능하다.

GPU 가시성(GPU Visibility)은 선택된 장치로 제한할 수도 있다. 다중 GPU 워크스테이션이나 로봇 컴퓨터에서는 하나의 컨테이너가 특정 GPU를 사용하고 다른 서비스가 별도의 GPU를 사용할 수 있다. 명시적인 장치 할당은 인식, 지도 작성(Mapping), 대규모 모델 추론 또는 시뮬레이션과 같은 워크로드를 분리하고 응용 프로그램이 시스템에 설치된 모든 가속기에 무제한으로 접근한다고 가정하는 것을 방지한다.

환경 변수(Environment Variable)와 런타임 설정을 이용하면 노출되는 GPU와 드라이버 기능을 추가로 제어할 수 있다. 이는 컨테이너가 계산 기능만 필요하거나 여러 서비스가 동일한 호스트를 공유할 때 유용하다. 그러나 GPU 가시성이 완전한 자원 격리(Resource Isolation)를 의미하는 것은 아니다. 하나의 물리적 GPU를 공유하는 응용 프로그램은 여전히 메모리 용량, 연산 자원, 대역폭 및 스케줄링 기회를 두고 경쟁할 수 있다.

이러한 차이는 GPU 메모리 부족이 여러 안전 관련 소프트웨어 기능에 동시에 영향을 줄 수 있는 로보틱스 환경에서 중요하다. 대규모 인식 모델이나 일시적인 텐서(Tensor) 할당이 다른 추론 프로세스에 필요한 메모리를 소비할 수 있다. 따라서 GPU 워크로드 아키텍처는 단순히 컨테이너 경계에 의존하기보다 메모리 예산(Memory Budget), 프로세스 분리, 모델 스케줄링, 모니터링 및 플랫폼별 자원 제어를 함께 고려해야 한다.

Docker Compose는 다른 로봇 서비스와 함께 GPU 요구사항을 표현할 수 있다. 인식 서비스는 GPU 접근을 요청하고 위치 추정, 데이터베이스, 통신 및 진단 서비스는 CPU 전용으로 유지할 수 있다. 이를 통해 로컬 개발 토폴로지(Development Topology)에서 계산 역할을 명시적으로 표현하고 GPU 가속이 필요하지 않은 컨테이너에 CUDA 또는 AI 의존성을 불필요하게 추가하는 것을 방지할 수 있다.

GPU 지원 이미지(GPU-enabled Image)는 호환성 계획 없이 서로 관련 없는 바이너리 구성요소를 조합하기보다 신중하게 선정한 CUDA 또는 응용 프로그램 프레임워크 이미지를 기반으로 구성해야 한다. 운영체제 버전, CUDA 런타임, 필요한 경우 cuDNN, TensorRT, Python 패키지 및 ROS 2 의존성은 하나의 호환성 체인(Compatibility Chain)을 형성한다. 한 계층의 변경이 최종 응용 프로그램의 라이브러리 로딩과 GPU 초기화 가능 여부에 영향을 줄 수 있다.

다단계 빌드(Multi-stage Build)는 GPU 기반 로봇 이미지의 크기를 줄이는 데 유용하다. 빌더 단계(Builder Stage)에는 CUDA 개발 도구, 컴파일러, 헤더, 모델 변환 유틸리티 및 기타 대규모 의존성을 포함할 수 있다. 이후 런타임 단계(Runtime Stage)에는 컴파일된 산출물, 필요한 CUDA 런타임 라이브러리, 추론 엔진(Inference Engine), 최적화된 모델 및 실제 로봇 운용에 필요한 ROS 2 구성요소만 포함할 수 있다.

따라서 개발용 GPU 이미지와 운영용 GPU 이미지(Production GPU Image)는 서로 다른 목적을 가질 수 있다. 개발 이미지는 컴파일러, 프로파일러(Profiler), 디버거, 시각화 도구 및 소스 코드를 포함할 수 있지만 운영 이미지는 일반적으로 더 작고 검증된 런타임 환경만 포함해야 한다. 관련된 빌드 단계를 통해 이러한 역할을 관리하면 중복을 줄이면서 개발 도구가 플릿 배포 이미지(Fleet Deployment Image)를 불필요하게 확장하는 것을 방지할 수 있다.

NVIDIA Jetson 플랫폼은 개별 NVIDIA GPU를 사용하는 일반적인 AMD64 워크스테이션과 소프트웨어 아키텍처가 다르기 때문에 추가적인 주의가 필요하다. Jetson은 ARM64 프로세서와 통합 NVIDIA 가속 기능 및 플랫폼별 소프트웨어 구성요소를 결합한다. 따라서 Jetson용 컨테이너 이미지는 일반적인 데스크톱 CUDA 이미지를 직접 호환되는 것으로 가정하지 않고 지원되는 Jetson 소프트웨어 스택(Software Stack)에 맞춰야 한다.

이러한 차이는 다중 아키텍처 로봇 플릿(Multi-architecture Robot Fleet)에서 중요해진다. AMD64 개발 워크스테이션과 ARM64 Jetson은 논리적으로 동일한 인식 소프트웨어를 실행할 수 있지만 각각의 컨테이너 변형(Container Variant)은 서로 다른 기본 이미지(Base Image), CUDA 구성요소, TensorRT 패키지 및 플랫폼 라이브러리를 필요로 할 수 있다. 다중 아키텍처 매니페스트(Multi-architecture Manifest)는 각 대상 아키텍처에 유효한 GPU 소프트웨어 스택을 유지하면서 공통 릴리스 식별자를 제공할 수 있다.

ROS 2 GPU 응용 프로그램에서는 하드웨어 가속 경계(Hardware Acceleration Boundary)를 명확하게 유지해야 한다. 카메라 획득(Camera Acquisition), 전처리(Preprocessing), 신경망 추론(Neural-network Inference), 포인트 클라우드 처리(Point-cloud Processing) 및 시각화는 서로 다른 GPU 요구사항을 가질 수 있다. 모든 ROS 2 노드를 하나의 GPU 지원 컨테이너에 배치하면 초기 개발은 단순해질 수 있지만 소프트웨어 스택이 커질수록 과도한 결합, 이미지 크기 증가 및 복잡한 자원 관리 문제가 발생할 수 있다.

보다 모듈화된 아키텍처(Modular Architecture)에서는 실제로 GPU가 필요한 서비스에만 GPU 접근을 제공할 수 있다. 예를 들어 AI 인식 컨테이너가 카메라 프레임을 처리하고 ROS 2를 통해 탐지 결과를 발행하면 내비게이션 서비스는 CUDA 라이브러리 없이 해당 결과를 사용할 수 있다. 이러한 접근 방식은 의존성 전파(Dependency Propagation)를 줄이고 GPU 전용 구성요소를 독립적으로 업데이트, 테스트, 모니터링 및 교체하기 쉽게 만든다.

컨테이너화된 GPU 응용 프로그램은 다른 로봇 하드웨어에도 적절하게 접근할 수 있어야 한다. 인식 서비스에는 NVIDIA GPU와 하나 이상의 카메라가 동시에 필요할 수 있으며 LiDAR 처리 서비스에는 네트워크 또는 USB 접근이 필요할 수 있다. GPU 활성화만으로 이러한 장치에 자동으로 접근할 수 있는 것은 아니므로 Docker 또는 Compose 런타임 설정에서 필요한 장치 매핑, 권한, 네트워킹 및 마운트 자원을 별도로 정의해야 한다.

검증(Verification)은 여러 수준에서 수행해야 한다. 먼저 호스트에서 NVIDIA 드라이버가 GPU를 정상적으로 인식하는지 확인해야 한다. 이후 간단한 GPU 지원 컨테이너를 이용하여 런타임이 가속기를 올바르게 노출하는지 검증하고, 마지막으로 실제 CUDA, TensorRT, PyTorch 또는 ROS 2 응용 프로그램을 시험해야 한다. 이러한 검증 단계를 분리하면 호스트 드라이버 문제와 컨테이너 런타임 문제, 응용 프로그램 의존성 오류를 구별하는 데 도움이 된다.

배포 이후에는 모니터링(Monitoring)도 중요하다. GPU 사용률, 메모리 소비량, 온도, 전력 동작 및 응용 프로그램 지연시간(Latency)은 단순한 컨테이너 상태 확인만으로 탐지할 수 없는 문제를 보여줄 수 있다. 지속적으로 운용되는 엣지 로봇에서는 GPU 원격 측정 정보(Telemetry)를 시스템 진단 및 플릿 모니터링과 통합하여 가속 성능 저하나 반복적인 메모리 압박을 전체 응용 프로그램 장애로 확대되기 전에 탐지할 수 있다.

GPU 장치를 노출하는 경우에도 보안 원칙(Security Principle)은 동일하게 적용된다. 컨테이너에는 필요한 장치와 기능만 제공하고 가능한 경우 비루트 사용자(Non-root User)로 실행하며 불필요한 특권 실행(Privileged Execution)을 피해야 한다. GPU 소프트웨어를 쉽게 실행하기 위해 광범위한 호스트 접근 권한을 부여하면 컨테이너 격리 수준이 약화되고 로봇 응용 프로그램이 실제로 요구하는 권한을 파악하기 어려워질 수 있다.

CI/CD 파이프라인은 단순히 GPU 이미지를 생성하는 빌드와 실제 GPU 하드웨어에서 실행되는 테스트를 구분해야 한다. 많은 구문 검사, 의존성 검사 및 단위 테스트(Unit Test)는 물리적 가속기 없이 수행할 수 있지만 CUDA 초기화, TensorRT 엔진 실행, 성능 검증 및 하드웨어별 통합 시험에는 GPU 지원 실행 환경(GPU-capable Runner)이 필요하다. 따라서 운영 릴리스는 플릿 배포 전에 적절한 네이티브 하드웨어 검증(Native Hardware Validation)을 포함해야 한다.

NVIDIA Container Toolkit은 궁극적으로 이식 가능한 컨테이너 기반 로봇 소프트웨어와 호스트에서 관리되는 NVIDIA 가속 기능을 연결하는 역할을 한다. 호스트 드라이버와 응용 프로그램 수준 GPU 라이브러리를 분리하고 런타임에서 장치 노출을 제어하며 Docker, Compose 및 CI/CD 워크플로와 통합함으로써 컨테이너 모델을 유지하면서 재현 가능한 GPU 지원 환경을 구축할 수 있도록 한다.

현대적인 로봇 소프트웨어 스택에서 이러한 기능은 GPU 개발 워크스테이션에서 AI 기반 엣지 컴퓨터 및 이기종 로봇 플랫폼(Heterogeneous Robot Platform)으로 이어지는 일관된 개발 및 배포 경로를 지원한다. 안정적인 배포를 위해서는 GPU를 명시적으로 관리되는 런타임 자원(Runtime Resource)으로 취급하고 드라이버와 CUDA 호환성을 유지하며 개발 이미지와 운영 이미지를 분리하고 대상 하드웨어를 검증하며 로봇 소프트웨어 수명주기 전체에서 가속기 동작을 지속적으로 모니터링해야 한다.

##  

## 3.7. Container Networking Modes Host Bridge Macvlan [w/Code]

![](images/image7.png){width="7.268055555555556in" height="7.268055555555556in"}

Container networking determines how containerized robot software communicates with other containers, the host operating system, sensors, edge computers, and external networks. Unlike conventional web applications, robotic systems frequently depend on multicast discovery, high-bandwidth sensor streams, low-latency control messages, and direct communication with physical devices. Choosing the correct networking mode is therefore an architectural decision rather than only a deployment detail.

Docker provides several networking models, among which host, bridge, and macvlan are particularly relevant to robotics. Each model creates a different relationship between the container and the physical network. Host networking minimizes network abstraction, bridge networking provides isolated virtual networks with address translation, and macvlan allows containers to appear as independent devices directly connected to the local network.

Bridge networking is the common default for Docker containers. Docker creates a virtual bridge on the host and connects containers to it through virtual network interfaces. Each container receives an internal IP address within the bridge subnet, while communication with networks outside the host normally passes through network address translation. This provides useful isolation while allowing containers to communicate with each other and external systems.

User-defined bridge networks improve multi-container development because containers can discover each other through service or container names instead of fixed IP addresses. A perception service can therefore communicate with a database, inference server, or monitoring service using stable logical names. Docker Compose commonly creates such a network automatically, making bridge mode convenient for reproducible local robot software environments.

Port publishing is required when services inside a bridge network must be accessed through the host network. A container may listen on an internal port while Docker maps that port to a selected host port. This approach works naturally for HTTP APIs, dashboards, databases, telemetry gateways, and many client-server applications, but becomes more complicated for protocols that dynamically use multiple ports or depend heavily on multicast communication.

ROS 2 communication requires particular attention in bridge networks because DDS discovery behavior depends on multicast, network interfaces, middleware configuration, and routing. Two ROS 2 containers on the same properly configured Docker network may communicate successfully, while communication between containers, the host, and external robots can require additional configuration. The result may also vary according to the selected DDS implementation and discovery mechanism.

Host networking removes much of this virtual networking boundary. A container using host mode shares the host network namespace rather than receiving an isolated container network interface. Applications inside the container can therefore use the host\'s interfaces and network addresses directly, avoiding the conventional bridge, network address translation, and explicit port-publishing mechanisms normally used by Docker.

This simplicity makes host networking attractive for ROS 2 development and robotics applications that rely on DDS discovery. Multicast packets and interfaces are often easier to manage when the application sees the same network environment as the host. Host mode can also eliminate some translation and forwarding overhead, although performance differences should be measured for the actual workload rather than assumed to be significant.

The primary tradeoff of host networking is reduced network isolation. Because the container shares the host network namespace, port conflicts can occur when multiple applications attempt to bind to the same port. Network exposure is also broader, and Docker cannot provide the same per-container network separation available with bridge networks. Host mode should therefore be selected for a clear communication requirement rather than simply used as a universal solution.

For robot computers running several containerized services, host networking may simplify ROS 2 communication but can make service boundaries less explicit. A perception container, navigation container, and diagnostics container may all observe the host\'s interfaces directly. Teams must consequently manage ports, middleware settings, security rules, and service ownership carefully to prevent accidental interference between independently maintained components.

Macvlan networking takes a different approach by assigning a distinct MAC address and network identity to each container. From the perspective of the physical network, a macvlan container can appear similar to an independent computer connected to the same Ethernet segment. It can receive its own IP address and communicate directly with other network devices without using the host\'s IP address as its primary network identity.

This model can be valuable when legacy equipment, industrial controllers, sensors, or robot subsystems expect each communicating endpoint to exist as a distinct device on the local network. A container can participate directly in an existing subnet and interact with devices using conventional Layer 2 and Layer 3 networking behavior. This can reduce the need for host-port mappings and make network topology more explicit.

Macvlan also introduces operational complexity. The physical network must accommodate additional MAC addresses, and IP address allocation must be coordinated to prevent conflicts with DHCP or statically configured devices. Switch configuration, VLAN policies, wireless interfaces, cloud environments, and security infrastructure may impose limitations. Macvlan should therefore be evaluated against the actual robot network rather than treated as a portable default.

A notable macvlan characteristic is that communication between the host and its macvlan containers is not always available through the same interface by default. This behavior can surprise developers who expect the host to communicate with a container exactly as another LAN device does. When host-to-container communication is required, an additional macvlan interface or another deliberate networking arrangement may be necessary.

The three modes therefore represent different isolation and connectivity priorities. Bridge mode favors container isolation and controlled service exposure, host mode favors direct use of the host networking environment, and macvlan favors independent network identities for containers. No single mode is optimal for every robot subsystem, and different services on the same robot may justify different networking strategies.

Sensor traffic is an important selection factor. Cameras, LiDAR units, radar devices, and other Ethernet sensors may produce continuous high-bandwidth streams using unicast, multicast, or vendor-specific protocols. The network design should consider packet rate, bandwidth, multicast behavior, socket requirements, and interface binding. Container networking must not become an unexamined bottleneck between physical sensing and perception software.

Latency-sensitive control traffic requires similar care. Containerization does not automatically guarantee deterministic networking, and selecting host mode alone does not make communication real-time. Scheduling, kernel configuration, middleware queues, DDS quality-of-service settings, network hardware, CPU contention, and application architecture can all influence latency and jitter. Networking mode is only one part of the end-to-end control path.

DDS quality-of-service policies remain logically separate from Docker networking. Reliability, durability, history depth, deadline, and related ROS 2 communication properties are configured at the middleware level, while Docker networking determines whether packets can reach the required endpoints. A correct DDS configuration cannot compensate for blocked multicast or incorrect routing, just as a working network cannot correct incompatible QoS policies.

Multi-homed robot computers add another layer of complexity. A robot may simultaneously use Ethernet for LiDAR, a second Ethernet interface for control equipment, Wi-Fi for local access, and LTE or 5G for cloud connectivity. Containers must use the intended interfaces rather than unintentionally exposing discovery or traffic across every available network. Explicit interface selection can improve reliability, security, and bandwidth management.

Network security should follow the principle of least exposure. Services that need only internal communication can remain on isolated bridge networks, while only required APIs or gateways are published externally. Host or macvlan networking should not automatically imply unrestricted access. Host firewalls, VLAN segmentation, application authentication, encrypted transport, and robot-specific network policies remain important regardless of container mode.

Docker Compose can describe networks alongside services, allowing developers to represent communication boundaries as part of the robot software configuration. Separate networks can isolate backend services from sensor-facing components, while selected containers participate in multiple networks when acting as gateways. This makes network topology version-controlled and reproducible rather than dependent on manually configured development machines.

Testing should include the actual communication patterns expected on the robot. Successful ICMP connectivity or a simple TCP connection does not prove that ROS 2 discovery, multicast sensor streams, high-rate UDP traffic, or failover behavior will work correctly. Integration tests should verify discovery, topic communication, bandwidth, latency, reconnection, and behavior after container restart or network interruption.

Simulation environments may use simpler bridge networks because all components execute on one development host, while physical robots can require host networking or carefully designed macvlan connections for hardware integration. The difference should be represented explicitly in configuration rather than hidden in manual setup. Environment-specific Compose files or overrides can preserve common service definitions while adapting networking to each target.

A robust robot networking strategy therefore begins with communication requirements rather than a preferred Docker mode. Bridge networking is appropriate when isolation and controlled exposure dominate, host networking is useful when direct interface visibility and discovery simplicity are required, and macvlan is valuable when containers must behave as independent LAN devices. The final choice should reflect ROS 2 behavior, hardware interfaces, security boundaries, and deployment topology.

Container networking ultimately connects software isolation with the physical communication architecture of the robot. Treating host, bridge, and macvlan as deliberate architectural tools allows developers to balance reproducibility, connectivity, isolation, performance, and maintainability. When combined with explicit DDS configuration, network security, monitoring, and realistic hardware testing, containerized robot services can communicate reliably from local development to deployed fleets.

컨테이너 네트워킹(Container Networking)은 컨테이너화된 로봇 소프트웨어가 다른 컨테이너, 호스트 운영체제, 센서, 엣지 컴퓨터 및 외부 네트워크와 통신하는 방식을 결정한다. 일반적인 웹 응용 프로그램과 달리 로봇 시스템은 멀티캐스트 탐색(Multicast Discovery), 고대역폭 센서 스트림(High-bandwidth Sensor Stream), 저지연 제어 메시지(Low-latency Control Message), 물리적 장치와의 직접 통신에 자주 의존한다. 따라서 올바른 네트워킹 모드를 선택하는 것은 단순한 배포 설정이 아니라 아키텍처 설계 결정이다.

Docker는 여러 네트워킹 모델(Networking Model)을 제공하며 그중 호스트(Host), 브리지(Bridge), 맥브이랜(Macvlan)은 로보틱스에서 특히 중요하다. 각 모델은 컨테이너와 물리적 네트워크 사이에 서로 다른 관계를 형성한다. 호스트 네트워킹은 네트워크 추상화를 최소화하고, 브리지 네트워킹은 주소 변환과 함께 격리된 가상 네트워크를 제공하며, Macvlan은 컨테이너가 로컬 네트워크에 직접 연결된 독립적인 장치처럼 동작하도록 한다.

브리지 네트워킹(Bridge Networking)은 Docker 컨테이너에서 일반적으로 사용되는 기본 방식이다. Docker는 호스트에 가상 브리지(Virtual Bridge)를 생성하고 가상 네트워크 인터페이스를 통해 컨테이너를 연결한다. 각 컨테이너는 브리지 서브넷(Bridge Subnet) 내부의 IP 주소를 할당받으며 호스트 외부 네트워크와의 통신은 일반적으로 네트워크 주소 변환(Network Address Translation)을 거친다. 이를 통해 컨테이너 간 및 외부 시스템과의 통신을 허용하면서 유용한 격리 기능을 제공한다.

사용자 정의 브리지 네트워크(User-defined Bridge Network)는 컨테이너가 고정 IP 주소 대신 서비스 또는 컨테이너 이름을 이용해 서로를 탐색할 수 있으므로 다중 컨테이너 개발 환경을 개선한다. 따라서 인식 서비스는 안정적인 논리적 이름을 이용하여 데이터베이스, 추론 서버 또는 모니터링 서비스와 통신할 수 있다. Docker Compose는 일반적으로 이러한 네트워크를 자동으로 생성하므로 브리지 모드는 재현 가능한 로컬 로봇 소프트웨어 환경에 적합하다.

브리지 네트워크 내부의 서비스를 호스트 네트워크를 통해 접근해야 할 경우 포트 공개(Port Publishing)가 필요하다. 컨테이너는 내부 포트에서 수신하고 Docker는 해당 포트를 선택된 호스트 포트에 매핑할 수 있다. 이러한 방식은 HTTP API, 대시보드, 데이터베이스, 텔레메트리 게이트웨이(Telemetry Gateway) 및 여러 클라이언트-서버 응용 프로그램에 적합하지만 여러 포트를 동적으로 사용하거나 멀티캐스트 통신에 크게 의존하는 프로토콜에서는 복잡해질 수 있다.

ROS 2 통신은 DDS 탐색 동작(DDS Discovery Behavior)이 멀티캐스트, 네트워크 인터페이스, 미들웨어 설정 및 라우팅에 의존하므로 브리지 네트워크에서 특별한 주의가 필요하다. 적절하게 구성된 동일한 Docker 네트워크의 두 ROS 2 컨테이너는 정상적으로 통신할 수 있지만 컨테이너, 호스트 및 외부 로봇 사이의 통신에는 추가 설정이 필요할 수 있다. 결과는 선택한 DDS 구현과 탐색 메커니즘(Discovery Mechanism)에 따라서도 달라질 수 있다.

호스트 네트워킹(Host Networking)은 이러한 가상 네트워크 경계의 상당 부분을 제거한다. 호스트 모드를 사용하는 컨테이너는 격리된 컨테이너 네트워크 인터페이스를 할당받는 대신 호스트 네트워크 네임스페이스(Network Namespace)를 공유한다. 따라서 컨테이너 내부의 응용 프로그램은 호스트의 인터페이스와 네트워크 주소를 직접 사용할 수 있으며 일반적인 Docker 브리지, 네트워크 주소 변환 및 명시적인 포트 공개 메커니즘을 우회할 수 있다.

이러한 단순성으로 인해 호스트 네트워킹은 DDS 탐색에 의존하는 ROS 2 개발과 로보틱스 응용 프로그램에서 유용할 수 있다. 응용 프로그램이 호스트와 동일한 네트워크 환경을 사용하면 멀티캐스트 패킷과 네트워크 인터페이스를 보다 쉽게 관리할 수 있다. 또한 일부 변환 및 포워딩 오버헤드(Forwarding Overhead)를 제거할 수 있지만 실제 성능 차이는 크다고 가정하기보다 대상 워크로드에서 직접 측정해야 한다.

호스트 네트워킹의 주요 절충점(Tradeoff)은 네트워크 격리 수준이 감소한다는 것이다. 컨테이너가 호스트 네트워크 네임스페이스를 공유하므로 여러 응용 프로그램이 동일한 포트에 바인딩하려고 하면 포트 충돌이 발생할 수 있다. 네트워크 노출 범위도 넓어지고 Docker가 브리지 네트워크에서 제공하는 컨테이너별 네트워크 분리를 동일하게 제공할 수 없다. 따라서 호스트 모드는 명확한 통신 요구사항이 있을 때 선택해야 하며 보편적인 해결책으로 사용해서는 안 된다.

여러 컨테이너화된 서비스를 실행하는 로봇 컴퓨터에서는 호스트 네트워킹이 ROS 2 통신을 단순화할 수 있지만 서비스 경계(Service Boundary)를 불명확하게 만들 수 있다. 인식 컨테이너, 내비게이션 컨테이너 및 진단 컨테이너가 모두 호스트 인터페이스를 직접 볼 수 있기 때문이다. 따라서 독립적으로 관리되는 구성요소 사이의 의도하지 않은 간섭을 방지하려면 포트, 미들웨어 설정, 보안 규칙 및 서비스 소유권을 신중하게 관리해야 한다.

Macvlan 네트워킹(Macvlan Networking)은 각 컨테이너에 고유한 MAC 주소와 네트워크 식별자(Network Identity)를 할당하는 다른 접근 방식을 사용한다. 물리적 네트워크의 관점에서 Macvlan 컨테이너는 동일한 이더넷 세그먼트(Ethernet Segment)에 연결된 독립적인 컴퓨터처럼 보일 수 있다. 자체 IP 주소를 할당받고 호스트 IP 주소를 기본 네트워크 식별자로 사용하지 않으면서 다른 네트워크 장치와 직접 통신할 수 있다.

이 모델은 레거시 장비(Legacy Equipment), 산업용 제어기(Industrial Controller), 센서 또는 로봇 하위 시스템이 각각의 통신 엔드포인트를 로컬 네트워크의 독립된 장치로 인식해야 할 때 유용할 수 있다. 컨테이너는 기존 서브넷에 직접 참여하여 일반적인 계층 2(Layer 2) 및 계층 3(Layer 3) 네트워킹 방식으로 장치와 통신할 수 있다. 이를 통해 호스트 포트 매핑의 필요성을 줄이고 네트워크 토폴로지(Network Topology)를 보다 명확하게 구성할 수 있다.

Macvlan은 운영상의 복잡성도 추가한다. 물리적 네트워크는 추가적인 MAC 주소를 처리할 수 있어야 하며 DHCP 또는 정적 설정 장치와 IP 주소가 충돌하지 않도록 주소 할당을 조정해야 한다. 스위치 설정, VLAN 정책, 무선 인터페이스, 클라우드 환경 및 보안 인프라에 따라 제한이 발생할 수 있다. 따라서 Macvlan은 이식 가능한 기본 설정으로 간주하기보다 실제 로봇 네트워크 환경을 기준으로 평가해야 한다.

Macvlan의 중요한 특성 중 하나는 동일한 인터페이스를 통한 호스트와 Macvlan 컨테이너 사이의 통신이 기본적으로 항상 가능하지는 않다는 점이다. 이는 호스트가 다른 LAN 장치와 동일한 방식으로 컨테이너와 통신할 것이라고 예상하는 개발자에게 혼란을 줄 수 있다. 호스트와 컨테이너 사이의 통신이 필요한 경우 추가 Macvlan 인터페이스 또는 별도로 설계된 네트워킹 구성이 필요할 수 있다.

따라서 세 가지 모드는 서로 다른 격리 및 연결 우선순위(Isolation and Connectivity Priority)를 나타낸다. 브리지 모드는 컨테이너 격리와 통제된 서비스 노출을 중시하고, 호스트 모드는 호스트 네트워크 환경의 직접적인 사용을 중시하며, Macvlan은 컨테이너에 독립적인 네트워크 식별자를 제공하는 데 중점을 둔다. 모든 로봇 하위 시스템에 최적인 단일 모드는 없으며 동일한 로봇에서도 서비스별로 서로 다른 네트워킹 전략이 적합할 수 있다.

센서 트래픽(Sensor Traffic)은 네트워킹 모드를 선택할 때 중요한 요소이다. 카메라, LiDAR, 레이더 및 기타 이더넷 센서는 유니캐스트(Unicast), 멀티캐스트 또는 제조사별 프로토콜을 이용하여 지속적인 고대역폭 스트림을 생성할 수 있다. 네트워크 설계에서는 패킷 전송률, 대역폭, 멀티캐스트 동작, 소켓 요구사항 및 인터페이스 바인딩을 고려해야 한다. 컨테이너 네트워킹이 물리적 센싱과 인식 소프트웨어 사이에서 검증되지 않은 병목지점(Bottleneck)이 되어서는 안 된다.

지연시간에 민감한 제어 트래픽(Latency-sensitive Control Traffic) 역시 신중하게 처리해야 한다. 컨테이너화가 자동으로 결정론적 네트워킹(Deterministic Networking)을 보장하지 않으며 호스트 모드를 선택하는 것만으로 통신이 실시간화되는 것도 아니다. 스케줄링, 커널 설정, 미들웨어 큐, DDS 서비스 품질(Quality of Service) 설정, 네트워크 하드웨어, CPU 경합 및 응용 프로그램 아키텍처가 모두 지연시간과 지터(Jitter)에 영향을 줄 수 있다.

DDS 서비스 품질 정책(Quality-of-Service Policy)은 Docker 네트워킹과 논리적으로 분리되어 있다. 신뢰성(Reliability), 지속성(Durability), 이력 깊이(History Depth), 데드라인(Deadline) 및 관련 ROS 2 통신 특성은 미들웨어 수준에서 설정되며 Docker 네트워킹은 패킷이 필요한 엔드포인트에 도달할 수 있는지를 결정한다. 올바른 DDS 설정으로 차단된 멀티캐스트나 잘못된 라우팅을 해결할 수 없으며 정상적인 네트워크 역시 서로 호환되지 않는 QoS 정책을 해결할 수 없다.

다중 네트워크 인터페이스를 갖는 로봇 컴퓨터(Multi-homed Robot Computer)는 추가적인 복잡성을 발생시킨다. 로봇은 LiDAR용 이더넷, 제어 장비용 두 번째 이더넷 인터페이스, 로컬 접근용 Wi-Fi, 클라우드 연결용 LTE 또는 5G를 동시에 사용할 수 있다. 컨테이너는 의도된 인터페이스를 사용해야 하며 탐색 또는 데이터 트래픽이 모든 네트워크로 의도치 않게 노출되지 않도록 해야 한다. 명시적인 인터페이스 선택은 신뢰성, 보안 및 대역폭 관리를 향상시킬 수 있다.

네트워크 보안(Network Security)은 최소 노출 원칙(Principle of Least Exposure)을 따라야 한다. 내부 통신만 필요한 서비스는 격리된 브리지 네트워크에 유지하고 필요한 API 또는 게이트웨이만 외부에 공개할 수 있다. 호스트 또는 Macvlan 네트워킹을 사용한다고 해서 무제한 접근을 허용해야 하는 것은 아니다. 컨테이너 모드와 관계없이 호스트 방화벽, VLAN 분할, 응용 프로그램 인증, 암호화 전송 및 로봇별 네트워크 정책이 중요하다.

Docker Compose는 서비스와 함께 네트워크를 정의할 수 있으므로 개발자는 통신 경계(Communication Boundary)를 로봇 소프트웨어 설정의 일부로 표현할 수 있다. 별도의 네트워크를 사용하여 백엔드 서비스와 센서 연결 구성요소를 격리할 수 있으며 게이트웨이 역할을 하는 특정 컨테이너는 여러 네트워크에 참여할 수 있다. 이를 통해 네트워크 토폴로지를 개발 시스템에서 수동으로 설정하는 대신 버전 관리되고 재현 가능한 구성으로 만들 수 있다.

테스트는 실제 로봇에서 예상되는 통신 패턴을 포함해야 한다. ICMP 연결이나 단순한 TCP 연결이 성공했다고 해서 ROS 2 탐색, 멀티캐스트 센서 스트림, 고속 UDP 트래픽 또는 장애 전환(Failover)이 정상적으로 동작한다는 의미는 아니다. 통합 테스트(Integration Test)에서는 탐색, 토픽 통신, 대역폭, 지연시간, 재연결 및 컨테이너 재시작이나 네트워크 중단 이후의 동작을 검증해야 한다.

시뮬레이션 환경에서는 모든 구성요소가 하나의 개발 호스트에서 실행되므로 비교적 단순한 브리지 네트워크를 사용할 수 있지만 실제 로봇에서는 하드웨어 통합을 위해 호스트 네트워킹 또는 신중하게 설계된 Macvlan 연결이 필요할 수 있다. 이러한 차이는 수동 설정으로 숨기기보다 설정에 명시적으로 표현해야 한다. 환경별 Compose 파일이나 오버라이드(Override)를 이용하면 공통 서비스 정의를 유지하면서 각 대상에 맞게 네트워킹을 조정할 수 있다.

견고한 로봇 네트워킹 전략은 선호하는 Docker 모드가 아니라 통신 요구사항에서 시작해야 한다. 격리와 통제된 노출이 중요한 경우 브리지 네트워킹이 적합하고, 직접적인 인터페이스 가시성과 단순한 탐색이 필요한 경우 호스트 네트워킹이 유용하며, 컨테이너가 독립적인 LAN 장치처럼 동작해야 하는 경우 Macvlan이 유용하다. 최종 선택은 ROS 2 동작, 하드웨어 인터페이스, 보안 경계 및 배포 토폴로지를 반영해야 한다.

궁극적으로 컨테이너 네트워킹은 소프트웨어 격리(Software Isolation)와 로봇의 물리적 통신 아키텍처(Physical Communication Architecture)를 연결한다. Host, Bridge 및 Macvlan을 명확한 목적을 가진 아키텍처 도구로 활용하면 재현성, 연결성, 격리, 성능 및 유지보수성 사이의 균형을 조정할 수 있다. 명시적인 DDS 설정, 네트워크 보안, 모니터링 및 실제 하드웨어 시험과 결합하면 컨테이너화된 로봇 서비스는 로컬 개발 환경에서 실제 플릿 배포까지 안정적으로 통신할 수 있다.

##  

## 3.8. Container Security Rootless Read Only Seccomp AppArmor [w/Code]

![](images/image8.png){width="7.268055555555556in" height="7.268055555555556in"}

Container security is especially important in robotics because containers frequently interact with cameras, LiDAR sensors, serial interfaces, CAN devices, GPUs, network interfaces, and control processes. A compromised container may therefore affect more than application data. Security design should assume that software can fail or be exploited and should limit what each container can access, modify, and execute.

Containers provide process isolation, but they are not equivalent to independent virtual machines. They normally share the host Linux kernel, which means a vulnerability or excessive privilege can create a path from a container toward host resources. Secure robot deployments should therefore combine several protection mechanisms rather than assuming that containerization alone creates a sufficient security boundary.

The principle of least privilege provides the foundation for container security. Each robot service should receive only the permissions, devices, files, network access, and kernel capabilities required for its function. A navigation service does not automatically need camera devices, and a perception container does not necessarily require access to CAN control interfaces. Explicit boundaries reduce the impact of software defects and compromise.

Running containers as root creates unnecessary risk when applications do not require administrative privileges. Even though root inside a container is constrained by namespaces and other isolation mechanisms, excessive privileges can increase the consequences of configuration errors or vulnerabilities. Application processes should therefore run as non-root users whenever practical, with file ownership and permissions designed accordingly.

Rootless containers extend this idea by allowing the container engine and containers to operate without conventional host root privileges. Rootless operation uses mechanisms such as user namespaces to map container identities to unprivileged host identities. If a container or runtime component is compromised, the attacker encounters an additional privilege boundary before obtaining administrative control over the host.

Rootless operation also introduces limitations that must be considered in robotics. Direct access to hardware devices, privileged network configuration, low-numbered ports, GPU resources, and certain kernel features may require additional configuration or may not behave identically to rootful containers. Rootless mode should therefore be validated against the actual camera, CAN, serial, LiDAR, and accelerator requirements of the robot.

A read-only root filesystem provides another strong protection layer. In this configuration, the container\'s primary filesystem cannot be modified during normal execution. An application can read its binaries, libraries, and configuration but cannot arbitrarily replace them or permanently write new files into the image filesystem. This reduces opportunities for persistence and unintended modification after deployment.

Robot applications still require writable locations for temporary files, logs, caches, maps, databases, or generated artifacts. Instead of making the complete filesystem writable, specific directories can be provided through writable volumes, bind mounts, or temporary filesystems. This creates a clear distinction between immutable application content and the limited data locations that are intentionally writable.

An immutable container design also improves operational consistency. If applications cannot modify their installed software during execution, restarting a container returns it to a known image state. Software changes should then occur through a new image build and controlled deployment rather than manual modification inside a running robot. This aligns container security with reproducible DevOps and OTA update practices.

Linux capabilities provide finer-grained privilege control than granting unrestricted root authority. Traditional root privileges contain many distinct powers, such as changing network settings, manipulating processes, or performing system administration operations. Containers can drop unnecessary capabilities and add only specific capabilities when an application genuinely requires them, reducing the privilege available to compromised processes.

Privileged container mode should be avoided unless there is a carefully justified requirement. A privileged container receives broad access to host devices and kernel capabilities, significantly weakening the isolation boundary. During robotics development it may appear convenient because hardware immediately becomes accessible, but it can conceal which permissions are actually necessary and encourage insecure production configurations.

Seccomp, or Secure Computing Mode, restricts the Linux system calls that processes are permitted to invoke. Containers interact with the host kernel through system calls, so limiting this interface reduces the kernel functionality exposed to an application. Docker can apply a seccomp profile that blocks selected system calls while permitting those commonly required by ordinary containerized software.

This mechanism is valuable because many applications use only a subset of the Linux system-call interface. A ROS 2 node performing perception does not normally need every kernel operation available to a general-purpose root process. If exploited software attempts a prohibited system call, the seccomp policy can prevent that operation and reduce the available attack surface before deeper host resources are reached.

Custom seccomp profiles can provide stronger restrictions but require careful testing. Robotics software may use unusual system calls through middleware, shared-memory transport, hardware drivers, debugging tools, real-time scheduling, or GPU runtimes. An overly restrictive policy can therefore break valid behavior. Profiles should be refined using observed application requirements rather than disabling seccomp whenever incompatibilities appear.

AppArmor provides mandatory access control at a different layer. Instead of focusing primarily on system calls, AppArmor profiles can restrict which files, paths, capabilities, and other operating-system resources a process may access. A containerized application can therefore be constrained even when conventional Unix file permissions would otherwise allow broader access.

For a robot service, an AppArmor policy can help limit access to configuration directories, device resources, application files, or other host paths exposed to the container. This becomes particularly valuable when bind mounts are used. Mounting a host directory into a container creates a direct relationship with host data, so access should be limited to the smallest required path and preferably made read-only when modification is unnecessary.

Seccomp and AppArmor address different aspects of the security boundary and can be used together. Seccomp reduces the set of kernel operations available through system calls, while AppArmor constrains access according to security policy. Combined with namespaces, capabilities, non-root execution, and filesystem restrictions, these mechanisms form defense in depth rather than depending on a single protection layer.

Device exposure should follow the same principle. Robot containers often require \`/dev\` resources for cameras, serial adapters, CAN interfaces, or other peripherals. Mapping the entire host device space into a container is rarely necessary. Only required devices should be exposed, with appropriate ownership and permissions, so that compromise of one service does not automatically provide access to unrelated actuators or sensors.

Network exposure must also be minimized. Internal services can remain on isolated container networks, while only necessary APIs, telemetry gateways, or ROS 2 interfaces are reachable externally. Host networking can simplify robotics communication but reduces network separation, so firewall rules, interface selection, DDS configuration, authentication, and encryption become more important when containers directly share host networking resources.

Secrets require separate handling from ordinary configuration. API tokens, registry credentials, certificates, cloud keys, and other sensitive values should not be embedded into container images or committed into source repositories. They should be injected at runtime through controlled mechanisms and made available only to services that require them. Rebuilding an image should never be necessary merely to rotate a credential.

Container images themselves form part of the robot\'s software supply chain. Production images should originate from controlled base images, use pinned or otherwise managed dependencies, avoid unnecessary packages, and maintain traceable build records. Smaller images reduce both storage requirements and potential attack surface because fewer tools, libraries, shells, and utilities are available to an attacker after compromise.

Development and production security profiles may differ, but the differences should be explicit. Developers may temporarily require debugging tools, source mounts, profilers, or additional device access. Production containers should remove these conveniences unless operationally necessary. Maintaining separate development and runtime targets prevents temporary debugging requirements from silently becoming permanent fleet privileges.

Security validation should be integrated into CI/CD rather than performed only after a robot image is completed. Pipelines can inspect configuration, scan container images for known vulnerabilities, verify expected users and filesystem permissions, and check whether prohibited privileged settings are present. Security tests can also confirm that applications still function when unnecessary capabilities and writable paths are removed.

Runtime monitoring complements preventive controls. Unexpected process execution, unusual network connections, repeated permission failures, filesystem changes, or abnormal device access may indicate software defects or compromise. Robot fleet monitoring should therefore combine application health information with relevant container and host security telemetry, especially for remotely deployed systems that operate without continuous human supervision.

No single mechanism makes a container secure. Rootless execution reduces host privilege, read-only filesystems restrict modification, Linux capabilities limit administrative powers, seccomp reduces system-call exposure, and AppArmor constrains resource access. Their value is greatest when applied together according to the requirements of each robot service rather than enabled or disabled uniformly across the entire software stack.

A secure robot container architecture therefore begins by defining what each service genuinely needs. From that requirement, developers can select users, capabilities, devices, writable paths, networks, system calls, and access-control policies. Combined with controlled images, CI/CD validation, runtime monitoring, and disciplined updates, these layers create a practical defense-in-depth model for containerized robot software.

컨테이너 보안(Container Security)은 컨테이너가 카메라, LiDAR 센서, 직렬 인터페이스(Serial Interface), CAN 장치, GPU, 네트워크 인터페이스 및 제어 프로세스와 자주 상호작용하는 로보틱스에서 특히 중요하다. 침해된 컨테이너는 응용 프로그램 데이터 이상의 영역에 영향을 줄 수 있다. 따라서 보안 설계는 소프트웨어가 실패하거나 공격받을 수 있음을 전제로 하고 각 컨테이너가 접근, 수정 및 실행할 수 있는 범위를 제한해야 한다.

컨테이너는 프로세스 격리(Process Isolation)를 제공하지만 독립적인 가상 머신(Virtual Machine)과 동일하지는 않다. 일반적으로 호스트 리눅스 커널(Host Linux Kernel)을 공유하기 때문에 취약점이나 과도한 권한이 컨테이너에서 호스트 자원으로 접근하는 경로를 만들 수 있다. 따라서 안전한 로봇 배포에서는 컨테이너화 자체만으로 충분한 보안 경계가 형성된다고 가정하지 말고 여러 보호 메커니즘을 함께 적용해야 한다.

최소 권한 원칙(Principle of Least Privilege)은 컨테이너 보안의 기반을 제공한다. 각각의 로봇 서비스에는 해당 기능을 수행하는 데 필요한 권한, 장치, 파일, 네트워크 접근 및 커널 기능(Kernel Capability)만 제공해야 한다. 내비게이션 서비스에 카메라 장치가 반드시 필요한 것은 아니며 인식 컨테이너가 CAN 제어 인터페이스에 접근할 필요도 없다. 명확한 경계는 소프트웨어 결함이나 침해가 발생했을 때 영향을 줄여준다.

응용 프로그램에 관리자 권한이 필요하지 않은 경우 컨테이너를 루트(Root)로 실행하면 불필요한 위험이 증가한다. 컨테이너 내부의 루트는 네임스페이스(Namespace)와 다른 격리 메커니즘에 의해 제한되지만 과도한 권한은 설정 오류나 취약점의 영향을 확대할 수 있다. 따라서 가능한 경우 응용 프로그램 프로세스를 비루트 사용자(Non-root User)로 실행하고 파일 소유권과 권한도 이에 맞게 설계해야 한다.

루트리스 컨테이너(Rootless Container)는 이러한 개념을 확장하여 일반적인 호스트 루트 권한 없이 컨테이너 엔진과 컨테이너를 실행할 수 있도록 한다. 루트리스 실행은 사용자 네임스페이스(User Namespace)와 같은 메커니즘을 이용하여 컨테이너의 사용자 식별자를 권한이 제한된 호스트 사용자 식별자에 매핑한다. 컨테이너나 런타임 구성요소가 침해되더라도 공격자가 호스트 관리자 권한을 획득하기 전에 추가적인 권한 경계를 만나게 된다.

루트리스 실행에는 로보틱스 환경에서 고려해야 할 제약도 존재한다. 하드웨어 장치에 대한 직접 접근, 특권 네트워크 설정, 낮은 번호의 포트, GPU 자원 및 일부 커널 기능은 추가 설정이 필요하거나 루트 권한 컨테이너(Rootful Container)와 동일하게 동작하지 않을 수 있다. 따라서 루트리스 모드는 실제 로봇의 카메라, CAN, 직렬 통신, LiDAR 및 가속기 요구사항을 기준으로 검증해야 한다.

읽기 전용 루트 파일 시스템(Read-only Root Filesystem)은 또 하나의 강력한 보호 계층을 제공한다. 이 구성에서는 정상적인 실행 중 컨테이너의 기본 파일 시스템을 수정할 수 없다. 응용 프로그램은 바이너리, 라이브러리 및 설정을 읽을 수 있지만 이미지 파일 시스템 내부의 파일을 임의로 교체하거나 새로운 파일을 영구적으로 기록할 수 없다. 이를 통해 배포 이후 지속적인 변경이나 의도하지 않은 수정 가능성을 줄일 수 있다.

로봇 응용 프로그램은 여전히 임시 파일, 로그, 캐시, 지도, 데이터베이스 또는 생성된 산출물을 위한 쓰기 가능한 위치가 필요할 수 있다. 전체 파일 시스템을 쓰기 가능하게 만드는 대신 특정 디렉터리에 쓰기 가능한 볼륨(Volume), 바인드 마운트(Bind Mount) 또는 임시 파일 시스템(Temporary Filesystem)을 제공할 수 있다. 이를 통해 변경 불가능한 응용 프로그램 내용과 의도적으로 쓰기를 허용한 제한된 데이터 영역을 명확하게 구분할 수 있다.

불변 컨테이너 설계(Immutable Container Design)는 운영 일관성도 향상시킨다. 응용 프로그램이 실행 중 설치된 소프트웨어를 변경할 수 없다면 컨테이너를 재시작했을 때 알려진 이미지 상태로 복귀할 수 있다. 소프트웨어 변경은 실행 중인 로봇 내부에서 수동으로 수정하는 방식이 아니라 새로운 이미지를 빌드하고 통제된 방식으로 배포해야 한다. 이는 컨테이너 보안을 재현 가능한 DevOps 및 OTA 업데이트 방식과 일치시킨다.

리눅스 기능(Linux Capability)은 제한 없는 루트 권한을 부여하는 것보다 세분화된 권한 제어를 제공한다. 전통적인 루트 권한에는 네트워크 설정 변경, 프로세스 조작 또는 시스템 관리 작업 수행과 같은 여러 개별 권한이 포함된다. 컨테이너는 불필요한 기능을 제거하고 응용 프로그램에서 실제로 필요한 특정 기능만 추가하여 침해된 프로세스가 사용할 수 있는 권한을 줄일 수 있다.

특권 컨테이너 모드(Privileged Container Mode)는 신중하게 정당화된 요구사항이 없는 한 피해야 한다. 특권 컨테이너는 호스트 장치와 커널 기능에 광범위하게 접근할 수 있어 격리 경계를 크게 약화시킨다. 로보틱스 개발 단계에서는 하드웨어에 즉시 접근할 수 있어 편리해 보일 수 있지만 실제로 어떤 권한이 필요한지 감추고 안전하지 않은 운영 설정으로 이어질 수 있다.

보안 컴퓨팅 모드(Secure Computing Mode)인 Seccomp는 프로세스가 호출할 수 있는 리눅스 시스템 호출(System Call)을 제한한다. 컨테이너는 시스템 호출을 통해 호스트 커널과 상호작용하므로 이러한 인터페이스를 제한하면 응용 프로그램에 노출되는 커널 기능을 줄일 수 있다. Docker는 일반적인 컨테이너 소프트웨어에 필요한 호출은 허용하면서 특정 시스템 호출을 차단하는 Seccomp 프로파일(Profile)을 적용할 수 있다.

많은 응용 프로그램은 전체 리눅스 시스템 호출 인터페이스 가운데 일부만 사용하므로 이러한 메커니즘은 효과적이다. 인식 기능을 수행하는 ROS 2 노드는 일반적인 루트 프로세스가 사용할 수 있는 모든 커널 연산을 필요로 하지 않는다. 침해된 소프트웨어가 금지된 시스템 호출을 시도하면 Seccomp 정책이 해당 연산을 차단하여 더 깊은 호스트 자원에 도달하기 전에 공격 표면(Attack Surface)을 줄일 수 있다.

사용자 정의 Seccomp 프로파일(Custom Seccomp Profile)은 더 강력한 제한을 제공할 수 있지만 신중한 테스트가 필요하다. 로보틱스 소프트웨어는 미들웨어, 공유 메모리 전송(Shared-memory Transport), 하드웨어 드라이버, 디버깅 도구, 실시간 스케줄링 또는 GPU 런타임을 통해 일반적이지 않은 시스템 호출을 사용할 수 있다. 따라서 지나치게 제한적인 정책은 정상적인 동작까지 방해할 수 있으며 호환성 문제가 발생할 때 Seccomp를 비활성화하기보다 실제 응용 프로그램 요구사항을 관찰하여 프로파일을 조정해야 한다.

AppArmor는 다른 계층에서 강제적 접근 제어(Mandatory Access Control)를 제공한다. AppArmor 프로파일은 주로 시스템 호출 자체를 제한하기보다 프로세스가 접근할 수 있는 파일, 경로, 기능 및 기타 운영체제 자원을 제한할 수 있다. 따라서 일반적인 유닉스 파일 권한(Unix File Permission)이 더 넓은 접근을 허용하는 경우에도 컨테이너화된 응용 프로그램을 추가적으로 제한할 수 있다.

로봇 서비스에서 AppArmor 정책은 설정 디렉터리, 장치 자원, 응용 프로그램 파일 또는 컨테이너에 노출된 기타 호스트 경로에 대한 접근을 제한하는 데 사용할 수 있다. 이는 바인드 마운트를 사용할 때 특히 중요하다. 호스트 디렉터리를 컨테이너에 마운트하면 호스트 데이터와 직접적인 연결이 생성되므로 필요한 최소 경로만 노출하고 수정이 필요하지 않은 경우에는 가능하면 읽기 전용(Read-only)으로 설정해야 한다.

Seccomp와 AppArmor는 보안 경계의 서로 다른 측면을 다루므로 함께 사용할 수 있다. Seccomp는 시스템 호출을 통해 사용할 수 있는 커널 연산의 범위를 줄이고 AppArmor는 보안 정책에 따라 자원 접근을 제한한다. 네임스페이스, 기능(Capability), 비루트 실행, 파일 시스템 제한과 함께 사용하면 하나의 보호 계층에만 의존하지 않는 심층 방어(Defense in Depth)를 구성할 수 있다.

장치 노출(Device Exposure)에도 동일한 원칙을 적용해야 한다. 로봇 컨테이너는 카메라, 직렬 어댑터, CAN 인터페이스 또는 기타 주변장치를 위해 \`/dev\` 자원에 접근해야 하는 경우가 많다. 그러나 호스트의 전체 장치 영역을 컨테이너에 매핑해야 하는 경우는 드물다. 필요한 장치만 적절한 소유권과 권한으로 노출하여 하나의 서비스가 침해되더라도 관련 없는 액추에이터(Actuator)나 센서까지 자동으로 접근하지 못하도록 해야 한다.

네트워크 노출(Network Exposure) 역시 최소화해야 한다. 내부 통신만 필요한 서비스는 격리된 컨테이너 네트워크에 유지하고 필요한 API, 텔레메트리 게이트웨이(Telemetry Gateway) 또는 ROS 2 인터페이스만 외부에서 접근할 수 있도록 구성할 수 있다. 호스트 네트워킹은 로보틱스 통신을 단순화할 수 있지만 네트워크 분리를 감소시키므로 컨테이너가 호스트 네트워크 자원을 직접 공유할 때는 방화벽 규칙, 인터페이스 선택, DDS 설정, 인증 및 암호화가 더욱 중요하다.

비밀 정보(Secret)는 일반적인 설정과 분리하여 관리해야 한다. API 토큰, 레지스트리 인증 정보, 인증서, 클라우드 키 및 기타 민감한 값을 컨테이너 이미지에 포함하거나 소스 저장소(Source Repository)에 커밋해서는 안 된다. 이러한 정보는 통제된 메커니즘을 통해 런타임에 주입하고 실제로 필요한 서비스에서만 사용할 수 있도록 해야 한다. 단순히 인증 정보를 변경하기 위해 이미지를 다시 빌드해야 하는 구조는 피해야 한다.

컨테이너 이미지 자체도 로봇 소프트웨어 공급망(Software Supply Chain)의 일부를 구성한다. 운영 이미지는 통제된 기본 이미지(Base Image)에서 생성하고 고정되거나 적절하게 관리되는 의존성을 사용하며 불필요한 패키지를 제거하고 추적 가능한 빌드 기록을 유지해야 한다. 작은 이미지는 저장 공간뿐 아니라 공격 표면도 줄여주는데, 침해 이후 공격자가 사용할 수 있는 도구, 라이브러리, 셸(Shell) 및 유틸리티가 감소하기 때문이다.

개발 환경과 운영 환경의 보안 프로파일(Security Profile)은 서로 다를 수 있지만 이러한 차이는 명시적으로 관리해야 한다. 개발자는 일시적으로 디버깅 도구, 소스 마운트, 프로파일러 또는 추가적인 장치 접근이 필요할 수 있다. 운영 컨테이너에서는 실제 운용에 필요하지 않은 이러한 편의 기능을 제거해야 한다. 개발용 및 런타임용 대상을 분리하면 일시적인 디버깅 요구사항이 플릿의 영구적인 권한으로 남는 것을 방지할 수 있다.

보안 검증(Security Validation)은 로봇 이미지가 완성된 이후에만 수행하는 것이 아니라 CI/CD 과정에 통합해야 한다. 파이프라인은 설정을 검사하고 알려진 취약점에 대해 컨테이너 이미지를 스캔하며 예상된 사용자와 파일 시스템 권한을 검증하고 금지된 특권 설정이 존재하는지 확인할 수 있다. 또한 불필요한 기능과 쓰기 가능한 경로를 제거한 상태에서도 응용 프로그램이 정상적으로 동작하는지 보안 테스트를 통해 확인할 수 있다.

런타임 모니터링(Runtime Monitoring)은 예방적 보안 제어를 보완한다. 예상하지 못한 프로세스 실행, 비정상적인 네트워크 연결, 반복적인 권한 오류, 파일 시스템 변경 또는 비정상적인 장치 접근은 소프트웨어 결함이나 침해 가능성을 나타낼 수 있다. 따라서 로봇 플릿 모니터링은 특히 지속적인 사람의 감독 없이 원격으로 운용되는 시스템에서 응용 프로그램 상태 정보와 관련 컨테이너 및 호스트 보안 텔레메트리(Security Telemetry)를 함께 관리해야 한다.

단일 메커니즘만으로 컨테이너를 안전하게 만들 수는 없다. 루트리스 실행(Rootless Execution)은 호스트 권한을 줄이고, 읽기 전용 파일 시스템(Read-only Filesystem)은 변경을 제한하며, 리눅스 기능은 관리자 권한을 세분화하고, Seccomp는 시스템 호출 노출을 줄이며, AppArmor는 자원 접근을 제한한다. 이러한 기술은 전체 소프트웨어 스택에 일괄적으로 활성화하거나 비활성화하는 것보다 각 로봇 서비스의 요구사항에 맞게 함께 적용할 때 가장 큰 효과를 제공한다.

따라서 안전한 로봇 컨테이너 아키텍처(Secure Robot Container Architecture)는 각 서비스에 실제로 필요한 기능을 정의하는 것에서 시작한다. 이러한 요구사항을 기준으로 사용자, 기능, 장치, 쓰기 가능한 경로, 네트워크, 시스템 호출 및 접근 제어 정책을 선택할 수 있다. 통제된 이미지, CI/CD 검증, 런타임 모니터링 및 체계적인 업데이트와 결합하면 이러한 계층들은 컨테이너화된 로봇 소프트웨어를 위한 실용적인 심층 방어 모델(Defense-in-depth Model)을 구성한다.

##  

## 3.9. Container Image Vulnerability Scanning Trivy Grype [w/Code]

![](images/image9.png){width="7.268055555555556in" height="7.268055555555556in"}

Container image vulnerability scanning identifies known security weaknesses in the operating-system packages, libraries, language dependencies, and other components included in a container image. In robotics, these images may operate close to sensors, actuators, networks, GPUs, and fleet infrastructure. A vulnerable package can therefore become part of the attack surface of a physical system rather than remaining only an application-security concern.

Container images are assembled from multiple dependency layers. A robot application may begin with an Ubuntu or ROS 2 base image and then add system packages, Python modules, C++ libraries, CUDA components, middleware, and application binaries. Even when the robot source code is secure, inherited components can contain published vulnerabilities. Scanning provides visibility into this accumulated software supply-chain risk.

Vulnerability scanners generally inspect the packages and metadata contained in an image and compare them with vulnerability databases. Findings are commonly associated with identifiers such as CVEs and include information about affected packages, installed versions, severity, and available fixes. The result is not proof that a vulnerability can be exploited in the robot, but it provides evidence for prioritizing investigation and remediation.

Trivy is widely used as a security scanner for container images and related software artifacts. It can inspect operating-system packages and application dependencies while also supporting broader security checks in development pipelines. Its straightforward command-line workflow makes it suitable for local developer checks, automated CI jobs, registry-oriented processes, and security gates applied before robot software is released.

Grype provides another container and filesystem vulnerability-scanning approach. It analyzes software packages discovered in an image or software inventory and matches them against known vulnerability information. Grype is often used together with Syft, which generates a Software Bill of Materials, allowing package inventory and vulnerability analysis to become connected parts of a software supply-chain workflow.

A Software Bill of Materials, or SBOM, describes the software components contained in an artifact. For robot containers, an SBOM can record operating-system packages, libraries, and other dependencies that form a release. Maintaining this inventory improves traceability because teams can determine whether deployed images contain a newly disclosed vulnerable component without manually reconstructing every historical build environment.

Scanning can be performed directly against a locally built image before it is pushed to a registry. This provides rapid feedback during development and allows obvious problems to be corrected before distribution. Developers can identify whether a newly introduced dependency significantly changes the vulnerability profile of an image, making security feedback part of normal image development rather than a separate late-stage activity.

CI/CD integration makes vulnerability scanning repeatable. After a pipeline builds a robot container image, Trivy or Grype can analyze it before the release proceeds to later stages. The pipeline can store scan results as artifacts, associate them with the image digest, and apply defined policies. This creates a documented security checkpoint between software construction and deployment to robots.

A security gate can fail a pipeline when findings exceed an organization\'s defined policy. For example, teams may treat critical or high-severity vulnerabilities differently from lower-severity findings. However, severity alone should not determine every decision. Whether vulnerable code is reachable, whether the affected component is used, whether a fix exists, and how the container is exposed all influence the actual operational risk.

False confidence is a major risk in vulnerability scanning. A report containing zero detected vulnerabilities does not prove that an image is secure. Scanners depend on available package metadata, vulnerability databases, ecosystem support, and accurate component identification. Unknown vulnerabilities, application logic flaws, insecure configuration, leaked credentials, excessive privileges, and unsafe network exposure require other security controls.

The opposite problem also occurs when scanners report large numbers of vulnerabilities that have limited relevance to the running application. A package may exist in the image but never be executed, or vulnerable functionality may not be reachable in the deployed configuration. Findings should therefore be triaged rather than blindly counted, with priority given to realistic exposure, exploitability, asset importance, and remediation availability.

Base-image selection strongly influences scan results. Large general-purpose images contain more packages and therefore create a larger potential vulnerability surface. Minimal runtime images can reduce unnecessary packages, tools, and libraries. Multi-stage builds are especially valuable because compilers, package managers, development headers, and build utilities can remain in builder stages instead of being included in the final robot runtime image.

Updating the base image can remove vulnerabilities when patched package versions become available, but uncontrolled updates can also introduce compatibility problems. Robotics stacks may depend on specific ROS 2, CUDA, TensorRT, driver, or middleware combinations. Security remediation must therefore balance vulnerability reduction with functional validation, using controlled image rebuilds and testing rather than indiscriminately upgrading every dependency.

Image tags alone are insufficient for precise security traceability because mutable tags can eventually refer to different image contents. Image digests provide immutable identifiers for exact image versions. Scan reports, SBOMs, test records, and deployment metadata can be associated with a digest so that teams know exactly which binary artifact was evaluated and which artifact is currently running on a robot.

Registry scanning can complement CI scanning by continuously evaluating stored images or rescanning them when vulnerability databases change. An image that passed its original pipeline may later become vulnerable when a new CVE is disclosed. This is particularly important for robots with long operational lifetimes because software that was considered acceptable at deployment time may require remediation months or years later.

Periodic rescanning should therefore be part of fleet security maintenance. Teams need to know not only whether a vulnerability exists in a repository image but also whether that exact image is deployed on physical robots. Connecting image inventory, digests, SBOM information, scanner results, and fleet deployment records allows newly disclosed vulnerabilities to be mapped to affected robot populations.

Robot software introduces special remediation constraints because an update cannot be evaluated only as an IT security patch. Changing a library may influence sensor processing, inference performance, DDS communication, GPU compatibility, or control behavior. A patched container should pass regression, integration, hardware, and performance tests before fleet rollout, particularly when it participates in safety-relevant robot functions.

Trivy and Grype should therefore be viewed as detection components within a larger DevSecOps process rather than complete security solutions. Their results can initiate dependency updates, image rebuilds, exception reviews, or deployment blocks. Other controls such as secret scanning, configuration analysis, least-privilege enforcement, image signing, provenance verification, and runtime monitoring address risks that vulnerability databases cannot cover.

Scanner databases must also remain current. A scan performed with outdated vulnerability intelligence may miss recently disclosed issues or report obsolete status information. Automated environments should update vulnerability data according to the scanner\'s supported workflow and preserve enough metadata to understand when and how an image was evaluated. Reproducible security decisions require both artifact identity and scan context.

Exceptions sometimes become necessary when no patched dependency is available or when upgrading would break validated robot functionality. Such findings should not simply be ignored. A documented exception can identify the affected component, operational exposure, compensating controls, responsible owner, review date, and expected remediation path, turning an unresolved vulnerability into a managed engineering risk.

Security scanning also benefits from comparing releases rather than examining each report in isolation. If a new robot image introduces several high-impact findings that were absent from the previous version, the change can be investigated before deployment. Conversely, successful remediation should be visible as vulnerabilities disappear after dependencies or base images are updated and the image is rebuilt.

For heterogeneous robot fleets, scanning should cover every architecture-specific image variant. AMD64 and ARM64 images may use different base packages, platform libraries, CUDA components, or vendor dependencies. A clean AMD64 scan cannot automatically represent an ARM64 Jetson image. Each published artifact should therefore have its own vulnerability results and traceability information.

A mature workflow connects source code, container builds, SBOM generation, vulnerability scanning, testing, registry storage, and fleet deployment into one traceable chain. Trivy or Grype can provide the vulnerability-analysis stage, while image digests connect scan evidence to immutable artifacts. This allows teams to answer not only what vulnerabilities exist, but also which release and which deployed robots are affected.

Container image scanning ultimately converts hidden dependency risk into actionable engineering information. Used continuously rather than as a one-time release check, Trivy and Grype help teams detect vulnerable components before deployment and respond when new vulnerabilities emerge later. Combined with controlled remediation, SBOMs, CI/CD gates, and fleet traceability, scanning becomes a core element of secure robot software lifecycle management.

컨테이너 이미지 취약점 스캐닝(Container Image Vulnerability Scanning)은 컨테이너 이미지에 포함된 운영체제 패키지, 라이브러리, 언어별 의존성 및 기타 구성요소에서 알려진 보안 취약점을 식별한다. 로보틱스에서 이러한 이미지는 센서, 액추에이터, 네트워크, GPU 및 플릿 인프라(Fleet Infrastructure)와 밀접하게 동작할 수 있다. 따라서 취약한 패키지는 단순한 응용 프로그램 보안 문제를 넘어 물리적 시스템의 공격 표면(Attack Surface)이 될 수 있다.

컨테이너 이미지는 여러 의존성 계층(Dependency Layer)으로 구성된다. 로봇 응용 프로그램은 Ubuntu 또는 ROS 2 기본 이미지(Base Image)에서 시작하여 시스템 패키지, Python 모듈, C++ 라이브러리, CUDA 구성요소, 미들웨어 및 응용 프로그램 바이너리를 추가할 수 있다. 로봇의 소스 코드 자체가 안전하더라도 상속된 구성요소에는 공개된 취약점이 포함될 수 있다. 스캐닝은 이렇게 누적된 소프트웨어 공급망 위험(Software Supply-chain Risk)을 파악할 수 있도록 한다.

취약점 스캐너(Vulnerability Scanner)는 일반적으로 이미지에 포함된 패키지와 메타데이터를 검사한 후 취약점 데이터베이스(Vulnerability Database)와 비교한다. 발견된 항목은 일반적으로 CVE와 같은 식별자와 연결되며 영향을 받는 패키지, 설치 버전, 심각도(Severity), 사용 가능한 수정 사항 등의 정보를 포함한다. 이러한 결과가 해당 취약점이 실제 로봇에서 악용될 수 있음을 증명하는 것은 아니지만 조사와 개선의 우선순위를 결정할 근거를 제공한다.

Trivy는 컨테이너 이미지 및 관련 소프트웨어 산출물(Software Artifact)을 위한 보안 스캐너로 널리 사용된다. 운영체제 패키지와 응용 프로그램 의존성을 검사할 수 있으며 개발 파이프라인에서 보다 광범위한 보안 검사도 지원한다. 간결한 명령줄 워크플로(Command-line Workflow)를 제공하므로 로컬 개발자 검사, 자동화된 CI 작업, 레지스트리 중심 프로세스 및 로봇 소프트웨어 릴리스 전 보안 게이트(Security Gate)에 적용하기 적합하다.

Grype는 컨테이너와 파일 시스템을 대상으로 하는 또 다른 취약점 스캐닝 접근 방식을 제공한다. 이미지 또는 소프트웨어 인벤토리(Software Inventory)에서 발견된 패키지를 분석하고 알려진 취약점 정보와 비교한다. Grype는 소프트웨어 자재 명세서(Software Bill of Materials)를 생성하는 Syft와 함께 사용되는 경우가 많으며 이를 통해 패키지 인벤토리와 취약점 분석을 하나의 소프트웨어 공급망 워크플로로 연결할 수 있다.

소프트웨어 자재 명세서(Software Bill of Materials, SBOM)는 하나의 산출물에 포함된 소프트웨어 구성요소를 설명한다. 로봇 컨테이너의 SBOM에는 릴리스를 구성하는 운영체제 패키지, 라이브러리 및 기타 의존성을 기록할 수 있다. 이러한 인벤토리를 유지하면 새롭게 공개된 취약한 구성요소가 배포된 이미지에 포함되어 있는지 확인할 때 과거의 모든 빌드 환경을 수동으로 다시 구성하지 않아도 되므로 추적성(Traceability)이 향상된다.

스캐닝은 로컬에서 빌드된 이미지를 레지스트리에 푸시하기 전에 직접 수행할 수 있다. 이를 통해 개발 과정에서 빠른 피드백을 얻고 명확한 문제를 배포 전에 수정할 수 있다. 개발자는 새롭게 추가된 의존성이 이미지의 취약점 프로파일(Vulnerability Profile)을 크게 변화시키는지 확인할 수 있으며 보안 피드백을 별도의 후반 작업이 아니라 일반적인 이미지 개발 과정에 포함할 수 있다.

CI/CD 통합은 취약점 스캐닝을 반복 가능한 과정으로 만든다. 파이프라인에서 로봇 컨테이너 이미지를 빌드한 후 릴리스가 다음 단계로 진행되기 전에 Trivy 또는 Grype로 분석할 수 있다. 파이프라인은 스캔 결과를 산출물로 저장하고 이미지 다이제스트(Image Digest)와 연결하며 정의된 정책을 적용할 수 있다. 이를 통해 소프트웨어 생성과 로봇 배포 사이에 문서화된 보안 검사 지점을 구성할 수 있다.

보안 게이트(Security Gate)는 발견된 취약점이 조직에서 정의한 정책 기준을 초과할 경우 파이프라인을 실패하도록 구성할 수 있다. 예를 들어 치명적(Critical) 또는 높은 심각도(High-severity)의 취약점을 낮은 심각도의 항목과 다르게 처리할 수 있다. 그러나 심각도만으로 모든 결정을 내려서는 안 된다. 취약한 코드에 실제로 접근 가능한지, 해당 구성요소가 사용되는지, 수정 버전이 존재하는지, 컨테이너가 어떻게 노출되어 있는지 등이 실제 운영 위험에 영향을 준다.

취약점 스캐닝에서 중요한 위험 중 하나는 잘못된 안전 확신(False Confidence)이다. 탐지된 취약점이 없다는 보고서가 이미지의 안전성을 증명하는 것은 아니다. 스캐너는 사용 가능한 패키지 메타데이터, 취약점 데이터베이스, 생태계 지원 및 정확한 구성요소 식별에 의존한다. 알려지지 않은 취약점, 응용 프로그램 논리 오류, 안전하지 않은 설정, 유출된 인증 정보, 과도한 권한 및 위험한 네트워크 노출은 다른 보안 제어가 필요하다.

반대로 스캐너가 실행 중인 응용 프로그램과 관련성이 낮은 다수의 취약점을 보고하는 문제도 발생할 수 있다. 특정 패키지가 이미지에 존재하지만 실제로 실행되지 않거나 취약한 기능이 배포 설정에서 접근 불가능할 수 있다. 따라서 발견된 항목의 개수만 단순히 계산하기보다 실제 노출 가능성, 악용 가능성(Exploitability), 자산 중요도 및 수정 가능성을 기준으로 분류하고 우선순위를 결정해야 한다.

기본 이미지 선택(Base-image Selection)은 스캔 결과에 큰 영향을 준다. 대규모 범용 이미지는 더 많은 패키지를 포함하므로 잠재적인 취약점 표면도 커진다. 최소화된 런타임 이미지(Minimal Runtime Image)는 불필요한 패키지, 도구 및 라이브러리를 줄일 수 있다. 다단계 빌드(Multi-stage Build)를 사용하면 컴파일러, 패키지 관리자, 개발 헤더 및 빌드 유틸리티를 최종 로봇 런타임 이미지에 포함하지 않고 빌더 단계에만 유지할 수 있다.

패치된 패키지 버전을 사용할 수 있다면 기본 이미지를 업데이트하여 취약점을 제거할 수 있지만 통제되지 않은 업데이트는 호환성 문제를 발생시킬 수 있다. 로보틱스 스택은 특정 ROS 2, CUDA, TensorRT, 드라이버 또는 미들웨어 조합에 의존할 수 있다. 따라서 보안 개선은 모든 의존성을 무조건 업그레이드하는 방식이 아니라 통제된 이미지 재빌드와 테스트를 통해 취약점 감소와 기능 검증 사이의 균형을 유지해야 한다.

이미지 태그(Image Tag)만으로는 정확한 보안 추적성을 확보하기 어렵다. 변경 가능한 태그(Mutable Tag)는 시간이 지나면서 서로 다른 이미지 내용을 가리킬 수 있기 때문이다. 이미지 다이제스트는 정확한 이미지 버전을 나타내는 불변 식별자(Immutable Identifier)를 제공한다. 스캔 보고서, SBOM, 테스트 기록 및 배포 메타데이터를 다이제스트와 연결하면 어떤 바이너리 산출물이 평가되었고 현재 어떤 산출물이 로봇에서 실행 중인지 정확하게 확인할 수 있다.

레지스트리 스캐닝(Registry Scanning)은 저장된 이미지를 지속적으로 평가하거나 취약점 데이터베이스가 변경될 때 다시 검사함으로써 CI 스캐닝을 보완할 수 있다. 최초 파이프라인을 통과한 이미지도 새로운 CVE가 공개되면 이후 취약한 상태가 될 수 있다. 이는 운용 수명이 긴 로봇에서 특히 중요하며 배포 당시 허용 가능했던 소프트웨어가 몇 개월 또는 몇 년 후에는 개선이 필요할 수 있다.

따라서 정기적인 재스캐닝(Periodic Rescanning)은 플릿 보안 유지관리의 일부가 되어야 한다. 팀은 취약점이 저장소 이미지에 존재하는지뿐만 아니라 정확히 동일한 이미지가 실제 물리적 로봇에 배포되어 있는지도 파악해야 한다. 이미지 인벤토리, 다이제스트, SBOM 정보, 스캐너 결과 및 플릿 배포 기록을 연결하면 새롭게 공개된 취약점과 영향을 받는 로봇 집단을 연결할 수 있다.

로봇 소프트웨어에는 특별한 개선 제약(Remediation Constraint)이 존재한다. 업데이트를 단순한 IT 보안 패치로만 평가할 수 없기 때문이다. 라이브러리 변경은 센서 처리, 추론 성능, DDS 통신, GPU 호환성 또는 제어 동작에 영향을 줄 수 있다. 특히 안전 관련 로봇 기능에 참여하는 컨테이너의 경우 패치된 이미지는 플릿에 배포하기 전에 회귀 테스트(Regression Test), 통합 테스트, 하드웨어 테스트 및 성능 테스트를 통과해야 한다.

따라서 Trivy와 Grype는 완전한 보안 솔루션이라기보다 더 큰 DevSecOps 프로세스 안에서 탐지 구성요소(Detection Component)로 이해해야 한다. 스캔 결과는 의존성 업데이트, 이미지 재빌드, 예외 검토 또는 배포 차단을 시작하는 근거가 될 수 있다. 비밀 정보 스캐닝(Secret Scanning), 설정 분석, 최소 권한 적용, 이미지 서명(Image Signing), 출처 검증(Provenance Verification) 및 런타임 모니터링과 같은 다른 보안 제어는 취약점 데이터베이스만으로 다룰 수 없는 위험을 보완한다.

스캐너 데이터베이스(Scanner Database)도 최신 상태를 유지해야 한다. 오래된 취약점 정보를 사용하여 수행한 스캔은 최근 공개된 문제를 놓치거나 오래된 상태 정보를 보고할 수 있다. 자동화된 환경에서는 스캐너가 지원하는 방식에 따라 취약점 데이터를 갱신하고 이미지가 언제 어떤 조건으로 평가되었는지 이해할 수 있도록 충분한 메타데이터를 보존해야 한다. 재현 가능한 보안 결정을 위해서는 산출물 식별 정보와 스캔 환경 정보가 모두 필요하다.

패치된 의존성을 사용할 수 없거나 업그레이드가 검증된 로봇 기능을 손상시키는 경우에는 예외(Exception)가 필요할 수 있다. 이러한 발견 사항을 단순히 무시해서는 안 된다. 문서화된 예외에는 영향을 받는 구성요소, 운영상 노출 정도, 보완 통제(Compensating Control), 담당자, 검토 날짜 및 예상 개선 경로를 기록할 수 있으며 이를 통해 해결되지 않은 취약점을 관리 가능한 엔지니어링 위험(Engineering Risk)으로 전환할 수 있다.

보안 스캐닝은 각각의 보고서를 독립적으로 검토하는 것보다 릴리스 간 결과를 비교할 때도 유용하다. 새로운 로봇 이미지가 이전 버전에는 없었던 여러 개의 영향도 높은 취약점을 추가한다면 배포 전에 해당 변경 사항을 조사할 수 있다. 반대로 의존성이나 기본 이미지를 업데이트하고 이미지를 다시 빌드한 이후 취약점이 제거되었다면 성공적인 개선 결과를 명확하게 확인할 수 있다.

이기종 로봇 플릿(Heterogeneous Robot Fleet)에서는 모든 아키텍처별 이미지 변형(Architecture-specific Image Variant)을 대상으로 스캐닝해야 한다. AMD64와 ARM64 이미지는 서로 다른 기본 패키지, 플랫폼 라이브러리, CUDA 구성요소 또는 제조사 의존성을 사용할 수 있다. 따라서 AMD64 이미지의 안전한 스캔 결과가 ARM64 Jetson 이미지를 자동으로 대표할 수 없으며 배포되는 각각의 산출물에 개별 취약점 결과와 추적성 정보를 유지해야 한다.

성숙한 워크플로는 소스 코드, 컨테이너 빌드, SBOM 생성, 취약점 스캐닝, 테스트, 레지스트리 저장 및 플릿 배포를 하나의 추적 가능한 체인(Traceable Chain)으로 연결한다. Trivy 또는 Grype는 이 과정에서 취약점 분석 단계를 제공하고 이미지 다이제스트는 스캔 증거와 불변 산출물을 연결한다. 이를 통해 어떤 취약점이 존재하는지뿐만 아니라 어떤 릴리스와 실제 배포된 어떤 로봇이 영향을 받는지도 파악할 수 있다.

궁극적으로 컨테이너 이미지 스캐닝(Container Image Scanning)은 숨겨진 의존성 위험을 실행 가능한 엔지니어링 정보(Actionable Engineering Information)로 변환한다. Trivy와 Grype를 일회성 릴리스 검사에 그치지 않고 지속적으로 사용하면 배포 전에 취약한 구성요소를 탐지하고 이후 새롭게 공개되는 취약점에도 대응할 수 있다. 통제된 개선, SBOM, CI/CD 게이트 및 플릿 추적성과 결합하면 취약점 스캐닝은 안전한 로봇 소프트웨어 수명주기 관리(Secure Robot Software Lifecycle Management)의 핵심 요소가 된다.

##  

## 3.10. Container Based OTA Update Design for Robot Fleets

![](images/image10.png){width="7.268055555555556in" height="7.268055555555556in"}

Container-based OTA updates provide a scalable mechanism for distributing robot software to deployed fleets without manually reinstalling complete operating systems or application stacks. Instead of modifying individual packages directly on each robot, software services are packaged as versioned container images. Robots retrieve approved images from a registry and replace running application containers according to a controlled deployment policy.

This approach separates the relatively stable host operating system from frequently updated robot applications. The host can provide the Linux kernel, container runtime, hardware drivers, networking, and essential device services, while perception, navigation, AI inference, monitoring, and fleet communication run in containers. Application updates can therefore occur independently without rebuilding the entire robot system image.

An OTA architecture normally begins with a CI/CD pipeline that builds and tests container images from controlled source code. Successful images are assigned version information, scanned for vulnerabilities, and pushed to a trusted container registry. Deployment metadata then identifies which image digest should run on a particular robot model, software channel, fleet group, or operational environment.

Immutable image digests are preferable to relying only on mutable tags. A tag such as \`latest\` can point to different content over time, while a digest uniquely identifies the exact image artifact. Recording the digest allows the fleet management system to determine precisely which software is installed on each robot and provides a reliable reference for deployment verification and rollback.

The robot-side update agent acts as the execution point for OTA operations. It can periodically contact the fleet management service, receive a desired software state, authenticate the update request, download required images, verify their integrity, and coordinate container replacement. The agent should itself be small and stable because failure of the update mechanism can make remote recovery significantly more difficult.

Downloading a new image should not immediately imply activating it. A safer design separates acquisition from deployment so that images can be pulled and verified while the robot remains operational. Before activation, the system can confirm sufficient storage, image integrity, configuration compatibility, hardware requirements, battery condition, network status, and whether the robot is currently in a safe operational state.

Container registries become critical infrastructure in this architecture. They store approved image versions and provide a distribution point for robots operating across multiple sites. Registry authentication and encrypted communication should prevent unauthorized access, while image signing and provenance verification can help ensure that robots execute artifacts produced by trusted build pipelines rather than modified or unapproved images.

OTA security must protect both software authenticity and deployment authorization. An attacker who can substitute an image or issue unauthorized deployment commands may gain control over robot software. Secure update designs therefore combine authenticated fleet communication, protected credentials, trusted registries, image verification, least-privilege update agents, audit records, and controlled release permissions.

Robot updates should normally occur only when the platform is in an appropriate state. Replacing navigation or perception software while a mobile robot is actively moving can create unacceptable operational behavior. The update controller can require conditions such as stopped motion, docking, adequate battery charge, no active mission, or explicit maintenance mode before switching safety-relevant application containers.

A useful deployment strategy prepares the new container version before stopping the current one. Required images are downloaded and checked in advance, reducing the interruption during the actual transition. The system can then stop the old service, start the new version, perform startup validation, and confirm communication with dependent services before declaring the update successful.

Health checks are essential because successful container startup does not guarantee correct robot operation. A process may be running while failing to access a camera, GPU, LiDAR, map database, ROS 2 network, or inference model. Post-update validation should therefore include application-level readiness checks and, where appropriate, hardware communication and functional checks before the robot resumes autonomous operation.

Rollback provides the primary recovery mechanism when a new release fails validation. The robot should retain information about the previously validated image and configuration so that it can restore the earlier software state without downloading everything again. Rollback criteria may include failed health checks, repeated container crashes, missing dependencies, communication failure, or unsuccessful initialization within a defined period.

Persistent data requires special attention during rollback. Containers may be replaceable, but maps, calibration files, mission databases, learned parameters, logs, and configuration can exist in persistent volumes. If a new application version changes a data format incompatibly, returning to an older container may not restore functionality. Data migration and backward compatibility must therefore be considered as part of OTA design.

Configuration should be versioned together with software compatibility requirements even when it is stored separately from the container image. A particular navigation image may require specific sensor parameters, map formats, middleware settings, or model versions. Fleet deployment records should identify the complete operational configuration rather than assuming that the image version alone describes the robot\'s software state.

Fleet-wide updates should avoid deploying a new version to every robot simultaneously. A staged rollout begins with development or test robots and then expands to a small production group before reaching larger portions of the fleet. Observed health, mission completion, resource consumption, and error rates can be evaluated at each stage, reducing the impact of defects that escaped pre-deployment testing.

Canary deployment applies this principle by exposing a limited number of robots to a new release first. The selected robots should provide meaningful evidence for the target hardware and operational environment. If monitoring indicates abnormal behavior, rollout can stop before the release reaches the remaining fleet. Successful validation allows the deployment controller to progressively expand the update population.

Robot fleets are often heterogeneous, making target selection essential. Different robots may use AMD64 or ARM64 processors, different GPU platforms, sensor configurations, or hardware revisions. The OTA system must select compatible image variants and configuration sets rather than assuming that every robot can execute the same binary artifact. Hardware identity should therefore participate in deployment policy.

Network conditions also influence OTA design. Robots connected through Wi-Fi, LTE, or 5G may experience limited bandwidth, unstable connectivity, or data-transfer costs. Images should be kept reasonably small, and downloads should support retry behavior without corrupting the update state. Scheduling large transfers during suitable operational windows can prevent software distribution from interfering with telemetry or mission communication.

Disk capacity must be managed because safe updates may temporarily require both old and new images. Repeated deployments can also leave unused layers consuming storage. The update agent should monitor available space, preserve images required for rollback, and safely remove obsolete artifacts according to retention policy. Storage cleanup should never delete the only known-good recovery version before a new release is validated.

Service dependencies complicate updates when several containers must change together. A perception service may publish a message format consumed by navigation, while an API gateway may depend on a particular backend version. Deployment manifests should define compatible service sets, startup ordering, health dependencies, and interface versions so that an update does not leave the robot operating with an incompatible mixture of containers.

Some updates require coordinated replacement of multiple services, while others can be performed independently. Separating robot software into well-defined services reduces the update scope and allows unchanged components to remain running. However, excessive fragmentation can increase orchestration complexity. Container boundaries should therefore reflect meaningful lifecycle, dependency, security, and recovery boundaries.

Observability is necessary throughout the OTA lifecycle. Fleet systems should record update requests, image versions, digests, download progress, activation status, health-check results, rollback events, and final deployment state. Operators need to distinguish robots that successfully updated from those that are offline, downloading, waiting for safe conditions, failed, or operating on a previous release.

CI/CD and OTA should form one traceable software delivery chain. Source commits lead to tested images, images receive immutable identities and security evidence, approved artifacts enter the registry, deployment policies assign them to robots, and telemetry confirms the resulting runtime state. This traceability enables engineers to connect field behavior to the exact software artifact that produced it.

Container OTA does not eliminate the need for host operating-system or firmware updates. Kernel security patches, bootloader changes, GPU drivers, device firmware, and low-level safety controllers may require separate mechanisms. A robust robot platform therefore distinguishes application-container updates from host and firmware lifecycle management while coordinating compatibility among all three layers.

Failure handling should assume that power loss, network interruption, container crashes, and partial updates will eventually occur. Update operations should be restartable and designed around explicit states rather than fragile sequences of commands. After reboot, the robot should determine whether it was downloading, validating, activating, or rolling back and continue toward a known safe software state.

A mature container-based OTA system therefore combines immutable artifacts, secure distribution, staged rollout, health validation, rollback, configuration management, fleet targeting, and continuous observability. The objective is not merely to transfer new software remotely, but to change the software state of physical robots predictably while preserving recoverability and operational safety.

For robot fleets, container-based OTA becomes the operational extension of DevOps. Containers provide reproducible deployment units, CI/CD produces verified releases, registries distribute trusted artifacts, and fleet management controls when and where they are activated. Together, these mechanisms enable frequent software evolution while maintaining traceability, security, controlled risk, and reliable recovery across deployed robots.

컨테이너 기반 OTA 업데이트(Container-based OTA Update)는 전체 운영체제나 응용 프로그램 스택을 각 로봇에 수동으로 다시 설치하지 않고도 배포된 로봇 플릿(Robot Fleet)에 소프트웨어를 확장 가능하게 배포하는 방법을 제공한다. 각 로봇에서 개별 패키지를 직접 수정하는 대신 소프트웨어 서비스를 버전이 지정된 컨테이너 이미지(Container Image)로 패키징한다. 로봇은 레지스트리(Registry)에서 승인된 이미지를 가져와 통제된 배포 정책(Deployment Policy)에 따라 실행 중인 응용 프로그램 컨테이너를 교체한다.

이러한 접근 방식은 상대적으로 안정적인 호스트 운영체제(Host Operating System)와 자주 업데이트되는 로봇 응용 프로그램을 분리한다. 호스트는 Linux 커널, 컨테이너 런타임(Container Runtime), 하드웨어 드라이버, 네트워킹 및 필수 장치 서비스를 제공하고 인식(Perception), 내비게이션(Navigation), AI 추론, 모니터링 및 플릿 통신은 컨테이너에서 실행할 수 있다. 따라서 전체 로봇 시스템 이미지를 다시 구축하지 않고도 응용 프로그램을 독립적으로 업데이트할 수 있다.

OTA 아키텍처(OTA Architecture)는 일반적으로 통제된 소스 코드에서 컨테이너 이미지를 빌드하고 테스트하는 CI/CD 파이프라인에서 시작한다. 성공적으로 검증된 이미지에는 버전 정보가 부여되고 취약점 스캐닝(Vulnerability Scanning)을 거친 후 신뢰할 수 있는 컨테이너 레지스트리에 푸시된다. 이후 배포 메타데이터(Deployment Metadata)는 특정 로봇 모델, 소프트웨어 채널, 플릿 그룹 또는 운영 환경에서 어떤 이미지 다이제스트(Image Digest)를 실행해야 하는지 정의한다.

변경 가능한 태그(Mutable Tag)에만 의존하는 것보다 불변 이미지 다이제스트(Immutable Image Digest)를 사용하는 것이 바람직하다. \`latest\`와 같은 태그는 시간이 지나면서 서로 다른 내용을 가리킬 수 있지만 다이제스트는 정확한 이미지 산출물을 고유하게 식별한다. 다이제스트를 기록하면 플릿 관리 시스템(Fleet Management System)이 각 로봇에 설치된 소프트웨어를 정확하게 파악할 수 있으며 배포 검증과 롤백(Rollback)을 위한 신뢰할 수 있는 기준을 제공한다.

로봇 측 업데이트 에이전트(Update Agent)는 OTA 작업을 실제로 실행하는 지점으로 동작한다. 주기적으로 플릿 관리 서비스에 접속하여 원하는 소프트웨어 상태(Desired Software State)를 수신하고 업데이트 요청을 인증하며 필요한 이미지를 다운로드하고 무결성(Integrity)을 검증한 후 컨테이너 교체를 조정할 수 있다. 업데이트 메커니즘 자체의 장애는 원격 복구를 매우 어렵게 만들 수 있으므로 업데이트 에이전트는 작고 안정적으로 유지하는 것이 바람직하다.

새로운 이미지를 다운로드했다고 해서 즉시 활성화해야 하는 것은 아니다. 보다 안전한 설계에서는 이미지 획득(Acquisition)과 배포(Deployment)를 분리하여 로봇이 계속 운용되는 동안 이미지를 미리 가져오고 검증할 수 있다. 활성화 전에 충분한 저장 공간, 이미지 무결성, 설정 호환성, 하드웨어 요구사항, 배터리 상태, 네트워크 상태 및 로봇이 현재 안전한 운용 상태인지 확인할 수 있다.

컨테이너 레지스트리는 이러한 아키텍처에서 핵심 인프라가 된다. 레지스트리는 승인된 이미지 버전을 저장하고 여러 현장에서 운용되는 로봇에 소프트웨어를 배포하는 지점으로 기능한다. 레지스트리 인증(Registry Authentication)과 암호화 통신은 비인가 접근을 방지해야 하며 이미지 서명(Image Signing)과 출처 검증(Provenance Verification)은 로봇이 변조되거나 승인되지 않은 이미지가 아니라 신뢰할 수 있는 빌드 파이프라인에서 생성된 산출물을 실행하도록 지원할 수 있다.

OTA 보안(OTA Security)은 소프트웨어의 진위성(Authenticity)과 배포 권한(Deployment Authorization)을 모두 보호해야 한다. 공격자가 이미지를 교체하거나 승인되지 않은 배포 명령을 전달할 수 있다면 로봇 소프트웨어에 대한 제어권을 획득할 수 있다. 따라서 안전한 업데이트 설계는 인증된 플릿 통신, 보호된 자격 증명, 신뢰할 수 있는 레지스트리, 이미지 검증, 최소 권한 업데이트 에이전트, 감사 기록(Audit Record) 및 통제된 릴리스 권한을 결합해야 한다.

로봇 업데이트는 일반적으로 플랫폼이 적절한 상태에 있을 때만 수행해야 한다. 이동 로봇이 실제로 움직이는 동안 내비게이션이나 인식 소프트웨어를 교체하면 허용하기 어려운 운용 동작이 발생할 수 있다. 따라서 업데이트 제어기(Update Controller)는 안전 관련 응용 프로그램 컨테이너을 전환하기 전에 로봇 정지, 도킹(Docking), 충분한 배터리 충전, 활성 임무 없음 또는 명시적인 유지보수 모드(Maintenance Mode)와 같은 조건을 요구할 수 있다.

유용한 배포 전략은 현재 버전을 중지하기 전에 새로운 컨테이너 버전을 준비하는 것이다. 필요한 이미지를 사전에 다운로드하고 검사하여 실제 전환 과정에서 발생하는 중단 시간을 줄일 수 있다. 이후 시스템은 기존 서비스를 중지하고 새 버전을 시작한 다음 시작 검증(Startup Validation)을 수행하고 의존 서비스와의 통신을 확인한 후 업데이트가 성공했다고 판단할 수 있다.

컨테이너가 성공적으로 시작되었다고 해서 로봇이 정상적으로 동작한다는 보장은 없기 때문에 상태 확인(Health Check)이 필수적이다. 프로세스는 실행 중이지만 카메라, GPU, LiDAR, 지도 데이터베이스, ROS 2 네트워크 또는 추론 모델에 접근하지 못할 수 있다. 따라서 업데이트 이후 검증(Post-update Validation)은 응용 프로그램 수준의 준비 상태 확인과 필요한 경우 하드웨어 통신 및 기능 검사를 포함한 후 로봇의 자율 운용을 재개해야 한다.

롤백(Rollback)은 새로운 릴리스가 검증에 실패했을 때 사용되는 주요 복구 메커니즘을 제공한다. 로봇은 이전에 검증된 이미지와 설정 정보를 유지하여 모든 데이터를 다시 다운로드하지 않고도 이전 소프트웨어 상태로 복원할 수 있어야 한다. 롤백 조건에는 상태 확인 실패, 반복적인 컨테이너 충돌, 의존성 누락, 통신 장애 또는 정의된 시간 안에 초기화하지 못하는 경우 등이 포함될 수 있다.

영구 데이터(Persistent Data)는 롤백 과정에서 특별한 주의가 필요하다. 컨테이너는 교체할 수 있지만 지도, 보정 파일(Calibration File), 임무 데이터베이스, 학습된 매개변수, 로그 및 설정은 영구 볼륨(Persistent Volume)에 존재할 수 있다. 새로운 응용 프로그램 버전이 데이터 형식을 호환되지 않게 변경하면 이전 컨테이너로 복귀하더라도 기능이 복원되지 않을 수 있다. 따라서 데이터 마이그레이션(Data Migration)과 하위 호환성(Backward Compatibility)을 OTA 설계의 일부로 고려해야 한다.

설정(Configuration)은 컨테이너 이미지와 별도로 저장되더라도 소프트웨어 호환성 요구사항과 함께 버전 관리해야 한다. 특정 내비게이션 이미지는 특정 센서 매개변수, 지도 형식, 미들웨어 설정 또는 모델 버전을 요구할 수 있다. 플릿 배포 기록은 이미지 버전만으로 로봇의 소프트웨어 상태가 완전히 표현된다고 가정하지 말고 전체 운영 설정(Operational Configuration)을 식별할 수 있어야 한다.

플릿 전체 업데이트는 새로운 버전을 모든 로봇에 동시에 배포하는 방식을 피해야 한다. 단계적 롤아웃(Staged Rollout)은 개발 또는 시험 로봇에서 시작한 후 소규모 운영 그룹으로 확장하고 최종적으로 더 큰 규모의 플릿에 적용한다. 각 단계에서 상태, 임무 완료, 자원 사용량 및 오류율을 평가하면 사전 배포 테스트에서 발견하지 못한 결함의 영향을 줄일 수 있다.

카나리 배포(Canary Deployment)는 제한된 수의 로봇에 새로운 릴리스를 먼저 적용하여 이러한 원칙을 구현한다. 선택된 로봇은 대상 하드웨어와 운영 환경을 평가할 수 있는 의미 있는 데이터를 제공해야 한다. 모니터링 결과 비정상적인 동작이 확인되면 나머지 플릿에 릴리스가 적용되기 전에 롤아웃을 중단할 수 있으며 검증이 성공하면 배포 제어기가 업데이트 대상 범위를 점진적으로 확대할 수 있다.

로봇 플릿은 이기종(Heterogeneous)으로 구성되는 경우가 많으므로 대상 선택(Target Selection)이 중요하다. 서로 다른 로봇은 AMD64 또는 ARM64 프로세서, 서로 다른 GPU 플랫폼, 센서 구성 또는 하드웨어 개정판(Hardware Revision)을 사용할 수 있다. OTA 시스템은 모든 로봇이 동일한 바이너리 산출물을 실행할 수 있다고 가정하지 말고 호환 가능한 이미지 변형(Image Variant)과 설정 집합을 선택해야 한다. 따라서 하드웨어 식별 정보도 배포 정책에 포함되어야 한다.

네트워크 상태(Network Condition) 역시 OTA 설계에 영향을 준다. Wi-Fi, LTE 또는 5G를 통해 연결되는 로봇은 제한된 대역폭, 불안정한 연결 또는 데이터 전송 비용 문제를 경험할 수 있다. 이미지 크기를 합리적인 수준으로 유지하고 다운로드 과정은 업데이트 상태를 손상시키지 않으면서 재시도할 수 있어야 한다. 대규모 데이터 전송을 적절한 운용 시간에 예약하면 소프트웨어 배포가 텔레메트리나 임무 통신을 방해하는 것을 줄일 수 있다.

안전한 업데이트에서는 기존 이미지와 새로운 이미지를 일시적으로 모두 보관해야 할 수 있으므로 디스크 용량(Disk Capacity)을 관리해야 한다. 반복적인 배포 과정에서 사용하지 않는 이미지 계층이 저장 공간을 차지할 수도 있다. 업데이트 에이전트는 사용 가능한 공간을 모니터링하고 롤백에 필요한 이미지를 보존하며 보존 정책(Retention Policy)에 따라 오래된 산출물을 안전하게 제거해야 한다. 새로운 릴리스가 검증되기 전에 유일하게 알려진 정상 복구 버전을 삭제해서는 안 된다.

여러 컨테이너를 함께 변경해야 하는 경우 서비스 의존성(Service Dependency)이 업데이트를 복잡하게 만들 수 있다. 인식 서비스가 내비게이션에서 사용하는 메시지 형식을 발행하거나 API 게이트웨이가 특정 백엔드 버전에 의존할 수 있다. 배포 매니페스트(Deployment Manifest)는 호환 가능한 서비스 집합, 시작 순서, 상태 의존성 및 인터페이스 버전을 정의하여 업데이트 이후 로봇이 서로 호환되지 않는 컨테이너 조합으로 동작하는 것을 방지해야 한다.

일부 업데이트는 여러 서비스를 동시에 교체해야 하지만 다른 업데이트는 독립적으로 수행할 수 있다. 로봇 소프트웨어를 명확하게 정의된 서비스로 분리하면 업데이트 범위를 줄이고 변경되지 않은 구성요소를 계속 실행할 수 있다. 그러나 지나치게 세분화하면 오케스트레이션 복잡성(Orchestration Complexity)이 증가할 수 있다. 따라서 컨테이너 경계는 의미 있는 수명주기, 의존성, 보안 및 복구 경계를 반영해야 한다.

OTA 수명주기 전체에는 관찰 가능성(Observability)이 필요하다. 플릿 시스템은 업데이트 요청, 이미지 버전, 다이제스트, 다운로드 진행 상태, 활성화 상태, 상태 확인 결과, 롤백 이벤트 및 최종 배포 상태를 기록해야 한다. 운영자는 성공적으로 업데이트된 로봇과 오프라인 상태, 다운로드 중, 안전 조건 대기 중, 실패 상태 또는 이전 릴리스로 운용 중인 로봇을 구분할 수 있어야 한다.

CI/CD와 OTA는 하나의 추적 가능한 소프트웨어 전달 체인(Traceable Software Delivery Chain)을 구성해야 한다. 소스 커밋(Source Commit)은 테스트된 이미지로 이어지고 이미지는 불변 식별자와 보안 증거를 부여받으며 승인된 산출물은 레지스트리에 저장된다. 이후 배포 정책이 이를 로봇에 할당하고 텔레메트리(Telemetry)가 최종 런타임 상태를 확인한다. 이러한 추적성을 통해 현장에서 발생한 동작을 해당 동작을 생성한 정확한 소프트웨어 산출물과 연결할 수 있다.

컨테이너 OTA가 호스트 운영체제나 펌웨어 업데이트의 필요성을 제거하는 것은 아니다. 커널 보안 패치, 부트로더(Bootloader) 변경, GPU 드라이버, 장치 펌웨어 및 저수준 안전 제어기는 별도의 업데이트 메커니즘을 필요로 할 수 있다. 따라서 견고한 로봇 플랫폼은 응용 프로그램 컨테이너 업데이트와 호스트 및 펌웨어 수명주기 관리(Firmware Lifecycle Management)를 구분하면서 세 계층 사이의 호환성을 함께 관리해야 한다.

장애 처리(Failure Handling)는 전원 손실, 네트워크 중단, 컨테이너 충돌 및 부분 업데이트가 언젠가는 발생한다는 것을 전제로 설계해야 한다. 업데이트 작업은 재시작 가능해야 하며 취약한 명령 순서가 아니라 명시적인 상태(State)를 중심으로 설계해야 한다. 재부팅 이후 로봇은 자신이 다운로드, 검증, 활성화 또는 롤백 중 어느 단계에 있었는지 판단하고 알려진 안전한 소프트웨어 상태로 계속 진행할 수 있어야 한다.

성숙한 컨테이너 기반 OTA 시스템은 불변 산출물(Immutable Artifact), 안전한 배포, 단계적 롤아웃, 상태 검증, 롤백, 설정 관리, 플릿 대상 지정 및 지속적인 관찰 가능성을 결합한다. 목적은 단순히 새로운 소프트웨어를 원격으로 전송하는 것이 아니라 복구 가능성(Recoverability)과 운영 안전성(Operational Safety)을 유지하면서 물리적 로봇의 소프트웨어 상태를 예측 가능한 방식으로 변경하는 것이다.

로봇 플릿에서 컨테이너 기반 OTA는 DevOps의 운영 단계 확장으로 볼 수 있다. 컨테이너는 재현 가능한 배포 단위(Deployment Unit)를 제공하고 CI/CD는 검증된 릴리스를 생성하며 레지스트리는 신뢰할 수 있는 산출물을 배포하고 플릿 관리는 언제 어디에서 이를 활성화할지를 제어한다. 이러한 메커니즘을 결합하면 추적성, 보안, 통제된 위험 및 안정적인 복구 능력을 유지하면서 배포된 로봇 전체에서 지속적인 소프트웨어 발전을 가능하게 한다.
