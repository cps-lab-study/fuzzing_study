# Fuzzing 기법 및 AFL++ Workflow

## 1. Fuzzing 개요

Fuzzing은 프로그램에 다양한 입력을 자동으로 주입하여 비정상적인 동작이나 취약점을 발견하는 소프트웨어 테스트 기법이다.
특히, 테스트 대상 프로그램의 **내부 정보를 얼마나 활용하는지**에 따라 Fuzzing은 크게 **Black-box Fuzzing**, **White-box Fuzzing**, **Gray-box Fuzzing**으로 구분된다.

---

### 1.1. Black-box Fuzzing

**Black-box Fuzzing**은 프로그램의 내부 구조를 전혀 보지 않고 수행하는 방식이다.
입력 값을 무작위로 생성하고, 프로그램이 출력하거나 예외를 처리하는 과정을 모니터링하여 취약점을 탐지한다.

* 특징: 내부 구조를 전혀 보지 않음
* 장점: 단순하고 빠르게 시작할 수 있음
* 단점: 코드 커버리지가 낮고, 탐지 효율성이 떨어질 수 있음
* 대표적인 예: **funfuzz**, **Peach** 등

---

### 1.2. White-box Fuzzing

**White-box Fuzzing**은 프로그램 내부 구조와 동작 원리에 대한 정보를 분석한 뒤, 이 정보를 바탕으로 입력 데이터를 생성하여 취약점을 탐색하는 방식이다.
주로 정적 분석과 심볼릭 실행(Symbolic Execution) 기법이 활용된다.

* 특징: 프로그램 내부 분석 필요
* 장점: 높은 정확성과 코드 경로 탐색 가능
* 단점: 분석에 많은 비용과 시간이 필요함
* 대표적인 예: **Driller**, **SAGE** 등

---

### 1.3. Gray-box Fuzzing

**Gray-box Fuzzing**은 Black-box와 White-box 방식의 중간 형태이다.
프로그램의 내부 정보를 **일부만 활용**하여 취약점을 탐지하며, 주로 **코드 커버리지 정보**를 수집하여 새로운 코드 경로를 탐색한다.
이를 통해 입력 데이터를 효율적으로 조작할 수 있고, 실제 소프트웨어 보안 테스트에서 가장 널리 활용된다.

* 특징: 내부 정보를 부분적으로 활용 (예: 코드 커버리지, 실행 흐름 등)
* 장점: 효율성과 탐색 능력의 균형을 가짐
* 단점: 여전히 특정한 코드 경로는 도달하기 어려움
* 대표적인 예: **AFL**, **AFL++**, **libFuzzer** 등

---

## 2. AFL++ Workflow

AFL++는 가장 널리 쓰이는 Gray-box Fuzzer 중 하나로, **Coverage-guided Fuzzing** 기법을 사용한다.
소스코드가 있을 때 기본적인 퍼징 과정은 **3단계**로 요약된다.

1. 계측 (Instrumentation)
2. Corpus 준비
3. 퍼징 실행 및 Crash Triaging

---

### 2.1. 계측 (Instrumentation)

퍼징 대상 프로그램을 AFL++ 전용 컴파일러로 빌드하여 코드 커버리지를 추적할 수 있도록 준비한다.

* **컴파일러 모드 선택**

  1. `afl-clang-lto` (LLVM 11+, 권장 / 가장 빠르고 강력)
  2. `afl-clang-fast` (LLVM 3.8+ 지원)
  3. `afl-gcc-fast` (GCC Plugin, LLVM이 없을 경우 대안)

* **예시 (configure 기반 빌드)**:

  ```bash
  CC=afl-clang-fast CXX=afl-clang-fast++ ./configure --disable-shared
  make
  ```

* **추가 계측 옵션**

  * `AFL_LLVM_LAF_ALL=1` : 비교문 단순화 (COMPCOV)
  * `AFL_LLVM_CMPLOG=1` : 비교값 추적 (Redqueen)
  * `AFL_USE_ASAN=1` : AddressSanitizer
  * `AFL_USE_UBSAN=1` : Undefined Behavior Sanitizer

---

### 2.2. Corpus 준비

퍼저가 효율적으로 동작하려면 적절한 **입력(seed corpus)** 이 필요하다.
Corpus는 가능한 한 다양한 입력을 모아야 하며, 중복을 제거하고 최소화해야 퍼징 속도가 향상된다.

#### 2.2.1. 시드 수집

* 대상 포맷의 실제 파일을 가능한 많이 모은다.
* 예: PDF 퍼징의 경우 다양한 PDF 샘플을 corpus에 포함한다.

#### 2.2.2. 중복 제거 (`afl-cmin`)

Corpus에서 중복되는 입력을 제거하여 새로운 코드 경로를 탐색하는 데 도움이 되도록 한다.

```bash
afl-cmin -i seeds -o seeds_unique -- ./target @@
```

#### 2.2.3. 최소화 (`afl-tmin`)

파일을 가능한 한 단순하게 줄이면서도 동일한 코드 경로를 탐색하게 만든다.
Crash 입력에도 동일하게 적용 가능하다.

```bash
afl-tmin -i crash.pdf -o minimized.pdf -- ./target @@
```

---

### 2.3. 퍼징 실행

#### 2.3.1. 기본 실행

```bash
afl-fuzz -i input -o output -- ./target @@
```

* `-i` : 입력 corpus 디렉토리
* `-o` : 결과 저장 디렉토리 (queue, crashes, hangs 구조)
* `@@` : fuzzing 입력 파일이 삽입될 자리

#### 2.3.2. 주요 옵션

* `-m N` → 메모리 제한 (MB 단위)
* `-t N` → 실행 타임아웃 (ms 단위)
* `-x dict.dict` → Dictionary 추가 (예: 포맷 키워드)

#### 2.3.3. 멀티코어 퍼징

여러 코어를 활용할 경우 **하나의 Master 노드**와 **여러 Secondary 노드**를 설정한다.

```bash
# Master 노드
afl-fuzz -M main -i input -o output -- ./target @@

# Secondary 노드
afl-fuzz -S sec1 -i input -o output -- ./target @@
afl-fuzz -S sec2 -i input -o output -- ./target @@
```

---

### 2.4. Crash Triaging

퍼징 도중 발생한 Crash는 `output/default/crashes/`에 저장된다.
Crash는 중복이 많으므로 정리 과정이 필요하다.

#### 2.4.1. 중복 Crash 제거

```bash
afl-cmin -i crashes -o unique_crashes -- ./target @@
```

#### 2.4.2. Crash 최소화

```bash
afl-tmin -i crash_input -o minimized_crash -- ./target @@
```

#### 2.4.3. 퍼징 상태 확인

```bash
afl-whatsup output/
```

#### 2.4.4. 퍼징 결과 시각화

```bash
afl-plot output/default plot_dir/
```

생성된 HTML 리포트를 통해 퍼징 진행 상황과 성과를 한눈에 확인할 수 있다.

---
