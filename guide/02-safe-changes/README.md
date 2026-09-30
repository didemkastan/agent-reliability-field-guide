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

Bunun için ortak kurallar tek bir **kanonik proje talimatında** tutulabilir. Basit bir kurulumda `AGENTS.md` dosyası repository'nin ana klasöründe oluşturulabilir:

```text
PROJECT/
├── AGENTS.md          ← bütün agentlar için ortak kurallar
├── [Agent A girişi]   ← AGENTS.md kurallarına bağlanır
├── [Agent B girişi]   ← AGENTS.md kurallarına bağlanır
├── [Agent C girişi]   ← AGENTS.md kurallarına bağlanır
└── ...
```

Bu kutu bir Terminal komutu değildir; repository içindeki örnek dosya yapısını gösterir. `AGENTS.md` dosyasının içine ortak ve kalıcı agent kuralları normal Markdown/metin olarak yazılır.

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

Bu örnekte Gemini GitHub'a bağlı olmadığı için aşağıdaki paket **bir GitHub ayarı veya dosya formatı değildir**. Kullanıcı/orkestratör bu metni Gemini sohbetine prompt olarak verir ve listelenen dosyaları aynı göreve ekler. Otomatik bir dış-agent entegrasyonu kurulursa aynı alanlar API/görev mesajı içinde taşınabilir.

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

Aşağıdaki metin repository'ye yazılmaz. GitHub'a bağlı olmayan Gemini'ye görev verirken **Gemini sohbet/prompt alanına** yazılır; görev paketi ve gerekli dosyalar da aynı konuşmaya eklenir.

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

### Bu bölümdeki kutular nasıl okunmalı?

Bu bölümde farklı amaçlarla kod ve metin kutuları kullanılır. **Her kutu bir yere yazılacak ayar değildir.**

- ```text``` kutuları çoğunlukla akışı veya örnek görev kaydını anlatır. Altında açıkça “şu dosyaya yazın” denmiyorsa bunları kopyalamanız gerekmez.
- ```bash``` kutuları bilgisayarınızdaki **Terminal / PowerShell** içinde çalıştırılan komutlardır.
- ```markdown``` ve ```yaml``` kutuları gerçek bir workflow ayarı gösteriyorsa, örneğin hemen üstünde **hangi dosyaya yazılacağı** belirtilir.
- GitHub Agentic Workflows kaynak dosyaları genellikle repository içindeki `.github/workflows/<ad>.md` dosyalarıdır. Bu dosyalardaki `---` işaretleri arasındaki bölüm **frontmatter (workflow ayar bölümü)**, altındaki normal Markdown metni ise **agenta verilecek görev talimatıdır**.
- Frontmatter içindeki trigger (tetikleyici), permission (izin), engine (agent motoru), safe output (güvenli çıktı) veya maliyet ayarı değiştirildiğinde repository klasöründeki Terminal/PowerShell'de `gh aw compile` çalıştırılarak ilgili `.lock.yml` dosyası güncellenir. Kaynak `.md` ve üretilen `.lock.yml` birlikte GitHub'a gönderilir.


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

Örneğin ChatGPT'nin normal bir sohbet içinde GitHub'a erişebilmesi, GitHub'daki her değişiklikten sonra bu sohbetin kendiliğinden çalışacağı anlamına gelmez.

Ancak ChatGPT'de bunun için ayrı bir yol vardır: uygun hesaplarda **Work içinde GitHub olaylarıyla tetiklenen görevler** oluşturulabilir. GitHub hesabı ChatGPT'ye bağlandıktan sonra desteklenen Pull Request olayları bir ChatGPT görevini otomatik başlatabilir. Bu, yalnızca “GitHub'a erişim” vermekten farklıdır; ayrıca bir **tetikleyici + koşul + görev talimatı** tanımlanır.

Aynı şekilde bir Codex veya Claude bağlantısının bulunması da tek başına otomatik agent geçişi oluşturmaz.

GitHub Agentic Workflows içinde GitHub Copilot, Claude Code, OpenAI Codex ve Google Gemini doğrudan **engine (çalıştırılacak agent motoru)** olarak seçilebilir. ChatGPT orkestratör olarak kullanılacaksa iki katman birlikte düşünülebilir: GitHub Agentic Workflows agent çalışmalarını yürütür; ChatGPT'nin olayla tetiklenen görevi ise desteklenen GitHub olaylarında durumu okuyup kontrol veya insan bildirimi görevi üstlenebilir. ChatGPT'nin sonraki agentı gerçekten başlatması isteniyorsa, bağlı araçların bu işlemi yapmaya izin vermesi ve gerekli yetkilerin ayrıca verilmiş olması gerekir.

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

