DB의 메타정보(테이블, 컬럼, 인덱스 등의 스키마 정보)를 모아둔 DB입니다. 
이 정보들은 읽기 전용 데이터로 조회만 가능합니다.

```sql
-- 특정 컬럼 없는 테이블 찾기 

**select** tab.table_schema **as** database_name,

       tab.table_name

**from** information_schema.tables tab

**left** **join** information_schema.**columns** col

          **on** tab.table_schema = col.table_schema

          **and** tab.table_name = col.table_name

          **and** col.column_name = 'COLUMN NAME'    -- put column name here

**where** tab.table_schema **not** **in** ('information_schema', 'mysql',

                           'performance_schema', 'sys')

      **and** tab.table_type = 'BASE TABLE'

      **and** col.column_name **is** **null**

**order** **by** tab.table_schema,

         tab.table_name;

-- 특정 컬럼 있는 테이블 찾기 

**select** table_name

**from** information_schema.**columns**

**where** column_name = 'CHNG_EMNB'

```

| 열 이름        | 설명                             | 데이터 형식        |
| ------------ | ------------------------------ | ------------- |
| table_schema | 테이블 스키마                        | nvarchar(128) |
| table_name   | 테이블 이름                         | sysname       |
| table_type   | 테이블 형식. 'VIEW' 또는 'BASE TABLE' | varchar(10)   |
| column_name  | 컬럼 이름                          |               |
| tables       | 모든 테이블 정보                      |               |
| columns      | 모든 테이블의 컬럼 정보                  |               |
|              |                                |               |
|              |                                |               |
|              |                                |               |
* nvarchar: 한글, 영문 구분 없이 1개의 문자당 2바이트 사용
* varchar: 한글은 2바이트, 영문은 1바이트 사용
* VIEW: buffer, global, session정보들이 나옴
* BASE TABLE: 실제 사용하는 테이블 정보가 나옴
![](https://i.imgur.com/pdOE6jd.png)




[table_type 정보](https://learn.microsoft.com/ko-kr/sql/relational-databases/system-information-schema-views/tables-transact-sql?view=sql-server-ver16)
[Find table that DON'T have a column with specific name in MySQL database](https://dataedo.com/kb/query/mysql/find-table-that-dont-have-a-column-with-specific-name)
