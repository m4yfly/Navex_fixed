# python-joern-fix
==

#todo fix two steps


start(it, [], 0, 'sql', false, queryMapList)


findSinkLocation(m, warnmessage, 'sql', queryMapList, it)


# test steps


```
//g.v(6133).out.loop(1){it.loops < 15 && it.object.id <= 6200}{true}.containsLowSource().parents().children()
// it.object.code=='_REQUEST' || it.object.code=='_POST' || it.object.code=='_GET'
//g.v(6148).resolveCallArgsNew()
g.V[6000..6200].containsLowSource()
//.resolveCallArgsNew()

('result:', [<Node graph=u'http://localhost:7474/db/data/' ref=u'node/6140' labels=set([u'AST']) properties={u'childnum': 0, u'funcid': 6133, u'type': u'AST_DIM', u'id': 6140, u'lineno': 3}>, <Node graph=u'http://localhost:7474/db/data/' ref=u'node/6148' labels=set([u'AST']) properties={u'childnum': 1, u'funcid': 6133, u'type': u'AST_DIM', u'id': 6148, u'lineno': 5}>, <Node graph=u'http://localhost:7474/db/data/' ref=u'node/6149' labels=set([u'AST']) properties={u'childnum': 0, u'funcid': 6133, u'type': u'AST_VAR', u'id': 6149, u'lineno': 5}>])
<Node graph=u'http://localhost:7474/db/data/' ref=u'node/6140' labels=set([u'AST']) properties={u'childnum': 0, u'funcid': 6133, u'type': u'AST_DIM', u'id': 6140, u'lineno': 3}>
<Node graph=u'http://localhost:7474/db/data/' ref=u'node/6148' labels=set([u'AST']) properties={u'childnum': 1, u'funcid': 6133, u'type': u'AST_DIM', u'id': 6148, u'lineno': 5}>
<Node graph=u'http://localhost:7474/db/data/' ref=u'node/6149' labels=set([u'AST']) properties={u'childnum': 0, u'funcid': 6133, u'type': u'AST_VAR', u'id': 6149, u'lineno': 5}>
```


```
//g.v(6133).out.loop(1){it.loops < 15 && it.object.id <= 6200}{true}.containsLowSource().parents().children()
// it.object.code=='_REQUEST' || it.object.code=='_POST' || it.object.code=='_GET'
//g.v(6148).resolveCallArgsNew()
g.V[6000..6200].containsLowSource().resolveCallArgsNew()

('result:', [u'$_REQUEST[Submit]', u'$_REQUEST[id]', u'$_REQUEST'])
u'$_REQUEST[Submit]'
u'$_REQUEST[id]'
u'$_REQUEST'
```

```
g.V().filter{sql_query_funcs.contains(it.code)  && isCallExpression(it.nameToCall().next()) && it.id==6166}
.callexpressions()
.callToArguments().resolveCallArgsNew()

result:
u'$query'
u'$GLOBALS[___mysqli_ston]'

```


```
g.v(6152).bothE("PARENT_OF","REACHES")//.filter{it.type == "PARENT_OF"}

result:
<Relationship graph=u'http://localhost:7474/db/data/' ref=u'relationship/6139' start=u'node/6152' end=u'node/6155' type=u'PARENT_OF' properties={}>
<Relationship graph=u'http://localhost:7474/db/data/' ref=u'relationship/6134' start=u'node/6152' end=u'node/6153' type=u'PARENT_OF' properties={}>
<Relationship graph=u'http://localhost:7474/db/data/' ref=u'relationship/123608' start=u'node/6152' end=u'node/6160' type=u'REACHES' properties={u'var': u'query'}>
<Relationship graph=u'http://localhost:7474/db/data/' ref=u'relationship/6140' start=u'node/6144' end=u'node/6152' type=u'PARENT_OF' properties={}>
```


```
filter shi xiao wen ti ,zui hao shi yong has()
g.v(1).outE.has('label','created')
//g.v(1).outE.filter{it.label=='created'}
```


```
get source 
g.V().filter{sql_query_funcs.contains(it.code)  && isCallExpression(it.nameToCall().next()) }.as('x')
.callexpressions().sources().sources().rval().resolveArgsNew2().as('y').select(){it.code}{it}
```

 
find sql or command  injection

loops < 10 , no check repair 

```
g.V().filter{sql_query_funcs.contains(it.code)  && isCallExpression(it.nameToCall().next()) }.as('sink')
.callexpressions().sources().loop(1){it.loops < 10 }{it.object.containsLowSource().toList() != []}
.rval().resolveArgsNew2().as('source').select(){it.code}{it}

```


```
in filter{}
it.xxx() maybe return a pipe object, so
you can use
it.xxx().next()
to get one node
```

loop 
https://github.com/tinkerpop/gremlin/wiki/Loop-Pattern

