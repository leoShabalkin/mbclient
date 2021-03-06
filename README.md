# Message Broker Client/Server

The MB client creates an abstraction over the interservice interaction on top of RabbitMQ. The library defines a common interface for messages and provides ways to send and subscribe to them. The client supports automatic reconnections to RabbitMQ and support for the Rabbit cluster.

The mechanism is quite simple and currently supports 2 simple operation modes (sending directly to the queue, sending to topic exchange)

When a client is created, a durable topic exchange ("dispatcher" by default) is automatically created, and a service queue (with the name that was passed as serviceName during initialization).

When sending a message indicating the recipients, the message is sent to their queue directly. Otherwise, the message is sent via routingKey "{serviceName}. {Action}" to the dispatcher exchange.

# Examples

## Subscribing

### Subscribe to broadcast messages by message type

```javascript
  const client = createClient({
    serviceName: 'news',
    connectOptions: {
      username: 'test',
      password: '123',
      host: 'localhost',
      port: 5672,
    },
  });

  // listening to messages
  client.consumeByAction('logAction', ({ message, ack, nack }) => {
    // do something
    ack();
  });

  client.consumeByAction('otherAction', ({ message, ack, nack }) => {
    // do something
    ack();
  });
  ```

## Publishing
### Publishing a message indicating recipients

```javascript
  client.send({
    action: 'comeAction',
    payload: 'some payload',
    requestId: 'id',
    recipients: ['news', 'test'] // a message will be sent to news and test
  });
```

### Publication without specifying recipients (broadcast)
```javascript
  client.send({
    action: 'comeAction', // Everyone who consumed on comeAction will receive this message
    payload: 'some payload',
    requestId: 'id',
  });
```

## Subscribe to connection status

It is possible to subscribe to a change in connection status with rabbitmq

```javascript
client.on('disconnected', () => {
 // do something
});

client.on('connected', () => {
 // do something
});
```

Supported Events:

`connecting` - Attempt to connect to amqp

`connected` - Successful connection to amqp

`disconnecting` - Close the connection (usually emitted when calling close for a graceful disconnect)

`disconnected` - Loss of connection with amqp due to an error or as a result of processing close ()
## License

```
Copyright 2019 Tinkoff Bank

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
