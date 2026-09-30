# Güvenli Değişiklik ve Agent Otomasyonu

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

## Güvenli Agent Geçişi ve Otomasyon Kurulumu

Buraya kadar görevlerin agentlar arasında hangi bilgilerle aktarılması gerektiğini gördük. Şimdi aynı geçişin kullanıcı her seferinde araya girmeden nasıl yapılabileceğine bakalım.

Bu bölümdeki amaç, yalnızca birkaç workflow dosyası oluşturmak değildir. Önce **agentların birbirlerinin yaptığı işi nasıl gördüğünü**, sonra **GitHub'ın sıradaki agentı nasıl başlattığını** anlamak gerekir.

### Agentlar birbirlerinin yaptığını nasıl görür?

En önemli nokta şudur:

> **Bir agentın yaptığı değişiklik diğer agentın ekranında kendiliğinden belirmez. Ortak nokta GitHub'dır.**

Örneğin Codex bir dosyayı yalnızca kendi çalışma ortamında değiştirdiyse Claude veya ChatGPT bu değişikliği henüz göremez.

Değişikliğin önce GitHub'a ulaşması gerekir:

```text
Codex dosyayı değiştirdi
        ↓
değişiklik commit edildi
        ↓
GitHub'a gönderildi
        ↓
branch veya Pull Request güncellendi
        ↓
GitHub artık değişikliği biliyor
```

Yani **yerel değişiklik ile GitHub'daki değişiklik aynı şey değildir.**

Codex dosyayı değiştirmiş fakat değişiklik henüz GitHub'a gönderilmemişse GitHub'da tetiklenecek bir olay da yoktur.

Değişiklik GitHub'a ulaştıktan sonra Claude gibi başka bir agent başlatılabilir. Yeni başlayan agent GitHub'daki güncel repository, branch, commit veya Pull Request üzerinden çalışır.

```text
Codex
  ↓
GitHub'a değişikliği gönderir
  ↓
GitHub güncel durumu saklar
  ↓
Claude başlatılır
  ↓
Claude GitHub'daki güncel değişikliği okur
```

Burada Codex'in Claude'a doğrudan mesaj göndermesi gerekmez. **GitHub ortak çalışma alanı ve Gerçeğin Kaynağı (Source of Truth) olur.**

### GitHub'a bağlı olmak ile otomatik çalışmak aynı şey değildir

Bir agentın GitHub'a erişebilmesi, GitHub'da sürekli beklediği ve her değişikliği anında gördüğü anlamına gelmez.

İki ayrı yetenek vardır:

```text
GITHUB ERİŞİMİ
= agent gerektiğinde repository'yi okuyabilir
  ve verilen yetkiye göre işlem yapabilir

OTOMATİK ÇALIŞTIRMA
= belirli bir olay olduğunda sistem
  agentı kendiliğinden başlatabilir
```

Örneğin ChatGPT'nin bu sohbet içinde GitHub'a erişebilmesi, GitHub'daki her commit sonrasında ChatGPT'nin otomatik olarak çalışacağı anlamına gelmez.

Aynı şekilde bir Codex veya Claude bağlantısının bulunması da tek başına otomatik agent geçişi oluşturmaz.

GitHub Agentic Workflows içinde GitHub Copilot, Claude Code, OpenAI Codex ve Google Gemini doğrudan **engine (çalıştırılacak agent motoru)** olarak seçilebilir. ChatGPT ürününü ayrı bir orkestratör olarak kullanmak istenirse, ChatGPT'nin de otomasyon tarafından programatik olarak çağrılabileceği ayrı bir entegrasyon gerekir.

### GitHub'da “olay” ne demektir?

“Olay”, GitHub'da otomasyonun fark edebileceği belirli bir değişikliktir.

Örneğin:

```text
Pull Request açıldı
veya
PR'a yeni commit geldi
veya
bir etiket eklendi
veya
bir workflow elle/otomatik başlatıldı
        ↓
GitHub Actions ilgili kuralı kontrol eder
        ↓
eşleşen workflow başlatılır
```

Örneğin Codex değişikliği GitHub'a gönderip bir Pull Request açtığında GitHub **“yeni PR açıldı”** olayını kaydeder. Önceden tanımladığımız kural bu olayı dinliyorsa Claude inceleme görevi otomatik olarak başlayabilir.