Codex, Claude veya Gemini gibi bir AI engine (AI motoru) API anahtarı kullanacaksa gerçek anahtar **repository dosyalarına, kaynak koda veya workflow metnine yazılmaz.** Anahtar GitHub'ın **Actions secrets (gizli değerler)** alanında saklanır.

#### Anahtar nereden alınır?

Kullanılan AI hizmetinin kendi hesabından bir API anahtarı oluşturulur:

```text
Codex   → OpenAI hesabından API anahtarı
Claude  → Anthropic hesabından API anahtarı
Gemini  → Google AI Studio'dan API anahtarı
```

Ardından bu değer GitHub'a secret olarak kaydedilir.

#### GitHub'da nereye yazılır?

GitHub web sitesinde otomasyonun çalışacağı repository'yi açın ve şu yolu izleyin:

```text
Repository
   ↓
Settings
   ↓
Secrets and variables
   ↓
Actions
   ↓
New repository secret
```

Açılan ekranda iki temel alan bulunur:

```text
Name
→ workflow'un anahtarı hangi adla bulacağını belirtir

Secret
→ AI sağlayıcısından aldığınız gerçek API anahtarıdır
```

Örneğin Claude için:

```text
Name:
ANTHROPIC_API_KEY

Secret:
[Anthropic hesabından aldığınız gerçek anahtar]
```

Codex için:

```text
Name:
OPENAI_API_KEY

Secret:
[OpenAI hesabından aldığınız gerçek anahtar]
```

Gemini için:

```text
Name:
GEMINI_API_KEY

Secret:
[Google AI Studio'dan aldığınız gerçek anahtar]
```

Son olarak **Add secret** seçilir.

GitHub Agentic Workflows için güncel temel adlar şunlardır:

```text
Codex   → OPENAI_API_KEY veya CODEX_API_KEY
Claude  → ANTHROPIC_API_KEY
Gemini  → GEMINI_API_KEY
```

Codex tarafında `CODEX_API_KEY` ve `OPENAI_API_KEY` desteklenir; ikisi de tanımlıysa `CODEX_API_KEY` öncelikli kullanılır.

#### Workflow anahtarı nasıl kullanır?

Mantık şöyledir:

```text
AI sağlayıcısından anahtar alınır
        ↓
GitHub Actions Secrets alanına kaydedilir
        ↓
workflow çalışır
        ↓
GitHub secret değerini çalışma sırasında güvenli biçimde sağlar
        ↓
AI engine kimlik doğrulaması yapılır
```

Böylece gerçek anahtar repository'deki normal dosyalarda görünmez.

Gerçek anahtar değeri şu alanlara **yazılmamalıdır**:

```text
AGENTS.md
workflow'un normal metni
görev paketi
README
kaynak kod
commit mesajı
Issue / Pull Request yorumu
```

Anahtarı Terminal/PowerShell üzerinden kaydetmek de mümkündür; ancak bu rehberde yeni başlayanlar için GitHub web arayüzündeki yol esas alınmıştır.

> **GitHub Copilot farklı çalışabilir:** Kuruluşa bağlı GitHub Copilot kullanımında `copilot-requests: write` izniyle ayrıca bir AI sağlayıcı API anahtarı gerekmeyebilir. Kullanılan engine ve hesap yapısına göre güncel kimlik doğrulama yöntemi kontrol edilmelidir.

Buradaki API anahtarı **AI hizmetine kimlik doğrulamak** içindir. Agentın GitHub'da dosya okuma, PR güncelleme, yorum ekleme veya başka workflow başlatma yetkileri ise ayrıca GitHub izinleri ve workflow ayarlarıyla kontrol edilir.

### 4. Codex işi bitirdiğinde Claude değişikliği nasıl görür?

Codex'in yalnızca **“işim bitti”** demesi yeterli değildir. Değişiklik commit/branch/Pull Request yoluyla GitHub'a ulaşmalıdır.

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

Yukarıdaki kutu yalnızca akışı gösterir; herhangi bir dosyaya yazılmaz.

Claude'un bir Pull Request açıldığında otomatik başlaması isteniyorsa bunun için **gerçek bir workflow dosyası oluşturulur**. Bu örnekte dosya:

