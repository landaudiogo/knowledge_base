Acrescentar referencia repositorio


# Monitor

Porque é que se fez o segundo teste, e porque é que os valores foram escolhidos

Falta dar um remate final ao segundo teste

# controller

Rscore, dar uma descrição mais verbal do que esta metrica calcula

adicionar pagina de figura distante.

# Integration

Adicionar uma breve descrição no inicio do capitulo.


# BPP

## Conclusion

Although the temporal aspect of the Bin Packing Problem has already been reviewed, there is a gap when it comes to the possibility of rebalancing the items between the different bin instances. Existing solutions consider a bin packing model where the items' sizes do not change with time, and the assignment is persistent and ephemeral, i.e., the items only have to be assigned for a specified duration. 

Modeling the BPP with items' sizes and consumer assignments that vary over time, presents a new variation of the BPP that has not been studied, and will be one of the contributions of this work. 

In view of the fact that the existing approximation algorithms were developed to solve a single iteration of the BPP, these have to be adapted to the problem at hand, that requires solving a new iteration of the BPP, given an already existing assignment. 

Despite the fact that the adaptation presented in Section \ref{} improves these algorithms' performance with respect to the rebalance cost, there is still room for improvement. The Modified Algorithms presented in Section \ref{} fill this gap, as shown in Section \ref{}



# Conclusion and Future Work

The work developed in this thesis, aims to challenge the existing methods to scale a Kafka consumer group, and how the load should be distributed between the various consumers.

The goal was to deterministically solve the autoscaling problem related to a group of consumers, which we model as a Bin Packing Problem where the items' sizes and bin assignments can change over time. In light of these variations, items' assignments can be rebalanced (bin assignment can change), and therefore we propose the Rscore to account for the rebalance cost, Section \ref{}.

Given the fact that, to the best of our knowledge, it is the first time the BPP is applied in these conditions, there was lack of heuristic algorithms that solve the Bin Packing assignment while considering the Rscore. As such, in Section \ref{} we propose four new heuristic algorithms based on the Rscore which are proven to be competitive solutions in Section \ref{}, when solving the multi-optimization problem.

Furthermore, in Chapter \ref{}, a system was developed to automatically scale a group of consumers and manage the partitions' assignments based on the aformentioned theoretical approach. This system was integrated in the company's infrastructure, and as expected is capable of providing a solution that scales a group of consumers based on the system's current load.

Isto é integras a abordagem numa ferramenta que faz aquilo que te comprometeste a fazer na introdução (event-driven consumer autoscaling)

This system has also been applied in the company's context, and as expected is capable of continuously monitoring and managing a consumer group so as to provide a fully automated scaling solution.

# Abstract

Message brokers provide with asynchronous communication between data producers and consumers in a distributed environment, being Kafka one of the several message broker alternatives. To scale data consumption rate, Kafka has Consumer Groups, which is an abstraction that allows to parallelize tasks between consumers in a group. This abstraction presents new concerns that depend on the broker's current load, which include: determining the number of consumers required; determining the partition assignment between the consumers in a group in order to guarantee that the consumption rate of each partition is not less than their respective production rate. Additionally considering the load varies with time, there is an increasing need to find autoscaling solutions to reduce operational cost, while guaranteeing that all data is being consumed within acceptable time after it has been produced.

As such, this problem is modeled as a new variation of the Bin Packing Problem where the bins are consumers of a consumer group, and the weights are the partitions and their respective write speeds. Due to the varying load applied to Kafka brokers, the weights change in size over time. Another variation is the fact that item assignments are not persistent, and therefore can be rebalanced between consumers in different time instants. This adds another concern related to the fact that while a partition is being rebalanced, data is not being consumed from it (rebalance cost). We propose a new metric to account for this cost (Rscore), and present four new heuristic algorithms based on the Rscore, three of which prove to be a competitive alternative when compared to existing heuristic algorithms with respect to the multi-objective optimization problem of minimizing both the number of consumers and the rebalance cost.

To deliver a fully automated solution to the autoscaling problem, using the aforementioned theoretical approach, we also present a system that is capable of responding to wide ranges of current system's loads to manage a group of consumers while reducing the operational cost as compared to exisiting solutions.



# Agradecimentos

I would first like to start off by thanking my supervisor Professor Jorge Barbosa, who made himself available to not only clarify my questions, but also for his valuable guidance in choosing between development alternatives.

I would like to thank my work colleagues, and especially Fernando Gomes. The freedom in solution approach and the tools you made available brought my work to a higher level. 

I would also like to thank my friend, Dr. Xavier Andrade for having extensively reviewed this document and provided valuable insights.

To my parents, Maria João Carrapato and Jorge Landau, thank you for your patient and unconditional support, and to my sister, Beatriz Landau, thank you being my source of inspiration. 