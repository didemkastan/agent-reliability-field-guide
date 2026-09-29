# Gerçeğin Kaynağı

[🇬🇧 English](README_EN.md)

## Problem
Bir agent tamamen mantıklı bir değişikliği yanlış dosya sürümü üzerinde hazırlayabilir.

Agent A'nın 4. sürüm üzerinde çalıştığını düşünün. Bu sırada Agent B dosyayı güncelleyerek 5. sürümü oluşturuyor. Agent A bu değişikliği fark etmezse hazırladığı işlemi artık güncel olmayan 4. sürüme göre tamamlayabilir. Yapılan işlem kendi içinde doğru olsa bile eski sürüme dayandığı için güncel dosyada hataya veya veri kaybına yol açabilir.

## Temel kural
> **Bir şeyi değiştirmeden önce hangi sürümün gerçek kaynak olduğunu bil; yazmadan hemen önce de hâlâ aynı sürüm olduğunu kontrol et.**

## Pratik yöntem
Kritik çalışmalarda SOURCE, VERSION, STATUS, VERIFIED_BY ve HASH veya benzeri kararlı bir sürüm kimliği tutulabilir.

Yazmadan önce:
1. Güncel artifact'i oku.
2. Sürüm/hash bilgisini kaydet.
3. Değişikliği hazırla.
4. Yazmadan hemen önce canlı sürüm/hash'i tekrar kontrol et.
5. Değişmişse üzerine yazma; dur ve yeniden oku.

## Sentetik örnek
İki agent tamamen hayali bir `settings.yaml` dosyasında çalışıyor.

Agent A `abc123` hash'ini okuyor. Agent B dosyayı güncelliyor ve yeni hash `def456` oluyor. Agent A yazmadan önce tekrar kontrol ediyor ve artık `def456` gördüğünü fark ediyor.

**Doğru davranış:** dur, güncel dosyayı yeniden oku ve değişikliği yeniden hazırla.

**Riskli davranış:** `abc123` için hazırlanmış değişikliği `def456` üzerine yaz.

## Ne zaman kullanılır?
Aynı artifact'i birden fazla insan, agent, branch, otomasyon veya bilgisayar değiştirebiliyorsa değerlidir.

Tek kişinin kullandığı, kaybolması sorun olmayan küçük bir deneyde bütün kayıt yapısı gereksiz olabilir.