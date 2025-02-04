# Project 4
 
# Querying Relations - Kata 8 - 0pts
One difference between SPARQL and SQL is that SPARQL can query relations as data, whereas in SQL relations are represented as columns in a schema.

Write a SPARQL query to find how the planet Venus is related to The Morning Star, as represented in the following RDF data:

RDF Data:
```turtle
@prefix : <http://space.com> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .

:Venus  rdf:type Planet ;
        rdfs:label "Venus"@en ; 
        owl:sameAs :Hesperus .

:MorningStar rdfs:label "The Morning Star"@en .
```

### Initial Solution:
```sparql
SELECT owl:sameAs
WHERE {
  :Venus owl:sameAs :MorningStar
}
```

### Solution:
```sparql
SELECT ?predicate
WHERE {
  :Venus ?predicate :MorningStar
}
```


| predicate  |
| ---------- |
| owl:sameAs |





# Greatest Ancestor - Kata 4 - 10pts
The property path expresions "*" and "+" can be used for recursive queries.
Write a recursive query using a property path expression to find the greatest ancestor superclass of BFO "role" that isn't owl:Thing.

RDF Data
```turtle
@prefix : <http://purl.obolibrary.org/obo/bfo.owl> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

:BFO_0000023 rdf:type owl:Class ;
                rdfs:subClassOf :BFO_0000017 ;
                rdfs:label "role"@en .

:BFO_0000017 rdf:type owl:Class ;
                rdfs:subClassOf :BFO_0000020 ;
                rdfs:label "realizable entity"@en .

:BFO_0000020 rdf:type owl:Class ;
                rdfs:subClassOf :BFO_0000002 ;
                rdfs:label "specifically dependent continuant"@en .

:BFO_0000040 rdf:type owl:Class ;
                rdfs:subClassOf :BFO_0000004 ;
                rdfs:label "material entity"@en .

:BFO_0000004 rdf:type owl:Class ;
                rdfs:subClassOf :BFO_0000002 ;
                rdfs:label "independent continuant"@en .

:BFO_0000002 rdf:type owl:Class ;
                rdfs:subClassOf :BFO_0000001 ;
                rdfs:label "continuant"@en .

:BFO_0000001 rdf:type owl:Class ;
                rdfs:subClassOf owl:Thing ;
                rdfs:label "entity"@en .
```

### Initial Solution
```sparql
SELECT DISTINCT *
WHERE { 
    <http://purl.obolibrary.org/obo/BFO_0000023> rdfs:subClassOf* ?ancestor .
    # Now find the "greatest ancestor" that isn't owl:Thing
}
```

### Solution
```sparql
SELECT DISTINCT * 
WHERE { 
    <http://purl.obolibrary.org/obo/BFO_0000023> rdfs:subClassOf* ?ancestor .
    ?s rdfs:subClassOf owl:Thing
}
```

| ancestor |
| -------- |
| http://purl.obolibrary.org/obo/BFO_0000001 |



# Deriving Uncles - Kata 4 - 10pts + SQL (5pts)
Being an uncle of someone depends on certain parent-child relationships, and not the other way around. This is a useful relationship to derive instead of adding as a separate fact (or "materialized inference") in a database, which would need to be kept consistent with the parent-child relationships.

Given a set of pairs of individuals who are related by the “parent of” relationship, find all pairs of individuals who are related by the “uncle of” relationship. 

RDF Data:
```turtle
@prefix : <http://example.com/family-tree#> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .
​
:parentOf rdf:type owl:ObjectProperty .
​
:Grandpa_Ted  rdf:type owl:NamedIndividual ;
              :parentOf :Jim ,
                        :Bob .
​
:Jim  rdf:type owl:NamedIndividual ;
      :parentOf :Jimmy .
​
:Bob  rdf:type owl:NamedIndividual ;
      :parentOf :Bobby .

:Bobby rdf:type owl:NamedIndividual .

:Jimmy rdf:type owl:NamedIndividual .
```

