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

### 3.1.2 Default HTTP Header

Format data yang digunakan oleh Veritrans Payment API adalah [JSON (JavaScript Object Notation)](http://www.json.org/). Sehingga secara default sistem Merchant perlu mengirim Content-Type dan Accept header dengan value application/json pada setiap request komunikasi yang dilakukan server-to-server ke Veritrans Payment API.

```

Content-Type  : application/json
Accept        : application/json

```

## 3.2 Payment Request

## 3.3 Payment Response

## 3.4 Transaction Status

## 3.5 Fraud Status

## 3.6 Payment Type

## 3.7 Status Code

## 3.8 Menggunakan RestClient

==> [Transaksi Kartu Kredit](../04-kartu-kredit/README.md)