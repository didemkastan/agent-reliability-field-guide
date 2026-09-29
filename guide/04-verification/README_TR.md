# Verification (Doğrulama)

[🇬🇧 English](README.md)

## Doğrulama neden önemlidir?

Bir agentın “tamamlandı” demesi, işlemin gerçekten doğru sonuç verdiğini kanıtlamaz. Bu yalnızca agentın kendi değerlendirmesidir.

Bu nedenle dört bilgiyi birbirinden ayırmak faydalıdır:

- **OBSERVED (Gözlemlenen)** — Araçta, dosyada, testte veya sistem sonucunda doğrudan görülen bilgi.
- **INTERPRETED (Yorumlanan)** — Agentın gördüğü bilgiden çıkardığı anlam.
- **ACTION (Yapılan işlem)** — Değiştirilen veya çalıştırılan işlem.
- **VERIFIED (Doğrulanan)** — İşlemden sonra sonucun gerçekten oluşup oluşmadığını kontrol eden kanıt.

## Sentetik örnek

Bir agent, hayali bir özelliği açmak için ayar dosyasını değiştiriyor.

- **OBSERVED (Gözlemlenen):** Dosyada `feature_enabled: false` değeri var.
- **INTERPRETED (Yorumlanan):** Bu ayar özelliğin kapalı olduğunu gösteriyor.
- **ACTION (Yapılan işlem):** Değer `true` olarak değiştiriliyor.
- **VERIFIED (Doğrulanan):** Dosya yeniden okunuyor ve ilgili test başarıyla çalıştırılıyor.

Dosyayı değiştirmek ile değişikliğin doğru çalıştığını doğrulamak aynı işlem değildir. Dosyanın başarıyla kaydedilmesi, özelliğin çalıştığını tek başına göstermez.

## Artifact (Çıktı) ve environment (çalışma ortamı) birlikte kontrol edilmelidir

Aynı dosya farklı ortamlarda farklı sonuç verebilir. Kullanılan çalışma sürümü, işletim sistemi, bağımlılıklar veya ayarlar sonucu etkileyebilir.

Önemli kontrollerde şu bilgiler kaydedilebilir:

- hangi dosyanın veya sürümün test edildiği;
- testin hangi ortamda çalıştırıldığı;
- hangi kontrolün yapıldığı;
- doğrulamanın hangi kaynağa dayandığı.

## RESULT_VERIFIED (Sonuç doğrulandı) ve PATH_VERIFIED (İzlenen yol doğrulandı)

İki ayrı soru sorulur:

1. **RESULT_VERIFIED (Sonuç doğrulandı)** — Beklenen sonuç oluştu mu?
2. **PATH_VERIFIED (İzlenen yol doğrulandı)** — Bu sonuca beklenen ve kabul edilen adımlar izlenerek mi ulaşıldı?

Örneğin oluşturulan bir dosya doğru görünebilir. Ancak yapılması gereken test atlandıysa sonuç görünürde doğru olsa bile doğrulama süreci tamamlanmış sayılmaz.

## Ne ters gidebilir?

Bu bilgiler birbirine karışırsa agentın yorumu zamanla kanıt gibi kabul edilebilir. Böylece yapılmamış bir test veya kontrol edilmemiş bir varsayım gözden kaçabilir.

## Temel kural

> **“Tamamlandı” ifadesini durum bilgisi olarak kabul et. Doğrulamayı ise görülebilen bir sonuca veya teste dayanan ayrı bir kontrol olarak yap.**

## Ne zaman kullanılır?

Bu ayrım özellikle agent dosya değiştirdiğinde, test çalıştırdığında, çıktı oluşturduğunda, başka bir araç kullandığında veya işi başka bir agenta devrettiğinde önemlidir.

Düşük riskli ve geçici denemelerde daha kısa bir kontrol yeterli olabilir.