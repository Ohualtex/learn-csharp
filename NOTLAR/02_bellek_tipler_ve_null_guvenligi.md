# Bölüm 2: C# Bellek Modeli, Değer/Referans Tipleri ve Null Güvenliği
*(Stack-Heap ayrımı, ref/out/in niteleyicileri ve modern C# null yönetimi)*

## 2.1) Bellek Modeli: Stack vs Heap & Değer vs Referans Tipleri

.NET çalışma zamanında (CLR) bellek temelde iki ana bölgeye ayrılır: **Stack** ve **Heap**.

| Özellik | Stack (Yığın) | Heap (Öbek) |
| :--- | :--- | :--- |
| **Hız** | Son derece hızlıdır (LIFO prensibi) | Stack'e göre daha yavaştır |
| **Yönetim** | Fonksiyon bitince otomatik temizlenir | **Garbage Collector (GC)** tarafından temizlenir |
| **Hangi Tipler?** | **Değer Tipleri (Value Types)**: `int`, `double`, `bool`, `char`, `struct`, `enum` | **Referans Tipleri (Reference Types)**: `class`, `string`, `object`, `array`, `delegate` |

### Değer Tipi (struct) vs Referans Tipi (class) Davranışı

```csharp
// 1. Değer Tipi (Value Type) - Bağımsız kopya oluşturur
int a = 10;
int b = a; // a'nın o anki DEĞERİ b'ye kopyalanır
b = 99;
Console.WriteLine(a); // 10 (a DEĞİŞMEZ!)

// 2. Referans Tipi (Reference Type) - Adres kopyalar
class Oyuncu
{
    public string Isim = "Oyuncu 1";
}

Oyuncu o1 = new Oyuncu();
Oyuncu o2 = o1; // Nesne kopyalanmaz! Heap'teki AYNI adresi gösteren yeni bir kumanda oluşur.
o2.Isim = "Değişti";

Console.WriteLine(o1.Isim); // Değişti (o1 de etkilendi!)
```

**Tuzaklar ve Nüanslar:**
- `string` bir referans tipidir (Heap'te yaşar), ancak **immutable (değiştirilemez)** olarak tasarlandığı için bir metinde değişiklik yaptığınızda orijinal metin değişmez; Heap'te yepyeni bir string nesnesi tahsis edilir.
- Bir `struct` kopyalandığında tüm alanları tek tek yeni bir alana kopyalanır. Büyük struct'ların sık sık kopyalanması performansı olumsuz etkileyebilir.

> 🔑 **Ezber:** "Değer tipinde fotokopiyi verirsin, orijinal değişmez. Referans tipinde kumandayı verirsin, televizyon (orijinal) değişir!"

---

## 2.2) Parametre Niteleyicileri: `ref`, `out` ve `in`

C#'ta metotlara argümanlar varsayılan olarak **değer kopyasıyla (by value)** aktarılır. Bu davranışı değiştirmek ve referans geçirmek için 3 ana anahtar kelime kullanılır:

```csharp
class ParametreOrnekleri
{
    // 1. ref: Değişken hem girmeden önce başlatılmış olmalı hem de içeride değişebilir
    public static void BakiyeArtir(ref int bakiye)
    {
        bakiye += 100; // Orijinal değişkeni doğrudan değiştirir
    }

    // 2. out: Metoda girmeden önce değer atanması zorunlu DEĞİLDİR;
    // ancak metot bitmeden önce İÇERİDE mutlaka bir değer atanmak ZORUNDADIR!
    public static bool BolmeYap(int bolunen, int bolen, out int sonuc)
    {
        if (bolen == 0)
        {
            sonuc = 0; // Hata durumunda bile out değişkenine atama şarttır!
            return false;
        }
        sonuc = bolunen / bolen;
        return true;
    }

    // 3. in: Referans ile aktarır ancak SALT OKUNURDUR (readonly).
    // Performans için büyük struct kopyasını engeller, içeride değiştirilmesine izin vermez.
    public static void KoordinatYazdir(in int buyukVeri)
    {
        // buyukVeri = 50; // DERLEME HATASI! 'in' parametresi değiştirilemez.
        Console.WriteLine(buyukVeri);
    }
}
```

**Kullanım Örneği:**
```csharp
int paramRef = 50;
ParametreOrnekleri.BakiyeArtir(ref paramRef); // paramRef artık 150

// out parametresi satır içinde anlık tanımlanabilir (out-var)
if (ParametreOrnekleri.BolmeYap(20, 4, out int bolum))
{
    Console.WriteLine($"Bölüm: {bolum}"); // 5
}
```

**Tuzaklar:**
- `ref` kullanırken çağrılan tarafta değişkene ilk değer atanmamışsa derleyici hata verir (`Use of unassigned local variable`).
- `out` kullanan bir metotta, metot tamamlanmadan önce `out` parametresine değer atamayı unutursanız derleme hatası alırsınız.

> 🔑 **Ezber:** "`ref` dolu gelir değişebilir; `out` boş gelir, dolu çıkmak zorundadır; `in` bakar ama elleyemez (readonly)!"

---

## 2.3) Modern C# Null Güvenliği ve Operatörleri

C# 8.0 ile birlikte gelen **Nullable Reference Types** özelliği, yazılımcıların en büyük kabusu olan `NullReferenceException` (Object reference not set to an instance of an object) hatalarını derleme aşamasında önlemeyi hedefler.

```csharp
// 1. Null Olabilir (?) vs Olamaz Referanslar
string kesinMetin = "Merhaba"; // null atanırsa derleyici uyarı verir (CS8600)
string? olabilirMetin = null;  // '?' işareti bu değişkenin null olabileceğini belirtir

// 2. Null-Conditional Operatörü (?.) - Güvenli Gezinme
string? isim = null;
int? uzunluk = isim?.Length; // isim null ise patlamaz; geriye null döner

// 3. Null-Coalescing Operatörü (??) - Null İse Varsayılan Ver
string kullaniciAdi = isim ?? "Misafir"; // isim null olduğu için "Misafir" atanır

// 4. Null-Coalescing Assignment Operatörü (??=) - Null İse Yeni Değer Ata
string? cache = null;
cache ??= "Varsayılan Önbellek"; // cache null olduğu için değer atandı
cache ??= "İkinci Değer";        // cache artık dolu olduğu için bu atama ÇALIŞMAZ!
Console.WriteLine(cache);        // Varsayılan Önbellek

// 5. Null-Forgiving (Bang) Operatörü (!) - Derleyiciyi Susturma
// "Ben geliştirici olarak buranın null gelmeyeceğine garanti veriyorum" demektir.
string? potansiyelNull = "Kesin Veri";
string kesinSonuc = potansiyelNull!; // Derleyici uyarısını susturur
```

**Tuzaklar ve Nüanslar:**
- `!` (bang) operatörü çalışma zamanındaki gerçek `null` değerini yok etmez. Sadece derleyici uyarısını susturur. Eğer o değer çalışma anında `null` gelirse programınız yine patlar (`NullReferenceException`).
- `?.` operatöründen dönen değer tipleri otomatik olarak nullable tipe dönüşür (`int` yerine `int?`).

> 🔑 **Ezber:** "`?.` sormadan kırmaz, `??` boşsa yedeği koyar, `??=` yoksa doldurur, `!` derleyiciyi susturur ama runtime'ı kandıramaz!"

