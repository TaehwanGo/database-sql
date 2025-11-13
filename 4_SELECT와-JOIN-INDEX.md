## SELECT와 WHERE

- employee_project, project 테이블 생성

```sql
create table project
(
    id         INT auto_increment,
    name       varchar(45)                        not null,
    created_at datetime default current_timestamp not null,
    due_date   date                               null,
    constraint project_pk
        primary key (id)
);

create table employee_project
(
    employee_fk INT null,
    project_fk  INT null,
    constraint employee_fk
        foreign key (employee_fk) references employee (id)
            on delete cascade,
    constraint project_fk
        foreign key (project_fk) references project (id)
            on delete cascade
);
```

```sql
select * from zerocho.employee where team = '개발팀';
select * from zerocho.employee where team = '개발팀' and quit_date is null; # null을 비교하는 경우 is null / is not null 사용
```

```sql
select * from zerocho.employee where team = '개발팀' and (created_at between '2025-09-20' and '2025-09-30');
```

- and가 햇갈릴 땐 괄호로 묶어주기

```sql
select * from zerocho.employee where team = '개발팀' or team = '기획팀';
```

```sql
select * from zerocho.employee where team = '개발팀' or (team = '기획팀' and salary < 5000);
```

- or가 and보다 우선순위가 낮음

## 함수와 AS, ORDER BY

```sql
select count(*) from zerocho.employee where team = '개발팀';
-- select avg(salary) from zerocho.employee where team = '개발팀';
-- select avg(salary) as '총액' from zerocho.employee where team = '개발팀';
select avg(salary) as '총액' from zerocho.employee em where team = '개발팀'; -- 별칭(AS) 사용 가능, 테이블명도 가능
select avg(salary) as '총액', 5 as '컬럼' from zerocho.employee em where team = '개발팀';
select sum(salary) from zerocho.employee where team = '개발팀';
select max(salary) from zerocho.employee where team = '개발팀';
select min(salary) from zerocho.employee where team = '개발팀';
```

```sql
select * from zerocho.employee where team = '개발팀' order by salary; -- 오름차순
select * from zerocho.employee where team = '개발팀' order by salary desc; -- 내림차순
select * from zerocho.employee where team = '개발팀' order by salary desc, created_at; -- 여러개 정렬(1순위: salary, 2순위: created_at)
```

## 페이지네이션에 자주 쓰이는 LIMIT, OFFSET

### OFFSET 방식의 페이지네이션

```sql
select * from zerocho.employee where team = '개발팀' order by salary desc limit 2; -- 2개만 가져와
select * from zerocho.employee where team = '개발팀' order by salary desc limit 2 offset 2; -- 2개 건너뛰고 2개 가져와
```

- 장점: 구현이 간단하다
- 단점
  - 5개를 가져와야 되는데 페이지가 넘어갈 수록 OFFSET이 커져서 느려짐
  - 중간에 데이터가 삭제되면 건너뛰는 데이터가 생김(soft delete로 해결 가능)

### 커서 방식의 페이지네이션

- 데이터 예시
  - 게시글의 아이디: 15, 14, 13, 12, 11, 10, 9, 8, 7, 6, 5, 4, 3, 2, 1

```sql
select * from zerocho.employee where team = '개발팀' and id < 9; -- id가 9보다 작은 것들 중에서
```

## 통계를 낼 때 자주 쓰는 GROUP BY, HAVING

### GROUP BY

- 특정 컬럼을 기준으로 그룹화할 때 사용

```sql
select team, count(*) as '인원수' from zerocho.employee group by team;
select team, avg(salary) as '평균급여' from zerocho.employee group by team;
```

### HAVING

- GROUP BY한 결과에 조건을 걸 때 사용

```sql
select avg(salary), team from zerocho.employee
group by team having team = '개발팀' or team = '기획팀';

select avg(salary), team from zerocho.employee where salary > 5000
group by team;

select avg(salary), team from zerocho.employee where salary > 5000
group by team having team = '개발팀' or team = '기획팀';
```

