# Basitçe RabbitMQ ve NodeJS Receiver

RabbitMQ'ya gönderilen bir mesaj, ilgili kuyruğu dinleyen bir listener ya da receiver olduğunda, ilgili mesaj queue'dan 
kalkar.

Node.JS uygulamamıza aşağıdaki bağımlılıkları ekleyelim.

* `npm install amqplib express`
* `npm install nodemon --save-dev`

şimdi `index.js`dosyamızı yazmaya başlayalım.

````JavaScript
const express = require('express');
const amqp = require('amqplib');

const app = express();
const PORT = 3000;

app.use(express.json());

app.listen(PORT, () => {
    console.log(`Application start..`);
});
````

Burada `amqplib` ile projemize rabbit'i dahil ettik.

**1.ADIM** <br/>
Önce connection bilgilerini yazalım.

````JavaScript
//RabbitMQ Connection Bilgileri
const rabbitmqUrl = 'amqp://localhost:5673';
const queueName = 'first-queue';
````

**2.ADIM** <br/>
Sonra mesajımızı receive edecek methodu geliştirelim.

````JavaScript
async function receiveMessage() {
    try {
        const connection = await amqp.connect(rabbitmqUrl);
        const channel = await connection.createChannel();

        await channel.assertQueue(queueName);

        console.log(`Connection is successfully`);

        await channel.consume(queueName, (data) => {
            if (data) {
                const message = JSON.parse(data.content.toString());
                console.log(`Received Message ==> ${message}`);
                channel.ack(data);
            }
        });

    }catch (error) {
        console.error(`Error ==> ${error}`);
    }
}
````
**3.ADIM** <br/>
Uygulama çalıştığında mesajımızı okuması için methodumuzu çağıralım.

````JavaScript
app.listen(PORT, () => {
    console.log(`Application start..`);
    receiveMessage();
});
````

Spring Boot tarafından bir mesaj atıldığında direkt olarak Queue'ya düşer ve anında buradaki Node.js uygulaması queue'yu
dinlediği için mesajı hemen okur. Bu yüzden queue'daki message boş kalır.
