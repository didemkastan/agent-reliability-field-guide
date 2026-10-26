# 6. Görev Devri Uygulaması (Agent Handoff in Practice)

[🇬🇧 English](README_EN.md)

## Amaç
Bir AI agentın başladığı işi başka bir agenta, bilgi kaybı ve gereksiz tekrar olmadan güvenli şekilde devretmek.

## 1. Problem
Bir agent kodu değiştirdiğinde diğer agent hangi dosyanın değiştiğini, nelerin korunacağını veya hangi testlerin yapıldığını bilmeyebilir. Bu durum hatalara, tekrarlanan işlemlere ve çalışan kodun bozulmasına neden olabilir.

## 2. Temel Kural
**Görevi devralan agent, işe başlamadan önce mevcut durumu ve sınırlarını bilmelidir.**

Her görev devrinde şu beş bilgi bulunmalıdır:
- **Görev:** Ne yapılacak?
- **Mevcut durum:** Şimdiye kadar ne yapıldı?
- **Korunacaklar:** Neye dokunulmayacak?
- **Doğrulama:** Hangi kontroller yapıldı, hangileri eksik?
- **Sonraki adım:** Yeni agent ne yapacak?

## 3. Örnek: ChatGPT → Codex → Reviewer
**Senaryo:** Bir uygulamada rapor oluşturma hatası var.

**ChatGPT — Görevi tanımlar**

> Görev: Rapor oluşturma hatasını düzelt.
>
> Mevcut durum: Hata tekrarlandı, ilgili dosya belirlendi.
>
> Korunacaklar: Mevcut ekran tasarımı ve diğer rapor türleri değişmeyecek.
>
> Doğrulama: Hatanın oluştuğu senaryo biliniyor, düzeltme henüz test edilmedi.
>
> Sonraki adım: Yalnızca ilgili hatayı düzelt ve test sonuçlarını bildir.

**Codex — Değişikliği gerçekleştirir.** Yaptığı değişiklikleri, test sonuçlarını ve eksik kontrolleri kaydederek görevi Reviewer'a devreder.

**Reviewer — Kontrol eder.** Değişiklikleri inceler, mümkünse testleri bağımsız çalıştırır ve sonucu bildirir:
- `PASS`: Gerekli kontroller başarılı.
- `FAIL`: Hata veya başarısız test var.
- `NEEDS_REVIEW`: Kontrol için yeterli kanıt yok.

## 4. Nasıl Uygulanır?
1. Görevi ve korunacak alanları belirle.
2. Mevcut dosya/sürüm bilgisini kontrol et.
3. Hazır [Handoff Receipt](../../patterns/handoff-receipt.md) şablonuyla görevi devret.
4. Yeni agentın görevi kabul ettiğini doğrula.
5. İş tamamlandığında test kanıtlarını kontrol et.

**Önemli:** Agentın "tamamlandı" demesi tek başına doğrulama değildir.

## 5. Ne Zaman Kullanılır?
Birden fazla AI agent aynı geliştirme görevinde çalışıyorsa veya bir görev farklı bir agent tarafından devam ettirilecekse kullanılır. Tek agentla yapılan basit işlemlerde ayrıntılı görev devri gerekli değildir.

## Sonuç
**İyi bir görev devri, yalnızca ne yapılacağını değil, neyin yapıldığını, neyin korunacağını ve neyin doğrulandığını da aktarır.**

Bu bölümdeki yöntem manuel olarak uygulanabilir. Sonraki bölümde aynı sürecin GitHub üzerinden nasıl otomatikleştirileceği anlatılacaktır.
