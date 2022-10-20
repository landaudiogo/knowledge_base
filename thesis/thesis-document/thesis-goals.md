# Introduction

1. What is the scientific context of the problem?
2. What are we trying to solve?
3. Describe the problem scientifically, and the company that proposed the problem in the first place.
4. What is the motivation to solving the problem? What do we gain by doing this? How did the problem arrise?
5. What are the contributions provided by the work developed? What can the algorithm do? What can the system do as a whole?

# Infrastructure

1. Describe Distributed Architectures.
2. Describe communication between microservices.
3. Present Message brokers.
4. Kafka In depth
5. Kubernetes in depth
6. What do both these systems have that the autoscaler requires to provide with an autoscaling group of consumers?


# Bin Packing Problem 

1. Describe the bin packing problem. 
2. Describe some linear programming solutions.
3. Describe heuristics that solve the problem. 
4. Present recent applications of the bin packing problem in the industry.


# Consumer Group Autoscaler

1. Present the systems architecture
2. Briefly describe what each component of the system has to do.
3. Describe the Monitor Process
4. Describe the Consumer Process
5. Describe the Controller Process. 

# Integration

1. End to end description of the events that occur to scale the system to the current load. 
2. Detailed description of what is happening in each event
3. Discuss how the response time is acceptable or not. For this we could use both values of the consumer_capacity and algorithm_capacity to determine whether the system is scaling in time to have seamless response to the current load of the system.
4. What would happen if the autoscaler hadn't scaled? 

# Conclusion

1. Summarize the findings. 
2. Describe future work in each component.