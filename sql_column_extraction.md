# SQL Query for DML Statements Generation

이 SQL 쿼리는 Oracle 데이터베이스의 테이블 구조를 기반으로 DML 문(특히 SELECT와 INSERT 문)을 동적으로 생성합니다. 각 테이블의 칼럼에 대한 정보를 추출하고, 해당 정보를 이용하여 데이터 조작 언어(DML) 명령을 구성합니다.

## 쿼리 설명

쿼리는 `all_tab_columns`와 `all_col_comments` 시스템 뷰를 사용하여, 주어진 스키마의 테이블에서 칼럼 이름과 관련된 설명을 추출합니다. 이를 통해 각 칼럼에 대한 SELECT 구문과 INSERT 구문을 생성합니다.

### 구조

쿼리는 두 부분으로 구성됩니다:

1. **SELECT Statement Generation**:
   - `DECODE` 함수를 사용하여 첫 번째 칼럼(`COLUMN_ID = 1`)일 때는 "SELECT" 절을 시작하고, 그 외에는 칼럼 이름 앞에 콤마(,)를 추가합니다.
   - 칼럼 이름 뒤에는 고정된 길이의 공백을 추가하고, 칼럼 설명을 '--' 주석으로 붙여 넣습니다.

2. **INSERT Statement Generation**:
   - `DECODE` 함수를 사용하여 첫 번째 칼럼일 때는 "INSERT INTO" 절을 시작하고, 그 외에는 칼럼 이름 앞에 콤마를 추가합니다.
   - 각 칼럼 이름은 소문자로 변환되며, 필요한 공백과 주석이 추가됩니다.

### 사용된 함수 및 표현식

- `DECODE`: 조건에 따라 다른 문자열을 출력합니다.
- `LOWER`: 칼럼 이름을 소문자로 변환합니다.
- `LPAD`: 주어진 길이에 맞춰 문자열 왼쪽에 공백을 추가합니다.

## 예제 사용

이 쿼리는 테이블 'SM001KS'와 'COND02KS'에 대한 SELECT 및 INSERT 구문을 생성하기 위해 사용됩니다. 각 테이블에 대해 동적으로 DML 명령을 생성하며, UNION ALL을 사용하여 두 테이블의 결과를 하나의 결과로 결합합니다.


<pre>
<code>
SELECT *
FROM (
  SELECT DECODE(A.COLUMN_ID, 1, '      SELECT  ', '            , ') || DECODE(:alias1, NULL, NULL, :alias1 || '.') 
      || A.COLUMN_NAME || LPAD(' ', 35 - LENGTHB(A.COLUMN_NAME)) || '-- ' || B.Comments   AS "SELECT"
      ,DECODE(A.COLUMN_ID, 1, 'INSERT INTO   ', '            , ') || ':' || DECODE(:alias3, NULL, NULL, :alias3 || '.') 
      || LOWER(A.COLUMN_NAME) || LPAD(' ', 35 - LENGTHB(A.COLUMN_NAME)) || '-- ' || B.Comments   AS "INSERT"
  FROM all_tab_columns A, all_col_comments B
  WHERE a.owner = 'SMIT'  
    AND a.table_name = 'SM001KS'
    AND a.owner = b.owner
    AND a.table_name = b.table_name
    AND a.column_name = b.column_name
  ORDER BY COLUMN_ID
)
UNION ALL
SELECT *
FROM (
  SELECT '            , ' || DECODE(:alias2, NULL, NULL, :alias2 || '.')  || A.COLUMN_NAME 
      || LPAD(' ', 35 - LENGTHB(A.COLUMN_NAME)) 
      || '-- ' || B.Comments 
      ,'            , ' || ':' || DECODE(:alias4, NULL, NULL, :alias4 || '.')  || LOWER(A.COLUMN_NAME) 
      || LPAD(' ', 35 - LENGTHB(A.COLUMN_NAME)) 
      || '-- ' || B.Comments 
  FROM all_tab_columns A, all_col_comments B
  WHERE a.owner = 'SMIT'
    AND a.table_name = 'COND02KS'
    AND a.owner = b.owner
    AND a.table_name = b.table_name
    AND a.column_name = b.column_name
  ORDER BY COLUMN_ID
)
</code>
</pre>
