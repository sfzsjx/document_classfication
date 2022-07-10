# Neo4j Cypher refcard

## 1. 读操作

### <font color = orange size = 5>1.1 读操作的结构</font>

```cypher
[MATCH WHERE]
[OPTIONAL MATCH WHERE]
[WITH [ORDER BY] [SKIP] [LIMIT]]
RETURN [ORDER BY] [SKIP] [LIMIT]
```

### <font color = orange size = 5>1.2 MATCH 子句</font>

```cypher
//节点的模式可以包含标签和属性
MATCH (n:Person)-[:KNOWS]->(m:Person)
WHERE n.name = 'Alice'

//MATCH 语句中可以使用任何模式
MATCH (n)-->(m)

//带有节点属性的模式
MATCH (n {name: 'Alice'})-->(m)

//将路径保存在变量 p 中
MATCH p = (n)-->(m)

//可选模式: 缺少部分用 nulls
OPTIONAL MATCH (n)-[r]->(m)
```



### <font color = orange size = 5>1.3 WHERE 子句</font>

#### <font color = orang size = 4>1.3.1 用判断式进行过滤</font>

```cypher
WHERE n.property <> $value
```

注意, WHERE 只能是 MATCH , OPTIONAL MATCH 或 WITH 语句的一部分。查询中将其放在不同语句之后产生的效果会不同



#### <font color = orang size = 4> 1.3.2 使用存在性判断的子查询进行过滤</font>

```cypher
WHERE EXISTS {
MATCH (n)-->(m) WHERE n.age = m.age
}
```





### <font color = orange size = 5>1.4 WITH 子句</font>

#### 1.4.1 WITH 语法类似于 RETURN 。显式分隔查询语句,用于声明需要向后续语句传递的变量。

```cypher
MATCH (user)-[:FRIEND]-(friend)
WHERE user.name = $name
WITH user, count(friend) AS friends
WHERE friends > 10
RETURN user
```

#### 1.4.2 ORDER BY , SKIP 和 LIMIT 可以与 WITH 一起使用。

```cypher
MATCH (user)-[:FRIEND]-(friend)
WITH user, count(friend) AS friends
ORDER BY friends DESC
SKIP 1
LIMIT 3
RETURN user
```



### <font color = orange size = 5>1.5 RETURN 子句</font>

```cypher
//返回所有变量的值。
RETURN *

//使用别名作为结果的列名。
RETURN n AS columnName

//返回不重复的行。
RETURN DISTINCT n

//结果排序。
ORDER BY n.property

//结果降序排序。
ORDER BY n.property DESC

//跳过某些结果。
SKIP $skipNumber

//限制结果数。
LIMIT $limitNumber

//跳过前边的一些结果并限制结果数。
SKIP $skipNumber LIMIT $limitNumber

//结果数量。
RETURN count(*)
```




###  <font color = orange size = 5>1.6 UNION 子句</font>

#### 1.6.1 返回所有查询结果的无重复并集,结果列的类型和名称必须一致。

```cypher
MATCH (a)-[:KNOWS]->(b)
RETURN b.name
UNION
MATCH (a)-[:LOVES]->(b)
RETURN b.name
```

#### 1.6.2 返回所有查询结果的并集,包括重复行。

```cypher
MATCH (a)-[:KNOWS]->(b)
RETURN b.name 
UNION ALL
MATCH (a)-[:LOVES]->(b)
RETURN b.name
```



## 2. 写操作

### 2.1 Write-Only 结构

```cypher
(CREATE | MERGE)*
[SET|DELETE|REMOVE|FOREACH]*
[RETURN [ORDER BY] [SKIP] [LIMIT]]
```



### 2.2 Read-Write 结构

```cypher
[MATCH WHERE]
[OPTIONAL MATCH WHERE]
[WITH [ORDER BY] [SKIP] [LIMIT]]
(CREATE | MERGE)*
[SET|DELETE|REMOVE|FOREACH]*
[RETURN [ORDER BY] [SKIP] [LIMIT]]
```



### 2.3 CREATE 子句

```cypher
// 创建给定属性的节点。
CREATE (n {name: $value})
            
//创建给定属性的节点。
CREATE (n $map)
         
//创建给定属性的节点
UNWIND $listOfMaps AS properties
CREATE (n) SET n = properties

//创建具有给定类型和方向的关系;绑定一个变量。
CREATE (n)-[r:KNOWS]->(m)

//创建具有给定类型、方向和属性的关系。
CREATE (n)-[:LOVES {since: $value}]->(m)
```



### 2.4 SET 子句

```cypher
//更新或创建属性。
SET n.property1 = $value1, n.property2 = $value2

//设置所有属性。这将删除所有现有属性。
SET n = $map

//添加和更新属性,同时保留现有属性。
SET n += $map

//将标签添加 Person 到节点。
SET n:Person
```



### 2.5 MERGE 子句

