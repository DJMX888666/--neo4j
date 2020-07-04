# py2neo——Neo4j&python的配合使用
## 通过python面向Neo4j的库py2neo来对Neo4j进行一些简单的操作，包括：
- 连接Neo4j数据库 
- 节点的建立 
- 节点之间关系的建立 
- 关系属性赋值以及属性值的更新
- 通过属性值查找节点/关系
- 通过节点/关系查找相关联的节点/关系


## 连接Neo4j数据库
要通过python来操作Neo4j，首先需要安装py2neo，可以直接使用pip安装。

`pip install py2neo`

在完成安装之后，在python中调用py2neo即可，常用的有 Graph,Node,Relationship。

`from py2neo import Graph,Node,Relationship`

连接Neo4j的方法很简单：
```python
test_graph = Graph(
    "http://localhost:7474",
    username="neo4j",
    password="neo4j"
)
```
test_graph就是我们建立好的Neo4j的连接。  
Neo4j的服务器装好了之后，默认的端口号就是7474,所以本地的主机就是"http://localhost:7474" 。  
默认的用户名密码都是neo4j，不过也可以在浏览器中进入http://localhost:7474 ，首次进入会提示你进行密码修改。  
## 节点的建立
节点的建立要用到py2neo.Node，建立节点的时候要定义它的节点类型（label）以及一个基本属性（property，包括property_key和property_value）。  
以下代码为建立了两个测试节点。  
```python
test_node_1 = Node(label = "Person",name = "test_node_1")
test_node_2 = Node(label = "Person",name = "test_node_2")
test_graph.create(test_node_1)
test_graph.create(test_node_2)
```

这两个节点的类型(label)都是Person，而且都具有属性（property_key）为name，属性值（property_value）分别为"test_node_1"，"test_node_2"。
## 节点间关系的建立
节点间的关系（Relationship）是有向的，所以在建立关系的时候，必须定义一个起始节点和一个结束节点。值得注意的是，起始节点可以和结束节点是同一个点，这时候的关系就是这个点指向它自己。
```python
node_1_call_node_2 = Relationship(test_node_1,'CALL',test_node_2)
node_1_call_node_2['count'] = 1
node_2_call_node_1 = Relationship(test_node_2,'CALL',test_node_1)
node_2_call_node_1['count'] = 2
test_graph.create(node_1_call_node_2)
test_graph.create(node_2_call_node_1)
```

如以上代码，分别建立了test_node_1指向test_node_2和test_node_2指向test_node_1两条关系，关系的类型为"CALL"，两条关系都有属性count，且值为1。  
在这里有必要提一下，如果建立关系的时候，起始节点或者结束节点不存在，则在建立关系的同时建立这个节点。  
## 节点/关系的属性赋值以及属性值的更新
节点和关系的属性初始赋值在前面节点和关系的建立的时候已经有了相应的代码，在这里主要讲述一下怎么更新一个节点/关系的属性值。  
我们以关系建立里的 node_1_call_node_2 为例，让它的count加1，再更新到图数据库里面。  
```python
node_1_call_node_2['count']+=1
test_graph.push(node_1_call_node_2)
```

更新属性值就使用push函数来进行更新即可。  
## 通过属性值来查找节点和关系（find,find_one）
通过find和find_one函数，可以根据类型和属性、属性值来查找节点和关系。
示例如下：  
```python
find_code_1 = test_graph.find_one(
  label="Person",
  property_key="name",
  property_value="test_node_1"
)
print find_code_1['name']
```

find和find_one的区别在于：  
find_one的返回结果是一个具体的节点/关系，可以直接查看它的属性和值。如果没有这个节点/关系，返回None。  
find查找的结果是一个游标，可以通过循环取到所找到的所有节点/关系。  
## 通过节点/关系查找相关联的节点/关系
如果已经确定了一个节点或者关系，想找到和它相关的关系和节点，就可以使用match和match_one。  

`find_relationship = test_graph.match_one(start_node=find_code_1,end_node=find_code_3,bidirectional=False)print find_relationship`

如以上代码所示，match和match_one的参数包括start_node,Relationship，end_node中的至少一个。  
bidirectional参数的意义是指关系是否可以双向。  
如果为False,则起始节点必须为start_node，结束节点必须为end_node。如果有Relationship参数，则一定按照Relationship对应的方向。  
如果为True，则不需要关心方向问题，会把两个方向的数据都返回。  
```python
match_relation  = test_graph.match(start_node=find_code_1,bidirectional=True)for i in match_relation:
    print(i)
    i['count']+=1
    test_graph.push(i)
```

如以上代码所示，查找和find_code_1相关的关系。  
match里面的参数只写了start_node，bidirectional的值为True，则不会考虑方向问题，返回的是以find_code_1为起始节点和结束节点的所有关联关系。  
如果,bidirectional的值为False，则只会返回以find_code_1为起始节点的所有关联关系。  










