# RabbitMQ Queuelarına Derinlemesine Bakış

````Java
@Bean
public Queue queue() {
    return new Queue("first-queue");
}
````

Bir queue tanımı en basit haliyle aslında yukarıdaki gibi oluyor. Aslında bu method sadece queue name'i parametre olarak
almıyor. Parametre olarak overload edilmiş 5 farklı hali mevcut.

* String name
* String name, boolean durable
* String name, boolean durable, boolean exclusive, boolean autoDelete
* String name, boolean durable, boolean exclusive, boolean autoDelete, Map<String, Object> arguments

### QUEUE'NUN ALDIĞI PARAMETRELER 
### name parametresi
Bu parametreyi zaten biliyoruz aslında. En basit şekliyle bir queue adı verip oluşturuyoruz.
Ama burada şöyle bir durum var. Sadece queue adı ile bir Queue instance'ı oluşturulursa, durable özelliği true olarak,
diğer parametreler ise false olarak ayarlanıyor. Ayrıca queue name'leri en fazla 255 karakterden oluşabilir.

> **DİKKAT..!**
>
>  **`amq.`** ile başlayan queue adları, broker tarafından dahili kullanım için ayrılmıştır. Bu kuralı ihlal eden bir 
> adla bir queue bildirme girişimleri, `403 (ACCESS_REFUSED)` yanıt koduyla channel düzeyinde bir exceptiona neden olur.
>
{style="warning"}

> **NOT**
>
> Normalde bir queue oluştururken bir isim veririz. Ancak isim yerine `boş bir metin ("")` gönderirsek, RabbitMQ bizim 
> için otomatik, benzersiz bir queue ismi üretir. Bu queue adını ileride tekrar kullanmak istersek, aynı channel içinde 
> yine queue adı yerine boş string ("") vererek son oluşturulan bu benzersiz isimli queue'ya ulaşabiliriz. 
> Sistem channel üzerinde "en son hangi otomatik isimli kuyruğu verdim?" diye hatırlar.
>
{style="note"}

### durable parametresi
Durable kelime manası olarak 'dayanıklı' anlamına gelir. Bir şeyin "durable" olması, onun zorlu koşullara veya 
beklenmeyen durumlara karşı dayanıklı ve kalıcı olmasını ifade eder. eğer durable true olursa server yeniden başlatılsa 
bile queue'nun korunacağı anlamına gelir.

### exclusive parametresi
Normalde, bir queue birden fazla producer tarafından mesaj alabilir ve birden fazla consumer de bu queue'yu dinleyebilir. 
Ancak bazen bir queue'nun tek bir bağlantıya, yani tek bir consumer'a özel olarak tahsis edilmesini isteyebiliriz. 
İşte bu noktada `exclusive queue` kavramı devreye girer.

### autoDelete parametresi
Queue artık kullanılmadığında server tarafından silinmesi gerekiyorsa true olarak ayarlanır.

### arguments parametresi
Queue tanımlanırken kullanılan argümanlar. 
En yaygın kullanılan argümanlar aşağıdadır.

* **`x-message-ttl:`** Mesajın queue'da ne kadar süre (milisaniye cinsinden) bekleyebileceğini belirleyen 
Time-To-Live (TTL) değeridir. Bu süre dolduğunda mesaj `dead-letter exchange`’e yönlendirilir (eğer DLX ayarı varsa) 
ya da silinir.
* **`x-expires:`** Kuyruğun kendisinin ne kadar süre boş kaldıktan sonra silineceğini belirler. Milisaniye cinsindendir.
* **`x-max-length:`** Kuyruktaki maksimum mesaj sayısını belirler. Bu sayı aşıldığında en eski mesajlar atılır veya 
DLX'e yönlendirilir.
* **`x-max-priority:`** Kuyrukta mesaj önceliklendirme özelliğini etkinleştirir ve en yüksek öncelik seviyesini belirtir.
Bu seviyeler 1 - 10 arasındadır. 
* **`x-dead-letter-exchange:`** Queuedaki mesajlar belirli koşullarda (örneğin TTL süresi dolduğunda veya queuenun tam 
kapasiteye ulaşması sonucu) başka bir exchange'e yönlendirilebilir. Bu argüman, bu yönlendirme için kullanılacak 
Dead Letter Exchange’in (DLX) adını belirler.

