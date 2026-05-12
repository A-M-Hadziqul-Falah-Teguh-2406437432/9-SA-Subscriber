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

## Answer for Slow Subscriber Commit

### Analysis of Queued Messages

![Slow subscriber queue](screen-captures/slow-subscriber.png)

The presence of 15 messages in the queue is a direct consequence of running the publisher program three times in quick succession while the subscriber was slowed down. According to the tutorial's logic, each individual run of the publisher sends exactly 5 events (for users Amir, Budi, Cica, Dira, and Emir) to the message broker. Because the subscriber's code was modified to include a 1-second delay (`thread::sleep`) for every message it handles, it can no longer process events at the same speed they are being produced.

As the publisher finishes its three runs almost instantly, it "fires and forgets" a total of 15 events into the `user_created` channel. The RabbitMQ broker then stores these pending events in a queue, which acts as a back pressure point to prevent the slow subscriber from being overwhelmed or causing a system crash. This build-up demonstrates a core benefit of event-driven architecture: the system remains responsive because the producer does not have to wait for the consumer to finish, allowing the broker to buffer the workload. Consequently, the subscriber will slowly process each of the 15 messages one by one until the queue is eventually cleared.

## Reflection: Scaling with Multiple Subscribers

![Three subscribers queue](screen-captures/three-subscribers.jpeg)

The visual evidence in the RabbitMQ dashboard shows that the spike of the message queue is reduced much quicker because the system is now utilizing parallel processing. In the previous simulation, a single subscriber with a 1-second delay created a significant backlog (a queue of 15-20 messages) because it could only process one event at a time. By spawning three subscribers, the message broker is able to distribute the workload among all three available consumers, allowing them to process events concurrently. For example, while one subscriber is busy waiting on the simulated delay for the first message, the broker can immediately hand off the second and third messages to the other two idle subscribers.

This demonstrates a core advantage of Event-Driven Architecture: the ability to achieve high scalability and responsiveness by simply increasing the number of subscribers to handle high demand. It effectively prevents the "bottleneck trap" where a slow component causes the entire system to lag or crash. The RabbitMQ broker acts as a back-pressure point, holding the messages safely in the queue until one of the three consumers is ready to acknowledge them, ensuring data consistency while maximizing throughput.

### Potential Code Improvements

Looking at the current implementation of the publisher and subscriber, several areas could be improved to reach production quality:

- **Library Versions**: The current `Cargo.toml` specifies an older version of the `crosstown_bus` library (`0.5.3` for subscriber and `0.5.0` for publisher). Upgrading to the latest version would be a primary improvement to ensure better compatibility and security features.
- **Hardcoded Configurations**: Both programs have the AMQP connection string and event names hardcoded in the `main.rs` file. These should be moved to an environment variable or a configuration file (like a `.env` file) to allow the programs to run in different environments (local vs. cloud) without code changes.
- **Error Handling**: The current code frequently uses `.unwrap()`, which causes the program to crash immediately if a connection fails. A more robust implementation would use proper Rust error handling (like `match` or `if let`) to attempt reconnections or log errors gracefully if the RabbitMQ broker becomes temporarily unavailable.
- **Dynamic Publisher Data**: The publisher currently sends a static list of five users. This could be improved by allowing the publisher to accept CLI arguments or read from a data source, making it a more versatile tool for testing system load.
