---
name: is-plani
description: OpenMaus botlarının onaylı işi küçük adımlara ayırması, çalışanlara ataması ve doğrulanmış ilerlemeyi göstermesi.
---

> OpenMaus içinde yönetilir. Köken: Mevcut yerel skill; 6 Eylül 2026 aktarımı.
> Yeni içeriği Skills ekranında inceleyip etkinleştir. Bu dosya eski kütüphaneden senkronize edilmez.


# İş Planı

İlk planı konuşmada hazırla: hedef, kapsam, ölçülebilir bitiş koşulu ve gerekli adımlar. Bir adım için ayrı teslimat gerekmiyorsa yeni ana iş oluşturma. Kullanıcının mevcut onayı varsa tekrar sorma.

Her adımın sorumlusu, durumu ve sıradaki adımı belli olsun. Basit görünüm alanları: **Bot · İş · Adım · Durum · Sıradaki · Son güncelleme**. Çalışan atanmışsa adımın altında gerçek çalışan adı/kimliği ve sonuç bağlantısı gösterilir.

## Durum güncelleme
- `bekliyor`: henüz başlamadı; bağımlılık veya kullanıcı yanıtı bekleniyorsa nedeni yaz.
- `çalışıyor`: çağırma aracı işi kabul etti ve çalıştığına ilişkin güncel kanıt var.
- `bitti`: adımın bitiş koşulu çıktı, test veya inceleme ile doğrulandı.
- `takıldı`: ilerlemeyi durduran hata ya da eksik bilgi açıklandı.
- İptal ve hata gerçekten doğrulanmışsa ayrı göster. Güncel sinyal yoksa son bilinen durumu ve zamanını yaz; çalışıyor diye tahmin etme.
- Bir tool çağrısının ya da model turunun bitmesi, bütün işin bitmesi değildir.

## Çalışanı çağırma
Bot kapsamı ve planı yönetir; üretim için o oturumda gerçekten mevcut olan çalıştırma/delegasyon aracını kullanır. İş brief'i hedefi, izin verilen yolları, bitiş koşulunu, çıktı yerini ve ilgili skill kaynaklarını içerir. Kullanıcı onayı olmadan yeni iş veya gereksiz paralel kadro oluşturulmaz.

OpenMaus'taki başka bota iş atanacaksa önce `list_bots`, sonra `delegate_bot` kullan; kabulü tamamlanma sayma. Claude/Codex/Grok alt-agentları yalnız seçili motor gerçekten destekliyorsa kullanılır. Gereken yürütücü yoksa adımı beklet, eksik yeteneği söyle; hayali çalışan oluşturma.

Bir adım bitince çıktısını bitiş koşuluna göre kontrol et ve onaylanan sıradaki adıma geç. Kapsam dışı bulguyu not et; kendiliğinden yeni işe dönüştürme. Kısmi başarısızlığı gizleme. Paylaşma, canlıya alma ve diğer dış işlemler mevcut yetkiye göre yapılır.

## Görünürlük sınırı
Şimdilik plan ve ilerleme konuşmada raporlanır. Bu skill kendi başına canlı UI, hook, kalıcı kuyruk veya otomatik yürütme motoru kurmaz. Böyle bir entegrasyon doğrulanmadan “UI otomatik güncellendi” veya “arka planda sürüyor” deme.
