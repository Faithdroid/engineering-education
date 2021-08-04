[RabbitMQ](https://www.rabbitmq.com/) is an open-source distributed message queue that supports many communication protocols. It is a message broker, and it receives from senders (producer) and forwards messages to receivers (consumer). These messages are maintained in a queue. A queue is, in essence, a huge message buffer. Many producers can submit messages to queues, and many consumers can attempt to get messages from these queues.

This guide will discuss RabbitMQ broker in detail and then dockerized a RabbitMQ instance using Docker containers.

### RabbitMQ architecture and essential components

RabbitMQ architecture starts with a producer and consumer. As we said, the producer is the application or service which is sending messages. The consumer is the application or service which will be receiving the messages. So instead of the producer calling consumer service directly, we sent up another server that acts as the broker between the two. In this case, we will have a RabbitMQ server.

There are significant components that facilitate RabbitMQ message brokerage between the consumer and the producer. These are

- A producer (publisher) - an application or a service that publishes a message to a RabbitMQ server.
- Exchange - it acts as a message filter. It receives messages from producers and publishes them in queues using routing rules to send them to a consumer. Take a case of the typical telephone Exchange for call routing. When the messages arrive at the telephone exchange, the Exchange determines the target consumer for a particular message and routes accordingly.
- Queue - a RabbitMQ buffer that stores published messages.
Consumer - application or a service that reads messages from the RabbitMQ server queue.

![rabbitmq-components](/engineering-education/what-is-rabbitmq-dockerize-a-rabbitmq-instance-using-docker-containers/rabbitmq-components.png)

[Image source: RabbitMQ Docs](https://www.rabbitmq.com/tutorials/amqp-concepts.html)

Here is a simple scenario of a user sending a request (message) to create a PDF web application (producer)

![rabbitmq-message-exchange-example](/engineering-education/what-is-rabbitmq-dockerize-a-rabbitmq-instance-using-docker-containers/rabbitmq-message-exchange-example.png)

[Image source: CloudAMQP](https://www.cloudamqp.com/blog/part1-rabbitmq-for-beginners-what-is-rabbitmq.html)

For the four components to effectively communicate with each other, the following properties will be essential in determining the message destination.

- Message - This is the data or the information being exchanged by the producers and the consumers. Each message produced by a publisher had two main parts. That is;

  - The message header - defines the message header attribute.
  - The message body - the message content being shared.

Depending on the messaging sharing pattern, a message can have properties such as published acknowledged, unacknowledged, redelivered, ready and received.

- Routing Key - a RabbitMQ can have many message queues. In this case, each produces message needs to have a virtual address key that the Exchange will use to determine which queue the message should be routed to. This is called the routing key.

- Binding (Binding key) - links the queue to an exchange. Exchange uses binding and routing keys to determine which message belongs to which queue.

![rabbitmq-message-exchange-example](/engineering-education/what-is-rabbitmq-dockerize-a-rabbitmq-instance-using-docker-containers/rabbit-message-exchange.png)

[Image source](https://www.javainuse.com/messaging/rabbitmq/exchange)

- Exchange types - There are four messages Exchange type

1. Topic - route messages based on patterns in the routing key. For example, distribute data specific to a geographical location depending on another factor such as point of sale.

![rabbitmq-topic-exchange](/engineering-education/what-is-rabbitmq-dockerize-a-rabbitmq-instance-using-docker-containers/rabbitmq-topic-exchange.png)

[Image source](https://www.javainuse.com/messaging/rabbitmq/exchange)

2. Fanout - routes messages to all the queues that are bound to it, and the routing key is ignored. A good example of Fanout Exchange is a Group chat. RabbitMQ will use this scenario to distribute messages to the different participants of that specific group.

![rabbitmq-fanout-exchanges](/engineering-education/what-is-rabbitmq-dockerize-a-rabbitmq-instance-using-docker-containers/rabbitmq-fanout-exchanges.png)

[Image source](https://lostechies.com/derekgreer/2012/03/28/rabbitmq-for-windows-exchange-types/)

3. Direct - delivers messages to queues based on a message routing key. This means the queue has to be the same as the routing key. It is applied when sending a message to individuals—for example, sending notifications to individuals in a specific geographical location.

![rabbimq-direct-exchange](/engineering-education/what-is-rabbitmq-dockerize-a-rabbitmq-instance-using-docker-containers/rabbimq-direct-exchange.png)

[Image source](https://lostechies.com/derekgreer/2012/03/28/rabbitmq-for-windows-exchange-types/)

4. Headers - it ignores the routing key and looks at the headers that were sent with the message.

![rabbitmq-header-exchange](/engineering-education/what-is-rabbitmq-dockerize-a-rabbitmq-instance-using-docker-containers/rabbitmq-header-exchange.png)

[Image source](https://www.javainuse.com/messaging/rabbitmq/exchange)

### How a RabbitMQ works

RabbitMQ is used as a message broker to implement advanced messaging queuing protocol (AMQP). A complete AMQP has three main components a broker, a consumer, and a producer. When exchanging messages between the producers and the consumers, messages are not sent directly to Queues. The first pass through an Exchange (in the broker) that redirects them to their destination Queues.

A producer will produce messages and then publish them in an Exchange. Each message will be sent with a routing key. Now the Exchange ensures this message ends up in the correct queue. For Exchange to ensures the message is the right Queues, it depends on several things, including the Exchange type, which specifies the number of routing rules, routing keys, and header attributes.

think of an Exchange as a mail delivery person ensuring that the message ends up in the

In this case, he will need the right address to ensure the mail are in the correct hands. In this example, routing rules, routing keys, and header attributes act as addresses for messages. RabbitMQ uses them as the bindings rules for the different Exchange types to determine which message links to which queues depending on the queue binding key. Once a message is in the right queue, a consumer can request the message he wants to consume.

In a short summary;

- A produce publishes messages to an Exchange.
- Binding rule (routing key) connects an Exchange with a queue using the binding key.
- A consumer receives messages from the queue.
- A consumer sends a message back to the broker and informs the server it got the message. Thus the broker can delete that message from the queue.

![rabbitmq-message-broker](/engineering-education/what-is-rabbitmq-dockerize-a-rabbitmq-instance-using-docker-containers/rabbitmq-message-broker.png)

[Image source: Medium](https://medium.com/ryans-dev-notes/learning-rabbitmq-3f59d11f66b4)

### Why do we need RabbitMQ (a message broker)

Let's say you have a microservice application of service to services communication, i.e., service A and service B. In this case, service A will make a direct call to service B synchronous. In a synchronous scenario service, A has to wait for service A to respond. This means if service A takes a long time to respond, this connection can potentially time out. For a successful response, service A has to retry multiple times. In case services B died or crashed, communication is lost, and no messages are exchanged.

With a RabbitMQ server in place, service A is only assigned to produced messages, and the RabbitMQ server will save them in a queue. So Whenever service A wants to reach out to service B, it does not directly send a message to service B. Whenever service B wants to get the message sent by service A, it will go over to the queue and get this message. Since this message is published and saved, if B fails to get the message for the first time, it will retry and get the message again since the message was not lost. The advantage of the message queues is that.

- One service is not directly dependent on another service.
- You can scale up the producer to produce more messages.
- You can scale up multiple consumers to access the messages directly from the queue.
- Adds resilience to your application. If one service fails, the other service will not be directly affected.

### Set up RabbitMQ using docker containers

Now that we understand RabbitMQ, how it works, and its importance, let's set up RabbitMQ using Docker containers. We will set up and start a simple RabbitMQ instance using Docker on our local computer along with the RabbitMQ management UI and basic Administrator functions.

I am assuming that you already have basic knowledge of Docker and how to use docker-compose. Also, ensure you have Docker installed on your local computer. You can download it [here](https://www.docker.com/products/docker-desktop). Finally, run `docker version` to check if Docker was successfully installed.

We will use docker compose.yml to set up a RabbitMQ instance. So go ahead and create a docker-compose.yml file in your desired directory. This is how we will set up our docker-compose file.

- First, define the docker-compose version you want to run.

```yml
version: "3.8"
```

- Add the RabbitMQ service and start to add the RabbitMQ properties and environment that you want your container to run.

```yml
services:
  rabbitmq3:
```

- Set the container name.

```yml
container_name: "rabbitmq"
```

- Add the container image to pull from the Docker hub. There are different tags available for the RabbitMQ image. Here we are pulling the image with the tag management because we want to access the RabbitMQbe management UI. This will enable you to manage RabbitMQ queues, channels, queues, and Exchange.

```yml
image: rabbitmq:3-management
```

- Add RabbitMQ container environments. We are setting the RabbitMQ username and password that we will use to log in and access the RabbitMQbe management UI.

```yml
environment:
    - RABBITMQ_DEFAULT_USER=myuser
    - RABBITMQ_DEFAULT_PASS=mypassword
```

- Define the container ports that will run on Docker and exposes the container externally. This port will expose its container so that we can access it over a browser.

```yml
ports:
    # AMQP protocol port
    - '5672:5672'
    # HTTP management UI
    - '15672:15672'
```

Now your docker-compose.yml file is set to spin up this RabbitMQ instance with Docker containers.

```yml
version: "3.8"
services:
    rabbitmq3:
        container_name: "rabbitmq"
        image: rabbitmq:3.8-management-alpine
        environment:
            - RABBITMQ_DEFAULT_USER=myuser
            - RABBITMQ_DEFAULT_PASS=mypassword
        ports:
            # AMQP protocol port
            - '5672:5672'
            # HTTP management UI
            - '15672:15672'
```

Let's now test the compose file by running the command `docker-compose up`. This will download and set up the Rabbit MQ container.

![rabbitmq-docker-compose](/engineering-education/what-is-rabbitmq-dockerize-a-rabbitmq-instance-using-docker-containers/rabbitmq-docker-compose.png)

If you open your Docker engine, you will see the RbbitMQ container is set and running.

![rabbitmq-docker-container](/engineering-education/what-is-rabbitmq-dockerize-a-rabbitmq-instance-using-docker-containers/rabbitmq-docker-container.png)

If you open `http://localhost:15672/` on a browser, you will be able to access the management Ui, and now you can log in using the docker-compose set username and password.

![rabbitmq](/engineering-education/what-is-rabbitmq-dockerize-a-rabbitmq-instance-using-docker-containers/rabbitmq.png)

And now you can see the RabbitMQ instance is up and running.

![rabbitmq-instance](/engineering-education/what-is-rabbitmq-dockerize-a-rabbitmq-instance-using-docker-containers/rabbitmq-instance.png)

### A simple test case

Now you can start adding queues and publishing messages. Let's test that. Go to the tab queue and add a new queue; call it a `test-queue`.

On the Exchange tab, add a new exchange call it `test-exchange`. Select exchange type as direct.

![test-exchange](/engineering-education/what-is-rabbitmq-dockerize-a-rabbitmq-instance-using-docker-containers/test-exchange.png)

Click and open the added `test-exchange`.

![test-exchange2](/engineering-education/what-is-rabbitmq-dockerize-a-rabbitmq-instance-using-docker-containers/test-exchange2.png)

Now we will create a binding to the newly created queue. Scroll down and Add binding from this Exchange. Here we will add the name of the queue we have created. Also, add the routing key as `green`.

![binding](/engineering-education/what-is-rabbitmq-dockerize-a-rabbitmq-instance-using-docker-containers/binding.png)

![queue-binding](/engineering-education/what-is-rabbitmq-dockerize-a-rabbitmq-instance-using-docker-containers/queue-binding.png)

At this point, the pattern you have created is similar to this one. Just as we have explained earlier. Now this time with the actual implementation.

![direct-exchange-queue](/engineering-education/what-is-rabbitmq-dockerize-a-rabbitmq-instance-using-docker-containers/direct-exchange-queue.png)

Let's test this scenario by publishing a message to the `test-exchange`. Here we are testing the direct Exchange use case. So on the publish message section, add the routing key as `white` and add a message at the Payload section.

![publish-a-message](/engineering-education/what-is-rabbitmq-dockerize-a-rabbitmq-instance-using-docker-containers/publish-a-message.png)

The Direct Exchange concept shows that messages are delivered to the queues based on a message routing key. In this case, the `test-queue` is assigned messages with the `green` routing key. Using `white` as the routing key will end up with this message not being routed to this queue. So, let's try and publish this message with `white` as the routing key.

![message-not-routed](/engineering-education/what-is-rabbitmq-dockerize-a-rabbitmq-instance-using-docker-containers/message-not-routed.png)

You can see that this message got published but not routed to the `test-queue`. Now try the same using `green` as the routing key.

![message-routed](/engineering-education/what-is-rabbitmq-dockerize-a-rabbitmq-instance-using-docker-containers/message-routed.png)

This message goes published and delivered to the `test-queue`. If you head over to tab queue and select `test-queue`, go to get messages, and click the get messages button, you can see the message was routed to the queue.

![message](/engineering-education/what-is-rabbitmq-dockerize-a-rabbitmq-instance-using-docker-containers/message.png)

This is a straightforward test case. You can go ahead and try using the other Exchanges scenarios.