# Bölüm 6: Yapıcı Metotlar (Constructors) ve this Mimarisi
*(ctor kuralları, default ctor silinme tuzağı, overloading, this(...) constructor chaining ve nesne ilklendirme)*

Bir nesnenin yaşam döngüsü Heap belleğinde tahsis edilmesiyle (allocation) başlar. Ancak bir nesnenin kullanılabilir hale gelmesi için ilk durumunun (state) doğru, tutarlı ve güvenli bir şekilde atanması gerekir. Bu görevi **Yapıcı Metotlar (Constructors)** üstlenir.

---

## 6.1) Yapıcı Metot (Constructor - `ctor`) Temelleri

Yapıcı metot, bir sınıftan `new` anahtar sözcüğü ile yeni bir nesne örneği üretildiği anda CLR tarafından **otomatik olarak ve yalnızca bir kez** çalıştırılan özel bir metottur.

### Temel Kurallar:
1. **İsim Kuralı:** Adı, tanımlandığı **sınıfın adıyla birebir aynı** olmak zorundadır (büyük/küçük harf duyarlıdır).
2. **Geri Dönüş Tipi Yoktur:** Geriye değer döndüremez. Başına `void` dahi **kesinlikle yazılamaz**! *(Eğer `void Ogrenci()` yazarsanız derleyici bunu yapıcı metot değil, alelade sıradan bir metot sayar ve nesne üretilirken otomatik çalıştırmaz!)*
3. **Erişim Belirleyicisi:** Genellikle `public` olur; ancak Singleton tasarım deseni gibi özel durumlarda dışarıdan nesne üretimini engellemek için `private` veya `protected` yapılabilir.

```csharp
public class Ogrenci
{
    public string Ad;
    public int Numara;

    // Yapıcı Metot (Constructor)
    public Ogrenci(string ad, int numara)
    {
        Ad = ad;
        Numara = numara;
    }
}

// Kullanım:
Ogrenci ogr = new Ogrenci("Miyamura", 542);
```

### Nesne Oluşumunun Bellek Aşamaları:
1. `new Ogrenci(...)` ifadesi yürütülür.
2. CLR Heap'te nesne için gerekli bellek alanını tahsis eder.
3. Tüm sınıf alanları önce otomatik olarak varsayılan değerlerine (`0`, `false`, `null`) sıfırlanır.
4. Tanımlanan yapıcı metot gövdesi çalıştırılır ve programcının belirlediği ilk değerler atanır.
5. Heap'teki bu yeni nesnenin bellek adresi Stack'teki değişkene (`ogr`) atanır.

---

## 6.2) Varsayılan Yapıcı (Default Constructor) ve Büyük Sınav Tuzağı

Eğer bir sınıfa **hiçbir yapıcı metot yazmazsanız**, C# derleyicisi arka planda parametresiz ve gövdesi boş bir varsayılan yapıcı (default constructor) üretir:

```csharp
public class Kitap
{
    public string Baslik;
    // Derleyici arka planda şunu üretir:
    // public Kitap() { }
}

Kitap k = new Kitap(); // Sorunsuz çalışır!
```

> 🚨 **BÜYÜK AKADEMİK VE MÜLAKAT TUZAĞI:**
> Sınıfınıza **herhangi bir parametreli yapıcı metot yazdığınız anda**, C# derleyicisi o otomatik varsayılan parametresiz yapıcıyı **ANINDA VE TAMAMEN SİLER!**

```csharp
public class Kitap
{
    public string Baslik;

    // Parametreli ctor eklendi!
    public Kitap(string baslik)
    {
        Baslik = baslik;
    }
}

// DERLEME HATASI! 
// CS7036: There is no argument given that corresponds to the required parameter 'baslik'
Kitap k1 = new Kitap(); 

// ÇÖZÜM:
// Parametresiz nesne üretiminin de çalışmasını istiyorsanız,
// parametresiz yapıcıyı kendi ellerinizle açıkça (explicitly) yazmalısınız:
public class Kitap
{
    public string Baslik;

    public Kitap() { } // Elle parametresiz ctor eklendi

    public Kitap(string baslik)
    {
        Baslik = baslik;
    }
}
```

