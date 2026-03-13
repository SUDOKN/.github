## What is SUDOKN?


SUDOKN is a knowledge graph focused on representing manufacturing capabilities of Small and Medium-Sized Manufacturers (SMM). SUDOKN is part of [NSF Proto-OKN](https://www.proto-okn.net/) initiative. 

In this project, we will prototype and deploy the Supply and Demand Open Knowledge Network (SUDOKN) which is composed of several open and interconnected knowledge graphs, aligned with formal ontologies, that collectively represent various types of supply and demand data needed to address the challenges posed by the use cases related to supplier discovery, capability and capacity analysis, and vulnerability assessment. This project is aimed at democratizing access to publicly available supply and demand data and maximizing its utility by proposing a three-pronged approach:

Develop a set of axiomatized and reusable manufacturing ontologies.
Develop and deploy a set of tools and methods for data collection, validation, and ingestion
Use these tools and ontologies to develop and deploy an open Manufacturing Capability Network (MCN)

NIST MEP is the partner agency in this project. 

## Want to learn more?
Check out [ASU SUDOKN Project](https://projects.engineering.asu.edu/sudokn/)
Ask a question to Dr Farhad Ameri (farhad.ameri@asu.edu)
Read about the NSF Sponsored [Proto-OKN](https://www.proto-okn.net/) projects

## Example Queries

### 1. Which manufacturers have stamping capability and serve the aerospace industry?

```sparql
PREFIX iof-core: <https://spec.industrialontologies.org/ontology/core/Core/>
PREFIX sdk: <http://asu.edu/semantics/SUDOKN/>

SELECT ?mfg
WHERE {
    ?mfg sdk:hasProcessCapability ?p.
    ?p a sdk:StampingCapability.
    ?mfg sdk:suppliesToIndustry ?ind.
    ?ind a sdk:AerospaceIndustry.
}
```

### 2. What are the top three manufacturing process capabilities available in North Carolina?

```sparql
PREFIX sdk: <http://asu.edu/semantics/SUDOKN/>
PREFIX iof-core: <https://spec.industrialontologies.org/ontology/core/Core/>

SELECT ?type (COUNT(DISTINCT ?capability) AS ?count)
WHERE {
  ?manufacturer a iof-core:Manufacturer ;
                sdk:organizationLocatedIn ?site ;
                sdk:hasProcessCapability ?capability .

  ?site sdk:locatedInState sdk:North%20Carolina-state-individual .
  ?capability a ?type .

  FILTER(STRSTARTS(STR(?type), STR(sdk:)))
}
GROUP BY ?type
ORDER BY DESC(?count)
LIMIT 3
```

### 3. How many manufacturers in NC were established after 1990?

```sparql
PREFIX iof-core: <https://spec.industrialontologies.org/ontology/core/Core/>
PREFIX sdk: <http://asu.edu/semantics/SUDOKN/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT (COUNT(DISTINCT ?manufacturer) AS ?count)
WHERE {
  ?manufacturer a iof-core:Manufacturer ;
                sdk:organizationLocatedIn ?site ;
                sdk:hasOrganizationYearOfEstablishment ?year .

  ?site sdk:locatedInState sdk:North%20Carolina-state-individual .

  FILTER(xsd:integer(?year) > 1990)
}
```

### 4. Which manufacturers only provide machining services and can machine both aluminum and stainless steel?

```sparql
PREFIX iof-core: <https://spec.industrialontologies.org/ontology/core/Core/>
PREFIX sdk: <http://asu.edu/semantics/SUDOKN/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?manufacturer ?label
WHERE {
    ?manufacturer a iof-core:Manufacturer ;
                  rdfs:label ?label ;
                  sdk:hasProcessCapability ?capability ;
                  sdk:hasMaterialCapability ?materialCapability1, ?materialCapability2 .

    ?capability a sdk:MachiningCapability .

    ?materialCapability1 a sdk:StainlessSteelProcessingCapability .
    ?materialCapability2 a sdk:AluminumProcessingCapability .

    FILTER NOT EXISTS {
        ?manufacturer sdk:hasProcessCapability ?otherCapability .
        FILTER NOT EXISTS { ?otherCapability a sdk:MachiningCapability . }
    }
}
```

### 5. Which manufacturers have a secondary NAICS classification of 326, and what certificates do they hold?

```sparql
PREFIX iof-core: <https://spec.industrialontologies.org/ontology/core/Core/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX sdk: <http://asu.edu/semantics/SUDOKN/>

SELECT ?manufacturer ?label ?certificate
WHERE {
    ?manufacturer a iof-core:Manufacturer ;
                  rdfs:label ?label ;
                  sdk:hasSecondaryNAICSClassifier ?naicsCode ;
                  sdk:hasCertificate ?certificate .

    ?naicsCode a sdk:NAICS326 .
}
```

### 6. Return all manufacturers, along with their city, state, and longitude/latitude information.

```sparql
PREFIX sdk: <http://asu.edu/semantics/SUDOKN/>
PREFIX iof-core: <https://spec.industrialontologies.org/ontology/core/Core/>
PREFIX iof-scro: <https://spec.industrialontologies.org/ontology/scro/SCRO/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?mfg ?cityLabel ?stateLabel ?lat ?long
WHERE {
  ?mfg a iof-core:Manufacturer ;
       sdk:organizationLocatedIn ?site .

  ?site sdk:locatedInCity ?city ;
        sdk:locatedInState ?state ;
        sdk:hasGeospatialLocation ?geo .

  ?city rdfs:label ?cityLabel .
  ?state rdfs:label ?stateLabel .

  ?geo sdk:hasLatitudeValue ?lat ;
       sdk:hasLongitudeValue ?long .
}
```
### 7. Which North Carolina aerospace suppliers can machine aluminum and stainless steel, have at least 50 employees, and are AS9100 or NADCAP certified?

```sparql
PREFIX sdk: <http://asu.edu/semantics/SUDOKN/>
PREFIX iof-core: <https://spec.industrialontologies.org/ontology/core/Core/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?company ?employees
WHERE {
  ?m a iof-core:Manufacturer ;
     rdfs:label ?company ;
     sdk:suppliesToIndustry [ a sdk:AerospaceIndustry ] ;
     sdk:hasProcessCapability [ a sdk:MachiningCapability ] ;
     sdk:hasMaterialCapability [ a sdk:AluminumProcessingCapability ],
                               [ a sdk:StainlessSteelProcessingCapability ] ;
     sdk:hasNumberOfEmployees ?employees ;
     sdk:organizationLocatedIn ?site .

  ?site sdk:locatedInState sdk:North%20Carolina-state-individual .
  sdk:North%20Carolina-state-individual rdfs:label ?state .

  FILTER(?employees >= 50)

  FILTER EXISTS {
    ?m sdk:hasCertificate ?c .
    { ?c a sdk:AS9100Certificate } UNION { ?c a sdk:NADCAPCertificate }
  }
}
ORDER BY LCASE(?company)
```
### 8. Which manufacturers in North Carolina have the largest number of distinct process and material capabilities?

```sparql
PREFIX sdk: <http://asu.edu/semantics/SUDOKN/>
PREFIX iof-core: <https://spec.industrialontologies.org/ontology/core/Core/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?m ?label (COUNT(DISTINCT ?capType) AS ?capabilityCount)
WHERE {
  ?m a iof-core:Manufacturer ;
     rdfs:label ?label ;
     sdk:organizationLocatedIn ?site .

  ?site sdk:locatedInState sdk:North%20Carolina-state-individual .

  {
    ?m sdk:hasProcessCapability ?cap .
    ?cap a ?capType .
  }
  UNION
  {
    ?m sdk:hasMaterialCapability ?cap .
    ?cap a ?capType .
  }

  FILTER(STRSTARTS(STR(?capType), STR(sdk:)))
}
GROUP BY ?m ?label
ORDER BY DESC(?capabilityCount)
LIMIT 10
```
### 9. Which manufacturers have both machining and fabrication capabilities, and what are all their listed process capabilities?

```sparql
PREFIX sdk: <http://asu.edu/semantics/SUDOKN/>
PREFIX iof-core: <https://spec.industrialontologies.org/ontology/core/Core/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT  ?label
       (GROUP_CONCAT(DISTINCT STRAFTER(STR(?ptype), STR(sdk:)); separator=", ") AS ?processCapabilities)
WHERE {
  ?m a iof-core:Manufacturer ;
     rdfs:label ?label ;
     sdk:hasProcessCapability [ a sdk:MachiningCapability ],
                              [ a sdk:FabricatingCapability ] ;
     sdk:hasProcessCapability ?p .

  ?p a ?ptype .
  FILTER(STRSTARTS(STR(?ptype), STR(sdk:)))
}
GROUP BY ?m ?label
ORDER BY LCASE(?label)
```

### 10. Which states have manufacturers with powder coating capability?

```sparql
PREFIX sdk: <http://asu.edu/semantics/SUDOKN/>
PREFIX iof-core: <https://spec.industrialontologies.org/ontology/core/Core/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?label
WHERE {
  ?m a iof-core:Manufacturer ;
     rdfs:label ?label ;
     sdk:hasMaterialCapability [ a sdk:StainlessSteelProcessingCapability ] ;
     sdk:organizationLocatedIn ?site .

  ?site sdk:locatedInState sdk:North%20Carolina-state-individual .
}
ORDER BY ?label
```

### 11. Which North Carolina manufacturers serve the aerospace industry?

```sparql
PREFIX sdk: <http://asu.edu/semantics/SUDOKN/>
PREFIX iof-core: <https://spec.industrialontologies.org/ontology/core/Core/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?label
WHERE {
  ?m a iof-core:Manufacturer ;
     rdfs:label ?label ;
     sdk:suppliesToIndustry [ a sdk:AerospaceIndustry ] ;
     sdk:organizationLocatedIn ?site .

  ?site sdk:locatedInState sdk:North%20Carolina-state-individual .
}
ORDER BY ?label
```
