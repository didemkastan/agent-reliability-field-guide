# Güvenli Değişiklik

[🇬🇧 English](README_EN.md)

## Problem

Bir agenttan küçük bir değişiklik istendiğinde, istenen sonucu üretirken gerekli olmayan başka alanları da değiştirebilir.

Örneğin yalnızca bir bağlantının düzeltilmesi istenirken agent aynı dosyadaki metinleri yeniden düzenleyebilir veya ilgisiz dosyalarda değişiklik yapabilir. İstenen bağlantı düzelmiş olsa bile değişiklik artık verilen görevin kapsamını aşmıştır.

## Temel kural

> **Değişiklikten önce neyin değiştirilebileceğini ve neyin korunacağını belirle. İstenen sonucu elde etmek için gerekli olan en küçük değişikliği yap.**

## Değişiklik sınırı

Basit bir görevde üç alan yeterli olabilir:

- **CHANGE (Değiştir):** Yapılması istenen değişiklik.
- **PRESERVE (Koru):** Değişmeden kalması gereken alanlar.
- **DO NOT (Yapma):** Görev kapsamında yapılmaması gereken işlemler.

Örneğin:

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

Bu sınırlar her görevde uzun bir liste olmak zorunda değildir. Küçük bir değişiklikte yalnızca kritik noktaları belirtmek yeterlidir.

## Gerekli en küçük değişiklik

Agent, verilen görevi tamamlamak için ihtiyaç duyulmayan düzenlemeleri aynı değişikliğe eklememelidir.

Bir yazım hatasını düzeltmek için tek satır yeterliyse aynı anda paragrafı yeniden yazmak veya dosyanın biçimini değiştirmek gerekli değildir. Ek bir sorun fark edilirse bunu ayrıca belirtmek, görev kapsamını kendiliğinden genişletmekten daha güvenlidir.

## Örnek

Bir README dosyasındaki tek bir bağlantının değiştirilmesi isteniyor.

Beklenen değişiklik:

```text
README.md
- 1 bağlantı değiştirildi
```

İşlem sonunda üç dosyanın değiştiği görülüyor:

```text
README.md
config.yaml
package.json
```

Bağlantı doğru olsa bile `config.yaml` ve `package.json` değişiklikleri verilen görevin parçası değildir. Bu durumda beklenmeyen değişiklikler incelenmeli ve gerekli değilse geri alınmalıdır.

## Kapsam nasıl kontrol edilir?

İşlem sonunda beklenen değişiklik ile gerçekten yapılan değişiklik karşılaştırılabilir:

```text
EXPECTED_FILES_CHANGED: 1
ACTUAL_FILES_CHANGED: 1

EXPECTED:
- README.md

ACTUAL:
- README.md

UNEXPECTED_CHANGES:
- none
```

Dosya sayısı tek başına yeterli değildir. Aynı dosyanın içinde korunması gereken bir bölüm de değiştirilmiş olabilir. Bu nedenle mümkünse değişen satırlar veya diff (değişiklik farkı) da kontrol edilmelidir.

## Kullanılabilir prompt

> **Göreve başlamadan önce istenen değişikliğin kapsamını belirle: hangi dosya veya bölümlerin değiştirileceğini ve nelerin korunması gerektiğini tespit et. Yalnızca verilen görevi tamamlamak için gerekli olan en küçük değişikliği yap. Görevle ilgisi olmayan dosyaları, metinleri, yapılandırmaları, bağımlılıkları veya çalışan mevcut davranışları değiştirme.**
>
> **Değişiklik sırasında kapsam dışında başka bir sorun fark edersen bunu kendiliğinden düzeltme; ayrı olarak bildir. Görevi tamamlamak için başlangıçta belirlenen kapsamın dışına çıkmak zorunlu hale gelirse değişikliğe devam etmeden önce bunu açıkça belirt ve neden gerekli olduğunu açıkla.**
>
> **İşlem tamamlandığında yapılan değişiklikleri başlangıçta belirlenen kapsamla karşılaştır. Hangi dosya ve bölümlerin değiştiğini kontrol et; mümkünse diff (değişiklik farkı) üzerinden beklenmeyen değişiklik olup olmadığını incele. Korunması gereken alanların değişmediğini doğrula. Kapsam dışında bir değişiklik oluşmuşsa bunu görevin parçası kabul etme; açıkça belirt ve gerekli değilse geri al.**
>
> **Sonuçta kısa bir kayıt ver:**
> - **CHANGED:** Gerçekte değiştirilenler
> - **PRESERVED:** Korunduğu kontrol edilenler
> - **UNEXPECTED_CHANGES:** Beklenmeyen değişiklikler; yoksa `none`
> - **SCOPE_STATUS:** Değişiklik verilen görev sınırları içinde kaldı mı?

