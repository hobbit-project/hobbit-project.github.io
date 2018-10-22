---
title: Internal Challenge Wokflow
keywords: HOBBIT Documentation
sidebar: main_sidebar
toc: false
permalink: internal_challenge_workflow
folder: docs
---

This page documents the internal workflow of a challenge. This includes the places at which information is stored and when it is moved. 

### Storage of challenge data

The platforms storage comprises 3 different RDF graphs:
* a challenge definition graph (throughout this article "def graph", `http://hobbit.org/graphs/ChallengeDefinitions`)
* a private result graph (`http://hobbit.org/graphs/PrivateResults`)
* a public result graph (`http://hobbit.org/graphs/PublicResults`)

The private and def graphs are only used for challenge information. After a challenge has been published, all its information is moved into the public graph.

### Challenge workflow

The internal workflow of a typical challenge is the following:

1. The organizer creates the challenge and the tasks.
  * The following information is stored in the def graph:
    * The challenge information
    * The challenge tasks
    * The benchmark configuration parameters of the tasks
2. The organizer makes the challenge visible.
  * In the def graph, a visibility flag of the challenge is set to `true`.
3. Participants register their systems.
  * The registration is stored in the def graph.
4. The organizer closes the challenge.
  * The platform controller will move the information necessary for executing the experiments into the platforms queue
  * In the def graph, a closed flag of the challenge is set to `true`.
5. The experiments are executed.
  * The platform stores the results in the private graph. If the publication date has been reached (or it already passed) the platform stores the results directly in the public graph.
6. The challenge is published.
  * The platform controller regularly checks whether it should publish challenges. If such a challenges is found in the def graph
    * its information is moved to the public graph
    * its experiment results are move from the private graph to the public graph
    * the private and the def graph are cleaned up, i.e., all information that is not connected to a challenge (in the def graph) or an experiment (in the private graph) is removed

### Repeatable challenges

The platform offers repeatable challenges, which have a slightly different workflow. After the 3rd step, the challenge can already be executed. If a challenge run is executed, the registrations of the systems are moved into the private graph. This makes sure that the participants can register again for the next challenge run. Step 4 is carried out automatically as soon as the end of the repitition phase is reached. 

### Setting a challenge back

Sometimes, it is necessary to set a challenge (or at least a part of it) back, because internal errors of the platform may have stopped one or several of the experiments. Although this can be done, it is _highly recommended to create a backup of the storage_ as the following queries will change the data in the storage.

Additionally, it is recommended to check, whether the SPARQL queries in this documentation are still up-to-date. For most of them, a reference query is given which is used by the platform.

#### "Unpublish" a challenge

