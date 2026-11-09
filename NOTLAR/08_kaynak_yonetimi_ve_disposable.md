# Bölüm 8: Kaynak Yönetimi, Yıkıcılar ve IDisposable
*(Managed vs Unmanaged kaynaklar, GC, Destructors ~Sinif(), IDisposable, SuppressFinalize ve using)*

.NET platformunda bellek yönetimi büyük ölçüde CLR bünyesindeki **Garbage Collector (Çöp Toplayıcı - GC)** tarafından otomatik olarak yürütülür. Ancak işletim sistemi tanıtıcıları (dosyalar, ağ soketleri, veritabanı bağlantıları) gibi dış kaynaklar söz konusu olduğunda, yazılımcının kaynakları kontrollü ve anında serbest bırakması hayati önem taşır.

---

## 8.1) Yönetilen (Managed) ve Yönetilmeyen (Unmanaged) Kaynaklar

* **Yönetilen Kaynaklar (Managed Resources):** Tamamen CLR kontrolünde olan ve Heap belleğinde tahsis edilen standart C# nesneleridir (`string`, `List<T>`, sınıflar vb.). Bellekleri GC tarafından takip edilir ve işleri bittiğinde otomatik olarak temizlenir.
* **Yönetilmeyen Kaynaklar (Unmanaged Resources):** İşletim sistemi veya donanım tarafından doğrudan tahsis edilen yapılardır:
  * Dosya tanıtıcıları (File Handles)
  * Ağ ve soket bağlantıları (Network Sockets)
  * Veritabanı bağlantıları (Database Connections)
  * İşletim sistemi pencereleri ve font çizim tutamaçları (GDI+ Handles)
  * Unmanaged C/C++ DLL çağrıları (P/Invoke)

> ⚠️ **Temel Sorun:** Garbage Collector yalnızca Heap belleğini bilir; işletim sisteminin açık tuttuğu bir dosyanın veya ağ soketinin ne zaman kapatılması gerektiğini **bilemez**. Bu kaynakların serbest bırakılması tamamen yazılımcının sorumluluğundadır.

---

## 8.2) Yıkıcı Metotlar (Destructors / Finalizers - `~Sınıf()`)

C# dilinde sözdizimi C++ yıkıcılarına benzeyen özel bir metottur:

```csharp
public class DosyaOkuyucu
{
    // Yıkıcı Metot (Finalizer)
    ~DosyaOkuyucu()
    {
        Console.WriteLine("Yıkıcı metot çalıştı.");
    }
}
```

### Kuralları ve Arka Planı:
1. **İmza Kuralları:** Parametre alamaz, erişim belirteci (`public`, `private` vb.) alamaz, aşırı yüklenemez.
2. **Arka Plan Dönüşümü:** C# derleyicisi bu metodu otomatik olarak `protected override void Finalize()` metoduna dönüştürür.
3. **Non-Deterministic (Belirsiz Zamanlı) Çalışma:** Yıkıcı metot programcı tarafından elle çağrılamaz. Yalnızca GC nesneyi bellekten süpürmeye karar verdiğinde çağrılır.
   * GC'nin ne zaman çalışacağı **öngörülemez**.
   * Uygulama kapanana kadar veya bellek dolana kadar hiç çalışmayabilir!
   * Bu yüzden açık kalan bir dosyayı veya veritabanı bağlantısını kapatmak için **yıkıcı metotlara asla güvenilemez**.
4. **Performans Maliyeti (F-Reachable Kuyruğu):** Yıkıcı metodu olan bir nesne ilk GC turunda bellekten silinemez. Önce "Finalization Kuyruğu"na alınır, bir üst nesle (Gen 1 / Gen 2) terfi eder ve bellekten silinmesi gecikir.

---

## 8.3) `IDisposable` Arayüzü ve `Dispose()` Deseni

Yönetilmeyen kaynakların ihtiyaç bittiği anda **deterministik (anında ve öngörülebilir)** bir şekilde serbest bırakılması için C# standardında **`IDisposable`** arayüzü ve `Dispose()` deseni kullanılır.

### Standart Dispose Deseni (Dispose Pattern)
```csharp
public class GuvenliKaynak : IDisposable
{
    private bool disposed = false; // Kaynağın serbest bırakılıp bırakılmadığını izler

    // 1. Dış dünyadan çağrılan genel metot
    public void Dispose()
    {
        Dispose(true);

        // CLR & GC'ye kritik mesaj:
        // "Bu nesnenin temizliğini ben elle yaptım, nesneyi Finalizer kuyruğuna sokup vakit kaybetme!"
        GC.SuppressFinalize(this);
    }

    // 2. Asıl temizliği yapan sanal metot
    protected virtual void Dispose(bool disposing)
    {
        if (!disposed)
        {
            if (disposing)
            {
                // Yönetilen (managed) nesneleri temizle (örn: içteki başka IDisposable nesneler)
            }

            // Yönetilmeyen (unmanaged) kaynakları serbest bırak (Windows handle, socket, bağlantı kapat vb.)
            Console.WriteLine("Kaynaklar başarıyla serbest bırakıldı.");

            disposed = true;
        }
    }

    // 3. Emniyet Sübabı: Eğer yazılımcı Dispose() çağırmayı unuttuysa, GC devreye girdiğinde son çare çalışır
    ~GuvenliKaynak()
    {
        Dispose(false);
    }
}
```