### Otomasyonun temel parçaları

Bütün yapıyı şu şekilde düşünebiliriz:

```text
1. ORTAK DURUM
GitHub'daki güncel kod / PR / commit

        ↓

2. TETİKLEYİCİ
“Bir şey değişti, workflow'u başlat.”

        ↓

3. YÖNLENDİRME
“Bu durumda sırada hangi görev var?”

        ↓

4. AGENT
Codex / Claude / Gemini verilen işi yapar

        ↓

5. SONUÇ
Bulgu, commit, PR, test sonucu veya görev kaydı

        ↓

6. DOĞRULAMA
Sonuç gerçekten doğru mu?

        ↓

7. SONRAKİ GEÇİŞ
Gerekliyse sıradaki workflow başlar
```

Agentlar arasında görünmez bir sohbet olduğu varsayılmaz. Her yeni çalışma, kendisine verilen görev ile GitHub'daki güncel durumu kullanır.

### Örnek akış: Codex → Claude → Codex

Örneğimizde Codex kod değişikliğini yapıyor, Claude değişikliği inceliyor. Claude sorun bulursa görev yeniden Codex'e dönüyor.

```text
Görev
  ↓
Codex değişikliği yapar
  ↓
değişiklik GitHub'a gönderilir
  ↓
Pull Request açılır
  ↓
Claude incelemesi başlatılır
  ↓
 ┌─────────────────────┐
 │                     │
sorun var           sorun yok
 │                     │
 ▼                     ▼
Codex'e dön          teste geç
 │
 ▼
Codex bulguyu doğrular
ve gerekiyorsa düzeltir
 │
 ▼
PR güncellenir
 │
 ▼
Claude yeniden inceler
```

Bu yapıda kullanıcı **“Codex bitti, şimdi Claude'a geç”** veya **“Claude sorun buldu, Codex'e geri dön”** mesajlarını taşımak zorunda kalmaz.

Ancak bunun gerçekten otomatik olması için aşağıdaki kurulumların yapılmış olması gerekir.

### 1. Kuruluma başlamadan önce gerekenler

GitHub Agentic Workflows kullanılacaksa önce şu temel koşullar sağlanmalıdır:

```text
Repository
→ yazma yetkisi var

GitHub Actions
→ repository için açık

GitHub CLI
→ kurulu ve GitHub hesabıyla giriş yapılmış

Agent hesabı
→ kullanılacak Codex / Claude / Gemini / Copilot erişimi var

Kimlik doğrulama
→ gerekli API anahtarı veya desteklenen kimlik doğrulama yöntemi hazır
```

Buradaki `gh` ile başlayan komutlar **GitHub web sitesine veya proje dosyasına yazılmaz. Bilgisayarınızdaki komut ekranında çalıştırılır.** Windows kullanıyorsanız **PowerShell** veya **Windows Terminal**, macOS/Linux kullanıyorsanız **Terminal** açabilirsiniz.

GitHub CLI, GitHub işlemlerini bu komut ekranından yapmamızı sağlayan araçtır. Önce Terminal/PowerShell'i açın ve GitHub CLI'ın kurulu ve hesabınıza bağlı olup olmadığını kontrol edin:

```bash
gh --version
gh auth status
```

İlk komut GitHub CLI'ın kurulu olup olmadığını, ikinci komut ise GitHub hesabıyla bağlantı durumunu gösterir.

Gerekirse aynı Terminal/PowerShell ekranında repository ve workflow yetkileriyle giriş yapılır:

```bash
gh auth login --scopes repo,workflow
```

### 2. Repository'yi Agentic Workflows için hazırla

GitHub Agentic Workflows uzantısı yine **aynı Terminal/PowerShell ekranında** kurulabilir:

```bash
gh extension install github/gh-aw
gh aw init
```

İlk komut Agentic Workflows uzantısını kurar. İkinci komut bulunduğunuz repository'yi Agentic Workflows kullanımı için hazırlar. Bu nedenle komutları çalıştırmadan önce Terminal/PowerShell'de üzerinde çalışacağınız repository klasöründe olduğunuzdan emin olun.

Agentlara ait workflow kaynakları `.github/workflows/` klasöründe tutulur.

Örneğin:

