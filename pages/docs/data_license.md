---
title: Data License
keywords: HOBBIT Documentation
sidebar: main_sidebar
toc: false
permalink: data_license
folder: docs
---

## Specifying the data license

It is possible to include data licensing and attribution information in the graph where public data, such as experiment results, is stored.

To do so, use a query similar to this (that's what http://master.project-hobbit.eu/ uses):

```
PREFIX cc: <https://creativecommons.org/ns#>
PREFIX dcat: <http://www.w3.org/ns/dcat#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xhv: <http://www.w3.org/1999/xhtml/vocab#>

INSERT DATA {
GRAPH <http://hobbit.org/graphs/PublicResults> {
<http://hobbit.org/graphs/PublicResults> a dcat:Dataset ;
cc:license <https://creativecommons.org/licenses/by/4.0/legalcode> ;
cc:attributionName "HOBBIT Project" ;
cc:attributionURL <http://project-hobbit.eu/> .
<https://creativecommons.org/licenses/by/4.0/legalcode> a cc:License ;
rdfs:label "CC BY 4.0" ;
xhv:icon <https://mirrors.creativecommons.org/presskit/buttons/88x31/svg/by.svg> .
}
}
```
