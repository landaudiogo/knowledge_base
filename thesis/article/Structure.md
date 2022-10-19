# Introduction
Where this problem commonly appears in businesses: 
Messages have to be consumed with a certain order, and achieving more parallelism would mean that the messages would be consumed out of order.

The probelm involves load balancing the different queues between the different consumers, which implies that the parallelism is related to the number of consumers assigned to the differente queues, and which consumer is assigned the queues.

The costs to take into account when solving the problem are the operational cost (the number of consumer instances), and the rebalance cost, associated with reassigning a queue from one consumer to another.

Provided the write rate of each queue, and the throughput capacity of each consumer, we solved this problem using a variation of the Bin Packing Problem.

BPP has been extensively reviewed as seen in \ref{...}, but due to the dynamic nature of the write speed to each queue, the time dimension has to... \ref{..., ...} have reviewed the BPP while taking time into account, and similar to their analysis, new items can arrive at different time instants, but what these problems do not take into account is the fact that items can change in size, and that they can be reassigned from one consumer to another. 

An optimal solution at a time instant, after the partition write speeds change, might not be optimal in another instant requiring a new computation of the optimal assignment of the BPP. 

This also means that items can be reassigned from one consumer to another, but because there cannot be 2 consumers reading data from the same partition at the same time, there is a rebalance cost to take into account. This cost is related to the fact that to reassign a partition from a consumer to another, ...

This problem is strongly related to the inner working of Kafka's consumption design, where queues represent the partitions in a topic, and a set of consumers that split the load between each other to parallelize the work, would represent a group of consumers. 

RabbitMQ provides a different way to parallelize tasks beteween different consumers, but to guarantee message order consumption, there can only be one consumer assigned to a single queue, therefore arriving at the same situation as described above.

This model is not only applicable to the message broker environment.
Although a different application of this BPP model, Kubernetes would also benefit of using this BP model, where the bins are the nodes, and pods are the items that are assigned nodes. Since the pods consume node resources, and the amount of resources being consumed changes with time, to determine the amount of nodes required to host a set of pods, this BPP model would fit this application almost perfectly, with the exception that the nodes don't all have the same size, but a variable size adding another variation to the BPP.

# Problem Formulation
Extract the text from the thesis.

# Related Work
Extract the data from the thesis.

# Algoritmo
## Rscore
## Modified Any Fit

# Aplicação no Kafka
## Monitor
## Consumidor
## Controlador

# Experimentação
## Metricas
## Geração de dados
## Resultados Algoritmos
## Resultados Aplicação Kafka


# Conclusão