```cypher
//匹配一个模式,如果不存在则创建一个新的模式。使用 ON CREATE 和 ON MATCH 执行带条件的 update。
MERGE (n:Person {name: $value})
ON CREATE SET n.created = timestamp()
ON MATCH SET
n.counter = coalesce(n.counter, 0) + 1,
n.accessTime = timestamp()

//查找或创建节点之间的关系。
MATCH (a:Person {name: $value1}), (b:Person {name: $value2})
MERGE (a)-[r:LOVES]->(b)

//查找或创建连接节点的路径。                      
MATCH (a:Person {name: $value1})
MERGE (a)-[r:KNOWS]->(b:Person {name: $value3})
```



### 2.6 DELETE 子句

```cypher
//删除一个节点和一个关系。
DELETE n, r

//删除一个节点以及与其连接的所有关系。
DETACH DELETE n

//删除数据库中所有节点和关系。
MATCH (n) DETACH DELETE n

```



### 2.7 REMOVE 子句

```cypher
//清除标签。
REMOVE n:Person

//清除属性。
REMOVE n.property
```



### 2.8 FOREACH 子句

```cypher
//对路径中的每个关系执行操作。
FOREACH (r IN relationships(path) | SET r.marked = true)

//对列表中的每个元素执行操作。
FOREACH (value IN coll |CREATE (:Person {name: value}))
```



### 2.9 CALL 子查询

```cypher
//调用具有联合的子查询,随后可以对结果进行进一步处理。
CALL {
MATCH (p:Person)-[:FRIEND_OF]->(other:Person) RETURN p, other
UNION
MATCH (p:Child)-[:CHILD_OF]->(other:Parent) RETURN p, other
}

//CALL 子程序
//对内置过程 db.labels 的独立调用,可以列出数据库中使用的所有标签。
//注意,所需的过程参数要在过程名称后的括号中明确给出。
CALL db.labels() YIELD label

//独立调用可以省略 YIELD ,也可以通过语句参数隐式提供参数,例如,可以通过传递 map参数 {input: 'foo'} 来运行需要一个 input 参数的独立调用。
CALL java.stored.procedureWithArgs

//在更大的查询中调用内置过程 db.labels 来计算数据库中使用的所有标签。要在更大的查询中调用则一定要传递参数并使用 YIELD 显式命名结果。
CALL db.labels() YIELD label
RETURN count(label) AS count

```



### 2.10 IMPORT 子句

```cypher
//从 CSV 文件加载数据并创建节点。
LOAD CSV FROM 'https://neo4j.com/docs/cypher- refcard/4.2/csv/artists.csv' AS line
CREATE (:Artist {name: line[1], year: toInteger(line[2])})

//加载包含标题的 CSV 数据。
LOAD CSV WITH HEADERS FROM
'https://neo4j.com/docs/cypher-refcard/4.2/csv/artists- with-headers.csv' AS line
CREATE (:Artist {name: line.Name, year: toInteger(line.Year)})

//导入大量数据时,每隔 500 行提交一次当前事务。
USING PERIODIC COMMIT 500 LOAD CSV WITH HEADERS FROM
'https://neo4j.com/docs/cypher-refcard/4.2/csv/artists- with-headers.csv' AS line
CREATE (:Artist {name: line.Name, year: toInteger(line.Year)})

//使用其他字段分割符,而不是默认的逗号(不带空格)。
LOAD CSV FROM 'https://neo4j.com/docs/cypher-refcard/4.2/csv/artists-fieldterminator.csv' AS line
FIELDTERMINATOR ';'
CREATE (:Artist {name: line[1], year: toInteger(line[2])})

//返回 LOAD CSV 正在处理的文件的绝对路径,如果 file() 不在 LOAD CSV 中调用则返回 null 。
LOAD CSV FROM 'https://neo4j.com/docs/cypher- refcard/4.2/csv/artists.csv' AS line
RETURN DISTINCT file()
                                                    
//返回 LOAD CSV 当前正在处理的行号,如果 linenumber () 不在 LOAD CSV 中调用则返回 null 。
LOAD CSV FROM 'https://neo4j.com/docs/cypher- refcard/4.2/csv/artists.csv' AS line
RETURN linenumber()
```



## 3. 通用类

### 3.1 运算符

```properties
通用: DISTINCT,.,[]
数学运算符: +,-,*,/,%,^
比较运算符: =,<>,<,>,<=,>=,IS NULL,IS NOT NULL
Boolean 运算符: AND,OR,XOR,NOT
String 运算符: +
List 运算: +,IN,[x],[x .. y]
正则表达式: =~
字符串匹配: STARTS WITH,ENDS WITH,CONTAINS
```



### 3.2 null

- null 表示缺失/未定义的值。
- null 不等于 null 。两个值未知并不意味着它们相同,因此 null = null 的返回结果是 null 而不是 true 。要检查是否为 null ,使用 IS NULL 。
- 如果算术运算、比较运算和函数调用( coalesce 除外)的任一参数为 null ,则结果返回null 。
-  访问列表中不存在的元素或访问不存在的属性会返回 null 。
-  在 OPTIONAL MATCH 子句中, nulls 将用于模式的缺失部分。



