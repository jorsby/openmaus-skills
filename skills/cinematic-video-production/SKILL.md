---
name: cinematic-video-production
description: ESKİ ELLE HAT (v1) — sinematik film için element sheet’leri ve blok promptları hazırlar, kullanıcı üretimi Higgsfield’da kendi yapar. Jorsby Studio v2 op’larıyla gerçek üretim için `jorsby-studio` skill’ini kullan. Bunu YALNIZ kullanıcı açıkça elle kopyala-yapıştır akışı istediğinde, ya da v1’in prompt yazımı / staging yasaları / size sheet arşivine bakılacağında aç. film yapalım tek başına bu skill’i açmaz.
trigger: /cinematic-video
---

> Paket ekleri aşağıda “Paket eki” başlıklarında tam metindir. Metindeki göreli paket yolları bu bölümlere karşılık gelir. Yalnız ihtiyaç duyulan bölümü oku. Scriptler disk üzerinde kurulu değildir; çalıştırma gerekirse onaylanan iş kapsamında geçici dosyaya çıkarılıp doğrulanır.


# cinematic-video

Fotogerçekçi kısa film üretim hattı. Seedance 2.5 (Higgsfield) üzerine kurulu.

**Ben video üretmem.** Element'leri MCP ile üretirim, promptları yazarım,
paneli güncellerim; kullanıcı seçer ve videoyu Higgsfield'da kendi üretir.
Aksi açıkça söylenmedikçe bu böyle.

| Referans | Ne zaman aç |
|---|---|
| `references/prompts.md` | Prompt yazarken, element/size sheet kurarken, bozuk klip teşhis ederken. **Blok yazmadan önce mutlaka.** |
| `references/production.md` | Panel kurarken, JSON şemasına bakarken, festival kurallarını çıkarırken |

---

## Değişmez kurallar

1. **HER görsel üretim `gpt_image_2 · resolution: 1k · quality: low`.**
   İstisnasız, her aşamada, her element, her sheet. Hâlâ test ediyoruz —
   yüksek kaliteye ancak kullanıcı açıkça "final" dediğinde geçilir.
   Aksi söylenmedikçe bu ayar sabittir; başka görsel modeli kullanılmaz.
   Aşama başına 5 aday üretilir, kullanıcı bir tanesini seçer.
2. **Hiçbir aşama, bir öncekinin finali kilitlenmeden başlamaz.**
   Kilitlenen her şey bir sonrakinin girdisidir.
3. **Sheet aşaması yoktur.** Style sheet faceless/animasyon işidir — orada bir
   "stil" seçersin. Fotogerçekçi filmde seçilecek stil yok; palet, ışık, lens
   kararları hikaye dosyasında yazılı ve her prompta metin olarak girer.
   World sheet de denendi: güzel görünüyor, hiçbir işe yaramıyor. Doğrudan
   elementlerden başla — **karakterler → mekânlar → objeler.**
4. **Pozitif prompt.** İstenen şey tarif edilir. Tek istisna blok şablonundaki
   NEGATIVE satırı — oraya sadece *gerçekten görülmüş* hatalar yazılır.
5. **Blok 20 saniye tavan, shot 4-6 saniye.** Süre kararı ilk karardır;
   sonradan değiştirmek her şeyi söker.
6. **Karakter iki yaşta görünüyorsa iki ayrı element.** Aynı yüzün genç ve
   yaşlı hali tek element'ten çıkmaz.
7. **Panel gerçeğin kaydıdır.** Her üretimden sonra ilgili JSON güncellenir.
   Panelde görünmeyen iş yapılmamış sayılır.

---

## Hat

| # | Aşama | Çıktı | Kim |
|---|---|---|---|
| 0 | Hikaye | logline · beat sheet · diyalog · ses tasarımı | ben, kullanıcı onaylar |
| 1 | Blok kararı | süre + blok sayısı + model | kullanıcı seçer |
| 2 | **Elementler** | karakter → mekân → obje, her biri 5 aday → seçim → final | ben üretir, kullanıcı seçer |
| 3 | Size sheet | ölçek gerektiren bloklar için | ben |
| 4 | Blok promptları | blok başına tam prompt | ben |
| 5 | Video generation | klipler | **kullanıcı elle** |
| 6 | Kurgu + ses | final dosya | kullanıcı |

