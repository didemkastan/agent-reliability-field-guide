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

Burada iki farklı yön bilgisi vardır: **`RETURN_TO`**, Gemini'nin sonucunu ilk olarak kime teslim edeceğini; **`NEXT_AGENT`** ise orkestratörün doğrulama veya uygulama için görevi daha sonra hangi agenta yönlendireceğini gösterir.

Bu örnekte Gemini'nin yanıtı önce **ChatGPT / orkestratöre** geri verilir (`RETURN_TO`). ChatGPT `TASK_ID`, `INPUT_VERSION`, bulgu, kanıt ve yapılmayan kontrolleri koruyarak yeni görev paketini hazırlar ve ardından Codex'e (`NEXT_AGENT`) aktarır. Gemini'nin çıktısı doğrudan kod değişikliği talimatı olarak kullanılmaz.

#### Gemini sonucunu Codex'e aktarma

Codex'e yalnızca “Gemini hata buldu, düzelt” demek yeterli değildir. Bulguyla birlikte hangi sürümün incelendiği ve hangi kontrollerin yapılmadığı da aktarılmalıdır.

Örnek prompt:

> **TASK_ID UI-024 için dış tester aşağıdaki bulguyu `commit abc123` üzerinde bildirdi. Önce GitHub'daki güncel branch/commit durumunu kontrol et. Güncel sürüm `abc123` ile aynı değilse bulguyu doğrudan uygulama; değişen dosyaları yeniden incele ve bulgunun hâlâ geçerli olup olmadığını doğrula.**
>
> **Bulguyu güncel kod üzerinde yeniden üretmeye veya mevcut testlerle doğrulamaya çalış. Sorun doğrulanırsa AGENTS.md kurallarına ve görevdeki PRESERVE sınırlarına uyarak gerekli en küçük düzeltmeyi yap. Sorun doğrulanmazsa kodu değiştirme ve kanıtıyla birlikte bildir.**
>
> **Dış tester sonucu:**
>
> ```text
> TASK_ID: UI-024
> INPUT_VERSION: commit abc123
> FINDING: Geri dönüş akışında seçilen tema bilgisinin kaybolma riski var.
> EVIDENCE: settings_screen.py geri dönüş işlemi; mevcut testte durum kontrolü yok.
> SKIPPED_CHECKS: Uygulama çalıştırılmadı; repository'nin diğer dosyaları incelenmedi.
> ```
>
> **İşlem sonunda CURRENT_VERSION, CHANGED, PRESERVED, VERIFIED, SKIPPED_CHECKS ve SCOPE_STATUS bilgilerini döndür.**

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

## Ne zaman kullanılır?

Bu yöntem özellikle mevcut bir dosya veya kod üzerinde sınırlı bir düzeltme yapılırken, çalışan bölümlerin korunması gerektiğinde veya agentın yalnızca belirli dosyalara dokunması istendiğinde kullanışlıdır.

Küçük ve kolayca geri alınabilen denemelerde ayrıntılı bir değişiklik sınırı kaydı gerekmeyebilir. Örneğin boş bir deneme dosyasında birkaç farklı metin biçimini karşılaştırırken hangi satırların korunacağını ayrıca tanımlamak çoğu zaman gerekli değildir.
