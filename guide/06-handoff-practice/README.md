# Görev Devri Uygulaması

[English](README_EN.md)

> Bu bölüm, [Agentlar Arası Görev Devri](../03-agent-handoffs/README.md) bölümündeki kuralları **ChatGPT, Codex, Claude, Gemini ve GitHub** ile çalışan bir akışta nasıl uygulayacağını gösterir.

Burada amaç otomasyon kurmak değildir. Önce görev devrinin elle ve kontrollü biçimde nasıl çalıştığını görürüz. Agentların birbirini otomatik başlatması **Agent Otomasyonu** bölümünde ele alınacaktır.

## Bu bölümün önceki bölümden farkı ne?

**Agentlar Arası Görev Devri** bölümü hangi bilgilerin kaybolmaması gerektiğini anlatır.

**Görev Devri Uygulaması** ise bu bilgilerin gerçek bir çoklu-agent akışında **kimden kime, hangi sırayla ve hangi kontrollerle** taşınacağını gösterir.

## Çalışma yapısı

Bu bölümde örnek akış şu rolleri kullanır:

| Parça | Bu örnekteki rolü |
|---|---|
| **ChatGPT** | Görevi düzenler, sonuçları toplar ve sıradaki adımı hazırlar. |
| **Gemini** | Test, analiz veya ikinci görüş üretir. |
| **Codex** | Gerekli kod veya dosya değişikliğini yapar. |
| **Claude** | Yapılan değişikliği bağımsız olarak inceler. |
| **GitHub** | Dosyaların, kaydedilmiş değişikliklerin ve inceleme kayıtlarının ortak çalışma alanıdır. |
| **İnsan** | Kapsam değişikliğini ve ana projeye ekleme kararını onaylar. |

Bu roller örnek akışı anlaşılır tutmak için sabitlenmiştir. Temel kural, bir agentın sonucunun diğer agent tarafından **doğrulanmadan doğru kabul edilmemesidir**.

## Büyük resim

**ÖRNEK — Sadece oku**

```text
İnsan
  |
  v
ChatGPT
Görevi ve sınırları hazırlar
  |
  v
Gemini
Test / analiz yapar
  |
  v
ChatGPT
Sonucu ve güncel sürümü kontrol eder
  |
  v
Codex
Bulguyu doğrular, gerekiyorsa en küçük değişikliği yapar
  |
  v
Claude
Değişikliği bağımsız inceler
  |
  v
İnsan
Son kararı verir

GitHub = ortak dosya, sürüm ve değişiklik kayıtlarının bulunduğu alan
```

Bu bölümde bir agentın bitmesi sonraki agentı kendiliğinden başlatmaz. Görev paketini sıradaki agenta kullanıcı veya ChatGPT kontrollü olarak verir. Otomatik tetikleme daha sonra ele alınacaktır.

## Temel kural

> **Bir agentın bulgusu, sonraki agent için emir değil; güncel durumda yeniden kontrol edilmesi gereken bir bilgidir.**

Örneğin Gemini bir hata bulduysa Codex'e yalnızca “bunu düzelt” denmez.

Bunun yerine şu bilgi taşınır:

```text
Gemini şu sürümde şu sorunu gördü.
Kanıtı şu.
Kontrol edemediği yerler şu.
Güncel sürümde hâlâ geçerli olup olmadığını doğrula.
Doğrulanırsa görev sınırları içinde en küçük değişikliği yap.
```

## Görev paketi

Her agent geçişinde aynı temel yapı kullanılabilir.

**ÖRNEK — Sadece oku**

```text
TASK_ID:          [Görevin kısa kimliği]
INPUT_VERSION:    [İncelenen başlangıç sürümü]

TASK:             [Yapılacak iş]
FILES / CONTEXT:  [Kullanılan dosyalar ve bilgiler]

CHANGE:           [Değişmesine izin verilenler]
PRESERVE:         [Korunması gerekenler]
DO NOT:           [Yapılmaması gerekenler]
LIMITATIONS:      [Agentın göremediği veya yapamadığı kontroller]

EXPECTED_OUTPUT:  [Beklenen sonuç biçimi]
RETURN_TO:        [Sonuç önce kime dönecek?]
NEXT_AGENT:       [Kontrolden sonra iş kime geçecek?]
ROUND:            [Kaçıncı tur? Ör. 1/3]
```

