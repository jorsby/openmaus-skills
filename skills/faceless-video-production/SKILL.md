---
name: faceless-video-production
description: ESKİ ELLE HAT (v1) — faceless explainer için prompt/kare paketi hazırlayıp bırakır; hiçbir şey üretmez, Serhat kopyala-yükle-tık yapar. Jorsby Studio v2 op’larıyla gerçek üretim için `jorsby-studio` skill’ini kullan. Bunu YALNIZ kullanıcı açıkça elle kopyala-yapıştır akışı istediğinde ya da v1 arşivine bakılacağında aç. faceless video yapalım tek başına bu skill’i açmaz.
trigger: /faceless-video
---

> Paket ekleri aşağıda “Paket eki” başlıklarında tam metindir. Metindeki göreli paket yolları bu bölümlere karşılık gelir. Yalnız ihtiyaç duyulan bölümü oku. Scriptler disk üzerinde kurulu değildir; çalıştırma gerekirse onaylanan iş kapsamında geçici dosyaya çıkarılıp doğrulanır.


# faceless-video

Elde bir anlatım metni varken — client yazmış ya da biz yazacağız — yüzsüz çizgi
açıklayıcı video serisi üretme hattı. SickKids "Let's Talk" serisinden çıktı, sonra
Higgsfield'ın explainer hattıyla karşılaştırılıp genişletildi.

**Ben video üretmem.** Prompt'ları, kareleri ve ses dosyalarını sıralı paket halinde
bırakırım; Serhat kopyala–yükle–tık yapar. Aksi açıkça söylenmedikçe bu böyle.

---

## Değişmez kurallar

1. **Client'ın metni dokunulmazdır.** Bölünebilir, ama tek kelime eklenmez, çıkarılmaz,
   yeniden yazılmaz. Bölme yaptıysan parçaları birleştirip orijinalle **karakter karakter**
   karşılaştır — gözle değil, makineyle. Bu kontrol tutmadan devam etme.
2. **Süre ölçülür, tahmin edilmez.** Karakter sayısından süre uyduran her tablo yalan söyler.
   Bir kere 301 sn tahmin edilmişti, gerçeği 453 sn çıktı. Ses üretilir, `ffprobe` ile ölçülür,
   sayı oradan gelir.
3. **Blok süresi her şeyi belirler** — hangi modeli kullanabileceğini, kaç sahne olacağını,
   kaç ses parçası okutacağını. İlk kararın bu olsun, sonradan değiştirmek her şeyi söker.
4. **Tutarlılık iki mekanizmayla taşınır, ikisi de zorunlu:** her üretime eklenen sabit
   **style key görseli**, ve her prompt'a birebir yapıştırılan **aynı STYLE metni**.
   Biri eksikse stil kayar.
5. **Videoda konuşma yok.** Klip sesi sadece ortam/müzik. Karakter ağzını oynatmaz,
   lip-sync yok, anlatım klibin içine gömülmez. Anlatım ayrı üretilir, kurguda birleşir.
6. **Pozitif prompt.** İstenen şey tarif edilir; "şu olmasın" ile sahne yönetilmez.
   Tek istisna aşağıda, prompt şablonundaki NEGATIVE satırı.
7. **Her kare oynayabilmeli.** Kare video olacak. İçinde sadece yazı olan kart, altı saniye
   boyunca hiçbir şey yapmayan bir görüntüdür.
8. **Alan bilgisi tasarım kararı değil.** Klinik, hukuki, finansal içerikte ikon/sembol
   uydurmak bizim işimiz değil. Üret, ama işaretle ve onaya gönder.

---

## Hat

| # | Aşama | Çıktı | Kim |
|---|---|---|---|
| 0 | Blok kararı | saniye sınırı + hedef model | kullanıcı seçer |
| 1 | Style key | tek görsel, kilitlenir | kullanıcı seçer |
| 2 | Referans sheet'ler | karakter · mekân · prop | ben |
| 3 | Metin bölme | bloklara ayrılmış anlatım | ben, kanıtlı |
| 4 | Voiceover | ses dosyaları + **ölçülmüş** süreler | ben |
| 5 | First frame grid'leri | sahne başına kareler | ben |
| 6 | Video promptları | kare başına hareket promptu | ben |
| 7 | Video generation | klipler | **Serhat elle** |

