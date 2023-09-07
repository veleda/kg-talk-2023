# Demos
## SPARQL demo
### First part
Runned on [Apache Jena Fuseki](https://jena.apache.org/documentation/fuseki2/) 4.4.0. A couple of simple queries on the `SPARQL/input.ttl` file.

```sparql
# Returns all triples from input.ttl
SELECT *
WHERE {
  ?s ?p ?o .
}
```
Returns all of the 8 triples in the dataset.

```sparql
# Returns everything that has some relationship ex:founder to something from input.ttl
PREFIX ex: <http://example.org/>
SELECT ?s ?o
WHERE {
  ?s ex:founder ?o .
}
```
Returns the URIs of `?s` and `?o`, which in this case is `ex:Porsche` and `ex:FerdinandPorsche`.

### Second part
Case: figure out if there are more founders of Porsche, rather than only Ferdinand. 
Solution: asking the [Wikidata](https://www.wikidata.org/wiki/Wikidata:Main_Page) endpoint.

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX p: <http://www.wikidata.org/prop/>

SELECT *
WHERE
  { SERVICE  <https://query.wikidata.org/bigdata/namespace/wdq/sparql> 
    {
      # Finding everything with label "Porsche" in english.            
      ?s rdfs:label "Porsche"@en .
    }
  }
```
Get 7 hits. We want only the one Porsce, automobile manufacturer. Need to narrow down the search.

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX p: <http://www.wikidata.org/prop/>

SELECT ?s ?industry
WHERE
  { SERVICE  <https://query.wikidata.org/bigdata/namespace/wdq/sparql> 
    {
      # Finding that one Porsche that is in the automotive industry.          
      ?s rdfs:label "Porsche"@en ;
                             p:P452 ?industry .
      ?industry ?p ?o .
      ?o rdfs:label "automotive industry"@en .
    }
  }
```
Now that we have the correct Porsche, let's find the founders!

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX p: <http://www.wikidata.org/prop/>

SELECT ?founder ?name
WHERE
  { SERVICE  <https://query.wikidata.org/bigdata/namespace/wdq/sparql> 
    {
      # Finding that one Porsche that is in the automotive industry.          
      ?s rdfs:label "Porsche"@en ;
                             p:P452 ?industry ;
                             p:P112 ?founders .
    
      ?founders ?p1 ?founder .
      ?founder rdfs:label ?name .
      # Need to filter on language to avoid way too many hits.
      FILTER(LANG(?name) = 'en')
    
      ?industry ?p ?o .
      ?o rdfs:label "automotive industry"@en .
    }
  }
```
Gives us three founders of Porsche. 

## Protégé reasoning demo
We construct a graph containing (`OWL/failed.owl`); 
* Classes (sets): `Engine`, `DieselVehicle`, `ElectricVehicle` all sets are disjoint
* Object Property (relationship): `engine` with domain to `DieselVehicle` and `ElectricVehicle`, and range to `Engine`
* Individuals (elements in the sets):
  * `911` typed as `DieselVehicle` and object property assertion `engine` to `DieselEngine`
  * `Taycan` typed as `ElectricVehicle` and object property assertion `engine` to `ElectricEngine`
  * `DieselEngine` typed as `Engine`
  * `ElectricEngine` typed as `Engine`

The classes are disjoint because we do not allow for any element to be both a `DieselVehicle` and `ElectricVehicle` (or `Engine`) at the same time. So they have nothing in common. 

If we run a _reasoner_ on such a graph, it will discover a _logical contradiction_. Since we have stated this disjointness between classes, and at the same time set the domain (what we expect in subject position of a triple) to be **both** `DieselVehicle` and `ElectricVehicle`. We imply that something _must_ be in common between these two sets. The relationship of `engine` goes from (domain) the _intersection_ between `DieselVehicle` and `ElectricVehicle`, but since the intersection between two disjoint classes always is the _empty set_, this relationship can never be used. 

The solution (`OWL/passed.owl`) is to introduce a higher abstraction. So we put `DieselVehicle` and `ElectricVehicle` as subsets in a new superset we call `Vehicle`, and update the domain of `engine` to be this new set. Then it works as we want to! Since `DieselVehicle` and `ElectricVehicle` is everything that `Vehicle` also is, and we can keep them disjoint.

You can try yourself by opening the files in the folder OWL in [Protégé](https://protege.stanford.edu/).

## Resources on the web

* [Apache Jena Fuseki](https://jena.apache.org/documentation/fuseki2/)
* [Wikidata SPARQL Endpoint](https://query.wikidata.org/)
* [Protégé Ontology modelling tool](https://protege.stanford.edu/)

* [Knowledge Graph Oslo Meetup](https://www.meetup.com/knowledge-graph-oslo/) (Slack channel: knowledgegraphoslo.slack.com)
