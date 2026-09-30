# Güvenli Değişiklik

[English](README_EN.md)

> Bu bölüm tek bir şeyi öğretir: **çoklu-agent çalışma akışında bir değişiklik yapılırken yalnızca istenen alanın değişmesini ve korunması gereken yerlerin bozulmamasını nasıl sağlarız?**

Bu rehberde çalışma yapısı **ChatGPT, Codex, Claude, Gemini ve GitHub** üzerinden ele alınır. Agentların görevleri nasıl devraldığı ve bu yapının nasıl otomatikleştirileceği sonraki bölümlerde açıklanacaktır.

## Problem

Bir agenta küçük bir değişiklik verildiğinde, agent istenen sonucu üretirken görevin dışındaki alanları da değiştirebilir.

Bu durum çoklu-agent akışında daha önemli hale gelir. İlk agentın yaptığı gereksiz bir değişiklik sonraki agenta aktarılabilir; sonraki agent da bu değişikliği doğru kabul ederek çalışmasına devam edebilir.

Sonuçta asıl görev doğru yapılmış olsa bile istenmeyen değişiklikler agentlar arasında taşınabilir.

## Temel kural

> **Değişiklik başlamadan önce neyin değişebileceğini, neyin korunacağını ve neyin yapılmaması gerektiğini açıkça belirle. İstenen sonuç için gereken en küçük değişikliği yap.**

Bu sınır yalnızca değişikliği yapan agent için değil, görevi daha sonra inceleyecek veya devralacak agentlar için de geçerlidir.

## Değişiklik sınırı: üç alan

- **CHANGE (Değiştir):** Yapılması istenen değişiklik.
- **PRESERVE (Koru):** Aynı kalması gereken alanlar.
- **DO NOT (Yapma):** Bu görev sırasında yapılmaması gerekenler.

**Agenta verilecek görev örneği**

```
CHANGE:
- README.md içindeki eski bağlantıyı yenisiyle değiştir.

PRESERVE:
- Diğer metinler
- Başlık sırası
- Dosya yapısı

DO NOT:
- Başka dosyaları değiştirme.
- İlgisiz metinleri yeniden yazma.
```

Liste uzun olmak zorunda değildir. Önemli olan, görevin sınırının değişiklik başlamadan önce görünür olmasıdır.

## En küçük değişiklik

Bir yazım hatası tek satırda düzeltilebiliyorsa aynı anda paragrafı yeniden yazmaya veya başka dosyaları düzenlemeye gerek yoktur.

Agent görev sırasında kapsam dışında başka bir sorun fark ederse onu kendiliğinden düzeltmek yerine **bildirmelidir**.

Kapsamın genişletilmesine ihtiyaç varsa bu yeni durum önce görünür hale getirilir. Böylece sonraki agent, başlangıçtaki görev ile sonradan fark edilen yeni ihtiyacı birbirine karıştırmaz.

## Beklenen ve gerçekleşen değişiklik

İstenen değişiklik yalnızca `README.md` dosyasındaki bir bağlantının güncellenmesiyse beklenen kapsam şöyledir:

```
BEKLENEN                GERÇEKLEŞEN
README.md               README.md
                        config.yaml     <- beklenmiyordu
                        package.json    <- beklenmiyordu
```

Bağlantı doğru düzelmiş olsa bile diğer iki dosya başlangıçtaki görevin parçası değildir.

Bu nedenle yalnızca **“istenen sonuç oluştu mu?”** sorusu yeterli değildir.

İkinci soru da sorulmalıdır:

> **“Yalnızca izin verilen değişiklikler mi yapıldı?”**

## Kapsam nasıl kontrol edilir?

İş tamamlandığında iki şey karşılaştırılır:

1. **Beklenen değişiklikler:** Görev başında CHANGE alanında tanımlananlar.
2. **Gerçekleşen değişiklikler:** Agentın gerçekten değiştirdiği dosya ve alanlar.

Kontrol sırasında şunlara bakılır:

- Beklenmeyen bir dosya değişmiş mi?
- PRESERVE alanındaki bir bölüm değiştirilmiş mi?
- DO NOT altında yasaklanan bir işlem yapılmış mı?
- Görevi tamamlamak için gerekmeyen ek bir değişiklik yapılmış mı?

Yalnızca agentın kendi raporuna güvenilmez. Mümkün olduğunda gerçek değişiklik farkı (**diff**) üzerinden kontrol yapılır.

## Çoklu-agent akışında kapsam kaydı

