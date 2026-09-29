# Verification (Doğrulama)

[🇬🇧 English](README_EN.md)

## Doğrulama neden önemlidir?

Bir agentın “tamamlandı” demesi, beklenen sonucun gerçekten oluştuğunu tek başına göstermez. İşlem tamamlandıktan sonra sonucun ayrıca kontrol edilmesi gerekir.

Bu nedenle aşağıdaki dört bilgi ayrı tutulabilir:

- **OBSERVED (Gözlemlenen)** — Dosyada, testte, kullanılan araçta veya sistem sonucunda doğrudan görülen bilgi.
- **INTERPRETED (Yorumlanan)** — Gözlemlenen bilginin ne anlama geldiğine ilişkin değerlendirme.
- **ACTION (Yapılan işlem)** — Gerçekleştirilen değişiklik veya çalıştırılan işlem.
- **VERIFIED (Doğrulanan)** — Yapılan işlemden sonra beklenen sonucun oluştuğunu gösteren kontrol sonucu.

## Sentetik örnek

Bir agent, hayali bir özelliği etkinleştirmek için ayar dosyasını değiştiriyor.

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

## Ne zaman kullanılır?

Bu ayrım özellikle agent dosya değiştirdiğinde, test çalıştırdığında, çıktı oluşturduğunda, başka bir araç kullandığında veya işi başka bir agenta devrettiğinde önemlidir.

Düşük riskli ve kolayca geri alınabilen denemelerde daha kısa bir kontrol yeterli olabilir.