**Sıra pazarlığa kapalı:** ses videodan önce gelir. Ölçülen süre sahne sayısını belirler,
sahne sayısı kare sayısını belirler. Ters çevirirsen çizdiğin kareler tutmaz ve baştan
üretirsin. Aynı sebeple 3 ile 4 arasında bariyer var: **tüm ses bitmeden hiçbir kare çizilmez.**

---

## 0 · Blok kararı

Tek soru, ilk soru: **bir sahne en fazla kaç saniye?**

| Sınır | Ne açar | Bedeli |
|---|---|---|
| **≤10 sn** | Grok, Gemini Omni, çoğu model | daha çok sahne, daha çok kare, daha çok üretim |
| **14–20 sn** | pratikte Seedance | az sahne ama görseller çok daha güçlü olmalı; uzun tek çekim zayıf görseli affetmez |

Sabit bloklu hat (her blok tam 10 sn, sunucu tarafı birleştirme) ile ölçülü bloklu hat
(her sahne kendi doğal uzunluğunda, sınırın altında) farklı şeylerdir. Client metni varsa
ölçülü blok kullan — sabit bloğa uydurmak metni yeniden yazmayı gerektirir, o da yasak.

Sınırı sonradan indirmek **bölünen her sahnenin sesini yeniden okutmak** demektir. Kararı
baştan net al.

---

## 1 · Style key

Tek sayfa: PALETTE · TYPOGRAPHY · TEXTURE · TECHNIQUE · LIGHT AND SHADOW · RULES.
**Karakter, mekân, prop içermez** — sheet içerik değil teknik anlatır.

Birkaç aday üret → **kullanıcı seçer** → biri kilitlenir, kalanı `_archive/`'a. Sessizce
kendin seçme; kullanıcı "sen seç" demedikçe stil kararı onun.

Kilitlenen sheet bundan sonra prompt'a yazılmaz, **reference image olarak bağlanır** —
ve aynı anda STYLE metni her prompt'a yapıştırılır. İkisi birlikte.

Marka paleti varsa **üretimden önce** sor. Palet sonradan değişirse üretilmiş her kare çöp.

Şablonlar: `references/prompts.md` → "Style key".

## 2 · Referans sheet'ler

Her karakter için **ayrı** sheet; kadro sayfası değil. Mekânlar için plaka, proplar için
tek sayfa. Kimlik prompt'ta değil sheet'te tutulur.

**Tuzak:** sheet üretirken style sheet'i referans olarak bağlarsan sheet'in **başlığı sızar** —
karakterin köşesinde "STYLE BIBLE" yazar. Anchor olarak yalnızca onaylanmış bir grid kullan.

## 3 · Metin bölme

Sınırı aşan her sahne bölünür. Bölme yeri sırasıyla: **cümle sonu → noktalı virgül →
bağlaçlı virgül**. Cümle ortasından bölmek zorunda kalırsan parça "or…", "and…" diye
başlayabilir; bu kabul edilebilir, kelime değiştirmek edilemez.

Bölünen her sahne için **client numarasını koru** (`client_no` + parça harfi) ve panoda
"önceden şöyleydi" notu bırak. Not kısa ve insan ağzından olsun — kural metni değil.

Bölmeden sonra iki şey **kendiliğinden olmaz**, elle yapılır:
- **Başlıklar.** Parçalar ebeveynin başlığını miras alır; 10 sahne arka arkaya aynı adı
  taşır ve pano okunmaz olur. Her parçaya kendi başlığını yaz.
- **Görsel yön.** Parçalar ebeveynin `visual` metnini de miras alır. Ayrıştırmadan kare
  üretme, yoksa 10 sahne aynı promptu üretir.

## 4 · Voiceover

Ses seçimi kullanıcının kararı. Komşu blokları `previous_text` / `next_text` ile bağla —
yoksa her parça yeni cümle gibi okunur ve kesim yerinde ton sıfırlanır, duyulur.