## Birden fazla agent kullanılıyorsa prompt nereye yazılır?

Birden fazla agent aynı projede çalışıyorsa kullanıcı her yeni görevde **“önce kuralları oku”** demek zorunda kalmamalıdır. Ortak kurallar bir kez tanımlanmalı ve her agentın proje başlangıcında bu kurallara ulaşacağı yapı kurulmalıdır.

Bunun için ortak kurallar tek bir **kanonik proje talimatında** tutulabilir. Örneğin:

```text
PROJECT/
├── AGENTS.md          ← bütün agentlar için ortak kurallar
├── [Agent A girişi]   ← AGENTS.md kurallarına bağlanır
├── [Agent B girişi]   ← AGENTS.md kurallarına bağlanır
├── [Agent C girişi]   ← AGENTS.md kurallarına bağlanır
└── ...
```

Buradaki önemli nokta dosya adından çok **tek bir ortak kural kaynağı** kullanılmasıdır. Farklı agent araçları proje talimatlarını farklı dosya veya ayarlardan yükleyebilir. Bu nedenle her agentın kendi başlangıç mekanizması ortak kurala bağlanmalıdır.

> **Bir dosyanın projede bulunması, bütün agentların onu otomatik olarak okuduğu anlamına gelmez. Kurulum sırasında her agentın ortak kuralları gerçekten yüklediği kontrol edilmelidir.**

Bu bağlantı bir kez doğru kurulduktan sonra ortak kuralların her görev promptuna tekrar yazılması gerekmez. Görev sırasında yalnızca o işe ait bilgiler verilir:

```text
TASK:
- README.md içindeki eski bağlantıyı düzelt.

CHANGE:
- İlgili bağlantı

PRESERVE:
- Diğer metinler ve başlıklar

DO NOT:
- Başka dosyaları değiştirme.
```

Böylece ayrım nettir:

```text
ORTAK / KALICI
AGENTS.md
- güvenli değişiklik kuralları
- doğrulama kuralları
- korunması gereken genel ilkeler

GÖREVE ÖZEL
TASK / CHANGE / PRESERVE / DO NOT
- yalnızca mevcut işin sınırları
```

Çoklu-agent iş akışında başlangıç kontrolü de yapılabilir:

```text
INSTRUCTION_SOURCE: AGENTS.md
INSTRUCTION_VERSION: v1.3
INSTRUCTIONS_LOADED: YES
```

Bu kayıt, agentın yalnızca kurallara erişebildiğini varsaymak yerine hangi talimat kaynağıyla çalıştığını görünür hale getirir. Kurallar değiştirildiğinde sürüm veya hash (dosyanın belirli bir sürümünü tanımlayan dijital değer) gibi ek bilgiler de kullanılabilir.

Otomatik bir çoklu-agent sisteminde bu işi orkestratör üstlenebilir: ortak kuralları ve göreve özel sınırları işi yapacak agenta aktarır. Görev başka bir agenta devredildiğinde de ilgili kapsam ve korunacak alanlar görev devri kaydıyla birlikte taşınır.

### Örnek: ChatGPT, Codex, Claude, Gemini ve GitHub ile görev akışı

Bir yazılım projesinde birden fazla agent farklı rollerle birlikte kullanılabilir. Örneğin:

- **ChatGPT — Orkestratör:** İsteği analiz eder, görevi hazırlar, sınırlarını belirler ve sıradaki işi hangi agentın yapacağını yönlendirir.
- **Codex — Yazılımcı:** Güncel proje üzerinde gerekli kod değişikliğini yapar ve yaptığı değişiklikleri kaydeder.
- **Claude — İnceleme / düzeltme:** Yapılan değişikliği bağımsız olarak inceler; hata, eksik veya kapsam dışı değişiklik bulursa düzeltme döngüsüne girdi sağlar.
- **Gemini — Tester / QA:** Uygulanan çözümü bağımsız olarak test eder ve işlevsel, regresyon veya arayüz sorunlarını arar.
- **GitHub — Ortak çalışma alanı:** Bir agent değildir. Kodun güncel sürümünün, değişiklik geçmişinin, ortak kuralların ve görev kayıtlarının tutulduğu kanonik çalışma alanıdır.

Örnek akış:

```text
                 ChatGPT
                ORKESTRATÖR
       TASK / CHANGE / PRESERVE / DO NOT
                     │
                     ▼
                   Codex
                 YAZILIMCI
              kod değişikliği
                     │
                     ▼
                   GitHub
             ORTAK ÇALIŞMA ALANI
        kod + sürüm + görev kayıtları
                     │
                     ▼
                   Claude
           İNCELEME / DÜZELTME
                     │
             sorun varsa Codex
             düzeltme döngüsü
                     │
                     ▼
                   GitHub
                     │
                     ▼
                   Gemini
                 TESTER / QA
```

