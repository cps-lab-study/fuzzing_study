# 공부 내용 정리내용 정리

## 퍼징?
자동화된 소프트웨어 테스트 기법으로, 컴퓨터 프로그램에 무작위 데이터를 입력하여 소프트웨어 성능, 안정성 및 보안 취약점을 찾아내는 과정
-> 프로그램의 브루트포스 느낌?

**퍼징을 위한 도구가 퍼저**
### 퍼징을 왜 하는가?
Zero-Day 취약점을 찾기 위해서 퍼징을 진행함.
<img width="1280" height="464" alt="image" src="https://github.com/user-attachments/assets/72a4d446-f20f-4dbc-8490-924e0e82d707" />

## 퍼징 기법 분류
### 프로그램 정보를 활용하는 정도에 따른 분류
- White-box Testing
대상 프로그램의 내부 구조와 실행 중 발생하는 정보들을 사용하는 기법

1. Control-Flow Test(제어흐름 테스트)
2. Data Flow Test (데이터 흐름 테스트)
3. Branch Test (분기 테스트)
4. Prime Path Test (주요 경로 테스트)
5. Path Test(경로 테스트)

- Black-box Testing

대상 프로그램의 내부 정보를 사용하지 않고, 입력과 출력 데이터만 사용하는 기법, Black-box Testing은 소스코드에 대한 접근 권한이 없기에 주로 입출력 데이터를 기반으로 검증을 수행

- Gray-box Testing

대상 프로그램의 내부 정보 중 일부만 알고 진행하는 테스트 기법, white box와 black 박스의 중간?

### 입력 데이터를 생성하는 법에 따른 분류
- 생성 기반 퍼징 기법

데이터의 구조 및 프로토콜을 이해하여 프로그램에 적합한 입력 데이터를 생성하는 기법, 파일의 포맷이나 프로토콜 등을 이해하여 적절한 INPUT을 생성하는 방식

- 변이 기반 퍼징 기법

입력 데이터를 특정하여, 그 입력 데이터에 조금씩 변이를 주어 새로운 입력 데이터를 생성하는 기법(입력데이터는 시드라고 부름)

ex) Bit Flipping 기법

데이터의 특정 영역의 값들을 임의로 바꿔치기 추가하는것
<img width="1280" height="682" alt="image" src="https://github.com/user-attachments/assets/1c56828d-9132-4cec-88cb-e260fdd89d14" />


## Code Coverage

퍼징을 진행할 때, 소프트웨어 테스트 케이스가 얼마나 충족되었는지 나타내는 지표

1. 구문 커버리지 - 코드 한줄이 한번 이상 실행된다면 충족
2. 조건 커버리지 - 조건이 얼마나 실행되었는지
3. 결정 커버리지 - 프로그램 내의 모든 결정 포인트(if, for, while등)가 True와 False를 가지게 되면 충

## Fuzzer Architecture

<img width="768" height="512" alt="image" src="https://github.com/user-attachments/assets/53cee046-cc8b-4f9e-a722-019ec307788a" />

### test Case Generator
대상 프로그램에 입력할 데이터를 생성하는 부분
- Smart Fuzzer: 프로그램의 입력 데이터 구조를 분석하여 해당 구조에 맞게 변형
- Dumb Fuzzer: 입력의 랜덤 데이터를 생성하여 변형

### Logger
퍼징 과정 중 발생하는 버그나 이상 행위를 분석하기 위해 필요한 정보를 기록

### Worker
TestCase Generator에서 생성한 입력 데이터를 받아 실행하고, 이상 행위를 탐지

## AFL 전체 로직
<img width="1280" height="617" alt="image" src="https://github.com/user-attachments/assets/d6027042-ade2-4cc9-9b0f-f30c8a1d82bb" />

1. AFL-Fuzz에서 퍼징하려는 프로그램을 실행
2. 프로그램이 처음 실행될 때, AFL-Fuzz와의 파이프 통신을 위해 fork 서버를 초기화한다. 이후 fork() 호출을 사용하여 타겟 인스턴스를 새롭게 실행
3. 표준 입력이나 파일로부터의 입력이 퍼징 대상 프로그램으로 전달
4. 실행된 결과는 공유 메모리에 기록되며, 프로그램은 실행을 완료
5. AFL-Fuzz는 공유 메모리에서 퍼징 대상이 남긴 실행 기록을 분석하고, 해당 정보를 기반으로 새로운 입력을 생성
6. 생성한 새로운 입력은 다시 프로그램에 전달되어 실행

출처 : https://dhkstn.tistory.com/entry/Fuzzing-Study

