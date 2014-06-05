<== [Veritrans Payment API](../03-payment-api/README.md)

4. Transaksi Kartu Kredit
=========================================

Metode pembayaran yang pertama yang akan kita bahas adalah ```credit_card``` (kartu kredit). 

## 4.1 Tahapan Transaksi Kartu Kredit

Dalam transaksi kartu kredit, terdapat dua tahapan yang diperlukan untuk melakukan transaksi;

- Token Request, untuk membuat token_id yang akan digunakan saat proses Charge Request
- Charge Request, mengirim data transaksi + token_id ke Veritrans Payment API

### 4.1.1 Token Request

Token Request merupakan tahapan yang dilakukan sebelum Charge Request. Dalam tahapan Token Request, data kartu kredit akan dikirim dari website Merchant (oleh pelanggan) ke sistem Veritrans Payment API secara client-to-server, bukan server-to-server. Jika dalam browser (aplikasi perambah), data kartu kredit dikirim menggunakan JavaScript ke server Veritrans Payment API.

Data apa saja yang harus dikirim pada tahapan Token Request? Tergantung transaksi apa yang akan dilakukan; misal kartu kredit normal, 3d secure, one click, two click, dan lain-lain. Setiap jenis transaksi kartu kredit punya spefisikasi masing-masing.

Setelah tahapan Token Request berhasil, maka Merchant akan mendapatkan ```token_id``` dari server Veritrans Payment API. Data ```token_id``` tersebut dapat digunakan sebagai perwakilan data kartu kredit. Namun perlu diperhatikan bahwa ```token_id``` hanya dapat digunakan untuk satu kali transaksi, bahkan jika transaksinya gagal, ```token_id``` akan otomatis dihapus dari sistem Veritrans Payment API. ```token_id``` juga hanya bisa digunakan kurang lebih 5 menit setelah token dibuat, jika Merchant terlalu lama melakukan Charge Request setelah mendapatkan ```token_id``` (lebih dari 5 menit), ada kemungkinan ```token_id``` akan dihapus dari server Veritrans Payment API.

#### 4.1.1.1 Kenapa Data Kartu Kredit Tidak Boleh Dikirim server-to-server?

Salah satu ketentuan PCI DSS adalah, data kartu kredit tidak boleh sampai masuk ke server Merchant, walaupun hanya lewat (tidak disimpan di database), tetap data kartu kredit tidak boleh lewat ke server Merchant. Kecuali server Merchant telah lulus sertifikasi PCI DSS.

Dikhawatirkan jika data kartu kredit lewat ke server Merchant, walaupun Merchant tidak menyimpannya (ke dalam database), bisa saja data kartu kredit akan ter-log di log system server Merchant, hal ini bisa berakibat fatal karena data kartu kredit bisa didapatkan oleh pihak yang tidak bertanggung jawab.

### 4.1.2 Charge Request



==> [Pengenalan Callback & Notification](../05-callback-notification/README.md)

