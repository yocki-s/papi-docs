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

## 3.5 Fraud Status

## 3.6 Payment Type

## 3.7 Status Code

## 3.8 Menggunakan RestClient

==> [Transaksi Kartu Kredit](../04-kartu-kredit/README.md)