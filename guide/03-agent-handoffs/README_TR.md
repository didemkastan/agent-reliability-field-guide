# Agentlar Arası Görev Devri

[🇬🇧 English](README.md)

## Problem
Agent A görevini doğru tamamlayabilir; buna rağmen Agent B, devir sırasında önemli bir kısıt kaybolduğu için hata yapabilir.

Yani hata her zaman agentın **içinde** değildir. Bazen agentların **arasındadır**.

## İnsan dilindeki kural
> **Bir görev devri neyin değiştiğini, neyin kesinlikle korunacağını, gerçekte neyin doğrulandığını ve neyin kontrol edilmediğini açıkça söylemelidir.**

## Minimum handoff kaydı
TASK_ID · INPUT_VERSION · OUTPUT_VERSION · DEĞİŞENLER · KORUNACAKLAR · DOĞRULANANLAR · SKIPPED_CHECKS · NEXT_AGENT

## Kapsam da kanıtlanmalı
“Her şeyi kontrol ettim” ifadesi kapsam kanıtı değildir. Sınırları belli görevlerde EXPECTED_ITEMS, OBSERVED_ITEMS, CHECKED_ITEMS, SKIPPED_ITEMS ve COVERAGE_STATUS tutulabilir.

12 dosya beklenirken yalnız 10 dosya incelendiyse bu durum kayıtta görünmelidir.

## Sentetik örnek
Agent A üç hayali configuration dosyasını değiştiriyor fakat yalnız ikisini test edebiliyor. İyi bir görev devri: 3 dosya değişti; 2 dosya test edildi; 1 dosya çalıştırılmadı; nedeni gerekli runtime'ın mevcut olmaması.

Böylece Agent B doğrulamaya nereden devam edeceğini bilir.

## Ne zaman kullanılır?
İş agentlar, oturumlar, bilgisayarlar veya insanlar arasında devrediliyorsa yapılandırılmış handoff değerlidir. Tek adımlık ve kaybolması önemsiz bir işte ayrıntılı receipt gereksiz olabilir.