### 案例涉及的表
#### 雇员表 emp

|  字段   | 类型  |   描述 |
|  ----  | ----  | ---- |
| EMPNO  | NUMBER(4) | 表示雇员编号，是唯一编号 | 
| ENAME  | VARCHAR2(10) | 表示雇员姓名 | 
| JOB  | VARCHAR2(9) | 表示工作职位 | 
| MGR  | NUMBER(4) | 表示一个雇员的领导编号 | 
| HIREDATE  | DATE | 表示雇佣日期 | 
| SAL  | NUMBER(7,2) | 表示雇员月薪工资 | 
| COMM  | NUMBER(7,2) | 表示雇员奖金，或者佣金 | 
| DEPTNO  | NUMBER(2) | 表示雇员部门编号 | 


#### 部门表 dept

|  字段   | 类型  |   描述 |
|  ----  | ----  | ---- |
| DEPTNO  | NUMBER(2) | 表示部门编号，是唯一编号 | 
| DNAME  | VARCHAR2(14) | 部门名称 | 
| LOC  | VARCHAR2(14) | 部门位置 | 




#### 工资等级表 salgrade

|  字段   | 类型  |   描述 |
|  ----  | ----  | ---- |
| GRADE  | NUMBER | 等级名称 | 
| LOASL  | NUMBER | 此等级的最低工资 | 
| HISAL  | NUMBER | 此等级的最高工资 | 



#### 奖金表 bonus

|  字段   | 类型  |   描述 |
|  ----  | ----  | ---- |
| ENAME  | VARCHAR2(10) | 雇员姓名 | 
| JOB  | VARCHAR2(9) | 雇员工作 | 
| SAL  | NUMBER | 雇员工资 | 
| COMM  | NUMBER | 雇员奖金佣金 | 