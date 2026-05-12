# Answer to Modul Questions

## a. What is AMQP?

**AMQP** stands for **Advanced Message Queuing Protocol**. It is an open-standard application layer protocol for message-oriented middleware that enables different systems and applications to communicate with each other regardless of their platforms.

In the context of this tutorial's broker topology, AMQP is significant because it supports features like programmatic load balancing and monitoring. It achieves this through a distinct separation between an exchange (where the publisher sends data) and a queue (where the subscriber listens for data).

## b. What does `guest:guest@localhost:5672` mean?

This string is a connection URL used by the publisher and subscriber to communicate with the RabbitMQ message broker. It breaks down as follows:

- `amqp://`: This specifies the protocol being used for the connection (Advanced Message Queuing Protocol).
- The first `guest`: This is the username required to authenticate with the RabbitMQ server.
- The second `guest`: This is the password associated with that username.
- `localhost`: This is the network address (host) where the message broker is running. In this tutorial, it refers to your own machine (or the Docker container mapped to it).
- `5672`: This is the specific port number that the RabbitMQ message broker listens on for incoming connections from programs.

_(Note: While port `5672` is for the message broker itself, port `15672` is used separately for the web-based management dashboard.)_
