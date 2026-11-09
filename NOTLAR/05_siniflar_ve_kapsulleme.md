# Bölüm 5: Sınıflar, Nesneler ve Kapsülleme
*(Class vs Object, Heap bellek modeli, erişim belirleyiciler, backing field, properties ve init)*

Nesne Yönelimli Programlama (OOP), yazılımı gerçek dünya varlıklarına modelleyerek kodun yeniden kullanılabilirliğini, güvenliğini ve bakım kolaylığını sağlayan temel programlama yaklaşımıdır. C# dili baştan sona saf bir nesne yönelimli dil olarak kurgulanmıştır.

---

## 5.1) Sınıf (Class) ve Nesne (Object) Temelleri

* **Sınıf (Class):** Bir nesnenin durumunu (state - alanlarını/verilerini) ve davranışlarını (behavior - metotlarını) tanımlayan soyut bir **şablondur (blueprint)**. Tanımlandığında bellekte nesne verisi için yer tutmaz.
* **Nesne (Object / Instance):** Bir sınıftan `new` anahtar sözcüğü kullanılarak Heap belleğinde somutlaştırılmış (örneklendirilmiş) canlı bir varlıktır.

### Bellek Modeli ve Referans Kopyalama Tuzağı
Sınıflar **Referans Tipidir (Reference Type)**. Değişkenin kendisi Stack bölgesinde yalnızca 8 byte'lık bir adres (kumanda) tutarken, nesnenin asıl verileri Heap bölgesinde saklanır:

```csharp
public class Oyuncu
{
    public string Ad;
    public int Skor;
}

// 1. Nesne üretimi (Heap'te tahsis)
Oyuncu p1 = new Oyuncu(); 
p1.Ad = "Miyamura";
p1.Skor = 100;

// 2. Referans kopyalama tuzağı!
Oyuncu p2 = p1; // DİKKAT: Yeni bir nesne üretilmez, p1'in bellek adresi kopyalanır!
p2.Skor = 500;

// p1'in skoru ne oldu?
Console.WriteLine(p1.Skor); // Çıktı: 500! (İki kumanda tek bir televizyonu yönetir)
```

> 🔑 **Ezber:** "Sınıf bir şablondur; nesne ise o şablondan `new` ile Heap'te doğan canlı yapıdır. Bir nesne değişkenini diğerine atamak nesneyi kopyalamaz, adresi (kumandayı) kopyalar!"

---

## 5.2) Sınıf Alanlarının Varsayılan Değerleri (Default Values)

Bir metot içindeki yerel değişkenler (local variables) ilk değer verilmeden okunamaz ve doğrudan derleme hatası verir (`CS0165: Use of unassigned local variable`).

Buna karşın, bir sınıfın alanları (fields) `new` ile nesne oluşturulduğu anda CLR tarafından otomatik olarak türlerinin varsayılan değerlerine sıfırlanır:
* Sayısal tipler (`byte`, `int`, `long`, `float`, `double`, `decimal`): `0`
* Mantıksal (`bool`): `false`
* Karakter (`char`): `'\0'` (null karakter)
* Referans tipleri (`string`, diziler, nesneler): `null`

```csharp
public class Urun
{
    public string Baslik; // Otomatik: null
    public decimal Fiyat; // Otomatik: 0
    public bool StoktaMi; // Otomatik: false
}
```

---

## 5.3) Erişim Belirleyiciler (Access Modifiers)

Erişim belirleyiciler, sınıfın ve üyelerinin dış dünyadan veya diğer projelerden ne kadar görünür olduğunu belirler. C#'ta **6 erişim seviyesi** bulunur:

