
![[Pasted image 20260623191149.png]]

### Feedback

We introduced a Kafka message queue for buffer, this basically turns process into eventual consistency at this point. The problem is now we can't reliably ack the sender the message is sent. Yes the message is in Kafka, but if network blips then message won't be persisted. So for reliable ack to send, we need to put Message DB in front of Kafka. And because now we are writing to DB and pushing to Kafka. Outbox pattern (Introduce an outbox table in the db) to wrap it in one transaction. And have CDC (Change Data Capture) to sync event to Kafka.