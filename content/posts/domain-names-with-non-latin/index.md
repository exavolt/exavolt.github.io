---
title: Nama domain dengan karakter non-Latin
date: 2022-06-06T14:02:22Z
draft: false
lang: id
slug: nama-domain-karakter-non-latin
cover:
  image: images/drake.jpg
---

Seperti biasa, di sebuah grup di Telegram ada diskusi. Ada yang sedang mengerjakan proyek portal untuk beberapa negara Asia dan *stuck*.

Kendala yang dihadapi adalah bagaimana untuk melokalisasi URL, yaitu menampilkan URL yang nama sub-domainnya menggunakan karakter non-Latin menyesuaikan nama tenant. Asumsi awalnya adalah bahwa semua ini bisa teratasi dengan UNICODE terutama UTF-8. Tetapi ternyata domain / hostname itu tidak mendukung UNICODE, karena hanya mendukung [karakter-karakter Latin / ASCII](https://en.wikipedia.org/wiki/Domain_name#Domain_name_syntax).

Saya teringat kita saya sering berurusan dengan *handling* alamat email (untuk login dsb), bahwa ada prosedur khusus untuk mencocokkan alamat email yang dimasukkan pengguna dengan apa yang ada di database. Salah satu bagian dari prosedur adalah dengan mengubah encoding apabila di bagian domain ada bagian yang menggunakan karakter non-Latin. Dan encoding ini tidak menggunakan UTF-8 atau standar UNICODE lain, tetapi menggunakan encoding tersendiri yang disebut [Punycode](https://en.wikipedia.org/wiki/Punycode). Pengaplikasian ke nama domain mengacu ke standar [Internationalized Domain Name (IDN)](https://en.wikipedia.org/wiki/Internationalized_domain_name).

![image](images/drake.jpg#center)

Jadi saya usulkan ke yang bersangkutan buat mencoba library / package dengan nama IDN atau IDNA yang sudah tersedia untuk berbagai bahasa pemograman, termasuk dalam kasus ini, Python.

Tidak lama kemudian, sudah ada laporan seperti ini:

```python
>>> import idna
>>> idna.encode('ドメイン.テスト')
b'xn--eckwd4c7c.xn--zckzah'
>>> print(idna.decode('xn--eckwd4c7c.xn--zckzah'))
ドメイン.テスト
```

Dari situ tinggal pengaplikasian ke bisnis model.

---

Kesimpulannya, gunakan library / package IDN / IDNA, dan bukan UNICODE, untuk mengolah nama domain yang menggunakan karakter non-Latin. Library IDN / IDNA sudah menyediakan fungsi untuk konversi dari UNICODE ke Punycode dan sebaliknya.

---

Catatan tambahan: perlu diingat bahwa tidak semua aplikasi mendukung IDN sehingga walaupun di browser terlihat seperti berikut

![Untitled](images/kr.jpg#center)

ketika URL tersebut disalin akan jadi `http://xn--v52b25r8ohcyb.kr/book` dan bukan `http://탈모케어.kr/book` . Meskipun begitu, ketika disalin-tempel ke browser lagi, biasanya browser akan menampilkan dengan karakter-karakter yang seharusnya.
