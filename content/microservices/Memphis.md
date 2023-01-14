### Memphis.dev

Simple as RabbitMQ, robust as Kafka, and perfect for busy developers.<br><br>
[Memphis](https://memphis.dev) is a next-generation message broker.<br>
A simple, robust, and durable cloud-native message broker wrapped with
an entire ecosystem that enables fast and reliable development of next-generation event-driven application.

Memphis enables building next-generation applications that require large volumes of streamed and enriched data,
modern protocols, zero ops, rapid development, extreme cost reduction,
and a significantly lower amount of dev time for data-oriented developers and data engineers.

Some of Memphis features:
- 🚀 Fully optimized message broker in under 3 minutes
- 💻 Easy-to-use UI, CLI, and SDKs
- 📺 Data-level observability
- ☠️ Dead-Letter Queue with automatic message retransmit
- 🔤 Schemaverse - Embedded schema management for produced data (Protobuf/JSON/GraphQL/Avro)
- ⛓ SDKs: Node.JS, Go, Python, TypeScript, NestJS
- 🐳☸ Runs on any Docker or Kubernetes
- 👨‍💻 Community driven

[Github](https://github.com/memphisdev/memphis-broker)

<br>

#### Installation

To start using Memphis, first install the required package:

```bash
$ npm i --save memphis-dev
```

#### Importing
```js
import { Module } from '@nestjs/common';
import { MemphisModule, MemphisService } from 'memphis-dev/nest';
import type { Memphis } from 'memphis-dev/types';
```

#### Connecting to Memphis
```js
@Module({
    imports: [MemphisModule.register()],
})

class ConsumerModule {
    constructor(private memphis: MemphisService) {}

    startConnection() {
        (async function () {
            let memphisConnection: Memphis;

            try {
               memphisConnection = await this.memphis.connect({
                    host: "<memphis-host>",
                    username: "<application type username>",
                    connectionToken: "<broker-token>",
                });
            } catch (ex) {
                console.log(ex);
                memphisConnection.close();
            }
        })();
    }
}
```

Once connected, the entire functionalities offered by Memphis are available.

#### Disconnecting from Memphis

To disconnect from Memphis, call `close()` on the memphis object.

```js
memphisConnection.close();
```

#### Creating a Station (= queue)

```js
@Module({
    imports: [MemphisModule.register()],
})

class stationModule {
    constructor(private memphis: MemphisService) { }

    createStation() {
        (async function () {
                  const station = await this.memphis.station({
                        name: "<station-name>",
                        schemaName: "<schema-name>",
                        retentionType: memphis.retentionTypes.MAX_MESSAGE_AGE_SECONDS, // defaults to memphis.retentionTypes.MAX_MESSAGE_AGE_SECONDS
                        retentionValue: 604800, // defaults to 604800
                        storageType: memphis.storageTypes.DISK, // defaults to memphis.storageTypes.DISK
                        replicas: 1, // defaults to 1
                        idempotencyWindowMs: 0, // defaults to 120000
                        sendPoisonMsgToDls: true, // defaults to true
                        sendSchemaFailedMsgToDls: true // defaults to true
                  });
        })();
    }
}
```

**Retention types**

Memphis currently supports the following types of retention:

```js
memphis.retentionTypes.MAX_MESSAGE_AGE_SECONDS;
```

Means that every message persists for the value set in retention value field (in seconds)

```js
memphis.retentionTypes.MESSAGES;
```

Means that after max amount of saved messages (set in retention value), the oldest messages will be deleted

```js
memphis.retentionTypes.BYTES;
```

Means that after max amount of saved bytes (set in retention value), the oldest messages will be deleted

**Storage types**

Memphis currently supports the following types of messages storage:

```js
memphis.storageTypes.DISK;
```

Means that messages persist on disk

```js
memphis.storageTypes.MEMORY;
```

Means that messages persist on the main memory

#### Destroying a Station

Destroying a station will remove all its resources (producers/consumers)

```js
await station.destroy();
```

#### Schemaverse
Memphis Schemaverse provides a robust schema store and schema management layer on top of memphis broker without a standalone compute or dedicated resources.

**Attaching a Schema to an Existing Station**

```js
await memphisConnection.attachSchema({ name: '<schema-name>', stationName: '<station-name>' });
```

**Detaching a Schema from Station**

```js
await memphisConnection.detachSchema({ stationName: '<station-name>' });
```

#### Produce and Consume messages

The most common client operations are `produce` to send messages and `consume` to
receive messages.<br>

Messages are published to a station and consumed from it by creating a consumer.<br>
Consumers are pull based and consume all the messages in a station unless you are using a consumers group, in this case messages are spread across all members in this group.<br>

Memphis messages are payload agnostic. Payloads are `Uint8Arrays`.

In order to stop getting messages, you have to call `consumer.destroy()`.<br>Destroy will terminate regardless
of whether there are messages in flight for the client.

**Creating a Producer**
```js
@Module({
    imports: [MemphisModule.register()],
})

class ProducerModule {
    constructor(private memphis: MemphisService) { }

    createProducer() {
        (async function () {
                const producer = await memphisConnection.producer({
                    stationName: "<station-name>",
                    producerName: "<producer-name>"
                });
        })();
    }
}
```

#### Producing a message
```js
await producer.produce({
    message: '<bytes array>/object/string/DocumentNode graphql', // Uint8Arrays/object (schema validated station - protobuf) or Uint8Arrays/object (schema validated station - json schema) or Uint8Arrays/string/DocumentNode graphql (schema validated station - graphql schema)
    ackWaitSec: 15 // defaults to 15
});
```

#### Add Header

```js
const headers = memphis.headers();
headers.add('<key>', '<value>');
await producer.produce({
    message: '<bytes array>/object/string/DocumentNode graphql', // Uint8Arrays/object (schema validated station - protobuf) or Uint8Arrays/object (schema validated station - json schema) or Uint8Arrays/string/DocumentNode graphql (schema validated station - graphql schema)
    headers: headers // defults to empty
});
```

#### Async produce

Meaning your application won't wait for broker acknowledgement - use only in case you are tolerant for data loss

```js
await producer.produce({
    message: '<bytes array>/object/string/DocumentNode graphql', // Uint8Arrays/object (schema validated station - protobuf) or Uint8Arrays/object (schema validated station - json schema) or Uint8Arrays/string/DocumentNode graphql (schema validated station - graphql schema)
    ackWaitSec: 15, // defaults to 15
    asyncProduce: true // defaults to false
});
```

#### Message ID

Stations are idempotent by default for 2 minutes (can be configured), Idempotency achieved by adding a message id

```js
await producer.produce({
    message: '<bytes array>/object/string/DocumentNode graphql', // Uint8Arrays/object (schema validated station - protobuf) or Uint8Arrays/object (schema validated station - json schema) or Uint8Arrays/string/DocumentNode graphql (schema validated station - graphql schema)
    ackWaitSec: 15, // defaults to 15
    msgId: 'fdfd' // defaults to null
});
```

#### Destroying a Producer

```js
await producer.destroy();
```

#### Creating a Consumer

```js
const consumer = await memphisConnection.consumer({
    stationName: '<station-name>',
    consumerName: '<consumer-name>',
    consumerGroup: '<group-name>', // defaults to the consumer name.
    pullIntervalMs: 1000, // defaults to 1000
    batchSize: 10, // defaults to 10
    batchMaxTimeToWaitMs: 5000, // defaults to 5000
    maxAckTimeMs: 30000, // defaults to 30000
    maxMsgDeliveries: 10, // defaults to 10
    genUniqueSuffix: false
});
```

To set Up connection in nestjs

```js
import { MemphisServer } from 'memphis-dev/nest'

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      strategy: new MemphisServer({
        host: '<memphis-host>',
        username: '<application type username>',
        connectionToken: '<broker-token>'
      }),
    },
  );

  await app.listen();
}
bootstrap();
```

To Consume messages in nestjs

```js
export class Controller {
    import { consumeMessage } from 'memphis-dev/nest';
    import { Message } from 'memphis-dev/types';

    @consumeMessage({
        stationName: '<station-name>',
        consumerName: '<consumer-name>',
        consumerGroup: ''
    })
    async messageHandler(message: Message) {
        console.log(message.getData().toString());
        message.ack();
    }
}
```

#### Processing messages

```js
consumer.on('message', (message) => {
    // processing
    console.log(message.getData());
    message.ack();
});
```

#### Acknowledge a message

Acknowledge a message indicates the Memphis server to not re-send the same message again to the same consumer / consumers group

```js
message.ack();
```

#### Get headers

Get headers per message

```js
headers = message.getHeaders();
```

#### Catching async errors

```js
consumer.on('error', (error) => {
    // error handling
});
```

#### Destroying a Consumer

```js
await consumer.destroy();
```