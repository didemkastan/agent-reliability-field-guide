# Agent Reliability Field Guide

[🇬🇧 English](README.md)

> **Güvenilir AI agentlar hiç hata yapmayan agentlar değildir. Ne olduğunu, neden olduğunu, neyin doğrulandığını ve neyin hâlâ belirsiz olduğunu gösterebilen sistemlerdir.**

Doğrulanması, izlenmesi, hatadan toparlanması ve güvenilmesi daha kolay AI-agent iş akışları oluşturmak için hazırlanmış pratik ve iki dilli bir rehber.

## Bu rehber neden var?

Agent sistemleri bazen oldukça sıradan nedenlerle hata verir: bir agent eski bir dosyayı değiştirir, iki agent birbirinin çalışmasını ezer, retry aynı yanlış varsayımı tekrarlar veya kendinden emin bir “tamamlandı” mesajı doğrulama sanılır.

Bu rehber, bu hata desenlerini basit mühendislik yöntemlerine dönüştürmeyi amaçlar.

Her yöntem tutarlı bir yapıyla ele alınır:

**Problem → Açıklama → Sentetik örnek → Ne ters gidebilir? → Pratik yöntem → Doğrulama → Ne zaman gereksiz olabilir?**

## Ana konular

1. **Gerçeğin kaynağı** — provenance, canonical sürüm ve karar kayıtları.
2. **Güvenli değişiklik** — eski durumla işlem yapmayı ve yanlış onayı önleme.
3. **Agentlar arası görev devri** — açık context, kapsam kanıtı ve atlanan kontroller.
4. **Doğrulama** — gözlem, yorum, işlem ve kanıtı birbirinden ayırma.
5. **Hata & retry** — bilinçli tekrar, retry sınırı ve gerçek hata kaynağını bulma.
6. **Hafıza & context** — geçici bilgi, geçersizleştirme, yanlış bilgi kaydı ve çakışma yönetimi.
7. **Güvenlik & yetki** — minimum yetki, kapsamlı izin ve yeniden doğrulama.
8. **Observability** — güvenilir olay kayıtları ve doğrulanabilir çalışma izi.

## Güvenlik ve gizlilik

Bu repository'de **özel proje verisi, şirket bilgisi, müşteri verisi, şirket içi kod, özel konuşmalar, kimlik bilgileri veya gizli loglar yayımlanmaz.**

Örnekler sentetik veya genelleştirilmiş olacaktır. Herkese açık fikirler yöntemlere ilham verebilir; rehberin asıl değeri özel veya sahipli içeriği kopyalamak değil, fikirleri sentezlemek, sadeleştirmek, test etmek ve uygulanabilir hale getirmektir.

## Durum

🚧 **Rehberin ilk sürümü hazırlanıyor.** Yapı adım adım geliştirilecek. Yalnızca farklı ve uygulanabilir fayda sağlayan yöntemler eklenecek.

## Temel ilke

> **Bir yöntemi yeni olduğu için kullanma. Gerçek ihtiyaç ortaya çıktığında küçük ölçekte dene, faydasını doğrula ve ancak bundan sonra çalışma sisteminin parçası yap.**

## Dil

- Türkçe: bu dosya
- English: [README.md](README.md)

## Lisans

İlk kararlı sürümden önce lisans seçimi kesinleştirilecektir.
