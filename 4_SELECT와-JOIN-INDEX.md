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
