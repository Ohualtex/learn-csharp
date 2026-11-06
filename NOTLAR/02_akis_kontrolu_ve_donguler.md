# Bölüm 1: Akış Kontrolü, Pattern Matching ve Döngüler
*(Karar mekanizmaları, modern C# switch ifadeleri ve döngü yapıları)*

## 1.1) Klasik switch-case ve C#'a Özgü Fall-Through Kuralı

C#'ta `switch-case` yapısı diğer dillere (C/C++/Java) kıyasla daha sıkı kurallara sahiptir.

```csharp
int gun = 2;

switch (gun)
{
    case 1:
        Console.WriteLine("Pazartesi");
        break;
    case 2:
        Console.WriteLine("Salı");
        break; // C#'TA ZORUNLUDUR!
    case 6:
    case 7:
        Console.WriteLine("Hafta sonu"); // Boş case'ler birlikte çalışabilir
        break;
    default:
        Console.WriteLine("Bilinmeyen gün");
        break;
}
```

**Tuzaklar ve Nüanslar:**
- **Örtülü Fall-Through Yasağı:** C#'ta bir `case` bloğunun içi doluysa (en az bir satır kod varsa), sonuna `break`, `return` veya `throw` koymak **zorunludur**. Java ve C'de olduğu gibi alttaki `case`'e kendiliğinden akmasına derleyici izin vermez (Derleme Hatası: `Control cannot fall through from one case label`).
- **İstisna:** Yalnızca gövdesi tamamen boş olan `case` etiketleri (yukarıdaki 6 ve 7 gibi) alt alta bağlanabilir.
- Eğer bilinçli olarak bir sonraki case'e atlamak istiyorsanız C#'ta açıkça `goto case 3;` yazmanız gerekir.

> 🔑 **Ezber:** "C#'ta dolu case'in sonunda `break` unutulamaz; derleyici alttaki case'e gizlice düşüşe (fall-through) asla izin vermez!"

---

## 1.2) Modern C# Switch Expressions ve Pattern Matching

Modern C# (C# 8.0 ve sonrası), geleneksel hantal switch yapıları yerine çok daha temiz ve fonksiyonel olan **Switch Expression** (`=>`) ve **Pattern Matching** yapısını sunar.

```csharp
// 1. Switch Expression (=>) ile Değer Döndürme
int gunNo = 3;
string gunAdi = gunNo switch
{
    1 => "Pazartesi",
    2 => "Salı",
    3 => "Çarşamba",
    4 => "Perşembe",
    5 => "Cuma",
    _ => "Hafta sonu veya geçersiz" // '_' (discard) klasik switch'teki default'tur
};
Console.WriteLine(gunAdi); // Çarşamba

// 2. Relational (İlişkisel) Pattern Matching (C# 9+)
int sicaklik = 25;
string durum = sicaklik switch
{
    < 0 => "Dondurucu",
    >= 0 and <= 15 => "Soğuk",
    > 15 and <= 25 => "Ilık", // Burası eşleşir
    _ => "Sıcak"
};
Console.WriteLine(durum); // Ilık

// 3. Tip Kontrolü ve Değişkene Atama (Type Pattern)
object veri = "Merhaba C#";

if (veri is string metin)
{
    // veri string ise otomatik olarak 'metin' değişkenine cast edilir
    Console.WriteLine($"Metin uzunluğu: {metin.Length}");
}
```

**Tuzaklar ve Nüanslar:**
- Switch Expression'da her kol virgül (`,`) ile ayrılır ve ifadenin en sonuna noktalı virgül (`;`) konur.
- Switch Expression'da tüm olası durumlar kapsanmalıdır. `_` (default) atılırsa ve eşleşmeyen bir değer gelirse `SwitchExpressionException` fırlatılır.

> 🔑 **Ezber:** "Modern C#'ta switch artık bir deyim değil, değer üreten bir ifadedir; `default` yerine alt çizgi `_`, iki nokta yerine ok `=>` kullanılır."

---

## 1.3) Döngüler ve Döngü Kırıcılar

C#'ta 4 temel döngü bulunur: `for`, `foreach`, `while` ve `do-while`.

```csharp
// 1. for Döngüsü (Sayaç belli olduğunda)
for (int i = 0; i < 3; i++)
{
    Console.Write(i + " "); // 0 1 2
}

// 2. foreach Döngüsü (Koleksiyon veya dizi üzerinde gezinirken)
string[] diller = { "C#", "F#", "TypeScript" };
foreach (string dil in diller)
{
    Console.WriteLine(dil);
    // dil = "Python"; // DERLEME HATASI! foreach iterasyon değişkeni readonly'dir!
}

// 3. while vs do-while
int sayac = 10;
while (sayac < 5)
{
    Console.WriteLine("while çalıştı"); // Şart baştan false olduğu için 0 kez çalışır
}

do
{
    Console.WriteLine("do-while çalıştı"); // Şart sonda kontrol edildiğinden EN AZ 1 KEZ çalışır
    sayac++;
} while (sayac < 5);
```

**Döngü Kırıcılar:**
- `break`: Döngüyü anında sonlandırır ve döngü bloğundan dışarı çıkar.
- `continue`: Döngünün o anki adımını pas geçer, doğrudan sonraki tura/artırmaya atlar.

**Tuzaklar:**
- `foreach` döngüsünde dönülen değişken (`dil`) **salt okunurdur (readonly)**. Döngü içinde `dil = "Yeni"` gibi bir atama yapmaya çalışmak derleme hatası verir.
- `do-while` döngüsünün sonundaki `while(...);` ifadesinin sonuna noktalı virgül koymak zorunludur.

> 🔑 **Ezber:** "`foreach` gezerken elemanı değiştiremezsin (readonly). `do-while` çıkış kapısındaki güvenliktir; şart ne olursa olsun en az 1 kez çalışır."

---

## 1.4) Ternary Operatörü ve Mantıksal Kısa Devre

```csharp
// Ternary (Koşul Operatörü ? :)
int bakiye = 150;
string mesaj = bakiye >= 100 ? "Yeterli Bakiye" : "Yetersiz Bakiye";

// Kısa Devre (Short-Circuit) Davranışı
bool flag = false;
int x = 5;

// && (VE): Sol taraf false ise sağ taraf ASLA çalıştırılmaz
if (flag && (++x > 5))
{
    // Burası çalışmaz
}
Console.WriteLine(x); // 5 (artırma tetiklenmedi!)

// || (VEYA): Sol taraf true ise sağ taraf ASLA çalıştırılmaz
bool dogru = true;
if (dogru || (++x > 5))
{
    // İçeri girer
}
Console.WriteLine(x); // 5 (yine artırma tetiklenmedi!)
```

**Tuzaklar:**
- Çift operatörler (`&&`, `||`) kısa devre yapar (tembeldir); tekli operatörler (`&`, `|`) ise her iki tarafı da inatla çalıştırır. Performans ve hata önleme açısından neredeyse her zaman `&&` ve `||` tercih edilir.

> 🔑 **Ezber:** "`? :` tek satırlık if-else'tir. `&&` ilk `false`'u görünce, `||` ilk `true`'yu görünce sağ tarafı çöpe atar, çalıştırmaz!"
