# Mesajlarda Öncelik

Bazen gönderdiğimiz mesajları önceliklendirmek isteyebiliriz. Dolayısıyla rabbitMQ'da mesajlar önceliklendirilebiliyor.

Aşağıdaki şekilde sender uygulamamız için ``configuration`` dosyamıza kodumuzu ekleyelim.

```Java
public static final String PRIORITY_QUEUE = "priority-queue";

@Bean
public Queue priorityQueue(){
    Map<String, Object> args = new HashMap<>();
    args.put("x-max-priority", 10);
    return new Queue(PRIORITY_QUEUE, true, false, false, args);
}
```
`priority` özelliği aslında 0 ila 255 arasında değişen bir sayıdır ve en yüksek öncelik en büyük sayıya aittir. Ama genelde
pratikte 1 - 10 arasındaki sayılar kullanılır.

Bu adımdan sonra aşağıdaki şekilde bir request class'ı ekleyelim.

```java
public record PriorityQueueSendingPayload(
        String message,
        int priority
) {
}
```

Şimdi yapmamız gereken artık Sender methodumuzu yazmak.

```java
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.fsk.producer.payload.PriorityQueueSendingPayload;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.core.MessageProperties;
import org.springframework.amqp.core.Queue;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.stereotype.Service;


@Service
@RequiredArgsConstructor
@Slf4j
public class PriorityQueueMessageSender {

    private final Logger logger = LoggerFactory.getLogger(PriorityQueueMessageSender.class);

    private final RabbitTemplate rabbitTemplate;
    private final Queue priorityQueue;

    public void sendMessage(PriorityQueueSendingPayload payload) {

        MessageProperties properties = new MessageProperties();
        properties.setPriority(payload.priority());

        rabbitTemplate.convertAndSend(priorityQueue.getName(), payload, message -> {
            MessageProperties props = message.getMessageProperties();
            props.setPriority(payload.priority());
            return message;
        });

        logger.info("*{}* Message sent to queue: {}", payload.message(), priorityQueue.getName());

    }

}
```

Aşağıdaki şekilde artık cURL isteklerimizi gönderebiliriz.

```cURL
curl -X POST \
  http://localhost:8082/sendPriorityQueue \
  -H 'Content-Type: application/json' \
  -d '{
    "message": "Spring Boot and RabbitMQ",
    "priority": 8
  }'
```

```cURL
curl -X POST \
  http://localhost:8082/sendPriorityQueue \
  -H 'Content-Type: application/json' \
  -d '{
    "message": "Turkiye Java Community",
    "priority": 5
  }'
```

```cURL
curl -X POST \
  http://localhost:8082/sendPriorityQueue \
  -H 'Content-Type: application/json' \
  -d '{
    "message": "FSK",
    "priority": 10
  }'
```

Receiver uygulamasını çalıştırdığımızda ise öncelik sırasına göre mesajları alacağız. 

Beklediğimiz sıra aşağıdaki şekilde olmalı. (Önceliği yüksek olan, ilk önce okunacak.)

* FSK
* Spring Boot and RabbitMQ
* Turkiye Java Community

Aşağıdaki loglara baktığımızda zaman ise böyle olduğunu görebiliyoruz.

![rabbitmq-tenth.png](rabbitmq-tenth.png)