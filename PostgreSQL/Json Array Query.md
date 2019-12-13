# PostgreSQL에서 JSON Array 쿼리하기

>JSON Array를 쿼리하는 방법을 설명해드리겠습니다.  
>기본적인 PostgreSQL에서 제공하는 JSON 함수 및 연산자를 알고 계신다는 가정하에 설명드리겠습니다.  
>제 환경은 PostgreSQL 10 버전을 사용하고 있으며 기본 설정을 유지하고 있습니다.
>JSON 관련 함수 및 연산자를 정의한 공식 문서를 한 번 읽어보시는 것을 추천드립니다.  
> 링크: [JSON Functions and Operators](https://www.postgresql.org/docs/10/functions-json.html)


## 테이블 및 데이터 생성
> 가장 먼저 테이블을 생성하고 기초적인 데이터를 임의로 입력해줍니다.  
> 설명하기 쉽게 여러 회사 리스트 테이블을 만들고 JSONB 형식의 필드는 직원들로 정의하겠습니다.

```sql
-- 테이블 생성
CREATE TABLE company (
	id INTEGER PRIMARY KEY,
    name CHARACTER VARYING NOT NULL,
	employees JSONB NOT NULL
);

-- 데이터 입력
INSERT INTO company VALUES(1, '삼성', '[{"name":"홍길동1", "age": "20"}, {"name":"홍길동2", "age": "21"}, {"name":"홍길동3", "age": "22"}]');
INSERT INTO company VALUES(2, '카카오', '[{"name":"장동건1", "age": "30"}, {"name":"장동건2", "age": "31"}, {"name":"장동건3", "age": "32"}]');
INSERT INTO company VALUES(3, '네이버', '[{"name":"스티브1", "age": "40"}, {"name":"스티브2", "age": "41"}, {"name":"스티브3", "age": "42"}]');

-- 데이터 출력
SELECT * FROM company;
```
|id   |name   |employees                                                                                                      |
|:---:|:-----:|:-------------------------------------------------------------------------------------------------------------|
|1    |삼성   |[{"age": "20", "name": "홍길동1"}, {"age": "21", "name": "홍길동2"}, {"age": "22", "name": "홍길동3"}]         |
|2    |카카오 |[{"age": "30", "name": "장동건1"}, {"age": "31", "name": "장동건2"}, {"age": "32", "name": "장동건3"}]         |
|3    |네이버 |[{"age": "40", "name": "스티브1"}, {"age": "41", "name": "스티브2"}, {"age": "42", "name": "스티브3"}]         |



## 문제


### Q1. `삼성` 회사의 직원에 `이름: 공유, 나이: 15` 추가  
> 아래 쿼리문은 JSONB 타입에 || 연산자를 사용하여 데이터를 추가하여 출력합니다.
```sql
-- " employees || '{"name": "공유", "age": "15"}' "의 의미 
SELECT employees || '{"name": "공유", "age": "15"}'
FROM company
WHERE name = '삼성'
```
|id   |name   |employees                                                                                                      |
|:---:|:-----:|:-------------------------------------------------------------------------------------------------------------|
|1    |삼성   |[{"age": "20", "name": "홍길동1"}, {"age": "21", "name": "홍길동2"}, {"age": "22", "name": "홍길동3"}, **{"age": "15", "name": "공유"}**]         |

> PostgreSQL 같은 경우는 JSON 필드 내부 중 일부만 수정할 수 없으며 기존 JSON 데이터를 가공하여 만든 데이터를 저장하는 방식입니다.  
> 그래서 위에서 SELECT한 구문의 결과를 employees 필드에 새로 입력하는 방식입니다.
```sql
-- UPDATE 쿼리에 적용

-- 데이터 수정
UPDATE company
SET employees = employees || '{"name": "공유", "age": "15"}'
WHERE name = '삼성';

-- 데이터 출력
SELECT * FROM company;
```
|id   |name   |employees                                                                                                      |
|:---:|:-----:|:--------------------------------------------------------------------------------------------------------------|
|1    |삼성   |[{"age": "20", "name": "홍길동1"}, {"age": "21", "name": "홍길동2"}, {"age": "22", "name": "홍길동3"}, **{"age": "15", "name": "공유"}**]         |
|2    |카카오 |[{"age": "30", "name": "장동건1"}, {"age": "31", "name": "장동건2"}, {"age": "32", "name": "장동건3"}]         |
|3    |네이버 |[{"age": "40", "name": "스티브1"}, {"age": "41", "name": "스티브2"}, {"age": "42", "name": "스티브3"}]         |


### Q2. 모든 회사 직원들 중 이름이 "홍길동2"인 사람 삭제
> JSONB_ARRAY_ELEMENTS 함수를 사용하여 JSON을 각각의 데이터로 나눠줍니다.
```sql
SELECT id, JSONB_ARRAY_ELEMENTS(employees) AS employees
FROM   company;
```
|id   |name                                     |
|:---:|:----------------------------------------|
|1    |	{"age": "22", "name": "홍길동3"}        |
|1    | {"age": "21", "name": "홍길동2"}        |
|1    | {"age": "20", "name": "홍길동1"}        |
|1    | {"age": "15", "name": "공유"}           |
|2    | {"age": "30", "name": "장동건1"}        |
|2    | {"age": "31", "name": "장동건2"}        |
|2    | {"age": "32", "name": "장동건3"}        |
|3    | {"age": "42", "name": "스티브3"}        |
|3    | {"age": "41", "name": "스티브2"}        |
|3    | {"age": "40", "name": "스티브1"}        |

> "홍길동2"라는 이름을 가진 사람을 제거된 결과를 만들어야합니다.  
> 그래서 홍길동2를 제외한 결과를 불러옵니다.
> 여기서 "->>" 연산자는 키에 대한 값을 Text 형식으로 불러옵니다.  
>> 참고로 "->" 연산자는 키에 대한 값을 Json 형식으로 불러옵니다.  
>> employees->>2 과 같이 정수를 키로 주면 인덱스로 인식하여 1번 인덱스의 데이터를 불러옵니다.  
>
> JSON_AGG는 나눠진 데이터를 하나의 JSON Array로 만들어줍니다.
> 결과적으로는 JSON Array를 분해하여 "홍길동2"라는 사람의 데이터를 제외하고 다시 합치는 방식입니다.
```sql
SELECT id, JSON_AGG(employees) as employees
FROM (
	SELECT id, JSONB_ARRAY_ELEMENTS(employees) AS employees
	FROM   company
) t
WHERE employees->>'name' != '홍길동2'
GROUP BY id;
```
|id   |employees                                                                                                      |
|:---:|:--------------------------------------------------------------------------------------------------------------|
|1    |[{"age": "20", "name": "홍길동1"}, {"age": "22", "name": "홍길동3"}, {"age": "15", "name": "공유"}]    	      |
|2    |[{"age": "30", "name": "장동건1"}, {"age": "31", "name": "장동건2"}, {"age": "32", "name": "장동건3"}]         |
|3    |[{"age": "40", "name": "스티브1"}, {"age": "41", "name": "스티브2"}, {"age": "42", "name": "스티브3"}]         |


> 위와 같은 SELECT 문을 WITH 구문으로 정의하여 기존 employees를 대체하게 됩니다.
```sql
--데이터 갱신
WITH t as (SELECT id, JSON_AGG(employees) as employees
	FROM (
		SELECT id, JSONB_ARRAY_ELEMENTS(employees) AS employees
		FROM   company
	) t
	WHERE employees->>'name' != '홍길동2'
	GROUP BY id
)
UPDATE company c SET employees = t.employees
FROM t
WHERE c.id = t.id;

--데이터 출력
SELECT * FROM COMPANY;
```

|id   |name   |employees                                                                                                      |
|:---:|:-----:|:--------------------------------------------------------------------------------------------------------------|
|1    |삼성   |[{"age": "20", "name": "홍길동1"}, {"age": "22", "name": "홍길동3"}, {"age": "15", "name": "공유"}]  	        |
|2    |카카오 |[{"age": "30", "name": "장동건1"}, {"age": "31", "name": "장동건2"}, {"age": "32", "name": "장동건3"}]         |
|3    |네이버 |[{"age": "40", "name": "스티브1"}, {"age": "41", "name": "스티브2"}, {"age": "42", "name": "스티브3"}]         |

### Q3. 모든 회사 직원들 중 이름이 "스티브3"인 사람의 이름을 "스티브잡스"로 변경  
> "||" 연산자는 데이터를 연결하는 역할을 합니다.
> "JSONB_ARRAY_ELEMENTS"로 각각의 요소를 나눈뒤 "ORDINALITY"를 사용하여 각각 데이터와 인덱스를 "employee, index"로 정의합니다.
> 다음 함수를 사용하기위한 경로를 만들어줍니다.
> 보통 {인덱스, 키}와 같은 형식의 경로를 많이 사용합니다.
```sql
SELECT ('{'||t.index-1||', "name"}')::text[] as path, c.id
FROM company c, JSONB_ARRAY_ELEMENTS(c.employees) WITH ORDINALITY t(employee, index)
WHERE t.employee->>'name' = '스티브3'
```
|path 		|id	|
|:--------|:-:|
|{2,name}	|	3	|

> JSONB_SET은 JSONB_SET은(target jsonb, path text[], new_value jsonb [, create_missing boolean]) 와 같은 형식을 가집니다.  
> employees에서 해당 경로에대한 결과를 "스티브잡스"로 변경하는 역할을 합니다.  
> 변경된 값을 같은 id값을 가진 employees를 대체합니다.
```sql
--데이터 갱신
WITH t AS (
	SELECT ('{'||t.index-1||', "name"}')::text[] as path, c.id
	FROM company c, JSONB_ARRAY_ELEMENTS(c.employees) WITH ORDINALITY t(employee, index)
	WHERE t.employee->>'name' = '스티브3'
)
UPDATE company c
SET employees = JSONB_SET(employees, t.path, '"스티브잡스"')
FROM t
WHERE c.id = t.id;

--데이터 출력
SELECT * FROM COMPANY;
```

|id   |name   |employees                                                                                                      |
|:---:|:-----:|:--------------------------------------------------------------------------------------------------------------|
|1    |삼성   |[{"age": "20", "name": "홍길동1"}, {"age": "22", "name": "홍길동3"}, {"age": "15", "name": "공유"}]       		  |
|2    |카카오 |[{"age": "30", "name": "장동건1"}, {"age": "31", "name": "장동건2"}, {"age": "32", "name": "장동건3"}]         |
|3    |네이버 |[{"age": "40", "name": "스티브1"}, {"age": "41", "name": "스티브2"}, **{"age": "42", "name": "스티브잡스"}**]  |

## 결과적으로...
> JSON 데이터를 직접 수정해주는 방법은 없고 수정된 데이터를 이용하여 재입력하는 방식입니다.  
> 응용프로그램에서 미리 JSON데이터를 만들어서 통으로 넘기는 방식을 주로 쓸 것이고 위와 같은 경우는 드물것으로 생각합니다.
> JSON Array를 수정할 수 있으면 JSON Array가 아닌 JSON 데이터 수정은 더욱 쉬울 것으로 생각합니다.
> **꼭 PostgreSQL의 JSON 관련 공식 문서를 읽어보세요.**