### 3.3 模式

```cypher
//带有 Person 标签的节点。
(n:Person)
//同时带有 Person 和 Swedish 标签的节点。
(n:Person:Swedish)
//具有已声明的属性的节点。
(n:Person {name: $value})
//有指定属性的关系。
()-[r {name: $value}]-()
//从 n 到 m 的关系。
(n)-->(m)
//n 和 m 之间任意方向的关系。
(n)--(m)
//带有 Person 标签的节点 n 与节点 m 的关系。
(n:Person)-->(m)
//从 n 到 m 的 KNOWS 类型的关系。
(m)<-[:KNOWS]-(n)
//从 n 到 m 的 KNOWS 或 LOVES 类型的关系。
(n)-[:KNOWS|:LOVES]->(m)
//将关系绑定到变量 r 。
(n)-[r]->(m)
//从 n 到 m 的、路径长度为1到5的关系。
(n)-[*1..5]->(m)
//从 n 到 m 的、任何路径长度的关系。参见“性能”部分。
(n)-[*]->(m)
//节点 n 到具有指定属性的节点 m 之间的 KNOWS 类型关系 。
(n)-[:KNOWS]->(m {property: $value})
//找到一条最短路径。
shortestPath((n1:Person)-[*..6]-(n2:Person))
//找到所有最短路径。
allShortestPaths((n1:Person)-[*..6]->(n2:Person))
//与模式匹配的路径的条数。
size((n)-->()-->())
```

### 3.4 Label

```cypher
//创建带有标签和属性的节点。
CREATE (n:Person {name: $value})
//匹配或创建带有标签和属性的唯一节点。
MERGE (n:Person {name: $value})
//给节点添加标签。
SET n:Spouse:Parent:Employee
//匹配标签为 Person 的节点。
MATCH (n:Person)
//匹配标签为 Person 且有指定 name 属性的节点。
MATCH (n:Person) WHERE n.name = $value
//判断节点的标签存在。
WHERE (n:Person)
//节点的所有标签。
labels(n)
//清除节点的标签。
REMOVE n:Person
```

### 3.5 List
```cypher
//用方括号声明 list。
['a', 'b', 'c'] AS list
//列表可以作为参数传递。
size($list) AS len, $list[0] AS value
//range() 创建数字型
range($firstNum, $lastNum, $step) AS list
//这些函数也返回 list 结果: labels() , nodes() , relationships() 。
list( step 可选)。

//使用命名路径和 relationships() 得到包含可变长度路径的关系的 list。
MATCH p = (a)-[:KNOWS*]->() RETURN relationships(p) AS r
//属性可以是字符串、数字或布尔型的 list。
RETURN matchedNode.list[0] AS value,
size(matchedNode.list) AS len
//list 的元素可以使用方括号中的 idx 下标访问。无效 index 返回 null 。可以从 start_idx 到 end_idx 间隔访问切片, start_idx 和 end_idx 可以省略或为负数。越界的元素将被忽略。
list[$idx] AS value,
list[$startIdx..$endIdx] AS slice
//使用 UNWIND 可以将 list 转换成单独的行。该示例匹配名称列表中的所有名称。
UNWIND $names AS name MATCH (n {name: name}) RETURN avg(n.age)
//模式理解可用于从匹配直接到 list 的自定义映射。
MATCH (a) RETURN [(a)-->(b) WHERE b.name = 'Bob' | b.age]
//从节点、关系和其他 map 值轻松构建 map 映射。
MATCH (person) RETURN person { .name, .age}
```



### 3.6 Map

```cypher
//在大括号中定义文本 map,就像属性 map 一样,可以使用 list。
{name: 'Alice', age: 38,address: {city: 'London', residential: true}}
//访问嵌套 map 的属性。
WITH {person: {name: 'Anne', age: 25}} AS p
RETURN p.person.name
//可以将 map 作为参数传递,并可以用作 map,也可以通过访问 key 使用。
MERGE (p:Person {name: $map.name})
ON CREATE SET p = $map
//节点和关系作为其数据 map 返回。
MATCH (matchedNode:Person)
RETURN matchedNode
//可以通过 key 访问 map。无效 key 会产生 error。
map.name, map.age, map.children[0]
```



### 3.7 CASE

```cypher
//从根据 WHEN 值的匹配情况返回 THEN 值。 ELSE 值可选,如果没有则为 null 。
CASE n.eyes
WHEN 'blue' THEN 1
WHEN 'brown' THEN 2
ELSE 3
END

//按顺序计算 WHEN 表达式,返回第一个 true 对应的 THEN 值。
CASE
WHEN n.eyes = 'blue' THEN 1
WHEN n.age < 40 THEN 2
ELSE 3
END

```



### 3.8 判断式

