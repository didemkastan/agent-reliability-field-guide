# Agent Reliability Field Guide (Agent Güvenilirliği Uygulama Rehberi)

## 📚 Rehbere Başla

Konu başlıklarına tıklayarak içeriğe ulaşabilirsiniz.

- [1. Gerçeğin Kaynağı](guide/01-source-of-truth/README.md) — Doğru dosya ve doğru sürümle çalışmayı güvence altına alma.
- [2. Agentlar Arası Görev Devri](guide/03-agent-handoffs/README.md) — Bir agentın yaptığı işi diğerine gerekli bilgilerle birlikte aktarma.
- [3. Verification (Doğrulama)](guide/04-verification/README.md) — “Tamamlandı” demek ile sonucu gerçekten doğrulamak arasındaki fark.
- [4. Handoff Receipt (Görev Devri Kayıt Şablonu)](patterns/handoff-receipt.md) — Görev devrinde kullanılabilecek hazır kayıt yapısı.

> Yeni bölümler eklendikçe bu liste güncellenecektir.

[🇬🇧 English](README_EN.md)

> **Güvenilir AI agentlar hiç hata yapmayan agentlar değildir. Ne olduğunu, neden olduğunu, neyin doğrulandığını ve neyin hâlâ belirsiz olduğunu gösterebilen sistemlerdir.**

Bu rehber, AI agent iş akışlarının daha güvenilir, izlenebilir ve doğrulanabilir şekilde oluşturulmasına katkı sağlamak amacıyla hazırlanmıştır. İçeriğin daha fazla kişiye ulaşabilmesi ve farklı topluluklar tarafından kullanılabilmesi için Türkçe ve İngilizce olarak sunulmaktadır.

## Bu rehberin amacı

Agent sistemlerinde farklı nedenlerle sorunlar ortaya çıkabilir. Bir agent güncel olmayan bir dosya üzerinde çalışabilir, iki agent aynı dosyada birbirinin yaptığı değişiklikleri etkileyebilir, bir retry (yeniden deneme) aynı hatalı varsayımla tekrar çalışabilir veya “tamamlandı” bilgisi yeterli kontrol yapılmadan doğru kabul edilebilir.

Bu rehber, bu tür sorunları önlemeye ve yönetmeye yardımcı olacak uygulanabilir yöntemler sunmayı amaçlar.

Her yöntem aynı yapı içinde ele alınır:

**Problem → Açıklama → Sentetik örnek → Ne ters gidebilir? → Pratik yöntem → Doğrulama → Ne zaman gereksiz olabilir?**

## Ana konular

1. **Gerçeğin kaynağı** — provenance (bilginin kaynağı ve geçmişi), canonical version (esas alınan sürüm) ve karar kayıtları.
2. **Güvenli değişiklik** — güncel olmayan bilgilerle işlem yapılmasını ve yanlış sürüm üzerinde değişiklik yapılmasını önleme.
3. **Agentlar arası görev devri** — gerekli bilgilerin aktarılması, yapılan kontrollerin belirtilmesi ve eksik kalan kontrollerin görünür olması.
4. **Handoff Receipt (Görev Devri Kayıt Şablonu)** — görev devrinde mevcut durumu, yapılan değişiklikleri, doğrulananları, eksik kontrolleri ve sonraki adımı standart bir kayıtla aktarma.
5. **Verification (Doğrulama)** — gözlem, yorum, yapılan işlem ve doğrulama sonucunu birbirinden ayırma.
6. **Retry (Yeniden deneme)** — aynı hatayı tekrarlamak yerine hata nedenini dikkate alarak yeniden deneme ve gerektiğinde işlemi durdurma.
7. **Memory & Context (Hafıza ve bağlam)** — geçici bilgilerin, değişen kaynakların, hatalı kayıtların ve bilgi çakışmalarının yönetimi.
8. **Security & Authority (Güvenlik ve yetki)** — yalnızca gerekli yetkilerin verilmesi ve yetkinin gerektiğinde yeniden kontrol edilmesi.
9. **Observability (İzlenebilirlik)** — yapılan işlemlerin ve doğrulama sonuçlarının sonradan kontrol edilebilecek şekilde kaydedilmesi.

## Güvenlik ve gizlilik

Bu repository'de **özel proje verisi, şirket bilgisi, müşteri verisi, şirket içi kod, özel konuşmalar, kimlik bilgileri veya gizli loglar yayımlanmaz.**

Örnekler sentetik veya genelleştirilmiş olacaktır. Herkese açık fikirler yöntemlere ilham verebilir; rehberin asıl değeri özel veya sahipli içeriği kopyalamak değil, fikirleri bir araya getirmek, değerlendirmek, test etmek ve uygulanabilir hale getirmektir.

## Durum

🚧 **Rehberin ilk sürümü hazırlanıyor.** Yeni bölümler ihtiyaç ve kullanım alanına göre eklenecektir.

## Temel ilke

> **Bir yöntemi yalnızca yeni olduğu için kullanma. İhtiyaç ortaya çıktığında önce küçük ölçekte dene, faydasını doğrula ve uygun olduğunda çalışma sistemine dahil et.**

## Dil

- Türkçe: bu dosya
- English: [README_EN.md](README_EN.md)

## Lisans

İlk kararlı sürümden önce lisans seçimi kesinleştirilecektir.
