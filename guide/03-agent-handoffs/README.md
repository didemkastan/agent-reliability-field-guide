# Agentlar Arası Görev Devri

[🇬🇧 English](README_EN.md)

## Problem

Bir agent kendi görevini doğru tamamlamış olsa bile işi devralan agent gerekli bilgileri almazsa sonraki adımda hata yapabilir.

Örneğin Agent A bir dosyayı değiştirirken korunması gereken önemli bir kuralı biliyor olabilir. Bu bilgi görev devri sırasında Agent B'ye aktarılmazsa Agent B önceki çalışmayı farkında olmadan bozabilir.

Buradaki “görev devri”, agentların mutlaka doğrudan birbirleriyle konuştuğu anlamına gelmez. Birçok kullanımda agentlar arasında doğrudan iletişim kanalı yoktur. Devir bilgisi kullanıcı tarafından taşınabilir, ortak bir dosyaya yazılabilir veya bir otomasyon tarafından sonraki agenta aktarılabilir.

## Temel kural

> **Görev devrinde neyin değiştiği, nelerin korunması gerektiği, hangi kontrollerin yapıldığı, hangi kontrollerin henüz yapılmadığı ve sonraki adımın ne olduğu açıkça belirtilmelidir.**

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
- **NEXT_ACTION (Sonraki işlem)**

## Kontrol edilen kapsam belirtilmelidir

Bir agentın “her şeyi kontrol ettim” demesi tek başına yeterli değildir. Kaç öğenin kontrol edilmesi gerektiği ve bunların kaçının gerçekten kontrol edildiği kaydedilebilir.

Bunun için EXPECTED_ITEMS (Beklenen öğeler), OBSERVED_ITEMS (Görülen öğeler), CHECKED_ITEMS (Kontrol edilen öğeler), SKIPPED_ITEMS (Atlanan öğeler) ve COVERAGE_STATUS (Kontrol durumu) kullanılabilir.

Örneğin görev kapsamında 12 dosya varsa ancak 10 dosya incelenmişse kalan 2 dosyanın kontrol edilmediği açıkça belirtilmelidir.

## Örnek

Agent A üç örnek ayar dosyasını değiştiriyor ancak çalışma ortamındaki bir eksiklik nedeniyle yalnızca iki dosyayı test edebiliyor.

Görev devrinde şu bilgiler yer alıyor:

- 3 dosya değiştirildi.
- 2 dosya test edildi.
- 1 dosya test edilemedi.
- Test edilememe nedeni kaydedildi.
- Sonraki agentın hangi kontrolü tamamlaması gerektiği belirtildi.

Böylece işi devralan agent, hangi işlemlerin tamamlandığını ve nereden devam etmesi gerektiğini görebilir.

## Nasıl uygulanır?

Görev devri üç farklı şekilde uygulanabilir. Hangi yöntemin kullanılacağı, agentların aynı proje dosyalarına erişip erişemediğine ve aralarında otomatik bir aktarım sistemi bulunup bulunmadığına bağlıdır.

### 1. Kullanıcının devri yaptığı kullanım

ChatGPT, Claude veya benzeri iki ayrı agent arasında doğrudan iletişim yoksa kullanıcı devir noktasını başlatır.

İlk agenta görevin başında veya görev devam ederken şu talimat verilebilir:

> **Bu görevi başka bir agent devralabilecek şekilde çalış. Görevin sonunda Handoff Receipt (Görev Devri Kaydı) oluştur. Başlangıç ve çıktı sürümünü, yaptığın değişiklikleri, korunması gereken noktaları, tamamladığın ve yapamadığın kontrolleri, doğrulama sonuçlarını ve sonraki agentın yapması gereken ilk işlemi belirt. Kontrol etmediğin bir öğeyi doğrulanmış olarak gösterme.**

İlk agent görevini bitirdiğinde kullanıcı bu kaydı ikinci agenta verir. İkinci agenta yalnızca “devam et” demek yerine görev devri kaydıyla birlikte şu talimat verilebilir:

> **Bu görev devri kaydını incele. İşleme başlamadan önce belirtilen dosya ve sürümlerin hâlâ güncel olduğunu kontrol et. PRESERVE (Korunacaklar) alanındaki kuralları değiştirme. SKIPPED_CHECKS (Yapılmayan kontroller) ve NEXT_ACTION (Sonraki işlem) alanlarından devam et. Kayıttaki bilgileri doğrulamadan doğru kabul etme.**

Bu yöntemde **kullanıcı köprü görevi görür**. Kullanıcının teknik ayrıntıları yeniden yazması gerekmez; ilk agentın oluşturduğu görev devri kaydını sonraki agenta aktarması yeterlidir.

### 2. Ortak proje dosyası kullanılan kullanım

Aynı proje üzerinde düzenli olarak birden fazla agent çalışıyorsa görev devri kuralı her seferinde yeniden yazılmak zorunda değildir.

Projenin kullandığı agentın okuyabildiği kalıcı talimat dosyasına görev devri kuralı eklenebilir. Kullanılan araca göre bu dosya `AGENTS.md`, proje talimat dosyası veya benzer bir yapı olabilir. Normal `README.md` ise proje kullanıcılarına yönelik açıklamalar için kullanılabilir; agentın README'yi her görevde otomatik olarak talimat kabul edeceği varsayılmamalıdır.

Görev devri kayıtları için ayrıca örneğin `HANDOFF.md` gibi bir dosya kullanılabilir. İlk agent görevin sonunda bu dosyayı günceller. Sonraki agent işe başlarken önce bu kaydı okur ve güncelliğini doğrular.

Örnek akış:

**Kullanıcı görevi Agent A'ya verir → Agent A çalışır → Agent A görev devri kaydını oluşturur → Kullanıcı veya ortak çalışma alanı kaydı Agent B'ye ulaştırır → Agent B sürüm ve koşulları yeniden kontrol eder → Agent B kaldığı yerden devam eder.**

### 3. Otomatik çoklu-agent sistemi

Bir orkestrasyon sistemi veya agentlar arasında veri aktarabilen başka bir otomasyon varsa kullanıcı her devirde kaydı elle taşımak zorunda kalmaz.

Bu durumda Handoff Receipt alanları yapılandırılmış veri olarak saklanabilir. Sistem Agent A'nın çıktısını, görev devri kaydını ve gerekli dosya/sürüm bilgilerini Agent B'nin girdisine ekler. Agent B yine işe başlamadan önce güncel durumu doğrular.

Otomasyon **bilgiyi taşır**; doğrulama ihtiyacını ortadan kaldırmaz.

## İnsan ne zaman devreye girer?

Doğrudan agent-to-agent aktarım yoksa kullanıcı şu noktalarda devreye girer:

1. İlk görevi hangi agentın yapacağını belirler ve görevi başlatır.
2. İş başka bir agenta geçecekse ilk agenttan görev devri kaydını ister.
3. Görev devri kaydını ve gerekiyorsa ilgili dosyaları sonraki agenta verir.
4. Sonraki agentın kaydı doğrulayarak devam etmesini ister.
5. Yetki, kapsam veya önemli bir karar değişecekse bunu ayrıca onaylar.

Kullanıcının agentların yaptığı teknik işi tekrar anlatması gerekmez. Ama otomatik bir aktarım sistemi yoksa **görev devrini başlatan ve kaydı sonraki agenta ulaştıran kişi kullanıcıdır.**

## Ne zaman kullanılır?

Bu yöntem, bir iş bir agenttan başka bir agenta veya kişiye geçecekse; çalışma farklı bir oturumda devam edecekse; aynı proje üzerinde farklı araçlar sırayla çalışacaksa; ya da yapılan iş daha sonra başka biri tarafından devam ettirilecekse kullanışlıdır.

Özellikle dosya veya kod değişikliklerinde, testin başka bir agent tarafından tamamlanacağı durumlarda ve uzun süren işlerin farklı oturumlarda devam etmesinde görev devri kaydı bilgi kaybını azaltır.

Tek bir agentın tek oturumda tamamladığı, başka bir kişiye veya agenta aktarılmayacak ve kolayca geri alınabilecek küçük görevlerde ayrıntılı görev devri kaydı gerekli olmayabilir.