---

## 6.3) Yapıcı Metot Aşırı Yükleme (Constructor Overloading)

Bir sınıfta farklı parametre sayılarına, farklı veri tiplerine veya farklı parametre sıralarına sahip birden fazla yapıcı metot tanımlanabilir. Buna **Constructor Overloading** denir.

```csharp
public class Araba
{
    public string Marka;
    public string Model;
    public int Yil;

    // 1. Parametresiz yapıcı
    public Araba()
    {
        Marka = "Belirtilmedi";
        Model = "Belirtilmedi";
        Yil = DateTime.Now.Year;
    }

    // 2. İki parametreli yapıcı
    public Araba(string marka, string model)
    {
        Marka = marka;
        Model = model;
        Yil = DateTime.Now.Year;
    }

    // 3. Üç parametreli yapıcı
    public Araba(string marka, string model, int yil)
    {
        Marka = marka;
        Model = model;
        Yil = yil;
    }
}
```

---

## 6.4) `this` Anahtar Sözcüğü ve Gölgeleme (Shadowing)

`this`, üzerinde çalışılan geçerli nesne örneğinin (instance) bellek adresini tutan gizli bir referanstır.

### Kullanım Amaçları:
1. **İsim Çakışmasını (Shadowing) Çözme:** Metot veya yapıcı parametresinin ismi ile sınıf alanının ismi aynı olduğunda, parametre sınıf alanını gölgeler. Sınıf alanına erişmek için `this.` kullanılır:
   ```csharp
   public class Kisi
   {
       private string ad;

       public Kisi(string ad)
       {
           this.ad = ad; // this.ad = sınıf alanı, ad = parametre
       }
   }
   ```
2. **Geçerli Nesneyi Başka Bir Metoda Parametre Olarak Gönderme:**
   ```csharp
   public void Raporla()
   {
       YaziciServisi.Yazdir(this); // Nesnenin kendisini gönderir
   }
   ```

---

## 6.5) Constructor Chaining (`this(...)`) ile Yapıcı Zincirleme

