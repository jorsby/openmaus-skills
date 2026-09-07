---
name: jstudio
description: >-
  Jorsby Studio v2 ile video projesi kurmak, üretmek, gözden geçirmek ve teslim
  etmek — MCP op'larıyla. Şu isteklerde: "yeni proje açalım", "şu episode'u
  üretelim", "take'i düzeltelim", "render alalım".
trigger: /studio
op_surface: c91e46734683
schema_version: 2
---

> Paket ekleri aşağıda “Paket eki” başlıklarında tam metindir. Metindeki göreli paket yolları bu bölümlere karşılık gelir.

# Jorsby Studio v2

Bu dosya `_base`. Her play bunun okunmuş olduğunu varsayar. Bir play açmadan
önce buradaki bölümler senin için geçerli.

---

## 0 · Kapsam ve rol

Sen bu hatta **üretimi yapan** taraftasın. Prompt paketi bırakıp çekilmiyorsun;
op'ları sen çağırıyorsun, para senin çağrınla harcanıyor.

Ne yaptığın:

- Projeyi kuruyorsun, cast'i açıyorsun, episode iskeletini yazıyorsun.
- Üretimi tetikliyorsun, sonucu yokluyorsun, take'i gözden geçiriyorsun.
- Her para harcayan adımdan önce duruyorsun ve soruyorsun.

Ne yapmadığın:

- Tuval oranı, hücre geometrisi, grid prefix'i, `idem_key`, hash — **hiçbirini
  hesaplamıyorsun.** Bunlar sistemin işi. Hesaplamaya kalkarsan sistemle
  çelişirsin ve çelişkiyi sistem kazanır.
- Sessiz karar vermiyorsun. Bayat girdiyle devam, tip değişimi, erim daraltma,
  toplu üretim — hepsi kullanıcıya söylenir.

**Üretim çağrısı hemen döner, "bitti" demek değildir.** `generate_*`
`{asset_id, job_id, status:"queued"}` ile anında döner. Bir video 4 dakika
sürer. Dönüş cevabını "üretildi" diye kullanıcıya söylemek en sık yapılan
hatadır — yoklama adabı §11'de.

Render motoru VAR (T-27): `render_episode` bölümün kurgusunu bizim Remotion
Lambda'mıza yollar, dosya callback'le R2'ye iner. Bölüm başına AYNI ANDA TEK
render (`RENDER_IN_FLIGHT`) ve kurgu değişmediyse biten render aynen geri verilir
(`reused`) — aynı kareler ikinci kez render edilmez. Bu bir üretim adımı DEĞİL:
para harcamaz, admission'a girmez, `jobs` satırı açmaz.

---

## 1 · Sözlük

| Terim | Ne demek |
|---|---|
| **slot** | Bir asset'in oturduğu yer. Sheet, frame, scene track, ikon, avatar, ses örneği. Adresi slug'dır (§4). |
| **take** | Bir slot için üretilmiş ya da yüklenmiş herhangi bir asset. Hiç silinmez. |
| **seçili** | Slot'un o an işaret ettiği take. İşareti taşıyan tek op `put`. |
| **yaşayan prompt** | Satırın kendi `prompt`/`text` kolonu. Üretim onu okur; çağrıda `prompt` parametresi **yoktur**. |
| **gen_prompt** | Üretim anında donmuş kopya — kalıp ve malzemeler işlenmiş hali. Geriye dönük okunur, düzeltilmez. |
| **otomatik küme** | Erimi proje ya da episode olan entity'lerin ana sheet'leri. Sistem her görsel üretime kendiliğinden sokar (§6). |
| **erim (reach)** | Bir entity'nin kendiliğinden nereye kadar girdiği: `project` · `episode` · `scene`. `chk_entities_reach`'in kabul ettiği üç değer bunlar; `scene` varsayılan ve otomatik kümeye **girmez** — o entity ancak prompt'ta adı geçtiğinde üretime katılır. |
| **stale** | Kolon değil, rozet. Hash karşılaştırmasından her okuyuşta hesaplanır (§10). |
| **N / shots** | Bir grid karesindeki shot sayısı, `frames.shots`. **Senin kararın**, geometri değil (§7). |
| **ingredient** | Üretime giren her şey: otomatik küme + prompt'taki slug'lar + `refs[]`. Üçü de hash'e girer. |

---

## 2 · Play yönlendirme tablosu

| Kullanıcı ne diyorsa | Aç |
|---|---|
| "yeni proje açalım", "şu brief'ten video yapalım", elinde PDF/metin var | `plays/onboarding.md` |
| "karakter ekleyelim", "şu mekânı kuralım", "stil oluşturalım", "varyasyon çıkar" | `plays/cast.md` |
| "sahneleri yazalım", "senaryoyu bölelim", "episode iskeletini kur" | `plays/episode-author.md` |
| "bu episode'u üretelim", "VO'yu al", "grid'i çıkar", "videoyu üret" | `plays/faceless.md` |
| "şu take'i beğenmedim", "eskisine dönelim", "onaylayalım", "yorum var" | `plays/review.md` |

Yazılı play seti bu beş dosya. Yazılmamış olanlar ve doğru cevapları:

- **half_avatar_half_broll** — hat kapalı. `save_project` `TYPE_NOT_ENABLED`
  döner. "Bu hat yakında; bugün faceless ya da cinematic" de, zorlama.
- **cinematic** — tip açık, ama `plays/cinematic.md` bu turda yazılmadı.
  Faceless reçetesi + bu dosyanın kuralları geçerli; farkı: frame türü sahne
  başına `first` ya da `grid` olur ve VO opsiyoneldir. `first` seçtiysen kare
  oranı projenin `ratio`'su ile birebirdir.
- **episode-run** (uçtan uca otomatik koşu) — yazılmadı. Sahne sahne
  `plays/faceless.md`'yi koştur; devam noktasını `get_episode(include:["scenes","tracks","frames","takes"])`
  ile oku. Tek çağrıda bütün episode'u üretmeye kalkma (§12).

---

## 3 · Op yüzeyi

Op listesi `packages/studio-op-contract/registry.mjs` — sayıyı oradan oku.
Açık olanların çoğu MCP tool listesindedir; kapalı olanlar listeye hiç girmez
ve çağrılırsa `OP_DISABLED` döner. Açık ama
ajan yüzeyine bilerek alınmayan dört op var (`create_workspace`,
`staff_create_workspace`, `complete_invite`, `create_agent`) — ilk üçü
tarayıcıdaki bir insanın kararı, senin değil. Dördüncüsü senin ASLA
çağıramayacağın op: `create_agent` yeni bir ajan hesabı ve canlı bir bağlantı
adresi doğurur, sunucu onu staff olmayan ve ajan olan çağırana kapatır, listede
olmaması yalnızca bağlam tasarrufudur.

Üretim — hepsi para harcar (`generate_music` kapalı):

| Op | Ne yapar | Hedef |
|---|---|---|
| `generate_image(target, refs?, params?, model?, ack_stale?, dry_run?)` | Var olan bir görsel slotunu doldurur | sheet · frame · `project/icon` · `char/x/avatar` |
| `generate_video(target, refs?, params?, model?, ack_stale?, dry_run?)` | Sahne videosunu ya da avatar track'ini canlandırır | `scene/N/video` · `scene/N/avatar` |
| `generate_voiceover(target, params?, model?, ack_stale?, dry_run?)` | Ses track'ini karakterin sesiyle, karakter yoksa şovun narrator sesiyle okur | `scene/N/voice` |
| `change_voice(target, params?, ack_stale?, dry_run?)` | Sahnenin **seçili video take'ini** aynı sahnenin sesine çevirir (§15) | `scene/N/voice` |

İşaretleme (1):

| Op | Ne yapar |
|---|---|
| `put(slot, asset_id)` | Bir take'i slot'un seçilisi yapar. **`url` alanı yoktur** — önce `upload_asset`, sonra dönen `asset_id`. |

Yazma (`save_music` kapalı, listede yok):

`save_project(...)` · `save_episode(...)` · `save_scene(...)` ·
`save_character(...)` · `save_entity(...)` · `save_sheet(...)` ·
`save_prompt(...)` · `save_track(...)` · `save_comment(...)`

Hepsi **upsert**: `id` yoksa create, `id` varsa patch, `id` var ama satır yoksa
hata. Sessiz yaratma yok. `save_frame` diye bir op **yoktur** — frame'ler
`save_scene.video.frames[]` içinden yazılır.

`save_prompt(project_id, type, name, text)` prompt bank'e **yalnız projeye ait**
satır açar. Platform satırı bu op'la **değiştirilemez** (`VALIDATION`): onlar her
kiracının okuduğu satırlardır, migration'la değişir. Bir platform kalıbıyla
anlaşamıyorsan **aynı adla** kendi satırını aç — merdiven ada göre yürüdüğü için
o satır senin şovunda platformunkinin önüne geçer (§13).

Okuma — para harcamaz:

| Op | Ne için |
|---|---|
| `list_projects(workspace_id?, include_archived?, limit?)` | Workspace'teki showlar, son düzenlenen başta. **Elinde proje id'si yoksa ilk çağrı budur** — geri kalan her op bir id ister. Arşivlenmiş showlar `include_archived: true` demedikçe listede yok. |
| `get_project(id, include?, limit?)` | Proje + dalları (`episodes`, `characters`, `entities`, `sheets`, `musics`, `voices`, `comments`, `models`) |
| `get_episode(id, include?, limit?)` | Episode + dalları (`scenes`, `tracks`, `frames`, `music`, `text`, `takes`, `comments`) — **devam noktasının op'u budur** |
| `get_takes(slot, limit?)` | Bir slot'un bütün take'leri, yenisi başta, seçili işaretli |
| `check_freshness(slug \| slug[], depth?)` | Slot(lar) hâlâ taze mi (§10, §11) |
| `list_catalog(kind, scope?, q?, limit?)` | `character` · `entity` · `voice` · `model` · `music` · `prompt` katalogları |

Yapı ve dosya (`import` kapalı):

| Op | Not |
|---|---|
| `upload_asset(filename, media, kind, content_base64?, …)` | Tek istek, 25 MB tavan (`FILE_TOO_LARGE`). **`kind` ZORUNLU** — türü hem medyayı hem bağlanacağı satırı seçer, bağlama upload'la aynı transaction'da olur. Sahipsiz upload yok. Sheet türlerinde (`character_sheet` + beş entity türü) `name` yerine **`sheet_id`** verilebilir ve elinde satır varken verilmelidir: `name` `slugifyName`'e katlanır, yani serbest seçilmiş bir `slug`'ı bulamayıp İKİNCİ satır açar, üstelik hep sahibin ANA sheet'ine iner. Aynı baytlar bir kapsamda tek satırdır: ikinci bir `media` ile göndermek `VALIDATION` / `$.media`. |
| `delete(target:{kind,id}, confirm?, confirm_name?)` | **Silmenin iki anlamı var, cevaptaki `emptied` hangisinin koştuğunu söyler.** (1) **Slot boşaltma** — sahnenin voice/video/avatar track'i: satır YAŞAR, işaretçi ve trim boşalır, `status` 'draft'a döner, take'ler durur ve `put` ile herhangi biri geri seçilir (`emptied:true`, manifest `would_empty`). Prompt/metin, gain/muted ve karakter kalır; frame'lere dokunulmaz. (2) **Yapı silme** — geri kalan her tür, metin track'leri (caption) dahil: satır kalır, kayıt her listeden ve okumadan düşer (`emptied:false`, manifest `would_delete`). Her iki yolda da **hiçbir asset silinmez**, R2'ye dokunulmaz. `confirm` yoksa `CONFIRM_REQUIRED` + manifest. `kind:'project'` ayrıca projenin **tam adını** `confirm_name` ile ister (uymazsa `VALIDATION` / `$.confirm_name`). Arşiv ayrı bir şeydir ve yalnız projeye aittir (`save_project.status='archived'`); bu op arşivlemez. Altında kuyrukta/üretimde iş varsa `JOB_IN_FLIGHT`. |
| `move_scene(scene_id, idx)` | Kalanları motor tek kilit altında yeniden numaralar. |
| `render_episode(episode_id, force_stale?)` | Bölümün son halini render eder; cevap render SATIRIdır (`render_id`, `status`, `input_hash`) — dosya değil. Bölüm başına tek uçuş: uçarken ikinci çağrı `RENDER_IN_FLIGHT`. Kurgu son biten render'la aynıysa o satır aynen döner (`reused:true`); yeniden render için `force_stale:true`. Bayrağı `force_stale`, `ack_stale` değil. Dosyanın adresi `get_episode`'un `render` alanındadır. |

Erişim — para harcamaz, ama **workspace dışına çıkar**:

| Op | Ne yapar | Not |
|---|---|---|
| `create_invite(email, role, project_id \| project_ids \| scope:'workspace', …)` | Dışarıdan birini şova çağırır | **Bir e-posta GÖNDERİR** — geri alınamaz, önce kullanıcıya adresi teyit ettir. Destructive. Kapsam üç şıktır: tek şov (`project_id`), sayılan şovlar (`project_ids`, en fazla 20) ve **`scope:'workspace'`** — bugünkü TÜM şovlar artı SONRA açılacaklar, otomatik. Üçüncüsü yalnız workspace admin'i gönderebilir (`FORBIDDEN reason:'scope'`) ve hiç şov adı almaz. |
| `revoke_invite(...)` | Gönderilmiş daveti iptal eder | Destructive |
| `resend_invite(...)` | Aynı daveti tekrar yollar | Destructive değil, idempotent |
| `grant_role(...)` | Var olan bir üyeye rol verir | Destructive değil, idempotent |
| `change_role(...)` | Üyenin rolünü değiştirir | Destructive |
| `set_invite_permission(...)` | Davetin izin kapsamını değiştirir | Destructive |
| `revoke_grant(user_id, project_id?, workspace_id?)` | Verilmiş rolü geri alır | Destructive. Cevaptaki `scope` hangisinin koştuğunu söyler: `project` (şovun grant satırı silindi) · `workspace_show` (workspace kapsamlı hak sahibinden TEK şov kapandı, kalanlar ve gelecek duruyor) · `workspace_seat` (workspace kapsamlı hakkın kendisi kalktı) · `admin` (workspace admin'i member'a indi). |

Yedisi de **erişim kararıdır, üretim adımı değil.** Kullanıcı açıkça istemeden
çağırma; beş tanesi destructive olduğu için MCP istemcisi onay ekranı çizer.

Katalog bakımı (1) — para harcamaz:

| Op | Not |
|---|---|
| `sync_voices(workspace_id?)` | ElevenLabs hesabının kendi seslerini platform katalogına çeker; yukarıda kalmayanı **arşivler, silmez**. **Yalnız staff** — satırlar platform kapsamında, üye rolü bunu ifade edemiyor; başkası çağırırsa `FORBIDDEN`. Anahtar tanımsızsa `PROVIDER_UNCONFIGURED` (tekrar denemek çözmez). Sesi bir karaktere `save_character.voice_id` bağlar; bu op bağlamaz. |

**Kapalı üç op:** `generate_music` · `save_music` · `import`. Tool
listesinde yoklar, çağrı `OP_DISABLED` alır. Pratik sonuçları:

- **Müzik yok.** Söz verme, "arka plan müziği ekleyelim" deme.
- **Vitrinden kopyalama yok.** Onboarding'de "preset'ten seç" diye bir dal
  **yoktur** — ya üret ya atla.

**Alan uydurma yasağı:** burada ve tool şemasında olmayan bir alanı gönderme.
`prompt`, `mode`, `url`, `idem_key`, `canvas_ratio`, `wait_ms` — bunların
hiçbiri yoktur.

---

## 4 · Slug grameri

**Adres tek string ve proje-görelidir.** Proje bağlamı parametreden gelir; yoksa
`PROJECT_REQUIRED`. **İlk çağrıda projeyi sabitle, her çağrıda `project_id` geç.**
Slug bir sahneyi işaret ediyorsa `episode_id` de gerekir.

**Projenin bir üstünde workspace var, kuralı aynı.** Projesi belli her çağrıda
workspace'i **proje söyler** — `save_scene`, `generate_*`, `save_comment` gibi
op'larda hiçbir şey yapmana gerek yok. Ama **projesiz op'lar** —
`list_projects`, `save_project` (create), `list_catalog`, `sync_voices`,
projesiz `upload_asset` — workspace'i kendileri çözmek zorunda:

- Kullanıcının tek üyeliği varsa sessizce çözülür, bugün olan bu.
- **Birden fazla üyelik varsa `WORKSPACE_REQUIRED {workspaces:[uuid,…]}`
  gelir.** İlk turda workspace'i sabitle ve bu op'lara `workspace_id` geç.
