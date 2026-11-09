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
4. [Diziler ve Generic Koleksiyonlar](04_diziler_ve_koleksiyonlar.md)
5. [Sınıflar, Nesneler ve Kapsülleme](05_siniflar_ve_kapsulleme.md)
6. [Yapıcı Metotlar (Constructors) ve this Mimarisi](06_yapici_metotlar_ve_this.md)
7. [Statik Mimari ve Sabitler](07_statik_mimari_ve_sabitler.md)
8. [Kaynak Yönetimi, Yıkıcılar ve IDisposable](08_kaynak_yonetimi_ve_disposable.md)

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
- **Diziler & Boyut:** "Diziler referans tipidir; `.Length` eleman sayısını, `GetUpperBound` son geçerli indeksi verir."
- **Matris vs Jagged:** "`[,]` tek blokta dikdörtgendir, `[][]` düzensiz dizilerin dizisidir."
- **params:** "`params` sonda ve tek olur; tam eşleşen metot varsa `params` arka plana atılır."
- **Index & Range:** "`^1` son elemandır, `[1..4]` 1 dahil 4 hariç dilimler."
- **ArrayList vs List:** "`ArrayList` her şeyi `object` yaparak kutular (boxing) ve yorar; modern C#'ta tip güvenli `List<T>` kullanılır."
- **Dictionary:** "Anahtar benzersizdir; doğrudan köşeli parantezle sorma patlarsın, `TryGetValue` ile sor güvenle al!"
- **Sınıf & Nesne:** "Sınıf bir taslaktır bellekte veri alanı tutmaz; nesne ise o taslaktan `new` ile Heap'te doğan canlı varlıktır. `b = a` nesneyi değil, kumandayı (adresi) kopyalar."
- **Erişim & Kapsülleme:** "Sınıf varsayılan olarak `internal`, üyeleri `private`'tır. Alanları gizleyip Property ile dışarı açmak veri bütünlüğünün garantisidir; accessor kuralında `set` asla property'den geniş olamaz."
- **init Accessor:** "`init` ile işaretlenen özellikler yalnızca nesne inşa edilirken atanabilir, sonrasında değiştirilemez (Immutable)."
- **Yapıcı Metot (ctor) Tuzağı:** "Parametreli yapıcı yazıldığı an derleyici varsayılan parametresiz yapıcıyı yok eder; parametresiz üretmek istiyorsan açıkça elle yazacaksın!"
- **Constructor Chaining (`this(...)`):** "Yapıcılar birbirini `: this(...)` ile çağırarak kod tekrarını önler; zincirde yönlendirilen ana yapıcı gövdesi önce, çağıranın gövdesi en son çalışır."
- **Statik Kuralı:** "Statik üye nesneye değil tipe aittir (tek kopyadır); statik metot içinden `this` kullanılamaz ve nesne alanları doğrudan çağrılamaz!"
- **Statik Yapıcı (`static Sınıf()`):** "Statik ctor parametresiz ve belirteçsizdir; sınıfa ilk temasta CLR tarafından tam 1 kez çalıştırılır."
- **const vs readonly:** "`const` derleme anında koda gömülür ve örtülü statiktir; `readonly` çalışma anında (ctor içinde) değerlenir ve nesneye aittir."
- **Managed vs Unmanaged:** "Managed kaynakları GC otomatik temizler; dosya, soket ve DB bağlantısı gibi unmanaged kaynakları temizlemek yazılımcının görevidir."
- **Dispose & SuppressFinalize:** "`IDisposable.Dispose()` kaynakları deterministik (anında) serbest bırakır; `GC.SuppressFinalize(this)` GC'ye 'ben temizledim, finalizer kuyruğuna sokup yorma' der."
- **using Garantisi:** "`using` arka planda `try-finally`'e derlenir; kod hata fırlatsa dahi `Dispose()` çağrısını garanti eder. Unutma: `Dispose()` dış kaynakları kapatır, RAM'den nesneyi silen ise yalnızca GC'dir!"


