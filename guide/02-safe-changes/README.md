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

Amaç basit:

```text
Bir agent işi bitirir
        ↓
GitHub bunu fark eder
        ↓
sıradaki görev başlatılır
        ↓
sonraki agent çalışır
```

Böylece kullanıcının sürekli **“Codex bitti, şimdi Claude'a geç”** veya **“Claude sorun buldu, tekrar Codex'e dön”** demesi gerekmez.

### Önce geçişin mantığını anlayalım

Bir agentın GitHub repository'sine (proje deposuna) bağlı olması, diğer agentı kendiliğinden çalıştıracağı anlamına gelmez.

Arada geçişi başlatacak bir mekanizma gerekir. Bu örnekte bu işi **GitHub Actions** yapar.

GitHub'da bir olay olduğunda — örneğin yeni bir Pull Request (PR) açıldığında, PR'a yeni bir commit geldiğinde veya belirli bir etiket eklendiğinde — GitHub Actions ilgili iş akışını başlatabilir.

```text
GitHub'da bir olay oluşur
        ↓
GitHub Actions bunu görür
        ↓
hangi işin başlayacağı belirlenir
        ↓
ilgili agent çalışır
        ↓
sonuç tekrar GitHub'a gelir
        ↓
bir sonraki geçiş başlar
```

Burada iki şey birbirinden ayrılmalıdır:

- **GitHub bağlantısı:** Agentın proje dosyalarını okuyabilmesini veya yetkisi varsa değiştirebilmesini sağlar.
- **Otomatik tetikleme:** Bir olay olduğunda sıradaki işi kendiliğinden başlatır.

Agentın otomatik başlatılabilmesi için kullandığımız agent hizmetinin de dışarıdan çalıştırılmayı desteklemesi gerekir. Bu destek API, komut satırı aracı veya GitHub ile çalışan hazır bir entegrasyon üzerinden sağlanabilir.

GitHub'ın **Agentic Workflows** özelliği bu tür akışları GitHub Actions üzerinde kurmak için kullanılabilen seçeneklerden biridir. Bu özellik bu rehber hazırlanırken **public preview (genel önizleme)** durumundadır. Bu nedenle aşağıdaki kurulum örnekleri kullanılmadan önce güncel GitHub dokümantasyonu kontrol edilmelidir.

### Örnek akış: Codex → Claude → Codex

Örneğimizde Codex kod değişikliğini yapıyor, Claude değişikliği inceliyor. Claude sorun bulursa görev yeniden Codex'e dönüyor.

```text
Görev
  ↓
Codex değişikliği yapar
  ↓
Pull Request açılır
  ↓
GitHub Actions devreye girer
  ↓
Claude değişikliği inceler
  ↓
 ┌─────────────────────┐
 │                     │
sorun var           sorun yok
 │                     │
 ▼                     ▼
Codex'e dön          teste geç
 │
 ▼
Codex düzeltir
 │
 ▼
PR güncellenir
 │
 ▼
Claude yeniden inceler
```

Buradaki önemli nokta şudur: **agentlar birbirlerine doğrudan mesaj göndermek zorunda değildir.** GitHub'daki görev durumu ve iş akışı kuralları hangi adımın sırada olduğunu belirleyebilir.

Şimdi bunu adım adım kuralım.

### 1. GitHub'da otomasyon alanını hazırla

Önce GitHub'ın komut satırı aracı olan **GitHub CLI** bilgisayarda kurulu ve ilgili repository için yetkilendirilmiş olmalıdır.

GitHub Agentic Workflows kullanılacaksa gerekli uzantı şu komutlarla hazırlanabilir:

```bash
gh extension install github/gh-aw
gh aw init
```

Agentlara ait otomatik görevler `.github/workflows/` klasöründe tutulur.

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

Buradaki Markdown dosyaları **hangi olayda hangi agentın ne yapacağını** anlatır.

Dosyanın en üstündeki ayar bölümü; görevin ne zaman başlayacağını, hangi agentın kullanılacağını ve hangi izinlere sahip olacağını belirtir. Altındaki normal metin ise agenta verilecek görevi açıklar.

`.lock.yml` dosyaları sistemin çalıştıracağı derlenmiş sürümlerdir. Kaynak workflow değiştirildiğinde şu komutla yeniden oluşturulurlar:

```bash
gh aw compile
```

### 2. Agent anahtarlarını kodun içine yazma

