# Bölüm 7: Statik Mimari ve Sabitler
*(static alanlar, metotlar, static ctor, static class, const, readonly ve static readonly)*

Nesne Yönelimli Programlamada her nesne Heap belleğinde kendi durumunu (state) ve verilerini ayrı bir alanda taşır. Ancak bazı veriler ve davranışlar tekil nesnelere değil, **sınıfın kendisine (türe)** aittir. C# dilinde bu ayrım `static` anahtar sözcüğü ve sabit niteleyicileri (`const`, `readonly`) ile yönetilir.

---

## 7.1) `static` Felsefesi ve Bellek Modeli

* **Instance (Örnek) Üyeler:** `new` ile her yeni nesne üretildiğinde Heap'te o nesneye özel olarak çoğaltılır.
* **`static` (Statik) Üyeler:** Nesneye değil, doğrudan **sınıfın tipine (Type)** aittir. Kaç tane nesne üretilirse üretilsin (ister 0, ister 1 milyon), bellekte (High Frequency Heap / Metot Tablosu) **yalnızca tek bir kopya** olarak saklanır ve tüm uygulama genelinde paylaşılır.

```csharp
public class Sayac
{
    public static int ToplamUretilen = 0; // Sınıfa ait tek ortak havuz
    public int NesneId = 0;               // Her nesneye özel alan

    public Sayac()
    {
        ToplamUretilen++;
        NesneId++;
    }
}

// Kullanım:
Sayac s1 = new Sayac();
Sayac s2 = new Sayac();

Console.WriteLine(s2.NesneId);              // Çıktı: 1 (Her nesnenin kendi alanı)
Console.WriteLine(Sayac.ToplamUretilen);    // Çıktı: 2 (Tüm nesnelerin paylaştığı tek havuz)

// DERLEME HATASI! 
// CS0176: Static member cannot be accessed with an instance reference
// Console.WriteLine(s1.ToplamUretilen);
```

> 🔑 **Ezber:** "Statik üye sınıfa aittir ve bellekte tek kopyadır; nesne üzerinden değil doğrudan `SinifAdi.Uye` şeklinde çağrılır!"

---

## 7.2) Statik Metotlar ve Altın Kural

Statik metotlar, nesneden bağımsız olarak doğrudan sınıf adıyla (`Sinif.Metot()`) çalıştırılan işlevlerdir.