### 2 · Elementler

Tarifler `references/prompts.md` içinde: karakter sheet'i düz ve sıkıcı
ışıkta, mekân sheet'i **ışık koşulu başına bir element**, obje sheet'i elsiz.
Aynı karakter iki yaşta görünüyorsa iki ayrı element.

---

## Hikaye çekirdeği zayıfsa

**Belirti:** hikaye her turda tamir istiyor, bir parçayı düzeltince başkası
bozuluyor. Sorun detaylarda değil, çekirdekte.

**Test:** karakterin ne istediği tek cümleyle söylenebiliyor mu? Söylenemiyorsa
konsepti değiştir, tamir etme. Herkesin olaylara *maruz kaldığı* bir hikayede
kimse bir şey istemiyordur ve hiçbir tamir bunu kurtarmaz.

Festival işiyse kuralları **önce** oku; süre sınırı, platform-içi üretim
zorunluluğu, watermark/packshot ve public post şartı hikaye kararlarını
değiştirir. Jüri belliyse konsepti jüri × kriter matrisine karşı puanla —
aranan, en yüksek toplamı alan tek aday değil, **zayıflıkları birbirini
tamamlayan iki aday**.


## Paket eki: `references/production.md`

Kaynak SHA256: `6232b786f4a4eb001e1199b9281247c7ea1fe62256996556fa41aea11fa378e9` · UTF-8 boyutu: 4600 bayt.

````text
# Üretim — panel, model ayarları, dosya düzeni

## Model ayarları

| İş | Model | Ayar |
|---|---|---|
| **Tüm görsel üretim** | `gpt_image_2` | `resolution: 1k` · `quality: low` · 5 aday · **istisnasız** |
| Final (yalnız kullanıcı "final" derse) | `gpt_image_2` | `resolution: 2k` · `quality: high` |
| Video blokları | `seedance_2_5` | `mode: t2v` · `1080p` · `21:9` · `generate_audio: true` · 4-30 sn |

