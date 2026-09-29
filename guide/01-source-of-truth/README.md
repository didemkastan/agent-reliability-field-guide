# Gerçeğin Kaynağı

[🇬🇧 English](README_EN.md)

## Problem

Bir agent doğru bir işlem hazırlasa bile güncel olmayan bir dosya sürümü üzerinde çalışıyorsa sonuç hatalı olabilir.

Agent A'nın 4. sürüm üzerinde çalıştığını düşünün. Bu sırada Agent B dosyayı güncelleyerek 5. sürümü oluşturuyor. Agent A bu değişikliği fark etmezse işlemini artık güncel olmayan 4. sürüme göre tamamlayabilir. Hazırladığı değişiklik doğru olsa bile eski sürüme dayandığı için 5. sürümde yapılan yeni değişiklikleri bozabilir veya veri kaybına yol açabilir.

## Temel kural

> **Bir dosyada değişiklik yapmadan önce hangi sürümün esas alındığını belirle. Değişikliği kaydetmeden hemen önce bu sürümün hâlâ güncel olduğunu tekrar kontrol et.**

## Pratik yöntem

Önemli dosyalarda SOURCE (Kaynak), VERSION (Sürüm), STATUS (Durum), VERIFIED_BY (Doğrulayan) ve HASH (Dosyanın belirli bir sürümünü tanımlayan dijital değer) gibi bilgiler kaydedilebilir.

Değişiklik yaparken:

1. Dosyanın güncel sürümünü kontrol et.
2. Sürüm veya hash bilgisini kaydet.
3. Değişikliği hazırla.
4. Kaydetmeden hemen önce dosyanın sürümünü veya hash değerini yeniden kontrol et.
5. Dosya bu sırada değişmişse işlemi durdur, güncel sürümü yeniden incele ve değişikliği buna göre hazırla.

## Örnek

İki agent hayali bir `settings.yaml` dosyasında çalışıyor.

Agent A dosyanın `abc123` hash değerine sahip sürümünü açıyor. Bu sırada Agent B dosyayı güncelliyor ve hash değeri `def456` oluyor. Agent A değişikliğini kaydetmeden önce dosyayı tekrar kontrol ettiğinde sürümün değiştiğini görüyor.

**Doğru işlem:** Güncel dosyayı yeniden incelemek ve değişikliği yeni sürüme göre hazırlamak.

**Riskli işlem:** Eski `abc123` sürümüne göre hazırlanan değişikliği doğrudan yeni `def456` sürümünün üzerine yazmak.

## Nasıl uygulanır?

Bu yöntem, bir AI agent ile dosya veya kod üzerinde çalışırken görev talimatına eklenebilir. Özellikle aynı dosyanın başka bir agent, kişi veya otomasyon tarafından da değiştirilebildiği çalışmalarda kullanılabilir.

Başlangıç için agenta şu kontrol kuralı verilebilir:

> **Değişiklik yapmadan önce dosyanın güncel sürümünü kontrol et ve sürüm veya hash bilgisini kaydet. Değişikliği kaydetmeden hemen önce dosyayı tekrar kontrol et. Sürüm veya hash değişmişse dosyanın üzerine yazma; güncel sürümü yeniden incele ve değişikliği buna göre hazırla.**

Bu talimat ChatGPT, Claude, Codex veya benzeri dosya ve kod üzerinde işlem yapabilen agentlarla çalışırken görev talimatının bir parçası olarak kullanılabilir.

Daha otomatik iş akışlarında aynı kontrol yalnızca prompt (talimat) ile bırakılmamalıdır. Dosyanın hash veya sürüm bilgisi işlem başlamadan önce ve değişiklik kaydedilmeden hemen önce sistem tarafından karşılaştırılabilir. Değer değişmişse işlem durdurularak agentın güncel dosyayı yeniden okuması sağlanabilir.

## Ne zaman kullanılır?

Aynı dosya üzerinde birden fazla kişi, agent, branch (dal), otomasyon veya bilgisayar çalışabiliyorsa bu kontrol önemlidir.

Tek kişinin çalıştığı ve yapılan değişikliklerin kolayca geri alınabildiği küçük denemelerde daha kısa bir kontrol yeterli olabilir.