### Initial Solution: 
```sparql
PREFIX : <http://example.com/family-tree#>
SELECT ?uncle ?nephew
WHERE { 
  ?grandpa :parentOf ?father .
  ?grandpa :parentOf ?uncle .
  ?father :parentOf ?nephew .
  # Missing condition here. We need to exclude the parents of each nephew.
}
```

### Solution:
SPARQL Query:
```sparql
PREFIX : <http://example.com/family-tree#>
SELECT ?uncle ?nephew
WHERE { 
  ?grandpa :parentOf ?father .
  ?grandpa :parentOf ?uncle .
  ?father :parentOf ?nephew .
  FILTER(?father != ?uncle)
}
```

| uncle | nephew |
| ----- | ------ |
| Bob   | Jimmy  |
| Jim   | Bobby  |


Equivalent SQL Data:

```sql
CREATE TABLE ParentChild(Parent varchar, Child varchar);
INSERT INTO ParentChild
VALUES ("Grandpa Ted","Jim"), ("Grandpa Ted","Bob"), ("Jim", "Jimmy"), ("Bob", "Bobby");
```
Equivalent SQL Query Solution: 

Testable here http://sqlfiddle.com/#!5/16abb2/2
```sql
SELECT grandkids.child as nephew, grandparents.child as uncle
FROM ParentChild as grandkids
INNER JOIN ParentChild parents ON parents.child = grandkids.parent
INNER JOIN ParentChild grandparents ON grandparents.parent = parents.parent
-- filter to uncles (grandparent's children who aren't their parent)
WHERE grandkids.parent != grandparents.child 
```


# Least Common Ancestor - Kata 3 - 20pts
The Least Common Ancestor (or "least common subsumer") of two classes is the nearest class that's an ancestor of both classess.
Find the least common ancestor of the BFO classes "role" and "material entity".

Use the RDF Data from the previous Kata that describes a fragment of BFO.

### Initial Solution:
```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> 
PREFIX owl: <http://www.w3.org/2002/07/owl#> 

SELECT DISTINCT *
WHERE { 
    <http://purl.obolibrary.org/obo/BFO_0000023> rdfs:subClassOf* ?LCA .
    <http://purl.obolibrary.org/obo/BFO_0000040> rdfs:subClassOf* ?LCA .
    # This gets all ancestors of both classes. Now find the "least common" to both.
}
```

### Solution:
```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> 
PREFIX owl: <http://www.w3.org/2002/07/owl#> 

SELECT DISTINCT *
WHERE { 
    <http://purl.obolibrary.org/obo/BFO_0000023> rdfs:subClassOf* ?LCA .
    <http://purl.obolibrary.org/obo/BFO_0000040> rdfs:subClassOf* ?LCA .
    FILTER NOT EXISTS {
      <http://purl.obolibrary.org/obo/BFO_0000023> rdfs:subClassOf* ?anotherAncestor .
      <http://purl.obolibrary.org/obo/BFO_0000040> rdfs:subClassOf* ?anotherAncestor .
      ?anotherAncestor rdfs:subClassOf ?LCA .
    }
}
```


| LCA |
| --- |
| http://purl.obolibrary.org/obo/BFO_0000002 |





# Missing Definitions - Kata 3 - 20pts
Write a query to find all OWL classes with missing definitions from the IAO ontology http://purl.obolibrary.org/obo/iao.owl. 
Filter these classes to only terms whose IRIs have the prefix "IAO_". 
Use http://purl.obolibrary.org/obo/IAO_0000115 for the "definition" property. 
Order these classes by their IRIs.

Note on testing this kata: If querying IAO from a public SPARQL web endpoint like https://ubergraph.apps.renci.org/sparql on https://yasgui.triply.cc/, include the clause `FROM <http://purl.obolibrary.org/obo/iao.owl>`
### Initial Solution:
```sparql
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX iao: <http://purl.obolibrary.org/obo/IAO_>

SELECT DISTINCT ?entity
FROM <http://purl.obolibrary.org/obo/iao.owl>
WHERE { 
  FILTER NOT EXISTS { ?entity obo:IAO_0000115 ?value } .
  ?entity a owl:Class .
  FILTER(!isBlank(?entity)) .
  # Now filter to classes whose IRI starts with "IAO_"
}
ORDER BY ?entity
```

