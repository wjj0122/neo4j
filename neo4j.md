记录使用Neo4j存储知识图谱的一些方法
- - -
### Neo4j的安装与启动

**Neo4j的安装**

下载之前需要确保本机上有java se的sdk。从 [官网](https://neo4j.com/download-center/#community)下载Neo4j社区版，本次下载的是最新版5.1.0版本，下载后直接解压即可。记得添加环境变量`NEO4J_HOME=path`,`path`为Neo4j文件夹路径，再在path中添加`%NEO4J_HOME%\bin`。
文件夹主要有以下几个目录：
- bin目录：存储Neo4j的可执行程序
- conf目录：控制Neo4j启动的配置文件
- data目录：存储核心数据库文件
- plugins目录：存储Neo4j的插件
- import目录：**可以导入csv文件**

**Neo4j的启动**

我使用的方法是以管理员身份运行CMD，并输入`neo4j.bat console`，不要关闭终端，并在浏览器中打开`http://localhost:7474`，即可进入Neo4j界面。默认用户名和密码是neo4j。

**Neo4j数据库的切换**

社区版Neo4j不能在Neo4j界面直接新建数据库并切换，可以采取的方法是：
在conf文件夹中编辑neo4j.conf文件，找到`# dbms.active_database=neo4j`这一行，在这一行下面输入`dbms.active_database=自己想要的数据库名`，再重启Neo4j即可使用新的数据库。换回原来的库只需要将名字改回去即可。

- - -
### CQL语句

CQL代表Cypher语言，是Neo4j图形数据库的查询语言，遵循SQL语法。
常用的**CQL命令**有：
|序号|CQL命令|用法|
|:---:|:---:|:---:|
1|create|创建结点，关系和属性
2|match|检索有关结点、关系和属性数据
3|return|返回查询结果
4|where|提供条件过滤检索数据
5|delete|删除节点和关系
6|remove|删除结点和关系的属性
7|order by|排序检索数据
8|set|添加或更新标签

**1.create命令**
```
CREATE (
   <node-name>:<label-name>
   { 	
      <Property1-name>:<Property1-Value>
      ........
      <Propertyn-name>:<Propertyn-Value>
   }
)
```
`node-name`是节点的假设名称，`label-name`是节点的标签名称即类别，花括号中的`property-name`为属性名，`value`为其值。
示例：`CREATE (dept:Dept { deptno:10,dname:"Accounting",location:"Hyderabad" })`

**2.match命令**
```
MATCH 
(
   <node-name>:<label-name>
)
```
match命令不能单独使用，一般配合return等命令使用，如`match(n) return n`
```
# 查询Dept下的内容
MATCH (dept:Dept) return dept

# 查询Employee标签下 id=123，name="Lokesh"的节点
MATCH (p:Employee {id:123,name:"Lokesh"}) RETURN p

## 查询Employee标签下name="Lokesh"的节点，使用（where命令）
MATCH (p:Employee)
WHERE p.name = "Lokesh"
RETURN p
```

**3.return命令**
```
RETURN 
   <node-name>.<property1-name>,
   ........
   <node-name>.<propertyn-name>
```
不能单独使用，配合`match`,`create`等命令使用

**4.关系**

- 已有节点，添加关系
  ```
  MATCH (<node1-name>:<node1-label-name>),(<node2-name>:<node2-label-name>)
  CREATE  
	(<node1-name>)-[<relationship-name>:<relationship-label-name>
	{<define-properties-list>}]->(<node2-name>)
  RETURN <relationship-name>
  ```
  `node-name`一般写为`from`,`to`即可，`node-label-name`为对应节点的标签即类别
- 同时创建节点和关系
  ```
  CREATE  
	(<node1-name>:<node1-label-name>{<define-properties-list>})-
	[<relationship-name>:<relationship-label-name>{<define-properties-list>}]
	->(<node1-name>:<node1-label-name>{<define-properties-list>})
  RETURN <relationship-name>
  ```
- `properties-list`一般为：
  ```
  { 
	<property1-name>:<property1-value>,
	<property2-name>:<property2-value>,
	...
	<propertyn-name>:<propertyn-value>
  }
  ```

**5.where命令**
```
WHERE <condition> <boolean-operator> <condition>
```
其中，`condition`为：
```
<property-name> <comparison-operator> <value>
```
`boolean-operator`有：
|序号|运算符|描述|
|:---:|:---:|:---:|
1|and|和
2|or|或
3|not|非
4|xor|异或
`operater`有`=,<>,<,>,<=,>=,in`
`in`的用法:`in[<Collection-of-values>]`

**6.delete命令**
删除节点或者关系，一般与`match`配合使用
```
MATCH (e: Employee) RETURN e

MATCH (cc: CreditCard)-[rel]-(c:Customer) 
DELETE cc,c,rel
```

**7.remove命令**
删除现有结点或者关系的标签或者属性，与`match`等配合使用
```
MATCH (dc:DebitCard) 
REMOVE dc.cvv
RETURN dc
```

**8.set命令**
向现有关系或者节点添加新的属性或者更新属性
```
# 将Book类别的节点添加新的属性title，值为superstar
MATCH (book:Book)
SET book.title = 'superstar'
RETURN book

# 改变某个属性的数据类型为int型
MATCH (book:Book)
SET book.id = toInteger(book.id)
RETURN book
```

**9.order by命令**
按属性值进行排序，默认为升序
`ORDER BY  <property-name-list>  [DESC]`，`DESC表示降序`
```
MATCH (emp:Employee)
RETURN emp.empid,emp.name,emp.salary,emp.deptno
ORDER BY emp.name
```

**10.union和union all命令**
`union`将两组结果中的公共行数据组合并返回到一组结果中，不返回重复的行
```
<MATCH Command1>
   UNION
<MATCH Command2>
```
两个`match`查询应该具有相同的列名，不同时可以使用`as`
```
MATCH (cc:CreditCard)
RETURN cc.id as id,cc.number as number,cc.name as name,
   cc.valid_from as valid_from,cc.valid_to as valid_to
UNION
MATCH (dc:DebitCard)
RETURN dc.id as id,dc.number as number,dc.name as name,
   dc.valid_from as valid_from,dc.valid_to as valid_to
```
`union all`会返回重复的行

**11.limit和skip命令**
使用`limit`命令来控制查询返回的行数，`skip`命令用来从顶部跳过一些行
```
# 只返回最顶部的2个结果
MATCH (emp:Employee) 
RETURN emp
LIMIT 2

# 跳过顶部的2个
MATCH (emp:Employee) 
RETURN emp
SKIP 2
```

**12.merge命令**
`merge`命令是`create`和`match`的组合，`MERGE`命令在图中搜索给定模式，如果存在，则返回结果，如果它不存在于图中，则它创建新的节点/关系并返回结果。**比`create`更安全**。
```
MERGE (<node-name>:<label-name>
{
   <Property1-name>:<Pro<rty1-Value>
   .....
   <Propertyn-name>:<Propertyn-Value>
})
```

**13.从csv文件导入节点和关系**

先将文件放到import文件夹下！
- 导入节点
  ```
  # filename为节点文件的文件名，line.gid和line.gname的gid和gname为csv文件中表头的名称
  LOAD CSV WITH HEADERS  FROM "file:///finename.csv" AS line
  MERGE (p:Genre{gid:toInteger(line.gid),name:line.gname})
  ```
- 导入关系
  ```
  # Person和Movie为节点的类型名称，将他们的属性（pid和mid）与文件中的属性对齐匹配，actedin为关系的标签
  LOAD CSV WITH HEADERS FROM "file:///person_to_movie.csv" AS line 
  match (from:Person{pid:toInteger(line.pid)}),(to:Movie{mid:toInteger(line.mid)})  
  merge (from)-[r:actedin{pid:toInteger(line.pid),mid:toInteger(line.mid)}]->(to)
  ```