```cypher
//使用比较运算符。
n.property <> $value

//判断是否存在函数。
exists(n.property)

//使用布尔运算符组合判断式。
n.number >= 1 AND n.number <= 10

//使用链式运算符组合判断式。
1 <= n.number <= 10

//验证节点标签。
n:Person

//验证是否为 null 。
variable IS NULL

//属性不存在或判断式为 true 。
NOT exists(n.property) OR n.property = $value

//不存在的属性返回null ,不等于任何值。
n.property = $value

//也可以使用动态计算的属性名称来访问属性。
n["property"] = $value

//字符串匹配。
n.property STARTS WITH 'Tim' OR
n.property ENDS WITH 'n' OR
n.property CONTAINS 'goodie'

//正则表达式匹配。
n.property =~ 'Tim.*'

//确保模式至少有一个匹配项。
(n)-[:KNOWS]->(m)

//从结果中排除匹配 (n)-[:KNOWS]->(m) 。
NOT (n)-[:KNOWS]->(m)

//检查list中是否存在元素。
n.property IN [$value1, $value2]
```



### 3.9 list 判断式

```cypher
//list 中所有元素都使判断式为 true 时才返回 true。
all(x IN coll WHERE exists(x.property))

//list 中至少有一个元素使判断式为 true 时就返回 true 。
any(x IN coll WHERE exists(x.property))

//list 中所有元素都使判断式为 false 时才返回 true 。
none(x IN coll WHERE exists(x.property))

//list 中只有一个元素使判断式为 true 时就返回 true 。
single(x IN coll WHERE exists(x.property))

```





### 3.10 list 表达式

```cypher
//列表中的元素数。
size($list)

//逆序列表中元素的顺序。
reverse($list)

//head() 返回列表的第一个元素。 last() 返回最后一个元素。 tail() 返回除第一个元素外的所有元素。对于空list这些函数都返回null。
head($list), last($list), tail($list)

//对列表中每个元素计算表达式的值,形成一个新列表。
[x IN list | x.prop]

//用表达式是否为 true 来筛选list的元素,形成一个新列表。
[x IN list WHERE x.prop <> $value]

//列表理解功能,用于过滤列表并为该列表中的每个元素提取表达式的值。
[x IN list WHERE x.prop <> $value | x.prop]

//以 $start 初始化 s ,对列表中每个元素执行表达式,最终结果保存到 s 中。
reduce(s = $start, x IN list | s + x.prop)
```



## 4. 函数类

### 4.1 函数

```cypher
coalesce(n.property, $defaultValue)
第一个非 null 表达式。
timestamp()
UTC 1970 年 1 月 1 日午夜以来的毫秒数。
id(nodeOrRelationship)
关系或节点的内部 id。
toInteger($expr)
转为整数,不能转则返回 null 。
toFloat($expr)
转为浮点数,不能转则返回 null 。
toBoolean($expr)
转为布尔值,不能转则返回 null 。
keys($expr)
将节点、关系或 map 的属性名称以 list 形式返回。
properties($expr)
返回包含节点或关系的所有属性的 map。
```





### 4.2 Path 函数

```cypher
length(path)
路径中的关系数。
nodes(path)
路径中节点的 list。
relationships(path)
路径中关系的 list。
[x IN nodes(path) | x.prop]
从路径中节点提取属性。
```





### 4.3 Relationship 函数

```cypher
type(a_relationship)
关系的类型。
startNode(a_relationship) ,endNode(a_relationship)
关系的开始节点、结束节点。
id(a_relationship)
关系的内部 ID。
```



### 4.4 空间函数

```cypher
point({x: $x, y: $y})
返回二维笛卡尔坐标系中的点。
point({latitude: $y, longitude: $x})
返回 2D 地理坐标系中的点,坐标以十进制度表示。
point({x: $x, y: $y, z: $z})
返回 3D 直角坐标系中的点。
point({latitude: $y, longitude: $x, height: $z})
返回 3D 地理坐标系中的点,以十进制度表示的纬度和经度,以米表示的高度。
distance(point({x: $x1, y: $y1}), point({x: $x2, y: $y2}))
9Neo4j Cypher Refcard 4.2
返回一个浮点数,表示两个点之间的线性距离。返回的单位将与点坐标的单位相同,并
且适用于 2D 和 3D 笛卡尔点。
distance(point({latitude: $y1, longitude: $x1}), point({latitude: $y2, longitude:
$x2}))
返回两点之间的测地距离(以米为单位)。也可以用于 3D 地理点。
```





### 4.5 时间函数

```cypher
date("2018-04-05")
字符串转日期。
localtime("12:45:30.25")
字符串转时间,不带时区。
time("12:45:30.25+01:00")
字符串转时间,带时区。
localdatetime("2018-04-05T12:34:00")
字符串转日期时间,不带时区。
datetime("2018-04-05T12:34:00[Europe/Berlin]")
字符串转日期时间,带时区。
datetime({epochMillis: 3360000})
UNIX 时间戳转换为正常日期时间。
date({year: $year, month: $month, day: $day})
所有时间函数可以用 map 作为参数来调用,每个函数支持的 key 可能不同。
此示例根据 year , month 和 day 返回日期。
datetime({date: $date, time: $time})
可以通过其他类型的组合来创建日期时间。本示例由 date 和 time 创建一个 datetime 。
date({date: $datetime, day: 5})
可以从更复杂的类型中选择部分创建时间,各部分也可以重新赋值。
本示例从 datetime 中创建 date ,并对 day 重新赋值。
WITH date("2018-04-05") AS d
RETURN d.year, d.month, d.day, d.week, d.dayOfWeek
时间类型的各部分可以单独访问。
```



