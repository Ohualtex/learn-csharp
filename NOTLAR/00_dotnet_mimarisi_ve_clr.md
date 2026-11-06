# Bölüm 0: .NET Mimarisi ve CLR
*(Yönetilen kod, IL, JIT türleri, CTS/CLS ve sistemin çalışma prensipleri)*

## 0.1) .NET Mimarisi ve Temel Bileşenler

C# dili, Microsoft tarafından geliştirilen **.NET** platformunun üzerinde koşan, modern, nesne yönelimli ve tip güvenli (type-safe) bir programlama dilidir. C# kodlarının çalışabilmesi için işletim sistemi üzerinde .NET çalışma ortamının (Runtime) bulunması gerekir.

```
+-----------------------------------------------------------+
|               Diller: C#, F#, VB.NET, C++/CLI             |
+-----------------------------------------------------------+
|       Temel Dil Kuralları: CLS (Ortak Dil Spesifikasyonu) |
|       Ortak Tip Sistemi:  CTS (Common Type System)        |
+-----------------------------------------------------------+
|   Temel Sınıf Kütüphanesi (BCL - System, IO, Net, vb.)   |
+-----------------------------------------------------------+
|          CLR (Common Language Runtime - Motor)            |
|       [JIT Derleyici]  |  [Garbage Collector]  | [Güvenlik] |
+-----------------------------------------------------------+
|          İşletim Sistemi (Windows / macOS / Linux)        |
+-----------------------------------------------------------+
```

* **Yönetilen Kod (Managed Code):** Yalnızca **CLR** gözetimi ve yönetimi altında çalışan koddur. Bellek tahsisi, bellek temizliği (Garbage Collection) ve tip güvenliği CLR tarafından otomatik denetlenir.
* **Yönetilmeyen Kod (Unmanaged Code):** Doğrudan işletim sistemine ve donanıma hitap eden, bellek yönetimini (malloc/free/pointer) geliştiricinin manuel yaptığı C/C++ gibi dillerin çıktılarıdır.
* **İsim Alanları (Namespaces):** Binlerce sınıfı organize eden hiyerarşik klasörleme mantığıdır (`using System;` ile `System` isim alanındaki `Console`, `Math` gibi temel sınıflara doğrudan erişiriz).

> 🔑 **Ezber:** "C# doğrudan donanımla değil, CLR ile konuşur. CLR'ın yönettiği güvenli koda `Managed Code` denir."

---

## 0.2) İki Aşamalı Derleme: C# -> IL (MSIL) -> Makine Kodu

Geleneksel dillerde (C/C++) kaynak kod doğrudan işlemcinin anlayacağı makine koduna (.exe / binary) çevrilirdi. Bu nedenle Intel işlemcide derlenen kod, ARM veya farklı bir işlemcide/işletim sisteminde çalışmazdı.

.NET mimarisi platform bağımsızlığını sağlamak için **iki aşamalı derleme** modelini kullanır:

```
[Kaynak Kod (C#)]
       │
       ▼ (Derleme Aşaması - csc / dotnet build)
[Assembly (.exe / .dll)]
       ├── IL / MSIL (Ara Dil Kodu)
       └── Metadata (Tür, Metot ve Versiyon Bilgileri)
       │
       ▼ (Çalıştırma Aşaması - CLR Devreye Girer)
[JIT Derleyici (Just-In-Time Compiler)]
       │
       ▼
[Çalıştırılabilir Yerel Makine Kodu (Native Machine Code)]
```

1. **IL / MSIL (Intermediate Language / Ara Dil):** İşlemciden ve işletim sisteminden tamamen bağımsız, evrensel ara komut kümesidir.
2. **Metadata:** IL kodunun yanında gömülü bulunan "kimlik kartıdır". Programda kullanılan tiplerin, sınıfların, metotların imzalarını ve referans bağlantılarını saklar. Harici bir başlık (`.h`) dosyasına ihtiyaç bırakmaz.
3. **Assembly:** Derleme sonucunda oluşan `.dll` (kütüphane) veya `.exe` (çalıştırılabilir) uzantılı paket dosyasıdır. Manifest, metadata ve IL'den oluşur.

> 🔑 **Ezber:** "C# derlenince doğrudan makine kodu değil, ara dil olan `IL` oluşur; IL'i çalışma anında makineye çeviren motor ise `JIT`'tir!"

---

## 0.3) JIT Derleyicileri (Just-In-Time) ve Çeşitleri

**JIT (Tam Zamanında Derleyici):** Program çalıştığı anda (Runtime) devreye girer. IL kodunu alır ve programın çalıştığı spesifik CPU mimarisine (x86, x64, ARM64) uygun yerel makine koduna dönüştürür.

Üniversite müfredatlarında ve teknik sınavlarda sorulan **3 klasik JIT derleyici türü** şunlardır:

| JIT Türü | Çalışma Mantığı | Avantaj / Dezavantaj |
| :--- | :--- | :--- |
| **Normal JIT** *(Varsayılan)* | Yalnızca çağrılan metotları ihtiyaç anında derler ve derlenen makine kodunu bellekte (cache) tutar. | Metot ikinci kez çağrıldığında tekrar derlenmez, önbellekten şimşek hızında çalışır. |
| **Pre-JIT** *(NGen / AOT)* | Program çalıştırılmadan önce tüm IL kodunu tek seferde makine koduna dönüştürür. | İlk açılış performansı çok yüksektir; ancak çok fazla bellek (RAM/Disk) kaplar. |
| **Eco-JIT** *(Kısıtlı Sistemler)* | Düşük bellekli sistemler için tasarlanmıştır. Yalnızca gereken kısmı derler. | Bellek belirli bir sınırın altına düştüğünde, daha önce derlenmiş kodları bellekten siler; gerekirse sonra tekrar derler. |

