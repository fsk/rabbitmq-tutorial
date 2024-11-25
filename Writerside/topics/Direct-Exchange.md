# Direct Exchange

Direct Exchange, en basit ve anlaşılır exchange türüdür. Bu exchange, bir mesajın yalnızca routing key ile tam eşleşen bir
kuyrukla ilişkilendirilmesini sağlar. Her mesaj, yalnızca routing key’in eşleştiği kuyruklara gider.

## Nasıl Çalışır

* Producer mesajı bir routing key ile birlikte Direct Exchange’e gönderir.
* Exchange, mesajın routing key'ine bakar ve o key ile tam olarak aynı olan kuyruklara mesajı yönlendirir

## Kullanım Senaryosu

Örneğin, bir e-ticaret sisteminde sipariş durumu hakkında bilgiler gönderen bir uygulama olduğunu düşünelim.
Her sipariş durumu (örneğin, "hazırlanıyor", "yolda", "teslim edildi") farklı kuyruklara yönlendirilmelidir. Bu durumda:
* Sipariş "hazırlanıyor" durumu için bir routing key "preparing" olarak belirlenir.
* Sipariş "yolda" durumu için bir routing key "shipped" olarak belirlenir.
* Bu routing key’ler ile Direct Exchange, ilgili kuyruklara mesajları yönlendirir.