Codex, Claude veya Gemini gibi bir hizmet API anahtarı istiyorsa bu bilgi repository içindeki normal dosyalara yazılmamalıdır.

GitHub'da:

**Settings → Secrets and variables → Actions**

alanında **secret (gizli değer)** olarak saklanır.

Örneğin kullanılan kuruluma göre:

```text
Codex   → CODEX_API_KEY veya OPENAI_API_KEY
Claude  → ANTHROPIC_API_KEY
Gemini  → GEMINI_API_KEY
```

Gerçek anahtar değeri `AGENTS.md`, görev dosyası, workflow veya kaynak kod içine eklenmez.

### 3. Codex işi bitirdiğinde Claude'u başlat

Codex değişikliği tamamlayıp bir PR açtığında GitHub bunu bir olay olarak görebilir.

Biz de şu kuralı kurabiliriz:

> **Yeni PR açılırsa veya mevcut PR'a yeni kod gelirse Claude incelemeyi başlatsın.**

Örnek workflow:

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

Buradaki iki teknik ifade yalnızca GitHub'a **ne zaman çalışacağını** söyler:

- `opened` → PR ilk açıldığında çalış.
- `synchronize` → aynı PR'a yeni commit geldiğinde tekrar çalış.

Claude inceleme sonunda iki basit durumdan birini üretir:

```text
agent:fix-required
= düzeltme gerekiyor

agent:review-passed
= inceleme geçti
```

Bu durumlar bir sonraki adımı başlatmak için kullanılabilir.

### 4. Claude sorun bulursa görevi tekrar Codex'e gönder

Şimdi ikinci geçişi kurabiliriz:

> **Claude `agent:fix-required` sonucunu üretirse Codex düzeltme görevini başlatsın.**

Örnek:

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

PR üzerindeki inceleme bulgularını kontrol et.

Önce:
1. PR'nin güncel sürümünü INPUT_VERSION olarak kaydet.
2. Bulguyu güncel kod üzerinde yeniden doğrula.
3. AGENTS.md kurallarını ve PRESERVE sınırlarını koru.

Sorun doğrulanırsa gerekli en küçük düzeltmeyi yap.
İlgili testleri çalıştır.
Sorun doğrulanmazsa kodu değiştirme.

Sonuçta CHANGED, PRESERVED, VERIFIED,
SKIPPED_CHECKS ve SCOPE_STATUS bilgilerini üret.
```

Burada Codex'e **“Claude söyledi, doğrudur; düzelt”** demiyoruz.

Codex önce PR'ın güncel sürümünü kontrol ediyor ve Claude'un bulgusunun hâlâ geçerli olup olmadığını doğruluyor. Ancak bundan sonra gerekiyorsa kodu değiştiriyor.

Codex düzeltmeyi PR'a gönderdiğinde GitHub yeni bir commit görür. Bir önceki adımda tanımladığımız `synchronize` kuralı çalışır ve Claude değişikliği yeniden inceler.

Böylece şu döngü kullanıcı mesaj taşımadan çalışabilir:

```text
Claude sorun buldu
        ↓
Codex düzeltmeyi doğruladı ve yaptı
        ↓
PR güncellendi
        ↓
Claude tekrar kontrol etti
```

### 5. Claude onaylarsa testleri başlat

Claude sorun bulmazsa `agent:review-passed` durumu oluşturulabilir.

Bu kez sonraki kural şöyle olabilir:

> **İnceleme geçtiyse testleri çalıştır.**

```text
agent:review-passed
        ↓
test / build / lint
        ↓
başarılı → VERIFIED
başarısız → düzeltme görevi
```

Her kontrol için yeni bir AI agent kullanmak gerekmez. Örneğin birim testi, build veya lint gibi sonucu açık olan kontroller normal GitHub Actions adımlarıyla yapılabilir.

AI agentı daha çok kodu yorumlamak, değişikliğin görev sınırlarına uyup uymadığını incelemek veya bağlama göre değerlendirme yapmak gerektiğinde kullanmak daha anlamlıdır.

### 6. Tetikleyici ile orkestratör aynı şey değildir

Bu iki kavram kolayca karışabilir.

En basit haliyle:

```text
Tetikleyici
= "şimdi başla"

Orkestrasyon
= "şimdi kim, hangi görevle başlayacak?"