- **`WORKSPACE_REQUIRED` aldığında: TAHMİN ETME, SOR.** Cevap sana çıplak uuid
  listesi veriyor ve **o uuid'lerin adını okuyabileceğin bir op YOK** —
  `list_catalog` workspace bilmiyor, `workspaces` view'ı tarayıcı yolunda.
  Yani listedeki ilk uuid'i seçmek ya da adını uydurmak yanlış kiracıya
  yazmak demektir. Uuid'leri kullanıcıya olduğu gibi göster, hangisi olduğunu
  sor, aldığın cevabı o turun geri kalanında sabit tut.
- **Skill'e sabit bir workspace id YAZILMAZ** ve burada da yoktur: bir uuid bu
  dosyaya yazıldığı an ikinci kiracıda yalan olur.

| Kalıp | Nereye bakar |
|---|---|
| `char/<slug>/sheet` | Karakterin ana sheet'i |
| `char/<slug>/sheet/<ad>` | Varyasyon |
| `prop\|loc\|style\|world/<slug>/sheet[/<ad>]` | Entity ana sheet / varyasyon |
| `scene/<idx>/voice` · `/video` · `/avatar` | Sahne track'i |
| `scene/<idx>/video/frame[/<n>\|/last]` | Frame — **frame yalnız video track'inde vardır** |
| `char/<slug>/avatar` · `char/<slug>/voice-example` · `project/icon` | `put`'un ek hedefleri |

Kurallar:

- **Göreli adres var:** `scene/prev/...`, `scene/next/...`, `.../frame/last`.
  Komşuya ve uç kareye idx saymadan bakılır.
- `scene/12`'deki **12 idx'tir, id değil**; dispatch anında çözülür.
- `scene/N/text` slot **değildir** — text track asset taşımaz, `put` edilemez.
  Onu `save_track(kind:"text")` ile yazarsın.
- **Ham URL yasak.** `refs[]` yalnız slug ya da `asset_id` alır; URL hash'lenemez
  diye reddedilir. `refs[]`'ten gelenler donarken `ref:0`, `ref:1` sentetik
  adına yazılır — sen bu adı hiçbir çağrıda kullanamazsın.
- Entity slug'ı proje içinde tektir; gramer `^[a-z0-9][a-z0-9_-]*$`.
- Slug'lar çağrı ateşlenmeden çözülür — bedava varlık kontrolü. Eksikse
  `SLUG_UNRESOLVED {missing[]}` gelir ve **iki seçeneğin vardır: bekle ya da
  üret. Üçüncüsü yok.**

**Bu gramer sonradan değişmez.** Yaşayan promptların içinde yazılı; değiştirmek
o güne kadarki her prompt'u elden geçirmek demek.

---

## 5 · put / generate ayrımı

**Tek cümle: `generate` okur, `put` yazar. Aynı kolona dokunmazlar.**

- `generate_*` yaşayan prompt'u **hedef satırdan** okur. **`prompt` parametresi
  yoktur.** Prompt kolonu olmayan iki slotun — `project/icon`, `char/x/avatar` —
  prompt'unu **op kendisi kurar**: bank prefix'i + şovun adı + amacı +
  `rules`'un **`STYLE` ile başlayan paragrafı** (varsa; yoksa görünüş kısmı boş
  geçer). Yalnız o paragraf: `rules`'un geri kalanı ajana konuşur ve orada
  yasaklar da yazılıdır (§13) — yasak cümlesi prompt'a girerse model yasaklanan
  şeyi çizer. Kullanıcı ikon prompt'u yazmaz; `params.description` yalnızca
  isteğe bağlı ek bir cümledir. İki slot da karedir; oranı op basar.
  (`char/x/voice-example` üretilmez — klon örneği yüklenir.)
- **`mode` parametresi yoktur.** t2i / i2v / edit ayrımı `refs`'ten türer: boşsa
  düz üretim, görsel ref varsa image-to, önceki take ref'iyse düzeltme/upscale.