Birden fazla yapıcı metot tanımlandığında alanlara ilk değer atama kodlarının her yapıcıda tekrar yazılması **DRY (Don't Repeat Yourself)** ilkesine aykırıdır.

C#'ta bir yapıcı metot, `: this(...)` sözdizimi ile aynı sınıftaki başka bir yapıcı metodu çağırabilir. Buna **Constructor Chaining (Yapıcı Zincirleme)** denir.

```csharp
public class Calisan
{
    public string Ad { get; set; }
    public decimal Maas { get; set; }
    public string Departman { get; set; }

    // 1. Ana (Master) Yapıcı: Bütün atamaların yapıldığı merkez
    public Calisan(string ad, decimal maas, string departman)
    {
        Ad = ad;
        Maas = maas;
        Departman = departman;
        Console.WriteLine("3 Parametreli Master Yapıcı Çalıştı.");
    }

    // 2. İki Parametreli Yapıcı: Departmanı varsayılan verip master ctor'ı çağırır
    public Calisan(string ad, decimal maas) : this(ad, maas, "Genel")
    {
        Console.WriteLine("2 Parametreli Yapıcı Gövdesi Çalıştı.");
    }

    // 3. Parametresiz Yapıcı: Asgari değerlerle iki parametreli ctor'ı tetikler
    public Calisan() : this("Tanımsız", 17002)
    {
        Console.WriteLine("Parametresiz Yapıcı Gövdesi Çalıştı.");
    }
}
```

### ⚠️ Zincirleme Çalışma Sırası Tuzağı!
Eğer `new Calisan()` çağrılırsa yapıcıların işletilme sırası nasıldır?

```
new Calisan()
  │  (Parametresiz ctor önce : this(...) yönlendirmesine gider)
  └──► Calisan(ad, maas)
         │  (O da : this(...) ile master ctor'a gider)
         └──► Calisan(ad, maas, departman) [MASTER]
```

**Konsol Çıktı Sırası:**
1. `3 Parametreli Master Yapıcı Çalıştı.` *(En derindeki yönlendirilen ctor önce tamamlanır)*
2. `2 Parametreli Yapıcı Gövdesi Çalıştı.`
3. `Parametresiz Yapıcı Gövdesi Çalıştı.` *(İlk çağrılan ctor gövdesi en son tamamlanır)*

> 🔑 **Ezber:** "Constructor Chaining'de yönlendirilen (`: this(...)`) yapıcı önce çalışır, çağıran yapıcının kendi süslü parantez `{ }` içi en son işletilir!"

---

## 6.6) Object Initializer (Nesne Başlatıcı) ve Constructor İlişkisi

C# 3.0 ile gelen Object Initializer sözdizimi, nesne üretilirken erişilebilir özelliklere süslü parantezler içinde doğrudan değer atamamızı sağlar:

```csharp
public class Oyuncu
{
    public string Takim { get; set; }
    public int Numara { get; set; }

    public Oyuncu(string takim)
    {
        Takim = takim;
        Console.WriteLine("Constructor çalıştı.");
    }
}

// Kullanım:
var o = new Oyuncu("Fenerbahçe") { Numara = 10 };
```

### Çalışma Sırası Kuralı:
1. Önce Constructor çalışır (`Takim = "Fenerbahçe"` atanır).
2. Nesne örneği oluştuktan hemen sonra Object Initializer içindeki property atamaları yapılır (`Numara = 10`).

---

## 6.7) Sınav ve Mülakat Tuzakları

1. **Parametreli Yapıcı Yazıp Parametresiz Üretmeye Çalışmak:**
   Sınıfa tek bir parametreli ctor eklendiği anda derleyici varsayılan ctor'ı imha eder. Parametresiz üretmek istiyorsanız `public Sinif() { }` satırını elle eklemelisiniz.
2. **Yapıcı Metoda `void` Yazma Hatası:**
   `public void Sinif() { }` yazılırsa bu bir yapıcı değil, geri dönüş tipi `void` olan sıradan bir metottur! `new Sinif()` dendiğinde otomatik olarak çalışmaz.
3. **Döngüsel Constructor Zinciri (Circular Chaining) Hatası:**
   İki yapıcı birbirini karşılıklı çağırırsa (`A : this() -> B : this()`), derleyici `CS0516: Constructor cannot call itself` hatası verir.
4. **Statik Bağlamda `this` Tuzağı:**
   `this` anahtar sözcüğü yalnızca nesne örneklerine (instance) aittir. `static` metotlarda veya `static` özelliklerde `this` kullanılamaz (`CS0026`).

---

## 6.8) Altın Ezber Cümleleri

> 🔑 **1. Yapıcı Kuralı:** "Yapıcı metodun adı sınıfla aynıdır, dönüş tipi (`void` dahil) asla olamaz; `new` anında Heap'te tek bir kez çalışır."
> 
> 🔑 **2. Varsayılan Yapıcı Tuzağı:** "Parametreli yapıcı yazıldığı an varsayılan parametresiz yapıcı silinir; yoksa `new Sinif()` patlar!"
> 
> 🔑 **3. Gölgeleme (`this`):** "Parametre ile sınıf alanı çakıştığında `this.` sınıfın alanını işaret eder."
> 
> 🔑 **4. Chaining Sırası:** "`: this(...)` zincirinde yönlendirilen ana yapıcı gövdesi önce, çağıran yapıcının gövdesi ise en son çalışır."
> 
> 🔑 **5. Object Initializer Sırası:** "`new Sinif(...) { Prop = x }` kullanımında önce constructor çalışır, hemen ardından süslü parantez içindeki özellikler atanır."