Toplu üretimde `generate_image_batch` → `jobs_wait` (12'lik gruplar) →
tek `show_generation_by_ids`. Sonuçları `curl` ile panel `assets/` altına indir;
CloudFront URL'lerine bağlı kalma.

**Kural:** test aşamasındayız. Her element, her aşama `1k · low` ile üretilir ve
başka bir görsel modeli kullanılmaz. Yüksek kalite ancak kullanıcı açıkça
istediğinde açılır — kendiliğinden "artık final üretelim" denmez.

## Panel

Sıfır bağımlılık, tamamen lokal. `./studio/serve.sh` → `http://localhost:4173`.
Panel 6 saniyede bir JSON'ları kendi yeniler; ben üretip JSON'u güncellerim,
kullanıcı hiçbir şeye basmadan görür.

```
studio/
  serve.sh          python3 -m http.server 4173
  index.html        style + panel iskeleti
  app.js            render + etkileşim
  tokens.css        sickkids-board'dan kopya
  data/             overview · competition · project · elements · blocks
  assets/           elements/ · renders/
```

### JSON şeması

| Dosya | Ne tutar |
|---|---|
| `data/overview.json` | `goal` (headline, body, method[]) · `concepts[]` → `id · title · status · logline · want · scores{juror} · why{juror} · risk · note` |
| `data/competition.json` | `prizes[]` · `rules[]` · `criteria[]` · `jury[]` → `id · name · hue · credit · wants · wins · loses` |
| `data/project.json` | `film` (başlık, format, deadline) · `brief` (headline, body, facts) · `aims` · `decisions` |
| `data/elements.json` | `elements[]` → `id · type · name · hue · note · fragment · image · status` |
| `data/blocks.json` | `film` · `statuses[]` · `blocks[]` → `id · act · tc · duration · summary · elements[] · prompt · status · render · notes` |

**Element aşamaları:** `pending` → `drafts` → `picked` → `final`
**Blok durumları:** `pending` → `elements` → `prompt` → `rendered` → `approved`

`elements[]` içindeki `hue` panelde renk kodlaması için: rose · amber · olive ·
teal · slate · plum · indigo. `fragment` alanı, o element'in prompta girecek
İngilizce cümlesi — panelde okunabilir dursun ki prompt yazarken kopyalanabilsin.

Blok `notes` alanında `⭐` varsa panel o bloğu yıldızlar — filmin belkemiği olan
ve önce test edilmesi gereken bloklar için.

### Sekmeler

**Overview** (amaç + konsept × jüri matrisi) · **Competition** (kurallar, ödüller,
jüri profilleri) · **Project** (brief, hedefler, kararlar) · **Characters** ·
**Locations** · **Props** · **Storyboard** (tablo / pano / şerit görünümleri).

Overview ve Competition proje-üstüdür: birden fazla proje olunca bu ikisi ortak
kalır, alt projeler kendi Project + element + storyboard sayfalarını taşır.

### Tasarım dili

`sickkids-board` panelinden alınır: `tokens.css` kopyalanır, yapı korunur —
brand + meta chip'ler · sayaçlı tab'lar · `sec-head`'li kartlar · plates grid ·
storyboard tablosu + pano/şerit görünüm değiştirici · lightbox · empty state'ler.
Light varsayılan, dark toggle, tercih `localStorage`'da.

## Üretim sırası

Filmin **belkemiği** ve **afişi** olan iki bloğu önce test et. Belkemiği,
teknik olarak en riskli olan (karanlık, tek ışık kaynağı, uzun iniş gibi);
afiş, jürinin hatırlayacağı tek kare. İkisi tutuyorsa yapı sağlamdır; biri
tutmuyorsa yapıyı değiştirmenin zamanı üretimin başıdır, sonu değil.

## Vibe testi

Hikaye kilitlendikten sonra, element üretimine geçmeden önce birkaç `t2v`
denemesi yap — sadece dokuyu görmek için, karakter tutarlılığı aramadan.
Amaç şu soruları erken cevaplamak: sis/su/partikül fiziği tutuyor mu, palet
tutarlı mı, `generate_audio` istenen sesi taşıyor mu, en riskli ışık kurgusu
çalışıyor mu.

**Video promptunda ölü kamera dili:** `static camera`, `holds still`,
`slow imperceptible drift`. Model hareket ipucu alamayınca fotoğrafı hafifçe
kıpırdatır. Her promptta üçü birden olmalı: sahnede fiziksel bir eylem,
somut bir kamera fiili (`pushes`, `cranes`, `whip-pans`, `tips over`), ve
sürekli hareket eden bir çevre öğesi (sis yırtılıyor, taş dökülüyor, gölge
oynuyor).

````


## Paket eki: `references/prompts.md`

Kaynak SHA256: `9b5910eb55dce1a6ba8f97754cdc9d3fe103fe592afdcfba8812839cf9b443d9` · UTF-8 boyutu: 20545 bayt.

````text
# Prompt yazımı — Seedance 2.5

A prompt is not a description of a scene. It is a **set of constraints tight enough that
the model's own habits cannot fill the gaps.** Everything below exists because the model
has a default — a default framing, a default gesture, a default body size, a default
glowing sword — and a prompt that leaves a hole gets the default poured into it.

---

## The 20-second law

**One generation is at most 20 seconds.** Every structural decision follows from this.

| | |
|---|---|
| Block length | 20 s ceiling |
| Shots per block | 3–5 |
| Shot length | **4–6 s. Never longer.** |
| A 20-minute film | ~60 blocks |

Six seconds is the real limit, not a style preference. Past it the model runs out of
written action and starts inventing: people stand up who were told to kneel, costumes
change, props morph. Give a shot less time than it can fill and it stays obedient.

---

## A block is a scene, not a shot group

Continuity is guaranteed **only inside one generation.** Run the same prompt twice and you
get a different framing, a different hour of the day, different set dressing — the faces
stay, nothing else does.

So a block is a **self-contained scene**, and everything belonging to one moment lives in
one block. Between blocks you cut in time or place, never within a continuous action.

This sounds like a limitation. It is closer to a style: it forces elliptical, chaptered
storytelling, which is what restrained cinema does anyway.

**Corollary:** you cannot storyboard on paper. Framing is not reproducible. Shoot, then
select.

---

## The block template

Fill the braces. Order matters — the model weights earlier tokens more, and a constant
clause in a constant position lands the same way every time.

```
STYLE REFERENCE: {medium}, {lens/format}, {focus}, {saturation & contrast law}, {finish}.
PALETTE LOCK: use ONLY {3–5 named colours}.
@{size-sheet} is a SIZE CHART, not a scene. Do not stage it, do not show it, do not copy
its backdrop or its ruler. Read the size relationships from it and hold them in every shot.
An object does not shrink when someone picks it up. The body strains to match the object;
the object never resizes to match the body.
IDENTITY LOCK for @{character}: the same face, bone structure, {hair} and skin as the
reference. {Garment, garment, garment}, in every shot. {Height or "an ordinary man"}.
{Repeat one IDENTITY LOCK block per character. If a character has non-human anatomy,
restate it here in full — see Anatomy below.}
REFERENCES: @{size-sheet} = the sizes. @{character} = {NAME}. @{location} = {the place,
one clause}. @{prop} = {what it is and how big, one clause}.
STAGING LAW: {only when bodies of unlike size share the frame — see Staging}
LIGHT, unchanging: {key: source, direction, colour}; {fill or ambient}; {atmosphere}.
REGISTER: {how the subject behaves as a matter of character, not emotion}
A {N}-second scene of {THREE|FOUR|FIVE} hard-cut shots, {one place / three places}. The
first frame is already the scene in progress. {Population count.} Nobody walks anywhere.
SHOT 1 — 0.0s to {a}s — {SIZE}, {static / move}, {angle and height}: {composition: who is
where in the frame, what is behind them, where the light falls}. {ONE action.} {2–3
micro-beats.}
Hard cut to.
SHOT 2 — {a}s to {b}s — …
Hard cut to.
SHOT 3 — {b}s to {N}s — …
{K} shots, hard cuts at {a}s and {b}s. No dissolves, no fades. One action per shot.
Continuous small motion inside each shot.
AUDIO: {diegetic cues in order}. Diegetic sound only. No music, no narrator.
NEGATIVE: {6–12 items, only the failures you have actually seen}
```

`Hard cut to.` is a literal scene-edit token. Without it, cuts collapse into smooth motion.

Time codes are respected within about a second — and they work as an **event schedule
inside a single uncut shot** too, not just as cut points.

---

## Writing a shot

### Write composition, not content

"Only her hand in frame, no face, no background" produces a flat macro that looks like a
medical photograph. A frame is a construction. Say what it is:

> size · angle · camera height · where the subject sits in the frame · what is behind ·
> where the light comes from · what line the body makes

**Weak:** `INSERT on his shoulder. No face, no fire.`
**Strong:** `MEDIUM CLOSE-UP, static, from slightly behind and above. His bare shoulder
fills the lower left, the line of his neck and one braid at the top edge, dark cave wall
behind, a warm spill from the fire below frame edging the skin. The black bar crosses the
frame as a hard diagonal.`

Same information about content. Completely different image.

### One shot, one event

Stack two events in a frame and the model plays both at once and lands neither. Lift the
object *or* show it burn him — not both. If the audience needs to notice something small,
**cut to it**; don't hide it inside a wide.

### Micro-beats, not inner states

`unafraid`, `grim`, `determined` render as a blank face. The model animates muscles, not
feelings. Two or three concrete physical beats per cut:

> narrows his eyes · wets his lips · swallows once · his jaw tightens · breathes out
> through his nose · blinks slowly · the corner of his mouth pulls and settles

### Consequence beats

Don't state a property — show what it does to the world. This is the cheapest way to make
a frame feel photographed.

**Weak:** `The iron is very hot.`
**Strong:** `Where the metal rests on the fleece one thin thread of white smoke lifts and
the wool darkens and curls along that edge.`

Three rules: make it **local** (name where), make it an **event** with a beginning and an
end (not a state), and allow **one per shot** — more than that is a fireworks display.

### Write forward, not backward

The model reads the words in your negatives too. Prefer the positive form and keep the
negative tail for failures you have actually seen in a render.

| instead of | write |
|---|---|
| no blurry background | deep focus, everything sharp front to back |
| don't make him look young | mature bone structure, defined jaw, weathered skin |
| no modern clothing | undyed wool, sheepskin, hand-forged iron |
| don't stand still | *(rewrite the beat: give him one contained physical action)* |

### Physics is never assumed

The model draws what you write without asking whether it is possible. Three things it will
not supply for you:

- **Duration.** A process that takes minutes cannot happen in 20 seconds. Shoot the
  *middle* of the process — the object starts already half-changed.
- **Consequence on the body.** Hot, sharp, heavy: if you don't write it, a character will
  casually palm white-hot iron. Write the wrap of cloth, the far-end grip, the weight in
  the posture.
- **Locality of a state.** Say *where* the heat, the wet, the damage is, or it spreads to
  the whole object and the object loses its identity. A spit glowing along its whole
  length stops being a spit and becomes a sword.
- **Visible cause.** Something glowing with no source in frame reads as fantasy. Show the
  iron coming out of the coals.

---

## Staging laws

These are not style choices. Each one is a failure mode with a workaround.

**Nobody walks.** Locomotion does not render. Ask for a man crossing a room and you get a
man standing still with an arm outstretched — the model's default idle. Stage
**station to station**: the character is already where he is and does one contained thing.
Movement between places happens in the cut.

**Same distance from camera, or scale dies.** Two bodies of unlike size hold their ratio
only when they are the same distance from the lens, ideally sharing a ground line. Put the
small one deep and the large one near and perspective doubles the difference. If you must
separate them in depth, put the **large** one further back — underselling a giant is
recoverable, overselling it is not.

**The action's prior beats your references.** Ask for a fight between a man and a giant and
the model reaches for "two men fighting", which normalises their heights — the size sheet
loses. Same-size fights work fine. So: keep unlike-size bodies out of physical struggle,
and build those scenes from before, after, inserts and reaction instead of the contact.

**Population, stated per shot.** `Exactly two figures in frame` prevents the extra body
that wanders in.

**Anatomy is restated every single time.** A non-human trait baked into a character
element will not survive into video on its own. Write it out in the prompt, using both the
archetype word the model already knows and an explicit negation of the default:

> He is a CYCLOPS. ONE eye in total on his entire head, a single large eye set high in the
> centre of his forehead. Where a human's two eyes would sit there is only smooth unbroken
> skin — no eyes, no eyelids, no sockets there.

**Costume, restated every single time.** Named garments in every shot's IDENTITY LOCK. The
further into a block a shot sits, the weaker the top-of-prompt description gets. And watch
the adjectives: the word *bare* in "bare shoulder" will undress the whole character.

---

## Reference elements

Elements are named images registered per workspace and mentioned as `@name` in the prompt.
They carry **identity, place and size**. They do not carry anatomy, and they do not carry
framing.

Judge a reference image by "can a model read the identity out of this", never by "is this
a nice picture".

**Pre-flight:** when you paste a prompt, every `@name` should resolve to an element chip.
If one stays plain text, that element is not attached and the clip is wasted — stop and
fix it before generating. Newly created elements sometimes need a page refresh to appear.

### Character element

Flat, boring, evenly lit — on purpose. The video model reads this image as *who the person
is*, so anything else in the frame becomes part of the person. Bake in a warm rim light and
the character arrives rim-lit in every night interior.

```
Character reference sheet, {aspect}, neutral mid-grey seamless backdrop, soft even frontal
light, no rim light, no coloured light, no cast shadows on the backdrop, deep focus.
Left panel: full body, standing upright, feet flat, arms relaxed at sides, head to toe in
frame. Right panel: tight chest-up portrait of the same character.
{Age band, build, heritage — an original character, never a real person's likeness.}
Face: {shape, jaw, cheekbones, nose, lips}, mature bone structure.
Eyes: {shape, colour}, naturally muted catchlights, no specular glare.
Hair: {colour, length, style, finish, parting}.
Visual anchor: {the one unmistakable identifier — a scarred brow, a red sash, a shaved head}.
Wardrobe head to toe, no gaps: {top} → {layers} → {bottom} → {belt/sash} → {shoes} →
{jewellery}, each with material, cut and colour.
Hands empty. Expression neutral.
{style constant only — no light clause}
NEGATIVE: exactly one person per panel, no other people, no duplicate figures, no props,
no furniture, no cast shadows on backdrop, no rim light, no coloured lighting, no depth of
field, no beauty filter, no plastic skin, no text, no watermark, no frame borders.
```

Give every character **one visual anchor** and let no two share it. At video resolution
under motion blur, fine facial structure dissolves; the anchor is what makes a viewer say
"that's him" across a cut.

### Location element

**Light lives in the location element, not in the style line.** You cannot talk a daylit
cave into night — the element's own light wins every time. So build **one element per
lighting condition** and choose the light by choosing the element.

```
Wide establishing shot of {place}, {aspect}, eye level, deep focus, empty — no people, no
animals, no action.
Geometry legible: {walls, openings, floor, ceiling, depth}.
One object of known size for scale: {a doorway, a cart, a step}.
Materials: {drawn from the world of the film, named specifically}.
Light: {this element's one lighting condition — name it in the element's name too}.
{style constant + this location's light clause}
NEGATIVE: no people, no figures, no animals, no text, no signage, no watermark, no frame
borders, no lens flare, no depth of field.
```

Note the trade-off: a location element pulls framing **wider**, because the image it
carries is a wide establishing shot. A shot is either close or located. For a close-up,
drop the location element and describe the background in one clause.

### Prop element

```
A single isolated object on a plain neutral mid-grey seamless background, three-quarter
angle, centred, soft even light with one gentle contact shadow, deep focus.
{The object: material, construction, wear, age — the things that make it THIS object.}
{Its real size expressed as a comparison: "about as long as a man's forearm".}
{style constant, trimmed — medium, palette, finish only}
NEGATIVE: no hands, no people, no other objects, no scabbard or container, no glowing
metal, no text, no watermark, no frame borders.
```

No hands in the image. A hand in the reference becomes a hand in every shot.

### Size sheet — the one that saves the film

The single most valuable element you can build. Scale stated as a number in the prompt does
not hold; scale carried by a picture does.

Build it **from the character and prop elements themselves**, so the faces and costumes on
the sheet match the ones in the film. Attach it to any block where relative size matters.

```
A film production SIZE CHART. Every subject stands or rests on ONE single flat ground line,
photographed straight on at eye level with a long lens, no perspective distortion, plain
neutral mid-grey seamless backdrop, flat even frontal light, no cast shadows, deep focus.
Everything complete and uncropped, with headroom above the tallest subject.
Left to right along the ground line, evenly spaced:
1. @{character} — full body head to toe, standing upright, arms at sides, keeping the exact
   face and clothing of the reference. HEIGHT {n} cm. THIS ONE IS THE UNIT OF MEASURE.
2. @{prop} — {resting position}, keeping its exact material and construction. LENGTH {n} cm
   — {what it reaches on the unit character}.
3. @{other character} — … HEIGHT {n} cm — {ratio stated as body landmarks: "the top of the
   man's head reaches his belt; his knee is level with the man's hip; his hand is the size
   of a man's head"}.
Along the left edge a thin white vertical ruler with tick marks labelled in cm, faint guide
lines across the frame, and each subject's height printed beneath it in small white text.
NEGATIVE: cropped subjects, different ground levels, low angle, wide angle distortion,
extra people, duplicate figures, hands holding the props, background objects, garbled text,
watermark, dramatic lighting, depth of field.
```

Rules that make a size sheet work:

- **A human is always on the sheet.** The human is the unit. A prop alone tells you nothing.
- **Express ratios as body landmarks**, never as similes. "Reaches his belt" is a
  measurement. "Tiny against him", "no larger than", "would only come up to" are hidden
  multipliers and hypotheticals — the model renders them as instructions, or not at all.
- **Cross-check every size against the plot.** If the hero must swing the giant's sword,
  the sword has to be swingable. If the monster is blinded with a cooking spit, it cannot
  be ten metres tall. A size that contradicts the story survives into every block until
  someone notices.
- **One sheet per scene** is cheap and worth it; a single master sheet with the whole cast
  and every prop at correct relative size saves you the combinatorics.
- Always pair the mention with the guard line from the template — without it the model
  tries to stage the chart, grey backdrop and all.

---

## What the model does well, and what it doesn't

Design scenes toward the left column. When the story needs the right column, use the
workaround rather than fighting it with more words.

### Strong — build on these

| | |
|---|---|
| Static camera, one composed frame, one action | the reliable core of the whole film |
| Faces in close-up with micro-beats | the model's single best asset |
| An object changing state over time | cold iron to hot, dry to wet, whole to broken |
| Environment and atmosphere for a full 20 s uncut | never breaks, never morphs |
| Identity — within a clip and across separate generations | faces hold; this is what makes a long film possible |
| Consequence beats — smoke, sparks, steam, hiss, dust | renders beautifully and cheaply |
| Correct scale with a size sheet + bodies on one plane | including objects changing hands |
| Composed inserts — a hand, a shoulder, a grip | when written as frames, not as body parts |
| Hard cuts landing on written time codes | ±1 s |
| Same-size bodies grappling | contact, weight and effort all render |

### Weak — avoid or work around

| symptom | what to do instead |
|---|---|
| Locomotion doesn't render | station-to-station staging; move people in the cut |
| Depth destroys size ratios | put both bodies the same distance from camera |
| Unlike-size bodies in a fight normalise | cut around the contact — before, after, inserts, reaction |
| Shots over ~6 s invent business | shorter shots, more of them |
| Two events in one shot collapse | one event per shot, cut to the second one |
| Glowing metal becomes a light-sabre | keep the hot end out of frame or inside the fire; show only cold metal wide |
| Lip-sync fails, especially outside English | shoot mouths closed or turned away, lay dialogue in post |
| Anatomy reverts to the human default | restate it fully in every prompt, archetype word + explicit negation |
| Framing lands a size wider than asked | ask a size tighter than you want; drop the location element for close-ups |
| Framing and set dressing don't repeat between generations | never plan a match cut across two blocks |
| An element's own details render inconsistently | name the ones that matter in the prompt too |

---

## Diagnosing a clip that came back wrong

Look at frames before you theorise. Then fix at the right layer — the smallest one that
covers the failure. Fixing a one-shot problem in the style line breaks the other 59 blocks.

| what broke | layer to fix |
|---|---|
| one shot | that shot's beat |
| every shot in one location | the light clause, or a new location element |
| every block a character appears in | the character element |
| everywhere | the style constant or the world rules |

| symptom | cause | fix |
|---|---|---|
| Face blank, "wooden presenter" | inner states instead of muscles | 2–3 micro-beats per cut |
| Night renders as daylight | light written in text, not carried by the element | separate element for that lighting condition |
| Giant far too large | one body deeper than the other | same distance from camera, shared ground line |
| Prop changed size in hand | no size sheet, or no "objects don't resize" clause | attach the sheet, add the clause |
| Character stood up / walked off | shot too long | cut it to 4–6 s |
| Costume disappeared late in a block | costume named only once at the top | restate garments in every IDENTITY LOCK |
| Character undressed unprompted | the word *bare* in a descriptor | name the garment, add `shirtless` to the negative |
| Clip opens on a reference sheet | missing guard | `opening on a character sheet` in the negative |
| Cuts became dissolves | missing token | `Hard cut to.` between every shot |
| Object glows all over | locality unstated | name which part, keep the rest explicitly cold |

Re-rolling the same prompt bills you again for the same failure. Change the prompt.

And when one shot inside an otherwise good block is wrong, pull that shot out as its own
short clip rather than re-rolling the whole block — you keep what already worked.

---

## Before you generate

1. Every `@name` resolved to an element chip.
2. Every shot 4–6 s, and 3–5 shots in the block.
3. One event per shot, and the shot says how it is framed, not just what is in it.
4. 2–3 micro-beats on every face.
5. Nobody walks anywhere.
6. Bodies of unlike size are the same distance from camera, with the size sheet attached.
7. Non-human anatomy and every garment restated in this prompt.
8. Physics: duration plausible, consequence on the body written, state localised, cause visible.
9. `Hard cut to.` between shots; cut times restated at the end.
10. The negative tail contains only failures you have actually seen.
````