Buradaki iki alan farklı işler yapar:

- **RETURN_TO:** Agent işi bitirdiğinde sonucu önce kime verir?
- **NEXT_AGENT:** Sonuç kontrol edildikten sonra işi hangi agent devralır?

Örneğin Gemini'nin sonucu önce ChatGPT'ye dönebilir; ChatGPT kontrol ettikten sonra görev Codex'e geçebilir.

## Sürüm neden görevle birlikte taşınır?

Bir agent bir dosyayı incelerken başka bir agent aynı dosyada yeni değişiklik yapabilir. Böyle bir durumda eski inceleme hâlâ doğru olmayabilir.

Bu nedenle görev paketinde **INPUT_VERSION (incelenen sürüm)** tutulur. GitHub'daki güncel sürüm daha sonra **CURRENT_VERSION (şu anki sürüm)** olarak kontrol edilir.

Git'te kaydedilmiş her değişikliğin bir kimliği vardır; buna **commit kimliği** denir.

**ÖRNEK — Sadece oku**

```text
INPUT_VERSION:    3f9a2c1
CURRENT_VERSION:  3f9a2c1
VERSION_STATUS:   MATCH
                  -> Aynı sürüm. Kontrole devam edilebilir.

INPUT_VERSION:    3f9a2c1
CURRENT_VERSION:  8b41d07
VERSION_STATUS:   CHANGED
                  -> Sürüm değişmiş. Eski bulgu doğrudan uygulanmaz.
                     Önce güncel sürümde yeniden doğrulanır.
```

Kural:

> **Sürüm değiştiyse eski bulguyu doğrudan uygulatma; önce güncel durumda yeniden doğrulat.**

## Örnek uygulama: Gemini → ChatGPT → Codex → Claude

Aşağıdaki örnek, bir test bulgusunun değişiklik ve inceleme aşamalarından geçmesini gösterir.

### 1. ChatGPT görevi hazırlar

İnsan yapılacak işi ChatGPT'ye verir. ChatGPT görevi küçük ve açık bir pakete dönüştürür.

**CHATGPT'YE YAPIŞTIR**

```text
Bu işi çoklu-agent görev paketine dönüştür.

TASK:
[Buraya yapılacak işi yaz.]

Önce görev sınırlarını belirle:
CHANGE
PRESERVE
DO NOT

Göreve kısa bir TASK_ID ver.
Kullanılacak sürüm bilgisini INPUT_VERSION olarak belirt.
İlk kontrolü Gemini yapacak.
Sonuç önce ChatGPT'ye dönecek.
Gerekirse sonraki değişiklik agentı Codex olacak.

Eksik veya belirsiz bir sınır varsa görevi genişletmeden önce bana bildir.
```

### 2. Gemini inceler

ChatGPT'nin hazırladığı paket ve gerekli dosyalar Gemini'ye verilir. Gemini bu aşamada değişiklik yapmaz; gözlem ve kanıt üretir.

**GEMINI'YE YAPIŞTIR**

```text
Aşağıdaki görev paketini incele.

Yalnızca erişebildiğin dosya ve bilgilere dayan.
Görmediğin bir alanı kontrol edilmiş sayma.

Sonucu şu yapıda döndür:

TASK_ID:
INPUT_VERSION:
OBSERVED: [Doğrudan gördüklerin]
INTERPRETED: [Gözlemlerden çıkardığın yorum]
FINDINGS: [Bulduğun sorunlar]
EVIDENCE: [Bulgunun dayandığı dosya/satır/bilgi]
VERIFIED: [Gerçekten kontrol ettiklerin]
SKIPPED_CHECKS: [Yapamadığın kontroller]
RECOMMENDED_ACTION:
RETURN_TO: ChatGPT
NEXT_AGENT: Codex

Görev paketi:
[ChatGPT'nin hazırladığı paketi buraya ekle.]
```

