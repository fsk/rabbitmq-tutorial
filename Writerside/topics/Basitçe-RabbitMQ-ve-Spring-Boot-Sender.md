# Basitçe RabbitMQ ve Spring Boot Sender

## Dockerda RabbitMQ Ayağa Kaldırma

````Docker
docker run -d --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3-management
````

Yukarıdaki docker commandi ile bir rabbitMQ ayağa kaldırmamız gerekir. (RabbitMQ'yu localimize de kurabiliriz.) 
Bu komutları teker teker incelediğimizde <br/>
**`docker run:`** Docker’da yeni bir container oluşturup çalıştırmak için kullanılan temel komuttur. <br/>
**`-d (detached mode):`** Bu config, containerin arka planda (background) çalışmasını sağlar. Komut çalıştıktan sonra 
terminali serbest bırakır ve containerin çıktısı terminali meşgul etmez. Container arka planda devam eder. <br/>
**`--name rabbitmq:`** Bu kısım, oluşturulacak containerin ismini “rabbitmq” olarak belirler. Aksi halde, Docker 
containera rastgele bir isim atar. Bu isim sayesinde containeri daha sonra yönetirken (örneğin, durdurmak, başlatmak veya 
loglara bakmak için) bu isimle kullanabiliriz. <br/>
**`-p 5672:5672:`** Bu seçenek, container içindeki 5672 numaralı portu (RabbitMQ’nun varsayılan AMQP bağlantı portudur) 
host makine üzerinde de 5672 numaralı port üzerinden erişilebilir hale getirir. Yani host makinede localhost:5672 ile 
bu containera bağlanabiliriz. <br/>
**`-p 15672:15672:`** Bu da container içindeki 15672 numaralı portu (RabbitMQ Management Plugin’in web arayüzü için 
varsayılan port) host makineye eşleştirir. Böylece http://localhost:15672 adresi üzerinden RabbitMQ yönetim paneline 
tarayıcımızla erişebiliriz.<br/>
**`rabbitmq:3-management::`** Bu, kullanılacak imageın ismi ve tag'ıdır. rabbitmq temel RabbitMQ sunucu imagedır. 
`3-management` tagı ise RabbitMQ yönetim konsolunu (Management Plugin) içeren sürümünü ifade eder. 
Bu sayede yukarıda bahsedilen web tabanlı yönetim paneline erişim mümkün olur.


RabbitMQ ayağa kalksın ve bizi beklesin.

## Java - Spring Boot 

`pom.xml` dosyamıza aşağıdaki bağımlılığı ekleyerek işe başlayalım.

````XML
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
````

`application.yml` dosyamıza aşağıdaki gibi configleri ekleyelim.

````YAML
server:
  port: 8082

spring:
  application:
    name: sender

  rabbitmq:
    host: localhost
    port: 5673
    username: guest
    password: guest

````

Öncelikle aşağıdaki gibi bir class oluşturmamız lazım.

````JAVA
@Data
@Builder
public class SendingPayload {

    private String message;

}
````

Bu class bizim için rabbitMq'ya gönderilecek Payload görevi görecek.

````JAVA
public record MessageRequest(String message) {

}

````

Bu record bizim requestimiz olacak.

Sonrasında artık rabbitMQ'ya gönderilecek mesaj için bir `MessageSender`service class'ını yazalım.

````JAVA
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;

@Service
@RequiredArgsConstructor
@Slf4j
public class MessageSender {

    private final Logger logger = LoggerFactory.getLogger(MessageSender.class);

    private final RabbitTemplate rabbitTemplate;

    private static final String QUEUE_NAME = "first-queue";

    public void sendMessage(MessageRequest request) {

        SendingPayload payload = SendingPayload.builder()
                .message(request.message())
                .build();

        rabbitTemplate.convertAndSend(QUEUE_NAME, payload);

        logger.info("*{}* Message sent to queue: {}", payload .getMessage(), QUEUE_NAME);
    
    }

}
````

Ve son olarak controller class'ımızı aşağıdaki şekilde oluşturabiliriz.

````JAVA
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

import lombok.RequiredArgsConstructor;

@RestController
@RequiredArgsConstructor
public class MessageController {

    private final MessageSender messageSender;

    @PostMapping("/send")
    public void sendMessage(@RequestBody MessageRequest request) {
        messageSender.sendMessage(request);
    }
}
````

Artık uygulamayı ayağa kaldırıp testimizi yapabiliriz.

Bir curl isteği yazalım.

````CURL
curl -X POST http://localhost:8082/send \
-H "Content-Type: application/json" \
-d '{"message": "Merhaba RabbitMQ!"}'
````

Bu cURL isteğini yazıp gönderdiğimizde bir hata almamız gerekecek.

`rabbitTemplate.convertAndSend(QUEUE_NAME, payload);` bu satır için aşağıdaki hatayı verdi.

````
java.lang.IllegalArgumentException: SimpleMessageConverter only 
supports String, byte[] and Serializable payloads, 
received: org.fsk.sender.SendingPayload
````

Buradaki problem şu. Aslında olay çok basit. Hata mesajını Türkçeye çevirirsek aşağıdaki gibi bir anlam
çıkar.

````
SimpleMessageConverter yalnızca String, byte[] ve Serializable 
payloadları destekler, alınan: org.fsk.sender.SendingPayload"
````

Yani kısaca gönderilecek mesajı serialize etmemiz gerektiği söyleniyor.

> **Mesajı gönderirken neden Serileştirmem gerekir**
>
>  Aslında bu durum çok fazla kullanılan bir pratiktir. Genellikle bu tarz işlemlerde java classları 
> **Serializable** interfacei ile implemente edilir. Ama neden.?
> 
> Nesnelerin ağ üzerinden iletilmesi gerektiğinde (RabbitMQ, Kafka gibi)
> 
> Nesnelerin diske yazılması/kaydedilmesi gerektiğinde
> 
> Nesnelerin cache'lenmesi gerektiğinde
> 
> Nesnelerin JVM dışına çıkması gereken durumlarda
> 
> Bir nesnenin byte dizisine dönüştürülmesi gerektiğinde
> 
> Farklı JVM'ler arasında nesne transferi yapılacağında
> 
> Bu gibi durumlarda nesneler ya da classlar Serializable interfacei ile implemente edilmeli.
>
{style="note"}

Bu nottan sonra classımız aşağıdaki gibi olmalıdır.

````JAVA
@Data
@Builder
public class SendingPayload implements Serializable {

    private String message;

}
````

Bu şekilde düzenleyip çalıştırdığımızda uygulamamız güzel bir şekilde çalışacaktır.

Curl isteğini atıp gönderdiğimizde aşağıdaki log'u görmemiz gerekir.

````Log
2024-12-10T23:10:58.283+03:00  INFO 35139 --- 
[sender] [nio-8082-exec-1] org.fsk.sender.MessageSender: 
** Merhaba RabbitMQ! ** Message sent to queue: first-queue
````
Eğer bu logu gördüysem Queue'da bir kuyruk oluşması lazım ve mesajımı görmem lazım.

Hemen docker'da ayağa kaldırdığımız RabbitMQ Web UI'a 15672 portu ile bağlanıp bakalım. 

![rabbitmq-first.png](rabbitmq-first.png)

Siyah kutu içerisinde görüldüğü üzere herhangi bir queue yok. 
Bir kuyruk oluşturmak için mavi kutu içerisindeki mor kutuya sadece bir tane queue adı yazarak `bu kuyruk adı uygulamadadki
kuyruk adı ile muhakkak eşleşmeli` kuyruk tanımlayabiliriz. Bu yüzden pembe kutuya `first-queue` yazarak Add Queue butonuna
basabiliriz. Ve ardından tekrardan cURL isteğimizi yollayabiliriz.

İsteği yolladıktan sonra tekrar queue'ya baktığımızda aşağıdaki gibi bir tablo ile karşılaşacağız.

![rabbitmq-second.png](rabbitmq-second.png)

1 tane message Ready state'inde ve totalde de 1 tane mesajımız var. 2.kolondaki first-queue adlı queueya tıklayarak
detayları görebiliriz.

![rabbitmq-third.png](rabbitmq-third.png)

Detay sayfasında biraz aşağı doğru baktığımızda yukarıdaki gibi bir ekran bizi karşılayacak. Tam burada Get Message diyerek
message'ın payloadını görebiliriz.

````Bash
rO0ABXNyAB1vcmcuZnNrLnNlbmRlci5TZW5kaW5nUGF5bG9hZLRXUmEVRiIDAgABTAAHbWVzc2FnZXQAEkxqYXZhL2xhbmcvU3RyaW5nO3hwdAARTWVyaGFi YSBSYWJiaXRNUSE= 
````
Bu mesaj, Java object serialization kullanılarak oluşturulmuş, Base64 ile kodlanmış bir seri hale getirilmiş object 
(Java serialized object) biçimindedir. Kısacası, bir Java uygulamasında bir objectin serileştirilerek Base64 iletilmiş halidir.

**Peki bu problemi nasıl çözeceğim.?**

Bu durumu çözmenin yolu bir `Configuration` classı oluşturmak ve içerisine gerekli configleri yazmak.

````Java
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.amqp.support.converter.Jackson2JsonMessageConverter;
import org.springframework.amqp.support.converter.MessageConverter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class RabbitMQConfig {


    @Bean
    public MessageConverter messageConverter() {
        return new Jackson2JsonMessageConverter();
    }

    @Bean
    public RabbitTemplate rabbitTemplate(ConnectionFactory factory) {
        RabbitTemplate template = new RabbitTemplate(factory);
        template.setMessageConverter(messageConverter());
        return template;
    }

}
````

Bu kodu biraz açıklayacak olursak

````Java
@Bean
public MessageConverter messageConverter() {
    return new Jackson2JsonMessageConverter();
}
````
* MessageConverter, AMQP üzerinden gönderilen veya alınan mesajların nasıl dönüştürüleceğini belirleyen bir bileşendir.
* Default olarak, RabbitMQ ile çalışırken mesajlar genellikle **byte array (binary)** olarak taşınır. 
Ancak modern uygulamalarda JSON formatında mesaj alışverişi yapmak daha yaygındır.
* Burada `Jackson2JsonMessageConverter` kullanılıyor. Bu converter, giden mesajları otomatik olarak JSON formatına 
serileştirir, gelen mesajları da JSON’dan Java objectlerine deserialize eder.


````Java
@Bean
public RabbitTemplate rabbitTemplate(ConnectionFactory factory) {
    RabbitTemplate template = new RabbitTemplate(factory);
    template.setMessageConverter(messageConverter());
    return template;
}
````
* **RabbitTemplate**, Spring AMQP içerisinde RabbitMQ ile etkileşim kurmanın yüksek seviyeli bir kütüphanesidir.
* RabbitTemplate, bağlantı yönetimi, channel açma-kapama gibi altyapısal ayrıntılarla uğraşmadan basit methot 
çağrılarıyla mesaj gönderip almamıza olanak tanır.
* Bu bean oluşturulurken Spring containeri tarafından bir `ConnectionFactory` nesnesi inject edilir. ConnectionFactory, 
RabbitMQ broker’ına (sunucusuna) bağlantının nasıl kurulacağını tarif eden bileşendir (örn. host, port, kullanıcı adı, şifre, vs.)
* `template.setMessageConverter(messageConverter());` satırı ile yukarıda tanımlanan `Jackson2JsonMessageConverter` bu 
template’e bağlanır. Böylece RabbitTemplate üzerinden gönderilen ve alınan tüm mesajlar bu converter aracılığıyla JSON’a 
dönüşür.

Tekrar bizim uygulamamıza dönecek olursak,
açıklamalardan da anlaşılacağı üzere artık `SendingPayload` classında Serializable interface'ini implemente etmemize gerek
kalmadı. Yani burayı silip denediğimizde artık aşağıdaki görselde de görülebileceği gibi mesajımızı güzel bir şekilde aldık.

![rabbit-fourth.png](rabbit-fourth.png)

**Soru** <br/>
**Aynı mantıkla kuyruğu rabbitMQ'dan manuel olarak değil de Spring Boot üzerinden oluşturabilir miyim.?**

RabbitTemplate yapısından dolayı cevap kesinlikle evet.

````Java
@Bean
public Queue queue() {
    return new Queue("first-queue");
}
````

Yukarıdaki bean'i `Configuration` classımıza eklememiz bizim için yeterli olacaktır. İmportlarla birlikte `Configuration`
classımız aşağıdaki gibi olacaktır.

````Java
import org.springframework.amqp.core.Queue;
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.amqp.support.converter.Jackson2JsonMessageConverter;
import org.springframework.amqp.support.converter.MessageConverter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;


@Configuration
public class RabbitMQConfig {

    @Bean
    public Queue queue() {
        return new Queue("first-queue");
    }

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