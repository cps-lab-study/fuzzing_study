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
- 변이 기반 퍼징 기법
