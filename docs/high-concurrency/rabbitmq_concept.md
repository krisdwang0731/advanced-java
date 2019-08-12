# RabbitMQ Concepts
## RabbitMQ Exchanges, routing keys and bindings
Messages are not published directly to a queue, instead, the producer sends messages to an exchange. Exchanges are message routing agents, defined per virtual host within RabbitMQ. An exchange is responsible for the routing of the messages to the different queues. An exchange accepts messages from the producer application and routes them to message queues with help of header attributes, bindings, and routing keys.

A **binding** is a "link" that you set up to bind a queue to an exchange.

The **routing key** is a message attribute. The exchange might look at this key when deciding how to route the message to queues (depending on exchange type).

Exchanges, connections, and queues can be configured with parameters such as durable, temporary, and auto delete upon creation. Durable exchanges survive server restarts and last until they are explicitly deleted. Temporary exchanges exist until RabbitMQ is shut down. Auto-deleted exchanges are removed once the last bound object unbound from the exchange.

In RabbitMQ, there are four different types of exchange that route the message differently using different parameters and bindings setups. Clients can create their own exchanges or use the predefined default exchanges, the exchanges created when the server starts for the first time.

![rabbitmq routing keys](/docs/high-concurrency/images/exchanges-bidings-routing-keys.png)

### Standard RabbitMQ message flow
* The producer publishes a message to the exchange
* The exchange receives the message and is now responsible for the routing of the message
* A binding has to be set up between the queue and the exchange. In this case, we have bindings to two different queues from the exchange. The exchange routes the message in to the queues.
* The messages stay in the queue until they are handled by a consumer
* The consumer handles the message.

### Exchange types

#### Direct Exchange
A direct exchange delivers messages to queues based on a message routing key. The routing key is a message attribute added into the message header by the producer. The routing key can be seen as an "address" that the exchange is using to decide how to route the message. **A message goes to the queue(s) whose binding key exactly matches the routing key of the message** .

The direct exchange type is useful when you would like to distinguish messages published to the same exchange using a simple string identifier.

The default exchange AMQP brokers must provide for the direct exchange is "amq.direct".

Imagine that queue A (create\_pdf\_queue) in the image below (Direct Exchange Figure) is bound to a direct exchange (pdf\_events) with the binding key pdf\_create. When a new message with routing key pdf\_create arrives at the direct exchange, the exchange routes it to the queue where the binding\_key = routing\_key, in the case to queue A (create\_pdf\_queue).

![rabbitmq direct exchange](/docs/high-concurrency/images/direct-exchange.png)

**SCENARIO 1**   
* Exchange: pdf\_events.  
* Queue A: create\_pdf\_queue.  
* Binding key between exchange (pdf\_events) and Queue A (create\_pdf\_queue): pdf\_create.  
**SCENARIO 2**   
* Exchange: pdf_events.  
* Queue B: pdf\_log\_queue.  
* Binding key between exchange (pdf\_events) and Queue B (pdf\_log\_queue): pdf\_log.  



