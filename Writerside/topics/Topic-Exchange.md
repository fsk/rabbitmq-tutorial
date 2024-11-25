# Topic Exchange

Topic Exchange, routing key'ler üzerinde desen tabanlı bir yönlendirme sunar. Yani, routing key’ler belirli bir desenle
(pattern) eşleştiğinde mesajlar ilgili kuyruklara yönlendirilir. Burada, routing key karakter dizileri arasında
joker karakterler kullanılır.

## Nasıl Çalışır
* Producer, mesajı bir routing key ile Topic Exchange'e gönderir.
* Exchange, bu routing key'i kendisine bağlı kuyrukların `binding key`desenleri ile karşılaştırır.
* Eğer eşleşme varsa mesaj o kuyruklara yönlendirilir.
* Burad iki farklı joker karakter kullanılır.

<b>-> *</b> Tek bir kelimeyi temsil eder. Routing key’de bir kelimelik herhangi bir bölümle eşleşir.

<b>-> #</b> Sıfır veya daha fazla kelimeyi temsil eder. Routing key’de birden fazla kelimelik bir bölümle eşleşebilir.

## Kullanım Senaryosu

Bir haber yayını sistemi düşünelim. Her haberin bir kategorisi ve bölgesi olabilir. Örneğin:

Routing key olarak "sports.us" kullanıldığında, bu ABD'deki spor haberlerini temsil eder.
Routing key olarak "politics.eu" kullanıldığında, bu Avrupa'daki siyasi haberleri temsil eder.

Queue'lar ise, binding key ile bu haber kategorilerine abone olabilir:

* **"sports.*":** Sadece sporla ilgili tüm haberleri alır, yani sports.us, sports.eu gibi her şeyi.
* **"*.eu":** Avrupa’daki her türlü haberi alır, yani sports.eu, politics.eu vb.
* **"#":** Tüm haberleri alır, çünkü # joker karakteri her şeyi kapsar.

## Desen Tabanlı Yaklaşıma Derinlemesine Bir Bakış

`"Desen tabanlı yönlendirme"` ifadesi, Topic Exchange ile mesaj yönlendirme sırasında routing key'lerin belirli bir kalıba 
göre eşleştirildiğini ifade eder. Bu kalıplar, belirli bir yapı izleyen anahtar kelimelerden oluşur ve bu yapı sayesinde 
mesajlar, konulara (topics) göre dinamik olarak farklı kuyruklara yönlendirilebilir.

Bu yönlendirme, RabbitMQ'da Topic Exchange kullanıldığında ortaya çıkar ve özellikle çok sayıda kuyruk arasında esnek bir 
mesaj yönlendirme yapısı kurmak istediğimizde işe yarar. Desen tabanlı yönlendirme, routing key’lerin bir pattern (desen) 
ile bağlandığı ve kuyrukların bu desenlere göre mesajları aldığı bir mekanizmadır.

### RabbitMQ'da Joker Karakterler
#### (*)
Bir kelimelik herhangi bir değeri temsil eder. Routing key’de tek bir kelimeyle uyumlu olan yerlerde kullanılır.

<b>Örn</b>

**`Desen:`** "order.*"

**`Eşleşebilecek Routing Keyler:`** "order.created", "order.canceled"

**`Eşleşemeeycek Routing Keyler:`** "order.shipping.started" (çünkü bu key üç kelime içerir ve yıldız sadece tek bir 
kelime ile eşleşir).


#### (#)
Sıfır veya daha fazla kelimeyi temsil eder. Yani bu karakter, routing key’de çok sayıda kelimeyi kapsayabilir.

<b>Örn</b>

**`Desen:`** "order.#"

**`Eşleşebilecek Routing Keyler:`** "order", "order.created", "order.canceled", "order.shipping.started"