- WHERE는 그룹화 전에 조건을 걸고, HAVING은 그룹화 후에 조건을 건다는 차이점이 있음

```sql
select avg(salary) as '평균급여', role_id from zerocho.employee
group by role_id;
```

## INNER JOIN

- employee 테이블과 role 테이블을 조인

### INNER JOIN

- 두 테이블에서 조건에 맞는 데이터만 가져옴

```sql
select * from zerocho.employee join zerocho.role; -- 모든 조합(CROSS JOIN) 사람 11명, 역할 5개 = 55개
select * from zerocho.employee join zerocho.role on zerocho.employee.role_id = zerocho.role.id; -- INNER JOIN
```

- group by는 having
- limit은 offset
- join은 on

```sql
select employee.id, e.name as '사원명', email, team, r.name as '직책명', min_salary
from zerocho.employee e join zerocho.role r
on zerocho.employee.role_id = zerocho.role.id;

select e.id, e.name as '사원명', email, team, r.name as '직책명', min_salary
from zerocho.employee e join zerocho.role r
on e.role_id = r.id;
```

- Column 'id' in field list is ambiguous
  - id 컬럼이 employee, role 테이블에 모두 존재하기 때문에 어느 테이블의 id인지 명확하지 않음

### OUTER JOIN

- LEFT JOIN: 왼쪽 테이블을 기준으로 조인

```sql
select * from zerocho.employee left join zerocho.role on zerocho.employee.role_id = zerocho.role.id;
```

- RIGHT JOIN: 오른쪽 테이블을 기준으로 조인

```sql
select * from zerocho.employee right join zerocho.role on zerocho.employee.role_id = zerocho.role.id;
```

## LEFT, RIGHT JOIN

- INNER JOIN: 둘을 합쳤을 때 null이 하나라도 있으면 제외(서로간의 관계를 맺고 있어야 포함)
- OUTER JOIN: null이 있어도 포함(서로간의 관계를 맺고 있지 않아도 포함)
  - LEFT JOIN: 왼쪽 테이블 기준
  - RIGHT JOIN: 오른쪽 테이블 기준

```sql
SELECT * FROM zerocho.employee e
LEFT JOIN zerocho.employee_project ep
ON e.id = ep.employee_fk;
```

- employee 테이블에 있는 모든 데이터가 나오고, employee_project에 매칭되는 데이터가 없으면 null로 표시됨

```sql
SELECT * FROM zerocho.employee e
RIGHT JOIN zerocho.employee_project ep
ON e.id = ep.employee_fk;
```

- employee_project 테이블에 있는 모든 데이터가 나오고, employee에 매칭되는 데이터가 없으면 null로 표시됨

```sql
SELECT * FROM (zerocho.employee e
LEFT JOIN zerocho.employee_project ep
ON e.id = ep.employee_fk)
LEFT JOIN zerocho.project p
ON ep.project_fk = p.id;
```

- 테이블 JOIN 이후 SELECT가 실행됨

## FULL OUTER JOIN, CROSS JOIN

### FULL OUTER JOIN

```sql
USE zerocho;
(SELECT * FROM employee e LEFT JOIN employee_project ep ON e.id = ep.employee_fk
LEFT JOIN project p ON ep.project_fk = p.id)
UNION
(SELECT * FROM employee e RIGHT JOIN employee_project ep ON e.id = ep.employee_fk
RIGHT JOIN project p ON ep.project_fk = p.id);
```

### CROSS JOIN(CARTESIAN JOIN)

- 두 테이블의 모든 조합을 보여줌

```sql
SELECT * FROM zerocho.employee JOIN zerocho.role;

SELECT * FROM zerocho.employee, zerocho.project;

SELECT * FROM zerocho.employee CROSS JOIN zerocho.role;
```

