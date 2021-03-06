### Pig 스크립트 

<hr>

**Pig 란**

- MapReduce 기반의 대규모 데이터 집합을 분석하기 위한 tool.
- Hadoop 기반으로 여러 데이터 manipulation operation을 통해 데이터를 분석한다.



처리할 데이터는 HDFS에 저장되어 있다고 가정한다.

```
> cat /input/test.log

key1	value1
key2	value3
key3	value3
```



**Pig Data Model**

- Bag: Tuple의 집합.

- Tuple: Field의 집합.

- Field: 데이터 조각.



### 데이터 읽기/쓰기 함수

**LOAD**

```
relation_name = LOAD 'input file path' USING function as schema;
```

input file path: 읽을 파일 path를 정의한다.

function: load function 중 선택한다. (BinStorage, JsonLoader, PigStorage, TextLoader)

schema: data의 스키마를 설정한다.



```
logs = LOAD '$input' AS (key: chararray, value: chararray);
```

input file은 parameter 형태로 받는다.

스키마는 key, value 형태로 지정한다.

function은 생략할 수 있다.



**STORE**

```
STORE relation_name INTO 'file path' [USING function];
```

file path: 쓸 파일 path를 정의한다.

function: function은 LOAD에서 소개한 function들과 동일하다.



```
STORE result_log INTO '$output' USING PigStorage('\t');
```

output file은 parameter 형태로 받는다.

PigStorage('\t')를 통해 필드 간 탭(\t)으로 구분한다.



### 필터 함수

**DISTINCT**

```
distinct_logs = DISTINCT logs;
```

중복을 제거할 logs을 적는다.



**FOREACH ... GENERATE**

```
FOREACH logs GENERATE key,value;
FOREACH logs GENERATE UDF_FUNCTION(value) as modify_value, key as key;
```

logs에서 tuple을 하나씩 읽고, 각 필드를 key, value로 구분한다.

logs에서 tuple을 하나씩 읽고, value 필드를 UDF_FUNCTION에 적용해 나온 결과를 modify_value 필드로 변경, key 필드는 그대로 key 필드로 둔다.



```
FOREACH logs GENERATE 
	key,
	value,
	FLATTEN(UDF_FUNCTION(value)) AS bag_value;
```

logs에서 tuple을 하나씩 읽고, 각 필드를 key, value로 구분한다.

마지막 bag_value는 value field를 읽어 Bag 형태로 만드는 UDF를 적용 후 FLATTEN() 함수에 적용한다.

```
key1	{"key1":"value1", "key2":"value2"}
key2	{"key1":"value1", "key2":"value2"}

-->

key 1	"key1":"value1"
key 1	"key2":"value2"
key 2	"key1":"value1"
key 2	"key2":"value2"
```



**FILTER BY**

```
FILTER logs BY (condition);
FILTER logs BY key IS NOT NULL AND log#'event' matches '@Event1'
```

특정 condition에 따라 logs에 filter를 수행한다.

logs의 key 필드가 NOT NULL이 아니여야 한다.

logs에 log 필드는 Map Type이기 때문에, #을 써서 특정 key의 value를 반환한다. 즉, event 키를 가진 value가 '@Event1'인지 아닌지 필터링 한다.



### 그룹 함수

**GROUP**

```
GROUP logs BY field;

> output
(field1, {(key, value), (key,value ... )})
(field2, {(key, value), (key,value ... )})
```

logs에서 group 하고 싶은 field를 정한다.



```
FOREACH (GROUP logs BY (field_1, field_2)) GENERATE
	FLATTEN(group) AS (field_1, field_2)
	COUNT($1) as count;
```

FOREACH랑 같이 사용하여, 특정 field group (field_1, field_2)에 COUNT나 SUM을 통해 집계할 수 있다.

SUM 집계를 하려면 field 중 Int 형 type을 지정해주어야 한다.



**JOIN**

```
JOIN logs1 BY key, logs2 BY key ;
JOIN logs1 BY (field1, field2, field3) LEFT OUTER logs2 BY (field1, field2, field3) ;
```

join 하고 싶은 두 Bag인 logs1, logs2의 각 field를 지정한다. Join은 크게 3가지가 있으며 위 예제는 inner join 이다.

- Self-join
- Inner-join
- Outer-join (left, right, full)



Self-join은 자체 Bag을 join할 때 사용하며, Outer-join은 inner-join과 마찬가지로 두 Bag을 join 하지만 left, right, full에 따라 match되지 않는 tuple도 출력한다.



### 정렬 함수

**ORDER BY ASC, DESC**

```
ORDER logs BY field DESC;
```

정렬하고 싶은 Bag인 logs에 field를 지정한다.

오름차순을 하고싶다면 ACS, 내림차순을 하고싶다면 DESC를 쓴다.



### 사용자 정의 함수

pig에 간단한 집계를 위한 함수들만 제공하기 때문에, 필요한 UDF 개발이 필요할 수 있다.

```java
import org.apache.pig.EvalFunc;
import org.apache.pig.data.DataBag;
import org.apache.pig.data.DataType;
import org.apache.pig.data.DefaultBagFactory;
import org.apache.pig.data.Tuple;
import org.apache.pig.data.TupleFactory;


public class UDF_FUNCTION extends EvalFunc<DataBag> {
    private static final TupleFactory TUPLE_FACTORY = TupleFactory.getInstance();
    
    @Override
    public DataBag exec(Tuple input) throws IOException {
        try {            
            DataBag bag = DefaultBagFactory.getInstance().newDefaultBag();
            
            for (int i = 0; i < tuple.size(); ++i) {
                if (tuple.getType(i) == DataType.TUPLE) {
                    bag.add((Tuple)tuple.get(i));
                } else if (tuple.getType(i) != DataType.NULL) {
                    bag.add(TUPLE_FACTORY.newTuple(tuple.get(i)));
                }
            }

            return bag;
        } catch (Exception e) {
            throw new RuntimeException("Error while creating a bag.", e);
        }
    }
}
```

EvalFunc<DataBag> 클래스를 상속받아 구현하며, exec() 메서드를 아래 구현하면 된다.