```text
.github/workflows/review.md
```

olacaktır. Dosyayı repository içinde siz oluşturabilirsiniz veya repository'ye erişebilen bir coding agenttan oluşturmasını isteyebilirsiniz.

Aşağıdaki örnek **`.github/workflows/review.md` dosyasının içeriğidir**:

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

Bu dosyanın iki bölümü vardır:

```text
--- ile --- arasındaki bölüm
→ GitHub'a workflow'un ne zaman ve nasıl çalışacağını söyler.

# Review ile başlayan bölüm
→ Claude'a çalıştığında ne yapacağını söyler.
```

Örnekteki ayarların anlamı:

- `on: pull_request` → bu workflow Pull Request olaylarını takip eder.
- `opened` → PR ilk açıldığında workflow'u başlatır.
- `synchronize` → aynı PR'a yeni commit gönderildiğinde workflow'u yeniden başlatır.
- `engine: claude` → bu görevi Claude engine'inin çalıştıracağını belirtir.
- `contents: read` ve `pull-requests: read` → Claude'un kodu ve PR bilgisini okuyabilmesi için okuma izinlerini tanımlar.
- `safe-outputs: add-comment` → agentın sonucunun kontrollü biçimde PR yorumu olarak yazılabilmesine izin verir.

**Neden bu dosyayı oluşturuyoruz?** Çünkü Claude'un GitHub'da sürekli bekleyip yeni PR'ları kendiliğinden fark ettiği varsayılmaz. Bu workflow, GitHub'a açıkça **“PR açılırsa veya bu PR'a yeni commit gelirse Claude incelemesini başlat”** kuralını verir.

Frontmatter oluşturulduğu veya değiştirildiği için repository klasöründe Terminal/PowerShell açıp:

```bash
gh aw compile .github/workflows/review.md
```

çalıştırılır. Bunun sonucunda oluşan `review.lock.yml` ile kaynak `review.md` birlikte commit edilip GitHub'a gönderilir. GitHub Actions'ın çalıştırdığı dosya derlenmiş `.lock.yml` sürümüdür.

Pull Request olayı geldiğinde workflow ilgili GitHub bağlamını kullanır. Bu nedenle Claude, Codex'in belleğine bakmaz; **GitHub'da kaydedilmiş PR ve kod değişikliğini okur.**

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

Claude'un bir sorun bulması, Codex'in kendiliğinden çalışacağı anlamına gelmez. GitHub'a **hangi workflow'un sırada başlayacağını** ayrıca söylemek gerekir.

#### Yöntem A — Sonraki workflow'u doğrudan başlatmak

Bu örnekte iki ayrı dosya vardır:

```text
.github/workflows/review.md  → Claude incelemesi
.github/workflows/fix.md     → Codex düzeltmesi
```

`review.md` içindeki frontmatter'a, Claude'un gerektiğinde yalnızca `fix` workflow'unu başlatabilmesine izin veren şu ayar eklenir:

```yaml
safe-outputs:
  dispatch-workflow:
    workflows: [fix]
    max: 1
```

Bu YAML parçası **Terminal'e yazılmaz** ve tek başına ayrı bir dosya değildir. `.github/workflows/review.md` dosyasının en üstündeki `---` işaretleri arasındaki ayar bölümüne eklenir.

`workflows: [fix]` yalnızca `fix` adlı workflow'un çağrılmasına izin verir. `max: 1` ise bu çalışmada en fazla bir kez böyle bir geçiş yapılmasına izin verir.

Diğer tarafta `.github/workflows/fix.md` dosyasının da `workflow_dispatch` ile başlatılmayı kabul edecek şekilde tanımlanmış olması gerekir. Yani `review.md` **“fix'i başlatabilirim”**, `fix.md` ise **“başka bir workflow beni başlatabilir”** tarafını oluşturur.

Bunun için `.github/workflows/fix.md` dosyasının en üstündeki frontmatter'da en azından tetikleyici tarafı şu mantıkta bulunur:

```markdown
---
on:
  workflow_dispatch:

engine: codex
---

# Fix

Aktarılan görevi ve bulguyu güncel GitHub sürümünde doğrula.
Geçerliyse izin verilen kapsam içinde gerekli en küçük düzeltmeyi hazırla.
```

Bu örnek yalnızca **fix workflow'unun dışarıdan başlatılabilmesini ve Codex engine'ini seçmeyi** gösterir. Gerçek projede ihtiyaç duyulan okuma/yazma izinleri, safe outputs ve görev inputları ayrıca tanımlanmalıdır; bu kısa örnek onları otomatik olarak sağlamaz.