```text
.github/workflows/
├── review.md
├── review.lock.yml
├── fix.md
└── fix.lock.yml
```

`.md` dosyası insanlar ve agentlar için okunabilir görev tanımıdır.

Dosyanın üst bölümünde görevin **ne zaman başlayacağı, hangi agentın kullanılacağı ve hangi izinlerin verileceği** belirtilir. Alt bölümde ise agenta verilecek görev normal metinle yazılır.

`.lock.yml` dosyası GitHub Actions'ın çalıştıracağı derlenmiş sürümdür.

Workflow ayarları değiştiğinde, repository klasöründe açık olan **Terminal/PowerShell** içinde:

```bash
gh aw compile
```

çalıştırılır. Bu komut workflow'un GitHub Actions tarafından çalıştırılacak `.lock.yml` sürümünü üretir. Kaynak `.md` dosyası ile oluşan `.lock.yml` daha sonra birlikte GitHub'a gönderilir.

Workflow yalnız bilgisayarda hazırlanıp GitHub'a gönderilmezse GitHub onu çalıştıramaz.

### 3. Agent anahtarlarını kodun içine yazma

Codex, Claude veya Gemini için gereken gizli bilgiler repository dosyalarına yazılmamalıdır.

GitHub'da:

**Settings → Secrets and variables → Actions**

alanında **secret (gizli değer)** olarak saklanır.

Güncel Agentic Workflows yapısında kullanılan başlıca değerler:

```text
Codex   → OPENAI_API_KEY veya CODEX_API_KEY
Claude  → ANTHROPIC_API_KEY
Gemini  → GEMINI_API_KEY
```

Gerçek anahtar değeri `AGENTS.md`, workflow, görev paketi veya kaynak kod içine eklenmez.

Bu anahtar agentın AI hizmetini çalıştırmak içindir. GitHub'da dosya okuma, PR güncelleme veya başka workflow başlatma yetkileri ise ayrıca GitHub izinleriyle kontrol edilir.

### 4. Codex işi bitirdiğinde Claude değişikliği nasıl görür?

Codex'in yalnızca **“işim bitti”** demesi yeterli değildir.

Değişikliğin GitHub'a ulaşması gerekir:

```text
Codex değişikliği yaptı
        ↓
commit / branch / PR GitHub'a gönderildi
        ↓
GitHub güncel sürümü kaydetti
        ↓
Claude workflow'u başladı
        ↓
Claude PR'ın güncel kodunu ve değişikliklerini okudu
```

Pull Request ile çalışan bir workflow'da GitHub Agentic Workflows varsayılan olarak ilgili repository'yi çalışma ortamına alır; PR olayıyla çalışıyorsa PR'ın güncel branch'ini de çalışma bağlamına getirir.

Bu nedenle Claude, Codex'in belleğine bakmaz. **GitHub'daki kaydedilmiş değişikliğe bakar.**

Örnek tetikleyici:

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
---

# Review

PR değişikliğini görev sınırlarına göre incele.

Önce PR'nin güncel HEAD commit'ini CURRENT_VERSION olarak kaydet.

Kontrol et:
- İstenen değişiklik yapılmış mı?
- PRESERVE alanları korunmuş mu?
- Kapsam dışı dosya değişmiş mi?
- Test veya doğrulama eksiği var mı?

Bulguları kanıtlarıyla birlikte yaz.
```

Burada:

```text
opened
= PR ilk açıldığında çalış

synchronize
= PR'a yeni commit geldiğinde tekrar çalış
```

### 5. Her geçişte sürümü yeniden kontrol et

Claude'un gördüğü sürüm ile Codex'in daha sonra düzelteceği sürümün aynı olduğu varsayılmamalıdır.

Arada başka bir commit gelmiş olabilir.

Bu nedenle görev devrinde en az şu kontrol yapılmalıdır:

```text
Claude inceleme yaptı
INPUT_VERSION: abc123

        ↓

Codex görevi aldı
CURRENT_VERSION: abc123

        ↓