**OBSERVED (gözlenen)** ile **INTERPRETED (yorumlanan)** ayrı tutulur. Böylece sonraki agent bir yorumu kanıt sanmaz.

### 3. ChatGPT geçiş kontrolünü yapar

Gemini'nin sonucu doğrudan Codex'e gönderilmez. Önce ChatGPT görev kimliğini, sürümü, kanıtı ve eksik kontrolleri kontrol eder.

**CHATGPT'YE YAPIŞTIR**

```text
Aşağıdaki Gemini sonucunu görev devri açısından kontrol et.

1. TASK_ID doğru mu?
2. INPUT_VERSION hangi sürüme ait?
3. GitHub'daki güncel sürümü CURRENT_VERSION olarak belirle.
4. INPUT_VERSION ile CURRENT_VERSION farklıysa bulguyu doğrudan
   düzeltme görevi olarak aktarma; yeniden doğrulama gerektiğini belirt.
5. Aynıysa FINDINGS, EVIDENCE, SKIPPED_CHECKS, CHANGE, PRESERVE ve
   DO NOT bilgilerini kaybetmeden Codex için görev paketi hazırla.
6. Kapsamı kendiliğinden genişletme.
7. Kodda değişiklik yapma.

Gemini sonucu:
[Gemini'nin sonucunu buraya ekle.]
```

ChatGPT'nin görevi burada testi tekrar yapmak değil, **devir bilgisinin eksiksiz ve hâlâ geçerli olup olmadığını kontrol etmektir**.

### 4. Codex bulguyu doğrular ve gerekiyorsa değiştirir

Codex, Gemini'nin yorumunu doğru kabul ederek başlamaz. Önce güncel dosyalarda bulguyu kendisi doğrular.

**CODEX'E YAPIŞTIR**

```text
Aşağıdaki görev paketini uygula.

Önce eriştiğin güncel sürümü paketteki CURRENT_VERSION ile karşılaştır.
Sürüm değişmişse kodu değiştirme; yeniden doğrulama gerektiğini bildir.

Sürüm aynıysa:
1. Bulguyu mevcut kod ve testlerde kendin doğrula.
2. Doğrulanmıyorsa değişiklik yapma ve kanıtını yaz.
3. Doğrulanıyorsa CHANGE / PRESERVE / DO NOT sınırlarına uyarak
   gereken en küçük değişikliği yap.
4. İlgili kontrolleri veya testleri çalıştır.
5. Kapsam dışında bir ihtiyaç oluşursa dur ve insan onayı iste.

Sonuç:
TASK_ID:
CURRENT_VERSION:
CHANGED:
PRESERVED:
VERIFIED:
SKIPPED_CHECKS:
UNEXPECTED_CHANGES:
SCOPE_STATUS:

Görev paketi:
[ChatGPT'nin hazırladığı paketi buraya ekle.]
```

Güvenli değişiklik sınırlarının ayrıntısı [Güvenli Değişiklik](../02-safe-changes/README.md) bölümünde ele alınmıştır.

### 5. Claude bağımsız inceleme yapar

Codex'in “tamamlandı” demesi son kontrol değildir. Claude, görevin sınırlarını ve gerçekleşen değişikliği bağımsız olarak karşılaştırır.

**CLAUDE'A YAPIŞTIR**

```text
Aşağıdaki görevi bağımsız incele.

Başlangıçtaki TASK / CHANGE / PRESERVE / DO NOT bilgilerini,
Codex'in sonucunu ve GitHub'daki gerçekleşen değişiklikleri karşılaştır.

Kontrol et:
- İstenen değişiklik yapılmış mı?
- PRESERVE alanları korunmuş mu?
- DO NOT sınırı aşılmış mı?
- Beklenmeyen dosya veya davranış değişmiş mi?
- Codex'in VERIFIED dediği sonuç için gerçek kanıt var mı?
- Yapılmayan kontroller açıkça belirtilmiş mi?

Sonucu şu yapıda döndür:

TASK_ID:
REVIEWED_VERSION:
OBSERVED:
FINDINGS:
EVIDENCE:
SKIPPED_CHECKS:
SCOPE_STATUS:
RECOMMENDATION: READY_FOR_HUMAN / NEEDS_CHANGES / NEEDS_RECHECK
```