Akış:

```text
review.md → Claude sorun buldu
        ↓
dispatch-workflow
        ↓
fix.md başlatıldı
        ↓
Codex görevi aldı
```

Bu akış kutusu açıklama amaçlıdır; dosyaya kopyalanmaz.

Geçişte yalnızca “sorun var” bilgisi taşınmamalıdır. `TASK_ID`, PR numarası, Claude'un incelediği commit, bulgu ve kanıt gibi bilgiler `fix` workflow'una aktarılmalıdır. Böylece Codex hangi görevin hangi sürümüne ait bulguyu aldığını bilir.

`review.md` veya `fix.md` frontmatter'ı değiştirildikten sonra Terminal/PowerShell'de ilgili workflow yeniden derlenir:

```bash
gh aw compile .github/workflows/review.md
gh aw compile .github/workflows/fix.md
```

Oluşan `.lock.yml` dosyaları da kaynak `.md` dosyalarıyla birlikte GitHub'a gönderilir.

#### Yöntem B — Etiketi durum veya komut olarak kullanmak

GitHub etiketi bir durumu göstermek için kullanılabilir:

```text
agent:fix-required
= düzeltme gerekiyor
```

Bu kutu yalnızca örnek etiket adını gösterir. Etiket, GitHub repository'sinin **Issues / Pull Requests label (etiket)** sistemi içinde oluşturulur; kaynak kod dosyasına yazılmaz.

Bir etiketin gerçekten workflow başlatması isteniyorsa yalnız etiketi oluşturmak yetmez. İlgili workflow'un `.github/workflows/<workflow-adı>.md` dosyasındaki frontmatter'da `label_command` gibi bir tetikleyici ayrıca tanımlanmalıdır.

Örneğin `agent:fix-required` etiketi `fix.md` workflow'unu başlatacaksa tetikleyici `.github/workflows/fix.md` dosyasının frontmatter'ına yazılır:

```yaml
on:
  label_command:
    name: agent:fix-required
    events: [pull_request]
```

Bu YAML GitHub web sitesindeki etiket açıklamasına veya Terminal'e yazılmaz; **`fix.md` dosyasının ayar bölümüdür.**

Ancak GitHub'ın varsayılan `GITHUB_TOKEN` kimliğiyle yapılan bazı otomatik yazma işlemleri yeni workflow/CI çalışmaları başlatmaz. Bu nedenle **“Claude etiketi ekledi, Codex kesin otomatik başlar”** varsayımı yapılmamalıdır.

Başlangıçta daha kolay izlenen yöntem şudur:

```text
DURUMU GÖSTER
→ yorum / etiket

SIRADAKİ AGENTI GERÇEKTEN BAŞLAT
→ review.md içindeki dispatch-workflow ayarı
```

İlk satır insana durumu görünür kılar; ikinci satır ise teknik olarak sonraki workflow'u başlatır.

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

Bu bilgilerin aşağıdaki gibi yazılması **GitHub'ın kendi ayarı değildir**. Bunlar Codex'e verilecek görev kaydıdır. Manuel kullanımda Codex promptuna eklenebilir; otomatik kullanımda ise `fix.md` workflow'una input/görev bağlamı olarak taşınabilir.

Örnek görev kaydı:

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

### 9. Test aşamasında AI agent mı, normal otomatik test mi kullanılmalı?

Her test için yeni bir AI agent çalıştırmak gerekmez. Önce şu soruyu sorun:

> **Bu kontrolün doğru veya yanlış sonucu bir program tarafından kesin olarak belirlenebilir mi, yoksa sonucu anlamak için yorum yapmak mı gerekiyor?**

İki farklı kontrol türünü ayıralım.

#### 1. Sonucu kesin olan kontroller

Bazı kontrollerin cevabı nettir. Örneğin:

```text
Unit test geçti mi?
→ EVET / HAYIR

Uygulama build edilebildi mi?
→ EVET / HAYIR

Kodda lint hatası var mı?
→ VAR / YOK

Type check geçti mi?
→ EVET / HAYIR
```

Bunlar için bir AI agentın kodu okuyup karar vermesine gerek yoktur. Mevcut test komutları, scriptler veya **GitHub Actions** bu kontrolleri otomatik olarak çalıştırabilir.

