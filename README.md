a simple project to show how to use reproduce a bug with spring-dev-tools:
https://github.com/spring-projects/spring-boot/issues/14622

Update: Delegated to upstream issue: https://issues.apache.org/jira/browse/AVRO-1425

# Steps to reproduce

## Run Kafka

For this test project I use the [Confuent Open Source](https://www.confluent.io/download/) platform.

    docker-compose up -d
    
## building and running the program

    mvn spring-boot:run
    
is all what is needed. It starts up the program, sets up a KafkaListener, produces the configured number of records 
into Kafka, consumes them and then shuts down the Listener and so ends the program.
... at least that's what happens in the [original version](https://github.com/pjmeisch-cc/spring-boot-kafka-with-avro) when you run it without **spring-dev-tools**. I forked it to show the bug.


sample run:
```
2018-09-27 19:19:48.214  INFO 6927 --- [  restartedMain] d.c.sbkaavro.SbkaavroApplication         : Starting SbkaavroApplication on manuel with PID 6927 (/tmp/14622/target/classes started by manu in /tmp/14622)
2018-09-27 19:19:48.218 DEBUG 6927 --- [  restartedMain] d.c.sbkaavro.SbkaavroApplication         : Running with Spring Boot v2.0.2.RELEASE, Spring v5.0.6.RELEASE
2018-09-27 19:19:48.219  INFO 6927 --- [  restartedMain] d.c.sbkaavro.SbkaavroApplication         : No active profile set, falling back to default profiles: default
2018-09-27 19:19:50.126  INFO 6927 --- [  restartedMain] d.c.sbkaavro.SbkaavroApplication         : Started SbkaavroApplication in 2.278 seconds (JVM running for 2.783)
2018-09-27 19:19:50.289  INFO 6927 --- [  restartedMain] de.codecentric.sbkaavro.DataProducer     : producing {"lastName": "Kampmann", "firstName": "Isbert"}, {"zip": "52539", "city": "Melle Bennien", "street": "Raschenstraße", "streetNumber": "101"}
2018-09-27 19:19:50.300  WARN 6927 --- [ntainer#0-0-C-1] org.apache.kafka.clients.NetworkClient   : [Consumer clientId=consumer-1, groupId=83b159ee-27a4-4f17-86f8-f6f3f12063ce] Error while fetching metadata with correlation id 2 : {persons=LEADER_NOT_AVAILABLE}
2018-09-27 19:19:50.320  WARN 6927 --- [  restartedMain] o.a.k.clients.producer.ProducerConfig    : The configuration 'specific.avro.reader' was supplied but isn't a known config.
2018-09-27 19:19:50.852  INFO 6927 --- [  restartedMain] de.codecentric.sbkaavro.DataProducer     : producing {"lastName": "Schmidtke", "firstName": "Britt"}, {"zip": "44811", "city": "Schieder", "street": "Kreuzeckweg", "streetNumber": "175"}
2018-09-27 19:19:50.854  INFO 6927 --- [  restartedMain] de.codecentric.sbkaavro.DataProducer     : producing {"lastName": "Mandel", "firstName": "Patrizia"}, {"zip": "19027", "city": "Neuenburg am Rhein", "street": "Alma-Rogge-Weg", "streetNumber": "191"}
2018-09-27 19:19:50.855  INFO 6927 --- [  restartedMain] de.codecentric.sbkaavro.DataProducer     : producing {"lastName": "Krings", "firstName": "Corina"}, {"zip": "00022", "city": "Bansin", "street": "Kempstraße", "streetNumber": "95"}
2018-09-27 19:19:50.857  INFO 6927 --- [  restartedMain] de.codecentric.sbkaavro.DataProducer     : producing {"lastName": "Ladwig", "firstName": "Volkard"}, {"zip": "48827", "city": "Heiligenstein", "street": "Meeraner Straße", "streetNumber": "3"}
2018-09-27 19:19:53.396 ERROR 6927 --- [ntainer#0-0-C-1] o.s.kafka.listener.LoggingErrorHandler   : Error while processing: ConsumerRecord(topic = persons, partition = 0, offset = 0, CreateTime = 1538068790834, serialized key size = 21, serialized value size = 44, headers = RecordHeaders(headers = [], isReadOnly = false), key = {"lastName": "Kampmann", "firstName": "Isbert"}, value = {"zip": "52539", "city": "Melle Bennien", "street": "Raschenstraße", "streetNumber": "101"})

org.springframework.kafka.listener.ListenerExecutionFailedException: Listener method 'public void de.codecentric.sbkaavro.DataConsumer.receive(org.apache.kafka.clients.consumer.ConsumerRecord<de.codecentric.sbkaavro.avro.Person, de.codecentric.sbkaavro.avro.Address>)' threw exception; nested exception is java.lang.ClassCastException: de.codecentric.sbkaavro.avro.Person cannot be cast to de.codecentric.sbkaavro.avro.Person
	at org.springframework.kafka.listener.adapter.MessagingMessageListenerAdapter.invokeHandler(MessagingMessageListenerAdapter.java:267)
	at org.springframework.kafka.listener.adapter.RecordMessagingMessageListenerAdapter.onMessage(RecordMessagingMessageListenerAdapter.java:80)
	at org.springframework.kafka.listener.adapter.RecordMessagingMessageListenerAdapter.onMessage(RecordMessagingMessageListenerAdapter.java:51)
	at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.doInvokeRecordListener(KafkaMessageListenerContainer.java:1071)
	at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.doInvokeWithRecords(KafkaMessageListenerContainer.java:1051)
	at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.invokeRecordListener(KafkaMessageListenerContainer.java:998)
	at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.invokeListener(KafkaMessageListenerContainer.java:866)
	at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.run(KafkaMessageListenerContainer.java:724)
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
	at java.util.concurrent.FutureTask.run(FutureTask.java:266)
	at java.lang.Thread.run(Thread.java:748)
Caused by: java.lang.ClassCastException: de.codecentric.sbkaavro.avro.Person cannot be cast to de.codecentric.sbkaavro.avro.Person
	at de.codecentric.sbkaavro.DataConsumer.receive(DataConsumer.java:27)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.springframework.messaging.handler.invocation.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:181)
	at org.springframework.messaging.handler.invocation.InvocableHandlerMethod.invoke(InvocableHandlerMethod.java:114)
	at org.springframework.kafka.listener.adapter.HandlerAdapter.invoke(HandlerAdapter.java:48)
	at org.springframework.kafka.listener.adapter.MessagingMessageListenerAdapter.invokeHandler(MessagingMessageListenerAdapter.java:248)
	... 10 common frames omitted
```
