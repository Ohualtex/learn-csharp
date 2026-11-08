# Bölüm 5: Nesne Yönelimli Programlama (OOP) - Sınıflar, Yapıcılar ve Kapsülleme
*(Sınıf vs Nesne, Erişim Belirleyiciler, Kapsülleme, Properties, `this` & Constructor Chaining, Statik Mimari, `const` vs `readonly`, Destructor & `IDisposable`)*

Nesne Yönelimli Programlama (Object-Oriented Programming - OOP), karmaşık yazılım sistemlerini gerçek dünya nesnelerine modelleyerek düzenlememizi sağlayan temel programlama paradigmasıdır. C# dili baştan sona saf bir OOP dili olarak tasarlanmıştır.

---

## 5.1) Sınıf (Class) ve Nesne (Object) Temelleri

* **Sınıf (Class):** Bir nesnenin durumunu (state - verilerini/alanlarını) ve davranışlarını (behavior - metotlarını) tanımlayan soyut bir **şablondur (blueprint)**. Kendi başına bellekte nesne verisi için yer tutmaz.
* **Nesne (Object / Instance):** Bir sınıftan `new` anahtar sözcüğü ile Heap belleğinde somutlaştırılmış (örneklendirilmiş) canlı bir varlıktır.

### Bellek Yerleşimi ve Referans Kopyalama Tuzağı
Bir sınıf referans tipidir (`Reference Type`). Değişken Stack'te yer alırken, asıl veriler Heap bölgesinde saklanır:

```csharp
public class Oyuncu
{
    public string Ad;
    public int Skor;
}

// Bellekte ne olur?
Oyuncu p1 = new Oyuncu(); // Stack'te p1 adresi -> Heap'te Oyuncu nesnesi (Ad = null, Skor = 0)
p1.Ad = "Miyamura";
p1.Skor = 100;

Oyuncu p2 = p1; // DİKKAT! Yeni bir oyuncu üretilmez! p1'in bellek adresi p2'ye kopyalanır.
p2.Skor = 500;

Console.WriteLine(p1.Skor); // Çıktı: 500! (İki kumanda tek bir televizyonu yönetir)
```

### Sınıf Alanlarının Varsayılan Değerleri (Default Values)
Metot içindeki yerel değişkenler (local variables) ilk değer verilmeden kullanılamazken (`CS0165: Use of unassigned local variable`), bir sınıfın alanları (fields) `new` ile nesne üretildiği anda CLR tarafından otomatik olarak türlerinin varsayılan değerlerine sıfırlanır:
* Sayısal tipler (`byte`, `int`, `double` vb.): `0`
* Mantıksal (`bool`): `false`
* Karakter (`char`): `'\0'` (null karakter)
* Referans tipleri (`string`, diziler, nesneler): `null`

> 🔑 **Ezber:** "Yerel değişken ilk değersiz okunamaz derleme hatası verir; sınıf alanları ise `new` anında otomatik olarak varsayılan değerlerini (0, false, null) alır!"

---

## 5.2) Erişim Belirleyiciler (Access Modifiers)

C# dilinde kod elemanlarının dışarıdan görünürlüğünü yönetmek için **6 temel erişim seviyesi** bulunur:

