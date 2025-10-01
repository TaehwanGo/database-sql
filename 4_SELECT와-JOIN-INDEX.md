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
