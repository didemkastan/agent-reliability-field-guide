# Güvenli Değişiklik

[🇬🇧 English](README_EN.md)

> Bu bölüm tek bir şeyi öğretir: **agenttan bir değişiklik istediğinde, yalnızca istediğin şeyin değişmesini nasıl sağlarsın?**
> Agentlar arasında görev devri ve otomasyon ayrı bölümlerde ele alınır.

## Bu sayfadaki kutular nasıl okunur?

- 👁 **Örnek:** Sadece okumak içindir.
- 📋 **Agenta yapıştır:** Agentın sohbet veya prompt alanına yazılır.
- 📄 **Dosyaya yaz:** Belirtilen dosyanın içine yazılır.
- 💻 **Terminalde çalıştır:** Bilgisayarındaki Terminal (macOS/Linux) veya PowerShell (Windows) ekranında çalıştırılır.

## Birkaç kelime

- **Repository (repo):** Projenin dosyalarının ve değişiklik geçmişinin tutulduğu çalışma alanı.
- **Commit:** Değişiklikların kaydedilmiş bir anlık görüntüsü.
- **Diff (değişiklik farkı):** Bir dosyada hangi satırların silindiğini ve eklendiğini gösteren karşılaştırma.
- **Kapsam:** Görevin sınırı; agentın neye dokunabileceğini ve neyi koruması gerektiğini belirler.

## Problem

Agenttan küçük bir değişiklik istediğinde, istediğin şeyi yaparken başka yerleri de değiştirebilir.

Örneğin yalnızca bir bağlantının düzeltilmesini istersin; agent aynı dosyadaki metinleri de yeniden düzenler veya başka dosyalara dokunur. Bağlantı düzelmiştir ama değişiklik artık istediğinden daha geniştir.

## Temel kural

> **Değişiklikten önce neyin değişebileceğini ve neyin korunacağını belirle. İstenen sonuç için gereken en küçük değişikliği yap.**

## Değişiklik sınırı: üç alan

- **CHANGE (Değiştir):** Yapılmasını istediğin değişiklik.
- **PRESERVE (Koru):** Aynı kalması gereken yerler.
- **DO NOT (Yapma):** Bu görevde yapılmaması gerekenler.

📋 **Agenta yapıştır**

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

Liste uzun olmak zorunda değildir. Küçük bir işte yalnızca kritik sınırları yazmak yeterlidir.

## En küçük değişiklik

Bir yazım hatası tek satırda düzeliyorsa aynı anda paragrafı yeniden yazmaya veya dosyanın biçimini değiştirmeye gerek yoktur.

Agent kapsam dışında başka bir sorun fark ederse onu kendiliğinden düzeltmek yerine **bildirmelidir**. Kapsamı genişletme kararı kullanıcıya aittir.

## Örnek: beklenen ve gerçekleşen

İstenen: README dosyasındaki tek bir bağlantının değişmesi.

👁 **Örnek**

```
BEKLENEN                GERÇEKLEŞEN
README.md               README.md
                        config.yaml     ← beklenmiyordu
                        package.json    ← beklenmiyordu
```

Bağlantı doğru düzelmiş olsa bile diğer iki dosya görevin parçası değildir. Beklenmeyen değişiklikler incelenmeli; gerekli değilse geri alınmalıdır.

## Kapsamı nasıl kontrol edersin?

İş bitince iki soruya bak:

1. **Hangi dosyalar değişti?** Beklediğin listeyle aynı mı?
2. **Dosyaların içinde ne değişti?** Korunması gereken bir bölüme dokunulmuş mu?

Yalnızca dosya sayısına bakmak yetmez. Doğru dosyanın yanlış satırı da değişmiş olabilir.

### GitHub'da kontrol

Pull Request sayfasında **Files changed** sekmesini aç. Burada PR içindeki değişen dosyaları ve satırları görebilirsin.

### Bilgisayarında: henüz commit edilmemiş değişiklik

Repo klasöründe:

💻 **Terminalde çalıştır**

```bash
git status
git diff --stat
git diff
```

- `git status` çalışma alanının durumunu gösterir.
- `git diff --stat` henüz commit edilmemiş değişikliklerin dosya bazında özetini verir.
- `git diff` henüz commit edilmemiş değişikliklerin satır farkını gösterir.

> ⚠️ Agent değişikliği zaten commit ettiyse normal `git diff` boş görünebilir. Bu, değişiklik yapılmadığı anlamına gelmez.

### Değişiklik zaten commit edildiyse

Son committe ne değiştiğini görmek için:

💻 **Terminalde çalıştır**

```bash
git show --stat HEAD
git show HEAD
```

- `git show --stat HEAD` son committe değişen dosyaları özetler.
- `git show HEAD` son commitin satır farkını gösterir.

Bir PR kullanıyorsan en kolay kontrol yine GitHub'daki **Files changed** sekmesidir.

### Beklenmeyen değişikliği geri almadan önce

Değişiklik **henüz commit edilmemişse** ve dosyadaki tüm yerel değişiklikleri gerçekten silmek istediğinden eminsen:

💻 **Terminalde çalıştır**

```bash
git restore config.yaml
```

