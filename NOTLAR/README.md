# 📘 C# OOP — Notlar (Fihrist & Özet)

Bu klasör, C# ve .NET ile Nesne Yönelimli Programlama (OOP) öğrenme sürecindeki temel kavramları, sık yapılan hataları (tuzakları) ve akılda kalıcı kuralları bölümler halinde barındırır.

Çalışan tam örnekler için → [`../ORNEKLER/`](../ORNEKLER/)  
Alıştırmalar için → [`../ALISTIRMALAR/`](../ALISTIRMALAR/)

---

## 🗂️ Konu Başlıkları

0. [.NET Mimarisi ve CLR](00_dotnet_mimarisi_ve_clr.md)
1. [C# Temelleri (Programlamaya Giriş)](01_csharp_temelleri.md)
2. [Akış Kontrolü, Pattern Matching ve Döngüler](02_akis_kontrolu_ve_donguler.md)
3. [C# Bellek Modeli, Değer/Referans Tipleri ve Null Güvenliği](03_bellek_modeli_ve_null_guvenligi.md)

---

## 🔑 Hızlı Tekrar: Altın Ezber Cümleleri (Kopya Kağıdı)

- **.NET & CLR:** "C# kodunu çalıştıran sanal motor CLR'dır; CLR'ın yönettiği bellek güvenli koda Managed Code denir."
- **Derleme Akışı:** "C# derlenince makine kodu değil `IL` (ara dil) ve `Metadata` çıkar. `JIT` bu IL'i çalışma anında makine diline çevirir."
- **JIT Türleri:** "`Normal JIT` çağırdıkça derler ve saklar, `Pre-JIT` her şeyi baştan derler, `Eco-JIT` hafıza bitince derlediğini siler."
- **CTS vs CLS:** "`CTS` ortak tipleri tanımlar (`int` = `System.Int32`), `CLS` diller arası iletişimin kurallarını koyar."
- **csc Parametreleri:** "`/t:library` DLL yapar, `/unsafe` pointer açar."
- **Yazdırma ve Kaçış:** "`Write` yan yana yazar, `WriteLine` yazıp alt satıra geçer. `$` ile süslü parantez içine değişken gömülür, `@` ile ters slash kaçışları iptal edilir."
- **Operatörler & Bölme:** "`=` atama yapar, `==` eşit mi diye sorar. `int / int` tam sayı verir; ondalık istiyorsan en az birini `double` yap!"
- **if-else:** "if'in sonuna noktalı virgül konmaz. Zincirdeki ilk doğru şart çalışır, gerisi atlanır."
- **Console.ReadLine:** "`Console.ReadLine` her zaman string okur. Sayı istiyorsan `Parse` et, program patlamasın istiyorsan `TryParse` kullan!"
- **Metotlar:** "`void` iş yapar susar, `return` değeri hesaplar ve sana fırlatır."
- **Switch & Pattern Matching:** "C#'ta dolu case'in sonunda break unutulamaz; modern C#'ta switch değer üretir: default yerine `_`, iki nokta yerine `=>` kullanılır."
- **Döngüler:** "`foreach` okur, gezerken elemanı değiştiremezsin (readonly). `do-while` çıkış kapısındaki güvenliktir; şart ne olursa olsun en az 1 kez çalışır."
- **Kısa Devre (Short-Circuit):** "`&&` ilk `false`'u, `||` ilk `true`'yu görünce sağ tarafı çöpe atar, çalıştırmaz!"
- **Bellek & Tipler:** "Değer tipinde fotokopiyi verirsin, orijinal değişmez. Referans tipinde kumandayı verirsin, televizyon (orijinal) değişir!"
- **Parametreler (ref/out/in):** "`ref` dolu gelir değişebilir; `out` boş gelir, dolu çıkmak zorundadır; `in` bakar ama elleyemez (readonly)!"
- **Null Güvenliği:** "`?.` sormadan kırmaz, `??` boşsa yedeği koyar, `??=` yoksa doldurur, `!` derleyiciyi susturur ama runtime'ı kandıramaz!"