CHANGE / PRESERVE / DO NOT bilgisi yalnızca ilk görev mesajında kalmamalıdır.

Görev bir agenttan diğerine geçtiğinde aynı kapsam bilgisi de görevle birlikte taşınmalıdır.

```
GÖREV
  |
  +-- CHANGE
  +-- PRESERVE
  +-- DO NOT
  |
  v
Agent çalışması
  |
  v
Gerçekleşen değişiklik
  |
  v
Kapsam kontrolü
  |
  v
Sonraki agenta görev + kapsam + sonuç
```

Böylece sonraki agent yalnızca ortaya çıkan dosyayı değil, **hangi değişikliğin izinli olduğunu ve hangi alanların korunması gerektiğini** de görür.

Görev devrinin nasıl kaydedileceği ve sürüm bilgisinin nasıl taşınacağı sonraki bölümde ele alınacaktır.

## Kullanılabilir görev talimatı

```
Göreve başlamadan önce kapsamı kontrol et.

CHANGE:
Yalnızca burada belirtilen değişiklikleri yap.

PRESERVE:
Burada belirtilen dosya, bölüm ve çalışan davranışları koru.

DO NOT:
Burada yasaklanan alanlara veya işlemlere dokunma.

İstenen sonucu üretmek için gereken en küçük değişikliği yap.

Kapsam dışında başka bir sorun fark edersen kendiliğinden düzeltme.
Ayrıca bildir.

Görevi tamamlamak için kapsamın dışına çıkmak zorunlu hale gelirse
değişiklik yapmadan önce dur ve nedenini belirt.

İş bitince gerçekleşen değişiklikleri başlangıçtaki kapsamla karşılaştır.

Sonuç kaydı:
CHANGED: Gerçekte değiştirilenler
PRESERVED: Korunduğu kontrol edilenler
UNEXPECTED_CHANGES: Beklenmeyen değişiklikler (yoksa: none)
SCOPE_STATUS: Değişiklik görev sınırında kaldı mı?
```

> **Agentın “kapsamda kaldım” demesi tek başına doğrulama değildir.** Mümkün olduğunda gerçekleşen değişiklikler ayrıca kontrol edilmelidir.

## Bu kural agentlar arasında nasıl korunur?

Bu rehberde kullanılan ChatGPT, Codex, Claude ve Gemini aynı görevin farklı aşamalarında çalışabilir. GitHub ise ortak çalışma alanı olarak kullanılabilir.

Bu bölümde önemli olan hangi agentın hangi rolü üstlendiği değil, **görev sınırının agent değiştiğinde kaybolmamasıdır**.

Her geçişte en az şu bilgiler korunur:

```
CHANGE
PRESERVE
DO NOT
CHANGED
UNEXPECTED_CHANGES
SCOPE_STATUS
```

Bir sonraki agent, önceki agentın yaptığı değişikliği otomatik olarak doğru kabul etmez. Önce görev sınırı ile gerçekleşen değişikliği karşılaştırır.

Bu aktarımın hangi kayıt yapısıyla yapılacağı **Görev Devri Uygulaması** bölümünde ele alınacaktır.

## Ne zaman kullanılır?

Bu kural çoklu-agent akışında bir agentın proje üzerinde değişiklik yaptığı her görevde kullanılabilir.

Özellikle şu durumlarda önemlidir:

- Çalışan bir kod veya dosya üzerinde sınırlı değişiklik yapılırken
- Belirli alanların kesinlikle korunması gerektiğinde
- Bir agentın yaptığı değişiklik başka bir agent tarafından devralınırken
- Aynı görev birden fazla agent tarafından incelenirken veya doğrulanırken

## Özet

1. Değişiklik başlamadan **CHANGE / PRESERVE / DO NOT** belirlenir.
2. Agent yalnızca gereken **en küçük değişikliği** yapar.
3. Gerçekleşen değişiklik başlangıçtaki kapsamla karşılaştırılır.
4. Beklenmeyen değişiklikler otomatik olarak görevin parçası kabul edilmez.
5. Kapsam bilgisi görevle birlikte sonraki agenta taşınır.
6. Agentın kendi raporu tek başına doğrulama sayılmaz.

## Sonraki adımlar

- **Görev Devri Uygulaması:** Görevin, kapsamın, sürüm bilgisinin ve sonuçların agentlar arasında nasıl taşınacağı.
- **Agent Otomasyonu:** Güvenli görev devrinin otomatik tetikleyiciler ve kontrollü agent geçişleriyle nasıl uygulanacağı.
