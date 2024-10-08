# ERD와 정규화

#### 사원 테이블

| 아이디(기본키) | 이름   | 전화번호      | 이메일        | 연봉  | 팀     | 역할     | 프로젝트     |
| -------------- | ------ | ------------- | ------------- | ----- | ------ | -------- | ------------ |
| 1              | 홍길동 | 010-1234-5678 | abc@gmail.com | 5000  | 개발   | 개발자   | 홈페이지, AI |
| 2              | 김철수 | 010-1234-5678 | bcd@gmail.com | 6000  | 디자인 | 디자이너 | AI           |
| 3              | 박영희 | 010-1234-5678 | cde@gmail.com | 7000  | 기획   | 기획자   | 홈페이지     |
| 4              | 이영수 | 010-1234-5678 | def@gmail.com | 8000  | 개발   | 개발자   | 홈페이지, AI |
| 5              | 김영희 |               | efg@gmail.com | 9000  | 디자인 | 디자이너 | 홈페이지     |
| 6              | 박철수 |               | fgh@gmail.com | 10000 | 기획   | 기획자   | 홈페이지     |

## DB를 엑셀이라고 생각하자

### 1정규형

- 1차 정규형: 모든 속성값이 원자값(더 이상 분해되지 않는 값)으로만 되어 있어야 한다.
- 2차 정규형: 기본키가 아닌 속성은 기본키 전체에 의존해야 한다.
- 3차 정규형: 기본키가 아닌 속성은 기본키 이외의 다른 속성에 의존하면 안 된다.

#### 전화번호 테이블

| 이름   | 전화번호      |
| ------ | ------------- |
| 홍길동 | 010-1234-5678 |
| 김철수 | 010-1234-5678 |
| 박영희 | 010-1234-5678 |
| 이영수 | 010-1234-5678 |
| 김영희 |               |
| 박철수 |               |

## 아이디가 필요한 이유(기본키, 외래키)

#### 프로젝트 테이블

| 이름   | 프로젝트     |
| ------ | ------------ |
| 홍길동 | 홈페이지, AI |
| 김철수 | AI           |
| 박영희 | 홈페이지     |
| 이영수 | 홈페이지, AI |
| 김영희 | 홈페이지     |
| 박철수 | 홈페이지     |

- 이름이 중복될 수 있기 때문에 아이디가 필요하다
- 이메일은 고유(unique)하다
  - 고유한 것은 다른 테이블에 들어갈 수 있다

#### 프로젝트 테이블 수정

| 이메일(외래키) | 프로젝트 |
| -------------- | -------- |
| abc@gmail.com  | 홈페이지 |
| abc@gmail.com  | AI       |
| bcd@gmail.com  | AI       |
| cde@gmail.com  | 홈페이지 |
| def@gmail.com  | 홈페이지 |
| def@gmail.com  | AI       |
| efg@gmail.com  | 홈페이지 |
| fgh@gmail.com  | 홈페이지 |

### 정규화

- 삽입, 수정, 삭제 시 문제가 없는지 생각해보기
- 절대 안바뀌는 것을 키로 만들어주자
  - 이메일은 바뀔 수도 있다(외래키로 쓰기 부적절)

#### 프로젝트 테이블 수정2

| 아이디 | 사원 아이디(외래키) | 프로젝트 |
| ------ | ------------------- | -------- |
| 1      | 1                   | 홈페이지 |
| 2      | 1                   | AI       |
| 3      | 2                   | AI       |
| 4      | 3                   | 홈페이지 |
| 5      | 4                   | 홈페이지 |
| 6      | 4                   | AI       |
| 7      | 5                   | 홈페이지 |
| 8      | 6                   | 홈페이지 |

## 아이디를 연이은 숫자로 할 때 단점

- 아이디가 연이은 숫자이면 회사의 정보가 노출될 수 있다(개수를 알 수 있음)

## 일대일, 일대다, 다대다 관계(ERD)

- 일대다
  - 사원 : 전화번호 = 1 : N
- 다대다
  - 사원 : 프로젝트 = M : N

### 다대다인 경우 중간 테이블이 필요하다

#### 사원-프로젝트 테이블

| 사원 아이디(외래키) | 프로젝트 아이디(외래키) |
| ------------------- | ----------------------- |
| 1                   | 1                       |
| 1                   | 2                       |
| 2                   | 2                       |
| 3                   | 1                       |
| 4                   | 1                       |
| 4                   | 2                       |
| 5                   | 1                       |
| 6                   | 1                       |

### 일대일: 자주 사용하지 않는 정보는 다른 테이블로 분리할 수도 있다

- 컬럼이 너무 많아지면 분리할 수 있다
- 합쳐서 쓰는 것이 더 효율적일 수도 있다

### ERD 그릴 때

- https://www.erdcloud.com/

### 정규화 예시(슬랙 따라 만들기)

- https://www.erdcloud.com/d/QqKNELMS6e3AB3xia
- users, workspaces, channels, channel_members, workspace_members, channel_messages, dm_messages

- users : dm_messages = 1 : N
- users : workspaces = M : N
  - 참가자입장에선 M : N
  - workspace_members => users, workspaces의 중간 테이블
- users : workspace = 1 : N
  - owner인 경우 1 : N

## soft delete vs hard delete

- soft delete: 삭제 플래그를 넣어서 삭제한 것처럼 보이게 하는 것
- hard delete: 실제로 삭제하는 것
