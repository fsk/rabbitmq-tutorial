# RabbitMQ - Producer Java

Basit bir java projesi açıp (build olarak maven seçelim) içerisine aşağıdaki dependency'yi ekleyelim.

```XML
<dependency>
    <groupId>com.rabbitmq</groupId>
    <artifactId>amqp-client</artifactId>
    <version>5.22.0</version>
 </dependency>
```

Send.java class'ı aşağıdaki gibidir. Bu class producer olarak çalışacaktır.

```Java
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
public class Send {

    private final static String QUEUE_NAME = "hello";
    public static void main(String[] args) throws Exception {

        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("localhost");
        connectionFactory.setPort(1905);
        try(Connection connection = connectionFactory.newConnection()) {
            Channel channel = connectionFactory.newConnection().createChannel();
            channel.queueDeclare(QUEUE_NAME, false, false, false, null);
            String message = "Hello Rabbit.!";
            channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
            System.out.println(" [x] Sent '" + message + "'");
        }

    }
}
```

Kodları teker teker inceleyelim.

<b>-></b> Bir connectionFactory kuruyoruz ve bu kurduğumuz connectionFactory üzerinden bağlanacağımız ip adresi ile portu veriyoruz.

```Java
ConnectionFactory connectionFactory = new ConnectionFactory();
connectionFactory.setHost("localhost");
connectionFactory.setPort(1905);
```

<b>-></b> Bir channel açıyoruz.

```Java
Channel channel = connectionFactory.newConnection().createChannel();
```


<b>-></b> Bir queue declare ediyoruz. Yani tanımlıyoruz.

```Java
channel.queueDeclare(QUEUE_NAME, false, false, false, null);
// Bu kodun syntaxı aşağıdaki gibidir.
// Queue.DeclareOk queueDeclare(String queue, boolean durable, boolean exclusive, boolean autoDelete,
//                         Map<String, Object> arguments) throws IOException;
```

* <b>1. parametre - String queue: </b> Queue'nun adını belirtir. Eğer boş bir isim verilirse(""), rabbitMq rastgele bir 
isimle bir kuyruk oluşturur ve geri döner. Eğer zaten belirtilen isimde bir kuyruk varsa, RabbitMQ aynı kuyruk 
ayarlarıyla kuyruğu tanımlamaya devam eder. Aksi halde bir hata oluşur.
* <b>2. parametre - boolean durable: </b> Queue'nun dayanıklı olup olmadığını belirtir. 
`Eğer true ise, queue RabbitMQ sunucusu yeniden başlatıldığında bile varlığını korur.` Yani sunucu kapandığında kaybolmaz. 
`Eğer false ise, kuyruk yalnızca RabbitMQ'nun çalıştığı süre boyunca var olur ve sunucu kapandığında silinir.`
* <b>3. parametre - boolean exclusive: </b> Queue'nun sadece bu bağlantıya özel olup olmadığını belirler. 
`Eğer true ise, queue yalnızca bu bağlantı (connection) tarafından kullanılabilir. Bağlantı kapatıldığında queue otomatik olarak silinir.` 
Genelde geçici kuyruklar için kullanılır. `Eğer false ise, kuyruk diğer bağlantılarla da paylaşılabilir.`
* <b>4. parametre - boolean autoDelete: </b>  Queue'nun otomatik silinme davranışını belirler. 
`Eğer true ise, queue artık kullanılır durumda olmadığında (örn. bağlanan tüm tüketiciler ayrıldığında) otomatik olarak silinir.` 
`Eğer false ise, kuyruk kullanılmıyor olsa bile varlığını sürdürür.`
* <b>5. parametre - Map<String, Object> arguments: </b> Queue'ya özel ek yapılandırma parametrelerini içerir. 
Bu parametreler, RabbitMQ'nun çeşitli özelliklerini (örn. mesaj zaman aşımı, mesaj boyutu sınırlamaları vb.) kontrol etmek için kullanılabilir

### 5.parametredeki argümanlara örnekler:

```Java
Map<String, Object> args = new HashMap<>();

//x-message-ttl
//Bu özellik, kuyruktaki mesajların ne kadar süre boyunca canlı 
//kalacağını (milisaniye cinsinden) belirler. 
//Belirtilen süre sonunda mesajlar otomatik olarak kuyruktan silinir.
args.put("x-message-ttl", 60000);


//x-expires (Queue Expiry)
//Bu özellik, kuyruk kullanılmadığı takdirde ne kadar süre sonra 
//silineceğini belirler. Kuyruğun belirli bir süre boyunca 
//kullanılmaması durumunda RabbitMQ kuyrukları otomatik olarak siler.
args.put("x-expires", 1800000);

//x-max-length (Max Queue Length)
//Bu özellik, kuyrukta en fazla kaç mesajın saklanabileceğini belirler
//Eğer belirtilen sayıya ulaşıldıysa, yeni mesajlar geldikçe 
//eski mesajlar silinir (FIFO mantığı).
args.put("x-max-length", 1000);

// ---ve dahası---
```
> **queueDeclare Methodu Hakkında**
>
> Bu method overload olmuş bir methoddur. Yani bir implementasyonu daha vardır.
> ```Java
> queue.declareQueue()
> ```
> şeklinde kullanılabilir.
> 
> * **Queue Name:** Queue'nun adı RabbitMQ sunucusu tarafından otomatik olarak atanır.
> * **Exclusive:** Bu queue yalnızca queue'nun oluşturulduğu bağlantı (connection) tarafından kullanılabilir. 
> Diğer bağlantılar bu kuyruğa erişemez.
> * **Auto-delete:** Queue, kullanılmadığı zaman (örneğin, tüketici bağlantıları kapatıldığında) otomatik olarak silinir.
>* **Non-durable:** Queue, RabbitMQ sunucusu kapatıldığında ya da yeniden başlatıldığında silinir. Yani, queue kalıcı değildir.
>
{style="note"}


