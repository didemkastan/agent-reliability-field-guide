[🇬🇧 English](handoff-receipt_EN.md)

# Handoff Receipt (Görev Devri Kayıt Şablonu)

Bu şablon, bir görevin bir agenttan başka bir agenta veya kişiye devredilirken mevcut durumun, yapılan değişikliklerin ve kalan kontrollerin kaybolmadan aktarılması için kullanılır.

> **Bu şablon belirli bir ürün veya platforma ait resmî bir standart değildir. Agentlar arası görev devrinde sürüm, değişiklik, doğrulama, kapsam ve sonraki adım bilgilerinin kaybolmasını önlemek amacıyla bu rehberde kullanılan ilkelerin bir araya getirilmesiyle hazırlanmıştır.**

## Kim doldurur?

Görevi devreden agent veya kişi doldurur. Kayıt, yapılan işi yalnızca özetlemek için değil, görevi devralacak tarafın nereden devam edeceğini açıkça göstermek için hazırlanır.

## Ne zaman kullanılır?

Bir görev başka bir agenta veya kişiye devredileceğinde, farklı bir oturumda devam edeceğinde ya da yapılan işin kalan kontrolleri başka bir tarafça tamamlanacağında kullanılabilir.

Devreden taraf önce kendi yapabildiği kontrolleri tamamlar, ardından görev devri kaydını oluşturur.

## Nereye konur?

Kullanım şekli çalışma ortamına göre değişebilir:

- Tek seferlik bir görevde doğrudan chat çıktısı olarak kullanılabilir.
- Ortak bir proje alanında `HANDOFF.md` gibi bir dosyada tutulabilir.
- Otomatik agent iş akışında aynı alanlar JSON veya görev kaydı olarak aktarılabilir.

Görev devri kaydının bulunması sonraki agentı kendiliğinden başlatmaz. Otomatik geçiş isteniyorsa ayrıca bir tetikleyici ve orkestrasyon mekanizması gerekir.

## Kullanıma hazır örnek şablon

```text
HANDOFF RECEIPT

TASK_ID:
INPUT_VERSION:
OUTPUT_VERSION:

CHANGED:
PRESERVE:

VERIFIED:
SKIPPED_CHECKS:

EXPECTED_ITEMS:
OBSERVED_ITEMS:
CHECKED_ITEMS:
SKIPPED_ITEMS:
COVERAGE_STATUS:

NEXT_AGENT:
NEXT_ACTION:
```

## Doldurulmuş örnek

Aşağıda şablonun nasıl doldurulabileceğini gösteren basit bir örnek yer alır:

```text
HANDOFF RECEIPT

TASK_ID: DOC-014
INPUT_VERSION: v1.2
OUTPUT_VERSION: v1.3

CHANGED:
- README.md içindeki kurulum komutu düzeltildi.
- Eksik yapılandırma örneği eklendi.

PRESERVE:
- Mevcut klasör yapısı değiştirilmemeli.
- Kullanılan komut adları korunmalı.

VERIFIED:
- README.md yeniden açılarak değişiklikler kontrol edildi.
- Kurulum komutu başarıyla çalıştırıldı.

SKIPPED_CHECKS:
- Windows ortamında test yapılmadı.

EXPECTED_ITEMS: 3
OBSERVED_ITEMS: 3
CHECKED_ITEMS: 2
SKIPPED_ITEMS: 1
COVERAGE_STATUS: 2/3 kontrol edildi

NEXT_AGENT: Agent B
NEXT_ACTION: Kurulum komutunu Windows ortamında çalıştır ve sonucu kaydet.
```

Bu örnekte hem tamamlanan işler hem de eksik kalan kontrol görünür durumdadır. Görevi devralan agent, yapılmamış bir kontrolü doğrulanmış kabul etmeden nereden devam edeceğini görebilir.

## Alanlar ne anlama gelir?

- **TASK_ID:** Devredilen görevin kimliği.
- **INPUT_VERSION:** Çalışmaya başlanırken esas alınan sürüm.
- **OUTPUT_VERSION:** Çalışma sonunda oluşan sürüm.
- **CHANGED:** Yapılan değişiklikler.
- **PRESERVE:** Sonraki agentın değiştirmemesi veya koruması gereken noktalar.
- **VERIFIED:** Gerçekten kontrol edilmiş ve doğrulanmış sonuçlar.
- **SKIPPED_CHECKS:** Yapılamayan veya bilinçli olarak atlanan kontroller.
- **EXPECTED_ITEMS:** Kontrol edilmesi beklenen toplam öğeler.
- **OBSERVED_ITEMS:** Çalışma sırasında gerçekten görülen öğeler.
- **CHECKED_ITEMS:** Gerçekten kontrol edilen öğeler.
- **SKIPPED_ITEMS:** Kontrol edilmeyen öğeler.
- **COVERAGE_STATUS:** Beklenen kapsamın ne kadarının kontrol edildiğini gösteren durum.
- **NEXT_AGENT:** Görevi devralması beklenen agent veya taraf.
- **NEXT_ACTION:** Görevi devralan tarafın yapması gereken ilk işlem.

## Sonraki agent ne yapar?

Görevi devralan agent kaydı okur ancak içindeki bilgileri doğrulamadan doğru kabul etmez. İşleme başlamadan önce ilgili dosya ve sürümlerin hâlâ güncel olduğunu kontrol eder, **PRESERVE (Korunacaklar)** alanındaki kısıtları dikkate alır ve özellikle **SKIPPED_CHECKS (Yapılmayan kontroller)** ile **NEXT_ACTION (Sonraki işlem)** alanlarından devam eder.

> **Görev devri kaydı, yapılan işi görünür hale getirir; doğrulamanın yerine geçmez.**

## Gizlilik

Herkese açık kayıtlarda şirket bilgileri, müşteri verileri, kimlik bilgileri, erişim anahtarları, özel kodlar, gizli loglar veya başka hassas bilgiler kullanılmamalıdır.