### 4.6 时长函数

```cypher
duration("P1Y2M10DT12H45M30.25S")
返回 duration:"P14M10DT45930.250000000S"
duration.between($date1,$date2)
返回两个时间之间的 duration。
WITH duration("P1Y2M10DT12H45M") AS d
RETURN d.years, d.months, d.days, d.hours, d.minutes
d.years d.months d.days d.hours d.minutes
1			14		10		12		765
WITH duration("P1Y2M10DT12H45M") AS d
RETURN d.years, d.monthsOfYear, d.days, d.hours, d.minutesOfHour
d.years d.monthsOfYear d.days d.hours d.minutesOfHour
1			2			10		12			45
date("2015-01-01") + duration("P1Y1M1D")
返回日期为 2016-02-02 。也可以减。
duration("PT30S") * 10
返回 5 分钟的 duration。也可以除。
```



### 4.7 数学函数

```cypher
abs($expr)
绝对值。
rand()
返回介于 0(含)到 1(不含)之间的随机数 [0,1) 。为每个调用返回一个新值。对于选择
子集或随机排序也很有用。
round($expr)
四舍五入到最接近的整数; ceil() 向上取整,
floor() 向下取整。
sqrt($expr)
平方根。
sign($expr)
零返回 0 ,负数返回 -1 ,正数返回 1 。
sin($expr)
三角函数还包括 cos() , tan() , cot() , asin() , acos() , atan() , atan2() ,和 haversin() 。
三角函数的所有参数都应以弧度表示,除非另有说明。
degrees($expr), radians($expr), pi()
将弧度转换为度; radians() 反向, pi() 为 π 。
log10($expr), log($expr), exp($expr), e()
以 10 为底的对数,自然对数, e 次幂, e 的值。
```



### 4.8 字符串函数

```cypher
toString($expression)
转字符串。
replace($original, $search, $replacement)
用 replacement 替换所有的 search 。所有参数必须是表达式。
substring($original, $begin, $subLength)
截取字符串。 subLength 可选。
left($original, $subLength), right($original, $subLength)
字符串的开头和结尾部分。
trim($original), lTrim($original),rTrim($original)
删除所有空格、开头空格、结尾空格。
toUpper($original), toLower($original)
转换大写和小写。
split($original, $delimiter)
字符串分割为 list。
reverse($original)
字符串反向。
size($string)
计算字符串中的字符数。
```



### 4.9 聚合函数

```cypher
count(*)
匹配的行数。
count(variable)
非 null 值的数量。
count(DISTINCT variable)
所有聚合函数都可以使用 DISTINCT 运算符删除重复项。
collect(n.property)
删除 null 后的 list。
sum(n.property)
数值总和。类似的功能 avg() , min() , max() 。
percentileDisc(n.property, $percentile)
离散百分比。连续百分比是 percentileCont() 。该 percentile 参数是从 0.0 到 1.0 。
stDev(n.property)
样本标准差。总体标准差使用 stDevP() 。
```



## 5. Schema

### 5.1 索引 INDEX

```cypher
CREATE INDEX FOR (p:Person) ON (p.name)
在 Person 标签和 name 属性上创建索引 。
CREATE INDEX index_name FOR (p:Person) ON (p.age)
在 Person 标签和 name 属性上创建名为 index_name 的索引 。
CREATE INDEX FOR (p:Person) ON (p.surname)
OPTIONS {indexProvider: 'native-btree-1.0', indexConfig:
{`spatial.cartesian.min`: [-100.0, -100.0],
`spatial.cartesian.max`: [100.0, 100.0]}}
使用 native-btree-1.0 和给定的 spatial.cartesian 设置在标签 Person 和属性 surname 上创建索
引。索引的其他设置取默认值。
CREATE INDEX FOR (p:Person) ON (p.name, p.age)
在标签 Person 和属性 name 及 age 上创建复合索引,如果索引已经存在抛出一个错误。
CREATE INDEX IF NOT EXISTS FOR (p:Person) ON (p.name,p.age)
在标签 Person 和属性 name 及 age 上创建复合索引,如果索引已经存在,不做任何处理。
SHOW INDEXES
列出所有索引。
MATCH (n:Person) WHERE n.name = $value
相等性比较中自动使用索引。注意,例如 toLower(n.name)
= $value 将不使用索引。
MATCH (n:Person) WHERE n.name IN [$value]
list 的 IN 检查中自动使用索引。
MATCH (n:Person) WHERE n.name = $value and n.age = $value2
两个属性的相等比较中自动使用他们的复合索引。复合索引的所有属性都要出现在判断
式中。
MATCH (n:Person)
USING INDEX n:Person(name)
WHERE n.name = $value
当 Cypher 使用次优索引或应使用多个索引时,可以强制使用索引。
DROP INDEX index_name
删除名为 index_name 的索引,如果该索引不存在,则会引发错误。
DROP INDEX index_name IF EXISTS
删除名为 index_name 的索引,如果不存在则不执行任何操作。
```



