# Spring Boot ile Queue Okumak

Oluşturulan queue'ları anlamak için bir receiver yazıp daha detaylı inceleyebiliriz.

Aslında yazacağımız bu kod spring boot ile yazdığımız `Sender`uygulamasına çok benzeyecek.

### application.yml dosyası
````yaml
spring:
  application:
    name: receiver
  rabbitmq:
    username: guest
    password: guest
    host: localhost
    port: 5673
````

### config dosyası
````java
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.amqp.support.converter.Jackson2JsonMessageConverter;
import org.springframework.amqp.support.converter.MessageConverter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;


@Configuration
public class RabbitMQReceiverConfiguration {

    @Bean
    public MessageConverter messageConverter() {
        return new Jackson2JsonMessageConverter();
    }

    @Bean
    public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {
        RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory);
        rabbitTemplate.setMessageConverter(messageConverter());
        return rabbitTemplate;
    }

}
````

### message payload dosyası
````java
public record ConsumeMessage(String message) {
}
````

### service dosyası
````
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Slf4j
@Component
public class MessageConsumer {

    public static final String QUEUE = "first-queue";

    @RabbitListener(queues = QUEUE)
    public void handleMessage(ConsumeMessage message) {
        log.info("Consume Message ==> {}", message.message());
    }

}
````

Sender uygulamasından aşağıdaki gibi bir cURL isteği attığımızda receiver uygulamamız gidip bu mesajı queue'dan okuyacak
ve loglara basacak.

````cURL
curl -X POST \
  http://localhost:8082/send \
  -H 'Content-Type: application/json' \
  -d '{"message": "RabbitMQ tutorial with Spring Boot!"}'
````

````cURL
curl -X POST \
  http://localhost:8082/send \
  -H 'Content-Type: application/json' \
  -d '{"message": "Turkiye Java Community"}' 
````

Yukarıdaki gibi 2 tane mesajı queue'ya bıraktığımızda aşağıdaki gibi queue'da mesajların olduğunu göreceğiz.

![rabbitmq-eight.png](rabbitmq-eight.png)

Artık mesajları dinlemek için yazdığımız receiver kodunu çalıştırıp loglara baktığımızda ise, aşağıdaki resimde oludğu gibi
logları görebileceğiz.

![rabbitmq-nineth.png](rabbitmq-nineth.png)

Ve receiver uygulaması ayakta iken gönderilen her mesajı receive edecek.