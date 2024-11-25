# Headers Exchange

Mesaj yönlendirme için routing key yerine mesaj başlıklarını (headers) kullanır. Bu exchange türü, mesajın başlıkları 
(header) ile kuyrukların belirlediği kriterleri (header bindings) karşılaştırarak mesaj yönlendirir.

## Nasıl Çalışır
* Producer, mesajı bazı headers (başlıklar) ile birlikte Headers Exchange’e gönderir.
* Exchange, bu başlıkları inceler ve bağlı kuyrukların başlık şartlarına (header binding) göre mesajı yönlendirir.
* Headers üzerinde eşleşmeler yapılırken, x-match parametresi kullanılır: <br></br>
<b>-></b> `x-match = all`: Tüm başlıklar eşleşmelidir.<br></br>
<b>-></b> `x-match = any`: Başlıklardan herhangi birisi eşleşirse mesajlar yönlendirilir.

## Kullanım Senaryosu

Mesajlarınızın içeriğine göre çok spesifik yönlendirmeler yapmak istiyorsanız Headers Exchange kullanabilirsiniz. 
<b>Örn</b>

Bir mesaj başlığı olarak `"department": "HR"` ve `"location": "NY"` olabilir.
Bu başlıklarla, yalnızca bu iki şartı karşılayan kuyruklara mesaj gönderebilirsiniz.