### 5.2 约束 CONSTRAINT

```cypher
CREATE CONSTRAINT ON (p:Person) ASSERT p.name IS UNIQUE
在标签 Person 和属性 name 上创建唯一属性约束。如果使用该标签的其他节点更新或创建时
用到属性 name ,则写操作将失败。此约束将创建一个对应的索引。
CREATE CONSTRAINT uniqueness ON (p:Person) ASSERT p.age IS UNIQUE
在标签 Person 和属性 age 上创建唯一属性约束,名为 uniqueness 。如果该标签的其他节点更
新或创建时用到属性 age ,则写操作失败。此约束将创建一个对应的索引。
CREATE CONSTRAINT ON (p:Person) ASSERT p.surname IS UNIQUE
OPTIONS {indexProvider: 'native-btree-1.0'}
在标签 Person 和属性 surname 上创建唯一属性约束,使用索引程序 native-btree-1.0 创建对应
的索引。
CREATE CONSTRAINT ON (p:Person) ASSERT exists(p.name)
(★)在标签 Person 和属性 name 上创建节点属性存在约束,如果该约束已经存在,则会引
发错误。如果创建 Person 节点时不带 name ,或者从现有的 Person 节点中删除 name 属性,则
操作失败。
CREATE CONSTRAINT node_exists IF NOT EXISTS ON (p:Person) ASSERT exists(p.name)
(★)在标签 Person 和属性 name 上创建节点属性存在约束,名为 node_exists。 如果约束已
经存在,则什么也不会发生。
CREATE CONSTRAINT ON ()-[l:LIKED]-() ASSERT exists(l.when)
(★)在类型 LIKED 和属性 when 上创建关系属性存在约束。如果创建 LIKED 关系时不带 when ,
或者从已有的 LIKED 关系中删除 when 属性,则操作失败。
CREATE CONSTRAINT relationship_exists ON ()-[l:LIKED]-() ASSERT exists(l.since)
(★)在类型 LIKED 和属性 since 上创建关系属性存在约束,名为 relationship_exists 。如果
创建 LIKED 关系时不带 since ,或者从已有的 LIKED 关系中删除 since 属性,则操作失败。
SHOW UNIQUE CONSTRAINTS VERBOSE
列出所有唯一约束。
CREATE CONSTRAINT ON (p:Person)
ASSERT (p.firstname, p.surname) IS NODE KEY
(★)在标签 Person 和属性 firstname、surname 上创建节点键约束。如果创建 Person 节点时
没有 firstname 和 surname ,或者两者的组合不是唯一的,或者修改现有 Person 节点上的
firstname 和/或 surname 时违反了约束,则操作失败。
CREATE CONSTRAINT node_key ON (p:Person)
ASSERT (p.name, p.surname) IS NODE KEY
(★)在标签 Person 、属性 name 和 surname 上创建节点键约束 node_key 。如果创建 Person 节
点时没有 name 和 surname ,或者两者的组合不是唯一的,或者修改现有 Person 节点上的 name
和/或 surname 时违反了约束,则操作失败。
CREATE CONSTRAINT node_key_with_config ON (p:Person)
ASSERT (p.name, p.age) IS NODE KEY
OPTIONS {indexConfig: {`spatial.wgs-84.min`:[-100.0, -100.0],
`spatial.wgs-84.max`: [100.0, 100.0]
}
}
(★)在标签 Person 、属性 name 和 age 上创建节点键约束,名为 node_key_with_config ,并指
定了对应索引的 spatial.wgs-84 设置。索引的其他设置使用默认值。
DROP CONSTRAINT uniqueness
删除名为 uniqueness 的约束,如果约束不存在,则会引发错误。
DROP CONSTRAINT uniqueness IF EXISTS
删除名为 uniqueness 的约束,如果约束不存在,则不执行任何操作。
```



## 6. Performance

### 6.1 性能

- 尽量使用参数而不是标称量。这样 cyper 可以重用查询,而不必解析和构建新的执行
  任务。

- 一定要为可变长度的模式设置上限。不小心包含了图中所有节点的查询会让人发疯。

- 仅返回所需要的数据。避免返回所有节点和关系。

- 使用 PROFILE / EXPLAIN 分析查询的性能。更多信息请参阅 Manual 的性能调优章节。



  ## 7. MultiDatabase

###   7.1 数据库管理

 ```cypher
 CREATE OR REPLACE DATABASE myDatabase
  (★)创建一个名为的数据库 myDatabase 。如果存在,则将删除并新建。
  STOP DATABASE myDatabase
  (★)停止数据库 myDatabase 。
  START DATABASE myDatabase
  (★)启动数据库 myDatabase 。
  SHOW DATABASES
  列出所有数据库以及信息。
  SHOW DATABASE myDatabase
  列出数据库 myDatabase 的信息。
  SHOW DEFAULT DATABASE
  列出默认数据库的信息。
  DROP DATABASE myDatabase IF EXISTS
  (★)删除数据库 myDatabase (如果存在)。
 ```