Bu roller sabit olmak zorunda değildir; projeye ve göreve göre değiştirilebilir. Önemli olan her agentın hangi işi yaptığının, hangi sürüm üzerinde çalıştığının ve sonucunu nereye bırakacağının açık olmasıdır.

#### Ortak çalışma alanına doğrudan erişemeyen bir agent

Her agent GitHub'a doğrudan bağlanamayabilir. Bu durum o agentın projede görev alamayacağı anlamına gelmez.

> **Bir agentın projeye dahil olması, GitHub'a bağlı olması demek değildir.**

Örneğin yukarıdaki yapıda tester rolündeki Gemini'nin GitHub'a doğrudan erişimi olmadığını düşünelim. Orkestratör, test görevi için gerekli ve güncel bilgileri bir görev paketi olarak aktarabilir:

```text
TASK_ID:
INPUT_VERSION:

TASK:
FILES / CONTEXT:

RULES:
PRESERVE:
DO NOT:

EXPECTED_OUTPUT:
LIMITATIONS:

RETURN_TO:
NEXT_AGENT:
NEXT_ACTION:
```

Bu pakette agentın hangi sürüm üzerinde çalıştığı, hangi dosya veya bilgileri gördüğü, neyi test edeceği, neleri koruması gerektiği ve hangi alanlara erişemediği açıkça belirtilir. Dışarıdaki agent da sonucunu aynı `TASK_ID` ve `INPUT_VERSION` ile geri verir.

### Doldurulmuş örnek: GitHub'a bağlı olmayan tester

Örneğin bir ekran geçişinde yapılan değişikliğin mevcut davranışı bozup bozmadığını kontrol etmek istiyoruz.

Gemini'ye yalnızca görev için gerekli olan **güncel sürümdeki** dosyalar eklenir:

```text
navigation.py
settings_screen.py
tests/test_navigation.py
test_output.txt
```

Tüm repository'yi göndermek yerine görev için gerekli dosyaların seçilmesi, hem kapsamı sınırlar hem de agentın görmediği alanlar hakkında varsayım yapmasını önler.

Gemini'ye verilecek görev paketi:

```text
TASK_ID: UI-024
INPUT_VERSION: commit abc123

TASK:
Ayarlar ekranından geri dönüş davranışını incele.
Mevcut ekran geçişlerinde regresyon riski olup olmadığını kontrol et.

FILES / CONTEXT:
- navigation.py
- settings_screen.py
- tests/test_navigation.py
- test_output.txt

RULES:
- Yalnızca verilen dosya ve test çıktısına dayan.
- Gözlem ile yorumu birbirinden ayır.
- Yapamadığın kontrolleri açıkça belirt.

PRESERVE:
- Mevcut çalışan ekran geçişleri
- Ayarlar dışındaki navigasyon davranışları

DO NOT:
- Dosyalarda değişiklik yapma.
- Görmediğin repository içeriğini kontrol edilmiş kabul etme.
- GitHub'daki güncel sürümü gördüğünü varsayma.

EXPECTED_OUTPUT:
- Bulunan sorunlar
- Sorunu destekleyen dosya/satır veya test kanıtı
- Önerilen sonraki işlem
- Yapılan ve yapılamayan kontroller

LIMITATIONS:
- GitHub'a doğrudan erişim yok.
- Yalnızca eklenen dosyalar ve test çıktısı görülebilir.

RETURN_TO: ChatGPT / Orkestratör
NEXT_AGENT: Codex
NEXT_ACTION:
Bulguyu GitHub'daki güncel sürüm üzerinde doğrula.
Sorun hâlâ geçerliyse gerekli en küçük düzeltmeyi yap.
```

#### Gemini'ye verilecek örnek prompt

> **Ekli görev paketini ve dosyaları incele. TASK_ID ve INPUT_VERSION bilgilerini yanıtında aynen koru. Yalnızca sana verilen dosyalar ve test çıktısı üzerinden değerlendirme yap. Önce OBSERVED (Gözlemlenen), ardından INTERPRETED (Yorumlanan) bilgilerini yaz. Bir sorun bulursan hangi dosya, bölüm veya test sonucunun bunu desteklediğini belirt. Çalıştıramadığın testleri veya erişemediğin alanları SKIPPED_CHECKS altında göster. Dosyalarda değişiklik yapma. Sonucu aşağıdaki yapıyla döndür:**
>
> ```text
> TASK_ID:
> INPUT_VERSION:
> 
> OBSERVED:
> INTERPRETED:
> 
> FINDINGS:
> EVIDENCE:
> 
> VERIFIED:
> SKIPPED_CHECKS:
> 
> RECOMMENDED_ACTION:
> RETURN_TO:
> NEXT_AGENT:
> ```

