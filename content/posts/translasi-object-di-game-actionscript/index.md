---
title: Dasar translasi object di game (ActionScript)
date: 2012-02-18T09:03:24Z
draft: false
lang: id
tags: ['game programming', 'actionscript']
cover:
  image: images/move-to-004.png
  hiddenInSingle: true
---

Di sini kita akan membahas seputar pergerakan object di dalam game yang digerakkan dengan cara mengklik titik tujuan kemana object harus bergerak. Cara input seperti ini banyak digunakan untuk game-game RTS dan RPG.

Secara garis besar bisa kita jelaskan sebagai: menggerakkan object ke arah yang dituju. Cukup simpel. Tapi kalo menggampangkan, hasilnya tidak seperti yang kita harapkan; misal, ketika sampai di tujuan, object-nya bergerak bolak-balik atau posisi akhirnya tidak tepat.

---

Semisal object yang akan kita gerakkan berada di posisi A. Kita ingin menggerakkan object ini ke titik B.

![image](images/move-to-001.png#center)

Pertama kali, simpan titik tujuan ini, koordinat titik B, sebagai atribute dari sang object.

Langkah selanjutnya kita lakukan adalah mengurangkan koordinat titik tujuan dengan koordinat titik awal. Di sini kita akan mendapatkan _displacement vector_ dari titik awal ke titik tujuan.

![image](images/move-to-002.png#center)

```actionscript
    // Displacement vector didapat dengan mengurangkan
    // koordinat tujuan dengan koordinat posisi sekarang
    var displacementVector = destinationPoint - this.position;
```

Selanjutnya kita hitung jaraknya (_distance_) dengan menghitung magnitude dari vector tadi. Kita simpan jarak ini sebagai attribute dari object; akan kita gunakan nanti.

![image](images/move-to-003.png#center)

```actionscript
    // Jarak antara dua titik adalah
    // magnitude / length dari displacement vector
    // yang dihitung dari dua titik tersebut
    this.moveDistance = displacementVector.length;
```

Nilai lain yang perlu kita hitung adalah arah (_direction_) dari titik awal ke titik tujuan. Direction didapat dengan menormalisasi displacement vector; biasanya dilakukan cukup dengan memanggil method normalize. Atau cara primitif, membagi displacement vector dengan magnitude-nya. Simpan nilai yang kita dapat ke object.

![image](images/move-to-004.png#center)

```actionscript
    // Untuk keperluan update tiap frame,
    // kita simpan displacement vector yang sudah di-normalize,
    // atau bisa disebut dengan unit vector karena
    // vector ini magnitude-nya satu unit.
    this.moveDirection = displacementVector;
    this.moveDirection.normalize(1.0);
```

Apa yang kita dapat sampai saat ini adalah tiga nilai: koordinat titik tujuan, jarak ke titik tujuan dan arah ke tujuan.

```actionscript
class Object {
  var speed:Number;
  var position:Point;
  var moveDestination:Point;
  var moveDistance:Number;
  var moveDirection:Point;
  var isMoving:Boolean = false;

  public function goToTarget(target:Point):void {
    // Simpan tujuan
    this.moveDestination = target;
    // Hitung displament vector berdasar posisi sekarang ke tujuan
    var displacementVector = target - this.position;
    // Hitung jarak ke tujuan dengan menghitung magnitude dari vector di atas
    this.moveDistance = displacementVector.length;
    // Hitung direction vector dengan cara menormalisasi displacement vector.
    // Direction ini akan kita gunakan untuk menghitung posisi di tiap langkah (tick)
    this.moveDirection = displacementVector;
    this.moveDirection.normalize(1.0);
    // Penanda bahwa object ini sedang bergerak
    this.isMoving = true;
  }
};
```

---

Untuk menggerakkan object ini, yang kita lakukan adalah sebagai berikut.

Untuk tiap frame atau time step, yang kita lakukan adalah menghitung posisi untuk frame tersebut dengan mengalikan vector ke target dengan nilai sesuai kecepatan object bergerak dan durasi frame.

```actionscript
    var stepLength = this.speed * deltaTime;
    this.position = this.moveDirection * stepLength;
```

Kemudian kita kurangi jarak ke tujuan dengan besar langkah:

```actionscript
    this.moveDistance -= stepLength;
```

Selanjutnya, kita cek apakah object telah melewati titik tujuan atau belum. Kita lakukan pengecekan ini dengan melihat tanda dari nilai jarak ke tujuan, jika (masih) positif, berarti object kita belum sampai ke tujuan. Jika nol, object tepat berada di titik tujuan. Jika negatif, berarti object telah melewatinya.

Jika jarak yang harus object tempuh negatif, maka object telah melewati titik tujuannya. Untuk menjaga konsistensi, object tidak boleh melewati titik tujuan ini. Caranya adalah dengan mengubah posisi object dengan posisi tujuan yang tadi telah kita simpan.

```actionscript
    if (this.moveDistance <= 0.0) {
      this.position = this.moveDestination;
    }
```

Method lengkapnya:

```actionscript
public function moveTick(deltaTime:Number):void {
  if (this.isMoving) {
    // Hitung panjang langkah sesuai perubahan waktu dan kecepatan
    var stepLength = deltaTime * this.speed;
    // Ubah posisi sesuai langkah dan vector
    this.position = this.moveDirection * stepLength;
    // Kurangi jarak ke tujuan dengan panjang langkah
    this.moveDistance -= stepLength;
    if (this.moveDistance <= 0.0) {
      // Apabila object sampai atau telah melewati tujuan,
      // kita pastikan bahwa posisi sudah pasti sesuai.
      this.position = this.moveDestination;
      this.isMoving = false;
    }
  }
}
```

---

Di atas adalah dasar untuk menggerakkan object. Dasar ini bisa diterapkan ke dimensi lain (3D) dengan menggunakan tipe data yang sesuai.

Perlu dicatat bahwa yang disampaikan di atas ini menggunakan pergerakan linear, tanpa akselerasi dan deselerasi, sehingga akan terlihat tidak natural.

Apabila object bergerak mengikuti waypoints, akan perlu pertimbangan supaya pergerakan kontinyu saat object melewati sebuah waypoint.

---

_Artikel ini sebelumnya saya publish sebagai gist https://gist.github.com/exavolt/1857767_