Örneğin Python projesi testleri zaten `pytest` ile çalıştırıyorsa, geliştirici bunu kendi bilgisayarında repository klasöründeki Terminal/PowerShell'de manuel olarak çalıştırabilir:

```bash
pytest
```

Otomasyonda ise `pytest` komutunu her seferinde insanın Terminal'e yazması beklenmez. Komut, repository içindeki normal bir **GitHub Actions test workflow'una** eklenir; örneğin:

```text
.github/workflows/tests.yml
```

Bu `tests.yml`, AI agent talimatı olan `review.md` veya `fix.md` ile aynı şey değildir. `tests.yml` GitHub Actions'a **“kod geldiğinde şu kesin test komutunu çalıştır”** der. Böylece GitHub testi otomatik çalıştırır ve başarılı/başarısız sonucunu üretir.

Mantık:

```text
Codex değişikliği yaptı
        ↓
GitHub'a gönderildi
        ↓
normal otomatik test çalıştı
        ↓
 ┌──────────────────────┐
 │                      │
TEST GEÇTİ           TEST KALDI
 │                      │
 ▼                      ▼
sonraki kontrol      Codex'e düzeltme görevi
```

Buradaki **normal otomatik test**, AI değildir. Projenin zaten kullandığı test aracının GitHub Actions tarafından otomatik çalıştırılmasıdır.

#### 2. Yorum gerektiren kontroller

Bazı soruların cevabı yalnızca bir test komutunun `PASS` veya `FAIL` sonucu ile anlaşılamaz.

Örneğin:

```text
Kullanıcının istediği davranış gerçekten uygulanmış mı?

Değişiklik görev kapsamının dışına taşmış mı?

Çalışan mevcut davranış gereksiz yere değiştirilmiş mi?

Kod teknik olarak çalışsa bile istek yanlış yorumlanmış olabilir mi?

Ekran veya kullanıcı akışı beklenen davranışla uyumlu mu?
```

Bu tür kontrollerde bağlamı okuyup değerlendirebilen **Claude, Gemini veya başka bir AI agent** kullanılabilir.

Dolayısıyla AI agent ile normal test birbirinin yerine geçen iki seçenek değildir. Çoğu güvenilir akışta ikisi birlikte kullanılır:

```text
Kod değişikliği
      ↓
NORMAL OTOMATİK KONTROLLER
unit test / build / lint / type check
      ↓
başarılı
      ↓
AI İNCELEMESİ
istek doğru uygulanmış mı?
kapsam korunmuş mu?
bağlamsal bir sorun var mı?
      ↓
doğrulama
```

Sıra projeye göre değişebilir. Örneğin hızlı otomatik kontrolleri AI incelemesinden önce çalıştırmak yararlıdır. Kod daha temel kontrollerden geçmiyorsa — örneğin proje derlenemiyor veya mevcut testler başarısız oluyorsa — önce bu teknik sorun görülür. Böylece henüz temel kontrolleri geçemeyen bir değişiklik için gereksiz yere AI incelemesi başlatılmaz.

Kısaca:

```text
Cevabı bir komut kesin olarak verebiliyor mu?
        ↓
EVET → normal test / script / GitHub Actions

Yorum, bağlam veya değerlendirme gerekiyor mu?
        ↓
EVET → AI agent
```

Amaç AI agentı testten çıkarmak değildir. **Kesin sonucu mevcut araçların verebildiği işi AI'a yaptırmamak; AI'ı yorumlama ve değerlendirme gereken yerde kullanmaktır.**

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

ChatGPT orkestratör olarak kullanılacaksa yalnızca normal GitHub bağlantısına güvenilmez. Uygun hesaplarda **ChatGPT Work içinde GitHub olaylarıyla tetiklenen görev** oluşturularak desteklenen Pull Request olaylarında ChatGPT'nin otomatik çalışması sağlanabilir. Böylece ChatGPT, olay gerçekleştiğinde GitHub'daki durumu okuyup orkestrasyon kontrolünü veya insan bildirimini çalıştırabilir. Sonraki agentı doğrudan başlatması gerekiyorsa kullanılan bağlantıların ve izinlerin bu eylemi de desteklemesi gerekir.

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

Görev sınırı ayrıca açıkça yazılabilir:

```text
ALLOWED_FILES:
- src/navigation/**
- tests/navigation/**

PROTECTED:
- AGENTS.md
- .github/**
- dependency / package dosyaları
```

