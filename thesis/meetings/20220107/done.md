# General

Remove any mentions of HUUB outside of the introduction. Whatever mentions of the company should be removed and the problem should be within the scientific context. ✅

In all references, add a description of what we are referencing (e.g. in figure 3.1., in section 3.3. ...) ✅

When mentioning a time interval in math notation ([0.01, 2]s) write the "s" as seconds to make it clearer. ✅

Verify the caption for all the figures ✅

Try to reformulate any single line paragraphs. ✅

# Introduction

What is the scientific scope of the problem? What are we trying to solve? ✅

Provide some context with the problem, it was proposed by the company to solve the issues related to QoS and operational cost. ✅

What are the motivations of the problem? What do we want to do with the solution? Process big volumes of data, in a more economic fashion, cost optimization, QoS. ✅

Remove comply with microservices within the system requirements. ✅

What are the contributions provided by this work? What was done? What can the algorithm do? ✅

# Bin Packing Problem

Change the Chapter title from literature review to Bin Packing Problem. ✅

Within the bin packing problem context, what recent applications in the industry have made use of the algorithms? Add more articles that demonstrate this problem is current. ✅

# Infrastructure

No need to include so much information in microservices and the monolith architecture, add just a simple introduction as to what these concepts are, and why message brokers and containerized applications help these kind of systems.✅

Explain Kafka✅

Explain Kubernetes✅

What do we need from both Kafka and Kubernetes which enables us to build the system?✅

We assume that we are always capable of creating new consumer resources, and this is achieved due to the autoscaler feature of the kubernetes cluster. ✅

# Consumer Group Autoscaler

Add a diagram of the system and briefly explain each of the components within the diagram before entering into any detail in the development. ✅

What is the goal of the consumer group autoscaler? What are we delivering and solving? ✅

change topic names: 
- data-engineering-controller: consumer.metadata ✅
- data-engineering-monitor: monitor.writeSpeed ✅

The first occurance of BATCH_BYTES and WAIT_TIME_SECS should be explained as soon as they appear. ✅

Invert the image that explains the bin assignment in one of the existing approximation algorithms. ✅

When displaying the data required to compute the Rscore, remove the $R_i$. ✅

Explain the Rscore in a seperate section from the modified algorithms section. Give it more emphasis ✅

Use a section to describe the metrics used to evaluate the performance of the approximation algorithms. ✅

Explain that the initial conditions were changed, and that no significant changes were verified in the data. Therefore the results presented are... ✅

Add a description of delta within the caption and its symbol. If not possible simply add the symbol. ✅

Remove for a random initial speed from the caption of the figures. ✅

Add delta symbol the the ggplot in R. ✅


# Integration

Adicionar uma explicação no inicio ✅

Add a unit to the Event Duration (seconds). ✅

Add delta to the figure of the images ✅

time to reach input production speed ✅

separar as ações que são do monitor e do controlador ✅

Explicar o flow de informação no sistema. consiste neste processo, tem uma primeira fase que pertence ao monitor e depois temos as restantes fases que impactam o controlador + consumidores. ✅

desenhar um fluxograma 
	monitor tira medidas
	controlador determina se ha escalonamento ✅
	
Ter um diagrama que exemplefique as ações que ocorrem no sistema
