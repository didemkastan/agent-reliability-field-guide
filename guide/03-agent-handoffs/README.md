# Agentlar Arası Görev Devri

[🇬🇧 English](README_EN.md)

## Problem

Bir agent kendi görevini doğru tamamlamış olsa bile işi devralan agent gerekli bilgileri almazsa sonraki adımda hata yapabilir.

Örneğin Agent A bir dosyayı değiştirirken korunması gereken önemli bir kuralı biliyor olabilir. Bu bilgi görev devri sırasında Agent B'ye aktarılmazsa Agent B önceki çalışmayı farkında olmadan bozabilir.

## Temel kural

> **Görev devrinde neyin değiştiği, nelerin korunması gerektiği, hangi kontrollerin yapıldığı ve hangi kontrollerin henüz yapılmadığı açıkça belirtilmelidir.**

## Handoff Receipt (Görev Devri Kaydı)

Görev devrinde aşağıdaki bilgiler kullanılabilir:

- **TASK_ID (Görev kimliği)**
- **INPUT_VERSION (Başlangıç sürümü)**
- **OUTPUT_VERSION (Çıktı sürümü)**
- **CHANGED (Değiştirilenler)**
- **PRESERVE (Korunacaklar)**
- **VERIFIED (Doğrulananlar)**
- **SKIPPED_CHECKS (Yapılmayan kontroller)**
- **NEXT_AGENT (Sonraki agent)**

## Kontrol edilen kapsam belirtilmelidir

Bir agentın “her şeyi kontrol ettim” demesi tek başına yeterli değildir. Kaç öğenin kontrol edilmesi gerektiği ve bunların kaçının gerçekten kontrol edildiği kaydedilebilir.

Bunun için EXPECTED_ITEMS (Beklenen öğeler), OBSERVED_ITEMS (Görülen öğeler), CHECKED_ITEMS (Kontrol edilen öğeler), SKIPPED_ITEMS (Atlanan öğeler) ve COVERAGE_STATUS (Kontrol durumu) kullanılabilir.

Örneğin görev kapsamında 12 dosya varsa ancak 10 dosya incelenmişse kalan 2 dosyanın kontrol edilmediği açıkça belirtilmelidir.

## Örnek

Agent A üç hayali ayar dosyasını değiştiriyor ancak çalışma ortamındaki bir eksiklik nedeniyle yalnızca iki dosyayı test edebiliyor.

Görev devrinde şu bilgiler yer alıyor:

- 3 dosya değiştirildi.
- 2 dosya test edildi.
- 1 dosya test edilemedi.
- Test edilememe nedeni kaydedildi.

Böylece Agent B hangi işlemlerin tamamlandığını ve hangi kontrolün hâlâ yapılması gerektiğini doğrudan görebilir.

## Nasıl uygulanır?

Bir agent işi başka bir agenta devredecekse görev talimatına kısa bir devir kuralı eklenebilir.

Örneğin:

> **Görevin sonunda bir görev devri kaydı oluştur. Başlangıç ve çıktı sürümünü, yaptığın değişiklikleri, korunması gereken noktaları, tamamladığın kontrolleri, yapamadığın kontrolleri ve sonraki agentın yapması gereken işi açıkça belirt. Kontrol etmediğin bir öğeyi doğrulanmış olarak gösterme.**

Görevde kontrol edilmesi gereken dosya, kayıt veya test sayısı belliyse beklenen, kontrol edilen ve atlanan öğe sayıları da istenebilir.

Daha otomatik çoklu-agent iş akışlarında bu bilgiler serbest metin yerine belirli alanlarda saklanarak doğrudan sonraki agenta aktarılabilir.

## Ne zaman kullanılır?

Bir iş farklı agentlar, oturumlar, bilgisayarlar veya kişiler arasında devrediliyorsa görev devri kaydı kullanmak faydalıdır.

Tek adımda tamamlanan ve başka bir kişi veya agenta aktarılmayacak küçük görevlerde ayrıntılı bir kayıt gerekmeyebilir.