Üret → `ffprobe` ile ölç → sınırı aşan varsa **tekrar böl ve tekrar okut**. Kaba tahmin
(sadece planlama için, asla kayda geçmez): sakin İngilizce anlatım ≈ **15 karakter/saniye**.

Parçalara ayırmak temposu değiştirir: model tek uzun cümleyi ağır, kısa parçayı akıcı okur.
Seri toplamı yakın kalır, sahne bazında oynar. Normal.

## 5 · First frame grid'leri

Sahne başına bir grid; kare sayısı = anlatımın beat sayısı.

**Zincir:** tutarlılığı önceki kare değil, her üretime eklenen sabit sheet'ler taşır.
Önceki grid yalnız bölüm içi akış içindir — bu yüzden **bölümler paralel üretilebilir.**
Gerçek sınır modelin eşzamanlı iş kotası (Higgsfield ultra: 8).

Tuval geometrisi ve kırpma tablosu: `references/production.md`.

## 6 · Video promptları

Girdi: first frame + o bloğun anlatımı + **ölçülmüş** süre.
Çıktı: kare başına hareket promptu, `references/prompts.md` şablonuyla.

Kare başına 8 saniyeyi aşan hareket uzundur — kareyi böl ya da sahneyi böl.

## 7 · Teslim

Sıralı numaralı paket: 9:16 kareler + kare başına prompt + ses dosyaları + hangi sesin
hangi kareye gittiğini gösteren manifest. Dağınık dosya bırakmak teslim sayılmaz.

---

## Her turda geçerli

**Her görsele gözle bak.** 1000px'e indir, aç, bak. Üç hata metrikte **görünmez**:
kadro taşması · kast kayması (hemşire ikinci doktora dönüşmüş) · metin kırpması
(9:16 kırpmada kenardaki yazı uçar). Reddedileni `*-REJECTED.png` diye sakla, sebep dosya adında.

**Bölmek görsel üretmez.** Bir sahne ikiye ayrıldığında elindeki kare sayısı artmaz.
Parça sayısı kareyi aşıyorsa parçalar kareyi paylaşır ve o kayıt **işaretlenir** —
yeniden üretim listesi budur. İşaretlemeden geçme.

**Yeniden numaralandırma bağları kırar.** Sahne numarasına bakan her şey — sheet'lerin
sahne listeleri, grid anahtarları, ses manifesti, yorumlar — numara değişince sessizce
yanlış yeri gösterir. Numarayı değiştirdiysen bunların hepsini aynı turda taşı.

**Doğrulama render'la yapılır.** "Syntax OK" yeterli değil. Sayfayı çiz, DOM'da say,
kırık link tara. Kendi diff'ine bakarak "çalışıyor" deme.

**Promptu kaydetmeyen üretim bitmemiştir.** Her görselin promptu, bağlanan referansları
ve modeli, o görselin **kendi kaydına** yazılır (sahneye değil — bir sahnenin ömründe
birden çok görsel olur, sahneye yazarsan ilk yeniden üretimde kayıt yalan söyler).
Alanlar: `prompt` · `gen_refs` · `model` · `generated_at`. Karakter ya da stil değişip
"hepsini yeniden bas" dendiğinde elde olması gereken şey budur. Referansları isimli
sheet'e bağlayan kısa bir anahtar da yaz; ham URL "bu karakter nerede kullanıldı"
sorusunu cevaplamaz.

**Immutable cache.** `immutable` yüklenen anahtarın üstüne yazma — edge bir yıl eskisini
servis eder. Yeniden üretilen dosya `-v2` gibi **yeni anahtara** gider.

---

## Referanslar

- `references/prompts.md` — style key, sahne, hareket ve anlatım şablonları
- `references/production.md` — tuval geometrisi, paralellik, teslim düzeni, cache


## Paket eki: `references/production.md`

Kaynak SHA256: `e8bb6114d373eb3f120982e3823324282c80927d93abd3ffd0c8b54b3810f663` · UTF-8 boyutu: 4768 bayt.

````text
# Üretim mekaniği

## Tuval geometrisi