| Belirleyici | Görünürlük Kapsamı |
| :--- | :--- |
| **`public`** | Her yerden (proje içi, dış projeler, referans verilen tüm assembly'ler) erişilebilir. |
| **`private`** | Yalnızca tanımlandığı sınıf veya struct gövdesi içinden erişilebilir. |
| **`internal`** | Yalnızca **aynı assembly (aynı proje / derleme çıktısı .dll/.exe)** içinden erişilebilir. |
| **`protected`** | Yalnızca tanımlandığı sınıf içinden ve ondan türeyen (miras alan) alt sınıflardan erişilebilir. |
| **`protected internal`** | Aynı assembly içindeki **herhangi bir kod** VEYA farklı assembly'deki **türetilmiş alt sınıflar** erişebilir (Union - VEYA mantığı). |
| **`private protected`** | Yalnızca **aynı assembly içindeki** ve o sınıftan **türetilmiş alt sınıflar** erişebilir (Intersection - VE mantığı). |

### Varsayılan (Default) Erişim Belirleyicileri (Büyük Sınav Tuzağı!)
Bir bildirimin başına hiçbir erişim belirteci yazılmazsa C# derleyicisi en kısıtlayıcı geçerli seviyeyi atar:

1. **Sınıf Düzeyinde (Top-Level Types):** Namespace altında doğrudan tanımlanan sınıflar, arayüzler ve enum'lar varsayılan olarak **`internal`**'dır. (Top-level bir sınıf asla `private` veya `protected` olamaz!).
2. **Sınıf Üyeleri Düzeyinde (Class Members):** Bir sınıfın alanları, metotları, özellikleri ve yapıcıları varsayılan olarak **`private`**'tır.

```csharp
namespace OyunProjesi
{
    class Dusman // Belirtilmedi -> 'internal class Dusman'
    {
        int can; // Belirtilmedi -> 'private int can'
        void Saldir() { } // Belirtilmedi -> 'private void Saldir()'
    }
}
```

---

## 5.3) Kapsülleme (Encapsulation) ve Özellikler (Properties)

Kapsülleme; bir sınıfın iç çalışma detaylarını ve kritik verilerini dış dünyadan gizleyerek, verilere erişimi ve değişiklikleri kontrollü kurallara bağlama ilkesidir ("Bilmesi Gerekenler Prensibi").

### Geleneksel Getter / Setter (Java Tarzı)
```csharp
public class BankaHesabi
{
    private decimal bakiye; // Field (Alan) - dışarıya kapalı

    public decimal GetBakiye() => bakiye;

    public void SetBakiye(decimal miktar)
    {
        if (miktar >= 0)
            bakiye = miktar;
    }
}
```

### Modern C# Özellikleri (Properties)
C#, `get` ve `set` bloklarını birinci sınıf dil desteği (Property) olarak sunar. Dışarıdan değişken gibi görünür ancak arka planda metot gibi çalışır.

#### 1. Full Property (Backing Field ile Doğrulama)
Değer atanırken `value` anahtar sözcüğü, dışarıdan gönderilen değeri temsil eder:
```csharp
public class Kisi
{
    private int yas; // Backing Field

    public int Yas
    {
        get { return yas; }
        set
        {
            if (value < 0 || value > 150)
                throw new ArgumentOutOfRangeException(nameof(value), "Yaş 0-150 arasında olmalıdır!");
            yas = value; // 'value' gizli parametredir
        }
    }
}
```

#### 2. Auto-Implemented Property
Ekstra bir doğrulama mantığı gerekmiyorsa backing field derleyici tarafından gizlice üretilir:
```csharp
public class Urun
{
    // C# derleyicisi arka planda gizli bir backing field oluşturur
    public string Ad { get; set; } = "İsimsiz"; // İlk değer atama (Property Initializer)
    public decimal Fiyat { get; set; }
}
```

#### 3. Salt Okunur (Read-Only) ve Expression-Bodied Properties
```csharp
public class Ogrenci
{
    public string Ad { get; } // Yalnızca yapıcı metot (ctor) içinden atanabilir
    public string Soyad { get; }

    public Ogrenci(string ad, string soyad)
    {
        Ad = ad;
        Soyad = soyad;
    }

    // Expression-bodied hesaplanan özellik (Backing field tutmaz, her çağrıldığında hesaplar)
    public string TamAd => $"{Ad} {Soyad}";
}
```

#### 4. `init` Accessor (C# 9.0+)
Nesne oluşturulurken (Object Initializer içinde) değer atanmasına izin verir, ancak nesne oluştuktan sonra değiştirilemez (**Immutable** nesneler için mükemmeldir):
```csharp
public class Siparis
{
    public int Id { get; init; } // Sadece 'new' anında atanabilir!
}

var s = new Siparis { Id = 101 }; // Geçerli
// s.Id = 102; // DERLEME HATASI! (CS8852: Init-only property can only be assigned in an object initializer)
```

#### 5. Accessor Düzeyinde Erişim Kısıtlama Kuralı
`get` veya `set` bloklarından birine daha dar bir erişim belirteci verilebilir.
* **Kural:** Accessor'ın erişim belirteci, özelliğin kendisinden **daha kısıtlayıcı** olmak zorundadır!
```csharp
public class Kullanici
{
    // Okuma herkese açık (public), değiştirme yalnızca sınıf içine açık (private)
    public string ParolaHash { get; private set; }

    // GEÇERSİZ: Özellik private iken get public yapılamaz!
    // private int Sayi { public get; set; } // DERLEME HATASI!
}
```

---

## 5.4) `this` Anahtar Sözcüğü ve Constructor Chaining

`this`, üzerinde çalışılan geçerli nesne örneğinin bellek adresini gösteren gizli bir referanstır.

### `this` Kullanım Alanları:
1. **İsim Çakışmasını Çözme (Shadowing):** Metot parametresi ile sınıf alanının ismi aynı olduğunda:
   ```csharp
   public void BilgiAta(string ad)
   {
       this.ad = ad; // this.ad sınıf alanıdır, ad ise parametredir
   }
   ```
2. **Geçerli Nesneyi Başka Bir Metoda Parametre Olarak Gönderme:**
   ```csharp
   public void Kaydet()
   {
       VeritabaniServisi.Kaydet(this); // Kendisini gönderir
   }
   ```
3. **Constructor Chaining (`this(...)`):** Bir yapıcı metodun aynı sınıftaki başka bir yapıcı metodu çağırmasıdır.

### Constructor Chaining ile Kod Tekrarını Önleme
Bir yapıcı metot, parametrelerini diğerine aktararak zincirleme çağrı yapabilir:

```csharp
public class Calisan
{
    public string Ad { get; set; }
    public decimal Maas { get; set; }
    public string Departman { get; set; }

    // Ana (Master) Yapıcı
    public Calisan(string ad, decimal maas, string departman)
    {
        Ad = ad;
        Maas = maas;
        Departman = departman;
    }

    // 2 Parametreli Yapıcı: Departmanı varsayılan olarak "Genel" geçip ana yapıcıyı çağırır
    public Calisan(string ad, decimal maas) : this(ad, maas, "Genel")
    {
        // Önce : this(...) çalışır, ardından bu gövde işletilir
    }

    // Parametresiz Yapıcı: Asgari değerlerle 2 parametreli yapıcıyı tetikler
    public Calisan() : this("Tanımsız", 17002, "Genel")
    {
    }
}
```

> ⚠️ **Çalışma Sırası Tuzağı:** `Calisan("Ali", 30000)` çağrıldığında, önce `: this(...)` ile yönlendirilen 3 parametreli yapıcının gövdesi çalışır, ardından çağıran yapıcının gövdesi işletilir!

---

## 5.5) Yapıcı Metotlar (Constructors)

Yapıcı metot (Constructor), bir sınıftan `new` ile nesne üretildiği anda bellekte ilk ayarları ve başlangıç değerlerini atamak üzere çalışan özel bir metottur.

### Kurallar:
* Adı, sınıfın adıyla **birebir aynı** olmak zorundadır.
* Geri dönüş tipi **yoktur** (hatta `void` dahi yazılamaz; yazılırsa sıradan bir metot sayılır).
* Aşırı yüklenebilir (Overloading yapılabilir).

### Varsayılan Yapıcı (Default Constructor) ve Büyük Sınav Tuzağı!
Eğer bir sınıfa **hiçbir yapıcı metot yazmazsanız**, C# derleyicisi arka planda parametresiz ve içi boş bir varsayılan yapıcı ekler:
```csharp
public class Kitap
{
    public string Baslik;
    // Derleyici otomatik ekler: public Kitap() { }
}

Kitap k = new Kitap(); // Sorunsuz çalışır!
```

> 🚨 **BÜYÜK SINAV VE MÜLAKAT TUZAĞI:**
> Sınıfınıza **herhangi bir parametreli yapıcı metot yazdığınız anda**, C# derleyicisi varsayılan parametresiz yapıcıyı **OTOMATİK OLARAK SİLER**!
>
> ```csharp
> public class Kitap
> {
>     public string Baslik;
>     public Kitap(string baslik) // Parametreli ctor eklendi!
>     {
>         Baslik = baslik;
>     }
> }
> 
> // DERLEME HATASI! (CS7036: There is no argument given that corresponds to the required parameter...)
> Kitap k = new Kitap(); 
> ```
> Parametresiz nesne üretimine devam etmek istiyorsanız, `public Kitap() { }` gövdesini **kendi ellerinizle açıkça (explicitly)** yazmak zorundasınız!

---

## 5.6) Statik Mimari (Static Members, Static Class, Static Constructor)

`static` anahtar sözcüğü, bir üyenin nesne örneklerine (instances) değil, doğrudan **sınıfın tipine (Type)** ait olduğunu belirtir.

### Bellek Davranışı
* Instance üyeler: Her yeni `new Sınıf()` çağrısında Heap'te her nesne için ayrı ayrı çoğaltılır.
* `static` üyeler: Kaç nesne üretilirse üretilsin (ister 0, ister 1 milyon), bellekte (High Frequency Heap / Metot Tablosu) **yalnızca tek bir kopya** olarak saklanır ve tüm uygulama genelinde paylaşılır.

```csharp
public class Sayac
{
    public static int ToplamNesneSayisi = 0; // Ortak havuz
    public int NesneNo = 0; // Her nesneye özel

    public Sayac()
    {
        ToplamNesneSayisi++;
        NesneNo++;
    }
}

Sayac s1 = new Sayac();
Sayac s2 = new Sayac();

Console.WriteLine(s2.NesneNo);               // Çıktı: 1 (Her nesnenin kendi alanı)
Console.WriteLine(Sayac.ToplamNesneSayisi);  // Çıktı: 2 (Tüm nesnelerin ortak sayacı)
// Console.WriteLine(s1.ToplamNesneSayisi);  // DERLEME HATASI! Static üyelere nesne referansı üzerinden erişilemez!
```

### Statik Metotların Altın Kuralı
Statik bir metot, nesneden bağımsız olarak `SinifAdi.Metot()` şeklinde çağrılır.
> 🚨 **ALTIN KURAL:** Statik bir metot içerisinden doğrudan `this` kullanılamaz ve sınıfın `static` olmayan (instance) alanlarına veya metotlarına **DOĞRUDAN ERİŞİLEMEZ!**
> *(Çünkü statik metot çalıştığı anda ortada henüz üretilmiş bir nesne örneği olmayabilir!)*

```csharp
public class Ornek
{
    public int x = 10;
    public static int y = 20;

    public static void StatikMetot()
    {
        Console.WriteLine(y); // GEÇERLİ (Statik üye)
        // Console.WriteLine(x); // DERLEME HATASI! (CS0120: An object reference is required for the non-static field 'x')
        // Console.WriteLine(this.x); // DERLEME HATASI! Statik bağlamda 'this' kullanılamaz!

        // Çözüm: Ancak yeni bir nesne oluşturup onun üzerinden erişebilir:
        Ornek o = new Ornek();
        Console.WriteLine(o.x); // GEÇERLİ
    }
}
```

### Statik Yapıcı Metot (`static Sınıf()`)
Sınıfın statik verilerini başlatmak (initialize etmek) veya tek seferlik konfigürasyonları yüklemek için kullanılır.

#### Kuralları (Sınavlarda Sıkça Sorulur):
1. **Erişim Belirleyicisi Alamaz:** Ne `public` ne de `private` yazılabilir.
2. **Parametre Alamaz:** Aşırı yüklenemez (overload edilemez).
3. **Elle Çağrılamaz:** Programcı tarafından doğrudan çağrılması imkansızdır.
4. **Ne Zaman Çalışır?** Sınıftan İLK nesne örneği üretilmeden hemen önce VEYA sınıfın herhangi bir statik üyesine İLK KEZ erişilmeden hemen önce CLR tarafından **otomatik olarak ve ömrü boyunca TAM BİR KEZ** çalıştırılır.

```csharp
public class VeritabaniBaglantisi
{
    public static string BaglantiMetni;

    // Statik Yapıcı Metot
    static VeritabaniBaglantisi()
    {
        // Uygulama boyunca ilk temasta sadece bir kere çalışır
        BaglantiMetni = "Server=myServerAddress;Database=myDataBase;";
        Console.WriteLine("Statik ctor çalıştı!");
    }

    public VeritabaniBaglantisi()
    {
        Console.WriteLine("Instance ctor çalıştı!");
    }
}

// Kullanım:
var b1 = new VeritabaniBaglantisi(); 
// Çıktı:
// 1. Statik ctor çalıştı!
// 2. Instance ctor çalıştı!

var b2 = new VeritabaniBaglantisi();
// Çıktı:
// 1. Instance ctor çalıştı! (Statik ctor bir daha ASLA çalışmaz!)
```

### Statik Sınıflar (`static class`)
Eğer bir sınıf yalnızca yardımcı (utility) metotlar ve sabitler içeriyorsa `static class` olarak işaretlenir (Örn: `System.Math`, `System.Console`):
* İçerisinde `static` olmayan hiçbir üye (alan, metot, property) barındıramaz.
* `new` anahtar sözcüğü ile nesnesi **asla türetilemez**.
* Başka bir sınıftan miras alamaz (`System.Object` hariç) ve kendisinden **miras alınamaz** (arka planda derleyici tarafından örtülü olarak `abstract sealed` işaretlenir).

---

## 5.7) Sabitler ve Salt Okunurlar: `const` vs `readonly` vs `static readonly`

Yazılım mülakatlarının ve akademik sınavların en popüler karşılaştırma konusudur.

| Kriter | `const` (Sabit) | `readonly` (Salt Okunur) | `static readonly` |
| :--- | :--- | :--- | :--- |
| **Değer Atama Zamanı** | **Derleme Anı** (Compile-time) | **Çalışma Anı** (Runtime) | **Çalışma Anı** (Runtime) |
| **Nerede Atanabilir?** | Yalnızca bildirildiği satırda | Bildirildiği satırda veya **Instance Constructor** içinde | Bildirildiği satırda veya **Static Constructor** içinde |
| **Ait Olduğu Yer** | Örtülü olarak **`static`**'tir. Nesneye değil, sınıfa aittir (`Sinif.SABIT`). | **Nesneye** (instance) aittir. Her nesnenin farklı bir readonly değeri olabilir. | **Sınıfa** (türe) aittir. Tüm nesneler tek bir değeri paylaşır. |
| **Desteklenen Tipler** | Yalnızca ilkel tipler (`int`, `double`, `bool`, `char`), `string` ve `null`. | **Her türlü tip** (nesneler, diziler, koleksiyonlar, struct'lar). | Her türlü tip. |
| **DLL Versiyonlama (Değişim) Riski** | **YÜKSEK.** Değer, derleme anında çağıran kodun içine *inlined* (sabit gömülü) edilir. DLL değişirse çağıran exe yeniden derlenmek zorundadır! | **GÜVENLİ.** Değer çalışma anında bellekten okunur. DLL güncellense de yeniden derleme gerekmez. | **GÜVENLİ.** |

```csharp
public class Matematik
{
    // 1. const: Derleme anı sabiti. Otomatik static'tir. Asla değiştirilemez.
    public const double PI = 3.14159;

    // 2. readonly: Çalışma anı sabiti. Her nesne için farklı belirlenebilir.
    public readonly DateTime OlusturulmaZamani;

    // 3. static readonly: Sınıf seviyesinde çalışma anı sabiti.
    public static readonly string SunucuIp;

    static Matematik()
    {
        SunucuIp = "192.168.1.1"; // Statik ctor içinde atanabilir
    }

    public Matematik()
    {
        OlusturulmaZamani = DateTime.Now; // Instance ctor içinde atanabilir
        // PI = 3.14; // DERLEME HATASI! const değiştirilemez.
    }

    public void Metot()
    {
        // OlusturulmaZamani = DateTime.Now; // DERLEME HATASI! Metot içinde readonly değiştirilemez!
    }
}
```

---

## 5.8) Yıkıcı Metotlar (Destructors / Finalizers) ve `IDisposable`

.NET platformunda bellek yönetimi Garbage Collector (Çöp Toplayıcı - GC) tarafından otomatik olarak yürütülür. Ancak dosya tanıtıcıları, veritabanı bağlantıları veya ağ soketleri gibi **yönetilmeyen kaynaklar (unmanaged resources)** için özel temizleme mekanizmalarına ihtiyaç duyulur.

### 1. Yıkıcı Metot (Finalizer / Destructor - `~SinifAdi()`)
* Sözdizimi C++ yıkıcılarına benzer: `~DosyaYoneticisi() { }`
* Parametre almaz, erişim belirteci alamaz, aşırı yüklenemez.
* Arka planda derleyici tarafından `protected override void Finalize()` metoduna dönüştürülür.
* **Büyük Sorun:** GC tarafından ne zaman çalıştırılacağı **belirsizdir (non-deterministic)**. Uygulama kapanana kadar veya bellek dolana kadar hiç çalışmayabilir! Bu yüzden kritik kaynak temizliği için yıkıcı metotlara güvenilmez.

### 2. `IDisposable` Deseni ve `Dispose()` Metodu
Yönetilmeyen kaynakları ihtiyaç bittiği anda **deterministik (anında ve öngörülebilir)** olarak serbest bırakmak için `IDisposable` arayüzü uygulanır:

```csharp
public class DosyaIsleyici : IDisposable
{
    private bool disposed = false;

    // Kullanıcının veya using bloğunun işi bitince çağırdığı metot
    public void Dispose()
    {
        Temizle(true);
        // GC'ye: "Bu nesnenin temizliğini ben elle yaptım, Finalizer'ı çalıştırmakla vakit kaybetme!" der.
        GC.SuppressFinalize(this); 
    }

    protected virtual void Temizle(bool disposing)
    {
        if (!disposed)
        {
            if (disposing)
            {
                // Yönetilen (managed) kaynakları temizle
            }
            // Yönetilmeyen (unmanaged) kaynakları serbest bırak (Windows handle, socket vb.)
            disposed = true;
        }
    }

    // Yıkıcı Metot (Eğer programcı Dispose() çağırmayı unuttuysa emniyet sübabı)
    ~DosyaIsleyici()
    {
        Temizle(false);
    }
}
```

### 3. `using` İfadesi (Garanti Temizlik)
`IDisposable` uygulayan bir nesne `using` bloğu içine alındığında, kod hata (exception) fırlatsa dahi arka planda otomatik olarak bir `try-finally` bloğu oluşturulur ve `finally` içerisinde nesnenin `Dispose()` metodu mutlaka çağrılır:

```csharp
// Modern C# 8+ using bildirimi (Skop bitiminde otomatik Dispose() çağrılır):
using var dosya = new DosyaIsleyici();
// dosya ile işlemler...
// Metot sona erdiğinde dosya.Dispose() garanti olarak çalışır.

// Klasik using bloğu:
using (var baglanti = new DosyaIsleyici())
{
    // işlemler...
} // Blok bitiminde baglanti.Dispose() çağrılır.
```

---

## 5.9) Sınav ve Mülakat Tuzakları

1. **Parametreli Yapıcı Yazıldığında Varsayılan Yapıcının Kaybolması:**
   Sınıfa tek bir parametreli ctor eklendiği anda derleyici varsayılan ctor'ı siler. `new Sinif()` yazarsanız derleme hatası (`CS7036`) alırsınız.
2. **Statik Metot İçinden Instance Üyeye veya `this`'e Ulaşma Hatası:**
   Statik metotlar nesneye değil tipe aittir. İçerisinde `this` kullanılamaz; sınıfın statik olmayan alanlarına doğrudan ulaşılamaz (`CS0120`).
3. **`const` ile `readonly` Karışıklığı:**
   * `const` derleme anında çözülür, örtülü olarak statiktir ve yalnızca ilkel tipler/string ile çalışır.
   * `readonly` çalışma anında çözülür, nesneye aittir ve constructor içinde değeri değiştirilebilir.
4. **Property Accessor Kısıtlama Hatası:**
   Bir property'nin `get` veya `set` bloğuna erişim belirteci verilirken, property'nin kendi erişim belirtecinden **daha geniş** bir belirteç verilemez (`private int Sayi { public get; set; }` hatadır).
5. **Statik Yapıcı Metot Kuralları:**
   Statik yapıcı metot parametre ALAMAZ, erişim belirteci ALAMAZ (`public static Sinif()` derleme hatasıdır).
6. **Referans Eşitleme Yanılsaması:**
   `Sinif b = a;` yazıldığında bellekte yeni bir nesne oluşmaz; yalnızca aynı Heap nesnesini gösteren ikinci bir adres (kumanda) kopyalanır. `b` üzerindeki değişiklik `a`'yı da etkiler!

---

## 5.10) Altın Ezber Cümleleri

> 🔑 **1. Sınıf & Nesne:** "Sınıf mimari bir plandır bellekte veri alanı kaplamaz; nesne ise o plandan `new` ile Heap'te üretilen canlı yapıdır."
> 
> 🔑 **2. Varsayılan Erişim:** "Namespace altındaki sınıflar varsayılan olarak `internal`, sınıf içindeki üyeler varsayılan olarak `private`'tır."
> 
> 🔑 **3. Yapıcı Tuzağı:** "Parametreli yapıcı yazıldığı anda C# derleyicisi varsayılan parametresiz yapıcıyı yok eder; parametresiz üretmek istiyorsan elle yazacaksın!"
> 
> 🔑 **4. Statik Kuralı:** "Statik metot nesneden bağımsızdır; içerisinde `this` barındıramaz, nesne alanlarını doğrudan çağıramaz!"
> 
> 🔑 **5. `const` vs `readonly`:** "`const` derleme anında değerlenir ve örtülü static'tir; `readonly` çalışma anında (ctor içinde) değerlenebilir ve nesneye aittir."
> 
> 🔑 **6. `Dispose()` ve `SuppressFinalize`:** "`using` bloğu `Dispose()` çağrısını garanti eder; `GC.SuppressFinalize(this)` ise GC'ye 'ben temizledim, finalizer ile vakit kaybetme' der."

