# RabbitMQ核心概念
## RabbitMQ Exchanges, routing keys and bindings
Messages are not published directly to a queue, instead, the producer sends messages to an exchange. Exchanges are message routing agents, defined per virtual host within RabbitMQ. An exchange is responsible for the routing of the messages to the different queues. An exchange accepts messages from the producer application and routes them to message queues with help of header attributes, bindings, and routing keys.

A **binding** is a "link" that you set up to bind a queue to an exchange.

The **routing key** is a message attribute. The exchange might look at this key when deciding how to route the message to queues (depending on exchange type).

Exchanges, connections, and queues can be configured with parameters such as durable, temporary, and auto delete upon creation. Durable exchanges survive server restarts and last until they are explicitly deleted. Temporary exchanges exist until RabbitMQ is shut down. Auto-deleted exchanges are removed once the last bound object unbound from the exchange.

In RabbitMQ, there are four different types of exchange that route the message differently using different parameters and bindings setups. Clients can create their own exchanges or use the predefined default exchanges, the exchanges created when the server starts for the first time.

![MacDown Screenshot](docs/high-concurrency/images/exchanges-bidings-routing-keys.png)