##   8.Security

###   8.1 用户管理

  ```cypher
CREATE USER alice SET PASSWORD $password
  创建新用户和密码。首次登录时必须更改此密码。
  ALTER USER alice SET PASSWORD $password CHANGE NOT REQUIRED
  为用户设置新密码。下次登录时,不需要该用户更改此密码。
  ALTER USER alice SET STATUS SUSPENDED
  (★)将用户状态更改为 SUSPENDED 。使用 SET
  STATUS ACTIVE 重新激活用户。
  ALTER CURRENT USER SET PASSWORD FROM $old TO $new
  修改已登录用户的密码。下次登录时用户不需要再更改此密码。
  SHOW CURRENT USER
  列出当前登录用户、状态、角色以及是否需要更改密码。 (★)状态和角色仅限企业版。
  SHOW USERS
  列出系统中的所有用户、状态、角色以及是否需要更改密码。 (★)状态和角色仅限企业版。
  DROP USER alice
  删除用户。
  ```



### 8.2   (★)角色管理

```cypher
  CREATE ROLE my_role
  创建一个角色。
  CREATE ROLE my_second_role IF NOT EXISTS AS COPY OF my_role
  创建角色 my_role 的副本 my_second_role (除非已存在)。
  GRANT ROLE my_role, my_second_role TO alice
  向用户分配角色。
  REVOKE ROLE my_second_role FROM alice
  从指定角色中删除用户。
  SHOW ROLES
  列出系统中的所有角色。
  SHOW POPULATED ROLES WITH USERS
  列出至少有一个用户的所有角色,以及这些角色的用户。
  DROP ROLE my_role
  删除角色。
```





###  8.3  (★)图的读权限

```cypher
  GRANT TRAVERSE ON GRAPH * NODES * TO my_role
  将所有节点和图的 traverse 权限授予角色。
  DENY READ {prop} ON GRAPH foo RELATIONSHIP Type TO my_role
  禁止角色 my_role 对图 foo 中类型 type 的关系某些属性 {prop} 的 read 权限。
  GRANT MATCH {*} ON DEFAULT GRAPH ELEMENTS Label TO my_role
  将默认图中所有属性的 read 权限和 traverse 权限授予角色。两种特权都适用于图中具有指
  定标签/类型的所有节点和关系。
```





###   8.4 (★)图的写权限

  ```cypher
GRANT CREATE ON GRAPH * NODES Label TO my_role
  授予角色所有图中具有指定标签的所有节点的 create 权限。
  DENY DELETE ON GRAPH neo4j TO my_role
  指定图中所有节点和关系上的禁止 delete 的权限。
  REVOKE SET LABEL Label ON GRAPH * FROM my_role
  撤销角色所有图上指定标签的 set
  label 权限。
  GRANT REMOVE LABEL * ON GRAPH foo TO my_role
  授予角色指定图上所有标签的 remove
  label 权限。
  DENY SET PROPERTY {prop} ON GRAPH foo RELATIONSHIPS Type TO my_role
  禁止角色 my_role 对图 foo 中 type 类型的关系的某些属性 {prop} 的 set 权限。
  GRANT MERGE {*} ON GRAPH * NODES Label TO my_role
  授予角色所有图中具有 Label 标签的节点的 merge 权限。
  REVOKE WRITE ON GRAPH * FROM my_role
  撤消角色对所有图的 write 权限。
  DENY ALL GRAPH PRIVILEGES ON GRAPH foo TO my_role
  拒绝角色在图 foo 上的所有权限。

  ```



###   8.5 (★)权限的查看

```cypher
  SHOW PRIVILEGES
  列出系统中的所有权限及其分配的角色。
  SHOW ROLE my_role PRIVILEGES
  列出角色的权限。
  SHOW ROLE my_role, my_second_role PRIVILEGES
  列出多个角色的权限。
  SHOW USER alice PRIVILEGES
  列出用户 alice 的所有权限及角色。
  15Neo4j Cypher Refcard 4.2
  SHOW USER PRIVILEGES
  列出当前登录用户的所有权限及角色。
  SHOW PRIVILEGES AS COMMANDS
  以 Cypher 命令列出系统中的所有权限。
```