Bu kutu **GitHub'ın yerleşik bir izin ayarı değildir**. `ALLOWED_FILES` ve `PROTECTED` burada agenta verilen görev talimatının alanlarıdır. Manuel akışta agent promptuna, otomatik akışta ise ilgili `review.md` / `fix.md` dosyasının görev metnine veya taşınan görev kaydına yazılır.

Bu talimat agentın ne yapması gerektiğini sınırlar; tek başına teknik erişim kontrolü sağlamaz. Kritik alanların gerçekten değiştirilememesi gerekiyorsa GitHub permissions, branch protection, safe outputs ve kullanılan araçların gerçek izinleriyle ayrıca teknik sınır uygulanmalıdır.

Agentın görev için gerek duymadığı workflow, güvenlik, talimat veya bağımlılık dosyalarını değiştirebilmesi otomatik olarak açılmamalıdır.

### 12. Aynı işin eski ve yeni sürümü aynı anda incelenmesin

Bu sorun en kolay bir örnekle anlaşılır.

Claude bir Pull Request içindeki **A sürümünü** inceliyor olsun. Claude incelemeyi bitirmeden Codex aynı Pull Request'e yeni bir commit gönderdi ve artık **B sürümü** oluştu:

```text
Claude
A sürümünü inceliyor
        ↓
inceleme henüz bitmedi

bu sırada

Codex yeni commit gönderdi
        ↓
Pull Request artık B sürümünde
```

Claude A sürümü için incelemeyi tamamlamaya devam ederse sonucu daha oluştuğu anda eski kalabilir. Çünkü GitHub'daki güncel kod artık B sürümüdür.

İşte **concurrency (aynı işin eşzamanlı çalışmalarını yönetme)** kontrolünün amacı budur: aynı Pull Request için eski ve yeni incelemelerin birbirine karışmasını önlemek.

GitHub Agentic Workflows, Pull Request ile tetiklenen çalışmalar için bu durumu zaten yönetir. Aynı PR'a yeni commit geldiğinde eski sürüme ait çalışma iptal edilerek güncel sürüm için yeni çalışma devam edebilir.

```text
Claude A sürümünü inceliyor
        ↓
Codex B sürümünü GitHub'a gönderdi
        ↓
A sürümüne ait inceleme artık güncel değil
        ↓
eski çalışma durdurulur
        ↓
Claude B sürümünü inceler
```

Bu temel kullanımda kullanıcının ayrıca bir `concurrency:` kodu yazması gerekmez. Bu nedenle önceki `group ... cancel-in-progress` örneği başlangıç anlatımından çıkarılmıştır.

Özel çalışma sıraları gereken ileri seviye projelerde concurrency ayarları değiştirilebilir. Ancak standart Pull Request inceleme akışında önce GitHub Agentic Workflows'un varsayılan davranışı kullanılmalıdır.

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

Agentic Workflows çalışma başına AI kullanım sınırı tanımlamayı da destekler.

Bu ayar **Terminal'e veya GitHub Settings ekranına yazılmaz**. Sınır hangi agentic workflow için geçerliyse o dosyanın, örneğin `.github/workflows/review.md` dosyasının en üstündeki **frontmatter** bölümüne eklenir:

```yaml
---
on:
  pull_request:
    types: [opened, synchronize]

engine: claude
max-ai-credits: 500
---
```

`max-ai-credits: 500`, bu workflow'un tek bir çalışması için AI kullanım bütçesine üst sınır koyan bir güvenlik ayarıdır. Buradaki `500` yalnızca örnektir; gerçek değer seçilen model ve görev büyüklüğüne göre belirlenmelidir.

Frontmatter değiştiği için sonrasında Terminal/PowerShell'de:

```bash
gh aw compile .github/workflows/review.md
```

çalıştırılır ve güncellenen `.md` ile `.lock.yml` GitHub'a gönderilir.

### 15. ChatGPT orkestratörse agentlar arasındaki akışı ortak görev dosyalarından yönet

ChatGPT orkestratör olarak kullanıldığında, kendi projemizde çalışan agentların sonuçları ortak ve izlenebilir bir yerde tutulur; ChatGPT bu kayıtları okuyarak sıradaki adımı belirler.

Agentların birbirlerinin sohbet belleğini görmesi gerekmez. Bunun yerine repository içinde orkestrasyona ayrılmış küçük bir alan kullanılabilir.

Örneğin:

```text
PROJECT/
├── orchestration/
│   ├── current-task.md
│   └── handoffs/
│       ├── TASK-001-codex.md
│       ├── TASK-001-claude.md
│       └── TASK-001-test.md
│
├── src/
└── tests/
```

Bu kutu bir komut değildir. Repository içinde oluşturulabilecek örnek klasör yapısını gösterir.

Buradaki dosyaların görevi:

- `orchestration/current-task.md` → görevin şu anda hangi aşamada olduğunu gösteren **kanonik görev durumu**;
- `orchestration/handoffs/TASK-001-codex.md` → Codex'in yaptığı işin ve bıraktığı kanıtların kaydı;
- `orchestration/handoffs/TASK-001-claude.md` → Claude incelemesinin sonucu;
- `orchestration/handoffs/TASK-001-test.md` → test/doğrulama sonucu.

Örneğin Codex işi bitirdiğinde kendi handoff kaydına şunları bırakabilir:

```text
TASK_ID: TASK-001
SOURCE_AGENT: Codex
INPUT_VERSION: abc123
OUTPUT_VERSION: def456
STATUS: IMPLEMENTATION_COMPLETE

CHANGED:
- src/navigation.py

PRESERVE:
- mevcut giriş akışı

VERIFIED:
- mevcut otomatik testler geçti

SKIPPED_CHECKS:
- görsel kontrol yapılmadı

NEXT_AGENT: Claude
NEXT_ACTION: değişikliği görev sınırlarına göre incele
```

Bu kayıt Codex'in belleği değildir. **Repository'de saklanan ortak proje kaydıdır.** Bu nedenle ChatGPT, Claude veya başka bir agent aynı göreve daha sonra katıldığında önceki agentın sohbet geçmişine ihtiyaç duymaz.

#### ChatGPT burada ne yapar?

ChatGPT orkestratörün görevi agentların yaptığı işi tekrar yapmak değil, ortak kaydı okuyup geçişin güvenli olup olmadığını kontrol etmektir.

```text
Codex işi tamamladı
        ↓
Codex handoff dosyasını güncelledi
        ↓
current-task.md güncellendi
        ↓
ChatGPT orkestratör kaydı okudu
        ↓
INPUT_VERSION / OUTPUT_VERSION / STATUS / kanıt kontrolü
        ↓
 ┌───────────────────────────────┐
 │                               │
geçiş güvenli                sorun var
 │                               │
 ▼                               ▼
NEXT_AGENT çalıştırılır      otomasyon durur
ör. Claude                   insana haber verilir
```

Burada ChatGPT'nin baktığı temel bilgi **bizim görev dosyalarımızdır**. Pull Request yorumu, Issue veya agent sohbet belleği kanonik görev durumu olarak kullanılmaz.

#### ChatGPT otomasyonu bu dosyaları nasıl takip eder?

İki farklı yöntem kullanılabilir.

**Yöntem 1 — Zamanlanmış / izleme görevi**

ChatGPT'de GitHub uygulaması bağlandıktan sonra bir zamanlanmış görev belirli aralıklarla kendi repository'mizdeki orkestrasyon kayıtlarını kontrol edebilir.

Örnek görev mantığı:

```text
KONTROL ET:
orchestration/current-task.md

EĞER:
STATUS yeni bir aşamaya geçtiyse

DOĞRULA:
- TASK_ID doğru mu?
- INPUT_VERSION beklenen sürüm mü?
- gerekli handoff dosyası var mı?
- VERIFIED alanında kanıt var mı?
- SKIPPED_CHECKS kabul edilebilir mi?

SONRA:
- güvenliyse NEXT_AGENT / NEXT_ACTION adımını uygula;
- insan kararı gerekiyorsa otomasyonu durdur ve bana bildir;
- hiçbir şey değişmediyse işlem yapma.
```

Bu metin repository'deki bir workflow dosyasına yazılmaz. ChatGPT'de oluşturulan **zamanlanmış/izleme görevinin talimatıdır.** Görev, bağlı GitHub uygulamasına verilen erişim kapsamında repository bilgisini okuyabilir.

**Yöntem 2 — GitHub olayı ChatGPT'yi uyandırsın**

Güncel ChatGPT olayla tetiklenen GitHub görevleri desteklenen **Pull Request etkinlikleri** ile başlayabilir. Proje zaten agent değişikliklerini PR üzerinden taşıyorsa bu olay yalnızca ChatGPT'yi hemen çalıştıran bir **uyandırma sinyali** olarak kullanılabilir.