*(Modern .NET notu: Günümüz modern .NET sürümlerinde bu yapı yerini yüksek optimizasyonlu **RyuJIT** ve katmanlı derleme olan **Tiered Compilation** mimarisine bırakmıştır).*

> 🔑 **Ezber:** "`Normal JIT` çağırdıkça derler ve saklar, `Pre-JIT` her şeyi baştan derler, `Eco-JIT` hafıza bitince derlediğini siler!"

---

## 0.4) CTS (Ortak Tip Sistemi) ve CLS (Ortak Dil Spesifikasyonu)

.NET platformunun en büyük gücü **diller arası entegrasyondur (Cross-Language Interoperability)**. Örneğin C# ile yazılmış bir sınıfı VB.NET veya F# içinde hiçbir uyum sorunu yaşamadan çağırabilirsiniz. Bunu sağlayan iki kardeş standart vardır:

### 1. CTS (Common Type System - Ortak Tip Sistemi)
.NET dünyasındaki tüm dillerin ortak veri tipleri sözlüğüdür. C#'taki `int` ile VB.NET'teki `Integer` aslında CTS'deki tek bir tipe karşılık gelir: **`System.Int32`**.
* C# `string` = CTS `System.String`
* C# `bool` = CTS `System.Boolean`
* C# `double` = CTS `System.Double`

### 2. CLS (Common Language Specification - Ortak Dil Spesifikasyonu)
Farklı dillerin birbirlerinin kodlarını hatasız tüketebilmesi için uymaları gereken asgari kurallar bütünüdür.
* **Tuzak Örneği (Büyük/Küçük Harf Farkı):** C# büyük-küçük harfe duyarlıdır (`case-sensitive`). Bir sınıfta hem `Topla()` hem de `topla()` adında iki farklı metot tanımlayabilirsiniz. Fakat VB.NET büyük-küçük harf ayrımı yapmaz (`case-insensitive`). Eğer C#'ta yazdığınız bir kütüphaneyi VB.NET'e sunacaksanız, CLS kuralları gereği yalnızca harf büyüklüğüyle ayrılan isimler kullanamazsınız!

> 🔑 **Ezber:** "`CTS` tiplerin ortak sözlüğüdür (int = System.Int32). `CLS` ise dillerin birbiriyle kavga etmeden anlaşması için uyması gereken ortak protokol kurallarıdır."

---

## 0.5) csc Derleyicisi ve Önemli Parametreleri

Modern projelerde `dotnet build` komutu kullanılsa da, akademik sınavlarda geleneksel C# derleyicisi olan **`csc` (C# Compiler)** parametreleri sıklıkla test edilir:

```bash
# Temel derleme: program.exe oluşturur
csc program.cs

# Çıktı dosyasının adını özelleştirme
csc /out:Hesaplayici.exe program.cs

# DLL (Kütüphane) olarak derleme
csc /t:library Kutuphane.cs  # Kutuphane.dll üretir

# Konsol penceresi açılmayan Windows GUI uygulaması derleme
csc /t:winexe FormUygulamasi.cs

# Pointer ve güvensiz kod kullanımına izin verme
csc /unsafe IsaretciKodu.cs

# Derleme hatalarını bir metin dosyasına raporlama
csc /bugreport:Hatalar.txt program.cs
```

> 🔑 **Ezber:** "`/t:library` DLL yapar, `/t:winexe` konsolsuz form açar, `/unsafe` işaretçi (pointer) kapısını aralar."

---

## 0.6) Sınav ve Mülakat Tuzakları

1. **"C# kodu doğrudan makine diline mi derlenir?"**  
   * **Tuzak:** Evet demek. C# doğrudan makine diline derlenmez! Önce derleyici (`csc`) tarafından `IL` koduna çevrilir; program çalıştırıldığında `CLR` içindeki `JIT` tarafından makine diline dönüştürülür.
2. **"Managed Code ile Unmanaged Code arasındaki fark nedir?"**  
   * **Tuzak:** Yalnızca dil farkı zannetmek. Managed Code, CLR'ın çöp toplayıcısı (GC) ve tip güvenlik duvarı altında çalışan koddur. C/C++ doğrudan işletim sistemi üzerinde unmanaged çalışır.
3. **"CTS ile CLS aynı şey midir?"**  
   * **Tuzak:** Birbirine karıştırmak. CTS tipleri tanımlar (hangi tipler var), CLS ise kuralları koyar (diller arası iletişimde nelere izin verilir).

---

## 🔑 Hızlı Tekrar: Altın Ezber Cümleleri (Kopya Kağıdı)

- **.NET & CLR:** "C# kodunu çalıştıran sanal motor CLR'dır; CLR'ın yönettiği bellek güvenli koda Managed Code denir."
- **Derleme Akışı:** "C# derlenince makine kodu değil `IL` (ara dil) ve `Metadata` çıkar. `JIT` bu IL'i çalışma anında makine diline çevirir."
- **JIT Türleri:** "`Normal JIT` çağırdıkça derler ve saklar, `Pre-JIT` her şeyi baştan derler, `Eco-JIT` hafıza bitince derlediğini siler."
- **CTS vs CLS:** "`CTS` ortak tipleri tanımlar (`int` = `System.Int32`), `CLS` diller arası iletişimin kurallarını koyar."
- **csc Parametreleri:** "`/t:library` DLL yapar, `/unsafe` pointer açar."