`gpt_image_2`'de 21:9 yok. Bir grid'de kaç kare istediğin hangi tuvali seçeceğini belirler,
tuval de kırpma oranını. Hedef kare oranı dikey video için 9:16 = **0.5625**:

| Kare | Tuval | Çıkan kare oranı | 9:16'ya kırpma |
|---|---|---|---|
| 1 | 9:16 | 0.5625 | yok |
| 2 | 1:1 | 0.500 | %11 |
| 3 | 16:9 | 0.593 | %5 |
| 4 | 16:9 | 0.444 | %21 |

**Üç kare tatlı nokta** — %5 kırpma göze görünmez. Dört karede %21 kırpma var, özneyi
panel ortasına toplamazsan kenarlar gider.

**Altı kare tek tuvale sığmaz.** Uzun sahne iki grid'e bölünür ve kayıt dizi olur.

### Dilimlenmiş kayıt

Bölünmüş sahneler tek görseli paylaşır, her biri karelerin bir aralığını alır:

```json
{ "image": "…/scene-20.jpg", "panels": 2, "image_panels": 4, "from": 3, "to": 4 }
```

`panels` = bu sahnenin gösterdiği kare · `image_panels` = görseldeki toplam kare ·
`from`/`to` = hangileri. **Konumlandırma `image_panels`'a göre hesaplanır**, `panels`'a göre
değil. Kare gösteren her kod bunu bilmek zorunda.

Parça sayısı mevcut kareyi aşarsa parçalar kareyi paylaşır ve kayda `needs_art: true` düşer.
Bu işaret yeniden üretim listesidir; koymadan geçme.

---

## Paralellik

Tutarlılığı **önceki kare taşımaz** — her üretime eklenen sabit sheet'ler taşır (style key +
karakter + mekân + prop). Önceki grid yalnızca bölüm içi akış içindir.

Sonuç: **bölümler paralel üretilebilir.** Her bölüm kendi açılış sahnesinden dallanır,
bir öncekinin bitmesini beklemez. Gerçek sınır modelin eşzamanlı iş kotası —
Higgsfield ultra planında **8**.

Ses tarafında da paralellik var ama **bariyer var**: tüm ses bitmeden kare çizilmez.
Ölçülen süre sahne sayısını belirler.

---

## Ses üretimi

- Komşu blokları `previous_text` / `next_text` ile bağla. Bunlar okunmaz, sadece modele
  bağlamı verir; olmadan her parça yeni cümle gibi okunur ve kesim yerinde ton sıfırlanır.
- Ayarları seri boyunca sabit tut. Ortada değişen `stability` duyulur.
- Üret → `ffprobe` ile ölç → sınırı aşan varsa tekrar böl, tekrar okut.
- Manifest tut: hangi dosya hangi sahnenin, kaç saniye, hangi URL, yeniden mi okundu.

Planlama tahmini (asla kayda geçmez): sakin İngilizce anlatım ≈ **15 karakter/saniye**.
Ölçüm her zaman bunu ezer.

---

## Depolama ve cache

Dosyalar `immutable` yükleniyorsa **bir anahtarın üstüne asla yazma** — edge eskisini
bir yıl servis eder ve sen doğru dosyayı yüklediğine bakıp "oldu" dersin, olmaz.

Yeniden üretilen dosya yeni anahtara gider: `scene-33.jpg` → `scene-33-v2.jpg`, sonra
kayıttaki URL güncellenir.

Yeni bir kesim/versiyon üretiyorsan tamamını **ayrı bir önek** altına koy
(`voiceover/10s/…`). Böylece eski kesim ayakta kalır, geri dönmek tek satır olur.

---

## Yeniden numaralandırma

Sahne numarası değiştiğinde numaraya bakan **her şey** sessizce yanlış yeri gösterir.
Aynı turda taşınacaklar:

- grid kayıtlarının anahtarları
- ses manifestinin anahtarları
- karakter / mekân / prop sheet'lerinin sahne listeleri
- sahneye bağlı yorumlar
- "önceden şöyleydi" notlarındaki numaralar

Client numarasını ayrı bir alanda sakla (`client_no` + parça harfi) ve **asla yeniden
numaralandırma** — client storyboard'unu o numarayla konuşuyor.