```
a = g.v(1).out().out().loop(2){false}.toList()
b = g.v(1).as('x').out().out().loop('x'){false}.toList()
a == b

result:
true
```

get AST_CALL name 
```
g.V().filter{isCallExpression(it)}.transform{getFuncName(it)}
```

get first call and it's name to filter repair
```
Gremlin.defineStep('calls', [Vertex, Pipe], {
    _().ifThenElse { isCallExpression(it) }
    { it }
            { it.children().loop(1) { !isCallExpression(it.object) } }
});

g.V().filter{ it.type == "AST_ASSIGN"}.calls().dedup().transform{getFuncName(it)}.toList()
```

get all calls and it's name to filter repair
```
g.V().filter{ it.type == "AST_ASSIGN"}.children().loop(1){it.object != null }{ isCallExpression(it.object) }.transform{getFuncName(it)}
```


we can get all calls and their arguments's name, link to Dataflow Edge's var, we can filter the repair function
```
// g.V().filter{ it.type == "AST_ASSIGN"} equals to  g.V().sources()

g.V().filter{ it.type == "AST_ASSIGN"}.children().loop(1){it.object != null }{ isCallExpression(it.object) }
.transform{ [it.id, getFuncName(it), it.callToArguments().varToName()] }

result:
('len:', 1752)
[140, u'dvwaPageNewGrab', []]
[326, u'file', []]
[348, u'explode', [u'line']]
[358, u'str_replace', [u'line']]
[383, u'urldecode', []]
[394, u'urldecode', []]
[463, u'fopen', []]
.....
[122089, u'mysqli_real_escape_string', [u'user']]
[122100, u'trigger_error', []]
[122081, u'is_object', []]
[122120, u'stripslashes', [u'pass']]
[122144, u'mysqli_real_escape_string', [u'pass']]
[122155, u'trigger_error', []]
[122136, u'is_object', []]
[122168, u'md5', [u'pass']]
[122188, u'mysqli_query', [u'query']]
[122239, u'mysqli_query', [u'query']]
[122274, u'mysqli_connect_error', []]
[122335, u'messagesPopAllToHtml', []]
```

some source node have two or more DATA_FLOW_EDGE
```
g.V().filter{it.in(DATA_FLOW_EDGE).dedup().toList().size() >=2 }.transform{it.inE(DATA_FLOW_EDGE).dedup().var}
```

```
groovy has function def cache, i don't kown why must add/del "return" to active modified function
```


set intersect -- get edge of two nodes
```
g.v(6166).sources().sideEffect{ prev_source = it }.sources().transform{ prev_source.inE('FLOWS_TO').id.toList().intersect(it.outE('FLOWS_TO').id.toList())}
```


use `.match{}` we can get all children nodes that type you want

Example: get all `AST_VAR` children
```
g.v(6166).sources().match{ it.type == 'AST_VAR' }.varToName()
```


sideEffect can be treat as a step , should use loop(1)
```
g.v(1).sideEffect{a = a+1}.out().sideEffect{b = b+1}.loop(1){it.loops <= 5}//.transform{[a, b]}

result:
('len:', 1)
(n2:AST {endlineno:10,flags:["TOPLEVEL_FILE"],id:2,lineno:1,name:"../php_src/dvwa/phpinfo.php",type:"AST_TOPLEVEL"})
```


```
[[g.v(1).outE().inV(),g.v(1).out()], [g.v(1).inE().outV(), g.v(1).in()]]

result:
('len:', 2)
[[<Node graph=u'http://localhost:7474/db/data/' ref=u'node/2' labels=set([u'AST']) properties={u'name': u'../php_src/dvwa/phpinfo.php', u'endlineno': 10, u'flags': [u'TOPLEVEL_FILE'], u'lineno': 1, u'type': u'AST_TOPLEVEL', u'id': 2}>],
 [<Node graph=u'http://localhost:7474/db/data/' ref=u'node/2' labels=set([u'AST']) properties={u'name': u'../php_src/dvwa/phpinfo.php', u'endlineno': 10, u'flags': [u'TOPLEVEL_FILE'], u'lineno': 1, u'type': u'AST_TOPLEVEL', u'id': 2}>]]
[[<Node graph=u'http://localhost:7474/db/data/' ref=u'node/0' labels=set([u'Filesystem']) properties={u'type': u'Directory', u'name': u'dvwa', u'id': 0}>],
 [<Node graph=u'http://localhost:7474/db/data/' ref=u'node/0' labels=set([u'Filesystem']) properties={u'type': u'Directory', u'name': u'dvwa', u'id': 0}>]]
```

```
we can add an elem into a list

a =[]
a << 123
a << 456
a

result:
('len:', 2)
123
456
```






# tp2 doc
https://github.com/spmallette/GremlinDocs