Gemini örneğin şöyle bir bulgu döndürebilir:

```text
TASK_ID: UI-024
INPUT_VERSION: commit abc123

OBSERVED:
settings_screen.py içindeki geri dönüş işlemi selected_theme değerini
yeniden oluşturuyor. test_navigation.py bu durumu kontrol etmiyor.

INTERPRETED:
Ayarlar ekranından geri dönüldüğünde seçilen tema bilgisinin
kaybolma riski var.

FINDINGS:
- Geri dönüş akışında durum bilgisinin korunması kontrol edilmeli.
- Mevcut test bu senaryoyu kapsamıyor.

EVIDENCE:
- settings_screen.py: geri dönüş işlemi
- tests/test_navigation.py: ilgili durum kontrolü bulunmuyor

VERIFIED:
- Verilen üç kaynak dosya ve test çıktısı incelendi.

SKIPPED_CHECKS:
- Uygulama çalıştırılmadı.
- Repository'nin diğer dosyaları incelenmedi.

RECOMMENDED_ACTION:
Güncel GitHub sürümünde davranışı yeniden üret.
Sorun doğrulanırsa en küçük düzeltmeyi yap ve regresyon testi ekle.

RETURN_TO: ChatGPT / Orkestratör
NEXT_AGENT: Codex
```

Burada iki farklı yön bilgisi vardır: **`RETURN_TO`**, Gemini'nin sonucunu ilk olarak kime teslim edeceğini; **`NEXT_AGENT`** ise kontrol tamamlandıktan sonra görevin hangi agenta devam edeceğini gösterir.

Bu örnekte iki farklı çalışma biçimini ayırmak gerekir.

#### GitHub'a bağlı olmayan agenttan sonucu geri alma

Bu örnekte Gemini GitHub'daki ortak çalışma alanına doğrudan bağlı değildir. Bu nedenle görev paketi ve gerekli dosyalar kullanıcı tarafından Gemini'ye verilmiştir. Gemini testi tamamladığında ürettiği sonuç da kullanıcı tarafından ChatGPT / orkestratöre aktarılır:

```text
Gemini testi tamamlar
        ↓
Kullanıcı Gemini sonucunu ChatGPT'ye verir
        ↓
ChatGPT / Orkestratör
TASK_ID ve INPUT_VERSION'ı okur
        ↓
GitHub'dan CURRENT_VERSION'ı kontrol eder
        ↓
INPUT_VERSION ↔ CURRENT_VERSION karşılaştırılır
        ↓
Codex için güncel görev paketi hazırlanır
        ↓
Kullanıcı görev paketini Codex'e verir
        ↓
Codex bulguyu doğrular
        ↓
gerekirse minimum düzeltme + test
        ↓
GitHub
```

Bu durumda ChatGPT'nin görevi Gemini'nin teknik bulgusunu yeniden test etmek değildir. ChatGPT **geçiş kontrolünü** yapar: bulgunun hangi göreve ve sürüme ait olduğunu kontrol eder, GitHub'daki güncel sürümle karşılaştırır, eksik bağlamı korur ve Codex için güvenli görev paketini hazırlar.

Gemini sonucunu ChatGPT'ye verirken örneğin şu prompt kullanılabilir:

> **Aşağıdaki sonuç GitHub'a doğrudan erişimi olmayan tester agenttan geldi. Önce TASK_ID ve INPUT_VERSION bilgilerini kontrol et. GitHub'daki güncel branch/commit bilgisini al ve CURRENT_VERSION olarak kaydet. INPUT_VERSION ile CURRENT_VERSION aynı değilse sonucu doğrudan Codex'e uygulama görevi olarak aktarma; hangi ilgili dosyaların değiştiğini kontrol et ve yeniden doğrulama gerektiğini belirt. Aynıysa bulgu, kanıt, SKIPPED_CHECKS, PRESERVE ve LIMITATIONS bilgilerini kaybetmeden Codex için bir sonraki görev paketini hazırla. Kodda değişiklik yapma.**
>
> **Tester sonucu:**
>
> ```text
> TASK_ID: UI-024
> INPUT_VERSION: commit abc123
> FINDING: Geri dönüş akışında seçilen tema bilgisinin kaybolma riski var.
> EVIDENCE: settings_screen.py geri dönüş işlemi; mevcut testte durum kontrolü yok.
> SKIPPED_CHECKS: Uygulama çalıştırılmadı; repository'nin diğer dosyaları incelenmedi.
> RETURN_TO: ChatGPT / Orkestratör
> NEXT_AGENT: Codex
> ```

