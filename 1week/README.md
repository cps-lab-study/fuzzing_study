## 1주차 과제: Fuzzing 기초

### 핵심 목표
- 퍼징(Fuzzing)의 기본 개념을 이해하고 설명할 수 있다.
- AFL(American Fuzzy Lop)을 사용하여 간단한 프로그램의 크래시(Crash)를 유발하는 입력을 찾고, 이를 분석할 수 있다.

---

### 1. 개념 학습

- **학습 자료:** [Fuzzing Study (dhkstn.tistory.com)](https://dhkstn.tistory.com/entry/Fuzzing-Study)
- **학습 내용:**
    - Fuzzing의 정의 및 목적
    - Fuzzing의 종류 (Dumb vs Smart, Mutation-based vs Generation-based)
    - Fuzzing 기법 분류 (White-box, Grey-box, Black-box)
    - 대표적인 Fuzzer (AFL, LibFuzzer 등)의 특징

---

### 2. 실습 (Fuzzing101 - Exercise 1)

AFL의 기본적인 사용법을 익히고, 실제 프로그램의 취약점을 찾는 과정을 경험한다.

- **실습 저장소:** [antonio-morales/Fuzzing101](https://github.com/antonio-morales/Fuzzing101)
- **과제 디렉토리:** [Fuzzing101/Exercise 1](https://github.com/antonio-morales/Fuzzing101/tree/main/Exercise%201)

#### 진행 절차

1.  **환경 준비**
    - `git clone`으로 Fuzzing101 저장소를 로컬에 복제한다.
    - AFL, clang 등 실습에 필요한 도구를 설치한다. (저장소의 `Dockerfile` 참고)

2.  **계측(Instrumentation) 컴파일**
    - `Exercise 1`의 `basic.c` 소스 코드를 분석한다.
    - AFL의 `afl-clang-fast`를 사용하여 커버리지 측정이 가능하도록 코드를 컴파일한다.
      ```bash
      afl-clang-fast basic.c -o basic_fuzzed
      ```

3.  **초기 입력(Seed) 준비**
    - 퍼징의 시작점이 될 초기 입력 파일을 생성한다.
      ```bash
      mkdir inputs
      echo "FUZZ" > inputs/input.txt
      ```

4.  **퍼징 실행**
    - `afl-fuzz` 명령어를 사용하여 퍼징을 시작한다.
        - `-i`: 입력 시드 디렉토리 지정
        - `-o`: 결과 저장 디렉토리 지정
        - `@@`: 퍼저가 생성한 입력 파일이 전달될 위치
      ```bash
      afl-fuzz -i inputs -o outputs -- ./basic_fuzzed @@
      ```

5.  **결과 분석**
    - AFL UI를 통해 퍼징 상태(unique crashes 등)를 모니터링한다.
    - `outputs/crashes` 디렉토리에 크래시가 발견되면, 해당 입력으로 프로그램이 비정상 종료되는지 직접 재현해본다.
      ```bash
      # crashes 디렉토리에 파일이 생성된 경우
      ./basic_fuzzed < outputs/crashes/id:000000*
      ```
    - (선택) GDB와 같은 디버거를 이용해 크래시의 정확한 원인을 분석한다.