> 🚨 **ALTIN KURAL:** 
> Statik bir metot içerisinden doğrudan `this` anahtar sözcüğü kullanılamaz ve sınıfın `static` olmayan (instance) alanları veya metotları **DOĞRUDAN ÇAĞRILAMAZ!**
> *(Nedeni: Statik metot çağrıldığı anda Heap'te henüz hiçbir nesne örneği üretilmemiş olabilir; metot hangi nesnenin verisini okuyacağını bilemez!)*

```csharp
public class MatematikServisi
{
    public int carpan = 5; // Instance alan
    public static int sabitEk = 10; // Statik alan

    public static void Hesapla()
    {
        Console.WriteLine(sabitEk); // GEÇERLİ: Statik üye okunabilir

        // DERLEME HATASI! (CS0120: An object reference is required for the non-static field 'carpan')
        // Console.WriteLine(carpan); 

        // DERLEME HATASI! (CS0026: 'this' is not valid in a static property, method, or field)
        // Console.WriteLine(this.carpan); 

        // ÇÖZÜM: Instance bir üyeye ancak yeni bir nesne üretip onun üzerinden erişilebilir:
        MatematikServisi servis = new MatematikServisi();
        Console.WriteLine(servis.carpan); // GEÇERLİ
    }
}
```

---

## 7.3) Statik Yapıcı Metot (`static Sınıf()`)

Statik yapıcı metot (Static Constructor), sınıfın statik verilerine ilk değerlerini atamak veya tek seferlik konfigürasyonları yüklemek için kullanılan özel bir mekanizmadır.

### Sınavlarda Sıkça Çıkan 4 Katı Kural:
1. **Erişim Belirleyicisi Alamaz:** Başına `public`, `private` vb. yazılamaz (`CS0515`).
2. **Parametre Alamaz:** Hiçbir parametre alamaz, dolayısıyla aşırı yüklenemez (overloading yapılamaz - `CS0132`).
3. **Elle Çağrılamaz:** Programcı tarafından doğrudan çağrılması imkansızdır.
4. **Çalışma Zamanı (Tek Seferlik):** Sınıftan **İLK nesne üretilmeden önce** VEYA sınıfın herhangi bir statik üyesine **İLK KEZ erişilmeden hemen önce** CLR tarafından otomatik olarak ve **ömrü boyunca TAM BİR KEZ** çalıştırılır.

```csharp
public class VeritabaniYapilandirici
{
    public static string BaglantiCumlesi;

    // Statik Yapıcı Metot
    static VeritabaniYapilandirici()
    {
        BaglantiCumlesi = "Server=localhost;Database=UygulamaDb;";
        Console.WriteLine(">> 1. Statik Constructor Çalıştı (Sadece 1 Kez)");
    }

    public VeritabaniYapilandirici()
    {
        Console.WriteLine(">> 2. Instance Constructor Çalıştı");
    }
}

// Test:
var n1 = new VeritabaniYapilandirici();
// Çıktı:
// >> 1. Statik Constructor Çalıştı (Sadece 1 Kez)
// >> 2. Instance Constructor Çalıştı

var n2 = new VeritabaniYapilandirici();
// Çıktı:
// >> 2. Instance Constructor Çalıştı (Statik ctor bir daha ASLA çalışmaz!)
```

---

## 7.4) Statik Sınıflar (`static class`)

Yalnızca genel yardımcı (utility) metotlar ve sabitler barındıran yapılar `static class` olarak tanımlanır (Örn: `System.Math`, `System.Console`, `System.IO.File`).

### Kurallar:
* Yalnızca `static` üyeler barındırabilir; içerisine `static` olmayan bir alan veya metot yazılamaz.
* `new` anahtar sözcüğü ile nesnesi **asla üretilemez** (`CS0712`).
* Başka bir sınıftan miras alamaz (`System.Object` hariç) ve kendisinden **asla miras alınamaz** (C# derleyicisi bu sınıfları arka planda örtülü olarak `abstract sealed` işaretler).

```csharp
public static class Donusturucu
{
    // public int sayi; // DERLEME HATASI! Statik sınıfta non-static üye olamaz.

    public static double DereceToRadyan(double derece)
    {
        return derece * (Math.PI / 180.0);
    }
}
```

---

## 7.5) Sabitler ve Salt Okunurlar: `const` vs `readonly` vs `static readonly`

Yazılım mülakatlarının ve akademik OOP sınavlarının en popüler karşılaştırma konusudur:

| Kriter | `const` (Sabit) | `readonly` (Salt Okunur) | `static readonly` |
| :--- | :--- | :--- | :--- |
| **Değer Atama Zamanı** | **Derleme Anı** (Compile-time) | **Çalışma Anı** (Runtime) | **Çalışma Anı** (Runtime) |
| **Nerede Atanabilir?** | Yalnızca bildirildiği satırda | Bildirildiği satırda veya **Instance Constructor** içinde | Bildirildiği satırda veya **Static Constructor** içinde |
| **Ait Olduğu Yer** | Örtülü olarak **`static`**'tir. Sınıfa aittir (`Sinif.SABIT`). | **Nesneye** aittir. Her nesnenin farklı bir readonly değeri olabilir. | **Sınıfa** aittir. Tüm nesneler tek bir değeri paylaşır. |
| **Desteklenen Tipler** | Yalnızca ilkel tipler (`int`, `double`, `bool`, `char`), `string` ve `null`. | **Her türlü tip** (nesneler, diziler, koleksiyonlar, struct'lar). | Her türlü tip. |
| **DLL Versiyonlama (Inlining) Riski** | **YÜKSEK.** Değer, derleme anında çağıran kodun içine sabit gömülür (inlined). DLL değişirse çağıran exe yeniden derlenmelidir! | **GÜVENLİ.** Değer çalışma anında bellekten okunur. DLL güncellense de çağıran projenin yeniden derlenmesine gerek kalmaz. | **GÜVENLİ.** |

```csharp
public class SunucuYapilandirmasi
{
    // 1. const: Derleme anı sabiti (Örtülü static)
    public const int VarsayilanPort = 8080;

    // 2. readonly: Çalışma anı sabiti (Nesneye özel)
    public readonly string BaglantiId;

    // 3. static readonly: Sınıf düzeyinde çalışma anı sabiti
    public static readonly DateTime BaslatilmaZamani;

    static SunucuYapilandirmasi()
    {
        BaslatilmaZamani = DateTime.Now; // Statik ctor'da atanabilir
    }

    public SunucuYapilandirmasi(string baglantiId)
    {
        BaglantiId = baglantiId; // Instance ctor'da nesneye özel atanabilir
        // VarsayilanPort = 9000; // DERLEME HATASI! const değiştirilemez.
    }

    public void PortGuncelle()
    {
        // BaglantiId = "NEW-ID"; // DERLEME HATASI! Normal metot içinde readonly değiştirilemez!
    }
}
```

---

## 7.6) Sınav ve Mülakat Tuzakları

1. **Statik Metottan `this` veya Instance Alan Çağırma Hatası:**
   Statik metot nesneden bağımsızdır. `this.alan` veya `alan` şeklinde doğrudan nesne üyelerine erişmeye çalışmak `CS0120` derleme hatası verir.
2. **Statik Üyeyi Nesne Üzerinden Çağırma Tuzağı:**
   Java'da nesne üzerinden statik üye çağrısına izin verilirken (uyarı verir), C#'ta `nesne.StatikAlan` yazmak **kesin derleme hatasıdır (`CS0176`)**. Mutlaka `SinifAdi.StatikAlan` yazılmalıdır.
3. **Statik Constructor'a Belirteç veya Parametre Verme Hatası:**
   `public static Sinif()` veya `static Sinif(int x)` yazmak kesinlikle yasaktır. Statik ctor belirteçsiz ve parametresiz olmak zorundadır.
4. **`const` Değerini Constructor İçinde Atamaya Çalışmak:**
   `const` çalışma anını beklemez; derleme anında çözülmek zorundadır. Bu yüzden constructor içinde `const` değişkenine değer atanamaz (`readonly` ile karıştırılmamalıdır).

---

## 7.7) Altın Ezber Cümleleri

> 🔑 **1. Statik Bellek:** "Statik üye nesneye değil tipe aittir; bellekte tek bir kopya olarak tüm nesnelerce paylaşılır."
> 
> 🔑 **2. Statik Metot Kuralı:** "Statik metot nesneden bağımsızdır; içerisinde `this` kullanamaz, nesne alanlarını doğrudan çağıramaz!"
> 
> 🔑 **3. Statik ctor:** "Statik yapıcı parametresiz ve belirteçsizdir; sınıfa ilk temasta CLR tarafından ömrü boyunca tam 1 kez çalıştırılır."
> 
> 🔑 **4. Statik Sınıf:** "Statik sınıftan `new` ile nesne türetilemez, miras alınamaz ve verilemez; yalnızca statik üye barındırabilir."
> 
> 🔑 **5. `const` vs `readonly`:** "`const` derleme anında koda gömülür ve örtülü statiktir; `readonly` çalışma anında (ctor içinde) değerlenir ve nesneye aittir."