Agent
= "verilen görevi yap"
```

Örneğin GitHub'da `agent:fix-required` durumu oluştuğunda GitHub Actions Codex workflow'unu başlatabilir. Burada GitHub Actions **tetikleyici** görevini görür.

Hangi sonucun hangi agenta gideceği ise kurduğumuz **iş akışı kuralıdır**.

ChatGPT ayrıca orkestratör olarak kullanılacaksa ChatGPT'nin de bu otomasyon tarafından çağrılabileceği bir bağlantı gerekir. Böyle bir bağlantı kurulmamışsa GitHub'daki otomatik geçişleri GitHub Actions ve workflow kuralları yönetebilir.

Bu nedenle **“ChatGPT orkestratör”** demek tek başına ChatGPT'nin GitHub'da otomatik çalışacağı anlamına gelmez.

### 7. Agentlara yalnızca ihtiyaç duydukları yetkiyi ver

Otomasyon kurulduğunda agentların kullanıcı beklemeden işlem yapabileceği unutulmamalıdır. Bu nedenle yetkiler mümkün olduğunca sınırlı tutulmalıdır.

Örneğin agentın doğrudan `main` branch'ini değiştirmesi yerine bir PR üzerinde çalışması daha kontrollüdür.

GitHub Agentic Workflows içindeki **safe outputs (güvenli çıktılar)**, agentın yapabileceği yazma işlemlerini sınırlandırmak için kullanılabilir. Örneğin yalnızca PR oluşturmasına, mevcut PR branch'ini güncellemesine veya belirli etiketleri kullanmasına izin verilebilir.

Dosya sınırı da görev içinde açıkça belirtilebilir:

```text
ALLOWED_FILES:
- src/navigation/**
- tests/navigation/**

PROTECTED:
- AGENTS.md
- .github/**
- dependency / package dosyaları
```

Böylece navigasyon görevi alan bir agentın gereksiz yere workflow, güvenlik veya bağımlılık dosyalarını değiştirmemesi beklenir.

### 8. Aynı görevin iki kez başlamasını önle

Otomasyonda aynı görev yanlışlıkla iki kez tetiklenebilir. İki agent aynı dosyaları aynı anda değiştirirse birbirlerinin çalışmasını bozabilir.

Bu nedenle aynı `TASK_ID` veya aynı PR için aynı anda kaç çalışma yapılabileceği sınırlandırılmalıdır.

GitHub Actions'taki **concurrency (eşzamanlı çalışma kontrolü)** bunun için kullanılabilir.

Mantık basittir:

```text
UI-024 zaten çalışıyor
        ↓
ikinci UI-024 geldi
        ↓
beklet / iptal et / yeniden kontrol et
```

### 9. Otomasyonu küçük bir geçişle başlat

İlk denemede bütün agentları birbirine bağlamak yerine yalnızca bir geçişi otomatikleştirmek daha kolay kontrol edilir.

İlk hedef:

```text
Codex PR oluşturdu
        ↓
Claude otomatik incelemeye başladı
```

Bu geçiş doğru çalıştıktan sonra sırayla yeni adımlar eklenebilir:

```text
Claude → Codex düzeltme
Codex → Claude yeniden inceleme
İnceleme → test
Test → doğrulama
```

Her geçişte görevle birlikte en az şu bilgiler korunmalıdır:

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

Amaç yalnızca agentları sırayla çalıştırmak değildir. **Görevin hangi sürümde yapıldığı, hangi bulgunun aktarıldığı, nelerin korunacağı ve hangi kontrollerin eksik kaldığı da bir sonraki adıma taşınmalıdır.**

> **Not:** GitHub Agentic Workflows bu rehber hazırlanırken public preview (genel önizleme) durumundadır. Kurulum sırasında GitHub'ın güncel engine (agent motoru), trigger (tetikleyici), permission (izin) ve safe output (güvenli çıktı) seçenekleri kontrol edilmelidir.

## Ne zaman kullanılır?

Bu yöntem özellikle mevcut bir dosya veya kod üzerinde sınırlı bir düzeltme yapılırken, çalışan bölümlerin korunması gerektiğinde veya agentın yalnızca belirli dosyalara dokunması istendiğinde kullanışlıdır.

Küçük ve kolayca geri alınabilen denemelerde ayrıntılı bir değişiklik sınırı kaydı gerekmeyebilir. Örneğin boş bir deneme dosyasında birkaç farklı metin biçimini karşılaştırırken hangi satırların korunacağını ayrıca tanımlamak çoğu zaman gerekli değildir.