MATCH
→ bulgu güncel sürüm üzerinde devam edebilir
```

Eğer:

```text
INPUT_VERSION: abc123
CURRENT_VERSION: def456
```

ise Codex eski bulguyu doğrudan uygulamamalıdır. Önce ilgili değişikliğin yeni sürümde hâlâ geçerli olup olmadığını kontrol etmelidir.

Bu kontrol otomasyon içinde de korunmalıdır.

### 6. Claude'un sonucu Codex'i nasıl başlatır?

Burada iki farklı yöntem birbirine karıştırılmamalıdır.

#### Yöntem A — Workflow'u doğrudan başlatmak

Agentlar arası açık bir otomasyon zinciri kurmak için bir workflow diğer izin verilen workflow'u **dispatch-workflow (workflow başlatma)** ile çağırabilir.

Mantık:

```text
Claude incelemeyi bitirdi
        ↓
sorun buldu
        ↓
fix workflow'unu başlat
        ↓
Codex çalıştı
```

Örneğin review workflow'unda yalnızca izin verilen `fix` workflow'unun başlatılmasına izin verilebilir:

```yaml
safe-outputs:
  dispatch-workflow:
    workflows: [fix]
    max: 1
```

`fix` workflow'u da `workflow_dispatch` ile çalışmayı kabul eder.

Bu yöntem agentlar arası yönlendirmeyi açık hale getirir: **Claude sonucu → fix workflow → Codex.**

Görevle birlikte `TASK_ID`, PR numarası, incelenen commit ve bulgu gibi bilgiler de sonraki workflow'a aktarılmalıdır.

#### Yöntem B — Etiketi durum veya komut olarak kullanmak

GitHub etiketi de kullanılabilir:

```text
agent:fix-required
= düzeltme gerekiyor
```

`label_command` kullanıldığında belirli bir etiket workflow'u başlatan tek kullanımlık komut gibi davranabilir.

Ancak burada önemli bir ayrıntı vardır: GitHub'ın varsayılan `GITHUB_TOKEN` kimliğiyle yapılan bazı otomatik yazma işlemleri yeni workflow/CI çalışmaları başlatmaz. Bu davranış otomasyonların kendi kendini sonsuza kadar tetiklemesini önlemek içindir.

Bu nedenle **“Claude etiketi ekledi, Codex kesin otomatik başlar”** varsayımı yapılmamalıdır.

Etiket veya agent tarafından oluşturulan PR/commit üzerinden yeni bir CI zinciri başlatılacaksa kullanılan token ve tetikleme yöntemi ayrıca kontrol edilmelidir.

Agentic Workflows içinde PR oluşturma veya PR branch'ine yapılan güvenli yazmaların yeni CI çalıştırması isteniyorsa bunun için uygun bir CI tetikleme kimliği yapılandırılabilir. Alternatif olarak agentlar arası geçiş için doğrudan `dispatch-workflow` kullanılabilir.

Başlangıç için daha anlaşılır model:

```text
DURUMU GÖSTER
→ yorum / etiket

SIRADAKİ AGENTI BAŞLAT
→ dispatch-workflow
```

Böylece “durumu göstermek” ile “başka bir agentı gerçekten çalıştırmak” birbirine karışmaz.

### 7. Codex bulguyu körü körüne uygulamamalı

Claude sorun bildirdiğinde Codex'e:

> “Claude söyledi, düzelt.”

demek güvenli değildir.

Codex önce güncel GitHub sürümünü kontrol etmelidir:

```text
1. Hangi PR üzerinde çalışıyorum?
2. Claude hangi commit'i inceledi?
3. PR'ın güncel HEAD commit'i ne?
4. Bulgu bu sürümde hâlâ geçerli mi?
5. Hangi dosyaları değiştirmeme izin var?
```

Ancak bundan sonra gerekiyorsa en küçük düzeltme yapılır.

Örnek görev:

```text
TASK_ID: UI-024
INPUT_VERSION: abc123
CURRENT_VERSION: abc123

FINDING:
- Geri dönüş akışında durum bilgisi korunmuyor.

PRESERVE:
- Diğer ekran geçişleri

