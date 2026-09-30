# Güvenli Değişiklik

[English](README_EN.md)

> Bu bölüm tek bir şeyi öğretir: **çoklu-agent çalışma akışında bir değişiklik yapılırken yalnızca istenen alanın değişmesini ve korunması gereken yerlerin bozulmamasını nasıl sağlarız?**

Bu rehberde çalışma yapısı **ChatGPT, Codex, Claude, Gemini ve GitHub** üzerinden ele alınır. Görev devri ve otomasyon sonraki bölümlerde ayrıca açıklanacaktır.

## Problem

Bir agenta küçük bir değişiklik verildiğinde, agent istenen sonucu üretirken görevin dışındaki alanları da değiştirebilir.

Bu durum çoklu-agent akışında daha önemlidir. İlk agentın yaptığı gereksiz bir değişiklik sonraki agenta aktarılabilir; sonraki agent da bu değişikliği doğru kabul ederek çalışmasına devam edebilir.

Sonuçta asıl görev doğru yapılmış olsa bile istenmeyen değişiklikler agentlar arasında taşınabilir.

## Temel kural

> **Değişiklik başlamadan önce neyin değişebileceğini, neyin korunacağını ve neyin yapılmaması gerektiğini açıkça belirle. İstenen sonuç için gereken en küçük değişikliği yap.**

Bu sınır, değişikliği yapan agenttan görevi inceleyen veya devralan agenta kadar korunmalıdır.

## Değişiklik sınırı: üç alan

- **CHANGE (Değiştir):** Yapılması istenen değişiklik.
- **PRESERVE (Koru):** Aynı kalması gereken alanlar.
- **DO NOT (Yapma):** Bu görev sırasında yapılmaması gerekenler.

**ÖRNEK — Sadece oku**

```text
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

Agent görev sırasında kapsam dışında başka bir sorun fark ederse onu kendiliğinden düzeltmek yerine **bildirmelidir**. Kapsamın genişletilmesine ihtiyaç varsa yeni sınır önce açıkça belirlenir.

## Beklenen ve gerçekleşen değişiklik

İstenen değişiklik yalnızca `README.md` dosyasındaki bir bağlantının güncellenmesiyse:

**ÖRNEK — Sadece oku**

```text
BEKLENEN                GERÇEKLEŞEN
README.md               README.md
                        config.yaml     <- beklenmiyordu
                        package.json    <- beklenmiyordu
```

Bağlantı doğru düzelmiş olsa bile diğer iki dosya başlangıçtaki görevin parçası değildir.

Bu nedenle iki soru birlikte sorulur:

1. **İstenen sonuç oluştu mu?**
2. **Yalnızca izin verilen değişiklikler mi yapıldı?**

## Kapsam nasıl kontrol edilir?

İlk kapsam kontrolünü, değişikliği yapan agenttan farklı bir agent yapabilir. Örneğin Codex değişikliği yaptıysa Claude veya Gemini diff'i inceleyebilir. Ancak bir değişiklik ana dala alınmadan (**merge edilmeden**) önce son kapsam kontrolü ve kabul kararı insana aittir.

Kontrol sırasında CHANGE ile gerçekleşen değişiklik karşılaştırılır ve şu sorular cevaplanır:

- Beklenmeyen bir dosya değişmiş mi?
- PRESERVE alanındaki bir bölüm değiştirilmiş mi?
- DO NOT altında yasaklanan bir işlem yapılmış mı?
- Görevi tamamlamak için gerekmeyen ek bir değişiklik yapılmış mı?

Agentın kendi raporu tek başına doğrulama değildir. Gerçek değişiklik farkı (**diff**) kontrol edilir.

### GitHub'da diff kontrolü

Değişiklik GitHub'da bir Pull Request (PR) içindeyse PR'ı aç ve **Files changed** sekmesine gir. Burada değişen dosyaları ve satırları görürsün.

Kontrol eden agent burada bağımsız inceleme yapabilir; insan da merge öncesi son kontrolde aynı diff'i gözden geçirir.

### Bilgisayarda diff kontrolü

Komutları bilgisayarındaki repository klasöründe Terminal veya PowerShell içinde çalıştır.

**TERMİNALDE ÇALIŞTIR**

```bash
git status
git diff --stat
git diff
```

- `git status` hangi dosyaların değiştiğini gösterir.
- `git diff --stat` değişiklikleri dosya bazında özetler.
- `git diff` değişen satırları gösterir.

> Bu komutlardaki normal `git diff`, henüz commit edilmemiş çalışma alanı değişikliklerini kontrol etmek için kullanılır. Agent değişikliği zaten commit ettiyse GitHub'daki **Files changed** görünümünü kullanabilir veya son commit için `git show HEAD` ile ayrıca kontrol edebilirsin.

## Beklenmeyen değişiklik bulunursa ne yapılır?

Beklenmeyen değişiklik otomatik olarak kabul edilmez. Önce neden oluştuğu belirlenir.

### Seçenek 1 — Agenta açıklat ve geri aldır

**AGENTA YAPIŞTIR**

```text
Başlangıçtaki CHANGE / PRESERVE / DO NOT sınırlarıyla mevcut diff'i karşılaştır.
Kapsam dışında kalan değişiklikleri listele.
Her beklenmeyen değişikliğin neden oluştuğunu açıkla.
Görevin tamamlanması için gerekli değilse yalnızca kapsam dışı değişikliği geri al.
Kapsamı kendiliğinden genişletme.
```

Düzeltmeden sonra diff yeniden kontrol edilir.

### Seçenek 2 — Commit edilmemiş dosyayı yerelde geri al

Önce ilgili dosyanın farkını gör:

**TERMİNALDE ÇALIŞTIR**

```bash
git diff config.yaml
```

Dosyadaki **tüm commit edilmemiş yerel değişiklikleri silmek istediğinden eminsen**:

**TERMİNALDE ÇALIŞTIR**

```bash
git restore config.yaml
```

> **Dikkat:** `git restore config.yaml` dosyadaki commit edilmemiş değişiklikleri silebilir. Emin değilsen çalıştırma. Değişiklik zaten commit edildiyse bu komutu çözüm olarak kullanma; önce commit farkını incele.

## Kapsam agentlar arasında nasıl korunur?

CHANGE / PRESERVE / DO NOT bilgisi yalnızca ilk görev mesajında kalmaz. Görev bir agenttan diğerine geçtiğinde kapsam ve gerçekleşen sonuç birlikte taşınır.

```text
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
CHANGED + UNEXPECTED_CHANGES + SCOPE_STATUS
  |
  v