- **Upscale ayrı op değildir:** `refs:[önceki take]` + `model:"<upscaler model_ref>"`.
  (Eski dokümanlarda bu "params.model" diye yazılıdır; registry'de model'in kendi
  alanı vardır ve slot'u ezen alan odur.)
- **Model proje slotundan gelir**, çağrıda `model` verirsen o kazanır. Slotlar
  `save_project` sırasında katalog default'larından dolar; buna rağmen
  `MODEL_SLOT_EMPTY {slot}` görürsen o slot gerçekten boştur (bugün yalnız avatar
  slotunda mümkün — hat kapalı).
- **`generate_*` hedef satırı YARATMAZ.** Önce `save_sheet` / `save_scene` /
  `save_track` ile satır açılır, üretim onu doldurur. En sık düşülen çukur bu.
- **Yeniden üretim** seçili asset'in `model` + `params` + malzemelerini aynen
  kullanır. Değiştireceğin neyse çağrıda geç.
- Üretim bitince pointer'ı **sistem (fetcher)** koyar — aynı `put` yolundan.
- **`approved` slot:** otomatik üretim pointer'ı ezmez. Sonuç şeride düşer,
  seçili take değişmez. **Ama bunu taze bir üretim zarfında `APPROVED_SLOT`
  uyarısı olarak GÖRMEZSİN:** `generate_*` cevabı düpedüz
  `{asset_id, job_id, status:"queued"}`'dur, içinde uyarı yoktur; red callback
  anında, `put_asset_v1` içinde oluyor ve sana hiç dönmüyor. Okuyabildiğin tek
  yer `reused` cevabındaki **`put:"approved_slot"`** alanı (§8). Onaylı bir
  slota üretim yolladıysan sonucun nereye gittiğini `get_takes(slot)` ile
  doğrula ve **kullanıcıya söyle**; istiyorsa elle `put`.
- Elle `put` her zaman serbest — review'un "eski take'e dön" adımı budur.
  Monoton guard yalnız sistem yolundadır.
- **`assets` ve `jobs` slot değildir.**

---

## 6 · Otomatik küme ve erim

Her görsel üretim malzemesini üç kapıdan alır:

1. **OTOMATİK — sistem basar, sen hesaplamazsın:** erimi `project` ya da
   `episode` olan entity'lerin ana sheet'leri, kalıplar (prefix/suffix), ratio ve
   grid mapping'i, projenin `aim` + `rules` bağlamı.
2. **SLUG — yaşayan prompt'ta senin yazdıkların:** `char/ayse/sheet`,
   `loc/istanbul/sheet/gece`.
3. **REFS — çağrıda verdiğin opsiyoneller:** upload, önceki take, komşu sahne.

Üçü de üretim anında aynı yerde donar (asset olan `asset_ingredients`'a, metin
olan `gen_prompt`'a, sayı olan `params`'a) ve **üçü de hash'e girer**.

Bilmen gerekenler:

- **Otomatik küme yalnız GÖRSEL üretimlere girer.** Ses hattı style ve world
  görmez.
- Otomatik kümede aynı anda **tek style ve tek world** olur; ikincisini
  kaydetmek `save_entity`'de hata döner.
- Ana sheet hazır değilse üretim onu **bekler**, sessizce atlamaz.
- **Proje erimi bedava değil:** her üretimin referans bütçesine ve fiyatına
  girer. Toplam `models.max_refs`'i aşarsa
  `TOO_MANY_REFS {limit, resolved[], automatic[]}`. **`automatic[]`'i oku:**
  `resolved[]` her şeyi sayar, `automatic[]` yalnız SENİN yazmadıklarını — yani
  kendi `refs[]` listeni saymadan farkı görürsün. Çözüm: bir erimi daralt ya da
  bir ref düş.
- Erim daraltmak o güne kadarki görselleri bayatlatır. **Kendiliğinden hiçbir
  şey yeniden üretme** — kaç görselin bayatlayacağını söyle, kararı kullanıcı
  versin.

---

## 7 · Grid mapping — okunur, hesaplanmaz

**N mührü:**

> `N` senin kararındır ve `frames.shots`'a yazılır. Tuval oranını, grid
> prefix'ini ve hücre geometrisini **yalnız sistem** hesaplar. Ajan
> `canvas_ratio` gönderemez — böyle bir alan yoktur.

Senin bildiklerin:

- Bir grid **tek görseldir**: satır = 1 shot = (first | last) çifti.
  `frames.shots` 1..4 arası tam sayı.
- Başlangıç önerisi: **~4 saniye anlatım = 1 shot** → `N = clamp(ceil(vo_sn/4), 1, 4)`.
  Bu bir öneridir; satıra yazılan sayı senin kararındır.
- Hücrenin oranı **track'in oranıdır**; `projects.ratio` yalnız default'tur.
- Model tavanı: bir klip modelin `max_duration_s`'ini aşamaz. Anlatım 4 shot'la
  bile kapanmıyorsa `SCENE_TOO_LONG` uyarısı gelir — **hata değil, uyarı**, ama
  ciddiye al: sahneyi böl.
- Hücreler çözünürlük tabanının (kısa kenar 512 px) altına düşecekse sistem N'i
  kendisi indirir ve `GRID_RESOLUTION_CAP` uyarısı taşır. Uyarıyı kullanıcıya
  söyle.

Senin bilmediğin ve bilmen gerekmeyen: aday tuval merdiveni, log-uzaklık
sıralaması, tie-break, artık boşluk yüzdesi, prefix metni. Hepsi
`packages/studio-op-contract/grid-mapping.mjs`'te ve tek ev orası.

**`models.spec` vitrin metnidir.** Grid'in mi frame'in mi hangi alanla
provider'a gittiği adaptör kodunun bilgisidir; `spec`'e bakıp çağrı şeklini
değiştiremezsin, zaten `mode` parametresi de yok.

---

## 8 · Durum makineleri

**İçerik (`progress_status`):** `draft → waiting → queued → generating → review → approved`

- `waiting` = ingredient bekliyor. Slug'lar çözülünce sistem kendiliğinden
  `queued`'a geçirir.
- **Tek istisna:** `failed`'den dönen satır otomatik `queued`'a **girmez.**
  Yeniden deneme açık karardır — arızalı provider sonsuz para döngüsü açamasın.
- **İçerik hiç `failed` olmaz.** Başarısızlık üretim kaydının işidir.
- Yeniden üretim satırı `waiting`'e döndürür.

**Üretim kaydı (`asset_status`):** `queued → generating → ready` ya da `failed`

- `failed` → job ve asset düşer, içerik `draft`'a döner. `waiting` değil:
  `waiting` uyanınca yeniden kuyruğa girerdi; otomatik retry yasak.
- `reused` da bir statüdür ve **hata değildir**: aynı hash'li asset zaten vardı,
  para yanmadı.
- **`reused` "ve slotta o duruyor" DEMEK DEĞİLDİR — cevaptaki `put` söyler.**
  Reuse hiçbir zaman bir SEÇİMİ ezmez; slot boşsa işaretçiyi yazar (`put:"put"`),
  doluysa dokunmaz (`put:"held"`). Bir slotu iki op doldurabildiği için bu artık
  görünür bir durum: `scene/N/voice`'ta `generate_voiceover` ile `change_voice`
  sırayla çalıştıysa, üçüncü çağrı ESKİ take'i `reused` diye döndürürken slotta
  öteki take durur. `held` gördüysen istediğin take slotta DEĞİL: `put` op'uyla
  seç. Diğer iki değer `put_asset_v1`'in kendi sözcükleri: `approved_slot`
  (onaylı slot otomatik put kabul etmez), `no_owner` (satırın böyle bir
  işaretçisi yok).

**Yorum:** `open → resolved`. Onay ayrı mekanizma değil — sahnede açık yorum
varken `approved` olunmaz (`OPEN_COMMENTS`). Onaylı bir sahneye yeni açık yorum
düşerse sahne kendiliğinden `review`'a iner; bunu kullanıcıya söyle.

**`stale` bu listelerin hiçbirinde yok** — statü değil, rozet (§10).

---

## 9 · Ortak zarf ve hata dili

Başarı:

```json
{ "ok": true, "op": "save_comment", "trace_id": "…", "data": { … },
  "warnings": [ { "code": "APPROVED_SLOT", "note": "…" } ],
  "replayed": true, "reused": true }
```

(Örnekteki uyarı `save_comment`'ten — `APPROVED_SLOT` taze bir üretim
zarfında GELMEZ, §8'e bak; örneği kopyalarken üretim op'una taşıma.)

`warnings`, `replayed`, `reused` **yalnız doğruyken** yazılır. `replayed: true`
hata değildir — aynı iş ikinci kez paraya gitmedi demektir.

**Medya linkleri 5 dakikada ölür.** Cevaptaki her asset MCP worker'ı tarafından
imzalı bir `resource_link` olarak basılıyor ve ömrü
`MEDIA_LINK_TTL_SECONDS = 300`. Pratik sonucu: bir videoyu 4 dakika bekleyip
döndüğünde eldeki link zaten sınırdadır. **Eski bir cevaptaki linki kullanıcıya
verme ya da ref diye taşıma** — `get_takes(slot)` ile o anda yeniden oku,
taze imzayı o çağrı basar.

Hata:

```json
{ "ok": false, "op": "generate_video", "trace_id": "…",
  "error": { "code": "TRACK_BUSY", "message": "…", "retryable": true,
             "detail": { "queue": 3, "retry_in": 45 } } }
```

Hata sözlüğü — tamamı, çünkü kodu okuyup davranışını buna göre seçeceksin:

| Kod | Yeniden denenir mi | Sen ne yaparsın |
|---|---|---|
| `VALIDATION` | hayır | Şemayı düzelt. Aynı gövdeyle tekrar deneme. |
| `OP_NOT_FOUND` | hayır | Böyle bir op yok. Uydurma. |
| `OP_DISABLED` | hayır | Gün-1'de kapalı (§3). Kullanıcıya söyle, alternatif sun. |
| `NOT_AVAILABLE` | hayır | Op var, motoru yok. Bugün bunu atan hiçbir op yok; motoru gelmemiş bir sonraki op için duruyor. |
| `RENDER_IN_FLIGHT` | hayır | O bölümün bir render'ı zaten uçuyor (`detail.render_id`). Bekle, sonra tekrar sor — döngüye girme. |
| `NOT_FOUND` | hayır | Satır yok. Önce `save_*` ile aç. |
| `SLUG_UNRESOLVED` | hayır | `missing[]`'i oku: bekle ya da üret. |
| `QUEUE_FULL` | **`detail.limit`'e bağlı** | `limit > 0` ise geçici: `retry_in` + jitter bekle. **`limit == 0` ise KALICI — retry etme, DUR, insana söyle** (aşağı bak). |
| `TRACK_BUSY` | evet | O **track**'te zaten bir üretim uçuyor (sahne değil, track). Bekle; aynı sahnenin diğer kolu paralel koşabilir. |
| `STALE_HASH` | hayır | §10. Ya tazele ya `ack_stale`. |
| `TOO_MANY_REFS` | hayır | Bir erimi daralt ya da bir ref düş. |
| `TOO_MANY_ROWS` | hayır | `include[]`'ı daralt ya da `limit` düşür. Sayfalama yok. |
| `CONFIRM_REQUIRED` | hayır | Manifest'i **kullanıcıya göster**, onay al, `confirm:true` ile dön. |
| `JOB_IN_FLIGHT` | hayır | Silinecek satırın altında kuyrukta/üretimde bir iş var. Bitmesini bekle ya da iptal et, sonra sil. `TRACK_BUSY` değil: o admission'ın tek-track kapısı, bu silmenin. |
| `OPEN_COMMENTS` | hayır | Yorumları çöz, sonra onayla. |
| `FORBIDDEN` | hayır | RLS/rol. Zorlama, söyle. |
| `WORKSPACE_REQUIRED` | hayır | Birden fazla üyeliğin var. `detail.workspaces` çıplak uuid listesidir ve **adlarını okuyacağın op yoktur** — uuid'leri kullanıcıya göster, sor, seçileni `workspace_id` ile geç. Tahmin etme (§4). |
| `NARRATOR_REQUIRED` | hayır | Şovda narrator sesi yok. §onboarding — `save_project(narrator_voice_id)` ile bir `voices` satırı seç. |
| `TYPE_NOT_ENABLED` | hayır | Gün-1'de `type ∈ {faceless, cinematic}`. |
| `MODEL_SLOT_EMPTY` | hayır | O slot gerçekten boş. |
| `FILE_TOO_LARGE` | hayır | 25 MB tavan. Küçült. |
| `PROVIDER_ERROR` | evet | Job `failed`. **Otomatik retry yok** — dur, sor. |
| `PROVIDER_UNCONFIGURED` | hayır | Sağlayıcının anahtarı bu kurulumda tanımlı değil. Tekrar denemek hiçbir şeyi değiştirmez; kurulumu yapan kişiye `detail.variable`'ı söyle. |

Uyarı kodları (zarfla birlikte gelir, statüyü değiştirmez):
`APPROVED_SLOT` · `SCENE_TOO_LONG` · `GRID_RESOLUTION_CAP`.

**`APPROVED_SLOT` uyarısını taze bir üretim zarfında BEKLEME.** Kodda tek bir
yerden çıkıyor (`handlers/save.ts:1169`) ve orada **başka bir şey** demek: açık
bir yorum, onaylı bir sahneyi `review`'a indirdi. Onaylı slota gelen otomatik
put reddi ise callback anında oluyor ve zarfa hiç girmiyor — onu `reused`
cevabındaki `put:"approved_slot"` alanından okursun (§5, §8).

`IDEM_CONFLICT` diye bir kod **yoktur** — `idem_key` türetilir (§12), çakışması
mümkün değildir.

---

## 10 · stale / `ack_stale` protokolü

- **`stale` bir kolon değil.** Hash karşılaştırmasından her okuyuşta hesaplanan
  rozettir, her state'in üstüne biner. Tek doğruluk kaynağı hash.
- **Bilmen gereken en önemli tek cümle:** stale, modeli **asset'in kendi kayıtlı
  modeliyle** karşılaştırır, projenin bugünkü slotuyla değil. Yani proje video
  modelini değiştirmek 80 onaylı sahneyi bayat yakmaz. Kullanıcı "model
  değiştirdim, her şey bozuldu mu?" derse cevap hazır: **hayır.**
- Kontrol **dispatch anında zorunludur.** `check_freshness` ile önceden bakmış
  olman guard'ı atlatmaz — o danışma ucudur.
- Bayat girdiyle çağrı → `STALE_HASH {changed[]}`. **İki meşru cevap var,
  üçüncüsü yok:**
  1. Bayat girdiyi tazele, sonra dön.
  2. `ack_stale: true` ile bilerek devam et.
- **`ack_stale` ne zaman meşru:** kullanıcı açıkça "eskisiyle devam" dedi ·
  uzun bir batch baştan onaylandı. **Ne zaman değil:** kullanıcıya sormadan,
  sırf akış kesilmesin diye.
- **Sessiz geçiş yok.** `ack_stale` kullandıysan tek cümleyle söyle:
  *"style sheet'i değişmiş; 12 kareyi bilerek eskisiyle bırakıyorum."*
- Yeni bir kolon eklenmesi geçmiş hash'leri toptan bozmaz — kanonik hash
  default değerli alanları atlar.
- `render_episode`'un bayrağı farklıdır: **`force_stale`**, `ack_stale` değil.

---

## 11 · Yoklama adabı

**Üretim çağrısı hemen döner, "bitti" demek değildir.** Elinde `job_id` ve
`status: "queued"` vardır, dosya yoktur.

Ayrı bir `get_status` op'u **yoktur**. Yoklamanın **iki ucu** var ve ikisi ayrı
soruya cevap veriyor — birini ötekinin yerine kullanamazsın:

| Uç | Ne söyler | Ne söylemez |
|---|---|---|
| `check_freshness(slug[])` | Slot taze mi: `fresh` · `stale` · `unbuilt` | **`failed` DİYEMEZ.** Düşmüş bir iş de, hiç başlamamış bir iş de `unbuilt` görünür |
| `get_takes(slot)` ya da `get_episode(include:["takes"])` | Asset satırının kendi `status`'u: `queued` · `generating` · `ready` · `failed` | — |

**Bu yüzden ilerlemeyi `check_freshness` ile, biteni/düşeni take'lerle
yoklarsın.** Bir slot iki turdur `unbuilt` duruyorsa bu "hâlâ çalışıyor" demek
değildir; `get_takes` ile asset satırının `status`'una bak — iş `failed` olmuş
ve para yanmış olabilir, `check_freshness` bunu sana asla söylemez.

1. **Taban 1 saniye. Önce bekle, sonra bak.** İlk çağrının hemen ardından
   yoklamanın hiçbir faydası yok.
2. **Üstel git:** 10 sn → 20 → 40 → 60 sn tavan. Süre referansları: video 8 sn
   ≈ 4 dk · görsel ≈ 1 dk · VO ≈ 30 sn.
3. **Sunucu `retry_in` diyorsa onu kullan**, kendi tahminini değil.
   `QUEUE_FULL` ve `TRACK_BUSY` `{queue, retry_in}` ile döner. Üstüne 0-10 sn
   rastgele ekle — iki ajan aynı anda uyanmasın.
4. **Toplu sor, tek tek değil.** `check_freshness` imzası dizi kabul eder:
   40 sahnelik koşuda 40 çağrı değil, **1 çağrı × 40 slug**. Sebebi hız değil
   bağlam — 40 ayrı cevap hafızanı yoklama gürültüsüyle doldurur ve koşunun
   ortasında biter.
5. **`failed` gördüğünde yoklamayı KES.** Otomatik retry yok. Dur, sebebi düz
   Türkçe anlat, insana sor. **`failed`'i görmenin tek yolu take okumaktır** —
   `check_freshness` sonsuza kadar `unbuilt` der. Bir slot beklenen süreyi
   (video ≈4 dk) belirgin biçimde aştıysa tavanda yoklamaya devam etme,
   `get_takes(slot)` ile statüye bak.
6. Uzun bir işi yoklarken kullanıcıyı boş boş bekletme; "video sırada, ~4 dakika"
   deyip başka bir işe geç, sonra dön.

`wait_ms` diye bir parametre **yoktur** — long-poll önerisi kesildi.

---

## 12 · Para guard'ları

**⛔ Para için protokol freni YOK — fren sensin.** `generate_*` op'ları MCP'de
`destructiveHint: false` taşıyor, yani **istemci onay ekranı çizmiyor**;
registry'nin `spendsCredits` bayrağı MCP standardında olmayan özel bir alan ve
claude.ai onu okumuyor. Bir `generate_video` çağrısı, kullanıcının önüne hiçbir
kapı çıkmadan, doğrudan paraya gider. Üretim op'ları arasında akışı gerçekten
durduran tek op `delete`. Bu yüzden §12'nin "önce fiyatı söyle, onay al"
disiplini bir görgü kuralı değil, **hattaki tek fren.**

Sistemin reddettikleri — delemezsin, sadece bilirsin:

| Guard | Ne olur | Sen ne yaparsın |
|---|---|---|
| Kuyruk limiti | Tavan **workspace'in `job_limit`'i**, NULL ise 7 (`coalesce(job_limit, 7)`) → `QUEUE_FULL {limit, queue, retry_in}` | `retry_in` + jitter bekle — **ama önce `detail.limit`'e bak** ↓ |
| Track başına tek aktif üretim | `TRACK_BUSY {queue, retry_in}` — sahne başına değil, **track** başına | Bekle; diğer kol paralel koşabilir |
| Çift alım | Aynı `idem_key` ikinci kez paraya gidemez | **`idem_key` senin işin değil.** `asset:input_hash:attempt` üçlüsünün sha256'sından türer; şemada böyle bir alan yok. `replayed: true` hata değil |
| Referans bütçesi | Otomatik küme + slug'lar + `refs[]` toplamı `models.max_refs`'i aşarsa `TOO_MANY_REFS` | Bir erim daralt ya da bir ref düş |
| Retry tavanı | Otomatik retry'ın süre ve deneme tavanı var | Arızalı provider sonsuz döngü açamaz |
| Açık yorum | `OPEN_COMMENTS` → `approved` olmaz | Yorumları çöz |

**⛔ `QUEUE_FULL` her zaman geçici DEĞİL — `detail.limit`'i oku.** Tavan sıfır
olabilir: `create_workspace` ve `staff_create_workspace` ile doğan bir workspace
**`job_limit = 0`** ile doğuyor (`chk_workspaces_job_limit` 0'ı meşru değer
sayıyor). O workspace'te `aktif (0) >= tavan (0)` her zaman doğrudur, yani
**her üretim, sonsuza kadar, `QUEUE_FULL {limit: 0, retry_in: 15}` alır.**
`retryable: true` bayrağına aldanıp bekle-dene döngüsüne girme — o döngü hiç
bitmez. Kural tek cümle: **`detail.limit == 0` ise DUR ve insana söyle** —
"bu workspace'in üretim tavanı 0, açtırman gerekiyor". Beklemek çözmez.

Senin kendi görgün:

- **`dry_run: true`** — her yeni hat, yeni tür, yeni model ilk denemesinde.
  Fiyatı söyle, onay al. `dry_run` kilit almaz, satır yazmaz, para harcamaz.
- **Toplu üretimden önce toplam tahmini fiyatı söyle ve onay al. Onaysız seri
  üretim yok.**
- Fiyat `estimated_cost {credits, usd, unit, unit_cost, approx}` ile gelir.
  **`approx: true` ise rakamı "≈" ile söyle** — o fiyat ölçülmedi, kitaptan
  çevrildi.

Gerçek katalog fiyatları (kullanıcıya rakam söylerken bunları kullan; 1 kie
kredisi = $0.005):

| Ne | Model | Hesap | Fiyat |
|---|---|---|---|
| Görsel (1K) | `kie:gpt-image-2` | 6 kredi | **$0.03** (ölçülmüş) |
| Video 8 sn | `kie:grok-imagine-1.5` 480p | 2.4 kredi/sn × 8 = 19.2 kredi | **≈$0.096** |
| VO 500 karakter | `elevenlabs:eleven_multilingual_v2` | $0.00022/kar. × 500 | **≈$0.11** |
| Ses değiştirme 8 sn | `elevenlabs:eleven_multilingual_sts_v2` | $0.003667/sn × 8 | **≈$0.029** |

Yalnız `gpt-image-2` `≈`'siz: prod'da `cost_basis = measured`. Diğer üçü
`estimated`, o yüzden rakamı hep `≈` ile söylersin.

Örnek: 10 sahnelik faceless episode, sahne başına 1 VO (~120 karakter, ≈$0.026) +
1 grid görseli ($0.03) + 1 video (≈$0.096) ≈ **$0.152**; 10 sahne ≈ **$1.52**,
%30 retry payıyla ≈ **$1.98**.

### 12.1 · Bridge — ikinci sağlayıcı, beş satır (T-75, T-82)

Katalogda **beş satır** daha var, hepsi bridge sağlayıcısında. Varsayılan
DEĞİLLER — hiçbir varsayılan değişmedi, `model:` ile açıkça seçersin. KIE
satırları yerinde duruyor.

| Satır (`model:` ile yazdığın) | Ekranda | Fiyat | Referans | Dayanak |
|---|---|---|---|---|
| `bridge:grok-image` | Grok Image 2 Quality | **$0.06**/görsel | almaz | ölçülmüş |
| `bridge:grok-image-fast` | Grok Image 2 Fast | **$0.02**/görsel | almaz | ölçülmüş |
| `bridge:grok-video` | Grok Imagine 1.5 480p | **$0.08**/sn | 3'e kadar | ölçülmüş (4 sn = $0.32) |
| `bridge:grok-video-720p` | Grok Imagine 1.5 720p | ≈**$0.15**/sn | 3'e kadar | ÖLÇÜLMEDİ — ara değer |
| `bridge:grok-video-1080p` | Grok Imagine 1.5 1080p | ≈**$0.25**/sn | **yalnız 1** | $3.75 referansından türetildi |

⛔ **KADEME ARTIK PARAMETRE DEĞİL, SATIR** (Serhat, 27 Ağu). Eskiden tek bir video
satırı vardı ve çözünürlüğü `params.resolution` ile seçiyordun. Artık seçtiğin
satır çözünürlüğü söylüyor:

- `params.resolution` / `params.quality` **boş bırak** — satırınki wire'a gider.
- Satırla **aynısını** yazarsan kabul edilir.
- **AYKIRI** yazarsan `VALIDATION` yersin ve cümle o değeri taşıyan satırı
  ref'iyle söyler. Sessizce ezilmez: `bridge:grok-video` satırına
  `resolution: "1080p"` yazmak 480p fiyatına 1080p iş almak DEĞİL, redde girmek.
- Görselde `resolution` (`1k`/`2k`) hâlâ parametre — fiyatı değiştirmiyor.

⚠ **`bridge:grok-image` ve `bridge:grok-video` ref'leri DEĞİŞMEDİ**, adları
değişti: birincisi görselin **quality** kademesi, ikincisi videonun **480p**'si.
Eski bir yüzeyde "Grok Image 2" ya da "Grok Imagine 1.5" (kademesiz) görürsen o
yüzey T-82 öncesi. `assets.model` ve `jobs.model` hep ref'i yazar.

Bridge dolarla faturalıyor (`usd_per_credit = 1.0`), yani `unit_cost` doğrudan
dolar; kredi çevrimi yok.

Video örnekleri: 6 sn 480p = **$0.48** · 10 sn 720p ≈ **$1.50** ·
15 sn 1080p ≈ **$3.75** — bugün alabileceğin en pahalı klip bu.

**`dry_run` artık her satırda DOĞRU rakamı veriyor** ve bu T-82'nin ikinci
turunda değişti. Eskiden tek video satırı üç kademeyi temsil ediyordu, tahmin
motoru da yalnız `models.unit_cost`'u (480p ücreti) okuyordu: 15 sn 1080p için
`dry_run` **$1.20** derken gerçek ≈$3.75'ti ve defter "bu rakama güvenme"
diyordu. Bölmeden sonra her satırın `unit_cost`'u kendi kademesinin ücreti.

⛔ **`approx: true` (yani "≈") ARTIK "ÖLÇÜLMEDİ" DEMEK, "yanlış" değil.** 480p ve
iki görsel satırı `measured` — rakamı tildesiz söylersin. 720p ve 1080p
`estimated`, çünkü ikisi de bu anahtarla hiç faturalanmadı; rakam doğru
hesaplanıyor ama tartılmadı, o yüzden ≈ ile söylersin.

Parametre kuralları — bunlar tahmin değil, canlı sözleşmeden okundu
(`/openapi.json`, 25 Ağu 2026). Admission bunları senden önce reddeder, ama
bilerek yaz ki reddi yeme:

- **Görsel her zaman 1 tane.** `count` sabit 1; 2 istemek reddedilir.
- **⛔ Görsel HİÇ referans almaz.** İki görsel satırı da metinden-görsele; tek bir
  ref bile 400. Referanslı iş videonun.
- **Süre bir ENUM: 6, 10 ya da 15 saniye.** Aralık değil. 7 sn diye bir şey yok —
  `duration_s: 7` yazarsan admission `VALIDATION` ile keser ve iş satırı hiç
  doğmaz. Üç değer de **üç video satırının hepsinde** geçerli (Serhat'ın 26 Ağu
  revize matrisi). ⚠ Bir gün "15 kapalı" diyen bir not ya da ekran görürsen o
  yüzey eski: 15 sn T-82 içinde bir tur boyunca kapatıldı ve aynı gün geri
  açıldı, hiç canlıya çıkmadı.
- **Oran:** görselde `1:1 16:9 9:16 4:3 3:4 3:2 2:3`, videoda `16:9 9:16 1:1 2:3
  3:2` — **`4:3` videoda YOK.**
- **`ref_urls` sırası anlamlı**: prompt onlara `ref[1]`, `ref[2]` diye hitap
  eder. 480p ve 720p 3 referans alır; 4+ endpoint'e kabul edilir ama
  `needs_human`'da durur, yani kuyruk yanar.
- **Tek ref'te oran görselden gelir**; gönderdiğin `aspect` kaydedilir ama dosyayı
  değiştirmez.
- **⛔ 1080p YALNIZ FIRST-FRAME GÖRSEL ALIR** (Serhat'ın cümlesi, T-82) ve bu
  kural artık **satırın `max_refs` kolonunda**: `bridge:grok-video-1080p` bir
  referans alır. Tek referans bridge gövdesine `image: {url}` olarak girer, yani
  **ilk kare**; ikinciden itibaren alan `reference_images` olur — başka bir iş —
  ve 1080p düşer. Sıfır referansla 1080p serbest: düz metinden-videoya (canlıda
  ölçüldü).
  ⚠ **HATA KODU `TOO_MANY_REFS`, `VALIDATION` DEĞİL.** İkinci referansı denersen
  katalog tavanından düşersin, `detail.limit = 1` gelir. Referans **listesi**
  isteyen iş `bridge:grok-video` ya da `bridge:grok-video-720p` satırında.
- **⛔ REFERANSLARINI SEN SAYMIYORSUN.** Şovda proje ya da episode erişimli bir
  **stil/prop/mekân/world sheet'i** varsa o sheet SEN İSTEMEDEN listeye giriyor
  (`automaticIngredients`). İki sonucu var ve ikisi de sessiz:
  **①** hiç `refs[]` yazmadığın bir sahne videosunda o sheet **tek referans**
  olur, yani bridge onu **ilk kare** sanır — sahnenin kendi karesi değil.
  **②** 1080p satırında tavan 1 olduğu için **iki sheet'li bir şov 1080p'ye hiç
  giremez**, ve sahnenin karesini `refs[]` ile geçersen tek sheet bile sayıyı
  2'ye çıkarıp seni dışarı atar. Yani "1080p + stil sheet'i olan şov" bugün
  olmayan bir bileşim. Red cümlesi otomatik gelenleri **adıyla** sayar
  (`style/…/sheet`) ve `detail.automatic` onları ayrı liste olarak verir — onu
  oku, kendi `refs[]` listeni sayma. Kalıcı düzeltme
  `reports/SONRA-defteri.md` T-82 → "Sahne videosunun ilk karesi bridge'e
  OTOMATİK gitmiyor", tetiği ilk gerçek 1080p sahnesi.

**Bittiğinde ne olur:**

- Dosya kalıcı olarak `cdn.jorsby.ai`'de. **R2 kopyası yok**, pointer doğrudan
  yazılır. Take normal görünür, önizlenir, sonraki üretime referans olur.
- Gerçek maliyet callback'ten gelir ve **measured** yazılır — tahmini ezer.
- **`needs_human` ve `failed` ikisi de Studio'da `failed`.** Ara durum
  açılmıyor. Fark, karttaki **error cümlesinde**: `BRIDGE_NEEDS_HUMAN: …`
  (iş reddedildi/park edildi — **isteği değiştirmeden tekrar deneme, aynı parayı
  aynı sonuç için harcarsın**) ya da `BRIDGE_FAILED: …` (teknik arıza — aynı
  isteği tekrar denemek makul).
- Retry'ı Bridge yönetir. Studio aynı task'a ikinci kez POST atmaz; senin
  "tekrar dene"n yeni bir take, yeni bir task ve **yeni bir ödeme** demektir.

Ve:

- **`status: "reused"` hata değil** — para yanmadı.
- **`failed` → otomatik retry YOK.** Dur, sebebi anlat, sor.
- **Reddi aynı parametrelerle tekrar deneme.** Reddin sebebi tipli döner; oku.
- **Tek çağrıda bütün projeyi/episode'u üretmeye kalkma.** Her para harcayan
  adımdan önce durak var.

---

## 13 · Prompt yazımı

- **Yaşayan prompt satırda yaşar.** `save_sheet.prompt`, `save_track.prompt`,
  `save_scene.video.prompt`, `frames[].prompt`. Üretim onu okur; çağrıya prompt
  yazamazsın.
- **Pozitif yaz.** İstediğin şeyi tarif et; "şu olmasın", "DO NOT", negatif liste
  yazma. İstenmeyeni anlatmanın doğru yeri `save_project.rules`'tur ve orası da
  düz cümleyle yazılır. Oraya yazılan yasak hiçbir prompt'a girmez — ikonun
  `rules`'tan aldığı tek şey `STYLE` ile başlayan paragraftır, onu da şovun
  görünüşünü ikona taşımak istiyorsan sen yazarsın.
- **Kalıbı sen yazmazsın.** Prefix/suffix prompt bank'ten gelir (`prefix_id`,
  `suffix_id`); boş bırakırsan sistem **satır doğarken** sahibin türünden doğru
  kalıbı mühürler: karakter, world, prop, mekân ve stil ayrı ayrı kalıplarda,
  başka her tür genel kalıpta. Sahnenin ilk karesi de doğarken mühürlenir. Grid
  prefix'ini **her zaman sistem basar** — tuval oranı, satır/sütun ve hangi
  cümlelerin geçerli olduğu koddan gelir; bir grid frame'ine elle prefix
  yazarsan üretim `VALIDATION` ile reddeder.
- **Kalıbın metnini şov bazında değiştirebilirsin.** Bank'teki kalıbın ADIYLA
  (`grid-frame-prefix`, `first-frame-prefix`, `world-sheet-prefix`, …)
  `save_prompt` ile projeye satır aç: o şovda seninki okunur, başka hiçbir şov
  etkilenmez. Grid kalıpları şablondur — metninde `{canvas_ratio}` `{cell_ratio}`
  `{rows}` `{cols}` `{cells}` `{shots}` `{emphasis}` `{residual}` yer tutucularını
  kullan; tanımadığı bir yer tutucu silinmez, olduğu gibi kalır ki yazım hatası
  görünsün. **Grid kalıbında `{canvas_ratio}` `{rows}` `{cols}` zorunludur:**
  geometri `params.grid`'e donuyor ve Review resmi o kadar panele bölüyor, tuval
  oranı da modele yalnız bu cümleyle gidiyor — üçünü taşımayan (ya da boş) bir
  satırla üretim `VALIDATION` ile reddedilir. **Mevcut satırlar geriye dönük
  değişmez:** kalıp yalnız YENİ sheet ve frame'lere mühürlenir, eski satırlar
  dünkü prompt'unu ve hash'ini korur; grid kalıbı ise adla okunduğu için satırı
  değiştirdiğin an o şovun grid frame'leri bayatlar ve yeniden üretilir.
- **Malzemeyi slug'la çağır**, tarifle değil. `char/ayse/sheet` yaz;
  "kahverengi saçlı kadın" diye tekrar tarif etme — ikisi çelişirse model
  ikisini de görür.
- **Somut olmayan tarifle üretme.** "Güzel bir kadın" bir sheet prompt'u
  değildir. Üretmeden önce sor.
- Referans verilen sheet'in kendi kalıbı tüketiciye taşınmaz — her satır kendi
  kalıbını kendi üretiminde işler.
- VO metni **harfiyen okunur.** Kullanıcının metnine kelime ekleme, çıkarma,
  düzeltme. Bölmen gerekiyorsa böl, ama parçaları birleştirince orijinal çıkmalı.

---

## 14 · Evrensel yasaklar

1. **Onaysız para harcama.** Her `generate_*` bir faturadır.
2. **`failed`'i otomatik retry etme.** Dur, sor.
3. **Reddi aynı parametrelerle tekrarlama.**
4. **Sessiz `ack_stale` yok.** Kullandıysan söyle.
5. **Tuval, hücre, prefix, `idem_key`, hash hesaplama.** Sistemin işi.
6. **Şemada olmayan alan gönderme:** `prompt`, `mode`, `url`, `idem_key`,
   `canvas_ratio`, `wait_ms` yoktur.
7. **Ham URL'i ref yapma.** Önce `upload_asset`, sonra `asset_id`.
8. **Kapalı op'a atıf yapma.** Müzik ve vitrinden kopyalama gün-1'de yok — söz
   verme. Render AÇIK, ama bitmesini beklemeden "video hazır" deme: cevap
   `generating` diyorsa dosya henüz yok.
9. **Açık yorum varken `approved` önerme.**
10. **`delete`'i onaysız `confirm:true` ile çağırma.** Manifest kullanıcıya
    gösterilir.
11. **Tek çağrıda bütün episode'u üretme.**
12. **"Üretildi" deme, üretilmeden.** Çağrı döndü ≠ iş bitti.
13. **Uydurma.** Op adı, alan adı, hata kodu, fiyat — bilmiyorsan `get_*` ile
    oku ya da sor.

---

## 15 · `change_voice` — sahnenin sesini değiştirmek

**Ne yapıyor.** Sahnenin **seçili video take'inde konuşulan** diyaloğu alıp aynı
sahnenin sesine çeviriyor. Dosyanın kendisi ElevenLabs'in speech-to-speech ucuna
gidiyor, geri mp3 geliyor, o mp3 sahnenin **VO take'i** oluyor. Video dosyasına
dokunulmuyor: render'da video sessiz, VO üstüne biniyor — bugün zaten öyle
çalışıyor.

**Neden lip-sync modeli gerekmiyor.** Zamanlama birebir korunuyor. Aynı dalga
formu, farklı tını: her hece daha önce hangi karede düşüyorsa orada düşüyor, yani
ekrandaki ağız hâlâ doğru. Lip-sync modelleri ters problemi çözer — görüntüden
SONRA yazılmış sesi resme uydurmak — ve bu yol o problemi hiç yaratmıyor.

**Ne zaman kullanılır.**

- Elindeki video klip konuşuyor ama sesi yanlış kişiye ait (stok görüntü, avatar
  modelinin kendi sesi, senin kendi çekimin).
- Karakterin kayıtlı sesi var ve sahne o sesle konuşsun istiyorsun.
- Metin yazıp `generate_voiceover` çağırmak, videodaki ağız hareketiyle
  tutmayacağı için işe yaramıyor.

**Ne zaman kullanılmaz.** Sahnede konuşma yoksa. Bu op ses ÜRETMİYOR, var olan
sesi çeviriyor: sessiz bir klip ElevenLabs tarafından reddedilir ve
`PROVIDER_ERROR` alırsın. Sıfırdan anlatım istiyorsan op'un adı
`generate_voiceover`.

**Çağrı.** `change_voice(target: "scene/N/voice", …)` — hedef **doldurulan
slot**, yani ses track'i. Her `generate_*` böyle: hedef, sonucun ineceği yer.

- **Kaynağı sen seçmiyorsun**: o sahnenin o anki seçili video take'i. Başka bir
  take istiyorsan önce `put` ile onu seç.
- **Sesi de sen seçmiyorsun**: track'in karakteri (`save_track.character_id`),
  karakter yoksa şovun anlatıcı sesi (`save_project.narrator_voice_id`) — yani
  `generate_voiceover` ile **birebir aynı** kural. Sesi değiştirmek istiyorsan o
  iki alandan birini değiştirirsin; take kendiliğinden bayatlar.
  **`params` içine `voice_ref` / `voice_id` yazmak da bir yol DEĞİL:** ikisi de
  `VALIDATION` ile reddediliyor (`$.params.voice_ref`). Sebebi hız değil kapsam —
  o iki alan `studio.voices` üzerinden geçiyor ve orada başka bir workspace'in
  klon sesini seçmeyi durduran bir tetikleyici var; ham bir ref o kapıya hiç
  uğramaz. Aynı kural `generate_voiceover` için de geçerli.
- **`model` alanı YOK.** Tek katalog satırı var, projede slotu da yok. Ters
  yönde de kapalı: bu katalog satırı şovun **seslendirme modeli** olarak
  seçilemez (`save_project.speech_model_id` → `VALIDATION`), çünkü girdisi metin
  değil dosya.
- `dry_run: true` fiyatı kaynak videonun **süresinden** hesaplar (§12).

**Dürüst retler:**

| Ne olduysa | Kod | Ne yaparsın |
|---|---|---|
| Sahnede seçili video take yok | `SLUG_UNRESOLVED {missing:["scene/N/video"]}` | Önce videoyu üret ya da `put` et |
| Video çok kısa (<1 sn) ya da süresi hiç ölçülmemiş | `VALIDATION` / `$.target` | Klip konuşma taşımıyor; sahneyi gözden geçir |
| Konuşacak kimse yok | `NARRATOR_REQUIRED` | Settings'ten anlatıcı sesi seç ya da track'e karakter ver |
| `params`'a ses koydun | `VALIDATION` / `$.params.voice_ref` | Sesi track'in karakteri ya da şovun anlatıcısı söyler; o alanı değiştir |
| ElevenLabs reddetti (ör. seste konuşma yok) | `PROVIDER_ERROR` | **Otomatik tekrar YOK.** Dur, sebebi söyle, sor |

**Bilmen gereken üç şey daha:**

- **Altyazı türemiyor.** Bu ucun `/with-timestamps` karşılığı yok, kelime
  zamanlaması gelmiyor. `generate_voiceover` kendi altyazısını yazar, bu yazmaz.
  Sahnede altyazı isteniyorsa `save_track(kind:"text")` ile elle yazılır.
- **Track'in `text` alanını değiştirmek take'i bayatlatır.** Ses videodan
  geliyor, metinden değil — ama sahnenin yazılı repliği hâlâ hash'in içinde, ve
  bayat rozeti burada "yazılı replik ile take artık aynı şeyi söylemiyor"
  demektir. `check_freshness` bunu `changed:["text"]` diye adlandırır — bu yolda
  `prompt` diye bir alan yok. Yeniden üretmek bunu düzeltmez (ses yine videodan
  gelir); ya metni düzeltirsin ya `ack_stale` ile bilerek geçersin — ve
  geçtiysen §10'a göre **söylersin**.
- **İki op aynı slotu doldurduğu için `reused` tek başına yetmiyor.** Aynı
  `scene/N/voice` üzerinde önce `generate_voiceover` sonra `change_voice`
  çalıştıysa, sonraki `generate_voiceover` eski VO take'ini `reused` diye
  döndürür ama slotta **voice-change take'i durur**: cevabın `put` alanı bu
  durumda `"held"` der (§8). Kullanıcıya "seslendirme geri geldi" deme; hangi
  take'in seçili olduğunu `put` söyler, değiştirmek istiyorsan `put` op'uyla
  seçersin.

---

## Op imzalarının tek kaynağı

Bu dosyadaki tablolar **yönlendirmedir, sözleşme değil.** Sözleşme
`packages/studio-op-contract/registry.mjs`'ten üretilen `docs/OPS.generated.md`
dosyasıdır; MCP bağlıysa `tools/list` zaten canlı şemaları taşır. İmzalar
buraya elle kopyalanmaz (bkz. `OPS.generated.md`).

Frontmatter'daki `op_surface` registry'nin sha256'sının ilk 12 hanesidir. Bu
dosyayla registry çeliştiğinde **registry kazanır** ve bu skill güncellenmemiş
demektir.


## Paket eki: `plays/onboarding.md`

# play · onboarding

Bir projeyi sıfırdan ilk episode iskeletine kadar götüren tek reçete.

## Ne zaman

Aç: "yeni proje açalım" · "şu brief'ten video yapalım" · kullanıcı elinde bir
doküman/metin ile geliyor · henüz `project_id` yok.

Açma: proje zaten var ve tek eksik bir karakter (→ `plays/cast.md`) ya da sahne
listesi (→ `plays/episode-author.md`).

## Ön koşullar

- MCP bağlı ve `list_catalog(kind:"model")` cevap veriyor.
- Workspace: `workspace_id` vermezsen kullanıcının tek workspace'i kullanılır.
  Birden fazlaysa `WORKSPACE_REQUIRED {workspaces[]}` gelir — o zaman hangisi
  olduğunu sor, tahmin etme.

## Adımlar

### 1 · Brief'i sen oku

Kullanıcı bir doküman, PDF, metin ya da link verdiyse **onu sen okursun** ve
özünü `save_project.aim` ile `save_project.rules`'a yazarsın.

Brief'i **dosya olarak** saklamak istersen tek yolu var ve üretime girmez:

```
upload_asset(filename, media:"document", kind:"project_document", project_id)
```

Proje dokümanı hiçbir işaretçiye yazılmaz, hiçbir take şeridinde görünmez ve
hiçbir üretim onu okumaz — rafta durur, o kadar. **Brief'in üretime giren tek
kalıcı izi `aim` + `rules`'tur**, dosyayı yüklemek onun yerine geçmez.

Doküman yoksa üç soruyla çıkar:

> **"Kime seslenecek?"**
> **"Ne için — izleyicinin bundan sonra ne yapmasını ya da ne bilmesini istiyorsun?"**
> **"Nerede yayınlanacak?"**

### 2 · Proje tipini sor — bu adım bloke edicidir

Tip kesinleşmeden **ratio, dil, stil, cast — hiçbirine geçme.** Hepsi tipin
üstüne kurulur.

> **"Bu hangi tip olacak: anlatımlı ve karaktersiz bir video mu (faceless),
> yoksa fotogerçekçi sinematik bir şey mi (cinematic)?"**

Hangi tipin ne getirdiğini tek cümleyle söyle:

- **`faceless`** — tek hat: grid. Anlatım var, konuşan yüz yok, altyazı
  kendiliğinden türer.
- **`cinematic`** — sahne başına first-frame ya da grid. VO opsiyonel.

Kullanıcı **half_avatar_half_broll** (üstte konuşan avatar, altta b-roll)
isterse: **"O hat yakında; bugün faceless ya da cinematic açabiliyoruz."**
Yazmaya kalkarsan `save_project` `TYPE_NOT_ENABLED {type, enabled[]}` döner.

### 3 · Ratio ve dil — öner, teyit al

Yayın yerinden öner, sonra **onaylat**:

> **"Yayın yerine göre 9:16 dikey ve Türkçe öneriyorum — onaylıyor musun?"**

`ratio` `^[0-9]+:[0-9]+$` kalıbındadır. `language` bir enum'dur ve **listesi bir
standart değil, bir modelin repertuvarı:** `eleven_multilingual_v2`'nin
konuştuğu 29 dil — her şovun düştüğü varsayılan model o. Kullanıcıyla dilin
adıyla konuş, alana kodu yaz ("Türkçe" yazamazsın); değer doğrudan
seslendirmeye `language_code` olarak gider.

Bilmen gereken üç incelik:

- Kodlar ISO 639-1'dir, **tek istisna `fil`** (Filipince) — o 639-2. Yani
  standarda bakıp listeyi doğrulamaya çalışma, tutmaz.
- **`hu`, `no`, `vi` listede YOK.** Bunları `turbo_v2_5` ve `flash_v2_5`
  konuşuyor, ama şov her an varsayılan modele taşınabildiği için enum onları
  taşıyamıyor. Yani red "sağlayıcı bunu söyleyemiyor" demek değil — "bu şovun
  düşeceği model söyleyemiyor" demek.
- **`en` sağlayıcıya auto-detect olarak gider.**

Red mesajı zaten bunu söylüyor; reddi gördüğünde çözüm listeden bir üye seçmek,
hata bildirmek değil.

### 4 · Projeyi aç

```
save_project(
  name, type, aim, rules, ratio, language
)
```

Altısı da create için zorunludur. `id` vermezsen create, verirsen patch.

**Model slotları kendiliğinden dolar** — `image_model_id`, `video_model_id`,
`speech_model_id` katalog default'larından gelir, sen doldurmazsın. Buna rağmen
bir yerde `MODEL_SLOT_EMPTY {slot}` görürsen o slot **gerçekten boştur**
(bugün yalnız avatar slotunda mümkün, hattı kapalı).

Dönen `id`'yi sabitle ve **bundan sonraki her çağrıda `project_id` olarak geç.**

**`WORKSPACE_REQUIRED` gelirse:** create'in henüz bir projesi yok, yani
workspace'i kendisi çözmek zorunda ve senin birden fazla üyeliğin var.
`detail.workspaces` sana çıplak uuid listesi verir; **o uuid'lerin adını
okuyabileceğin bir op yoktur.** Listeyi kullanıcıya olduğu gibi göster, hangisi
olduğunu sor, seçileni `workspace_id` ile geç ve turun geri kalanında sabit
tut. Tahmin etme, ilk uuid'i seçme (SKILL §4).

### 5 · Stil sorusu — harfiyen

> **"Spesifik bir stil istiyor musun?"**

İki yol var:

- **Üret** → `plays/cast.md`, tür `style`.
- **Atla** → hata değil. Cinematic'te stilsiz çalışmak normaldir.

**"Preset'ten seç" diye üçüncü bir dal yok.** Vitrinden kopyalama (`import`)
gün-1'de kapalıdır; teklif etme.

### 6 · Sabitler sorusu — harfiyen

> **"Bu projede her karede sabit duracak bir şey var mı — stil, dünya, tekrar
> eden bir mekân ya da nesne?"**

Söylenenleri **proje erimiyle** aç (`save_entity(reach:"project", project_id)`).
Sonra tek cümleyle söyle: *"Bundan sonraki her görsel üretime kendiliğinden
girecek."*

İki sınır:

- **Söylenmeyeni proje geneli yapma.** Proje erimi her üretimin referans
  bütçesine ve fiyatına girer (`TOO_MANY_REFS`).
- **Episode erimi burada sorulmaz** — henüz iskelet yok.

**Dünya sorusu yalnız seri/evren varsa** açılır. Tekil video ya da reklamda hiç
sorma. Otomatik kümede tek style ve tek world olur; ikincisi hata döner.

### 7 · Cast

Brief'te geçen karakterler ve mekânlar için hangisinin sıfırdan üretileceğine
karar ver. Kurulumun kendisi `plays/cast.md`'dedir.

Kimin zaten var olduğunu görmek için `list_catalog(kind:"character")` /
`list_catalog(kind:"entity")`. **Vitrinden kopyalama yok** — bulduğunu bu projeye
taşımanın gün-1'de bir yolu yoktur, yeniden kurarsın.

### 8 · Narrator sesi — atlanmaz

**Anlatıcı bir SESTİR, bir karakter değil (T-39②, 21 Ağu 2026).**
`projects.narrator_voice_id` doğrudan bir `voices` satırına bakar. Araya
"Narrator" adında bir karakter açmak GEREKMEZ — o kolon (`narrator_id`)
DEPRECATED ve `save_project` artık onu kabul etmez.

```
list_catalog(kind:"voice")                            → sesi seçtir
save_project(id, narrator_voice_id:<voices.id>)       → şova bağla
```

Bunu atlarsan ilk `generate_voiceover` `NARRATOR_REQUIRED {project_id}` ile
düşer — ve o noktada proje kurulmuş, sahneler yazılmış olur.

Sesi kullanıcıya seçtir:

> **"Anlatıcı sesi olarak hangisini istersin?"** (katalogdan 2-3 tane oku)

Satırların `labels`'ı (gender · age · accent · language) ve `preview_url`'ü var —
ikisini de ver, seçim isimden değil sesten yapılır. Katalog boşsa `sync_voices`
onu doldurur: ElevenLabs hesabının seslerini indirir, kredi harcamaz. **Staff'ın
op'u** — MCP çağrıların hesabın sahibi olarak gittiği ve o staff olduğu için
senin çağrın geçer; staff olmayan bir oturumda `FORBIDDEN` döner, o zaman
kullanıcıya söyle ve katalogda olanla devam et.

### 9 · İkon

```
generate_image(target:"project/icon", project_id)
```

**İkon prompt'unu sen yazmazsın.** `project/icon` satırının prompt kolonu yok;
op prompt'u şovun kendisinden kuruyor — bank prefix'i + `Show: <ad>.` + amaç +
`rules`'un `STYLE` ile başlayan paragrafı. İkon şovun görünüşünü ancak öyle bir
paragraf varsa taşır: §10'u bu adımdan önce yaptıysan bir kere üret,
yapmadıysan rules yazıldıktan sonra tekrar üret. Ekleyecek tek bir cümle varsa
`params:{ description:"…" }` ile gider ve sona eklenir. Önce `dry_run:true` ile
fiyatı söyle (bugün ≈$0.03), onay al.

### 10 · Rules doldurma — atlanmaz

> **"Uymamızı istediğiniz kurallar var mı?"**

Cevap `save_project.rules`'a yazılır. **`rules` bir standing instruction'dır —
sana konuşur, prompt'lara miras GEÇMEZ.** Prompt'a giden tek şey `STYLE` ile
başlayan paragraftır (aşağıda), o da yalnız prompt kolonu olmayan iki slotta:
`project/icon` ve `char/x/avatar`. Kullanıcı "yok" derse örnek ver, çünkü soru
soyut:

- "kesikten kan hiçbir yerde görünmeyecek"
- "şu karakterin yüzü hiç görünmeyecek"
- "hareketli olsun, statik kare istemiyorum"

Kuralı pozitif cümleyle yaz; bu bir prompt değil, bir standing instruction.
Yasak cümleleri buraya rahatça yaz — hiçbiri prompt'a girmez.

**Şovun görünüşünü ayrı bir paragrafa yaz ve `STYLE` ile başlat:**

```
STYLE — düz kesme-kâğıt kolaj, yumuşak kenarlar, tek yönlü ışık.

Kesikten kan hiçbir yerde görünmeyecek.
```

`STYLE` paragrafı `rules`'un resim için yazılmış tek parçasıdır ve ikonun
görünüşü oradan gelir; blob'un neresinde durduğu fark etmez. Yoksa ikon adı ve
amacıyla üretilir — düz bir ikon çıkar, yanlış bir ikon değil.

### 11 · İlk episode iskeleti

```
save_episode(project_id, title, scenes:[ … ])
```

**Tek nested çağrı, sıfır para.** Okunabilir bir taslak çıkar, kullanıcıyla
tartışılır, beğenilmezse yeniden yazılır — hiçbiri para harcamaz.

Sahne listesinin zanaatı `plays/episode-author.md`'dedir; buradan oraya geç.

### 12 · Alan doldurma disiplini

Boş kalan alan **sessizce atlanmaz.** İsim, amaç, kurallar, ratio, dil,
narrator sesi, ikon — biri eksik kaldıysa akış içinde tekrar sor. Onboarding'in
sonunda `get_project(id)` ile bir tur at ve eksik varsa söyle.

### 13 · Sıra kuralı

Yukarıdaki sıra **öneridir.** Kullanıcı hazır karakterle geliyorsa cast'ten
başla, brief'i varsa ondan. Tek gerçek bağımlılık: **görsel üretim, otomatik
kümedeki ana sheet'leri bekler.**

## Kullanıcıya sorulacak sorular

Kendi cümleni uydurma; bunlar harfiyen sorulur:

1. "Kime seslenecek?"
2. "Ne için — izleyicinin bundan sonra ne yapmasını ya da ne bilmesini istiyorsun?"
3. "Nerede yayınlanacak?"
4. "Bu hangi tip olacak: anlatımlı ve karaktersiz bir video mu (faceless), yoksa fotogerçekçi sinematik bir şey mi (cinematic)?"
5. "Yayın yerine göre <ratio> ve <dil> öneriyorum — onaylıyor musun?"
6. "Spesifik bir stil istiyor musun?"
7. "Bu projede her karede sabit duracak bir şey var mı — stil, dünya, tekrar eden bir mekân ya da nesne?"
8. "Anlatıcı sesi olarak hangisini istersin?"
9. "Uymamızı istediğiniz kurallar var mı?"

## Durma koşulları

- **Tip kesinleşmeden ilerleme.**
- **İkondan önce dur** — ilk para harcayan adım odur. `dry_run` → fiyat → onay.
- **Cast üretiminden önce dur** (`plays/cast.md`'nin kendi durağı var).
- İskeleti yazdıktan sonra dur: **sahne listesi onaylanmadan hiçbir sahne
  üretilmez.**

## Tipik hatalar

| Kod | Sebebi | Ne yaparsın |
|---|---|---|
| `WORKSPACE_REQUIRED` | Birden fazla workspace | Hangisi olduğunu sor, `workspace_id` geç |
| `TYPE_NOT_ENABLED` | `half_avatar_half_broll` denendi | "Hat yakında" de, iki tipten birini seçtir |
| `VALIDATION` | `ratio` kalıba uymuyor ya da create alanı eksik | Altı zorunlu alanı tamamla: name, type, aim, rules, ratio, language |
| `NARRATOR_REQUIRED` | Adım 8 atlandı | `save_project(id, narrator_voice_id:<voices.id>)` — karakter açma, ses seç |
| `NOT_FOUND` | `id` verildi ama satır yok | Upsert sessiz yaratmaz; `id`'yi düşür ya da doğrusunu bul |
| `MODEL_SLOT_EMPTY` | O slot gerçekten boş | Bugün yalnız avatar slotunda olur; hat kapalı |

## Sistemin garantileri

Bunları sen hesaplamazsın:

- Proje tipi seçilince hangi üretim hattının kullanılacağını **sistem bilir.**
- Model slotları katalog default'larından **kendiliğinden dolar.**
- `save_project`: `id` yoksa create, varsa patch, satır yoksa hata —
  **sessiz yaratma yok.**
- Erimi proje ya da episode olan entity'lerin ana sheet'leri kapsamına giren
  **her görsel üretime otomatik girer**; unutma riski sende değil.
- Otomatik kümede aynı anda **tek style ve tek world** olur — ikincisini
  kaydetmek hata döner.
- **Ana sheet hazır değilse üretim onu bekler**, sessizce atlamaz.
- `save_episode`'un nested iskeleti **tek transaction**'dır: ya hepsi yazılır ya
  hiçbiri.


## Paket eki: `plays/cast.md`

# play · cast

Bir karakteri, mekânı, prop'u, style'ı ya da world'ü açmak, ana görselini
üretmek ve varyasyonunu çıkarmak.

## Ne zaman

Aç: "karakter ekleyelim" · "şu mekânı kuralım" · "bir stil oluşturalım" ·
"bunun gece hali de lazım" · onboarding'in cast adımı · proje ortasında yeni bir
kimlik gerektiğinde.

Açma: var olan bir sheet'in prompt'unu düzeltmek istiyorsan — o `plays/review.md`.

## Ön koşullar

- `project_id` sabit.
- Proje tipi seçili (`faceless` ya da `cinematic`).
- Tarif somut. Değilse **üretmeden önce sor** (aşağıda).

## Adımlar

### 1 · Türü belirle

| Ne | Op | Slug kökü |
|---|---|---|
| Karakter | `save_character(name, slug, project_id)` | `char/<slug>` |
| Prop | `save_entity(type:"prop", name, slug, …)` | `prop/<slug>` |
| Mekân | `save_entity(type:"location", name, slug, …)` | `loc/<slug>` |
| Stil | `save_entity(type:"style", name, slug, …)` | `style/<slug>` |
| Dünya | `save_entity(type:"world", name, slug, …)` | `world/<slug>` |
| Referans | `save_entity(type:"reference", name, slug, …)` | **yok** |

`slug` gramerine dikkat: `^[a-z0-9][a-z0-9_-]*$`, proje içinde tek.

`reference` beşinci türdür ve **slug kökü yoktur** (gramer sonradan
değişmiyor). Yani ona `generate_*`, `put` ve `check_freshness` ile
adreslenemez; üretime `refs:[<asset_id>]` ile girer. Raf, silme ve take şeridi
diğer dördüyle aynıdır.

### 2 · Erimi sor — harfiyen

`save_entity` çağrısından **önce** sor, çünkü `reach` create'te yazılır:

> **"Bu [mekân / nesne / stil / dünya] nerede geçerli — proje geneli mi (her
> karede), bir episode boyunca mı, yoksa sahne sahne mi?"**

Cevap gelmezse varsayılanı kullan:

| Tür | Varsayılan erim |
|---|---|
| `style`, `world` | `project` |
| `prop`, `location` | `scene` — sahne bazında (proje erimine **alma**) |

`reach:"project"` seçtiysen `project_id` zorunludur. `reach:"episode"` seçtiysen
`episode_id` verilir — yani iskelet kurulmadan episode erimi seçilemez.

Proje ya da episode erimi seçildiyse tek cümleyle söyle:

> *"Bundan sonraki her görsel üretime kendiliğinden girecek."*

### 3 · Ana sheet

```
save_sheet(character_id | entity_id, prompt, project_id)
```

`character_id` ve `entity_id`'den **tam olarak biri** verilir. `name` boş
bırakılırsa bu **ana sheet**'tir — sahip başına bir tane.

`prefix_id` / `suffix_id` boş bırak: sistem sahibin türünden doğru kalıbı seçer.
Kalıbı sen yazmazsın.

**Tarif somut değilse üretmeden önce sor.** "Güzel bir kadın" bir sheet
prompt'u değildir:

> **"Biraz daha somut tarif eder misin — yaş aralığı, saç, giyim, duruş, ışık?"**

Prompt'u pozitif yaz. İstenmeyen şey `save_project.rules`'a yazılır, prompt'a
değil.

### 4 · Üret

```
generate_image(target:"<kök>/sheet", dry_run:true, project_id)   → fiyat
```

Fiyatı söyle (`estimated_cost.approx` true ise "≈" ile), onay al, sonra
`dry_run` olmadan aynı çağrıyı yap.

Çağrı hemen döner — `{asset_id, job_id, status:"queued"}`. **"Üretildi" deme.**
`check_freshness("<kök>/sheet")` ile yokla; taban 1 sn, önce bekle sonra bak.

Beğenilmezse: **yaşayan prompt'u düzelt** (`save_sheet(id, prompt)`), yeniden
üret. Asset'in kendisine dokunulmaz, eski take şeritte kalır.

### 5 · Karakterse ses

```
list_catalog(kind:"voice")            → kullanıcıya 2-3 seçenek oku
save_character(id, voice_id:<seçilen>)
```

Satırlar `labels` (gender · age · accent · language · use_case · category) ve
`preview_url` taşır. **İki isim okumak seçim değildir:** seçeneği etiketlerden
daralt, `preview_url`'ü ver ki kullanıcı dinleyip karar versin. Etiketi boş bir
satır elle açılmış demektir, eksik değil.

Katalogda aradığın ses yoksa onu sen ekleyemezsin. Sesler ElevenLabs hesabında
klonlanır, oradan `sync_voices` ile iner — **yalnız staff'ın op'u ve nadiren
gerekir.** Kullanıcıya söyle, katalogda olanla devam et.

Ses klonu gerekiyorsa örnek dosya:

```
upload_asset(filename, media:"voice", kind:"voice_example", character_id)   → asset_id
```

Tür bağlamayı kendi yapar: `characters.voice_example_id` bu çağrı dönmeden
yazılmıştır, ayrıca `put` gerekmez. **`put` `url` almaz** — eski bir take'e
dönmek istersen `put(slot:"char/<slug>/voice-example", asset_id)`.

### 6 · Upload'dan kurulum

Kullanıcı kendi fotoğrafını ya da bir marka görselini verdiyse:

```
upload_asset(filename, media:"image", kind:"character_sheet", name:"<ad>", project_id)
```

Tek çağrı: o addaki karakter yoksa açılır, ana sheet'i yoksa açılır, dosya o
sheet'in take'i olur ve işaretçisine yazılır.

⛔ **`name` SLUG'A KATLANIR, ADA BAKMAZ.** Arama `slugifyName(name)` ile yapılır,
yani `save_character(name:"Doctor Ada", slug:"ada")` satırı `"Doctor Ada"` adıyla
BULUNMAZ — `doctor-ada` diye İKİNCİ bir karakter açılır ve unique index bunu
reddedemez, çünkü iki slug farklıdır. Elinde satır varsa adla değil **id ile**
git:

```
upload_asset(filename, media:"image", kind:"character_sheet", sheet_id:<sheet id>)
```

`sheet_id` altı sheet türünde (`character_sheet` + beş entity türü) `name`'in
yerine geçer: hiçbir şey açılmaz, dosya TAM O sheet'e bağlanır ve onun
işaretçisine yazılır. **Varyasyona dosya bırakmanın tek yolu budur** — `name`
kolu tanımı gereği sahibin ANA sheet'ine iner, yani varyasyon sanıp ana sheet'i
ezersin.

Aynı slug'a düşen ikinci dosya İKİNCİ KARAKTER değil, ikinci take'tir. Sonra
prompt'u yaz (`save_sheet(id, prompt)`)
ve istersen üstüne üret:
`generate_image(target:"char/<slug>/sheet", refs:[<asset_id>], project_id)`.

Yükleme 25 MB'ı geçemez (`FILE_TOO_LARGE`). Avatar kendi türüyle gider:
`upload_asset(media:"image", kind:"character_avatar", character_id)`.

### 7 · Varyasyon

Önce fark cümlesini sor:

> **"Ana halinden farkı ne — kıyafet mi, ışık mı, yaş mı, ruh hali mi?"**

```
save_sheet(character_id | entity_id, parent_id:<ana sheet id>, name:"gece", prompt:"…")
```

Slug'ı `char/ayse/sheet/gece` olur. **Kök sheet referans olarak kendiliğinden
girer — kimlik köktedir.** Varyasyon prompt'unda karakteri baştan tarif etme,
yalnız farkı yaz.

Varyasyona dışarıdan dosya koyacaksan `sheet_id` ile koy
(`upload_asset(kind:"character_sheet", sheet_id:<varyasyonun id'si>)`); `name`
ana sheet'e iner.

### 8 · Erimi sonradan değiştirmek

Bir entity'nin erimini genişletmek ya da daraltmak **o güne kadarki görselleri
bayatlatır.** Değiştirmeden önce say ve söyle:

> **"Bunu proje geneline almak bugüne kadarki N görseli bayatlatır — yeniden
> üretelim mi, yoksa yalnız bundan sonrası mı?"**

Kaç görselin etkilendiğini `check_freshness(slug[], depth:2)` ile ölçebilirsin.

**Kendiliğinden hiçbir şey yeniden üretme.** Erim değişikliği tek başına para
harcamaz; harcayan şey senin ardından attığın `generate_*` çağrılarıdır.

## Kullanıcıya sorulacak sorular

1. "Bu [mekân / nesne / stil / dünya] nerede geçerli — proje geneli mi (her karede), bir episode boyunca mı, yoksa sahne sahne mi?"
2. "Biraz daha somut tarif eder misin — yaş aralığı, saç, giyim, duruş, ışık?"
3. "Anlatıcı/karakter sesi olarak hangisini istersin?"
4. "Ana halinden farkı ne — kıyafet mi, ışık mı, yaş mı, ruh hali mi?"
5. "Bunu proje geneline almak bugüne kadarki N görseli bayatlatır — yeniden üretelim mi, yoksa yalnız bundan sonrası mı?"

## Durma koşulları

- **Somut olmayan tarifle üretme.**
- **`dry_run` → fiyat → onay** olmadan ilk sheet'i üretme.
- Birden fazla sheet'i seri üretecekseniz **toplam fiyatı söyle ve onay al.**
- Erim daraltmadan önce dur, bayatlayacak sayıyı söyle.

## Tipik hatalar

| Kod | Sebebi | Ne yaparsın |
|---|---|---|
| `VALIDATION` | `character_id` ve `entity_id` birlikte verildi, ya da slug grameri | Tam olarak birini ver; slug'ı `^[a-z0-9][a-z0-9_-]*$`'e uydur |
| `NOT_FOUND` | `generate_image` sheet satırı açılmadan çağrıldı | **`generate_*` satır yaratmaz** — önce `save_sheet` |
| `save_entity` hatası | Otomatik kümede ikinci bir `style` ya da `world` | Tek olanı arşivle (`status:"archived"`) ya da erimini daralt |
| `TOO_MANY_REFS` | Proje erimindeki sheet sayısı `models.max_refs`'i aştı | Bir erim daralt ya da `refs[]`'ten bir tane düş |
| `SLUG_UNRESOLVED` | Prompt'ta olmayan bir slug geçiyor | Bekle ya da üret; üçüncüsü yok |
| `STALE_HASH` | Kök sheet varyasyondan sonra değişti | Tazele, ya da `ack_stale:true` + **söyle** |

## Sistemin garantileri

- **Varyasyon kökten bağımsız bir kimlik yaratmaz** — aynı sahibin altında,
  `parent_id` ile köke bağlı, kök referans olarak kendiliğinden girer.
- **Karakterde davranış/görünüm kolonu yoktur:** kimlik kökte, görünüm sheet'te,
  davranış sahne prompt'unda.
- Erimi proje ya da episode olan entity'lerin ana sheet'leri kapsamına giren
  **her görsel üretime otomatik girer.**
- Otomatik kümede aynı anda **tek style ve tek world** olur — ikincisini
  kaydetmek hata döner, "hangisi geçerli" sorusu doğmaz.
- **Ana sheet hazır değilse üretim onu bekler**, sessizce atlamaz.
- **Erim değişikliği kendiliğinden hiçbir şeyi yeniden üretmez** — yalnız bayat
  rozeti düşer.
- Kalıp (prefix/suffix) sahibin türünden **sistemce** seçilir.


## Paket eki: `plays/episode-author.md`

# play · episode-author

Brief'ten sahne listesine. Bir episode'un iskeletini yazmak — sahneler, roller,
anlatım metni, frame türü ve shot sayısı — ve hepsini **tek nested
`save_episode` çağrısına** dökmek.

Bu play para harcamaz. Bu yüzden burada cömert ol: yaz, oku, beğenmezsen
yeniden yaz. Para bir sonraki play'de başlıyor.

## Ne zaman

Aç: "sahneleri yazalım" · "senaryoyu bölelim" · "iskeleti kur" · onboarding'in
son adımı · var olan bir episode'a sahne eklenecek.

Açma: sahneler zaten yazılı ve iş üretime geldi (→ `plays/faceless.md`) · tek bir
sahnenin metnini düzeltmek yetiyor (→ `save_scene` ya da `save_track`, tek çağrı).

## Ön koşullar

- `project_id` sabit, proje tipi seçili.
- `aim` ve `rules` dolu — sahne listesi bunların üstüne kurulur.
- Anlatım metni ya elinde, ya da brief'ten sen çıkaracaksın.

## Adımlar

### 1 · Yapıyı kur — hook ve payoff

Sahne listesi bir liste değil, bir eğri. Her episode'da en az üç yer vardır:

| Rol (`role`) | İşi |
|---|---|
| `hook` | İlk sahne. İzleyicinin kalmasına sebep verir — soru, iddia, çelişki, ya da görülmemiş bir görüntü. Açıklama değil. |
| `setup` | Gövde. Her sahne **tek bir şey** söyler. İki şey söyleyen sahne iki sahnedir. |
| `payoff` | Hook'un borcunu öder. Hook'ta sorduğun soruyu burada kapat; kapatmıyorsan hook yanlış. |

`role` açık listedir — `hook`, `setup`, `payoff` dışında `turn`, `proof`, `cta`
gibi adlar da yazabilirsin. Ama **hook ve payoff birbirine bağlı olmalı**; bu
bağı kurmadan iskeleti yazma.

Yazdıktan sonra kendine sor: *hook'u okuyup payoff'a atlasam, borç ödenmiş mi?*
Cevap hayırsa aradaki sahneler değil, iki uç yanlıştır.

### 2 · Sahne süresi — sert kural

> **Bir sahne ~8 saniyelik anlatımı geçmez.**
> **2 dakikalık video ≈ 15 sahne.**

Bu ritim kuralıdır ve pahalıdır: uzun sahne izleyiciyi de düşürür, tek karede
anlatılamayacak kadar çok şey de yükler.

Pratik ölçü: Türkçe anlatım kabaca **saniyede ~14 karakter**. Yani sahne başına
**~110-120 karakter** VO metni. Bunu yazarken say — VO üretilmeden önce sayman
bedava, sonra saymak $0.026.

Hedef süreden sahne sayısı:

| Video süresi | Sahne |
|---|---|
| 30 sn | ~4 |
| 60 sn | ~8 |
| 2 dk | ~15 |
| 3 dk | ~22 |

Sahne uzunsa **böl.** Bölerken cümleyi ortadan kesme; anlatımın kendi
duraklarını kullan.

Sistem tarafındaki karşılığı: VO metni modelin kapasitesini aşarsa
`generate_voiceover` zarfında **`SCENE_TOO_LONG` uyarısı** gelir. Hata değildir,
üretim olur ve para gider. **Uyarıyı ciddiye al** — çözümü teknik değil
editoryaldir: sahneyi böl, sonra yeniden üret.

### 3 · Anlatım metnini yaz

Her sahnenin `voice.text`'i **harfiyen okunacak** metindir.

- Kullanıcının metni varsa **dokunulmazdır.** Bölebilirsin; kelime ekleyemez,
  çıkaramaz, düzeltemezsin. Böldüysen parçaları birleştir ve orijinalle
  karşılaştır.
- Metni sen yazıyorsan konuşma dilinde yaz — okunacak, okunmayacak.
  Kısaltma, parantez, madde işareti, emoji yok.
- Sayıları yazıyla yaz ("yüzde otuz", "%30" değil) — TTS ikisini aynı okumaz.
- `voice.character_id` boş bırakılırsa **şovun narrator sesi** konuşur
  (`projects.narrator_voice_id`). Faceless hatta bunu boş bırak.

### 4 · `frames.type` kararı

Her sahnenin video track'i altında en az bir frame satırı olur.

| Tip | Ne zaman | Kısıt |
|---|---|---|
| `grid` | **faceless'ta tek seçenek.** Anlatım sahnesi, birden fazla an gösterilecek. | `shots` alanı yalnız burada anlamlı |
| `first` | Yalnız cinematic'te. Sahne tek bir andır, kamera onun içinde gezer. | Kare oranı projenin `ratio`'su ile birebir |

Karar **sahne satırına bir kez yazılır**, üretim onu sorgusuz okur. Böylece iki
farklı ajan aynı sahneyi aynı türde üretir.

### 5 · `shots` (N) kararı — senin

> **N senin kararındır ve `frames.shots`'a yazılır. Tuval oranını, prefix'i ve
> hücre geometrisini yalnız sistem hesaplar; `canvas_ratio` diye bir alan
> yoktur.**

Başlangıç önerisi — sistemin kendi öneri fonksiyonuyla aynı mantık:

```
N = clamp(ceil(vo_saniye / 4), 1, 4)
```

Yani kabaca **4 saniye anlatım = 1 shot.** 8 saniyelik bir sahne → N = 2.

Sonra kendi kararınla düzelt:

- Sahne **tek bir an** anlatıyorsa N = 1 — anlatım 8 sn olsa bile. Formül
  ritmi bilmez, sen bilirsin.
- Sahnede sayılabilir bir dizi varsa ("üç aşama") N'i o sayıya çek, 4'ü geçme.
- N ∈ 1..4 tam sayı. 4'ün üstü **yoktur.**

Sistem N'i indirebilir: hücreler çözünürlük tabanının altına düşecekse
`GRID_RESOLUTION_CAP` uyarısı gelir ve N kırpılır. Uyarıyı kullanıcıya söyle.

### 6 · Video prompt'u

`video.prompt` sahnenin **hareketini** tarif eder, içeriğini değil — içerik
frame'de. Kısa yaz, pozitif yaz, malzemeyi slug'la çağır (`char/ayse/sheet`),
tarifi tekrar etme.

`frames[].prompt` ise o karenin **içeriğidir.** Grid'te tek prompt bütün tuvali
tarif eder; hücreleri tek tek numaralandırmaya çalışma, kalıbı sistem basar.

### 7 · Altyazı ve ekran yazısı

- **Faceless'ta altyazı yazılmaz.** VO bittiğinde sistem transcript'ten
  `source:"derived"` bir text track'i kendiliğinden düşürür. `text[]`'e altyazı
  yazarsan iki kat altyazı olur.
- `text[]` **başlık, isim etiketi, alt bant** gibi authored yazılar içindir.
  Yazacaksan `start_ms`, `duration_ms`, `anchor`, `preset` ile yaz —
  `duration_ms` sahneyi uzatamaz.

### 8 · `transition_out`

Preset adıdır; boş bırakmak **kesme** demektir. Emin değilsen boş bırak.

### 9 · Tek nested çağrıya dök

```
save_episode(
  project_id,
  title,
  rules,                       // opsiyonel — proje kurallarının ÜSTÜNE eklenir
  scenes: [
    {
      role: "hook",
      summary: "…",            // insan için tek cümle
      transition_out: "",
      voice: { text: "…" },    // character_id boş = şovun narrator sesi
      video: {
        prompt: "…",
        frames: [ { type: "grid", idx: 0, shots: 2, prompt: "…" } ]
      }
    },
    …
  ]
)
```

Kurallar:

- **Sıra dizinin sırasıdır.** `idx` yazmazsın, motor damgalar.
- **Tek transaction** — ya hepsi yazılır ya hiçbiri.
- Bu iskelet çağrısı **düzenleme alanlarını almaz**: `in_ms`, `out_ms`, `gain`,
  `muted`, `status`, `refs` burada yoktur. Onlar `save_scene`'in işidir. On beş
  sahneyi planlarken kimse fade-in ayarlamaz.
- **`avatar` bloğu yazma** — avatar hattı gün-1'de kapalı.
- **`music` dizisi yazma** — müzik gün-1'de kapalı, `music_id` üretemezsin.
- **`save_frame` diye bir op yoktur.** Frame'ler buradan ya da
  `save_scene.video.frames[]`'ten yazılır.

### 10 · Oku ve onaylat

İskeleti yazdıktan sonra `get_episode(id, include:["scenes","tracks","frames"])`
ile geri oku ve kullanıcıya **düz bir liste** olarak göster: sahne no · rol ·
anlatım metni · shot sayısı. Toplam süre tahminini de söyle
(karakter sayısı ÷ 14).

> **"Toplam N sahne, ~M saniye. Böyle mi ilerleyelim?"**

Onay gelmeden `plays/faceless.md`'ye geçme. Buraya kadar hiç para harcanmadı;
bundan sonrası harcıyor.

## Kullanıcıya sorulacak sorular

1. "Video ne kadar sürsün — 30 saniye, 1 dakika, 2 dakika?"
2. "Anlatım metni sende mi, yoksa ben mi yazayım?" *(sendeyse harfiyen kullanılır)*
3. "Açılışta izleyiciyi ne tutacak — bir soru mu, bir iddia mı, bir görüntü mü?"
4. "İzleyici sonunda ne bilsin ya da ne yapsın?" *(payoff bunu ödeyecek)*
5. "Toplam N sahne, ~M saniye. Böyle mi ilerleyelim?"

## Durma koşulları

- **İskeleti onaylatmadan üretime geçme.** Bu, para harcamadan önceki son ucuz
  durak.
- Kullanıcının metni varsa ve bölmen gerekiyorsa, **bölmeyi onaylat.**
- Sahne sayısı hedef süreyle tutmuyorsa dur ve söyle; sessizce 25 sahne yazma.

## Tipik hatalar

| Kod | Sebebi | Ne yaparsın |
|---|---|---|
| `VALIDATION` | `shots` 4'ü aştı ya da `frames.type` bilinmeyen bir değer | N ∈ 1..4, tip ∈ {grid, first} |
| `VALIDATION` | Şemada olmayan alan (`canvas_ratio`, `prompt` kökte, `id` uydurma) | Yalnız şemadaki alanları gönder |
| `NOT_FOUND` | `id` verildi ama episode yok | Upsert sessiz yaratmaz |
| `TOO_MANY_ROWS` | `get_episode` çok geniş `include[]` ile çağrıldı | Dalları daralt ya da `limit` düşür |
| `SCENE_TOO_LONG` *(uyarı)* | Anlatım, modelin kapasitesini aşıyor | **Sahneyi böl**, sonra yeniden üret. Teknik değil, editoryal düzeltme |
| `GRID_RESOLUTION_CAP` *(uyarı)* | N, hücreleri çözünürlük tabanının altına düşürüyordu | Sistem N'i indirdi; kullanıcıya söyle |

## Sistemin garantileri

- **Sıra dizinin sırasıdır**; indexleri motor damgalar, sen saymazsın.
- Nested `save_episode` **tek transaction**'dır.
- Frame türü satıra bir kez yazılır; **üretim onu sorgusuz okur** — iki ajan
  aynı sahneyi aynı türde üretir.
- Tuval oranı, hücre geometrisi ve grid prefix'i **yalnız sistem** hesaplar.
- Altyazı VO'dan **kendiliğinden türer** — yazmazsın.
- `duration_ms` bir text track'i sahneyi uzatamaz.
- Sahne taşımak istersen `move_scene(scene_id, idx)`; kalanları motor tek kilit
  altında yeniden numaralar.


## Paket eki: `plays/faceless.md`

# play · faceless

`faceless` proje tipinin tek hattı — anlatımlı, karaktersiz grid üretimi.
İskeletten üretilmiş sahnelere.

**Bu play para harcar.** Her adımda durak var.

## Ne zaman

Aç: "bu episode'u üretelim" · "VO'yu alalım" · "grid'i çıkar" · "videoyu üret" ·
`plays/episode-author.md` biter bitmez.

Açma: iskelet yok (→ `plays/episode-author.md`) · üretilmiş bir take
düzeltilecek (→ `plays/review.md`).

## Ön koşullar

- Episode iskeleti yazılı ve **kullanıcı onayladı.**
- Şovda **narrator sesi var** — `projects.narrator_voice_id` dolu. Yoksa ilk VO
  `NARRATOR_REQUIRED` ile düşer (→ `plays/onboarding.md` adım 8).
- Otomatik kümedeki ana sheet'ler **`ready`.** Değilse üretim onları bekler.
- `project_id` ve `episode_id` elinde — sahne slug'ları ikisini de ister.

**İskelet eksikse tek çağrıda yazılır.** `save_episode` sahneleri **ve
frame'leri birlikte** kabul ediyor:

```
save_episode(project_id, title, scenes:[
  { role, summary,
    voice: { text },
    video: { prompt, frames:[ { type:"grid", shots:N } ] } }, … ])
```

Sahne başına ayrı `save_scene` çağrısı **şart değil**, `save_frame` diye bir op
zaten yok. Buraya gelmeden önce iskeleti kurman gerekiyorsa
`plays/episode-author.md`'ye dön; oradan geldiysen frame'ler yazılıdır.

## Hat

Tek hat vardır: **grid.** Frame türü `grid`, başka seçenek yok. Sahne başına
sıra sabittir ve bozulmaz:

```
generate_voiceover(scene/N/voice)
        ↓  ölçülen süre sahnenin saatini kurar
generate_image(scene/N/video/frame)
        ↓  grid hazır
generate_video(scene/N/video)
```

## Adımlar

### 1 · Nereden devam edileceğini oku

```
get_episode(id, include:["scenes","tracks","frames","takes"])
```

**Devam noktası içerik statülerinden okunur** — ayrı bir koşu kaydı yok. Sekme
kapansa, oturum bitse, doğru yerden devam edersin. Ayrı bir `get_status` op'u
yoktur.

### 2 · Toplam fiyatı söyle ve onay al

Seri üretime başlamadan **toplamı** söyle. Sahne başına gün-1 katalog fiyatı:

| Adım | Model | Hesap | Fiyat |
|---|---|---|---|
| VO (~120 karakter) | `elevenlabs:eleven_multilingual_v2` | $0.00022/kar. × 120 | ≈$0.026 |
| Grid görseli (1 tuval) | `kie:gpt-image-2` 1K | 6 kredi | $0.03 |
| Video 8 sn | `kie:grok-imagine-1.5` 480p | 2.4 kredi/sn × 8 = 19.2 kredi | ≈$0.096 |
| **Sahne toplamı** | | | **≈$0.152** |

10 sahne ≈ **$1.52**, %30 retry payıyla ≈ **$1.98**.

(1 kie kredisi = $0.005. Tablodaki VO ve video satırları prod'da `estimated` —
o yüzden `≈` ile; görsel satırı `measured`, kesin.)

Yeni bir model ya da yeni bir tür ilk kez deneniyorsa **`dry_run: true`** ile
gerçek rakamı al. `estimated_cost.approx` true ise rakamı "≈" ile söyle — o
fiyat ölçülmedi.

> **"N sahne için toplam ≈$X. Başlayayım mı?"**

Onaysız seri üretim yok.

### 3 · VO önce — her zaman

```
generate_voiceover(target:"scene/0/voice", project_id, episode_id)
```

VO önce gelir çünkü **ölçülen süre sahnenin saatini kurar**: video track'inin
uzunluğu, shot başına düşen saniye, altyazı zamanlaması — hepsi bu ölçümden.
Tahmin edilen süre değil, **ölçülen** süre.

Metin `scene/N/voice` satırının kendi `text` kolonundan okunur; çağrıda metin
göndermezsin. Metni düzeltmen gerekiyorsa `save_track(id, text)` ya da
`save_scene(...voice.text)`, sonra yeniden üret.

Zarf `SCENE_TOO_LONG` uyarısı taşıyorsa **dur ve söyle** — sahne bölünmeli
(`plays/episode-author.md` §2).

### 4 · Grid

```
generate_image(target:"scene/0/video/frame", project_id, episode_id)
```

**Tuval oranını, hücre sayısını, hücre geometrisini ve template prefix'ini
SİSTEM basar. HESAPLAMA.**

Senin elindeki tek sayı `frames.shots` (N) ve o zaten iskelette yazılı. Değiştirmek
istersen `save_scene(...video.frames[].shots)` ile satırı düzelt, sonra üret.
**`canvas_ratio` diye bir alan yoktur**; göndermeye kalkarsan `VALIDATION` yersin.

Otomatik küme buraya kendiliğinden girer — style, world, proje erimindeki
mekânlar. Kaç ref girdiğini görmek istiyorsan `dry_run:true`; toplam
`models.max_refs`'i aşarsa `TOO_MANY_REFS {limit, resolved[], automatic[]}` gelir
ve çözüm bir erim daraltmaktır, ref eklemek değil. `automatic[]` senin
yazmadıklarını ayrı listeler — hangi erimi daraltacağını oradan seç.

Grid **tek görseldir** — N shot'ın hepsi aynı tuvalde. N kere üretim yapma.

### 5 · Video

```
generate_video(target:"scene/0/video", project_id, episode_id)
```

Grid modele **bütün** gider; satırlar sıralı shot'lardır. "Animate" diye ayrı
bir mod yoktur, `mode` parametresi de yoktur.

**Frame'siz sahne videosu üretilmez** — guard reddeder, uydurmaz. Adım 4
bitmeden buraya gelme.

Hangi modele referansın hangi alanla gittiği **adaptör kodunun bilgisidir.**
`models.spec` yalnız vitrin metnidir; ona bakıp çağrı şeklini değiştiremezsin.

### 6 · Altyazı — yazma, kendiliğinden türer

VO `ready` olduğunda sistem transcript'ten `source:"derived"` bir text track'i
sahneye **kendiliğinden** düşürür. Sen hiçbir şey çağırmazsın.

Buna ek olarak **authored** bir yazı istiyorsan (başlık, isim etiketi, alt bant):

```
save_track(scene_id, kind:"text", text:"…", start_ms, duration_ms, anchor, preset)
```

`source:"authored"` olur ve derived satırla yan yana yaşar. `duration_ms`
sahneyi uzatamaz.

Derived altyazı beğenilmediyse: **düzeltmenin yolu VO'dur.** Metni düzelt,
VO'yu yeniden üret, altyazı da yenilenir. Derived satırı elle yamamaya çalışma.

### 7 · Yoklama

**Üretim çağrısı hemen döner, "bitti" demek değildir.**

İlerlemeyi toplu yokla:

```
check_freshness(slug:[ "scene/0/voice", "scene/0/video/frame", "scene/0/video", … ])
```

- **Toplu sor.** 10 sahnelik koşuda 30 çağrı değil, **1 çağrı × 30 slug.** Sebebi
  hız değil bağlam.
- Taban 1 sn, **önce bekle sonra bak.** Üstel: 10 → 20 → 40 → 60 sn tavan.
- `QUEUE_FULL` / `TRACK_BUSY` `retry_in` verirse **onu kullan**, üstüne 0-10 sn
  rastgele ekle. **`QUEUE_FULL` `{limit: 0}` ile geldiyse bekleme — o workspace
  hiç üretemez, dur ve söyle** (SKILL §12).
- Süre referansı: VO ≈ 30 sn · görsel ≈ 1 dk · video 8 sn ≈ 4 dk.

**⛔ `check_freshness` düşen işi GÖSTERMEZ.** Döndürdüğü üç değer var —
`fresh` · `stale` · `unbuilt` — ve **`failed` bunların arasında yok.** Düşmüş
bir üretim de, hiç başlamamış bir slot da aynı şekilde `unbuilt` görünür. Bu
listeyi harfiyen yoklayan bir ajan, parası yanmış bir işi 60 saniyelik tavanla
sonsuza kadar yoklar.

Bu yüzden ikinci ok şart — **gerçek statü take satırındadır:**

```
get_takes(slot)                        # tek slotu ölçmek için
get_episode(id, include:["takes"])     # bütün bölümü tek çağrıda taramak için
```

- Bir slot beklenen süreyi (video ≈4 dk) belirgin biçimde aştığı halde hâlâ
  `unbuilt` ise **yoklamaya devam etme**, take'in `status`'una bak.
- **`failed` görünce yoklamayı kes.** Otomatik retry yok — dur, sebebi düz
  Türkçe anlat, sor.

Beklerken kullanıcıyı boş bırakma: "videolar sırada, ~4 dakika" de, sonra dön.

### 8 · Paralellik

Guard **track başınadır**, sahne başına değil. Yani aynı sahnenin VO'su ve bir
başka sahnenin grid'i **paralel koşabilir**; aynı track'e ikinci istek
`TRACK_BUSY` alır.

Üst sınır workspace'in kendi tavanıdır: `coalesce(job_limit, 7)` — çoğu
workspace'te 7, ama **0 olabilir** (üretim kapalı doğan kiracı; `QUEUE_FULL`
`detail.limit: 0` verir ve retryable DEĞİLDİR, §8'e bak). Tavanı aşacak
şekilde sıraya sokma; sıradaki sahneye geçmeden önce yoklama turunu tamamla.

### 9 · Bitince

Bütün sahneler `ready` olunca bütünlüğe bak:

- Ritim akıyor mu — üst üste iki uzun sahne var mı?
- Hook'un borcu payoff'ta ödendi mi?
- Sahneler arası görsel tutarlılık — aynı style sheet'i her karede göründü mü?

Sonra `plays/review.md`'ye geç.

### 10 · Müzik ve render

- **Müzik yok.** Müzik op'ları gün-1'de kapalı (`OP_DISABLED`). "Arka plan
  müziği ekleyelim" **deme, söz verme.**
- **Render var.** Bütün sahneler onaylandıktan sonra `render_episode(episode_id)`.
  Cevap `status:'generating'` ise dosya HENÜZ YOK — `get_episode`'un `render`
  alanı `ready` olana kadar "video hazır" deme. `RENDER_IN_FLIGHT` gelirse zaten
  uçan bir render var, bekle. Kurgu değişmediyse biten render aynen döner
  (`reused`), bu bir hata değil.

## Kullanıcıya sorulacak sorular

1. "N sahne için toplam ≈$X. Başlayayım mı?"
2. "Bu sahnenin anlatımı tavanı aşıyor — bölelim mi?" *(`SCENE_TOO_LONG` gelince)*
3. "Şu üretim başarısız oldu: <sebep>. Yeniden deneyelim mi, yoksa metni mi
   değiştirelim?" *(`failed` gelince — asla kendiliğinden deneme)*
4. "Referans bütçesi doldu. Hangi erimi daraltalım?" *(`TOO_MANY_REFS` gelince)*

## Durma koşulları

- **Toplam fiyat onaylanmadan seri üretim yok.**
- **`failed`'de dur.** Otomatik retry yasak.
- **`STALE_HASH`'te dur.** Ya tazele ya `ack_stale:true` + söyle. Sessiz geçiş yok.
- **Onaylı bir slota üretim yolladıysan sonucu doğrula.** Taze üretim zarfında
  `APPROVED_SLOT` uyarısı **BEKLEME** — gelmez; red callback anında oluyor ve
  cevaba hiç girmiyor. Sonucun nereye gittiğini `get_takes(slot)` ile oku
  (`reused` cevaplarında `put:"approved_slot"` alanı da söyler): take şeride
  düşer, seçili take değişmez, taşımak istiyorsan elle `put` gerekir. Sessizce
  geçme, kullanıcıya söyle.
- Bir sahne beklenmedik biçimde pahalıya çıkıyorsa dur, seriyi durdur, söyle.

## Tipik hatalar

| Kod | Sebebi | Ne yaparsın |
|---|---|---|
| `NARRATOR_REQUIRED` | Şovda narrator sesi yok (ya da konuşan karakterin sesi yok) | `plays/onboarding.md` adım 8 |
| `NOT_FOUND` | Track ya da frame satırı yok | `generate_*` satır yaratmaz — `save_scene` / `save_track` ile aç |
| `SLUG_UNRESOLVED` | Grid, VO bitmeden istendi ya da sheet yok | Bekle ya da üret; üçüncüsü yok |
| `TRACK_BUSY` | O track'te zaten bir üretim uçuyor | `retry_in` + jitter bekle; başka track'e geç |
| `QUEUE_FULL` | Workspace'in tavanı doldu (`coalesce(job_limit, 7)`) | `detail.limit > 0` ise `retry_in` + jitter. **`limit == 0` ise dur** — o workspace hiç üretemez, bekleyerek geçmez |
| `TOO_MANY_REFS` | Otomatik küme + slug + refs > `models.max_refs` | Bir erim daralt ya da bir ref düş |
| `STALE_HASH` | Sheet ya da metin, üretimden sonra değişti | Tazele, ya da `ack_stale:true` + **söyle** |
| `VALIDATION` | `canvas_ratio` / `prompt` / `mode` gibi olmayan alan | Şemada olmayanı gönderme |
| `PROVIDER_ERROR` | Sağlayıcı düştü, job `failed` | Dur, sor. Otomatik retry yok |
| `OP_DISABLED` | Müzik op'u çağrıldı | Gün-1'de kapalı; söz verme |
| `RENDER_IN_FLIGHT` | Bölümün render'ı zaten uçuyor | Bekle, sonra tekrar sor |
| `SCENE_TOO_LONG` *(uyarı)* | Anlatım kapasiteyi aşıyor | Sahneyi böl, yeniden üret |
| `GRID_RESOLUTION_CAP` *(uyarı)* | N indirildi | Kullanıcıya söyle |
| `APPROVED_SLOT` *(uyarı)* | **Taze üretimde GELMEZ** — bu uyarı yalnız açık yorumun onaylı sahneyi `review`'a indirdiği yerden çıkar | Onaylı slot reddini `get_takes` ya da `reused` cevabının `put:"approved_slot"` alanından oku; pointer ezilmedi, elle `put` gerekir |

## Sistemin garantileri

- **Hücre sayısı, tuval oranı ve grid prefix'i sistemden gelir** — ajan
  parametresi değildir.
- **Frame'siz sahne videosu üretilmez** (guard).
- Erimi proje ya da episode olan entity'lerin ana sheet'leri otomatik girer;
  küme boşsa **sessizce boş geçer**, hata değil.
- **Altyazı VO transcript'inden kendiliğinden türer** (`source:"derived"`).
- **Track başına tek aktif üretim.** İkinci istek `TRACK_BUSY` döner —
  bekletir, kırmaz.
- **Aynı `idem_key` ikinci kez paraya gitmez.** `idem_key` senin işin değil;
  `replayed: true` ve `status: "reused"` hata değildir.
- Sıra kimse tarafından zorlanmaz: her adım kendi ingredient'ını bekler.
- Üretim bittiğinde pointer'ı **sistem** koyar — tek istisna `approved` slot.
- **Bayat girdiyle sessiz üretim yok.**


## Paket eki: `plays/review.md`

# play · review

Take şeridi üzerinden gözden geçirme, düzeltme ve onay dili.

## Ne zaman

Aç: "şu take'i beğenmedim" · "eskisine dönelim" · "bunu biraz daha …yapalım" ·
"onaylayalım" · "yorum bıraktım" · bir sahne üretildikten sonra.

Açma: hiç take yok (önce üret) · yapısal değişiklik gerekiyor, sahne bölünecek
ya da eklenecek (→ `plays/episode-author.md`).

## Ön koşullar

- İlgili slot'ta en az bir take var.
- `project_id` (ve sahne slot'uysa `episode_id`) elinde.

## Adımlar

### 1 · Şeridi getir

```
get_takes(slot:"scene/3/video/frame", project_id, episode_id)
```

Dönen liste **o slot için üretilmiş ya da yüklenmiş her şeydir**, yenisi başta,
`selected_asset_id` işaretli. Uploadlar da içindedir.

Kullanıcıya şeridi anlatırken hangisinin seçili olduğunu **açıkça** söyle.

### 2 · Müşterinin referans görsellerini oku

`scene_refs` ÖLDÜ (T-19): tablo damgalandı, hiçbir op yazmıyor, hiçbir okuma
servis etmiyor. Müşterinin "şuna benzesin" görseli artık bir entity:

```
upload_asset(filename, media:"image", kind:"reference", name:"<ad>", project_id)
```

Diğer entity'lerle aynı rafta durur (`list_catalog`, Cast ekranı) ve üretime
`refs:[<asset_id>]` ile girer — otomatik enjeksiyon almaz, yani adı geçmeden
hiçbir kareye girmez. Düzeltme isterken **onları oku.** Böyle bir görsel
varken kendi yorumunla üretmek en sık yapılan kayma.

### 3 · Eski bir take beğenildiyse — `put`

```
put(slot:"scene/3/video/frame", asset_id:<beğenilen take>, project_id, episode_id)
```

**Hiçbir asset silinmez, yalnız işaret taşınır.** Elle `put` her zaman
serbesttir; "eski take'e dön" adımı tam olarak budur.

`put` **`url` almaz.** Kullanıcı dışarıdan bir dosya verdiyse:

```
upload_asset(filename, media:"image", kind:"scene_frame", scene_id)   → asset_id
```

Sahne türleri (`scene_video`, `scene_voice`, `scene_frame`) dosyayı doğrudan o
sahnenin take'i yapar; şeritte görünür ve işaretçi ona geçer. Başka bir slot
için `put(slot, asset_id)` her zaman serbesttir.

`put` `for_id`'ye dokunmaz — şeridin görünürlüğü bundan etkilenmez.

### 4 · Düzeltme isteniyorsa — yaşayan prompt'a yaz

**Asset'in kendisine dokunulmaz.** Düzeltme her zaman satırın kendi prompt'una
yazılır:

| Ne düzeltiliyor | Op |
|---|---|
| Sheet görseli | `save_sheet(id, prompt)` |
| Sahne frame'i | `save_scene(id, video:{ frames:[{ id, prompt }] })` |
| Sahne videosunun hareketi | `save_scene(id, video:{ prompt })` ya da `save_track(id, prompt)` |
| VO metni | `save_track(id, text)` |
| Ekran yazısı | `save_track(id, text, start_ms, duration_ms)` |
| Proje geneli bir kural | `save_project(id, rules)` |

Sonra yeniden üret. **Önceki take'i `refs`'e koy** — model neyin düzeltileceğini
görsün:

```
generate_image(target:"scene/3/video/frame", refs:[<önceki take asset_id>], project_id, episode_id)
```

Yeni take şeride düşer, eskisi durur.

Düzeltmeyi **pozitif** yaz. "Şu olmasın" değil, istediğini tarif et. Sürekli
tekrarlanan bir kısıt varsa yeri prompt değil, `save_project.rules`'tur.

### 5 · Upscale — ayrı bir iş değil

```
generate_image(target:<slot>, refs:[<önceki take asset_id>], model:"<upscaler model_ref>", project_id)
```

Ayrı bir op yoktur, ayrı bir `mode` yoktur. Sıradan bir take olarak şeride
düşer. Hangi upscaler'ın olduğunu `list_catalog(kind:"model")` söyler.

Model **kendi alanıyla** geçilir — `model`, projenin slotunu o çağrı için ezer.
Eski dokümanlarda bu "params.model" diye yazılıdır; `params` sağlayıcı
parametreleri içindir ve slot'u ezmez.

### 6 · Konuşma — `save_comment`

```
save_comment(project_id, scene_id, body:"…")                 → yeni, status:"open"
save_comment(id:<kök yorum>, status:"resolved")              → kapat
save_comment(project_id, scene_id, parent_id:<kök>, body:"…") → cevap
```

- Yorum projeye, episode'a ya da sahneye asılır. `episode_id` ve `scene_id`'den
  **en fazla biri** verilir; ikisi de yoksa proje yorumudur.
- Cevaplar `parent_id`'ye bağlanır; **kök yorum** resolve edilir.
- **Yorum silinmez.** Kapatmak `status:"resolved"` patch'idir.

### 7 · Onay kapısı

Onay ayrı bir mekanizma değildir: **"konuşulacak şey kalmadı"nın kendisidir.**

```
save_scene(id, status:"approved")
```

- **Açık yorum varken `approved` önerme.** Denersen `OPEN_COMMENTS {comment_ids[]}`
  gelir. Önce yorumları çöz.
- **Onaylı bir sahneye yeni açık yorum düşerse sahne kendiliğinden `review`'a
  iner.** Bunu kullanıcıya söyle — sessizce olmasın.
- Onaylı slot'un pointer'ını **otomatik üretim ezmez.** Sonuç şeride düşer,
  seçili take değişmez; üzerine yazmak elle `put` ister. **Ama bunu taze üretim
  zarfında bir `APPROVED_SLOT` uyarısı olarak GÖRMEZSİN** — red callback anında
  oluyor ve cevaba girmiyor. Okuyabileceğin yer: `get_takes(slot)`, ya da
  `reused` cevabındaki `put:"approved_slot"` alanı.

### 8 · Şüphede tazelik

```
check_freshness(slug:[ … ], depth:2)
```

`decision ∈ {fresh, stale, unbuilt}` ve `changed[]` neyin eskidiğini söyler.

**Sessiz geçiş yok.** İki meşru yol vardır: tazele, ya da `ack_stale:true` ile
bilerek devam et **ve söyle**:

> *"style sheet'i değişmiş; 12 kareyi bilerek eskisiyle bırakıyorum."*

Hatırlat: stale, asset'in **kendi kayıtlı modeliyle** karşılaştırır. Proje
modelini değiştirmek onaylı sahneleri bayat yakmaz. Kullanıcı "model değiştirdim,
her şey bozuldu mu?" derse cevap **hayır**.

`check_freshness` danışma ucudur; dispatch anındaki guard'ı atlatmaz.

### 9 · Silme — istisnai

```
delete(target:{ kind, id }, confirm?, confirm_name?)
```

`kind` şunlardan biri: `scene` · `track` · `sheet` · `entity` · `character` ·
`music` · `episode` · `episode_text` · `project`.

`confirm` olmadan `CONFIRM_REQUIRED` + manifest döner. **Manifest'i kullanıcıya
göster**, onayını al, sonra `confirm:true` ile dön.
**`kind:"project"` ayrıca projenin TAM ADINI `confirm_name` ile ister**
(uymazsa `VALIDATION` / `$.confirm_name`). Arşivlemek başka bir şeydir ve bu op
yapmaz — o `save_project(status:"archived")`.

**Silmenin iki anlamı var; hangisinin koştuğunu cevaptaki `emptied` söyler:**

| Hedef | Ne olur | Cevap |
|---|---|---|
| Sahnenin voice / video / avatar track'i | **Slot boşaltma.** Satır YAŞAR: işaretçi ve trim boşalır, `status` `draft`'a döner. Take'ler durur, `put` ile herhangi biri geri seçilir. Prompt/metin, gain/muted ve karakter kalır, frame'lere dokunulmaz | `emptied: true`, manifest `would_empty` |
| Geri kalan her tür (metin/caption track'leri dahil) | **Yapı silme.** Satır kalır ama kayıt her listeden ve okumadan düşer | `emptied: false`, manifest `would_delete` |

**Her iki yolda da hiçbir asset silinmez**, R2'ye dokunulmaz, take'ler yerinde
kalır. Altında kuyrukta ya da üretimde bir iş varsa `JOB_IN_FLIGHT` döner; bekle
ya da iptal et, sonra sil.

### 10 · Teslim

Teslim şudur:

1. Bütün sahneler `approved`, açık yorum yok.
2. `check_freshness` ile son bir tazelik turu; `changed[]` varsa **kullanıcıya
   açıkça söyle.**
3. `get_episode(id, include:["scenes","tracks","frames","takes"])` ile seçili
   asset'lerin listesini ver.
4. `render_episode(episode_id)`. Cevap `generating` ise dosya HENÜZ YOK; teslim
   cümlesini `get_episode`'un `render.status` alanı `ready` olunca kur ve
   `render.url`'i ver.
5. `render.stale` **true** ise, o video artık kurguyu göstermiyor: kullanıcıya
   söyle ve yeniden render öner. `stale` alanı hiç yoksa bölüm hiç render
   edilmemiştir — bu ayrı bir cümledir, "eskimiş" değil.

Söz verme, tarih verme.

## Kullanıcıya sorulacak sorular

1. "Şeritte N take var, şu an <hangisi> seçili. Hangisinde kalalım?"
2. "Neyi değiştirelim — ne olmasını istiyorsun?" *(negatif değil, pozitif cevap iste)*
3. "Bu düzeltme yalnız bu kareye mi, yoksa proje kuralı mı olsun?" *(ikincisi `rules`)*
4. "Şu sahnede açık yorum var, onay için önce onu kapatmamız gerekiyor — çözelim mi?"
5. "<N> girdi eskimiş. Yeniden üretelim mi, yoksa bilerek eskisiyle mi devam edelim?"
6. "Silinecekler: <manifest>. Onaylıyor musun?"

## Durma koşulları

- **Açık yorum varken onay önerme.**
- **`ack_stale`'i sormadan kullanma.**
- **`delete`'i manifest gösterilmeden `confirm:true` ile çağırma.**
- Yeniden üretim para harcar — **her regen'den önce fiyatı söyle.** Bir seride
  toplamı söyle.
- Onaylı bir slota üretim düştüyse dur ve söyle — bunu zarftaki bir uyarıdan
  değil, `get_takes` ya da `put:"approved_slot"` alanından öğrenirsin.

## Tipik hatalar

| Kod | Sebebi | Ne yaparsın |
|---|---|---|
| `OPEN_COMMENTS` | Açık yorum varken `approved` denendi | `comment_ids[]`'i oku, çöz, sonra onayla |
| `NOT_FOUND` | `put` var olmayan bir `asset_id` ile çağrıldı | `get_takes` ile gerçek id'yi al |
| `VALIDATION` | `put(url:…)` denendi | `url` alanı yok — `upload_asset` → `asset_id` |
| `CONFIRM_REQUIRED` | `delete` onaysız | Manifest'i göster, onay al, `confirm:true` |
| `STALE_HASH` | Girdi eskimiş | Tazele ya da `ack_stale:true` + söyle |
| `TOO_MANY_ROWS` | `get_takes` / `get_episode` çok geniş | `limit` düşür, `include[]` daralt |
| `FORBIDDEN` | Rol yetmiyor (yorumcu üretim tetikleyemez) | Zorlama, söyle |
| `RENDER_IN_FLIGHT` | Bölümün render'ı zaten uçuyor | Bekle, sonra tekrar sor |
| `APPROVED_SLOT` *(uyarı)* | **Onaylı slot reddi DEĞİL** — bu uyarı açık yorumun onaylı sahneyi `review`'a indirdiği yerden çıkar | Sahnenin `review`'a indiğini kullanıcıya söyle. Onaylı slot reddi için `get_takes` / `put:"approved_slot"` |

## Sistemin garantileri

- **Hiçbir take silinmez** — geçmiş hep durur, "seçili" olan tek gerçek
  pointer'dır.
- **Onay ayrı bir mekanizma değil:** tüm yorumlar `resolved` olunca `approved`
  açılır.
- **Onaylı slot'un pointer'ını otomatik üretim ezmez** — üzerine yazmak elle
  `put` ister.
- Onaylı bir sahneye yeni açık yorum düşerse sahne **kendiliğinden `review`'a
  iner.**
- `put` `for_id`'ye dokunmaz; şerit "for_id geçmişi ∪ seçili" birliğidir.
- **Silme manifesti hiçbir asset içermez.**
- Yeniden üretim, seçili asset'in `model` + `params` + malzemelerini **aynen**
  kullanır; değiştireceğini çağrıda geçersin.
- Elle `put` monoton guard'a takılmaz — o guard yalnız sistem yolundadır.


## Paket eki: `OPS.generated.md`

# Op imzaları burada değil

Bu dosya bir işaret levhasıdır, sözleşme değildir. **Bu klasörde hiçbir op
imzası elle yazılmaz.**

## Sözleşme nerede

| Nerede | Ne | Kim okur |
|---|---|---|
| `packages/studio-op-contract/registry.mjs` | Tek kaynak: 39 op, JSON şemaları, policy'leri | Dispatcher, generator, frontend |
| `docs/OPS.generated.md` *(bu reponun kökünden)* | Üretilmiş insan-okur döküm: her op'un description'ı, policy'si, input ve output şeması | İnsan |
| MCP `tools/list` | Canlı tool listesi — yalnız `enabled` op'lar | Ajan, her istekte |

Üretme komutu, repo kökünden:

```
node scripts/gen-mcp-tools.mjs           # yazar
node scripts/gen-mcp-tools.mjs --check   # drift varsa exit 1
```

Yazdığı iki dosya: `workers/studio-mcp/src/tool-definitions.generated.ts` ve
`docs/OPS.generated.md`. İkisi de elle düzenlenmez.

## Ajan olarak sen ne yaparsın

**MCP bağlıysa hiçbir şey.** `tools/list` zaten canlı şemaları taşır ve o senin
gördüğün tek gerçektir. `SKILL.md` §3'teki tablolar yönlendirmedir; imza değil.

**MCP bağlı değilse ve imzaya bakman gerekiyorsa** `jorsby-studio-v2` reposunun
kökündeki `docs/OPS.generated.md`'yi oku. Bu klasörden göreli yol
`../../docs/OPS.generated.md`'dir — ama bu klasör `~/.claude/skills/` altına
symlink edilmiş olabileceği için göreli yola güvenme, repo kökünden git.

## Neden kopya yok

Elle kopyalanan bir imza tablosu ilk migration'da yalan söyler ve **yalanı
hiçbir test yakalamaz** — CI yeşil kalır, yalnız ajan olmayan bir alanı
doldurur. Bu, bu projedeki en pahalı sessiz hata sınıfı.

`SKILL.md` frontmatter'ındaki `op_surface` bunun tek uyarı fenerdir:
`registry.mjs`'in sha256'sının ilk 12 hanesi. Registry değişip bu hane
güncellenmediyse skill güncellenmemiş demektir ve **çelişkide registry kazanır.**

Bugünkü değer: `c91e46734683` · `schema_version: 2` · 33 op, 29'u açık (biri ajan yüzeyinde yok: accept_invite).
