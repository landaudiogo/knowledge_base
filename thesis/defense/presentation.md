Importance of the research project What methods were used What were the outcomes
of the research Implications of the study to the field Recomendations for future
work


# Design

Use a plain white background Go beyond the title and bulltepoints provided by
default powerpoint templates Attempt to minimize the amount of information on
the slides. If possible present only a figure or graph to further be described.
sans-serif typography


# Layout

1. Introduction
	1. Present the problem
	2. Explain the structure of the presentation
2. Literature Review
	1. Explain the Approximation Algorithms
	2. Go through existing applications of the BPP
	3. Elucidate the gaps in the literature review for the problem at hand
3. Methodology
	1. Monitor 
	2. Consumer
	3. Controller
		1. State Machine
		2. Modified Algorithms
4. Integration Test 
	1. Present the System's reaction time
5. Conclusion

# Introduction ## Describe the problem's context
1. Extracting data from Kafka Topics into a Datalake. 
3. Kafka divides topics into parititions, and this is the smallest unit from
	 which a consumer extracts data.
4. To fetch data from a topic, Kafka provides consumer groups, which is simply a
	 group of data consumers.
5. A consumer group divides the partitions between its consumers to achieve
	 parallelism.
6. There cannot be more than 1 consumer from a group fetching data from the same
	 partition.

## What are we trying to solve?
1. Given a set of partitions from which we want to consume data from, based on
	 the current system's load, we want to determine:
1. The amount of consumers required.
2. The partitions each consumer is assigned to.
3. To guarantee the rate at which data is consumed from a partition >= to the
	 write speed.
4. Operational cost
5. Rebalance cost

## Problem Formulation
1. We model the problem as a BPP, where the partition's write speeds are the
	 size of the items, and the consumers are the constant size bins.
2. This provides with a variant of the BPP, which has not yet been studied,
	 where the item sizes and their bin assignments can vary over time.
3. Additional rebalance cost since items can be rebalanced between consumers.

# BPP ## Temporal BPP
1. Virtual Machine Placement Problem
2. Paper where all tasks arrive at the same time.
3. Paper where the tasks have differing start times.

## Gaps in the Literature Since this is a novel Bin packing variant, there is a
lack of research over the item reassignment problem which has associated a
rebalance cost.  Lack of algorithms that take the rebalance cost into account.

# Methodology
1. Brief overview of the system and a description of what each component is
	 responsible for doing.
2. Monitor determines the size of the items by providing with the write speed of
	 each partition of interest
3. controller computes the # of consumers and each of their assignments.
4. The consumers fetch data from the topics to insert into the datalake.

## Consumer
1. The consumer process was tested in multiple testing scenarios which always
	 required it to be functioning at it's max consumption rate, and provided the
	 following results, we approximate the consumer to a single bin size variant
	 of the BPP.

## Monitor
1. When testing the speed measurements provided by the monitor process, since we
	 are performing the average over the last 30s, it takes this process 30s to
	 converge to a step input. 
2. Due to the average it filters out the noise in scenarios where there are
	 sudden bursts of data as can be seen in the second figure.

## Controller
1. Reads the last measurement provided by the monitor
2. Mention each of the transitions towards the reassign algorithm
3. computes the group's new assignment based on the BP heuristics.
4. Rscore

### Algorithms Present the differences in the sorting strategy Show assignment
strategy
- Try to insert into any existing consumer If none available fit into current
- consumer If no longer fit, save for later
Show final phase of the Modified algorithms

# Integration

# Conclusions and Future Work
