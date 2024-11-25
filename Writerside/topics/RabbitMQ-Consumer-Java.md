# RabbitMQ - Consumer Java

Basit bir java projesi açıp (build olarak maven seçelim) içerisine aşağıdaki dependency'yi ekleyelim.

```XML
<dependency>
    <groupId>com.rabbitmq</groupId>
    <artifactId>amqp-client</artifactId>
    <version>5.22.0</version>
 </dependency>
```

Receive.java class'ı aşağıdaki gibidir.

```Java
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.DeliverCallback;

import java.nio.charset.StandardCharsets;

public class Receive {

    private final static String QUEUE_NAME = "hello";

    public static void main(String[] args) throws Exception{

        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        factory.setPort(1905);
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
            System.out.println(" [x] Received '" + message + "'");
        };
        channel.basicConsume(QUEUE_NAME, true, deliverCallback, consumerTag -> { });
    }
}
```

Kodları teker teker inceleyelim.

<b>-></b> Bir connectionFactory kuruyoruz ve bu kurduğumuz connectionFactory üzerinden bağlanacağımız ip adresi ile portu veriyoruz.

```Java
ConnectionFactory factory = new ConnectionFactory();
factory.setHost("localhost");
factory.setPort(1905);
Connection connection = factory.newConnection();
```

<b>-></b> Bir channel açıyoruz.

```Java
Channel channel = connectionFactory.newConnection().createChannel();
```

<b>-></b> Bir queue declare ediyoruz. Yani tanımlıyoruz.

```Java
channel.queueDeclare(QUEUE_NAME, false, false, false, null);
```
Bu kodun detayları Producer Java tab'inda detaylıca var.

<b>-></b> RabbitMQ Java client API'sinde mesajların teslim edildiğinde alınmasını sağlayan bir callback fonksiyon tanımlıyoruz. 

```Java
DeliverCallback deliverCallback = (consumerTag, delivery) -> {
    String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
    System.out.println(" [x] Received '" + message + "'");
};
```
``DeliverCallback`` RabbitMQ Java Client API'sinde mesajların teslim edildiğinde alınmasını sağlayan bir callback fonksiyondur.
Bu `Functional Interface` lambda'larla kullanılmak üzere optimize edilmiştir ve bir tüketiciye (cosumer) bir mesaj teslim
edildiğinde, mesajı işlemek için gerekli olan fonksiyonalite'yi sağlar.

```Java
// DelierCallback functional interface'i syntaxı aşağıdaki gibidir.

@FunctionalInterface
public interface DeliverCallback {
    void handle(String consumerTag, Delivery message) throws IOException;
}
```

**Temel Rolü**

Bu interface, bir RabbitMQ tüketicisi için mesaj teslimatı gerçekleştiğinde çağrılan bir callback'i temsil eder. Bu callback, 
RabbitMQ'da bir mesajın bir kuyruktan alındığı (delivered) ve bir tüketiciye iletildiği zaman devreye girer.

**handle() methodu**

Bu method 2 tane parametre alır.

* `String consumerTag: ` Bu, mesajın hangi tüketiciye (consumer) ait olduğunu tanımlayan bir etikettir. RabbitMQ'da her 
bir tüketici, RabbitMQ sunucusu tarafından otomatik olarak bir tüketici etiketi (consumer tag) ile ilişkilendirilir. 
Consumer Tag, bir kuyruktaki mesajların farklı tüketiciler tarafından işlenip işlenmediğini izlemek için kullanılır.
Örneğin, birden fazla tüketici aynı kuyruğu dinliyorsa, her tüketicinin consumerTag'i farklı olacaktır.

* `Delivery message: ` Bu, RabbitMQ tarafından teslim edilen gerçek mesajdır. Delivery sınıfı, mesajın içeriğini (body) 
ve bazı ek meta verileri (BasicProperties) içerir. Bu nesne, mesajın kendisini ve mesaj ile ilgili başlıkları (headers) 
veya diğer bilgileri sağlar.


<b>-></b> RabbitMQ'daki queue'dan mesaj tüketmek için aşağıdaki kodu yazıyoruz.

```Java
channel.basicConsume(QUEUE_NAME, true, deliverCallback, consumerTag -> { });
```

Bu methodun syntaxı aşağıdaki gibidir.

```Java
basicConsume(String queue, boolean autoAck, DeliverCallback deliverCallback, CancelCallback cancelCallback)
```

Bu methot, RabbitMQ'dan mesaj almak (consume) için bir tüketici oluşturur. Queue'dan gelen mesajlar belirli bir 
tüketiciye iletilir ve bu tüketici mesajları işlemek için callback fonksiyonları kullanır.

<b>1. Parametre - String queue: </b> Mesajların alınacağı queue'nun adını belirtir. Bu, RabbitMQ sunucusunda tanımlanmış 
bir kuyruktur. Tüketici bu kuyruktan mesajları dinler ve gelen her mesaj, belirtilen callback fonksiyonuna iletilir. Eğer
bu isimle bir kuyruk yoksa, RabbitMQ bir hata döndürebilir.

<b>2. Parametre - boolean autoAck: </b> (automatic acknowledgment) mesajların RabbitMQ sunucusuna otomatik olarak onaylanıp 
onaylanmayacağını belirler.

`true` Mesajlar otomatik olarak onaylanır. Yani, mesajın tüketiciye ulaştığı anda RabbitMQ, mesajın başarıyla işlendiğini 
varsayar ve bu mesaj kuyruktan silinir.

`false` Mesajlar otomatik olarak onaylanmaz. Bu durumda, tüketici mesajı aldıktan sonra açıkça mesajın işlendiğini 
belirtmek için basicAck metodunu çağırmalıdır. Bu, mesajın güvenli bir şekilde işlendiğinden emin olunmasını sağlar. 
Eğer mesaj onaylanmazsa ve tüketici çökerse, mesaj yeniden işlenebilir.

> **Not**
>
>  Eğer mesajların doğru bir şekilde işlendiğinden emin olmak istiyorsak ve tüketici çökerse mesajları yeniden işlemek 
>  istiyorsak, `autoAck'i false yapmalıyız`. Bu durumda mesajlar açık bir onaylama (acknowledgment) işlemi ile onaylanmalıdır.
>
{style="info"}

<b>3. Parametre - DeliveryCallback deliveryCallback: </b> Bu callback fonksiyonu, mesajlar teslim edildiğinde çağrılır. 
Mesajlar bu callback ile işlenir. DeliverCallback, bir mesaj teslim edildiğinde (basic.deliver) çağrılan bir functional 
interface'dir. Bu fonksiyon, mesajın gövdesini (body) ve diğer meta verileri alır.

<b>4. parametre - CancelCallback cancelCallback: </b> Bu callback fonksiyonu, tüketici iptal edildiğinde çağrılır. RabbitMQ sunucusu, 
bir tüketiciyi çeşitli nedenlerle iptal edebilir (örneğin, kuyruk silindiğinde veya RabbitMQ sunucusu kapandığında).