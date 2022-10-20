# Proposed Solution

There are 2 kinds of errors that might occur when processing messages: 
1. The process can fail to deserialize the record.
2. The process can fail to upload the message to GCP.

In the first scenario, the consumer should send this message to another internal topic, with additional context in the header which indicates how the message is to be deserialized.

In the second scenario, a new file structure is proposed to allow the Data Engineering team to better understand from which topic the message originated.

# Changes

## Failed Deserialization
In case a message cannot be deserialized, this is where the consumer is sending the message to the bucket (utils.py):

![[Pasted image 20220509154056.png]]

Where the processes catches the exception, another function has to be called so the consumer can send the failed message to the dead letter queue, adding the topic string to the msg header.

## Failed Upload
When the consumer fails to upload the bucket, this is where the csv is being sent to the bucket (bq_helper.py):

![[Pasted image 20220509154512.png]]

1. Refactor the code
2. Instead of sending the message to the bucket, send it to a folder specific to the topic the message was consumed from.
3. As a last resort, if the message cannot be uploaded to GCS, send the message to the dead letter queue as well

# Development

Another consumer will have to be created, which is responsible for consuming the messages from the queue. 
Provided a map that contains as key the topic string, and as value the TopicClass used to deserialize the message, the messages in the dead letter queue are deserialized.


# Tasks

## Failed Upload
1. Find out how to upload file to folder structure in GCS
2. Upload a csv file to the folder without a schema
3. Insert the rows in the file into a table.

## Failed Deserialization
1. Create internal topic with a single partition
2. Create producer
3. Add new field in the message header
4. Produce the message to the internal topic
5. (Optional) Add another field which contains the error
6. Change async functions to other functions other than replacing them


## Development

1. Create a map for a topic and their deserializer.
2. Consume the messages from the internal topic.
3. Deserialize the message
5. If successful, commit the msg offset.
6. If unsuccessful, request user interaction, showing the error and more information.

```python
msg_list = consumer.consume()
for msg in msg_list: 
	deserializer = msg.headers["topic_name"]
	deserialized_msg = deserialize(msg, deserializer)
	if successful: 
		send message to bigquery
		commit offset
	else: 
		show user message headers
		deserializer_string = prompt user for input 
		deserializer = import(deserializer_string)
		deserialize(msg, deserializer)
		persist deserializer_string in topic_map
```


# Testing

When performing the unit tests, run mock tests that raise exceptions when 