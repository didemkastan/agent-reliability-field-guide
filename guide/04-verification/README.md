# Verification (Doğrulama)

[🇬🇧 English](README_EN.md)

## Doğrulama neden önemlidir?

Bir agentın “tamamlandı” demesi, beklenen sonucun gerçekten oluştuğunu tek başına göstermez. İşlem tamamlandıktan sonra sonucun ayrıca kontrol edilmesi gerekir.

Bu nedenle aşağıdaki dört bilgi ayrı tutulabilir:

- **OBSERVED (Gözlemlenen)** — Dosyada, testte, kullanılan araçta veya sistem sonucunda doğrudan görülen bilgi.
- **INTERPRETED (Yorumlanan)** — Gözlemlenen bilginin ne anlama geldiğine ilişkin değerlendirme.
- **ACTION (Yapılan işlem)** — Gerçekleştirilen değişiklik veya çalıştırılan işlem.
- **VERIFIED (Doğrulanan)** — Yapılan işlemden sonra beklenen sonucun oluştuğunu gösteren kontrol sonucu.

## Örnek

Bir agent, bir özelliği etkinleştirmek için ayar dosyasını değiştiriyor.

- **OBSERVED (Gözlemlenen):** Dosyada `feature_enabled: false` değeri bulunuyor.
- **INTERPRETED (Yorumlanan):** Bu değer özelliğin kapalı olduğunu gösteriyor.
- **ACTION (Yapılan işlem):** Değer `true` olarak değiştiriliyor.
- **VERIFIED (Doğrulanan):** Dosya yeniden kontrol ediliyor ve ilgili test başarıyla tamamlanıyor.

Dosyanın başarıyla kaydedilmesi yalnızca değişikliğin dosyaya yazıldığını gösterir. Özelliğin beklendiği gibi çalıştığını göstermek için ilgili testin veya kontrolün de yapılması gerekir.

## Artifact (Çıktı) ve Environment (Çalışma ortamı) birlikte kontrol edilmelidir

Aynı dosya farklı çalışma ortamlarında farklı sonuç verebilir. Kullanılan yazılım sürümü, işletim sistemi, bağımlılıklar veya yapılandırma ayarları sonucu etkileyebilir.

Önemli kontrollerde şu bilgiler kaydedilebilir:

- test edilen dosya veya sürüm;
- testin çalıştırıldığı ortam;
- yapılan kontrol veya test;
- doğrulama sonucunun kaynağı.

## RESULT_VERIFIED (Sonuç doğrulandı) ve PATH_VERIFIED (İzlenen yol doğrulandı)

Doğrulama sırasında iki ayrı konu kontrol edilebilir:

1. **RESULT_VERIFIED (Sonuç doğrulandı)** — Beklenen sonuç oluştu mu?
2. **PATH_VERIFIED (İzlenen yol doğrulandı)** — Sonuca ulaşırken yapılması gereken adımlar ve kontroller tamamlandı mı?

Örneğin oluşturulan bir dosya ilk bakışta doğru görünebilir. Ancak süreçte çalıştırılması gereken bir test atlandıysa dosyanın görünümü doğru olsa bile gerekli kontroller tamamlanmamıştır.

## Ne ters gidebilir?

Gözlem, yorum ve doğrulama birbirinden ayrılmazsa bir varsayım zamanla doğrulanmış bilgi gibi kabul edilebilir. Bu durumda yapılmamış bir test veya kontrol edilmemiş bir ayrıntı fark edilmeyebilir.

## Temel kural

> **“Tamamlandı” bilgisi işlemin bittiğini gösterir. Sonucun doğru olduğunu kabul etmek için ayrıca doğrulama yapılmalıdır.**

## Nasıl uygulanır?

Bu doğrulama kuralının nereye yazılacağı, agentla nasıl çalışıldığına bağlıdır. Her durumda aynı promptu her mesajda tekrar yazmak gerekmez.

### 1. Tek bir görev için chat içinde

Agentla yalnızca belirli bir görev üzerinde çalışılıyorsa talimat doğrudan görevin verildiği chat mesajına eklenebilir. Özellikle dosya değişikliği, test, build veya başka bir doğrulama gerektiren işlem istenirken kullanılması uygundur.

Örneğin:

> **Bu görevde OBSERVED (Gözlemlenen), INTERPRETED (Yorumlanan), ACTION (Yapılan işlem) ve VERIFIED (Doğrulanan) bilgilerini ayrı ayrı yaz. Yapılan işlemi tek başına başarı kanıtı olarak kullanma. VERIFIED alanında sonucu doğrulayan test, dosya kontrolü, komut sonucu veya başka bir kanıtı belirt. Doğrulama yapılmadıysa bunu açıkça yaz.**

Bu kullanımda ayrıca bir dosya oluşturmak zorunlu değildir. Talimat yalnızca o görev için chat içinde verilebilir.

### 2. Aynı projede sürekli kullanılacaksa

Aynı doğrulama kuralının projedeki birçok görevde uygulanması isteniyorsa kural, kullanılan agentın gerçekten okuyacağı kalıcı proje talimatına eklenebilir.

Bu dosyanın adı kullanılan araca göre değişebilir. Örneğin bazı araçlar `AGENTS.md` veya kendilerine ait proje talimat dosyalarını okuyabilir. Kural normal bir `README.md` içine de yazılabilir ancak **agentın README dosyasını her görevde otomatik olarak okuyacağı varsayılmamalıdır**. Önce kullanılan aracın hangi talimat dosyasını otomatik okuduğu kontrol edilmelidir.

Bu yöntemde kullanıcı her görevde aynı promptu yeniden yazmak yerine görevi verir. Agent proje talimatını okuyarak doğrulama kuralını uygular.

### 3. Otomatik agent iş akışında

Bir workflow veya orchestrator (orkestratör) agentı otomatik olarak çalıştırıyorsa doğrulama kuralı agenta gönderilen görev talimatının kalıcı bir parçası yapılabilir.

Sistem agenttan OBSERVED (Gözlemlenen), INTERPRETED (Yorumlanan), ACTION (Yapılan işlem) ve VERIFIED (Doğrulanan) bilgilerini ayrı alanlar halinde üretmesini isteyebilir. Bu bilgiler kullanılan sisteme göre JSON çıktısında, görev kaydında veya agentların erişebildiği ortak bir çalışma dosyasında tutulabilir. Böylece yalnızca “tamamlandı” mesajı alınması başarı olarak kabul edilmez.

Örneğin otomatik sistem aşağıdaki gibi bir çıktı üretebilir:

```json
{
  "OBSERVED": "Ayar dosyasında feature_enabled: false değeri bulundu.",
  "INTERPRETED": "Özellik mevcut ayara göre kapalı.",
  "ACTION": "feature_enabled değeri true olarak değiştirildi.",
  "VERIFIED": "Dosya yeniden kontrol edildi ve ilgili test başarıyla tamamlandı."
}
```

Bu örnekte her bilgi ayrı bir alanda tutulduğu için sistem yapılan işlem ile doğrulama sonucunu birbirinden ayırabilir. `VERIFIED` alanı boşsa veya doğrulamanın yapılmadığını belirtiyorsa görev yalnızca agentın “tamamlandı” demesine dayanarak doğrulanmış kabul edilmemelidir.

### Kullanıcı ne zaman bu kuralı kullanmalı?

Kullanıcı bu kuralı özellikle agenttan **bir şeyi değiştirmesini veya doğru çalıştığını göstermesini istediğinde** devreye almalıdır. Dosya veya kod değişikliği, test, build, yapılandırma değişikliği, araç kullanımı ve başka bir agenta devredilecek işler buna örnektir.

Önemli görevlerde doğrulama kaydına hangi dosya veya sürümün kontrol edildiği ve kontrolün hangi çalışma ortamında yapıldığı da eklenebilir.

## Ne zaman kullanılır?

Bu ayrım özellikle agent dosya değiştirdiğinde, test çalıştırdığında, çıktı oluşturduğunda, başka bir araç kullandığında veya işi başka bir agenta devrettiğinde önemlidir.

Düşük riskli ve kolayca geri alınabilen işlemlerde daha kısa bir kontrol yeterli olabilir. Örneğin bir README dosyasındaki yazım hatasını düzeltirken değişikliğin dosyada doğru göründüğünü yeniden kontrol etmek yeterli olabilir; ayrıca kapsamlı bir test çalıştırmak gerekmeyebilir.