ChatGPT'nin hazırladığı çıktı örneğin şöyle olabilir:

```text
TASK_ID: UI-024
TESTER_INPUT_VERSION: commit abc123
CURRENT_VERSION: commit abc123
VERSION_STATUS: MATCH

SOURCE_FINDING:
Geri dönüş akışında seçilen tema bilgisinin kaybolma riski var.

EVIDENCE:
settings_screen.py geri dönüş işlemi;
mevcut testte durum kontrolü yok.

SKIPPED_CHECKS:
- Uygulama tester tarafından çalıştırılmadı.
- Repository'nin diğer dosyaları tester tarafından incelenmedi.

NEXT_AGENT: Codex
NEXT_ACTION:
Bulguyu güncel kod üzerinde doğrula.
Doğrulanırsa gerekli en küçük düzeltmeyi yap ve testi tamamla.
```

Bu çıktı **Codex'e verilecek görev paketidir**. Böylece Gemini'den gelen serbest bir bulgu doğrudan “düzelt” komutuna dönüşmez; önce sürüm ve bağlam kontrolünden geçer.

#### ChatGPT'nin hazırladığı görevi Codex'e verme

Manuel veya yarı otomatik düzende kullanıcı, ChatGPT'nin hazırladığı yukarıdaki görev paketini Codex'e verir. Codex'e verilecek talimat şöyle olabilir:

> **Aşağıdaki görev paketi orkestratör tarafından hazırlanmıştır. Önce kendi eriştiğin GitHub branch/commit bilgisini tekrar kontrol et ve paketteki CURRENT_VERSION ile karşılaştır. Eşleşmiyorsa değişiklik yapmadan önce görevi güncel sürüm üzerinde yeniden değerlendir. Eşleşiyorsa dış tester bulgusunu mevcut kod ve testlerle bağımsız olarak doğrula. Sorun doğrulanırsa AGENTS.md ve PRESERVE sınırlarına uyarak gerekli en küçük düzeltmeyi yap. Doğrulanmazsa kodu değiştirme ve kanıtıyla birlikte bildir.**
>
> **İşlem sonunda CURRENT_VERSION, CHANGED, PRESERVED, VERIFIED, SKIPPED_CHECKS ve SCOPE_STATUS bilgilerini döndür.**

#### Tam otomatik geçişte tetikleyici ne yapar?

Tam otomatik sistemde kullanıcının yukarıdaki iki taşıma işlemini yapması gerekmez. Bunun için orkestratörün yanında bir **tetikleme mekanizması** bulunur.

```text
Gemini TASK_COMPLETE sonucu üretir
        ↓
Tetikleyici sonucu orkestratöre iletir
        ↓
Orkestratör GitHub CURRENT_VERSION kontrolünü yapar
        ↓
sürüm ve kapsam uygunsa
        ↓
Codex görevi oluşturulur / başlatılır
        ↓
Codex sonucu tekrar orkestratöre döner
```

Tetikleyici bir API olayı, webhook, kuyruk, CI/CD adımı veya kullanılan agent platformunun görev tamamlama olayı olabilir. Hangi teknoloji kullanılırsa kullanılsın tetikleyicinin görevi **“önceki agent tamamlandı” bilgisini yakalayıp sonucu orkestratöre ulaştırmaktır**. Orkestratör ise sürüm, kapsam ve yetki kontrollerini yaptıktan sonra sıradaki görevi oluşturur.

Bu nedenle **orkestratör** ile **tetikleyici** aynı şey değildir: tetikleyici geçişi başlatır; orkestratör neyin, hangi bilgilerle ve hangi agenta aktarılacağını yönetir.

Bilgiyi alan Codex'in görevi dış agentın sonucunu doğru kabul etmek değil, **güncel proje üzerinde yeniden doğrulamaktır**:

```text
Gemini bulgusu
TASK_ID + INPUT_VERSION
        ↓
ChatGPT / Orkestratör
bilgiyi ve sınırları korur
        ↓
Codex
CURRENT_VERSION kontrolü
        ↓
bulguyu yeniden doğrular
        ↓
doğrulanmadı ──→ değişiklik yapma, sonucu bildir
        │
     doğrulandı
        ↓
minimum düzeltme
        ↓
test / diff / kapsam kontrolü
        ↓
GitHub
```

Bu yöntem, GitHub'a bağlı olmayan agentın projeye değer katmasını sağlar; ancak dış agentın bulgusunu doğrudan proje gerçeği veya otomatik değişiklik yetkisi haline getirmez.

