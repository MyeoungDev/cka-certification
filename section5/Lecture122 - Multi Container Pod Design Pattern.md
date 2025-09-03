# Lecture 122 - Multi Container Pod Design Pattern

## Multi Container Pod Design Pattern

### Multi Containers (Co-Located Container)

- 가장 기본적인 멀티 컨테이너 패턴.
- 파드 내의 모든 컨테이너가 시작 순서 보장 없이 동시에 시작되어 파드의 전체 수명 주기 동안 함께 실행.
- 두 서비스가 서로 강하게 의존하고 있을 대 주로 사용.

### Init Containers

- 메인 애플리케이션 컨테이너가 시작되지 전에, 필수적인 초기화 작업을 수행하기 위해 사용.
- 초기화 컨테이너가 작업을 마치고 성공적으로 종료되어야만 메인 컨테이너 시작.
- 여러 개의 최화 컨테이너를 정의할 경우, 정의된 순서대로 동작.

### Sidecar Containers

- 메인 컨테이너와 함께 파드의 수멍 주기 동안 계속 실행되는 보조 컨테이너.
- 메인 컨테이너의 기능을 보완하거나 확장하는 역할.
- 작업이 끝나도 종료되지 않고 메인 컨테이너와 동일하게 계속 실행.

### **Co-Located Container vs Sidecar Container**

- 두 패턴 모두 파드의 생명 주기 동안 함께 실행되지만, 가장 큰 차이점은 컨테이너의 시작 순서를 제어할 수 있는지 여부이다.
    - Co-located Container
        - 시작 순서에 대한 보장이 없어 모든 컨테이너가 동시에 시작된다.
    - Sidecar Container
        - 시작 순서를 명확하게 지정하고, 메인 애플리케이션이 실행되는 동안에도 계속 유지할 때 사용.