Bir kere kaçırıldı: 34'ten 42'ye geçildi, sheet'lerin sahne listeleri 34'lük düzende kaldı
ve kartlardaki tıklanabilir numaralar aylarca yanlış sahneye atladı.

---

## Teslim paketi

```
grids/       sahne-NN.jpg            (kareye bölünmüş 9:16 görseller)
prompts/     sahne-NN.txt            (kare başına hareket promptu)
voiceover/   ep-N/sahne-NN.mp3
MANIFEST.json                        (sahne → kare → ses → süre eşlemesi)
```

Sıralı, numaralı, eksiksiz. Dağınık dosya bırakmak teslim sayılmaz — Serhat bunu
kopyala–yükle–tık ile çalıştıracak, arada karar vermek zorunda kalmamalı.

---

## Doğrulama

Üretilen her görsele **gözle bak**. 1000px'e indir, aç, bak. Metrikte görünmeyen üç hata:

| Hata | Nasıl anlaşılır |
|---|---|
| kadro taşması | özne panel sınırını aşmış, kırpmada kesilecek |
| kast kayması | hemşire ikinci doktora dönüşmüş, karakter kaymış |
| metin kırpması | yazı kenarda, 9:16 kırpmada uçacak |
| ölü kare | panelde hareket edecek hiçbir şey yok |

Reddedileni sil**me**: `*-REJECTED.png` diye sakla, sebebi dosya adına yaz. Aynı hataya
ikinci kez düşmemenin tek yolu bu.

Pano/arayüz tarafında doğrulama **render'la** yapılır: sayfayı headless çiz, DOM'da eleman
say, kırık link tara. Kendi diff'ine bakıp "çalışıyor" deme.

````


## Paket eki: `references/prompts.md`

Kaynak SHA256: `ac2162e4214de152f77da6c73338e6ddd20f212ed4ff15f12afd9e94b2f24fbd` · UTF-8 boyutu: 5716 bayt.

````text
# Prompt şablonları

**Bütün görsel ve video promptları İngilizce yazılır.** Sadece anlatım metni kullanıcının
seçtiği dilde kalır. Model İngilizce görsel talimatı belirgin biçimde daha iyi anlıyor,
anlatım ise seslendirmeye gidiyor — ikisi farklı yere akıyor.

Filmi tutarlı tutan iki şey var ve **ikisi de gerekli**: her üretime bağlanan tek style key
görseli, ve her prompt'a birebir yapıştırılan aynı STYLE metni. Görsel olmadan metin kayar,
metin olmadan görsel yorumlanır.

---

## STYLE metni

Bir kere yaz, her yere yapıştır: medium, palet, çizgi karakteri, dolgu davranışı, doku/finish.

```text
layered construction-paper collage, torn matte edges, flat opaque fills, soft drop shadows,
visible paper grain, warm cream ground
```

Örnekler:

- `flat 2D vector animation, bold clean outlines, solid vibrant flat fills, no shading`
- `hand-painted storybook gouache, soft textures, warm muted palette, visible brush strokes`
- `strict monochrome minimalism, black silhouettes on white void, high contrast, deep negative space`
- `hand-inked black marker on off-white paper, solid jet-black fills, thin white scratch highlights`

Palet katıysa hex ver. "Mavi tonları" diye yazarsan her sahnede başka mavi gelir.

---

## Style key

Karakter, mekân, yazı **içermeyen** saf bir stil plakası. Sheet tekniği anlatır, içerik değil.

```text
Pure {STYLE} style-reference plate. An abstract style swatch: a balanced arrangement that
demonstrates the rendering grammar clearly — {line quality}, {fill behavior}, {edge and
highlight behavior}, {texture or grain}. {palette constraint, hex values if strict}.
High contrast, clean ground, generous negative space.
```

Maskotlu seri için:

```text
{STYLE}. Full-body character: {HOST} — a {species/persona} narrator, expressive, clear
readable silhouette, facing camera, centered, plain ground. Recurring-character design.
```

### Referans görselle çalışırken