```text
GitHub
ortak çalışma alanı
     │
     ▼
ChatGPT / Orkestratör
güncel görev paketini hazırlar
     │
     ▼
Gemini
GitHub erişimi olmadan testi yapar
     │
     ▼
TEST RESULT
TASK_ID + INPUT_VERSION
     │
     ▼
Orkestratör / bağlı agent
güncel sürümle sonucu karşılaştırır
     │
     ▼
GitHub
```

Sonuç doğrudan projeye uygulanmamalıdır. Önce dışarıdaki agenta verilen `INPUT_VERSION` ile GitHub'daki güncel sürüm karşılaştırılır. Proje bu sırada değişmişse sonuç güncel sürüm üzerinde yeniden değerlendirilir.

Bu sayede ortak çalışma alanına doğrudan erişemeyen bir agent da kontrollü biçimde projeye dahil edilebilir. Gerekli olan doğrudan GitHub bağlantısından çok **güncel görev bağlamı, sürüm bilgisi, açık yetki sınırı ve yapılandırılmış görev devridir.**

Bu akışın agentlar arasında kendiliğinden ilerlemesi için ayrıca bir **orkestratör veya tetikleme mekanizması** gerekir. Agentların aynı GitHub repository'sine erişebilmesi, tek başına bir agentın işi bitirdiğinde diğerinin otomatik olarak başlayacağı anlamına gelmez.

## Otomasyonda kullanım

Otomatik bir iş akışında beklenen dosyalar görev başlamadan önce tanımlanabilir. İşlem sonunda sistem gerçekten değişen dosyaları bu listeyle karşılaştırabilir.

Beklenmeyen bir dosya değişmişse işlem doğrudan kabul edilmek yerine incelemeye alınabilir. Daha sıkı kontrollerde dosya içindeki değişen satırlar da izin verilen alanlarla karşılaştırılabilir.

### İnsan olmadan agentlar arası geçiş nasıl kurulur?

Bir agentın GitHub repository'sine erişebilmesi ile başka bir agentı otomatik olarak başlatabilmesi aynı şey değildir. İnsan olmadan geçiş için üç ayrı katman gerekir:

```text
GitHub olayı
PR / push / label / workflow sonucu
        ↓
TETİKLEYİCİ
GitHub Actions
        ↓
YÖNLENDİRME KURALI
hangi durum → hangi görev?
        ↓
AGENT ÇALIŞTIRMA
Codex / Claude / Gemini
        ↓
YAPILANDIRILMIŞ SONUÇ
        ↓
GitHub'da yeni olay
        ↓
sıradaki aşama
```

Agentın gerçekten otomatik çalışabilmesi için ayrıca programatik bir çalışma yolu bulunmalıdır. Bu bir API, CLI, GitHub entegrasyonu veya agent platformunun desteklediği başka bir çalışma mekanizması olabilir. Yalnızca repository'ye erişim verilmiş olması bu mekanizmanın var olduğu anlamına gelmez.

GitHub'ın **Agentic Workflows** özelliği bu yapıyı GitHub Actions üzerinde kurmak için kullanılabilen güncel seçeneklerden biridir. Bu özellik halen **public preview** durumundadır; bu nedenle kurulum ve sözdizimi zaman içinde değişebilir. Güncel sürümde GitHub Copilot, Claude Code, OpenAI Codex ve Google Gemini gibi agent motorları workflow içinde seçilebilmektedir.

### Örnek: Codex → Claude → Codex otomatik döngüsü

Örneğin ilk görev Codex'e verilsin, Claude değişikliği incelesin ve sorun bulursa Codex düzeltmeyi otomatik olarak devralsın:

```text
Görev
  ↓
Codex / Yazılımcı
  ↓
Pull Request oluşturur
  ↓
PR opened / synchronize
  ↓
GitHub Actions tetiklenir
  ↓
Claude / Review
  ↓
 ┌──────────────────────┐
 │                      │
sorun var             uygun
 │                      │
 ▼                      ▼
agent:fix-required   agent:review-passed
 │
 ▼
GitHub Actions
 │
 ▼
Codex / Düzeltme
 │
 ▼
PR branch güncellenir
 │
 ▼
synchronize olayı
 │
 └──────────────→ Claude tekrar inceler
```

Burada kullanıcı **“Codex bitti, şimdi Claude'a geç”** veya **“Claude hata buldu, Codex'e geri dön”** mesajlarını taşımaz. Geçişi GitHub olayları ve workflow kuralları yapar.

### 1. Repository'yi agentic workflow için hazırlama

GitHub CLI kurulu ve repository için yetkilendirilmiş olmalıdır. GitHub Agentic Workflows uzantısı daha sonra kurulabilir:

```bash
gh extension install github/gh-aw
gh aw init
```

Workflow kaynakları `.github/workflows/` altında Markdown olarak tutulur. Frontmatter bölümünde tetikleyici, agent motoru, izinler ve güvenli çıktılar; Markdown gövdesinde ise agentın görevi tanımlanır.

Örneğin:

```text
.github/workflows/
├── implement.md
├── implement.lock.yml
├── review.md
├── review.lock.yml
├── fix.md
└── fix.lock.yml
```

`.lock.yml` dosyaları `gh aw compile` tarafından üretilir. Workflow frontmatter'ı değiştirildiğinde yeniden derlenmeli ve kaynak Markdown dosyasıyla birlikte commit edilmelidir:

```bash
gh aw compile
```

### 2. Agent kimlik bilgilerini güvenli biçimde tanımlama

Agent motorunun gerektirdiği kimlik bilgisi repository dosyalarına yazılmaz. GitHub **Settings → Secrets and variables → Actions** altında secret olarak tutulur.

Güncel GitHub Agentic Workflows kurulumunda örneğin:

```text
Codex   → CODEX_API_KEY veya OPENAI_API_KEY
Claude  → ANTHROPIC_API_KEY
Gemini  → GEMINI_API_KEY
```

Secret'ın gerçek değeri `AGENTS.md`, workflow dosyası, görev paketi veya kaynak kod içine yazılmamalıdır.

### 3. Claude review workflow'unu PR olayıyla tetikleme

Örneğin Codex'in oluşturduğu veya güncellediği PR, Claude review workflow'unu otomatik başlatabilir:

```markdown
---
on:
  pull_request:
    types: [opened, synchronize]

engine: claude

permissions:
  contents: read
  pull-requests: read

safe-outputs:
  add-comment:
  add-labels:
    allowed: ["agent:fix-required", "agent:review-passed"]
---

# Review

PR değişikliğini AGENTS.md ve görev sınırlarına göre incele.

Kontrol et:
- İstenen değişiklik yapılmış mı?
- PRESERVE alanları korunmuş mu?
- Kapsam dışı dosya değişmiş mi?
- Test veya doğrulama eksiği var mı?

Sorun varsa bulguyu kanıtıyla birlikte yaz ve
`agent:fix-required` etiketini iste.

Sorun yoksa yapılan kontrolleri belirt ve
`agent:review-passed` etiketini iste.
```

Buradaki `opened` ilk PR oluşturulduğunda, `synchronize` ise PR branch'ine yeni commit geldiğinde review'u yeniden tetikler.

### 4. Claude'un bulgusu Codex'i nasıl otomatik başlatır?

Review sonucunda kullanılan etiket bir **durum bilgisi** olmanın yanında sıradaki workflow için tetikleyici olabilir.

Örneğin Codex düzeltme workflow'u yalnız `agent:fix-required` etiketi geldiğinde çalıştırılabilir:

```markdown
---
on:
  label_command:
    name: agent:fix-required
    events: [pull_request]

engine: codex

permissions:
  contents: read
  pull-requests: read

safe-outputs:
  push-to-pull-request-branch:
---

# Fix

Tetiklenen PR üzerindeki review bulgularını incele.

Önce:
1. PR'nin güncel HEAD commit'ini INPUT_VERSION olarak kaydet.
2. Review bulgusunun bu sürümde hâlâ geçerli olduğunu doğrula.
3. AGENTS.md içindeki ortak kuralları ve PRESERVE sınırlarını uygula.

Bulgu doğrulanırsa yalnızca gerekli en küçük düzeltmeyi yap.
İlgili testleri çalıştır.
Doğrulanmayan bir bulgu için kod değiştirme.

Sonuçta CHANGED, PRESERVED, VERIFIED,
SKIPPED_CHECKS ve SCOPE_STATUS bilgilerini üret.
```

Codex PR branch'ini güncellediğinde yeni commit bir `synchronize` olayı oluşturur. Bu olay yukarıdaki Claude review workflow'unu tekrar çalıştırır. Böylece düzeltme döngüsü kullanıcı mesaj taşımadan devam edebilir.

### 5. Review geçtiğinde test aşamasına geçme

Claude `agent:review-passed` durumunu ürettiğinde bu da ayrı bir test workflow'unu tetikleyebilir:

```text
agent:review-passed
        ↓
GitHub Actions
        ↓
test / build / lint
        ↓
PASS → VERIFIED
FAIL → agent:fix-required veya ayrı hata görevi
```

Deterministik testler için ayrıca bir AI agent kullanmak zorunlu değildir. Birim testleri, lint, build ve benzeri kesin kontroller normal GitHub Actions adımlarıyla çalıştırılabilir. Agent yalnızca yorumlama veya bağlamsal değerlendirme gereken yerde kullanılabilir.

