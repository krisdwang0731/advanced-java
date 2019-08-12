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

#### Default Exchange

The default exchange is a pre-declared direct exchange with no name, usually referred by the empty string "". When you use the default exchange, your message is delivered to the queue with a name equal to the routing key of the message. Every queue is automatically bound to the default exchange with a routing key which is the same as the queue name.  

#### Topic Exchange

Topic exchanges route messages to queues based on wildcard matches between the routing key and something called the routing pattern specified by the queue binding. Messages are routed to one or many queues based on a matching between a message routing key and this pattern.

The routing key must be a list of words, delimited by a period (.), examples are agreements.us and agreements.eu.stockholm which in this case identifies agreements that are set up for a company with offices in lots of different locations. The routing patterns may contain an asterisk (\" * \") to match a word in a specific position of the routing key (e.g., a routing pattern of "agreements.*.*.b.*" only match routing keys where the first word is "agreements" and the fourth word is "b"). A pound symbol (\" # \") indicates match on zero or more words (e.g., a routing pattern of "agreements.eu.berlin.#" matches any routing keys beginning with "agreements.eu.berlin").

The consumers indicate which topics they are interested in (like subscribing to a feed for an individual tag). The consumer creates a queue and sets up a binding with a given routing pattern to the exchange. All messages with a routing key that match the routing pattern are routed to the queue and stay there until the consumer consumes the message.

The default exchange AMQP brokers must provide for the topic exchange is "amq.topic". 

![rabbitmq topic exchange](/docs/high-concurrency/images/topic-exchange.png)

**SCENARIO 1**   
The image to the right show an example where consumer A is interested in all the agreements in Berlin.  
* Exchange: agreements  
* Queue A: berlin\_agreements  
* Routing pattern between exchange (agreements) and Queue A (berlin_agreements): agreements.eu.berlin.#  
* Example of message routing key that matches: agreements.eu.berlin and agreements.eu.berlin.headstore 
  
**SCENARIO 2**   
* Consumer B is interested in all the agreements.   
* Exchange: agreements  
* Queue B: all\_agreements  
* Routing pattern between exchange (agreements) and Queue B (all\_agreements): agreements.#  
* Example of message routing key that matches: agreements.eu.berlin and agreements.us

**SCENARIO 3**
Consumer C is interested in all agreements for European head stores.

* Exchange: agreements
* Queue C: headstore\_agreements
* Routing pattern between exchange (agreements) and Queue C (headstore\_agreements): agreements.eu.*.headstore
* Example of message routing keys that will match: agreements.eu.berlin.headstore and agreements.eu.stockholm.headstore

#### Fanout Exchange
The fanout copies and routes a received message to all queues that are bound to it regardless of routing keys or pattern matching as with direct and topic exchanges. Keys provided will simply be ignored.

Fanout exchanges can be useful when the same message needs to be sent to one or more queues with consumers who may process the same message in different ways.

The image to the right (Fanout Exchange Figure) shows an example where a message received by the exchange is copied and routed to all three queues that are bound to the exchange. It could be sport or weather news updates that should be sent out to each connected mobile device when something happens.

The default exchange AMQP brokers must provide for the topic exchange is "amq.fanout".

![rabbitmq fanout exchange](/docs/high-concurrency/images/fanout-exchange.png)

#### Headers Exchange
Headers exchanges route based on arguments containing headers and optional values. Headers exchanges are very similar to topic exchanges, but it routes based on header values instead of routing keys. A message is considered matching if the value of the header equals the value specified upon binding.

A special argument named "x-match", which can be added in the binding between your exchange and your queue, tells if all headers must match or just one. Either any common header between the message and the binding count as a match, or all the headers referenced in the binding need to be present in the message for it to match. The "x-match" property can have two different values: "any" or "all", where "all" is the default value. A value of "all" means all header pairs (key, value) must match and a value of "any" means at least one of the header pairs must match. Headers can be constructed using a wider range of data types - integer or hash for example instead of a string. The headers exchange type (used with the binding argument "any") is useful for directing messages which may contain a subset of known (unordered) criteria.

The default exchange AMQP brokers must provide for the topic exchange is "amq.headers".

* Exchange: Binding to Queue A with arguments (key = value): format = pdf, type = report, x-match = all
* Exchange: Binding to Queue B with arguments (key = value): format = pdf, type = log, x-match = any
* Exchange: Binding to Queue C with arguments (key = value): format = zip, type = report, x-match = all

![rabbitmq header exchange](/docs/high-concurrency/images/headers-exchange.png)