###   8.6 (★)数据库权限

 ```cypher
 GRANT ACCESS ON DATABASE * TO my_role
  向角色授予所有数据库的访问权限。
  GRANT START ON DATABASE * TO my_role
  向角色授予所有数据库的启动权限。
  GRANT STOP ON DATABASE * TO my_role
  向角色授予所有数据库的停止权限。
  GRANT CREATE INDEX ON DATABASE foo TO my_role
  向角色授予数据库 foo 上创建索引的权限。
  GRANT DROP INDEX ON DATABASE foo TO my_role
  向角色授予数据库 foo 上删除索引的权限。
  GRANT SHOW INDEX ON DATABASE * TO my_role
  向角色授予所有数据库上查看索引的权限。
  DENY INDEX MANAGEMENT ON DATABASE bar TO my_role
  禁止角色在数据库 bar 上创建和删除索引的权限。
  GRANT CREATE CONSTRAINT ON DATABASE * TO my_role
  向角色授予所有数据库上创建约束的权限。
  DENY DROP CONSTRAINT ON DATABASE * TO my_role
  禁止角色在所有数据库上创建约束的权限。
  DENY SHOW CONSTRAINT ON DATABASE foo TO my_role
  禁止角色在数据库 foo 上查看约束的权限。
  REVOKE CONSTRAINT ON DATABASE * FROM my_role
  撤销角色已授予和禁止的在所有数据库上创建和删除约束的权限。
  GRANT CREATE NEW LABELS ON DATABASE * TO my_role
  向角色授予所有数据库上创建新标签的权限。
  DENY CREATE NEW TYPES ON DATABASE foo TO my_role
  禁止角色在数据库 foo 上创建新类型的权限。
  REVOKE GRANT CREATE NEW PROPERTY NAMES ON DATABASE bar FROM my_role
  撤销角色已授予的在数据库 bar 上创建新属性名称的权限。
  GRANT NAME MANAGEMENT ON DEFAULT DATABASE TO my_role
  向角色授予默认数据库上创建标签、关系类型和属性名称的权限。
  GRANT ALL ON DATABASE baz TO my_role
  向角色授予数据库 baz 上访问、创建和删除索引/约束、创建新的标签、类型和属性名称
  的权限。
  GRANT SHOW TRANSACTION (*) ON DATABASE foo TO my_role
  向角色授予查看所有用户在数据库 foo 上的事务和查询的权限。
  DENY TERMINATE TRANSACTION (user1, user2) ON DATABASES * TO my_role
  禁止角色在所有数据库上终止用户 user1 和 user2 的事务的权限。
  REVOKE GRANT TRANSACTION MANAGEMENT ON DEFAULT DATABASE
  撤销角色在默认数据库上被授予的列出和终止所有用户的事务和查询的权限。
 ```





###  8.7  (★)角色管理权限

```cypher
  GRANT CREATE ROLE ON DBMS TO my_role
  向角色授予创建角色的权限。
  GRANT DROP ROLE ON DBMS TO my_role
  向角色授予删除角色的权限。
  16Neo4j Cypher Refcard 4.2
  DENY ASSIGN ROLE ON DBMS TO my_role
  禁止角色给用户分配角色的权限。
  DENY REMOVE ROLE ON DBMS TO my_role
  禁止角色给用户撤销角色的权限。
  REVOKE DENY SHOW ROLE ON DBMS FROM my_role
  撤消角色的禁止查询角色的权限。
  GRANT ROLE MANAGEMENT ON DBMS TO my_role
  向角色授予角色管理的所有权限。
```



###   8.8 (★)用户管理权限

```cypher
  GRANT CREATE USER ON DBMS TO my_role
  向角色授予创建用户的权限。
  GRANT DROP USER ON DBMS TO my_role
  向角色授予删除用户的权限。
  DENY ALTER USER ON DBMS TO my_role
  禁止角色变更用户的权限。
  REVOKE SET PASSWORDS ON DBMS FROM my_role
  撤消角色更改用户密码的权限。
  REVOKE GRANT SET USER STATUS ON DBMS FROM my_role
  撤消角色已被授予的更改用户状态的权限。
  REVOKE DENY SHOW USER ON DBMS FROM my_role
  撤消角色已被禁止的显示用户的权限。
  GRANT USER MANAGEMENT ON DBMS TO my_role
  向角色授予用户管理的所有权限。
```



### 8.9  (★)数据库管理权限

```cypher
  GRANT CREATE DATABASE ON DBMS TO my_role
  向角色授予创建数据库的权限。
  REVOKE DENY DROP DATABASE ON DBMS FROM my_role
  撤消角色被禁止的删除数据库的权限。
  DENY DATABASE MANAGEMENT ON DBMS TO my_role
  禁止角色所有数据库管理权限。
```



###  8.10  (★)权限管理的权限

```cypher
  GRANT SHOW PRIVILEGE ON DBMS TO my_role
  向角色授予查看权限的权限。
  DENY ASSIGN PRIVILEGE ON DBMS TO my_role
  禁止角色的分配权限的权限。
  REVOKE GRANT REMOVE PRIVILEGE ON DBMS FROM my_role
  撤消角色被授予的撤销权限的权限。
  REVOKE PRIVILEGE MANAGEMENT ON DBMS FROM my_role
  撤消角色的所有权限管理的权限。
```





###   8.11 (★)DBMS 权限

```cypher
  GRANT ALL ON DBMS TO my_role
  向角色授予所有角色管理、用户管理、数据库管理和权限管理的权限。
```