Kullanıcı örnek görsel verdiyse, prompt'un başına birebir şunu koy:

```text
Take only the visual render style and colour grading of the input image(s); mix the styles
if there is more than one. Use the render style only, and follow the instructions below:
{asıl prompt}
```

Referans görsel **stil bağışçısıdır**. İçindeki insanları, yazıları, logoları, nesneleri
kopyalama — kullanıcı açıkça istemedikçe.

---

## Sahne (first frame grid)

```text
{STYLE}. A {N}-panel storyboard strip, {N} equal vertical panels side by side, each panel a
complete 9:16 composition with its subject centred and clear of the panel edge.

Panel 1: {tek net an — kim, nerede, ne yapıyor}
Panel 2: {…}
Panel {N}: {…}

Consistent characters and location across all panels, matching the attached reference sheets.
```

Kurallar:

- **Her panelde hareket edebilecek bir şey bırak.** Panel video olacak. Sadece yazı taşıyan
  panel, altı saniye boyunca hiçbir şey yapmayan bir görüntüdür — ve bu hata saniye/kare
  metriğinde görünmez, gözle bakmak gerekir.
- **Özneyi kenardan uzak tut.** Grid 9:16'ya kırpılacak; kenara yaslanan yazı ve yüz uçar.
- Panel başına **tek** net eylem. İki şey anlatan panel ikiye bölünmelidir.
- Karakter kimliğini prompt'a yazma — referans sheet'i bağla. Yazarsan her sahnede biraz
  başka biri gelir.

---

## Hareket (video promptu)

First frame elindeyken prompt **statik kareyi yeniden tarif etmez** — modelde zaten var.
Sadece neyin nasıl hareket ettiğini söyler.

```text
STYLE REFERENCE: match the attached image exactly — {STYLE metni}. Every element rendered
in that identical style.
SCENE: {kareden başlayarak ne oluyor}
MOTION: {kamera hareketi + özne hareketi — slow push-in, gentle drift, paper pieces settling}
AUDIO: {ortam sesi veya müzik — asla konuşma}
NEGATIVE: photorealism, 3D render, lip-sync, captions, on-screen text, logos, watermark
```

**Pozitif prompt kuralıyla NEGATIVE satırının çelişkisi burada çözülür:** sahneyi pozitif
tarif et — NEGATIVE satırı sahne yönetmek için değil, yalnız **stil kayması, fotogerçekçilik,
altyazı ve filigran** yasakları için var. "Kadın üzgün olmasın" NEGATIVE'e yazılmaz;
"kadın sakin ve rahat" diye SCENE'e yazılır.

Örnek:

```text
STYLE REFERENCE: match the attached image exactly — layered construction-paper collage, torn
matte edges, flat opaque fills, soft drop shadows, visible paper grain, warm cream ground.
Every element rendered in that identical style.
SCENE: The young man sits back in the yellow armchair and lets his shoulders drop; the paper
leaves of the plant behind him lift very slightly.
MOTION: Very slow push-in. Cut paper pieces settle a few millimetres as if just placed.
AUDIO: Quiet room tone, a soft paper rustle.
NEGATIVE: photorealism, 3D render, lip-sync, captions, on-screen text, logos, watermark.
```

---

## Anlatım bloğu

Blok başına tek düz satır. Zaman kodu, duygu notu, parantez içi yönerge, sahne direktifi yok —
metnin tamamı okunacak.

```text
Block 1
For four and a half thousand years the pyramids have stood against the desert, silent and immense.
Block 2
They rose along the Nile, the river that fed a civilisation ruled by pharaohs believed to be gods.
```

Kurallar:

- Sayıları **yazıyla** yaz — "4500" okunurken bozulur.
- Tonu kelime seçimiyle kur, talimatla değil.
- "Bu videoda" deme.
- **Client metni varsa hiçbiri geçerli değil:** metin olduğu gibi okunur, sadece bölünür.
  Sayı yazıyla yazılmamışsa bile dokunulmaz — düzeltme önerisi client'a gider.
- Taslak işaretleri (`[TASLAK]` gibi) panoda kalır ama **okunmaz**; seslendirmeye giderken
  temizle.

````
