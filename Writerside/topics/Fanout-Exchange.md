# Fanout Exchange

Herhangi bir routing key kullanmadan mesajları tüm bağlı kuyruklara yönlendirir. Bu exchange türü, mesajı hangi 
kuyrukların alacağına dair hiçbir kural uygulamaz. Mesaj, exchange’e bağlı olan tüm kuyruklara otomatik olarak iletilir.

## Nasıl Çalışır
* Producer, herhangi bir routing key belirtmeden mesajı Fanout Exchange’e gönderir.
* Exchange, bu mesajı kendisine bağlı olan tüm kuyruklara iletir.

## Kullanım Senaryosu
Birden fazla servise aynı anda mesaj göndermek istediğiniz durumlar için çok uygundur.

<b>Örn</b>

Bir spor haber portalı, canlı bir spor etkinliği olduğunda bir Fanout Exchange kullanarak bu etkinliği tüm abonelere 
gönderebilir. Tüm kullanıcılar aynı yayını alır. Bu tür exchange, yayın yapma (broadcast) senaryoları için idealdir.