Bağımsız kapsam kontrolü
  |
  v
Sonraki agenta görev + kapsam + sonuç
```

Her geçişte en az şu bilgiler korunur:

```text
CHANGE
PRESERVE
DO NOT
CHANGED
UNEXPECTED_CHANGES
SCOPE_STATUS
```

Kalıcı ortak kuralların `AGENTS.md` gibi proje talimatlarında nasıl tutulacağı ve sonraki agenta nasıl aktarılacağı [Agentlar Arası Görev Devri](../03-agent-handoffs/README.md) bölümünde ele alınır.

## Kullanılabilir görev talimatı

**AGENTA YAPIŞTIR — Köşeli parantezli alanları kendi görevinle doldur**

```text
Göreve başlamadan önce kapsamı kontrol et.

CHANGE:
- [Buraya değiştirilmesini istediğin dosya, bölüm veya davranışı yaz.]

PRESERVE:
- [Buraya kesinlikle korunması gereken dosya, bölüm veya çalışan davranışları yaz.]

DO NOT:
- [Buraya bu görevde yapılmaması gereken işlemleri yaz.]

İstenen sonucu üretmek için gereken en küçük değişikliği yap.

Kapsam dışında başka bir sorun fark edersen kendiliğinden düzeltme;
ayrıca bildir.

Görevi tamamlamak için kapsamın dışına çıkmak zorunlu hale gelirse
değişiklik yapmadan önce dur ve nedenini belirt.

İş bitince gerçekleşen değişiklikleri başlangıçtaki kapsamla karşılaştır.

Sonuç kaydı:
CHANGED: [Gerçekte değiştirdiklerini yaz.]
PRESERVED: [Korunduğunu kontrol ettiklerini yaz.]
UNEXPECTED_CHANGES: [Beklenmeyen değişiklikleri yaz; yoksa none.]
SCOPE_STATUS: [IN_SCOPE veya OUT_OF_SCOPE]
```

## Ne zaman kullanılır?

Bu kural çoklu-agent akışında bir agent proje üzerinde değişiklik yaptığında özellikle şu durumlarda önemlidir:

- Çalışan kod veya dosya üzerinde sınırlı değişiklik yapılırken
- Belirli alanların kesinlikle korunması gerektiğinde
- Bir agentın yaptığı değişiklik başka bir agent tarafından devralınırken
- Aynı görev başka bir agent tarafından incelenirken veya doğrulanırken

### Ne zaman ayrıntılı kullanmak gerekmez?

Kolayca geri alınabilen, gerçek proje davranışını etkilemeyen küçük denemelerde ayrıntılı CHANGE / PRESERVE / DO NOT kaydı gerekmeyebilir. Örneğin boş bir deneme dosyasında birkaç metin biçimini karşılaştırmak için tam kapsam kaydı oluşturmak gereksiz olabilir.

## Özet

1. Değişiklik başlamadan **CHANGE / PRESERVE / DO NOT** belirlenir.
2. Agent yalnızca gereken **en küçük değişikliği** yapar.
3. Gerçekleşen değişiklik **diff** üzerinden kontrol edilir.
4. Beklenmeyen değişikliğin nedeni incelenir ve gerekli değilse geri alınır.
5. Kapsam bilgisi görevle birlikte sonraki agenta taşınır.
6. Bağımsız agent kontrolü kullanılabilir; **merge öncesi son kabul insana aittir.**

## Sonraki adımlar

- [**Agentlar Arası Görev Devri**](../03-agent-handoffs/README.md) — Görevin, kapsamın, sürüm bilgisinin ve sonuçların agentlar arasında nasıl taşınacağı.
- **Agent Otomasyonu** — Güvenli görev devrinin otomatik tetikleyiciler ve kontrollü agent geçişleriyle nasıl uygulanacağı. Bu bölüm henüz yayımlanmamıştır.
