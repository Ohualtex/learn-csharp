# Bölüm 4: Diziler ve Generic Koleksiyonlar
*(System.Array, matrisler, jagged diziler, params ve modern generic koleksiyonlar)*

## 4.1) Dizi (Array) Temelleri ve Bellek Yerleşimi

C#'ta dizi; aynı türden verilerin bellekte **ardışık (contiguous)** olarak saklandığı veri yapısıdır.

* **Diziler Birer Referans Tipidir:** C ve C++ dillerinde dizilerin boyutu derleme anında bilinmek zorundadır. C#'ta ise tüm diziler arka planda **`System.Array`** sınıfından türetilmiş birer nesnedir ve boyutları çalışma anında (Runtime) belirlenebilir.
* **Heap'te Tahsis Edilir:** Dizi değişkeni Stack'te bir referans (adres) tutarken, dizinin elemanları Heap bölgesinde tahsis edilir.
* **Varsayılan Değerler:** `new` anahtar sözcüğü ile dizi oluşturulduğunda tüm elemanlar otomatik olarak varsayılan değerlerini alır:
  * Sayısal tipler (`int`, `double` vb.): `0`
  * Mantıksal (`bool`): `false`
  * Referans tipleri (`string`, nesneler): `null`

```csharp
// Tanımlama ve Başlatma Çeşitleri
int[] sayilar = new int[5]; // 5 elemanlı [0, 0, 0, 0, 0]
string[] diller = { "C#", "F#", "Python" }; // Boyut belirtmeden ilk değer atama
var dinamikDizi = new[] { 10, 20, 30 }; // Tür çıkarımı (int[])

// Diziler referans ile kopyalanır (Adres kopyası)
int[] a = { 1, 2, 3 };
int[] b = a; // Nesne kopyalanmaz, kumanda kopyalanır!
b[0] = 999;
Console.WriteLine(a[0]); // 999 (a da etkilendi!)
```

> 🔑 **Ezber:** "Dizi bir referans tipidir (`new` ile Heap'te doğar). Boyutu bir kez belirlendikten sonra sabittir, `b = a` dersen yeni dizi oluşmaz, adres paylaşılır!"

---

## 4.2) System.Array Sınıfı ve Statik Metotları

C#'taki tüm diziler `System.Array` sınıfının üyesidir. Hem özellikler (properties) hem de statik yardımcı metotlar sunar:

### Temel Özellikler
* **`Length`:** Dizideki toplam eleman sayısını verir (`dizi.Length`).
* **`Rank`:** Dizinin boyut sayısını verir (`dizi.Rank` -> 1 boyutlu için 1, matris için 2).

### En Sık Kullanılan Statik Metotlar
```csharp
int[] sayilar = { 45, 12, 85, 32, 9 };

// 1. Array.Sort: Küçükten büyüğe sıralar
Array.Sort(sayilar); // [9, 12, 32, 45, 85]

// 2. Array.Reverse: Diziyi tersine çevirir
Array.Reverse(sayilar); // [85, 45, 32, 12, 9]

// 3. Array.IndexOf: Elemanın ilk görüldüğü indeksi döner (Yoksa -1 döner)
int indeks = Array.IndexOf(sayilar, 32); // 2

// 4. Array.BinarySearch: SIRALI dizide ikili arama yapar (Bulamazsa negatif sayı döner!)
Array.Sort(sayilar);
int bulunan = Array.BinarySearch(sayilar, 45); // Dizi mutlaka sıralı olmalıdır!

// 5. Array.Clear: Belirtilen aralıktaki elemanları varsayılan değerine (0/false/null) çeker
// Array.Clear(dizi, baslangicIndeksi, elemanSayisi);
Array.Clear(sayilar, 1, 2); // 1. indisten itibaren 2 elemanı sıfırlar

// 6. Array.Copy ve CopyTo
int[] hedef = new int[10];
// Array.Copy(kaynak, kaynakIndeks, hedef, hedefIndeks, adet);
Array.Copy(sayilar, 0, hedef, 3, sayilar.Length); // sayilar'ı hedef'in 3. indisinden itibaren yapıştırır

// 7. Array.Resize: Dizinin boyutunu eski elemanları koruyarak değiştirir
// Arka planda yeni bir dizi oluşturur ve elemanları kopyalar ('ref' zorunludur!)
Array.Resize(ref sayilar, 8); // Artık 8 elemanlı
```

