+++
title = 'Translasi Object di game (ActionScript)'
date = 2012-02-18T09:03:24Z
draft = false
+++

Di sini kita akan membahas seputar pergerakan object di dalam game yang digerakkan dengan cara mengklik titik tujuan kemana object harus bergerak. Cara input seperti ini banyak digunakan untuk game-game RTS dan RPG.

Secara garis besar bisa kita jelaskan sebagai: menggerakkan object ke arah yang dituju. Cukup simpel. Tapi kalo menggampangkan, hasilnya tidak seperti yang kita harapkan; misal, ketika sampai di tujuan, object-nya bergerak bolak-balik.

Semisal object yang akan kita gerakkan berada di posisi A. Kita ingin menggerakkan object ini ke titik B.

Pertama kali, simpan titik tujuan ini, koordinat titik B, sebagai atribute dari sang object.

Langkah selanjutnya kita lakukan adalah mengurangkan koordinat titik tujuan dengan koordinat titik awal. Di sini kita akan mendapatkan (non-normalized) vector dari titik awal ke titik tujuan.

Dari vector ini, kita hitung magnitude-nya, atau dengan kata lain, jarak lurus antara titik awal ke titik tujuan. Kita simpan jarak ini sebagai attribute dari object; akan kita gunakan nanti.

Nilai lain yang perlu kita hitung adalah normalized vector dari titik awal ke titik tujuan. Dan biasanya dilakukan cukup dengan memanggil method normalize. Atau cara primitif, membagi non-normalized vector dengan jarak / magnitude-nya. Simpan nilai yang kita dapat ke object.

Apa yang kita dapat sampai saat ini adalah tiga nilai: koordinat titik tujuan, jarak ke titik tujan dan normalized vector ke titik tujuan.

```actionscript
class Object {
  var targetPosition:Point;
  var targetDistance:Number;
  var targetVector:Point;
  var position:Point;
  var movingToTarget:Boolean = false;
  public function goToTarget(target:Point):void {
    this.targetPosition = target;
    var delta = target - this.position;
    this.targetDistance = delta.length;
    this.targetVector = delta;
    this.targetVector.normalize(1.0);
    this.movingToTarget = true;
  }
};
```

Untuk menggerakkan object ini, yang kita lakukan adalah sebagai berikut.

Untuk tiap frame atau time step, yang kita lakukan adalah menghitung posisi untuk frame tersebut dengan mengalikan vector ke target dengan nilai sesuai kecepatan object bergerak.

```actionscript
    this.position = this.targetVector * stepLength;
```

Kemudian kita kurangi jarak ke tujuan dengan besar langkah:

```actionscript
    this.targetDistance -= stepLength;
```

Selanjutnya, kita cek apakah object telah melewati titik tujuan atau belum. Kita lakukan pengecekan ini dengan melihat tanda dari nilai jarak ke tujuan, jika (masih) positif, berarti object kita belum sampai ke tujuan. Jika nol, object tepat berada di titik tujuan. Jika negatif, berarti object telah melewatinya.

Jika jarak yang harus object tempuh negatif, maka object telah melewati titik tujuannya. Untuk menjaga konsistensi, sebaiknya object tidak melewati titik tujuan ini. Caranya adalah dengan mengubah posisi object dengan posisi tujuan yang tadi telah kita simpan.

```actionscript
    if (this.targetDistance <= 0.0) {
      this.position = this.targetPosition;
    }
```

Method lengkapnya:

```actionscript
public function moveTick(stepLength:Number):void {
  if (this.movingToTarget) {
    this.position = this.targetVector * stepLength;
    this.targetDistance -= stepLength;
    if (this.targetDistance <= 0.0) {
      this.position = this.targetPosition;
      this.movingToTarget = false;
    }
  }
}
```
