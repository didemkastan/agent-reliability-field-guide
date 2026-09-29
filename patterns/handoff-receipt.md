[🇬🇧 English](handoff-receipt_EN.md)

# Handoff Receipt (Görev Devri Kayıt Şablonu)

Bu şablon, bir görevin bir agenttan başka bir agenta veya kişiye devredilirken mevcut durumun, yapılan değişikliklerin ve kalan kontrollerin kaybolmadan aktarılması için kullanılır.

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

## Kullanıma hazır şablon

```text
HANDOFF RECEIPT (GÖREV DEVRİ KAYDI)

TASK_ID (Görev kimliği):
INPUT_VERSION (Başlangıç sürümü):
OUTPUT_VERSION (Çıktı sürümü):

CHANGED (Değiştirilenler):
PRESERVE (Korunacaklar):

VERIFIED (Doğrulananlar):
SKIPPED_CHECKS (Yapılmayan kontroller):

EXPECTED_ITEMS (Beklenen öğeler):
OBSERVED_ITEMS (Gözlemlenen öğeler):
CHECKED_ITEMS (Kontrol edilen öğeler):
SKIPPED_ITEMS (Atlanan öğeler):
COVERAGE_STATUS (Kapsam durumu):

NEXT_AGENT (Sonraki agent):
NEXT_ACTION (Sonraki işlem):
```

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
