# Overview

When Controller is starting, it is possible that 1 of 2 things might have happened. 

1. The controller is starting for the first time.
2. The controller is restarting after a crash.

In the first scenario, there isn't much the controller needs to do, as it is literally starting from scratch so it can move on to the next state (State Sentinel).

In the second scenario, the controller might happen to be out of sync with the consumer group. This is problematic because it would imply the controller has a perceived consumer group state which is not the real group's state, leading to wrong delta messages being sent to a consumer. To solve this issue, a synchronization protocol is defined between the controller and the consumers, which has the controller send a query message to all the active consumers with its label, and only after every active consumer has replied to this message with its current state, does the controller leave the synchronization process.

# Implementation

To implement the synchronization protocol, the controller has to go through the following phases: 
1. Verify which deployments are currectly up with its specific label.
2. To each of the consumers represented by the deployment, send out a query state message.
3. If a message is not a response to the query message in partition 0 ignore.
4. If a message is a reponse to the query message, then from the consumer's id in the header, add that assignment to the consumer in the consumer list.
5. After receiving a Query response from all consumers move on to the next state.

Changes: 
1. The controller has to change the change_state code to allow the controller to ignore query responses in that phase as well.

## Consumer

When consuming the metadata, the consumer has to verify if there is any query state command, and in case there is: 
1. Set a flag;
2. Compute the change in state of the previous messages; 
3. Send out its current state; 
4. Unset the flag.

## Kafka

Create a new topic intended for query messages.

## Communication

When the controller wants to request the consumer to send its state, it does so in the data-engineering-query in the same way it sends a command through the data-engineering-controller topic. 

As for the consumer, although the type of message is different, the consumer uses the data-engineering-controller topic to send its current state through partition 0.