> **DİKKAT..!!**
>
> `autoDelete` ve `exclusive` özellikleri genellikle geçici ve sadece belirli bir uygulama ya da bağlantı için anlamlıdır. 
> Bu nedenle, bu tür queue'ların isimlerini de genelde sistemin (RabbitMQ'nun) kendisinin oluşturulması aslında güzel bir
> practicedir. Yani bu da queue ismini sabit, bilinen bir isimle değil, sistemin otomatik olarak oluşturduğu rastgele bir isimle 
> kullanmak demektir. Böylece her bağlantı için eşsiz ve geçici bir queue oluşur.
> 
> Sunucu (RabbitMQ), her seferinde benzersiz (örneğin "amq.gen-xxxxxxxx") tarzda bir queue ismi oluşturabilir. Bu isimler 
> rastgele ve benzersizdir. Bu sayede şu avantajlar ortaya çıkar:
> * Her bağlantıya özel, çakışmayan bir queue ismi olur.
> * Queue kapandığında (bağlantı kesildiğinde) sorun çıkmaz.
> 
> **Peki Sorun Nerede Ortaya Çıkar** <br/>
> Eğer `auto-delete` ve `exclusive` özelliklerini kullanıp, ama yine de queue'ya sabit bir isim 
> (mesela "my_temporary_queue") verirsek, şu durum gerçekleşebilir:
> * Uygulama bu queue'ya bağlandı ve çalıştı.
> * Sonra uygulama bir sebeple bağlantısını kopardı (mesela ağ gitti, uygulama yeniden başlatıldı).
> * Hemen ardından uygulama tekrar bağlanmaya çalıştı ve aynı isimle queue'yu tekrar oluşturmak istedi.
> * Bu arada, RabbitMQ sunucusu, önceki bağlantı koptuğu için `auto-delete` özelliğinden dolayı o queue'yu silmeye 
> çalışıyor olabilir.
> 
> **İşte tam bu noktada bir `RACE CONDITION`durumu meydana gelebilir.**
> * RabbitMQ queue'yu silmek istiyor (çünkü bağlantı koptu, "auto-delete" devrede).
> * Uygulama ise aynı isimle queue'yu yeniden oluşturmak istiyor (çünkü tekrar bağlandı ve sabit bir kuyruk adı kullanıyor).
>
{style="warning"}

**SORU** <br/>
**Docker'da bir rabbitMQ ayağa kaldırıp durable olan bir kuyruk oluşturduk diyelim. Bu kuyruğun bilgilerini diskte nasıl görebiliriz.?** <br/>
RabbitMQ, queue bilgilerini ve metadatasını disk üzerinde `Mnesia` adı verilen dahili bir veritabanında saklar. Durable 
queue bilgileri burada bulunur. Ancak bu veriler düz metin dosyaları şeklinde değil, özel bir formatta depolanır. 
Yani doğrudan bir dosyayı açıp kuyruk isimlerini, mesajları veya yapılandırmaları metin olarak görüntüleyemeyiz

Ama queue bilgilerini almanın bir yolu vardır.

`docker exec -it <container_name> rabbitmqctl list_queues name durable messages`

Bu komut ile docker'da ayağa kaldırdığımız rabbitMQ'da oluşturulan queue bilgilerini görebiliriz.

### argümanlarla çalışmak
`Map<String, Object> arguments`parametresi ile argümanları alıp parametre olarak Queue'ya verebiliriz.

```Java
@Bean
public Queue firstQueue() {
    Map<String, Object> args = new HashMap<>();
    /**
     * x-message-ttl ==> Mesajın queue'da ne kadar süre (milisaniye cinsinden) bekleyebileceğini belirleyen
     * Time-To-Live (TTL) değeridir. Bu süre dolduğunda mesaj dead-letter exchange ’e yönlendirilir
     * (eğer DLX ayarı varsa) ya da silinir.
     */
    args.put("x-message-ttl", 100000);
    /**
     * x-expires ==> Queue'un ne kadar süre (milisaniye cinsinden) boşta kalabileceğini belirleyen
     * expiration değeridir. Bu süre dolduğunda queue silinir.
     */
    args.put("x-expires", 200000);
    /**
     * x-max-length ==> Queue'da en fazla kaç adet mesaj tutulabileceğini belirleyen maksimum mesaj sayısıdır.
     */
    args.put("x-max-length", 10);
    return new Queue(FIRST_QUEUE, true, false, false, args);
}
```

Bu şekilde kodu geliştirip, args parametresine verdiğimiz değerlerle birlikte kodu ayağa kaldırdığımız zaman aşağıdaki gibi
bir cURL istekleri atabiliriz.

````cURL
curl -X POST \
  http://localhost:8082/send \
  -H 'Content-Type: application/json' \
  -d '{"message": "Merhaba RabbitMQ!"}'

````

````cURL
curl -X POST \
  http://localhost:8082/send \
  -H 'Content-Type: application/json' \
  -d '{"message": "Turkiye Java Community!"}'

````

Bu Curl isteklerini attığımızda aşağıdaki fotoğraftaki gibi queue'muzda mesajları görebiliriz. 

![rabbitmq-fifth.png](rabbitmq-fifth.png)

`args.put("x-message-ttl", 100000);` vermiş olduğumuz bu argümandan dolayı mesajlar 100000ms (100 sn) sonra silinecektir.
Öncelikle `{"message": "Merhaba RabbitMQ!"}` bu mesaj, ardından da `{"message": "Turkiye Java Community!"}` bu mesaj
silinip queue boşalacaktır.

Aşağıdaki görselde de bunu görebiliriz.

![rabbitmq-sixth.png](rabbitmq-sixth.png)

`args.put("x-expires", 200000);` vermiş olduğumuz bu parametreden dolayı da, 200 sn sonra queue'nun otomatik olarak silindiğini
görebiliriz. Tabi bunu görmek için ekranı yenilemek gerekir.

![rabbitmq-seventh.png](rabbitmq-seventh.png)




