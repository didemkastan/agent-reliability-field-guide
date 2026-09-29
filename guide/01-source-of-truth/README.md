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
5. Değişiklikten etkilenecek ve geri dönüş için gerekli dosyaların geri yüklenebilir bir sürümünün bulunduğunu doğrula.
6. Dosya bu sırada değişmişse işlemi durdur, güncel sürümü yeniden incele ve değişikliği buna göre hazırla.

## Rollback Point (Geri Dönüş Noktası)

Yedekleme, her dosyanın gereksiz yere kopyalanması anlamına gelmemelidir. Değişiklikten doğrudan etkilenecek ve bir sorun durumunda eski duruma dönmek için gerekli olan dosyalar korunmalıdır.

**Git kullanılan projelerde:** Değişiklikten önce çalışma alanının durumunu kontrol et. Geri dönülmesi gereken mevcut sürümün commit geçmişinde bulunduğundan emin ol. Henüz kaydedilmemiş önemli değişiklikler varsa bunları güvenli bir commit, branch (dal) veya uygun başka bir Git yöntemiyle korumadan üzerine yazma.

**Git kullanılmayan projelerde:** Değiştirilecek dosyanın mevcut sürümünü ayrı bir yedek klasörüne kopyala. Dosya adında tarih/saat veya sürüm bilgisi kullanarak hangi yedeğin hangi değişiklikten önce alındığını anlaşılır hale getir. Örneğin: `settings.before-change.2026-09-29.yaml`.

Yedeğin varlığı tek başına yeterli değildir. Gerektiğinde hangi dosyanın geri yükleneceği anlaşılabilmeli ve yedek, yapılan değişiklikten etkilenmeyen ayrı bir konumda tutulmalıdır.

> **Amaç mümkün olduğunca çok yedek üretmek değil, riskli bir değişiklikten önce gerekli geri dönüş noktasını oluşturmaktır.**

## Örnek

İki agent hayali bir `settings.yaml` dosyasında çalışıyor.

Agent A dosyanın `abc123` hash değerine sahip sürümünü açıyor. Bu sırada Agent B dosyayı güncelliyor ve hash değeri `def456` oluyor. Agent A değişikliğini kaydetmeden önce dosyayı tekrar kontrol ettiğinde sürümün değiştiğini görüyor.

**Doğru işlem:** Güncel dosyayı yeniden incelemek ve değişikliği yeni sürüme göre hazırlamak.

**Riskli işlem:** Eski `abc123` sürümüne göre hazırlanan değişikliği doğrudan yeni `def456` sürümünün üzerine yazmak.

## Nasıl uygulanır?

Bu yöntem, bir AI agent ile dosya veya kod üzerinde çalışırken görev talimatına eklenebilir. Özellikle aynı dosyanın başka bir agent, kişi veya otomasyon tarafından da değiştirilebildiği çalışmalarda kullanılabilir.

Başlangıç için agenta şu kontrol kuralı verilebilir:

> **Değişiklik yapmadan önce dosyanın güncel sürümünü ve hash bilgisini kontrol et. Değişiklikten etkilenecek ve geri dönüş için gerekli dosyaların geri yüklenebilir bir sürümünün bulunduğunu doğrula. Dosya sistemi veya Git erişimin varsa gerekli geri dönüş noktasını kendin oluştur ve oluşturulduğunu doğrula. Bu erişimin yoksa yedek alındığını varsayma; kullanıcıdan geri dönüş noktası oluşturmasını iste veya değişiklik işlemini durdur. Değişikliği kaydetmeden hemen önce dosyayı tekrar kontrol et. Sürüm veya hash değişmişse dosyanın üzerine yazma; güncel sürümü yeniden incele ve değişikliği buna göre hazırla.**

Bu talimat ChatGPT, Claude, Codex veya benzeri dosya ve kod üzerinde işlem yapabilen agentlarla çalışırken görev talimatının bir parçası olarak kullanılabilir. Agentın dosya sistemi veya Git üzerinde işlem yapma yetkisi varsa yedek veya geri dönüş noktası oluşturma adımı da agent tarafından gerçekleştirilebilir. Yetkisi yoksa agent bu adımı yapılmış kabul etmemelidir.

Daha otomatik iş akışlarında aynı kontrol yalnızca prompt (talimat) ile bırakılmamalıdır. Dosyanın hash veya sürüm bilgisi işlem başlamadan önce ve değişiklik kaydedilmeden hemen önce sistem tarafından karşılaştırılabilir. Değer değişmişse işlem durdurularak agentın güncel dosyayı yeniden okuması sağlanabilir.

## Ne zaman kullanılır?

Aynı dosya birden fazla kişi, agent, branch (dal) veya otomasyon tarafından değiştirilebiliyorsa ya da dosya farklı çalışma ortamlarında güncellenebiliyorsa bu kontrol önemlidir.

Tek kişinin çalıştığı ve yapılan değişikliklerin kolayca geri alınabildiği küçük denemelerde daha kısa bir kontrol yeterli olabilir.