### 6. Tetikleyici ile orkestratörü ayır

Bu yapıda:

```text
GitHub olayı
= "bir şey oldu"

Tetikleyici
= "ilgili workflow'u başlat"

Orkestrasyon kuralı
= "bu sonuçtan sonra hangi aşama çalışmalı?"

Agent
= "kendisine verilen işi yap"
```

Örneğin `agent:fix-required` etiketi **“Codex'i çalıştır”** anlamına gelen bir komut olarak kullanılabilir. `agent:review-passed` ise test aşamasını başlatabilir.

Ayrı bir ChatGPT orkestratörü kullanılacaksa onun da programatik olarak çağrılabileceği bir entegrasyon gerekir. Böyle bir entegrasyon yoksa GitHub içindeki otomatik yönlendirmeyi GitHub Actions ve workflow durumları üstlenebilir. Bu ayrım, “ChatGPT orkestratör” rolü ile “GitHub üzerinde gerçekten çalışan otomasyon” mekanizmasını birbirine karıştırmayı önler.

### 7. Agentlara doğrudan sınırsız yazma yetkisi verme

Otomasyon kurulurken mümkün olan en düşük yetki kullanılmalıdır. Agentın doğrudan `main` branch'ini değiştirmesi yerine değişikliği PR üzerinden üretmesi daha kontrollü bir akış sağlar.

GitHub Agentic Workflows içindeki **safe outputs (güvenli çıktılar)** agentın önerdiği yazma işlemini ayrı ve izin kontrollü bir adımda uygulayabilir. Örneğin `create-pull-request`, `push-to-pull-request-branch` ve sınırlı etiket işlemleri kullanılabilir.

Dosya kapsamı da sınırlandırılabilir:

```text
ALLOWED_FILES:
- src/navigation/**
- tests/navigation/**

PROTECTED:
- AGENTS.md
- .github/**
- dependency / package dosyaları
```

Agentın görev için gerek duymadığı güvenlik, workflow veya talimat dosyalarını değiştirebilmesi otomatik olarak açılmamalıdır.

### 8. Aynı görevin iki kez çalışmasını engelle

Aynı PR veya TASK_ID için iki agent çalışmasının çakışması veri kaybına veya birbirinin değişikliğini ezmesine yol açabilir. GitHub Actions tarafında `concurrency` veya agentic workflow'un uygun kilitleme mekanizması kullanılarak aynı görev için eşzamanlı çalışmalar sınırlandırılabilir.

Mantık:

```text
CONCURRENCY_KEY:
TASK_ID veya PR_NUMBER

UI-024 çalışıyor
        ↓
ikinci UI-024 geldi
        ↓
beklet / iptal et / yeniden değerlendir
```

### 9. İlk otomasyonda tüm sistemi birden kurma

İlk denemede dört agentı birbirine bağlamak yerine tek bir geçiş otomatikleştirilmelidir:

```text
Codex PR oluşturdu
        ↓
Claude otomatik review yaptı
```

Bu geçiş güvenilir biçimde çalıştıktan sonra:

```text
Claude → Codex düzeltme
Codex → Claude yeniden review
Review → test
Test → doğrulama
```

adımları sırayla eklenebilir.

Her yeni geçişte en az şu bilgiler korunmalıdır:

```text
TASK_ID
INPUT_VERSION
CURRENT_VERSION
SOURCE_AGENT
NEXT_AGENT
FINDINGS
EVIDENCE
PRESERVE
SKIPPED_CHECKS
NEXT_ACTION
```

Böylece otomasyon yalnızca agentları sırayla çalıştırmaz; **hangi görevin, hangi sürümden, hangi kanıt ve sınırlarla bir sonraki aşamaya geçtiğini de korur.**

> **Not:** GitHub Agentic Workflows bu rehber hazırlanırken public preview durumundadır. Kurulumdan önce güncel GitHub dokümantasyonundaki engine, trigger, permission ve safe-output seçenekleri tekrar kontrol edilmelidir.

## Ne zaman kullanılır?

Bu yöntem özellikle mevcut bir dosya veya kod üzerinde sınırlı bir düzeltme yapılırken, çalışan bölümlerin korunması gerektiğinde veya agentın yalnızca belirli dosyalara dokunması istendiğinde kullanışlıdır.

Küçük ve kolayca geri alınabilen denemelerde ayrıntılı bir değişiklik sınırı kaydı gerekmeyebilir. Örneğin boş bir deneme dosyasında birkaç farklı metin biçimini karşılaştırırken hangi satırların korunacağını ayrıca tanımlamak çoğu zaman gerekli değildir.
