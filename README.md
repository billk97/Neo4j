# Neo4j

### 
IMPORTING DATA
```sql
create constraint on (a:Author) assert a.name is unique; 
create constraint on (o:Article) assert o.id is unique;
```

loading article nodes 29555 nodes
```sql
LOAD CSV FROM "file:///ArticleNodes.csv" as row
CREATE (a:Article{id: toInteger(row[0])})
set a.title = row[1],
a.year = date(row[2]),
a.jurnal = row[3],
a.abstract = row[4]
```

```sql
LOAD CSV FROM "file:///AuthorNodes.csv" as row
Merge (a:Author{name: row[1]})

```
Added 15420 labels, created 15420 nodes, set 15420 properties, completed after 833 ms.

```sql
LOAD CSV FROM "file:///AuthorNodes.csv" as row
match(a:Article{id: toInteger(row[0])})
match(o:Author{name: row[1]})
create(o)-[:WROTE]->(a)
```

```
LOAD CSV FROM "file:///Citations.csv" as row
with row, split(row[0], "	") as srow
match(article1:Article{id: toInteger(srow[0])})
match(article2:Article{id: toInteger(srow[1])})
where article1.id <> article2.id
create(article1)-[:REFRERENCES]->(article2)
```


### one 
~~~cypher
match(a:Article)-[:REFERENCES]->(reciver:Article) match(o:Author)-[:WROTE]->(reciver:Article)
return o.name, count(reciver) as count order 
by count desc 
limit 5
~~~
Created 58340 relationships, completed after 1529 ms.


### two
```sql
MATCH(author1:Author)-[:WROTE]->(a:Article)
MATCH(author2:Author)-[:WROTE]->(a)
where author1.name <> author2.name
MERGE (author1)-[:COLLABORATED]->(author2)

//then 

MATCH(a:Author)-[c:COLLABORATED]->(b:Author)
return a.name , count(c) as count
order by count desc
limit 5
```
|author|colaborations|
|-|-|
|"C.N. Pope"	|50|
|"S. Ferrara"	|46|
|"M. Schweda"	|46|
|"H. Lu"	|45|
|"C. Vafa"|	45|

### three
```sql
match(a:Author)-[:WROTE]->(article:Article)
with article as article,count(*) as c, a as a
where c<2
return a.name, count(a.name) as amount
order by amount desc 
```
|author name| number of papers|
|-|-|
|"C.N. Pope"|	127|
|"H. Lu"|	122|
|"A.A. Tseytlin"|	111|
|"Edward Witten"	|98|
|"Shinichi Nojiri"|	92|

### for
```sql
match(o:Author)-[:WROTE]->(a:Article)
where a.year = date("2001")
return o.name, count(o) as count
order by count desc limit 1
```
result -> "Ashok Das"	17

### five
```sql
match(a:Article)
where a.title CONTAINS 'gravity' and a.year = date("1998")
return a.jurnal, count(*) as rank
order by rank desc
limit 1
```
result -> "Nucl.Phys."	25

### six
```sql
match(:Article)-[:REFERENCES]->(a:Article)
return a.title, count(*) as rank
order by rank desc
limit 5
```
result:
|title|citations|
|-|-|
|The Large N Limit of Superconformal Field Theories and Supergravity"|	2414|
|Anti De Sitter Space And Holography"	|1775|
|"Gauge Theory Correlators from Non-Critical String Theory"	|1641|
|"Monopole Condensation And Confinement In N=2 Supersymmetric Yang-Mills"	|1299|
|"M Theory As A Matrix Model: A Conjecture"|	1199|


### seven 
```sql
match(o:Author)-[:WROTE]->(a:Article)
where a.abstract contains "holography" and a.abstract contains "anti de sitter"
return o.name, a.title
```
result-> null
```sql
match(o:Author)-[:WROTE]->(a:Article)
where a.abstract contains "holography" 
return o.name, a.title
```
result: 
|author name | title|
|-|-|
|"Petr Horava"	|"Probable Values of the Cosmological Constant in a Holographic Theory"|
|"Djordje Minic"|	"Probable Values of the Cosmological Constant in a Holographic Theory"|
|"U. Moschella"	|"Decomposing Quantum Fields on Branes"|
|"R. Schaeffer"	|"Decomposing Quantum Fields on Branes"|
|"J. Bros"	|"Decomposing Quantum Fields on Branes"|
|"M. Bertola"|	"Decomposing Quantum Fields on Branes"|
|"V. Gorini"	|"Decomposing Quantum Fields on Branes"|
|"Itzhak Bars"|	"Two-Time Physics in Field Theory"|
|"J.G. Russo"	|"Hyperbolic Spaces in String and M-Theory"|
|"A. Kehagias"	|"Hyperbolic Spaces in String and M-Theory"|

```sql
match(o:Author)-[:WROTE]->(a:Article)
where a.abstract contains "anti de sitter" 
return o.name, a.title
```
return -> null

### eight

```sql
match(a:Author{name: 'C.N. Pope'})
match(b:Author{name: 'M. Schweda'})
match p=shortestPath((a)-[*]-(b))
return p
```
result -> length = 4

### nine
```sql
match(a:Author{name: 'C.N. Pope'})
match(b:Author{name: 'M. Schweda'})
match p=shortestPath((a)-[:WROTE]-(b))
return p, length(p) as length
```
no result like no papers connect the two