Bu durumda ChatGPT'nin görevi PR'a yorum yazmak değildir:

```text
PR etkinliği oluştu
        ↓
ChatGPT görevi başladı
        ↓
orchestration/current-task.md dosyasını oku
        ↓
ilgili handoff kaydını oku
        ↓
sürüm + durum + kanıtı doğrula
        ↓
sıradaki agent / durma / insan bildirimi kararını ver
```

Yani:

> **PR olayı tetikleyici olabilir; orkestrasyon bilgisinin kaynağı bizim görev ve handoff dosyalarımızdır.**

ChatGPT'nin GitHub olay tetikleyicileri her dosya değişikliğini doğrudan dinleyen genel bir repository webhook'u değildir. Bu nedenle PR kullanılmayan bir yapıda **zamanlanmış izleme görevi** daha anlaşılır başlangıç seçeneğidir.

#### İnsan ne zaman devreye girer?

ChatGPT orkestratör rutin ve doğrulanmış geçişleri kendi kurallarına göre sürdürebilir. Ancak örneğin şu durumlarda akışı durdurup kullanıcıya haber vermelidir:

- `INPUT_VERSION` ile güncel sürüm uyuşmuyor;
- gerekli handoff dosyası yok;
- doğrulama başarısız veya belirsiz;
- `SKIPPED_CHECKS` içinde kritik bir kontrol atlanmış;
- agent izin verilen kapsamın dışına çıkmak istiyor;
- daha yüksek yetki gerekiyor;
- tekrar deneme sınırı dolmuş;
- `NEXT_AGENT` veya `NEXT_ACTION` belirsiz.

Bildirim yalnızca **“hata oldu”** dememelidir. En azından `TASK_ID`, güncel sürüm, ne olduğu, kanıt, neyin denendiği ve kullanıcıdan hangi kararın beklendiği gösterilmelidir.

Böylece insan her agent geçişini elle taşımak zorunda kalmaz; yalnızca otomasyonun güvenle karar veremediği noktada devreye girer.

### 16. İlk otomasyonda yalnızca tek agent geçişini kur

İlk denemede bütün agent zincirini aynı anda otomatikleştirmek yerine yalnızca bir geçişi çalıştırmak daha güvenlidir.

Örneğin ilk hedef:

```text
Codex görevi tamamladı
        ↓
orchestration/handoffs/TASK-001-codex.md oluştu
        ↓
orchestration/current-task.md
NEXT_AGENT: Claude oldu
        ↓
ChatGPT orkestratör kaydı kontrol etti
        ↓
Claude incelemesi başlatıldı
```

Bu tek geçiş güvenilir biçimde çalıştıktan sonra sırayla:

```text
Claude → Codex düzeltme
Codex → Claude yeniden inceleme
İnceleme → normal otomatik testler
Test → doğrulama
Doğrulama → tamamlandı / insan kararı
```

adımları eklenebilir.

Her geçişte en az şu bilgiler korunmalıdır:

```text
TASK_ID
INPUT_VERSION
CURRENT_VERSION
SOURCE_AGENT
STATUS
NEXT_AGENT
FINDINGS
EVIDENCE
PRESERVE
SKIPPED_CHECKS
NEXT_ACTION
```

Amaç yalnızca agentları sırayla çalıştırmak değildir.

> **Güvenli otomasyon; doğru görevin doğru sürüm, kanıt ve sınırlarla bir sonraki agenta geçtiğini kontrol etmelidir.**

> **Not:** GitHub Agentic Workflows ve ChatGPT'nin bağlı uygulama/görev özellikleri değişebilen ürün özellikleridir. Engine (agent motoru), trigger (tetikleyici), permission (izin), safe output (güvenli çıktı), bağlı uygulama ve görev seçenekleri gerçek kurulum sırasında güncel ürün dokümantasyonundan tekrar kontrol edilmelidir.

## Ne zaman kullanılır?

Bu yöntem özellikle mevcut bir dosya veya kod üzerinde sınırlı bir düzeltme yapılırken, çalışan bölümlerin korunması gerektiğinde veya agentın yalnızca belirli dosyalara dokunması istendiğinde kullanışlıdır.

Küçük ve kolayca geri alınabilen denemelerde ayrıntılı bir değişiklik sınırı kaydı gerekmeyebilir. Örneğin boş bir deneme dosyasında birkaç farklı metin biçimini karşılaştırırken hangi satırların korunacağını ayrıca tanımlamak çoğu zaman gerekli değildir.
