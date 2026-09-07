---
name: dev-flow
description: Kod işi yapan çalışanlar için kapsam, branch, doğrulama, PR ve inceleme akışı.
---

> OpenMaus içinde yönetilir. Köken: Multica studio, bridge; 6 Eylül 2026 dışa aktarımı.
> Yeni içeriği Skills ekranında inceleyip etkinleştir. Bu dosya eski kütüphaneden senkronize edilmez.


# Dev Flow

Bu skill kod işi atanmış çalışana aittir; yönetici botun üretim yasağını kaldırmaz.

1. Onaylanan işi ve bitiş koşulunu oku; repo AGENTS.md talimatlarını ve mevcut çalışmayı kontrol et. Başkasının değişikliklerini ezme.
2. Kod değişikliği için ayrı branch kullan. Kısa iş adı yeterlidir; Multica ticket anahtarı gerekmez.
3. Yalnız onaylı kapsamı uygula; değişikliğe uygun testleri çalıştır. Gerçek çıktıyı kaydet, çalışmayan kontrolü açıkça belirt.
4. PR hazırlarken ne değiştiğini, nasıl doğrulandığını ve ilgili iş/konuşma bağlantısını yaz. Var olmayan `Closes KEY` ekleme.
5. Gerekli bağımsız incelemeyi gerçekten mevcut araçla talep et. Webhook'un kendiliğinden Reviewer başlatacağı varsayılmaz. İnceleme yoksa bunu görünür bırak.
6. İnceleme sonucunu ele al. Merge veya canlıya alma yalnız mevcut açık yetki kapsamında yapılır; review sonucu tek başına yetki değildir. Yetkili merge squash olur.

Davranış değişince ilgili dokümanı aynı değişiklikle güncelle. Sonuç ve kanıtı iş planını yöneten bota/konuşmaya döndür. Kapsam dışı bulguyu ayrı not et; yeni ticket veya iş açma. Bir işi bitti saymak için bitiş koşulunu doğrula.

Alt-agent sayısını ihtiyaca göre tut; küçük iş için kadro kurma. Yeni alt-agent çağrısı kullanıcı yetkisi ve oturumun araç kurallarına bağlıdır. Aynı dosyayı değiştiren işleri çakıştırma.
