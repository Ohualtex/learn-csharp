# Bölüm 0: C# 101 (Programlamaya Giriş)
*(Nesnelere ve döngülere geçmeden önce bilinmesi gereken en temel kavramlar)*

## 0.1) Değişkenler, Yazdırma ve Kaçış Karakterleri

```csharp
int yas = 25;
double maas = 15000.50;
bool ogrenciMi = true;
char harf = 'A';
string isim = "Ahmet";

// Yazdırma (Write vs WriteLine)
Console.Write("Yan yana yazar");
Console.WriteLine("Yazar ve alt satıra geçer");

// String Interpolation ($) - Modern ve Tavsiye Edilen Yöntem
Console.WriteLine($"Ad: {isim}, Yaş: {yas}, Maaş: {maas}");

// Kaçış Karakterleri (\n, \t, vb.)
Console.WriteLine("Merhaba\nC#");       // \n: Alt satıra iner
Console.WriteLine("Ad\tSoyad");         // \t: Bir tab (sekme) boşluk bırakır
Console.WriteLine("\"C#\" harika");     // \": Çift tırnak karakteri yazdırır
Console.WriteLine("C:\\Kullanicilar");  // \\: Tek bir ters slash yazdırır

// Verbatim String (@) - Kaçış karakterlerini devre dışı bırakır (Dosya yollarında çok sevilir)
Console.WriteLine(@"C:\Kullanicilar\Ahmet\Belgeler");
```

**Tuzak:** `Console.Write` yazıp alt satıra geçmesini beklemek. `char` için tek tırnak (`'A'`), `string` için çift tırnak (`"A"`) kuralını unutmak. C#'ın büyük/küçük harf duyarlı (case-sensitive) bir dil olduğunu unutup `console.writeline` yazmak.

> 🔑 **Ezber:** "`Write` yan yana yazar, `WriteLine` yazıp alt satıra geçer. `$` ile süslü parantez içine değişken gömülür, `@` ile ters slash kaçışları iptal edilir."

---

## 0.2) Operatörler (Aritmetik, Karşılaştırma ve Mantıksal)

```csharp
// Mod Alma (%) - Kalanı verir
int kalan = 10 % 3; // 1 (10'un 3'e bölümünden kalan 1)

// Tam Sayı Bölmesi Tuzağı (int / int = int)
int a = 5;
int b = 2;
int sonucTam = a / b;       // 2 (Ondalık kısım atılır!)
double sonucDogru = (double)a / b; // 2.5 (En az bir taraf double olmalı)

// Mantıksal Operatörler (&&, ||, !)
bool x = true;
bool y = false;

Console.WriteLine(x && y); // false (VE: İkisi de true olmalı)
Console.WriteLine(x || y); // true  (VEYA: Biri true olsa yeter)
Console.WriteLine(!x);     // false (DEĞİL: Tersine çevirir)

// Karşılaştırma
Console.WriteLine(5 == 5); // true (Eşit mi?)
Console.WriteLine(5 != 4); // true (Eşit DEĞİL mi?)
```

**Tuzak:** Atama operatörü (`=`) ile eşitlik kontrolü (`==`) operatörünü karıştırmak. İki `int` sayıyı böldüğünde sonucun otomatik olarak `double` çıkacağını varsaymak (`5 / 2` sonucu `2`'dir, `2.5` değildir).

> 🔑 **Ezber:** "`=` atama yapar, `==` eşit mi diye sorar. `int / int` tam sayı verir; ondalık istiyorsan en az birini `double` yap!"

---

## 0.3) Temel Şart Blokları (if - else if - else)

```csharp
int puan = 75;

if (puan >= 90)
{
    Console.WriteLine("AA");
}
else if (puan >= 70)
{
    Console.WriteLine("BB"); // Burası çalışır
}
else
{
    Console.WriteLine("Kaldınız");
}
```

**Tuzak:** `if` şartının hemen sonuna noktalı virgül (`;`) koymak. `if (puan > 50); { ... }` yazarsanız, `if` bloğu o noktada sonlanır ve süslü parantez içindeki kod şarttan bağımsız olarak her zaman çalışır.

> 🔑 **Ezber:** "if'in sonuna noktalı virgül konmaz. Zincirdeki ilk doğru şart çalışır, gerisi atlanır."

---

## 0.4) Kullanıcıdan Veri Alma (Console.ReadLine ve Tür Dönüşümleri)

```csharp
Console.Write("Adınızı girin: ");
string? ad = Console.ReadLine(); // ReadLine HER ZAMAN string döner

Console.Write("Yaşınızı girin: ");
string? yasGirdisi = Console.ReadLine();

// Yöntem 1: int.Parse (Kullanıcı harf girerse program çöker / FormatException)
int yas = int.Parse(yasGirdisi!);

// Yöntem 2: int.TryParse (Güvenli yöntem - Çökme yaşanmaz)
if (int.TryParse(yasGirdisi, out int guvenliYas))
{
    Console.WriteLine($"Giriş başarılı, yaşınız: {guvenliYas}");
}
else
{
    Console.WriteLine("Lütfen geçerli bir tam sayı giriniz!");
}
```

**Tuzak:** `Console.ReadLine()` çıktısını doğrudan sayısal bir değişkene atamaya çalışmak (örn: `int x = Console.ReadLine();`). Kullanıcı sayı yerine harf girdiğinde `int.Parse` fonksiyonunun programı kırması.

> 🔑 **Ezber:** "`Console.ReadLine` her zaman string okur. Sayı istiyorsan `Parse` et, program patlamasın istiyorsan `TryParse` kullan!"

---

## 0.5) Metotların Temeli (void vs return)

```csharp
class Program
{
    // 1. Geriye değer DÖNDÜRMEYEN metot (void)
    static void EkranaYazdir(string mesaj)
    {
        Console.WriteLine($"[Sistem Mesajı]: {mesaj}");
        // return değeri alamaz (yalnızca erken çıkış için 'return;' yazılabilir)
    }

    // 2. Geriye değer DÖNDÜREN metot (int, double, string vb.)
    static int Topla(int sayi1, int sayi2)
    {
        int toplam = sayi1 + sayi2;
        return toplam; // Değeri çağrıldığı yere iletir
    }

    static void Main()
    {
        EkranaYazdir("Hesaplama başlıyor..."); // Ekrana yazar, değer üretmez

        int sonuc = Topla(10, 20); // 30 değerini döndürür ve değişkene atar
        Console.WriteLine($"Toplam: {sonuc}");
    }
}
```

**Tuzak:** Geriye değer döndüren (`int`, `string` vb.) bir metotta `return` ifadesini unutmak (derleme hatası verir). `void` metottan dönen sonucu bir değişkene atamaya çalışmak (örn: `int a = EkranaYazdir("..");`).

> 🔑 **Ezber:** "`void` iş yapar susar, `return` değeri hesaplar ve sana fırlatır."