### `GC.SuppressFinalize(this)` Neden Hayatidir?
Eğer yazılımcı `Dispose()` metodunu çağırarak kaynakları temizlediyse, nesnenin bir de GC tarafından Finalizer kuyruğuna sokulması gereksiz bir performans yüküdür. `GC.SuppressFinalize(this)` çağrısı, CLR'a bu nesnenin yıkıcı metodunu çalıştırmamasını söyler; böylece nesne ilk GC turunda anında bellekten silinir.

---

## 8.4) `using` İfadesi ve Bildirimi (Otomatik Temizlik Garantisi)

Bir nesne `IDisposable` arayüzünü uyguluyorsa, en güvenli kullanım yöntemi **`using`** yapısıdır.

### 1. Klasik `using` Bloğu
```csharp
using (var kaynak = new GuvenliKaynak())
{
    // İşlemler yürütülür...
} // Bloktan çıkıldığı anda kaynak.Dispose() otomatik olarak çağrılır!
```

### 2. Modern `using` Bildirimi (C# 8.0+)
Süslü parantez kalabalığını önler; tanımlandığı skop (metot) sona erdiğinde `Dispose()` çalıştırılır:
```csharp
public void VeriOku()
{
    using var kaynak = new GuvenliKaynak();
    // İşlemler yürütülür...
    // Metot bittiğinde otomatik kaynak.Dispose() çağrılır.
}
```

### Arka Plandaki Gerçek: `try - finally` Garantisi
Derleyici her `using` yapısını arka planda gizlice bir **`try - finally`** bloğuna dönüştürür. Kodun içinde bir hata (Exception) patlasa dahi, `finally` bloğu güvencesiyle `Dispose()` metodunun çalışması **kesin olarak garanti edilir**:

```csharp
// Derleyicinin arka planda ürettiği IL kodu:
GuvenliKaynak kaynak = new GuvenliKaynak();
try
{
    // İşlemler...
}
finally
{
    if (kaynak != null)
        ((IDisposable)kaynak).Dispose();
}
```

---

## 8.5) Sınav ve Mülakat Tuzakları

1. **`Dispose()` Metodunun Belleği Sildiğini Sanma Yanılgısı:**
   `Dispose()` metodu bir nesneyi Heap belleğinden (RAM'den) **silmez**! Yalnızca o nesnenin işletim sisteminden kiraladığı dış kaynakları (dosya, soket, DB connection) iade eder. Nesnenin kendisi hala Heap'tedir ve RAM'den onu yalnızca GC silebilir.
2. **Yıkıcı Metodun Ne Zaman Çalışacağını Bilme İddiası:**
   Sınavlarda "Yıkıcı metot nesnenin referansı null yapılınca hemen çalışır mı?" sorusu klasik bir tuzaktır. **HAYIR!** GC'nin ne zaman çalışacağı kesinlikle non-deterministiktir.
3. **`using` İle Sadece `IDisposable` Tiplerin Kullanılabilmesi:**
   `IDisposable` arayüzünü uygulamayan bir sınıf `using` bloğu veya bildirimi içine konulamaz (`CS1674` derleme hatası).
4. **`GC.SuppressFinalize(this)` Eksikliği:**
   `Dispose()` içinde `GC.SuppressFinalize(this)` çağrılmazsa, nesne `Dispose` edilmiş olsa dahi GC tarafından tekrar Finalize kuyruğuna alınarak sisteme çift iş yükü bindirilir.

---

## 8.6) Altın Ezber Cümleleri

> 🔑 **1. Managed vs Unmanaged:** "Managed kaynakları GC kendi temizler; işletim sistemi tutamaçları gibi unmanaged kaynakları temizlemek yazılımcının görevidir."
> 
> 🔑 **2. Yıkıcı Metot (Finalizer):** "Yıkıcı metot `~Sinif()` parametresizdir; ne zaman çalışacağını yalnızca GC bilir, kritik kaynak temizliğinde asla ona güvenilmez."
> 
> 🔑 **3. Deterministik Temizlik:** "`IDisposable` ve `Dispose()` kaynakları anında ve kesin olarak serbest bırakmanın C# standart yoludur."
> 
> 🔑 **4. SuppressFinalize:** "`GC.SuppressFinalize(this)`, 'ben kaynakları temizledim, finalizer kuyruğuna sokup sistemi yorma' talimatıdır."
> 
> 🔑 **5. `using` Garantisi:** "`using` arka planda `try-finally`'e derlenir; kod hata fırlatsa dahi `Dispose()` çağrısını garanti eder."
> 
> 🔑 **6. Dispose vs RAM:** "`Dispose()` dış kaynakları (bağlantı, dosya) kapatır; nesneyi RAM'den silen ise yalnızca Garbage Collector'dır!"