### Solution:
```sparql
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX iao: <http://purl.obolibrary.org/obo/IAO_>

SELECT DISTINCT ?entity
FROM <http://purl.obolibrary.org/obo/iao.owl>
WHERE { 
  FILTER NOT EXISTS { ?entity obo:IAO_0000115 ?value } .
  ?entity a owl:Class .
  FILTER(!isBlank(?entity)) .
  FILTER(STRSTARTS(str(?entity), str(iao:))) .
}
ORDER BY ?entity
```

| entity |
| ------ |
| obo:IAO_0000008 |
| obo:IAO_0000018 |
| obo:IAO_0000034 |
| obo:IAO_0000057 |
| obo:IAO_0000128 |
| obo:IAO_0000141 |
| obo:IAO_0000429 |
| obo:IAO_0000582 |
| obo:IAO_8000000 |
| obo:IAO_8000016 |
| obo:IAO_8000017 |
| obo:IAO_8000019 |
| obo:IAO_8000020 |





# OWL Property Restrictions - Kata 3 - 20pts
Informally, an infection is something that is part of an organism and has an infectious agent as a part. 

Formally, this might be represented as: `"infection subClassOf (part of some extended organism) and (has part some infectious agent)"`.

Write a query to find what an infection is part of, using this turtle file as the definition encoded in RDF.

RDF Data:
```turtle
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix obo: <http://purl.obolibrary.org/obo/> .

<http://purl.obolibrary.org/obo/IDO_0000586>
  a owl:Class ;
  rdfs:label "infection"@en ;
  rdfs:subClassOf <http://purl.obolibrary.org/obo/BFO_0000040>, [
    a owl:Class ;
    owl:intersectionOf (
     _:genid2
     _:genid4
   )
  ] .

<http://purl.obolibrary.org/obo/BFO_0000040>
  a owl:Class ;
  rdfs:label "material entity"@en .

_:genid2
  a owl:Restriction ;
  owl:onProperty obo:BFO_0000050 ;
  owl:someValuesFrom obo:OGMS_0000087 .

_:genid4
  a owl:Restriction ;
  owl:onProperty obo:BFO_0000051 ;
  owl:someValuesFrom obo:IDO_0000596 .

<http://purl.obolibrary.org/obo/BFO_0000050>
  a owl:ObjectProperty, owl:TransitiveProperty ;
  rdfs:label "part of"@en .

obo:OGMS_0000087
  a owl:Class ;
  rdfs:label "extended organism" .

<http://purl.obolibrary.org/obo/BFO_0000051>
  a owl:ObjectProperty, owl:TransitiveProperty ;
  rdfs:label "has part"@en .

obo:IDO_0000596
  a owl:Class ;
  rdfs:label "infectious agent"@en .

```
### Initial Solution:
```sparql
SELECT DISTINCT ?whole
WHERE { 
  <http://purl.obolibrary.org/obo/IDO_0000586> rdfs:subClassOf ?superclass .
    ?superclass owl:intersectionOf ?intersection .
    # Now query the elements of the intersection
}
```

### Solution:
```sparql
SELECT DISTINCT ?whole
WHERE { 
    <http://purl.obolibrary.org/obo/IDO_0000586> rdfs:subClassOf ?superclass .
    ?superclass owl:intersectionOf ?intersection .
    ?intersection rdf:rest*/rdf:first ?o .
    ?o owl:onProperty <http://purl.obolibrary.org/obo/BFO_0000050> .
    ?o owl:someValuesFrom ?whole . 
}
```


| whole |
| ----- |
| http://purl.obolibrary.org/obo/OGMS_0000087 |