> ⚠️ `git restore config.yaml` bu dosyadaki commit edilmemiş değişiklikleri geri alınamaz biçimde silebilir. Emin değilsen çalıştırma; önce `git diff config.yaml` ile neyin değiştiğini incele.
>
> Değişiklik zaten commit edildiyse bu komutu çözüm olarak kullanma. Önce commit farkını incele ve uygun geri alma yöntemini ayrıca belirle.

## Kullanılabilir prompt

📋 **Agenta yapıştır**

```
Göreve başlamadan önce kapsamı belirle: hangi dosya veya bölümlerin
değişeceğini ve nelerin korunacağını yaz.

Yalnızca görevi tamamlamak için gereken en küçük değişikliği yap.
Görevle ilgisi olmayan dosyaları, metinleri, ayarları, bağımlılıkları
veya çalışan mevcut davranışları değiştirme.

Kapsam dışında bir sorun fark edersen düzeltme; ayrıca bildir.
Görevi bitirmek için kapsamın dışına çıkmak zorunlu hale gelirse,
değişiklik yapmadan önce dur ve nedenini açıkla.

İş bitince yaptığın değişiklikleri başlangıçtaki kapsamla karşılaştır.
Mümkünse diff üzerinden kontrol et. Kapsam dışı bir değişiklik varsa
açıkça belirt.

Sonunda şu kaydı ver:
CHANGED: gerçekte değiştirilenler
PRESERVED: korunduğunu kontrol ettiklerin
UNEXPECTED_CHANGES: beklenmeyen değişiklikler (yoksa: none)
SCOPE_STATUS: değişiklik görev sınırında kaldı mı?
```

> **Agentın raporu tek başına kanıt değildir.** Son kontrolü mümkün olduğunda gerçek diff üzerinden yap.

## Birden fazla agent kullanıyorsan kurallar nereye yazılır?

Her görevde aynı kalıcı kuralları tekrar yazmak yerine ortak kuralları tek bir proje talimatında tutabilirsin. Basit bir kurulumda repository kökünde `AGENTS.md` kullanılabilir.

👁 **Örnek**

```
PROJE/
├── AGENTS.md
├── src/
└── ...
```

📄 **Dosyaya yaz:** `AGENTS.md`

Bu dosyanın içine güvenli değişiklik kurallarını ve projede sürekli korunması gereken sınırları normal Markdown metni olarak yaz.

> ⚠️ **Dosyanın repoda bulunması, her agentın onu otomatik olarak okuduğu anlamına gelmez.** Araçların proje talimatlarını yükleme yöntemleri farklıdır ve zamanla değişebilir.

| Araç | Varsayılan proje talimatı | Ortak kurala bağlama |
|---|---|---|
| Codex CLI | `AGENTS.md` | Repository kökündeki kuralları kullanabilir. |
| Claude Code | `CLAUDE.md` | Ortak kuralları Claude'un proje talimatına bağla veya ilgili kuralları orada tut. |
| Gemini CLI | `GEMINI.md` | `GEMINI.md` kullan veya `context.fileName` ayarıyla ortak dosya adını yapılandır. |
| Sohbet arayüzleri | Araca göre değişir | Proje talimatını ekle veya gerekli dosyayı konuşmaya ver. |

Kurulumdan sonra aracın gerçekten hangi talimatları yüklediğini kendi güncel mekanizmasıyla kontrol et.

### Talimat dosyasının yüklendiğini nasıl kontrol edersin?

`AGENTS.md` içine kısa ve benzersiz bir kontrol satırı koyabilirsin:

📄 **Dosyaya yaz:** `AGENTS.md`

```
KURAL-SÜRÜMÜ: v1.3
```

Yeni bir görevin başında agenttan bu satırı aynen aktarmasını iste.

> Satırı doğru aktaramıyorsa talimat dosyasının yüklendiğini **varsayma**. Doğru aktarması ise yararlı bir başlangıç kontrolüdür; kritik işlerde tek başına kesin kanıt sayılmaz. Gerekirse kullanılan talimat dosyasının sürümünü veya hash'ini ayrıca kaydet.

Kurallar doğru bağlandıktan sonra her görevde yalnızca o işe özel sınırları verirsin:

👁 **Örnek**

```
ORTAK / KALICI  → proje talimatı
  güvenli değişiklik kuralları
  kalıcı proje sınırları

GÖREVE ÖZEL     → görev mesajı
  TASK / CHANGE / PRESERVE / DO NOT
```

## Ne zaman kullanılır?

- Çalışan bir dosya veya kod üzerinde sınırlı bir düzeltme yapılırken
- Belirli bölümlerin kesinlikle korunması gerektiğinde
- Agentın yalnızca belirli dosyalara dokunması istendiğinde

Kolayca geri alınabilen küçük denemelerde ayrıntılı bir sınır kaydı gerekmeyebilir.

## Özet

1. İşe başlamadan **CHANGE / PRESERVE / DO NOT** yaz.
2. Agenttan **en küçük değişikliği** ve kısa bir kapsam raporu iste.
3. Commit edilmemiş ve commit edilmiş değişiklikleri doğru yöntemle ayır.
4. Agent raporunu mümkün olduğunda **diff ile doğrula**.
5. Beklenmeyen değişikliği otomatik olarak görevin parçası sayma.

## Sonraki adımlar

- **Agentlar Arası Görev Devri:** Görev paketi, sürüm kontrolü ve dış agentla çalışma.
- **Agent Otomasyonu:** Otomatik tetikleme ve agent geçişleri ayrı bölümde ele alınır.