> 🔑 **Ezber:** "`Array.Clear` silmez, sıfırlar (0/null yapar). `BinarySearch` sıralı dizi ister, bulamazsa negatif döner. `Resize` arka planda yeni dizi açar, bu yüzden `ref` ile çalışır."

---

## 4.3) Çok Boyutlu (Matris `[,]`) vs Düzensiz (Jagged `[][]`) Diziler

C#'ta iki farklı çok boyutlu dizi yaklaşımı vardır ve birbirine karıştırılmamalıdır:

### 1. Düzenli Çok Boyutlu Diziler (Matrisler - `[,]`)
Tüm satırların sütun sayısı **birbirine eşittir**. Bellekte tek parça halinde dikdörtgen bir blok olarak saklanır. Boyut sayısı virgüllerle (`[,]`, `[,,]`) belirtilir.

```csharp
// 3 satır, 2 sütunlu matris
int[,] matris = new int[3, 2]
{
    { 10, 20 },
    { 30, 40 },
    { 50, 60 }
};

Console.WriteLine(matris[1, 0]); // 30 (1. satır, 0. sütun)
Console.WriteLine(matris.Rank);  // 2 (Boyut sayısı)

// Sınav Tuzağı: GetLength vs GetUpperBound
Console.WriteLine(matris.GetLength(0));      // 3 (0. boyutun eleman sayısı / satır adedi)
Console.WriteLine(matris.GetUpperBound(0));  // 2 (0. boyutun EN BÜYÜK İNDEKSİ)
```

### 2. Düzensiz Diziler (Jagged Arrays - `[][]`)
**"Dizilerin dizisidir."** Her satırın sütun sayısı birbirinden **farklı** olabilir. Her satır Heap'te bağımsız ayrı bir dizi nesnesidir.

```csharp
// 3 satırlı düzensiz dizi (sütun boyutları baştan belirlenmez)
int[][] jagged = new int[3][];
jagged[0] = new int[2] { 1, 2 };          // 0. satır 2 elemanlı
jagged[1] = new int[4] { 3, 4, 5, 6 };    // 1. satır 4 elemanlı
jagged[2] = new int[1] { 7 };             // 2. satır 1 elemanlı

Console.WriteLine(jagged[1][2]); // 5 (1. satırın 2. elemanı)
```

| Özellik | Matris (`int[,]`) | Düzensiz Dizi (`int[][]`) |
| :--- | :--- | :--- |
| **Sözdizimi** | `dizi[satir, sutun]` | `dizi[satir][sutun]` |
| **Satır Boyutları** | Eşit olmak zorundadır (Dikdörtgen) | Her satır farklı uzunlukta olabilir |
| **Bellek** | Tek parça contiguous bellek bloğu | Her satır için ayrı heap tahsisi |

> 🔑 **Ezber:** "`[,]` dikdörtgendir, virgülle yazılır. `[][]` merdivendir (düzensizdir), her satırın boyutu farklı olabilir."

---

## 4.4) Değişken Sayıda Parametre Alan Metotlar: `params`

Bir metoda kaç adet parametre gönderileceğini önceden kestiremediğimiz durumlarda `params` anahtar kelimesi imdadımıza yetişir. Gelen argümanları metot içerisinde bir **dizi** olarak karşılar.

```csharp
class Matematik
{
    // params sayesinde ister 0, ister 1, ister 100 sayı gönderebiliriz
    public static int Topla(params int[] sayilar)
    {
        int toplam = 0;
        foreach (int sayi in sayilar)
        {
            toplam += sayi;
        }
        return toplam;
    }

    // Overload (Aşırı Yükleme) Tuzağı:
    public static void Islem(int a, int b) => Console.WriteLine("Tam eşleşen (int, int) çalıştı.");
    public static void Islem(params int[] dizi) => Console.WriteLine("params çalıştı.");
}

// Kullanım:
int t1 = Matematik.Topla();              // 0 döner (boş dizi gider)
int t2 = Matematik.Topla(5, 10, 15);     // 30 döner
int t3 = Matematik.Topla(new int[] { 1, 2, 3 }); // Dizi olarak da gönderilebilir

Matematik.Islem(3, 5); // "Tam eşleşen (int, int) çalıştı." (params'tan DAİMA önceliklidir!)
```