- ON을 빼먹으면 CROSS JOIN이 됨
- 또는 FROM 뒤에 콤마(,)로 구분하여 여러 테이블을 적으면 CROSS JOIN이 됨
- CROSS JOIN은 조인 조건이 없기 때문에 조인되는 두 테이블의 모든 조합을 결과로 반환함

## 검색을 빠르게 해주는 INDEX와 EXPLAIN

### INDEX

- employee 테이블 기준 인덱스가 있는 곳

  - PRIMARY KEY: id
  - UNIQUE INDEX: email
  - INDEX: role_id (foreign key)

- 인덱스가 필요한 이유: 검색 속도 향상
- 인덱스가 없는 경우: 테이블의 모든 데이터를 처음부터 끝까지 검색(Full Table Scan)
- 인덱스가 있는 경우: 인덱스를 통해 검색 대상이 되는 데이터의 위치를 빠르게 찾음

- 모든 컬럼에 인덱스를 걸면 좋은거 아닌가?
  - 용량 문제: 인덱스도 데이터베이스에 저장되기 때문에 인덱스가 많아지면 그만큼 저장 공간을 더 차지함
  - 성능 문제: 인덱스가 많아지면 데이터 삽입, 수정, 삭제 시 인덱스도 함께 갱신되어야 하기 때문에 오히려 성능이 저하될 수 있음
- 인덱스는 자주 검색되는 컬럼이나, WHERE 절에 자주 사용되는 컬럼에 주로 걸어주는 것이 좋음
- 인덱스가 있는 컬럼으로 정렬하면 빠름
- 유니크 컬럼은 기존이랑 겹치는지 확인하기 때문에 인덱스가 걸려있음

#### 인덱스를 만드는 명령어

```sql
ALTER TABLE employee ADD INDEX idx_team (team);
```

- team 컬럼에 idx_team이라는 이름으로 인덱스 생성

#### 인덱스를 삭제하는 명령어

```sql
ALTER TABLE employee DROP INDEX idx_team;
```

- team 컬럼에 걸려있는 idx_team 인덱스 삭제

#### 인덱스 순위

- 1순위 조건은 꼭 where절에 포함되어야 함

### EXPLAIN

- 쿼리 실행 계획을 보여주는 명령어
- 인덱스가 사용되는지, 어떤 방식으로 조인이 이루어지는지 등을 확인할 수 있음

```sql
EXPLAIN SELECT * FROM zerocho.employee WHERE team = '개발팀';
```

- type: ALL -> FULL TABLE SCAN

```sql
EXPLAIN SELECT * FROM zerocho.employee WHERE id = 6;
```

- type: const -> PRIMARY KEY나 UNIQUE INDEX로 검색할 때 사용됨

## 퀴즈

- `WHERE` 절과 `HAVING` 절의 주요 차이점은 무엇일까요?

  - `WHERE`는 개별 행 필터링, `HAVING`은 그룹 필터링에 사용됩니다.

- 두 테이블 조인 시, 왼쪽 테이블의 모든 행을 포함하고 일치하는 오른쪽 행만 가져오되, 불일치 시 NULL을 채우는 방식은 무엇일까요?

  - LEFT JOIN

- SQL 쿼리 결과에서 열이나 테이블 이름을 임시적으로 다른 이름으로 지정할 때 사용하는 키워드는 무엇일까요?

  - AS

- 데이터 검색 및 정렬 속도를 빠르게 하지만, 저장 공간을 더 차지하고 데이터가 변경될 때 추가 작업이 필요한 데이터베이스 요소는 무엇일까요?

  - INDEX

- `LIMIT`와 `OFFSET`을 사용한 기본적인 페이지네이션 방식의 단점은 무엇일까요?
  - OFFSET 방식은 데이터를 가져오는 시점에 새로운 데이터가 추가되거나 기존 데이터가 삭제되면 다음 페이지의 내용이 밀리거나 빠지는 등 사용자에게 혼란을 줄 수 있는 문제가 있어요.
