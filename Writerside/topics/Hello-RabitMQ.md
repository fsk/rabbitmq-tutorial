# Hello RabitMQ

RabbitMQ bir message broker'dır. Mesajları kabul eder ve iletir. Bunu bir post office gibi düşünebiliriz. Postalamak
istediğimiz mektubu posta kutusuna koyduğumuzda, postacının sonunda mektubu alıcımıza teslim edeceğinden emin olabiliriz.
RabbitMQ aslında hem posta kutusu, hem postacı hem de posta ofisidir.

<b>Posta Kutusu:</b> Mesajların gönderildiği yerdir. Gönderici sistem, mesajı RabbitMQ'ya ilettiğinde, bu mesaj rabbitMQ'nun
bir queue'suna yerleştirilir. Yani bir posta kutusu olarak düşünülebilir.

<b>Posta Ofisi:</b> Mesajları yönlendiren ve organize eden merkez.RabbitMQ gelen mesajları çeşitli alıcılara iletmekle
sorumludur. Tıpkı posta ofislerinin gelen mektupları alıcı adreslerine göre ayırması gibi.

<b>Postacı:</b> Mesajarı alıcıya teslim eden kişi. RabbitMQ mesajı bir alıcıya gönderirken, postacının mektubu alıcıya
teslim etmesi gibi çalışır. Alıcı, sistemi uygun gördüğü anda mesajı alır ve işler.

<b>İletim Süreci:</b> RabbitMQ mesajları güvenli bir şekilde alır ve iletir. Tıpkı bir mektubu posta kutusuna attığımızda, 
mektubun eninde sonunda alıcıya ulaşağına güvendiğimiz gibi, RabbitMQ'ya verilen mesajlar da hedeflerine ulaşacaklardır.
RabbitMQ mesajların iletilmesi ve kaybolmaması için sorumluluk alır.

RabbitMQ ile postane arasındaki en büyük fark, onun kağıt işleriyle uğraşmaması, bunun yerine binary mesajlar 
kabul etmesi, depolaması ve iletmesi