If a challenge has already been published, its information has to be moved back into the def graph. This can be done by creating a SPARQL UPDATE query which selects this information (based on the [challenge select query](https://github.com/hobbit-project/core/blob/master/src/main/resources/org/hobbit/storage/queries/getChallenge.query)), inserts it into the def graph.

```sparql
PREFIX hobbit: <http://w3id.org/hobbit/vocab#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

INSERT {
  GRAPH <http://hobbit.org/graphs/ChallengeDefinitions> {
    %CHALLENGE_URI% a hobbit:Challenge .
    %CHALLENGE_URI% ?challengeProp ?challengeObj .
    ?challengeTask a hobbit:ChallengeTask .
    ?challengeTask ?taskProp ?taskObj .
    ?challengeTask hobbit:isTaskOf %CHALLENGE_URI% .
    ?challengeTask hobbit:involvesBenchmark ?benchmark .
    ?challengeTask hobbit:involvesSystemInstance ?system .
    ?challengeTask ?parameterProp ?parameterValue .
    ?parameterProp ?parameterPropProp ?parameterPropObject .
    ?parameterProp rdfs:range ?parameterPropRange .
    ?parameterPropRange ?parameterPropRangeProp ?parameterPropRangeObject .
    ?parameterValue ?parameterValueRangeRelation ?parameterPropRange .
    ?benchmark hobbit:hasParameter ?parameterProp .
    ?benchmark ?benchmarkProp ?benchmarkObject .
    ?system ?systemProp ?systemObject .
    ?rankingKPIs ?rankingKPIsProp ?rankingKPIsObject .
  }
}
WHERE {
    {
        GRAPH <http://hobbit.org/graphs/PublicResults> {
            %CHALLENGE_URI% a hobbit:Challenge .
            %CHALLENGE_URI% ?challengeProp ?challengeObj .
        }
    }
    UNION
    {
        GRAPH <http://hobbit.org/graphs/PublicResults> {
            %CHALLENGE_URI% a hobbit:Challenge .
            ?challengeTask hobbit:isTaskOf %CHALLENGE_URI% .
            ?challengeTask a hobbit:ChallengeTask .
            ?challengeTask ?taskProp ?taskObj .
            OPTIONAL {
                ?challengeTask hobbit:rankingKPIs ?rankingKPIs .
                ?rankingKPIs ?rankingKPIsProp ?rankingKPIsObject .
            }
        }
    }
    UNION
    {
        GRAPH <http://hobbit.org/graphs/PublicResults> {
            %CHALLENGE_URI% a hobbit:Challenge .
            ?challengeTask hobbit:isTaskOf %CHALLENGE_URI% .
            ?challengeTask a hobbit:ChallengeTask .
            ?challengeTask hobbit:involvesBenchmark ?benchmark .
            OPTIONAL { ?benchmark ?benchmarkProp ?benchmarkObject . }
        }
    }
    UNION
    {
        GRAPH <http://hobbit.org/graphs/PublicResults> {
            %CHALLENGE_URI% a hobbit:Challenge .
            ?challengeTask hobbit:isTaskOf %CHALLENGE_URI% .
            ?challengeTask a hobbit:ChallengeTask .
            ?challengeTask hobbit:involvesSystemInstance ?system .
            OPTIONAL { ?system ?systemProp ?systemObject . }
        }
    }
    UNION
    {
        GRAPH <http://hobbit.org/graphs/PublicResults> {
            %CHALLENGE_URI% a hobbit:Challenge .
            ?challengeTask hobbit:isTaskOf %CHALLENGE_URI% .
            ?challengeTask a hobbit:ChallengeTask .
            ?challengeTask hobbit:involvesBenchmark ?benchmark .
            ?benchmark hobbit:hasParameter ?parameterProp .
            ?challengeTask ?parameterProp ?parameterValue .
            {?parameterProp a hobbit:Parameter} UNION {?parameterProp a hobbit:ConfigurableParameter} UNION {?parameterProp a hobbit:FeatureParameter}.
            OPTIONAL { ?parameterProp ?parameterPropProp ?parameterPropObject . }
            OPTIONAL {
                ?parameterProp rdfs:range ?parameterPropRange .
                OPTIONAL { ?parameterPropRange ?parameterPropRangeProp ?parameterPropRangeObject . }
                OPTIONAL { ?parameterValue ?parameterValueRangeRelation ?parameterPropRange . }
            }
        }
    }
}
```
`%CHALLENGE_URI%` needs to be replaced with the challenge URI in `<>` brackets.

To make sure that the platform is working on the correct challenge, we should remove at least the challenge (and its tasks) from the public graph.

```sparql
PREFIX hobbit: <http://w3id.org/hobbit/vocab#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>


WITH <http://hobbit.org/graphs/PublicResults>
DELETE {
    %CHALLENGE_URI% a hobbit:Challenge .
    %CHALLENGE_URI% ?challengeProp ?challengeObj .
    ?challengeTask a hobbit:ChallengeTask .
    ?challengeTask ?taskProp ?taskObj .
}
WHERE {
    {
        %CHALLENGE_URI% a hobbit:Challenge .
        %CHALLENGE_URI% ?challengeProp ?challengeObj .
    }
    UNION
    {
        %CHALLENGE_URI% a hobbit:Challenge .
        ?challengeTask hobbit:isTaskOf %CHALLENGE_URI% .
        ?challengeTask a hobbit:ChallengeTask .
        ?challengeTask ?taskProp ?taskObj .
    }
}
```

#### Deleting faulty experiments

The easiest approach is to use the [delete experiment query](https://github.com/hobbit-project/core/blob/master/src/main/resources/org/hobbit/storage/queries/deleteExperiment.query). However, for this query, you will need to know all the single URIs of the experiments. If these are too many, it is possible to adapt the query in the following way:

```sparql
PREFIX hobbit: <http://w3id.org/hobbit/vocab#>

WITH %GRAPH_URI%
DELETE {
    ?experiment ?p ?o .
}
WHERE {
    ?experiment a hobbit:Experiment .
    ?experiment hobbit:isPartOf ?challengeTask .
    ?challengeTask hobbit:isTaskOf %CHALLENGE_URI% .
    ?experiment ?p ?o .
}
```
`%CHALLENGE_URI%` needs to be replaced with the challenge URI in `<>` brackets. The `%GRAPH_URI` needs to be replaced in the same way with the URI of the private or the public graph.

#### "Re-open" a challenge

If a challenge has been closed but its information is still available in the def graph, it can be easily closed by executing the "opposite" of the [close challenge query](https://github.com/hobbit-project/core/blob/master/src/main/resources/org/hobbit/storage/queries/closeChallenge.query).

```sparql
PREFIX hobbit: <http://w3id.org/hobbit/vocab#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

WITH <http://hobbit.org/graphs/ChallengeDefinitions>
DELETE { %CHALLENGE_URI% hobbit:closed ?value . }
INSERT { %CHALLENGE_URI% hobbit:closed "false"^^xsd:boolean . }
WHERE {
    %CHALLENGE_URI% a hobbit:Challenge .
    OPTIONAL { %CHALLENGE_URI% hobbit:closed ?value . }
}
```
`%CHALLENGE_URI%` needs to be replaced with the challenge URI in `<>` brackets.

After this step, the challenge organizer can close the challenge again via the UI and the experiments are recreated. **Note** that you should make sure that all systems are still registered - especially if the challenge is a repeatable challenge.

#### Rerunning single challenge tasks

If only a single task has failed, it might be better to rerun only the experiments of this task. This can be done exactly in the same way as described above. However, before closing the challenge, the triples connecting the other challenge tasks have to be removed temporarily. **Note** that you should add them after the challenge has been closed and the experiments of the single task have been added to the experiment queue, to make sure that the tasks are reconnected to their challenge.

### Queries to repair challenges

In this section, we collect SPARQL queries which help to repair challenges.

#### Remove experiments from a challenge task
Experiments are connected to the single challenge tasks. Sometimes, it might be necessary to remove failed experiments from a single task.

```sparql
PREFIX hobbit: <http://w3id.org/hobbit/vocab#>

WITH %GRAPH_URI%
DELETE {
    ?experiment hobbit:isPartOf %CHALLENGE_TASK_URI% .
}
WHERE {
    ?experiment a hobbit:Experiment .
    ?experiment hobbit:isPartOf %CHALLENGE_TASK_URI% .
}
```
`%CHALLENGE_TASK_URI%` needs to be replaced with the challenge URI in `<>` brackets. The `%GRAPH_URI` needs to be replaced in the same way with the URI of the private or the public graph.

#### Add an experiment to a challenge task
If a single experiment has been repeated and should be added to a challenge task, the following query can be used.

```sparql
PREFIX hobbit: <http://w3id.org/hobbit/vocab#>

WITH %GRAPH_URI%
INSERT {
    %EXPERIMENT_URI% hobbit:isPartOf %CHALLENGE_TASK_URI% .
}
```
`%CHALLENGE_TASK_URI%` as well as %EXPERIMENT_URI% need to be replaced with the challenge URI in `<>` brackets. The `%GRAPH_URI` needs to be replaced in the same way with the URI of graph containing the experiment.

#### Delete a challenge
Sometimes, it might be necessary to delete a challenge. In this case, it has to be decided whether the experiments of the challenge should be kept. In this case, they should be removed from the challenge tasks as described above. If they should be deleted, a query for that can be found in the "Deleting faulty experiments" section above.

**After** the experiments have been handled, the challenge and its tasks can be removed. For deleting the challenge itself, the query used for removing the challenge from the public graph in the secion "“Unpublish” a challenge" above can be used. However, it might be possible, to adapt the graph if the challenge has not been published before.

