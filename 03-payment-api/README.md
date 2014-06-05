<== [Pengenalan Sistem Sandbox Veritrans](../02-sandbox/README.md)

3. Veritrans Payment API
=========================================

Pada BAB ini kita akan mulai pembahasan secara teknis (Development), bagaimana standard API Veritrans Indonesia dan bagaimana Merchant dapat menggunakannya.

## 3.1 RESTful API

Komunikasi yang dapat kita lakukan dengan Veritrans Payment API adalah menggunakan [RESTful Web Service](http://en.wikipedia.org/wiki/Representational_state_transfer). Secara garis besar RESTful Web Service hanyalah HTTP REQUEST dengan ketentuan yang telah ditetapkan.

Berikut adalah daftar endpoint (url) Veritrans Payment API 

- Sandbox : [https://api.sandbox.veritrans.co.id/v2/](https://api.sandbox.veritrans.co.id/v2/)
- Production : [https://api.veritrans.co.id/v2/](https://api.veritrans.co.id/v2/)

Apa perbedaan sandbox dengan production? Tidak ada yang berbeda jika dilihat dari sisi development, semua spesifikasi nya sama, yang berbeda hanya endpoint(url)-nya saja, dan tentunya sandbox untuk testing dan development sedangkan production untuk data transaksi real (asli)

### 3.1.1 API Authentication

Setiap komunikasi yang dilakukan server-to-server dari sistem Merchant ke sistem Veritrans Payment API, akan ada tahapan autentikasi yang dilakukan oleh sistem Veritrans. Veritrans Payment API menggunakan [BasicAuth](http://en.wikipedia.org/wiki/Basic_access_authentication) untuk melakukan proses authentikasinya.

Data Authentication yang dapat digunakan oleh Merchant terdapat di halaman Settings -> Access Keys di MAP.

![MAP Settings -> Access Keys](../images/image-004.png)

- Client Key : merupakan public key yang digunakan untuk proses autentikasi ke Veritrans Payment API. Client Key boleh diketahui oleh public. Salah satu penggunaannya adalah untuk proses Token API (dibahas di [BAB 4. Transaksi Kartu Kredit](../04-kartu-kredit/README.md))
- Server Key : merupakan private key yang digunakan untuk proses autentikasi ke Veritrans Payment API. Server Key tidak boleh diketahui oleh public karena Server Key digunakan untuk melakukan transaksi finansial ke Veritrans Payment API (seperti charge, cancel, dan lain-lain)

Misal jika Server Key yang kita gunakan adalah <b>11d39abc-e921-450c-945a-684670efa74d</b>, maka hasil Basic Auth nya adalah HTTP Header sebagai berikut :

```
Authorization : Basic MTFkMzlhYmMtZTkyMS00NTBjLTk0NWEtNjg0NjcwZWZhNzRkOg==
```

Dari mana hasil tersebut didapat? Dalam spesifikasi Basic Auth, terdapat username dan password yang diperlukan, di Veritrans Payment API, kita dapat menggunakan server key sebagai username dan password nya adalah text kosong. Untuk membuat header Basis Auth, kita perlu menggabungkan username dan password lalu diubah menjadi base 64. Berikut adalah formatnya :

```
Authorization : Basic Base64(username:password)
```

Seandainya username nya adalah server key 11d39abc-e921-450c-945a-684670efa74d dan passwordnya kosong, maka hasilnya adalah sebagai berikut :

```
Authorization : Basic Base64(11d39abc-e921-450c-945a-684670efa74d:)
```

> PERLU DIPERHATIKAN, bahkan terdapat tanda : (titik dua) dibelakang server key, itu adalah karakter pembatas username dan password, karena password-nya kosong, maka otomatis hanya terdapat teks server key dan tanda : (titik dua)

Hasil akhirnya jika teks "11d39abc-e921-450c-945a-684670efa74d:" diubah menjadi Base 64 adalah :

```
Authorization : Basic MTFkMzlhYmMtZTkyMS00NTBjLTk0NWEtNjg0NjcwZWZhNzRkOg==
```

### 3.1.2 Default HTTP Header

Format data yang digunakan oleh Veritrans Payment API adalah [JSON (JavaScript Object Notation)](http://www.json.org/). Sehingga secara default sistem Merchant perlu mengirim Content-Type dan Accept header dengan value application/json pada setiap request komunikasi yang dilakukan server-to-server ke Veritrans Payment API.

```
Content-Type  : application/json
Accept        : application/json
```

## 3.2 Payment Request

Seperti yang telah dibahas sebelumnya, Veritrans Payment API menggunakan JSON sebagai format data untuk melakukan komunikasi server-to-server. Berikut adalah format JSON untuk melakukan transaksi pembayaran.

```json
{
      "payment_type": "...",
      "transaction_details": {
        "order_id": "138898199031",
        "gross_amount": 1000000
      },
      "item_details": [
        {
          "id": "ITEM1",
          "price": 10000,
          "quantity": 100,
          "name": "Mie Ayam Enak"
        }
      ],
      "customer_details": {
        "first_name" : "Eko",
        "last_name" : "Khannedy",
        "phone" : "0893534534",
        "email": "eko.khannedy@gmail.com",
        "billing_address": {
          "first_name": "Eko",
          "last_name": "Khannedy",
          "address": "Jalan Raya Kalijati",
          "city": "Subang",
          "postal_code": "41271",
          "phone": "+6281 123 12345"
        },
        "shipping_address": {
          "first_name": "Eko",
          "last_name": "Khannedy",
          "address": "Jalan Raya Kalijati",
          "city": "Subang",
          "postal_code": "41271",
          "phone": "+6281 123 12345"
        }
      },
      "....": {

      }
}
```

Yuk kita bahas satu per satu, apa data yang diperlukan untuk melakukan transaksi menggunakan Veritrans Payment API.

### 3.2.1 Payment Type

```json
{
      "payment_type": "...",
}
```

```payment_type``` merupakan metode pembayaran yang akan dipilih oleh Merchant. Untuk lebih detail metode pembayaran apa saja yang tersedia, bisa dilihat di [3.6 Payment Type](#36-payment-type)

### 3.2.2 Transaction Details

```json
{
      "transaction_details": {
        "order_id": "138898199031",
        "gross_amount": 1000000
      }
}
```

```transaction_details``` merupakan detail dari transaksi yang dilakukan, yang berisikan order id dan juga gross amount transaksi.

| Atribut      | Tipe Data  | Keterangan                                                       |
|--------------|------------|------------------------------------------------------------------|
| order_id     | String(50) | Id pembayaran customer                                           |
| gross_amount | Long       | Total harga transaksi pembayaran yang akan dibayar oleh customer |

### 3.2.3 Item Details

```json
{
      "item_details": [
        {
          "id": "ITEM1",
          "price": 10000,
          "quantity": 3,
          "name": "Bakso"
        },
        {
          "id": "ITEM2",
          "price": 8000,
          "quantity": 2,
          "name": "Mie Ayam"
        }
      ]
}
```

```item_details``` merupakan kumpulan (Array) produk yang dibeli oleh pelanggan yang melakukan transaksi.

| Atribut  | Tipe Data  | Keterangan    |
|----------|------------|---------------|
| id       | String(50) | Id produk     |
| price    | Long       | Harga produk  |
| quantity | Integer    | Jumlah produk |
| name     | String(50) | Nama produk   |

### 3.2.4 Customer Details

```json
{
      "customer_details": {
        "first_name" : "Eko",
        "last_name" : "Khannedy",
        "phone" : "0893534534",
        "email": "eko.khannedy@gmail.com",
        "billing_address": {
          "first_name": "Eko",
          "last_name": "Khannedy",
          "address": "Jalan Raya Kalijati",
          "city": "Subang",
          "postal_code": "41271",
          "phone": "+6281 123 12345"
        },
        "shipping_address": {
          "first_name": "Eko",
          "last_name": "Khannedy",
          "address": "Jalan Raya Kalijati",
          "city": "Subang",
          "postal_code": "41271",
          "phone": "+6281 123 12345"
        }
      }
}
```

```customers_details``` merupakan data pelanggan yang melakukan transaksi.

| Atribut          | Tipe Data   | Keterangan                                                                                                         |
|------------------|-------------|--------------------------------------------------------------------------------------------------------------------|
| first_name       | String(20)  | Nama depan pelanggan                                                                                               |
| last_name        | String(20)  | Nama belakang pelanggan                                                                                            |
| email            | String(45)  | Email pelanggan, format email harus valid                                                                          |
| phone            | String(19)  | Nomor telpon pelanggan, format nomor harus valid, hanya diperbolehkan karaketer 0-9, dan tanda + - ( ) dan ```spasi``` |
| billing_address  | JSON Object | Billing address pelanggan                                                                                          |
| shipping_address | JSON Object | Shipping address pelanggan                                                                                         |

#### 3.2.4.1 Billing Address dan Shipping Address

Khusus untuk transaksi dengan ```payment_type``` kartu kredit, maka diperlukan data customers tambahan, yaitu ```billing_address``` dan ```shipping_address```.

| Atribut      | Tipe Data   | Keterangan                                                                                                         |
|--------------|-------------|--------------------------------------------------------------------------------------------------------------------|
| first_name   | String(20)  | Nama depan pelanggan                                                                                               |
| last_name    | String(20)  | Nama belakang pelanggan                                                                                            |
| phone        | String(19)  | Nomor telpon pelanggan, format nomor harus valid, hanya diperbolehkan karaketer 0-9, dan tanda + - ( ) dan [spasi] |
| address      | String(200) | Alamat pelanggan                                                                                                   |
| city         | String(20)  | Kota pelanggan                                                                                                     |
| postal_code  | String(10)  | Kode pos pelanggan                                                                                                 |
| country_code | String(10)  | Kode negara sesuai dengan [ISO 3166-1_alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3)                    |

### 3.2.5 Data Apa Saja yang Wajib Dikirim?

Begitu banyak data yang diperlukan, apa saja yang wajib dikirim dan apa saja yang tidak wajib dikirim ke Veritrans Payment API?

![JSON Format](../images/image-005.png)

- Semua yang berwarna <b>merah</b> adalah atribut yang wajib dikirim oleh merchant untuk melakukan transaksi di Payment API.
- Semua yang berwarna <b>biru</b> adalah atribut yang wajib dikirim khusus untuk tipe pembayaran kartu kredit
- Semua yang berwarna <b>hitam</b> adalah atribut yang tidak wajib (optional) dikirim oleh merchant, namun direkomendasikan untuk dikirim

## 3.3 Payment Response

Veritrans Payment API memiliki format pesan response yang standard untuk semua API request. Berikut adalah contoh JSON format untuk response transaksi yang sukses :

```json
{
    "status_code": "200",
    "status_message": "OK, Credit Card transaction is successful",
    "transaction_id": "fb8b09f9-aca5-4e4b-a921-7d0e9e85eb73",
    "order_id": "138898199032",
    "payment_type": "credit_card",
    "transaction_time": "2014-06-04 13:40:41",
    "transaction_status": "capture",
    "fraud_status": "accept",
    "gross_amount": "1000000.00"
}
```

dan berikut adalah contoh JSON format untuk response gagal :

```json
{
    "status_code": "406",
    "status_message": "Merchant charges the same order id more than once or transaction has been paid"
}
```

Untuk transaksi sukses, Merchant akan selalu mendapatkan ```transaction_id``` dari setiap transaksi yang dilakukan. Sangat disarankan untuk menyimpan ```transaction_id``` tersebut di sistem Merchant karena dapat digunakan untuk transaksi API lain seperti; cek status, membatalkan transaksi, dan lain-lain.

| Atribut            | Keterangan                                                                       |
|--------------------|----------------------------------------------------------------------------------|
| status_code        | Status code dari Veritrans Payment API (lihat 3.7 Status Code)                   |
| status_message     | Status message yang berisikan pesan dari Veritrans Payment API                   |
| transaction_id     | Nomor id transaksi unik di sistem Veritrans                                      |
| order_id           | Nomor id pembayaran yang dikirim oleh Merchant                                   |
| transaction_time   | Waktu transaksi terjadi                                                          |
| transaction_status | Status transaksi, apakah sukses atau gagal (lihat 3.4 Transaction Status)        |
| fraud_status       | Status pendeteksian kecurigaan akan penipuan atau bukan (lihat 3.5 Fraud Status) |
| gross_amount       | Total biaya transaksi yang dikirim oleh Merchant                                 |

## 3.4 Transaction Status

Setiap transaksi yang dikirim oleh Merchant ke Veritrans Payment API, maka akan memiliki ```transaction_status``` (Status Transaksi). Status transaksi digunakan untuk mengetahui apakah sebuah transaksi berhasil atau gagal, atau bahkan belum selesai dilakukan oleh pelanggan. Berikut adalah daftar ```transaction_status``` yang ada pada Veritrans Payment API :

- <b>authorize</b>, status transaksi ini hanya akan terjadi untuk proses kartu kredit, jika sebuah transaksi memiliki ```transaction_status``` authorize, artinya saldo sebuah kartu kredit telah ditahan, namun belum di potong.
- <b>capture</b>, status transaksi ini hanya akan terjadi untuk proses kartu kredit, jika sebuah transaksi memiliki ```transaction_status``` capture, artinya saldo sebuah kartu kredit telah dipotong, namun belum berpindah ke akun Veritrans atau Merchant. Maksudnya belum berpindah adalah, transaksi tersebut masih bisa dibatalkan.
- <b>deny</b>, status transaksi ini terjadi jika sebuah transaksi ditolak oleh Payment Provider, misal dikarenakan saldo tidak mencukupi, dan lain-lain.
- <b>pending</b>, status transaksi ini terjadi jika sebuah transaksi belum selesai dibayar oleh pelanggan. Misal untuk Virtual Account (ATM), status transaksi tidak akan langsung sukses, karena menunggu pelanggan membayar dengan cara transfer via ATM atau IB (Internet Banking).
- <b>settlement</b>, status transaksi ini terjadi jika sebuah transaksi susah sukses dan saldo dari akun pelanggan telah berpindah ke akun Veritrans atau Merchant. Jika status transaksi sudah settlement, transaksi tidak bisa dibatalkan lagi secara otomatis oleh sistem, namun harus secara manual.
- <b>cancel</b>, status transaksi ini terjadi jika sebelumnya sebuah transaksi telah sukses (authorize, capture), namun Merchant membatalkan transaksi tersebut.
- <b>refund</b>, status transaksi ini terjadi jika sebelumnya sebuah transaksi telah sukses dan saldo sudah berpindah (settlement), namun Merchant meminta untuk mengembalikan saldo pelanggan.
- <b>expire</b>, status transaksi ini terjadi jika ada transaksi yang belum dibayar oleh pelanggan (pending) dalam jangka waktu tertentu; misal transaksi Virtual Account yang tidak diselesaikan oleh pelanggan.

## 3.5 Fraud Status

Perlu diketahui jika Fraud Status hanya akan ada untuk transaksi kartu kredit, selain kartu kredit, tidak akan ada Fraud Status.

- <b>accept</b>, jika status fraud sebuah transaksi adalah accept, artinya Veritrans Fraud Detection System menyatakan bahwa transaksi tersebut tidak terdapat indikasi penipuan, sehingga cukup aman untuk Merchant.
- <b>deny</b>, jika status fraud sebuah transaksi adalah deny, artinya Veritrans Fraud Detection System menyatakan bahwa transaksi tersebut terindikasi fraud (penipuan), sehingga sangat berbahaya jika Merchant menerima transaksi tersebut.
- <b>challenge</b>, jika status fraud sebuah transaksi adalah challenge, artinya Veritrans Fraud Detection System menyatakan bahwa transaksi tersebut aman, namun ada sedikit kecurigaan mengandung fraud (penipuan). 
- <b>noscore</b>, jika status fraud sebuah transaksi adalah noscore, artinya Veritrans Fraud Detection System tidak bisa memberi hasil apapun tentang transaksi tersebut; misal dikarenakan sebuah transaksi tersebut benar-benar baru.

### 3.5.1 Apa yang harus dilakukan jika transaksi Challenge?

Khusus untuk transaksi yang fraud status nya challenge, Merchant harus memutuskannya sendiri, apakah transaksi tersebut akan diterima atau ditolak. Jika Merchant tidak mengambil keputusan, maka Veritrans Payment API akan otomatis membatalkan transaksi tersebut, sehingga hal ini perlu diperhatikan oleh Merchant.

## 3.6 Payment Type

- <b>credit_card</b> untuk transaksi Kartu Kredit

Internet Banking :

- <b>bca_klikpay</b> untuk transaksi BCA KlikPay
- <b>bca_klikbca</b> untuk transaksi KlikBCA
- <b>mandiri_clickpay</b> untuk transaksi Mandiri ClickPay
- <b>cimb_clicks</b> untuk transaksi CIMB Clicks
- <b>bri_epay</b> untuk transaksi BRI Epay

Virtual Account :

- <b>bii</b> untuk transaksi Virtual Account Bank BII
- <b>permata</b> untuk transaksi Virtual Account Bank Permata

Mobile Payment :

- <b>xl_tunai</b> untuk transaksi XL Tunai
- <b>telkomsel_cash</b> untuk transaksi Telkomsel TCash

Lainnya :

- <b>vtweb</b> untuk transaksi VTWeb
- <b>vtlink</b> untuk transaksi VTLink

## 3.7 Status Code

Status code di Veritrans Payment API terbagi menjadi beberapa bagian, yaitu; 2xx, 3xx, 4xx dan 5xx.

### 3.7.1 Status Code 2xx

Status code 2xx berarti transaksi berjalan normal, tidak terdapat masalah ataupun kesalahan yang terjadi selama transaksi berlangsung.

| Status Code | Keterangan                                                                                                                                                                      |
|-------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 200         | OK, request is success, and transaction is success (authorize, capture, settlement, cancel, refund, get order, approve challenge transactions), accepted by Veritrans and bank. |
| 201         | Pending, transaction is success but the processing has not been completed, example, challenge transaction and pending transaction.                                              |
| 202         | Denied, request is success but transaction is denied by bank or Veritrans fraud detection system.                                                                               |

### 3.7.2 Status Code 3xx

Status code 3xx berarti endpoint (url) yang digunakan sudah berubah dan tidak akan digunakan lagi.

| Status Code | Keterangan                                                                      |
|-------------|---------------------------------------------------------------------------------|
| 300         | Move Permanently, this and all future request should be directed to the new URL |

### 3.7.3 Status Code 4xx

Status code 4xx berarti terdapat kesalahan dari sisi Merchant, misal saja Merchant salah mengirim format data atau proses autentikasi gagal karena Merchant salah mengirim Server Key.

| Status Code | Keterangan                                                                                                                            |
|-------------|---------------------------------------------------------------------------------------------------------------------------------------|
| 400         | Validation Error, merchant send bad request dataexample; validation error, invalid transaction type, invalid credit card format, etc. |
| 401         | Access denied due to unauthorized transaction, please check client key or server key                                                  |
| 402         | Merchant doesn't have access for this payment type                                                                                    |
| 403         | The requested resource is only capable of generating content not acceptable according to the accept headers sent in the request       |
| 404         | The requested resource is not found                                                                                                   |
| 405         | Http method is not allowed                                                                                                            |
| 406         | Merchant charges the same order id more than once or transaction has been paid                                                        |
| 407         | Resource is no longer available and will not be available again                                                                       |
| 408         | Merchant sends the wrong data type                                                                                                    |
| 409         | Merchant has sent too many transactions for the same card number                                                                      |
| 410         | Merchant account is deactivated or not active for a long period                                                                       |
| 411         | Token id is missing, invalid, or timed out                                                                                            |
| 412         | Merchant cannot modify status of the transaction                                                                                      |
| 413         | The request cannot be processed due to malformed syntax in the request body                                                           |

### 3.7.4 Status Code 5xx

Status code 5xx berarti terdapat kesalahan di sistem Veritrans Payment API, misal saja koneksi ke Payment Provider sedang bermasalah.

| Status Code | Keterangan                                                                                             |
|-------------|--------------------------------------------------------------------------------------------------------|
| 500         | Sorry our internal system has experienced an internal error, we will fix it as soon as possible        |
| 501         | The feature has not finished yet, it will be available soon                                            |
| 502         | Bank Exception, system can not connect to bank                                                         |
| 503         | Sorry our gateway system has experienced an internal error, we will fix it as soon as possible         |
| 504         | Sorry our fraud detection system has experienced an internal error, we will fix it as soon as possible |

## 3.8 Menggunakan RestClient

Untuk mencoba melakukan transaksi, saya sarankan menggunakan aplikasi Rest Client dibandingkan langsung menggunakan bahasa pemrograman seperti Java, PHP dan yang lainnya. Dengan menggunakan Rest Client, kita bisa tahu, apa yang terjadi saat proses request server-to-server dari Merchant ke Veritrans Payment API.

### 3.8.1 POSTMAN

Saya menyarankan untuk menggunakan plugin Google Chrome yang bernama POSTMAN sebagai Rest Client yang bisa kita gunakan untuk mencoba melakukan transaksi ke Veritrans Payment API. Kita bisa download plugin-nya secara gratis [disini](https://chrome.google.com/webstore/detail/postman-rest-client/fdmmgilgnpjigdojojpjoooidkmcomcm?utm_source=chrome-ntp-icon).

![POSTMAN](../images/image-006.png)

### 3.8.2 Melakukan Transaksi Menggunakan POSTMAN

Sekarang kita akan coba melakukan sebuah transaksi dengan metode pembayaran ```mandiri_clickpay``` menggunakan POSTMAN. Sebelumnya pastikan sudah melakukan registrasi terlebih dahulu di MAP Sandbox Veritrans Indonesia untuk mendapatkan Server Key nya.

Masukkan Server Key ke bagian Basic Auth di POSTMAN, lalu klik tombol [Refresh Headers] untuk mengubah Server Key menjadi Base 64. Dan tambahkan Content-Type dan Accept application/json di request nya; seperti terlihat digambar berikut.

![Contoh Charge Menggunakan POSTMAN](../images/image-007.png)

Untuk melakukan transaksi pembayaran (Charge), HTTP Method yang digunakan adalah POST dan endpoint (url) nya adalah https://api.sandbox.veritrans.co.id/v2/charge. Silahkan gunakan body request seperti JSON berikut :

```json
{
      "payment_type": "mandiri_clickpay",
      "transaction_details": {
        "order_id": "1388998298219",
        "gross_amount": 2500
      },
      "item_details": [
        {
          "id": "1388998298204",
          "price": 250,
          "quantity": 10,
          "name": "Mie Ayam Komplit"
        }
      ],
      "customer_details": {
        "email": "eko.khannedy@veritrans.co.id",
        "first_name": "Eko Kurniawan",
        "last_name": "Khannedy",
        "phone": "+6281 1234 1234"
      },
      "mandiri_clickpay": {
        "card_number": "4111111111111111",
        "input1": "1111111111",
        "input2": "2500",
        "input3": "00000",
        "token": "000000"
      }
    }
```

Coba lakukan transaksi dengan mengklik tombol Send, maka data transaksi tersebut akan dikirim ke Veritrans Payment API. Veritrans Payment API akan membalas dengan hasil response misal sebagai berikut :

```json
{
    "status_code": "200",
    "status_message": "OK, Mandiri Clickpay transaction is successful",
    "transaction_id": "5278f252-f1ed-41a5-950a-1b188c9cadde",
    "order_id": "1388998298219",
    "payment_type": "mandiri_clickpay",
    "transaction_time": "2014-06-04 18:59:18",
    "transaction_status": "settlement",
    "gross_amount": "2500.00"
}
```

Data transaksi yang telah kita kirim, otomatis akan masuk ke website MAP Veritrans, dan kita dapat melihatnya secara realtime.

![Transaksi di MAP](../images/image-008.png)

Jika Anda masih bingung tentang JSON request untuk metode pembayaran Mandiri ClickPay, nanti akan dibahas di BAB 6. Internet Banking.

==> [Transaksi Kartu Kredit](../04-kartu-kredit/README.md)