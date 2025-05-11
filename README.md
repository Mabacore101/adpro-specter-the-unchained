## 1. What is AMQP?
AMQP (Advanced Message Queuing Protocol) is a standard for message-oriented middleware that enables systems to communicate asynchronously by sending messages through brokers like RabbitMQ. It helps build scalable, decoupled, and reliable architectures by allowing producers to send messages without needing to know who the consumers are. With features like message queuing, acknowledgments, routing, and transactions, AMQP is well-suited for microservices and event-driven systems. It's like dropping a task into a shared to-do list, and one team member picks it up to complete — you don’t know who, but it gets handled.

## 2. What it means? guest:guest@localhost:5672 , what is the first guest, and what is the second guest, and what is localhost:5672 is for?
a. guest:guest — This part represents the username and password for authenticating with the RabbitMQ broker. The first guest is the username, and the second guest is the password. This is the default username/password for RabbitMQ unless changed.

b. localhost:5672 — localhost is the host address where RabbitMQ broker is running. In this case, it is our local machine. While 5672 is the port number where RabbitMQ broker is listening for AMQP conncections. 

![alt text](SubscriberSS1.png)
- The large number of queues is caused by the high volume of messages sent by the publisher to the message broker, while not all of them have been processed by the subscriber, which is slower than the publisher. As a result, all unprocessed messages are stored in the queue. On my machine, the total queue count was 11. This happens because the subscriber takes longer to handle each event in the message queue, leading to a buildup of messages since the publisher publishes messages faster than the subscriber can process them.

![alt text](SubscriberSS2.png)
![alt text](SubscriberSS3.png)
![alt text](SubscriberSS4.png)
![alt text](SubscriberSS5.png)
- We can observe that the high message spike has decreased. This is because each subscriber receives a unique set of data when the publisher sends a large amount of data to the queue. Each subscriber acts like a separate application, so the process of retrieving data from the message queue is distributed. Once data is fetched from the queue by a subscriber, the message is removed and cannot be accessed by other applications.

- From the publisher and subscriber code, there are several things to improve:
## A. Publisher
1. Error Handling:
The use of _ = to ignore the result of publish_event discards any potential errors. This will make debuging and detecting failed message deliveries harder. We could implement proper error handling like this:
```rust
if let Err(e) = p.publish_event(...) {
    eprintln!("Failed to publish event: {:?}", e);
}
```

## B. Subscriber
1. Durability and Reliability:
The durable: false setting means messages won't persist if RabbitMQ restarts. In a production scenario, this should be true to ensure message durability.

2. Logging & Monitoring:
Instead of using println!, a proper logging library (e.g., log, env_logger) could help categorize logs (info, warn, error) and enable better debugging or monitoring.