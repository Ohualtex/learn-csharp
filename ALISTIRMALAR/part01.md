# ✍️ ALIŞTIRMALAR — Part 1

Konular Bölüm 0 (C# 101 Temelleri) odaklı ve tuzak içeriklidir. Her sorunun cevabı **hemen altında gizli** — önce kendin çöz, sonra "Cevap"a tıkla.  
Konu anlatımı için → [`NOTLAR/00_csharp_101_temeller.md`](../NOTLAR/00_csharp_101_temeller.md)

---

**1. Soru** (Tam Sayı Bölmesi ve Tür Dönüşümü)
```csharp
int x = 7;
int y = 2;
double z = x / y;

Console.WriteLine(z);
// Çıktı nedir?
```
<details><summary>Cevap</summary>

`3` (veya `3` olarak yazdırılır, `3.5` DEĞİL).  
`x` ve `y` değişkenlerinin her ikisi de `int` türünde olduğundan bölme işlemi tam sayı bölmesi olarak yapılır ve ondalık kısım atılarak sonuç `3` bulunur. Ardından `double` olan `z` değişkenine `3.0` olarak atanır. Doğru sonucun `3.5` çıkması için en az bir tarafın `(double)x / y` veya `7.0 / 2` şeklinde yazılması gerekir.
</details>

---

**2. Soru** (Verbatim String `@` ve Karşılaştırma)
```csharp
string yol1 = "C:\\Projeler\\Test";
string yol2 = @"C:\Projeler\Test";

Console.WriteLine(yol1 == yol2);
// Çıktı nedir?
```
<details><summary>Cevap</summary>

`True`.  
Normal string içindeki `\\` çift ters slash ifadesi tek bir ters slash (`\`) kaçışını temsil eder. Başına `@` konulan verbatim string'de ise ters slash karakterleri olduğu gibi saklanır. Sonuç olarak iki string bellekte karakter dizilimi bakımından tamamen aynıdır.
</details>

---

**3. Soru** (Mantıksal Operatörler & Kısa Devre / Short-Circuit Davranışı)
```csharp
bool kosul = false;
int sayac = 10;

if (kosul && (++sayac > 10))
{
    Console.WriteLine("İçeride");
}

Console.WriteLine(sayac);
// Çıktı nedir?
```
<details><summary>Cevap</summary>

`10`.  
`&&` (VE) operatörü "kısa devre" (short-circuit) prensibiyle çalışır. Sol taraftaki ifade (`kosul`) `false` olduğu anda, sonucun her halükarda `false` olacağı kesinleştiği için sağ taraftaki `(++sayac > 10)` kodu hiç çalıştırılmaz. Bu yüzden `sayac` artmaz ve `10` kalır.
</details>

---

**4. Soru** (if Bloğunda Noktalı Virgül Tuzağı)
```csharp
int puan = 35;

if (puan >= 50);
{
    Console.WriteLine("Dersi Geçtiniz!");
}
// Çıktı nedir?
```
<details><summary>Cevap</summary>

`Dersi Geçtiniz!`.  
`if (puan >= 50);` satırının sonundaki noktalı virgül (`;`), şart kontrolünün etki alanını orada sonlandırır. Altındaki süslü parantezli kod bloğu artık if şartına bağlı olmayan bağımsız bir blok haline gelir ve şart sağlanmasa bile her halükarda çalışır.
</details>

---

**5. Soru** (`int.TryParse` Başarısızlık Durumu)
```csharp
string giris = "1923Cumhuriyet";

if (int.TryParse(giris, out int yil))
{
    Console.WriteLine($"Yıl: {yil}");
}
else
{
    Console.WriteLine($"Dönüştürülemedi: {yil}");
}
// Çıktı nedir?
```
<details><summary>Cevap</summary>

`Dönüştürülemedi: 0`.  
Metin içinde harf bulunduğu için `TryParse` başarısız olur ve `else` bloğuna düşer. `out` parametresi olarak tanımlanan `yil` değişkeni, dönüşüm başarısız olduğunda ait olduğu tipin varsayılan değerini (`default(int)` yani `0`) alır.
</details>

---

**6. Soru** (Metotlar ve Değer Tipi / Değer Aktarımı)
```csharp
class Program
{
    static void Arttir(int sayi)
    {
        sayi += 50;
    }

    static void Main()
    {
        int bakiye = 100;
        Arttir(bakiye);
        Console.WriteLine(bakiye);
    }
}
// Çıktı nedir?
```
<details><summary>Cevap</summary>

`100`.  
C#'ta `int` bir değer tipidir (value type). Metoda parametre olarak gönderildiğinde değişkenin kendisi değil, o anki değerinin bir **kopyası** metoda gider. Metot içinde yapılan artış yerel kopyayı etkiler; `Main` metodundaki orijinal `bakiye` değişkeni `100` olarak kalır (Orijinalin değişmesi için `ref` veya `out` gereklidir).
</details>

---

**7. Soru** (Mod Alma Operatörü ve Negatif Sayılar)
```csharp
int sonuc1 = 17 % 5;
int sonuc2 = -17 % 5;

Console.WriteLine($"{sonuc1} | {sonuc2}");
// Çıktı nedir?
```
<details><summary>Cevap</summary>

`2 | -2`.  
C#'ta `%` (kalan/mod) operatörünün sonucunun işareti, bölünen sayının (sol tarafın) işaretini taşır. `17 % 5` işlemi `2` verirken, `-17 % 5` işlemi `-2` sonucunu üretir.
</details>

---

**8. Soru** (`Console.Write` ve Kaçış Sıralaması)
```csharp
Console.Write("A");
Console.Write("\nB\t");
Console.WriteLine("C");
Console.Write("D");
// Ekranda kaç satır oluşur ve satır içerikleri ne olur?
```
<details><summary>Cevap</summary>

Ekranda toplam **2 tam satır** (ve 3. satırda imleç bekleyen bir `D`) oluşur:  
1. Satır: `A`  
2. Satır: `B   C` (`\t` bir sekme boşluk bırakır, `WriteLine` ardından alt satıra geçer)  
3. Satır: `D`
</details>