| Belirleyici | Görünürlük Kapsamı |
| :--- | :--- |
| **`public`** | Her yerden (proje içi, dış projeler, referans verilen tüm assembly'ler) serbestçe erişilebilir. |
| **`private`** | Yalnızca tanımlandığı sınıf veya struct gövdesi içinden erişilebilir. Dışarıya tamamen kapalıdır. |
| **`internal`** | Yalnızca **aynı derleme birimi (aynı assembly / aynı .dll veya .exe projesi)** içinden erişilebilir. |
| **`protected`** | Tanımlandığı sınıf içinden ve ondan türeyen (miras alan) alt sınıflardan erişilebilir. |
| **`protected internal`** | Aynı assembly içindeki **herhangi bir kod** VEYA farklı assembly'deki **türetilmiş sınıflar** erişebilir (Union / VEYA mantığı). |
| **`private protected`** | Yalnızca **aynı assembly içindeki** ve o sınıftan **türetilmiş sınıflar** erişebilir (Intersection / VE mantığı). |

### Varsayılan (Default) Erişim Belirleyicileri (Büyük Sınav Tuzağı!)
Kod yazarken başına hiçbir belirteç yazılmazsa derleyici otomatik olarak en güvenli (en kısıtlayıcı) seviyeyi atar:

1. **Sınıf Düzeyinde (Top-Level Types):** Namespace altında doğrudan tanımlanan bir sınıf, arayüz veya enum varsayılan olarak **`internal`**'dır. *(Top-level bir sınıf asla `private` veya `protected` yapılamaz!)*
2. **Sınıf Üyeleri Düzeyinde (Class Members):** Bir sınıfın alanları, metotları ve özellikleri varsayılan olarak **`private`**'tır.

```csharp
namespace Magaza
{
    class Sepet // Belirtilmedi -> 'internal class Sepet'
    {
        int urunAdedi; // Belirtilmedi -> 'private int urunAdedi'
        void Hesapla() { } // Belirtilmedi -> 'private void Hesapla()'
    }
}
```

---

## 5.4) Kapsülleme (Encapsulation) Kavramı

Kapsülleme; bir sınıfın iç çalışma detaylarını ve kritik alanlarını (fields) dış dünyadan gizleyerek (`private`), verilere erişimi ve değişiklikleri kontrollü kurallara bağlama ilkesidir ("Bilmesi Gerekenler Prensibi" / Data Hiding).

### Neden Kapsülleme?
Eğer alanlar doğrudan `public` yapılırsa, dışarıdan tutarsız ve geçersiz veriler girilerek sistemin kararlılığı bozulabilir:
```csharp
// Kötü Tasarım (Kapsülleme Yok):
public class BankaHesabi
{
    public decimal Bakiye; // Herkes doğrudan değiştirebilir!
}

var hesap = new BankaHesabi();
hesap.Bakiye = -50000; // Geçersiz veri engellenemez!
```

---

## 5.5) Modern C# Özellikleri (Properties)

C#, kapsüllemeyi getter ve setter metotlarıyla boğuşmadan çözmek için birinci sınıf **Property (Özellik)** desteği sunar. Özellikler dışarıdan bakıldığında bir değişken gibi kullanılırken, arka planda güvenli metotlar (`get` ve `set`) gibi çalışır.

### 1. Full Property (Backing Field ile Doğrulama)
Veriyi tutan özel bir alan (`private backing field`) ve dışarıya açılan kontrollü bir özellik tanımlanır. Değer atanırken `value` anahtar sözcüğü dışarıdan gönderilen değeri taşır:

```csharp
public class Kisi
{
    private int yas; // Backing field (Veri kasası)

    public int Yas
    {
        get { return yas; }
        set
        {
            if (value < 0 || value > 120)
                throw new ArgumentOutOfRangeException(nameof(value), "Yaş 0 ile 120 arasında olmalıdır!");
            yas = value; // 'value' gizli parametredir
        }
    }
}
```

### 2. Auto-Implemented Property
Eğer ekstra bir doğrulama kuralı gerekmiyorsa, backing field yazmaya gerek yoktur; C# derleyicisi arka planda adı görünmeyen gizli bir alan üretir:

```csharp
public class Kitap
{
    public string Baslik { get; set; } = "İsimsiz"; // İlk değer atama (Property Initializer)
    public decimal Fiyat { get; set; }
}
```

### 3. Read-Only ve Expression-Bodied Properties
Sadece okunabilen özellikler dışarıdan değiştirilemez:

```csharp
public class Musteri
{
    public string Ad { get; }
    public string Soyad { get; }

    public Musteri(string ad, string soyad)
    {
        Ad = ad;
        Soyad = soyad;
    }

    // Expression-bodied hesaplanan özellik (Backing field tutmaz, çağrıldıkça anlık üretir)
    public string TamAd => $"{Ad} {Soyad}";
}
```

### 4. `init` Accessor (C# 9.0+ Değişmezlik / Immutability)
`init`, nesne oluşturulduğu anda (Object Initializer içinde) değer atanmasına izin verir, fakat nesne oluştuktan sonra değerin değiştirilmesini kesin olarak yasaklar:

```csharp
public class Fatura
{
    public int FaturaNo { get; init; }
    public decimal Tutar { get; init; }
}

// Kullanım:
var f = new Fatura { FaturaNo = 1001, Tutar = 250.50m }; // Geçerli
// f.Tutar = 300; // DERLEME HATASI! (CS8852: Init-only property can only be assigned in an object initializer)
```

### 5. Accessor Düzeyinde Erişim Kısıtlama Kuralı
`get` veya `set` bloklarından birine daha dar bir erişim seviyesi verilebilir:
* **Altın Kural:** Accessor'ın erişim belirteci, özelliğin kendi erişim belirtecinden **daha kısıtlayıcı** olmak zorundadır!

```csharp
public class Kullanici
{
    // Okuma herkese açık, değiştirme yalnızca sınıf içinden yapılabilir
    public string ParolaHash { get; private set; }

    // DERLEME HATASI: Özellik private iken get public yapılamaz!
    // private int Kod { public get; set; }
}
```

---

## 5.6) Sınav ve Mülakat Tuzakları

1. **Referans Eşitleme Yanılsaması:**
   `Oyuncu p2 = p1;` yazıldığında bellekte yeni bir oyuncu nesnesi oluşmaz; yalnızca aynı nesneyi gösteren Stack'teki kumanda adresi kopyalanır. Birinden yapılan değişiklik diğerine de yansır.
2. **Varsayılan Erişim Düzeyi Sürprizi:**
   Namespace altında `class Test { }` yazıldığında varsayılan olarak `internal`'dır. Sınıf içine `int sayi;` yazıldığında varsayılan olarak `private`'tır.
3. **Property Accessor Kısıtlama Kuralı İhlali:**
   Bir özelliğin genel seviyesi `internal` iken `set` bloğuna `public` verilemez; accessor her zaman özelliğin kendisinden daha dar kapsamlı olmalıdır.
4. **Yerel Değişken vs Sınıf Alanı İlk Değeri:**
   Yerel değişken ilk değersiz okunamaz (derleme hatası), sınıf alanları ise `new` anında otomatik olarak 0, false veya null değerini alır.

---

## 5.7) Altın Ezber Cümleleri

> 🔑 **1. Sınıf & Nesne:** "Sınıf bir taslaktır bellekte veri alanı tutmaz; nesne ise o taslaktan `new` ile Heap'te doğan canlı varlıktır."
> 
> 🔑 **2. Referans Kopyalama:** "Nesnelerde atama (`b = a`) nesneyi değil, kumandayı (adresi) kopyalar; televizyon tektir!"
> 
> 🔑 **3. Varsayılan Erişim:** "Sınıfın kendisi varsayılan olarak `internal`, sınıfın üyeleri varsayılan olarak `private`'tır."
> 
> 🔑 **4. Kapsülleme:** "Alanları (field) dış dünyaya kapatıp `private` tutmak, kontrollü erişimi `Property` ile sağlamak veri bütünlüğünün temelidir."
> 
> 🔑 **5. `init` Mantığı:** "`init` accessor'ı nesne inşa edilirken değer alabilir, inşa bittikten sonra değiştirilemez (Immutable veri)."
> 
> 🔑 **6. Accessor Kuralı:** "`get` veya `set`'in erişim seviyesi, özelliğin kendi erişim seviyesinden asla daha geniş olamaz."