Claude'un sonucu da son karar değildir. Son kabul insana aittir.

## İnsan hangi noktalarda karar verir?

Akış aşağıdaki durumlarda otomatik devam etmez:

| Durum | İnsan kararı |
|---|---|
| Kapsamın genişletilmesi gerekiyor | Yeni sınırı onayla veya görevi daralt. |
| İncelenen sürüm ile güncel sürüm farklı | Yeniden kontrol iste veya görevi durdur. |
| Agentlar aynı bulgu üzerinde anlaşamıyor | Kanıtları karşılaştır ve gerekirse yeni kontrol iste. |
| Beklenmeyen değişiklik var | Nedenini incele; gerekli değilse geri aldır. |
| Tur sınırına ulaşıldı | Görevi böl, kapsamı değiştir veya durdur. |
| Değişiklik ana projeye eklenmeye hazır | Son değişiklik farkını kontrol et ve birleştirme (**merge**) kararını ver. |

## Döngüye sınır koy

İnceleme → düzeltme → yeniden inceleme döngüsü sonsuza kadar sürmemelidir.

Görev paketine örneğin:

```text
ROUND: 1/3
```

eklenebilir.

Her yeni turda sayı artırılır. Son tura ulaşıldığında agentlar kendi başına yeni bir düzeltme turu başlatmaz; görev insana döner.

Buradaki `3` evrensel bir kural değildir. Amaç, sistemin aynı sorun üzerinde kontrolsüz biçimde dönmesini engelleyen açık bir sınır koymaktır.

## Yazılı sınır teknik bir kilit değildir

CHANGE, PRESERVE ve DO NOT alanları agentın nasıl davranması gerektiğini söyler; agentın teknik olarak bu sınırı aşmasını tek başına engellemez.

Bu nedenle:

1. Agentın sonucu görev sınırıyla karşılaştırılır.
2. Gerçek değişiklik GitHub'daki değişiklik farkından (**diff**) kontrol edilir.
3. Kritik kapsam veya yetki değişiklikleri insan onayı olmadan genişletilmez.
4. Ana projeye ekleme (**merge**) kararı insanda kalır.

## Ortak kurallar nerede durur?

Kalıcı proje kuralları ortak bir proje talimatında tutulabilir. Örneğin `AGENTS.md` kullanılabilir.

Ancak dosyanın GitHub'da bulunması, ChatGPT, Codex, Claude ve Gemini'nin tamamının onu otomatik olarak okuduğu anlamına gelmez.

Bir agent ortak talimat dosyasına erişemiyorsa o görev için gerekli kurallar görev paketindeki **PRESERVE / DO NOT / LIMITATIONS** alanlarıyla açıkça taşınır.

Ortak kuralların yüklenmesi ve görev devri kaydının yapısı [Agentlar Arası Görev Devri](../03-agent-handoffs/README.md) bölümünde ele alınmıştır.

## Özet

1. ChatGPT görevi ve sınırları düzenler.
2. Gemini gözlem, kanıt ve eksik kontrolleri üretir.
3. ChatGPT sürüm ve devir kontrolünü yapar.
4. Codex bulguyu yeniden doğrular; gerekiyorsa en küçük değişikliği yapar.
5. Claude değişikliği bağımsız inceler.
6. GitHub ortak dosya, sürüm ve değişiklik kayıtlarını tutar.
7. Kapsam genişletme ve ana projeye ekleme kararı insanda kalır.
8. Her devirde görev kimliği, sürüm, kapsam, kanıt ve yapılmayan kontroller kaybolmadan taşınır.

## Sonraki adım

- **Agent Otomasyonu** — Bu kontrollü görev devrinin, mesajları elle taşımadan tetikleyiciler ve otomatik agent geçişleriyle nasıl çalıştırılacağı. Bu bölüm henüz yayımlanmamıştır.