<b>-> </b> Aşağıdaki kodla rabbitMQ sunucusuna bir mesaj yayınlıyoruz.

```Java
channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
```

Bu method belirli bir `exchange` üzerinden yönlendirilir ve belirtilen `routingKey` ile iletilir. RabbitMQ'nun mesaj 
yönlendirme ve işleme sistemi bu parametrelerle kontrol edilir.

<b>1. Parametre - String exchange: </b> Mesajın yönlendirileceği exchange adını belirtir. Exchange, RabbitMQ'nun mesajları
belirli kuyruklara yönlendirmek için kullandığı bileşendir. Yaygın olarak kullanılan exchange türleri aşağıdaki gibidir.
* **`Direct Exchange:`** Mesajlar doğrudan routing key ile eşleşen kuyruğa yönlendirilir.
* **`Fanout Exchange:`** Mesajlar, routing key'e bakılmaksızın tüm bağlı kuyruklara yönlendirilir.
* **`Topic Exchange:`** Routing Key desenlerine göre mesajları yönlendirir. (örn. logs.info, logs.error).
* **`Headers Exchange:`** Routing key'e değil, mesajın başlıklarına göre yönlendirme yapılır.

> **Önemli Not**
>
>  Eğer `exchange` parametresi yanlış veya mevcut değilse, RabbitMQ bir kanal düzeyinde hata verir ve bu hata kanalı 
>  kapatır. Bu nedenle geçerli bir exchange adı kullanılmalıdır.
>
{style="warning"}

<b>2. parametre - String routingKey: </b> Mesajların yönlendirilmesi için kullanılan anahtardır. Routing Key, exchange'in
mesajı hangi queue'ya yönlendireleceğini belirlemek için kullanılır. Bu anahtar, genellikle mesajın hedeflenmiş olduğu 
kuyruğun adı ya da bir deseni olabilir. `Direct Exchange` için, bu routing key'in bir queue adıyla eşleşmesi gerekir.
`Topic Exchange` için, routing key belirli bir desene göre işlenir. Örneğin, logs.info veya logs.error gibi anahtarlar 
kullanılabilir.

<b>3. parametre - BasicProperties props: </b> Mesajın ek özelliklerini ve meta verilerini içerir. 
RabbitMQ'da BasicProperties, `mesajın başlıkları (headers), zaman aşımı (expiration), öncelik (priority), içerik türü (content type)`
gibi özellikleri ayarlamak için kullanılır.
Özellikleri
* **`contentType:`** Mesajın içeriğinin türü (örn. application/json, text/plain).
* **`deliveryMode:`** Mesajın kalıcılığını ayarlamak için kullanılır (örn. kalıcı mesajlar sunucu yeniden başlatıldığında bile saklanır).
* **`priority:`** Mesajın kuyruğa göre önceliği (örn. yüksek öncelikli mesajlar önce işlenir).
* **`headers:`** Mesajı yönlendirmek için ek başlık bilgileri.

<b>4. parametre - byte[] body: </b> Mesajın içerik kısmıdır. Gönderilecek olan gerçek mesajın bayt dizisi (byte array) olarak iletilir.
Mesajın içeriği herhangi bir veri türünde olabilir (metin, JSON, XML, ikili veri vb.). Ancak, bu verinin bayt dizisi olarak gönderilmesi gerekir.

> **channel.basicPublish Methodu Hakkında**
>
> ```Java
> void basicPublish(String exchange, String routingKey, 
> boolean mandatory, BasicProperties props, byte[] body) throws IOException;
> ```
> Diğer implementasyonundan farklı olarak burada bir mandatory flag'i bulunmaktadır.
> 
> **boolean mandatory:** mesajın en az bir kuyruk tarafından alınmasını zorunlu kılar.
> 
> ``true``: Eğer mesajın belirlenen routing key ile eşleşen hiçbir queue yoksa, mesaj sunucu tarafından geri döner. 
> Bu durumda, mesaj kaybolmaz ve publisher'a bir hata geri bildirilir. publisher, mesajın hiçbir kuyruğa yönlendirilmediğini öğrenmiş olur.
> 
> ``false``:  Eğer mesaj için bir queue bulunamazsa (yani routing key ile eşleşen bir kuyruk yoksa), mesaj sunucu 
> tarafından sessizce atılır, herhangi bir geri bildirim verilmez.
> 
> Eğer mesajın mutlaka bir queue'da işlenmesi gerekiyorsa, mandatory flagini true yapılmalıdır. Aksi takdirde, mesajın 
> hiçbir kuyrukla eşleşemeyip kaybolması riski vardır.
> 
> `Direct Exchange` veya `Topic Exchange` kullanıyorsanız ve mesajın bir queue ile eşleşmesi gerekiyorsa, mandatory 
> bayrağını kullanmak önemlidir.
>
{style="note"}