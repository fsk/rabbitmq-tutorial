# Basic Concepts

## Producer
Producer ya da producing aslında mesaj gönderen demek. Bir program mesaj gönderiyorsa o producer'dır.

## Queue
Bir queue, rabbitMQ'da posta kutusunun adıdır. Yani mesajların depolandığı yerdir. Tıpkı fiziksel bir posta kutusuna 
benzetildiği gibi, gelen tüm mesajlar bu kutuya bırakılır ve orada tutulur.
Mesajlar RabbitMQ aracılığıyla sistemler arasında dolaşabilir, ancak geçici olarak ya da kalıcı bir şekilde yalnızca 
kuyrukta saklanabilir.
Kuyruk, bir tampon (buffer) gibi çalışır; yani mesajlar, alıcıları hazır olana kadar orada bekletilir.

## Consumer
Consumer mesajları almak için bekleyen programlardır