ALLOWED_FILES:
- src/navigation/**
- tests/navigation/**

NEXT_ACTION:
- Bulguyu doğrula.
- Geçerliyse en küçük düzeltmeyi yap.
- İlgili testi çalıştır.
```

### 8. Düzeltmeden sonra Claude nasıl tekrar çalışır?

Codex düzeltmeyi PR branch'ine gönderdiğinde GitHub'daki kod güncellenir.

Fakat **kodun güncellenmesi ile sıradaki workflow'un kesin olarak başlaması aynı şey değildir.**

GitHub Actions, varsayılan token ile otomasyonların kendi oluşturduğu bazı olaylardan yeni workflow zincirleri başlatılmasını sınırlar.

Bu nedenle tam otomatik döngü kurulurken geçiş açıkça tasarlanmalıdır:

```text
Codex düzeltmeyi yaptı
        ↓
PR güncellendi
        ↓
review workflow'u açıkça yeniden başlatıldı
        ↓
Claude güncel commit'i yeniden okudu
```

Bunu iki şekilde kurmak mümkündür:

```text
A) Codex sonucundan review workflow'unu
   dispatch-workflow ile başlat

veya

B) PR güncellemesinin yeni CI/workflow tetiklemesine
   izin veren uygun GitHub kimliğini yapılandır
```

İlk kurulumda **A yöntemi**, yani geçişlerin açıkça workflow'dan workflow'a yapılması, akışı anlamayı ve hata ayıklamayı kolaylaştırır.

### 9. Testleri her zaman AI agenta yaptırma

Claude incelemeyi geçtiğinde test aşamasına geçilebilir:

```text
Claude review
        ↓
uygun
        ↓
test / build / lint
        ↓
başarılı → VERIFIED
başarısız → düzeltme görevi
```

Sonucu açık olan kontroller için yeni bir AI agent çalıştırmak gerekmez.

Örneğin:

```text
unit test
build
lint
type check
```

normal GitHub Actions adımlarıyla çalıştırılabilir.

AI agentı; kodun istenen davranışa uyup uymadığını yorumlamak, kapsam dışı değişikliği değerlendirmek veya bağlama göre inceleme yapmak gerektiğinde kullanmak daha anlamlıdır.

### 10. Tetikleyici, orkestrasyon ve agent farklı görevlerdir

Bu kavramları en basit haliyle ayıralım:

```text
TETİKLEYİCİ
= "şimdi başla"

ORKESTRASYON KURALI
= "şimdi kim, hangi görevle başlayacak?"

AGENT
= "verilen görevi yap"

GITHUB
= "ortak güncel durumu ve çalışma kayıtlarını tut"
```

Örneğin:

```text
PR açıldı
        ↓
tetikleyici review workflow'unu başlattı
        ↓
Claude inceleme yaptı
        ↓
orkestrasyon kuralı soruna göre fix workflow'unu seçti
        ↓
Codex düzeltme görevini aldı
```

ChatGPT ayrıca orkestratör olarak kullanılacaksa ChatGPT'nin de otomasyon tarafından çağrılabileceği gerçek bir entegrasyon gerekir. Yalnızca GitHub bağlantısının bulunması bunu sağlamaz.

### 11. Agentlara yalnızca ihtiyaç duydukları yetkiyi ver

Otomasyon kurulduğunda agent kullanıcı beklemeden çalışabilir. Bu nedenle **least privilege (en az yetki)** uygulanmalıdır.

GitHub Agentic Workflows agent çalışmalarını varsayılan olarak okuma ağırlıklı tutar. Yazma işlemleri **safe outputs (güvenli çıktılar)** üzerinden ayrı ve kontrollü bir adımda uygulanabilir.

Örneğin:

```text
Claude
→ kodu oku
→ PR'ı incele
→ yorum iste

Codex
→ yalnızca izin verilen dosyalarda düzeltme hazırla
→ PR branch'ine kontrollü biçimde gönder
```

Dosya sınırı da doğrudan tanımlanabilir:

```text
ALLOWED_FILES:
- src/navigation/**
- tests/navigation/**

PROTECTED:
- AGENTS.md
- .github/**
- dependency / package dosyaları
```

Agentın görev için gerek duymadığı workflow, güvenlik, talimat veya bağımlılık dosyalarını değiştirebilmesi otomatik olarak açılmamalıdır.

### 12. Aynı görevin iki kez çalışmasını engelle

Aynı PR veya görev için iki çalışma aynı anda başlarsa birbirlerinin sonucunu geçersiz hale getirebilir.

GitHub Agentic Workflows ve GitHub Actions eşzamanlı çalışmaları sınırlandırmak için concurrency (eşzamanlı çalışma kontrolü) kullanabilir.

Mantık:

```text
UI-024 çalışıyor
        ↓
aynı görev tekrar geldi
        ↓
eski çalışma / yeni çalışma politikası kontrol edilir
        ↓
çakışan iki değişiklik aynı anda uygulanmaz
```

PR tabanlı agentic workflow'larda yeni commit geldiğinde eski ve artık güncel olmayan çalışmanın iptal edilmesi gibi kontroller de kullanılabilir.

### 13. Bir adım başarısız olursa zinciri devam ettirme

Otomasyonda **“workflow çalıştı”** ile **“workflow başarılı oldu”** aynı şey değildir.

Örneğin Claude çalıştırılamadıysa, API anahtarı hatalıysa, gerekli dosya okunamadıysa veya Codex değişikliği GitHub'a gönderemediyse sıradaki adım otomatik olarak başarılı kabul edilmemelidir.

```text
WORKFLOW BAŞLADI
        ↓
işlem tamamlandı mı?
        ↓
gerekli çıktı oluştu mu?
        ↓
doğrulama geçti mi?
        ↓
EVET → sıradaki aşama
HAYIR → dur / hata kaydı oluştur
```

Başarısız çalışma önce GitHub web sitesindeki **Actions** sekmesinden incelenebilir.

Daha ayrıntılı kontrol gerektiğinde bilgisayarınızdaki **Terminal/PowerShell** de kullanılabilir. Repository klasöründe Terminal/PowerShell'i açıp:

```bash
gh aw logs
```

komutunu çalıştırarak Agentic Workflows çalışmalarını görebilirsiniz. İncelemek istediğiniz çalışmanın **RUN_ID (çalışma numarası)** bilgisini buradan aldıktan sonra:

```bash
gh aw audit <RUN_ID>
```

komutundaki `<RUN_ID>` yerine gerçek çalışma numarasını yazın. Örneğin çalışma numarası `123456` ise:

```bash
gh aw audit 123456
```

Bu komutlar otomasyonu kurmak için değil, **kurulmuş otomasyonun nasıl çalıştığını kontrol etmek ve sorun olduğunda nedenini araştırmak için** kullanılır.

### 14. Otomasyonun maliyetini de sınırla

Her agent çalışması ücretsiz ve sınırsız bir işlem gibi düşünülmemelidir.

Agentic workflow çalıştırıldığında hem GitHub Actions çalışma süresi hem de kullanılan AI sağlayıcısının model çalıştırma maliyeti oluşabilir.

Bu nedenle gereksiz yere AI agent başlatmamak önemlidir.

```text
Kesin kontrol yapılabiliyor mu?
        ↓
EVET → normal test / script / GitHub Actions
HAYIR → yorumlama gerekiyorsa AI agent
```

Agentic Workflows çalışma başına AI kullanım sınırı tanımlamayı ve kullanım kayıtlarını incelemeyi de destekler.

Örneğin:

```yaml
max-ai-credits: 500
```

Bu değer gerçek projede kullanılmadan önce seçilen modelin maliyeti ve istenen görev büyüklüğüne göre ayarlanmalıdır.

### 15. İnsana ne zaman haber verileceğini de otomatikleştir

Tam otomasyonun amacı insanı sistemden tamamen çıkarmak değildir. Amaç, rutin geçişleri agentlara bırakırken **karar veya müdahale gerektiren durumları doğru kişiye görünür hale getirmektir.**

Bu nedenle orkestrasyon akışında bir **insan bildirim / müdahale noktası** da tanımlanmalıdır.

Örneğin:

```text
Agent görevi yürütüyor
        ↓
normal ve doğrulanmış sonuç
        ↓
otomasyon devam eder

AMA

sürüm uyuşmuyor
veya
doğrulama başarısız
veya
izin verilen kapsamın dışına çıkmak gerekiyor
veya
daha fazla yetki gerekiyor
veya
belirlenen tekrar deneme sınırı aşıldı
        ↓
OTOMASYONU DURDUR
        ↓
insana haber ver
        ↓
karar / onay bekle
```

İnsana haber verilmesi gereken durumlar proje riskine göre belirlenebilir. Özellikle şu durumlar iyi adaylardır:

- görev başarıyla tamamlandı ve nihai sonuç hazır;
- workflow veya agent art arda başarısız oldu;
- `INPUT_VERSION` ile `CURRENT_VERSION` uyuşmuyor;
- agent görev kapsamının dışına çıkmak istiyor;
- yeni dosya, servis veya daha yüksek yetki gerekiyor;
- doğrulama sonucu belirsiz veya başarısız;
- otomatik tekrar deneme sınırı tükendi;
- güvenlik açısından insan kararı gerektiren bir durum oluştu.

Bildirim yalnızca **“hata oldu”** dememelidir. Karar verecek kişinin ne olduğunu anlayabilmesi için en az şu bilgileri taşımalıdır:

```text
TASK_ID
STATUS
CURRENT_VERSION
WHAT_HAPPENED
EVIDENCE
WHAT_WAS_TRIED
WHAT_NEEDS_HUMAN_DECISION
SAFE_NEXT_OPTIONS
```

Örneğin:

```text
TASK_ID: UI-024
STATUS: HUMAN_REVIEW_REQUIRED
CURRENT_VERSION: def456

WHAT_HAPPENED:
Claude bulgusundan sonra repository sürümü değişti.

EVIDENCE:
INPUT_VERSION: abc123
CURRENT_VERSION: def456

WHAT_NEEDS_HUMAN_DECISION:
Yeni sürüm üzerinde görevin yeniden başlatılması onaylanmalı mı?
```

Bildirimin nereye gönderileceği kullanılan sisteme göre değişebilir. GitHub üzerinde Issue, Pull Request yorumu veya belirlenmiş başka bir bildirim kanalı kullanılabilir. E-posta, Slack veya benzeri harici bir kanal kullanılacaksa ayrıca o kanala erişebilen bir entegrasyon gerekir.

Her küçük agent hareketinde insana bildirim göndermek yerine **tamamlanma, durma, hata ve karar gerektiren eşikler** için bildirim oluşturmak daha kullanışlıdır. Aksi halde çok fazla bildirim önemli uyarıların gözden kaçmasına neden olabilir.

Bu nedenle orkestrasyon yalnızca:

```text
"Sıradaki agent kim?"
```

sorusunu değil, gerektiğinde:

```text
"Burada otomasyon durmalı mı?"
"İnsana haber verilmeli mi?"
"Devam etmek için insan onayı gerekiyor mu?"
```

sorularını da cevaplamalıdır.

### 16. İlk otomasyonda bütün sistemi birden kurma

İlk hedef yalnızca tek bir geçiş olmalıdır:

```text
Codex değişikliği GitHub'a gönderdi
        ↓
PR oluştu
        ↓
Claude otomatik incelemeye başladı
```

Bu geçiş güvenilir biçimde çalıştıktan sonra:

```text
Claude → Codex düzeltme
Codex → Claude yeniden inceleme
İnceleme → test
Test → doğrulama
```

adımları sırayla eklenebilir.

Her geçişte en az şu bilgiler korunmalıdır:

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

Amaç yalnızca agentları sırayla çalıştırmak değildir.

> **Güvenli otomasyon; doğru agentı başlatmanın yanında, doğru görevin doğru sürüm ve doğru sınırlarla bir sonraki aşamaya geçtiğini de kontrol etmelidir.**

> **Not:** GitHub Agentic Workflows bu rehber hazırlanırken public preview (genel önizleme) durumundadır. Engine (agent motoru), trigger (tetikleyici), permission (izin), safe output (güvenli çıktı) ve kimlik doğrulama seçenekleri kurulum sırasında güncel GitHub dokümantasyonundan tekrar kontrol edilmelidir.

## Ne zaman kullanılır?

Bu yöntem özellikle mevcut bir dosya veya kod üzerinde sınırlı bir düzeltme yapılırken, çalışan bölümlerin korunması gerektiğinde veya agentın yalnızca belirli dosyalara dokunması istendiğinde kullanışlıdır.

Küçük ve kolayca geri alınabilen denemelerde ayrıntılı bir değişiklik sınırı kaydı gerekmeyebilir. Örneğin boş bir deneme dosyasında birkaç farklı metin biçimini karşılaştırırken hangi satırların korunacağını ayrıca tanımlamak çoğu zaman gerekli değildir.