**Tuzaklar:**
1. `params` parametresi metot imzasında **yalnızca bir tane** olabilir.
2. `params` parametresi her zaman parametre listesinin **en sonunda** yer almak zorundadır (`void Test(int id, params string[] roller)` geçerlidir; `void Test(params string[] roller, int id)` derleme hatasıdır).

> 🔑 **Ezber:** "`params` açgözlüdür, sona yazılır; ama tam eşleşen normal metot varsa derleyici `params`'a bakmaz, tam eşleşeni çağırır!"

---

## 4.5) Modern C# Index (`^`) ve Range (`..`) Operatörleri (C# 8+)

Modern C#, diziler ve koleksiyonlar üzerinde gezinmeyi ve dilimlemeyi (slicing) son derece pratik hale getiren iki operatör sunar:

```csharp
string[] meyveler = { "Elma", "Armut", "Muz", "Çilek", "Kivi" };

// 1. Hat Operatörü (^) - Sondan İndeksleme
// ^1: Son eleman, ^2: Sondan bir önceki eleman
string sonuncu = meyveler[^1]; // "Kivi"
string sondanIkinci = meyveler[^2]; // "Çilek"

// 2. Aralık Operatörü (..) - Dilimleme (Slicing)
// [baslangic..bitis] -> Baslangic DAHİL, Bitis HARİÇTİR!
string[] dilim1 = meyveler[1..4]; // ["Armut", "Muz", "Çilek"]
string[] ilkUcu = meyveler[..3];   // ["Elma", "Armut", "Muz"] (0'dan başlar)
string[] sonUcu = meyveler[^3..];  // ["Muz", "Çilek", "Kivi"] (Sondan 3'ten sona kadar)
```

> 🔑 **Ezber:** "`^1` sonuncu elemandır (0 değil 1'den başlar!). `[1..4]` derken 1 dahil, 4 hariçtir!"

---

## 4.6) Koleksiyonların Evrimi: `ArrayList` Neden Terk Edildi?

C# 1.0 döneminde dinamik boyutlu listeler için `ArrayList` (`System.Collections`) kullanılırdı. Günümüzde `ArrayList` kullanımı neredeyse tamamen terk edilmiştir.

### Boxing / Unboxing (Kutulama) Tuzağı
* `ArrayList` içerisindeki tüm verileri **`object`** türünde saklar.
* Değer tipi olan bir değişken (`int`, `double`) `ArrayList`'e eklendiğinde Stack'ten alınıp Heap'te bir kutu içerisine konur (**Boxing**).
* Değere geri erişilmek istendiğinde kutudan çıkarılıp cast edilmelidir (**Unboxing**).
* Bu durum binlerce veri eklendiğinde devasa bir CPU ve bellek (GC) yükü yaratır; ayrıca tür güvenliği yoktur (aynı listeye hem sayı hem metin eklenirse derleyici uyarmaz, çalışma anında çöker).

```csharp
// ESKİ VE TEHLİKELİ (ArrayList):
ArrayList eskiListe = new ArrayList();
eskiListe.Add(10); // Boxing (int -> object)
eskiListe.Add("Hata"); // Tür güvenliği yok!
// int sayi = (int)eskiListe[1]; // ÇALIŞMA ANINDA ÇÖKER! (InvalidCastException)

// MODERN VE GÜVENLİ (Generic List<T>):
List<int> modernListe = new List<int>();
modernListe.Add(10); // Boxing YOK! Doğrudan int olarak saklanır.
// modernListe.Add("Hata"); // DERLEME ANINDA HATA VERİR! Hataları canlıya çıkmadan yakalar.
```

---

