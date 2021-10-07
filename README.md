# Building a knowledge graph of frames defined within the context of the the PropBank project

In what follows, `fx` refers to the following command
```
java -jar sparql-anything-0.3.2-SNAPSHOT.jar  
```

## Input

The input for the process is a [release of the Propbank dataset](https://github.com/propbank/propbank-frames/releases). 
The process supposes that the PropBank release is distributed as a single zip file containing a folder ``frames`` which stores all the XML files defining the PropBank frames.
An example of such zip file can be found [here](https://github.com/propbank/propbank-frames/archive/refs/tags/v3.1.zip).

## Process

SPARQL anything allows you to extract the frames from all the XML files and specify them according you favourite ontology with a single query.
In this showcase we specify frame data according to the [Framester ontology](https://w3id.org/framester).


```

PREFIX frame: <http://www.ontologydesignpatterns.org/ont/framenet/abox/frame/>
PREFIX fx: <http://sparql.xyz/facade-x/ns/>
PREFIX xyz: <http://sparql.xyz/facade-x/data/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
prefix pbschema: <https://w3id.org/framester/pb/pbschema/>
PREFIX fschema: <https://w3id.org/framester/schema/>
CONSTRUCT {
  ?fPredicate a pbschema:Predicate ;
    rdfs:label ?lemma ;
    pbschema:hasRoleSet ?fRoleset .
  ?fRoleset a pbschema:RoleSet ;
    pbschema:hasAlias ?fFrame ;
    rdfs:label ?name ;
    pbschema:hasRole ?fRole .
  ?fRole a pbschema:Role ;
    rdfs:label ?roleDescription ;
    fschema:subsumedUnder ?argClass .

}
WHERE {
  SERVICE <x-sparql-anything:> {
    fx:properties fx:location "https://github.com/propbank/propbank-frames/archive/refs/tags/v3.1.zip" .
    fx:properties fx:archive.matches "propbank-frames-3.1/frames/[^_]*.xml" .
    ?s1 ?p1 ?file1 .
    SERVICE <x-sparql-anything:> {
      fx:properties fx:location ?file1 .
      fx:properties fx:from-archive "https://github.com/propbank/propbank-frames/archive/refs/tags/v3.1.zip" .
      ?s a xyz:frameset .
      ?s ?li ?predicate .
      ?predicate a xyz:predicate .
      ?predicate xyz:lemma ?lemma .
      ?predicate ?li2 ?roleset .
      ?roleset a xyz:roleset .
      ?roleset xyz:name ?name .
      ?roleset ?li3 ?aliases .
      ?roleset xyz:id ?id .
      ?roleset ?li5 ?roles .
      ?roles a xyz:roles .
      ?roles ?li6 ?role .
      ?role a xyz:role .
      ?role xyz:n ?num .
      ?role xyz:descr ?roleDescription .
      ?aliases a xyz:aliases .
      ?aliases ?li4 ?alias .
      ?alias a xyz:alias .
      ?alias xyz:framenet ?f .
      BIND(IRI(CONCAT("https://w3id.org/framester/pb/data/predicate/",str(?lemma))) AS ?fPredicate)
      BIND(IRI(CONCAT("https://w3id.org/framester/pb/data/roleset/",str(?id))) AS ?fRoleset)
      BIND(IRI(CONCAT("https://w3id.org/framester/pb/data/roleset/",str(?id),"__",str(?num))) AS ?fRole)
      BIND(IRI(CONCAT("https://w3id.org/framester/pb/pbschema/ARG",str(?num))) AS ?argClass)
      OPTIONAL{
        SERVICE <x-sparql-anything:> {
          fx:properties fx:content ?f .
          fx:properties fx:txt.split " " .
          Filter(isLiteral(?frameId))
          FILTER(strlen(str(?f))>0)
          ?a ?b ?frameId
          BIND(IRI(CONCAT("https://w3id.org/framester/framenet/abox/frame/", ?frameId)) AS ?fFrame)
        }
      }
    }
  }
}

```