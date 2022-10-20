> Microservices.io Saga Pattern: 
> https://microservices.io/patterns/data/saga.html

>  Distributed Transactions: 
>  https://www.baeldung.com/cs/saga-pattern-microservices

# Overview

When dealing with distributed transactions in a microservices environment, a service might need to communicate with another to guarantee that a transaction is successful.

As an example, in an e-commerce application with the following structure:
- Order Service
- Stock Service
- Billing Service
When creating an order using the Order Service API, this transaction is only successful if there is enough stock, and if the billing is successful. 

# Definition

The Saga pattern is a sequence of local transactions. Each local transaction updates its database, and publishes a message/event to trigger the next local transaction.

If a local transaction fails because it violates a business rule then the saga executes a series of compensating transactions that undo the changes that were made by the preceding local transactions. 

This means that performing the transactions and undoing the transactions is very business logic specific.

## Challenge

ACID: Atomic, consistent, isolated and durable.
The _atomicity_ ensures that all or none of the steps of a transaction should complete. _Consistency_ takes data from one valid state to another valid state. _Isolation_ guarantees that concurrent transactions should produce the same result that sequentially transactions would have produced. Lastly, _durability_ means that committed transactions remain committed irrespective of any type of system failure.

We want to guarantee ACIDity in a distributed microservices environment.

**Problem**: While a transaction is still in its execution, if a service has already persisted its data regarding its transaction, and another service requests for that data, which should it present? The old state or the new state? 

## Choreography


## Orchestration 