## 4.7) Modern Generic Koleksiyonlar (`System.Collections.Generic`)

Modern .NET geliştirmede dizilerin yerini çoğunlukla generic koleksiyonlar alır:

```csharp
// 1. List<T>: Dinamik boyutlu, tip güvenli dizi alternatifi
List<string> sehirler = new List<string> { "Ankara", "İstanbul" };
sehirler.Add("İzmir");
sehirler.Remove("İstanbul");
Console.WriteLine(sehirler.Count); // 2 (.Length değil .Count!)

// 2. Dictionary<TKey, TValue>: Anahtar-Değer çifti (Key-Value)
// Anahtarlar BENZERSİZ olmak zorundadır. Arama hızı O(1)'dir (şipşak bulur).
Dictionary<string, int> plakalar = new Dictionary<string, int>
{
    ["Ankara"] = 6,
    ["İstanbul"] = 34
};
plakalar["İzmir"] = 35; // Ekleme / Güncelleme

if (plakalar.TryGetValue("Bursa", out int plakaNo))
{
    Console.WriteLine($"Plaka: {plakaNo}");
}
else
{
    Console.WriteLine("Bursa bulunamadı."); // Güvenli okuma (KeyNotFoundException fırlatmaz)
}

// 3. HashSet<T>: Tekil (Benzersiz) Elemanlar Kümesi
// Matematiksel küme mantığıdır. Mükerrer (çift) kayıt barındırmaz.
HashSet<int> tekilSayilar = new HashSet<int>();
tekilSayilar.Add(5);
tekilSayilar.Add(5); // İkinci 5 eklenmez, sessizce yok sayılır!
Console.WriteLine(tekilSayilar.Count); // 1
```

---

## 4.8) Sınav ve Mülakat Tuzakları

1. **"Bir dizinin boyutunu `Array.Resize` ile genişletirsek bellekte ne olur?"**  
   * **Tuzak:** Mevcut dizinin sonuna yeni alan eklendiğini sanmak.  
   * **Gerçek:** Diziler sabit boyutludur. `Array.Resize`, arka planda istenen boyutta **yepyeni bir dizi nesnesi tahsis eder**, eski elemanları oraya kopyalar ve referansı yeni diziye yönlendirir.
2. **"Matris `GetLength(0)` ile `GetUpperBound(0)` farkı nedir?"**  
   * **Tuzak:** İkisini de aynı sanmak. `GetLength(0)` eleman sayısını verir (örn: 3). `GetUpperBound(0)` ise o boyutun son indeksini verir (0, 1, 2 olduğu için `2` döner).
3. **"Dizilerde `.Length`, `List<T>`'de `.Count` kullanılır."**  
   * Dizi boyutunu sorgularken `.Count` yazmak veya `List<T>` için `.Length` yazmak derleme hatasıdır.
4. **"Dictionary'de olmayan bir anahtarı köşeli parantezle çağırmak:**  
   * `plakalar["Trabzon"]` derseniz ve anahtar yoksa `KeyNotFoundException` ile program çöker. Güvenli erişim için her zaman `TryGetValue` veya `ContainsKey` kontrolü yapılmalıdır.

---

## 🔑 Hızlı Tekrar: Altın Ezber Cümleleri (Kopya Kağıdı)

- **Dizi & Boyut:** "Diziler referans tipidir; `.Length` eleman sayısını, `GetUpperBound` son geçerli indeksi verir."
- **Matris vs Jagged:** "`[,]` dikdörtgendir tek blokta yaşar, `[][]` düzensiz dizilerin dizisidir."
- **params:** "`params` metot imzasında tek olur ve en sona yazılır; tam eşleşen normal metot varsa `params` arka plana atılır."
- **Index & Range:** "`^1` son elemandır, `[1..4]` 1 dahil 4 hariç dilimler."
- **ArrayList vs List:** "`ArrayList` her şeyi `object` yaparak kutular (boxing) ve yorar; modern C#'ta tip güvenli `List<T>` kullanılır."
- **Dictionary:** "Anahtar benzersizdir; doğrudan köşeli parantezle sorma patlarsın, `TryGetValue` ile sor güvenle al!"

