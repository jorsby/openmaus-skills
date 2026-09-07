---
name: plain-writing
description: Plain, short, human prose in any language. Use for every draft, answer, rewrite or marketing text.
---

> OpenMaus içinde yönetilir. Köken: Multica studio; 6 Eylül 2026 dışa aktarımı.
> Yeni içeriği Skills ekranında inceleyip etkinleştir. Bu dosya eski kütüphaneden senkronize edilmez.
> Paket ekleri aşağıda “Paket eki” başlıklarında tam metindir. Metindeki göreli paket yolları bu bölümlere karşılık gelir. Yalnız ihtiyaç duyulan bölümü oku. Scriptler disk üzerinde kurulu değildir; çalıştırma gerekirse onaylanan iş kapsamında geçici dosyaya çıkarılıp doğrulanır.

<!-- v3.1 after blind round 5: lead-in stays only if what it announces follows; 'whether you're' kept when natural; contractions and everyday phrasing allowed. -->

Ornament costs the reader energy and hides empty claims. Write so a tired non-native reader gets it in one pass. If a claim is empty once the ornament is gone, keep it once in its plainest words; do not replace it with something better.

## Procedure
1. List every claim in the source, with its hedge ("arguably", "designed to", "often"), its count words ("several", "two") and its order words ("first and foremost", "main"). This list is the contract: nothing added, nothing dropped, no hedge stronger or weaker, priority kept.
2. Delete only sentences that carry no claim: greetings, praise of the question, hype around a claim, and announcements of steps that do not follow ("Let's break it down step by step" before plain prose -> delete). A closing claim stays. A lead-in to something that really follows keeps its plain form ("Let's dive in!" before a how-it-works section -> "Here's how it works." / "Şöyle çalışıyor:").
3. Write each claim as one plain sentence the way a person says it: everyday verb, contractions where natural, no intensifier, no metaphor, technical term kept as the source has it. A vague claim stays vague at the same strength ("elevate your workflow" -> "improve your workflow", never "speed up"; "AI-powered solutions" -> "AI tools"). Do not define, exemplify, or advise beyond the source.
4. Pick one addressee (you / they / we) and hold it to the end. If the source names a group ("creators") and also says "you", choose one and recast the other.
5. Write in the target language's own idiom, not the source's word order; if a phrase back-translates word for word, redo it. Vary sentence length; no run of same-length sentences, no fragments for effect.
6. Self-check against the list from step 1; fix, do not polish.

## Patterns
- Hype, superlatives: revolutionary, cutting-edge, unprecedented, game-changing, seamlessly -> the plain fact.
- Filler openers/closers: Great question, Let's dive in, In today's world -> start on the first claim, end on the last claim.
- "Not X but Y" -> state Y.
- Tricolons, padded triples -> as many items as there are.
- Em-dash asides, colon reveals -> comma, period, or "because".
- Rhetorical questions -> the answer.
- Cheerleading, exclamation marks -> a period.
- "Whether you're X or Y" -> keep it if it reads naturally ("whether you're a pro or just starting out"); do not recast it into a trailing "for X and Y alike".
- "Designed to / aims to X" -> "meant to X" or "X için yapıldı": keep the intent, do not assert the result.
- Importance puffery: it's worth noting, crucially, at its core, first and foremost -> the point itself.
- Connector padding: Moreover, Additionally -> nothing; First/Second only for a real list.
- Buzz verbs, abstract nouns: empower, leverage, elevate, unlock, is a necessity -> helps, use, changes, you need.

## Turkish equivalents
| Slop | Write |
|---|---|
| Harika bir soru! / Hadi gelin ... göz atalım! | (sil; ilk iddiayla başla) |
| artık bir tercih değil, bir zorunluluk | artık şart / ... gerekiyor |
| devrim niteliğinde, benzeri görülmemiş, oyunun kurallarını değiştiren, kökten | (sil) |
| kesinlikle, son derece, kritik öneme sahip, belirtmekte fayda var | (sil) |
| İşte nasıl çalıştığı / sizin için çalışır | Şöyle çalışıyor / Nasıl çalıştığını anlatalım |
| ister ... ister ... olun | -sanız da -sanız da / ... için de ... için de |
| ödünleşim | artısı ve eksisi |
| yararlanmak, bambaşka bir seviyeye taşımak | kullanmak, hızlandırmak |
| ... olarak kabul edilir | ... sayılır |
| -ilmektedir / -masıdır zincirleri, ders kitabı -dır/-dir | çekimli fiil, konuşma dili |
| performanslı | hızlı çalışan |
| X konusunu düşünmek | X'i düşünmek |

## Self-check
- Same claims as the source, same count and order words, hedges at original strength, closer present in plain form?
- Nothing added: no definition, example, number, advice, or specificity the source lacks?
- One addressee throughout; no verb or content word repeated in consecutive sentences ("you need... you need"); no exclamation marks, em dashes, "not X but Y"?
- Read aloud in the target language: would a colleague say this across a desk?


## Paket eki: `eval/README.md`

Kaynak SHA256: `793076702d5a2526f609503a40ddabe7c12fedcbb130b36a97ad3b0575cd4096` · UTF-8 boyutu: 1810 bayt.

````text
# plain-writing — kör değerlendirme tezgâhı

`blind-eval-workflow.js`: Claude Code Workflow scripti. Koşullar (skill dosyaları + skill'siz kontrol) × N koşu → 4 örnek metni yeniden yazar → 6 kör hakem (farklı lensler) 1-10 puanlar → tür ortalaması.

Sonuç geçmişi (18-19 Ağu 2026, Fable 5):
- 20 dış skill (skills.sh + GitHub + web + Wikipedia/Orwell/PG/plainlanguage.gov): en iyisi kontrolü +1.0 geçti, 11'i kontrolün altında. petergyang/no-ai-slop (6K kurulum) kontrolün altında.
- v1 (601 kelime, çok kural) ≈ kontrol. Kurallar uydurmaya itti (gloss, yarı uzunluk, kapanışı değiştir).
- v2 (az kural, sert sadakat): tur 4'te kontrol +1.3; tur 5'te −0.25. Gürültü ±0.7.
- v3: "whether you're → for X and Y" ve "kapanışı koru" kuralları geri tepti: tur 6'da kontrol −1.75.
- v3.1 (kurulu): tur 6'da 8.08 vs kontrol 7.89 (×6, 6 hakem). Kalan itirazlar kelime zevki.

Ders: kötü bir kural 2 puan götürür; iyi kural seti kontrole ± döner. Sonraki kaldıraç = beğenilen gerçek metin örnekleri (few-shot), kural değil.
Hakem heyeti turdan tura ±0.7 kayar; sadece aynı tur içindeki sıra anlamlı. Tek koşuya güvenme, ≥4 koşu.

## Bilinen sınır (19 Ağu, faceless-video denemesi)
Kendi SKILL.md'mize uygulandı: 1279→1251 kelime (−%2), iş ton düzeltmesi oldu (edilgen→emir, tek muhatap, tire→virgül). İddia defteri: 61 madde, kod/sayı birebir, eklenen 0, **v2 güvenli** — ama "not X but Y → Y" kuralı talimat metninde 5 yerde X'i sildi ve X çoğunda *yanlış varsayımın adı*ydı ("önceki kare değil, sabit sheet'ler taşır"). Pazarlamada X boş retorik; öğreti metninde X değerli.
Sonraki düzeltme adayı (TEST EDİLMEDİ, SKILL.md'ye henüz girmedi): "X yaygın bir yanlışı adlandırıyorsa X'i tut."

````


## Paket eki: `eval/blind-eval-workflow.js`

Kaynak SHA256: `8466eb5c5ea6c8428addce4497d888c644d3d9ee0fe914cbde58ae7e5b75a467` · UTF-8 boyutu: 8359 bayt.

````text
export const meta = {
  name: 'antislop-round6',
  description: 'plain-writing (kurulu SKILL.md) vs kontrol; ×6 koşu, 6 kör hakem',
  phases: [
    { title: 'Rewrite', detail: 'skill×6, kontrol×6' },
    { title: 'Judge', detail: '6 kör hakem' },
  ],
}
const LAB = '/Users/serhatcamici/Development/skill-lab/writing'
const SAMPLES = {
  en_marketing: `In today's fast-paced digital landscape, leveraging cutting-edge AI-powered solutions is not just an option—it's a necessity. Our revolutionary platform seamlessly empowers creators to unlock unprecedented levels of productivity, transforming the way content is crafted. Whether you're a seasoned professional or just starting your journey, this game-changing tool is designed to elevate your workflow to new heights. Let's dive in and explore how it works!`,
  en_explainer: `Great question! Understanding how caching works is absolutely crucial for building performant applications. Let's break it down step by step. At its core, caching is all about storing frequently accessed data in a faster storage layer. This might seem simple, but there are several key considerations to keep in mind. First and foremost, you need to think about cache invalidation—arguably one of the hardest problems in computer science. Additionally, it's worth noting that different caching strategies have different trade-offs. In summary, caching is a powerful tool, but it's important to use it wisely!`,
  tr_marketing: `Günümüzün hızla değişen dijital dünyasında, yapay zekâ destekli çözümlerden yararlanmak artık bir tercih değil, bir zorunluluk. Devrim niteliğindeki platformumuz, içerik üreticilerinin verimliliklerini benzeri görülmemiş seviyelere taşımalarını sağlıyor ve içerik üretme biçimini kökten dönüştürüyor. İster deneyimli bir profesyonel olun ister yolculuğunuza yeni başlıyor olun, bu oyunun kurallarını değiştiren araç, iş akışınızı bambaşka bir seviyeye taşımak için tasarlandı. Hadi gelin, nasıl çalıştığına birlikte göz atalım!`,
  tr_explainer: `Harika bir soru! Önbellekleme mantığını anlamak, performanslı uygulamalar geliştirmek için kesinlikle kritik öneme sahip. Gelin adım adım inceleyelim. Özünde önbellekleme, sık erişilen verinin daha hızlı bir katmanda saklanmasıdır. Kulağa basit gelebilir; ancak göz önünde bulundurulması gereken birkaç önemli nokta var. Her şeyden önce, önbellek geçersiz kılma konusunu düşünmeniz gerekiyor — ki bu, bilgisayar bilimlerinin en zor problemlerinden biri olarak kabul edilir. Ayrıca, farklı önbellekleme stratejilerinin farklı ödünleşimleri olduğunu belirtmekte fayda var. Özetle, önbellekleme güçlü bir araçtır; ancak akıllıca kullanılması son derece önemlidir!`,
}
const SAMPLE_BLOCK = Object.entries(SAMPLES).map(([k,v]) => `[${k}]\n${v}`).join('\n\n')

const RW_SCHEMA = { type: 'object', required: ['en_marketing','en_explainer','tr_marketing','tr_explainer'],
  properties: { en_marketing:{type:'string'}, en_explainer:{type:'string'}, tr_marketing:{type:'string'}, tr_explainer:{type:'string'} } }

const JUDGE_SCHEMA = {
  type: 'object', required: ['ranking','notes'],
  properties: {
    ranking: { type: 'array', items: { type: 'object', required: ['label','score','note'], properties: { label:{type:'string'}, score:{type:'integer',minimum:1,maximum:10}, note:{type:'string', description:'what still keeps it from 10, concretely'} } } },
    notes: { type: 'string', description: 'cross-cutting: which labels look like the same method? what did the best do that others did not?' },
  },
}

const conditions = [
  { id: 'OURS-v31#1', kind: 'v31', path: `${LAB}/_ours/plain-writing/SKILL.md` },
  { id: 'OURS-v31#2', kind: 'v31', path: `${LAB}/_ours/plain-writing/SKILL.md` },
  { id: 'OURS-v31#3', kind: 'v31', path: `${LAB}/_ours/plain-writing/SKILL.md` },
  { id: 'OURS-v31#4', kind: 'v31', path: `${LAB}/_ours/plain-writing/SKILL.md` },
  { id: 'OURS-v31#5', kind: 'v31', path: `${LAB}/_ours/plain-writing/SKILL.md` },
  { id: 'OURS-v31#6', kind: 'v31', path: `${LAB}/_ours/plain-writing/SKILL.md` },
  { id: 'BASELINE#1', kind: 'baseline' },
  { id: 'BASELINE#2', kind: 'baseline' },
  { id: 'BASELINE#3', kind: 'baseline' },
  { id: 'BASELINE#4', kind: 'baseline' },
  { id: 'BASELINE#5', kind: 'baseline' },
  { id: 'BASELINE#6', kind: 'baseline' },
]

phase('Rewrite')
const rewrites = await parallel(conditions.map((c, i) => () => {
  let p
  if (c.kind === 'baseline') {
    p = `Rewrite the four texts below to be plain, short and human. Keep the meaning, add no facts, cut hype and filler. Return rewrites only (pure text, no notes). Run ${i}.\n\n${SAMPLE_BLOCK}`
  } else {
    p = `Read this writing skill fully: ${c.path} (cat it; follow any references it names). Then apply it STRICTLY to rewrite the four texts below. Keep the meaning, add no facts. Return rewrites only (pure text, no notes, no "what changed"). Run ${i}.\n\n${SAMPLE_BLOCK}`
  }
  return agent(p, { label: `rw:${c.id}`, phase: 'Rewrite', schema: RW_SCHEMA }).then(r => r ? ({ ...c, rewrites: r }) : null)
}))
const ok = rewrites.filter(Boolean)
log(`Rewrite: ${ok.length}/${conditions.length}`)

phase('Judge')
const labels = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'.split('')
const rotated = ok.map((e, i) => ({ e, key: (i * 7 + 4) % ok.length })).sort((a, b) => a.key - b.key).map(x => x.e)
const blind = rotated.map((e, i) => ({ label: labels[i], e }))
const blk = (f) => blind.map(b => `--- ${b.label} ---\n${b.e.rewrites[f]}`).join('\n\n')
const lenses = [
  'a plain-language editor (plainlanguage.gov / GOV.UK school): shortest clear sentence wins, no ornament',
  'a native Turkish copy editor: judge mainly the Turkish rewrites — do they read like a Turkish person wrote them, or like translated English? calques and coined words are heavy penalties',
  'a skeptical reader who hates marketing: which rewrite would you actually trust and keep reading?',
  'a senior engineer reading docs/chat answers: judge mainly the explainer rewrites — every source claim intact at its original hedge strength, nothing padded, nothing invented?',
  'a translation QA reviewer doing a claim-by-claim ledger: for each rewrite, count claims added, dropped, or changed in strength (including count words like several/two and closers) — each such change costs at least 1 point',
  'a reader who cares about rhythm and voice: does it sound like one person talking, with natural sentence variety, one consistent addressee, no machine cadence, no fragments for effect?',
]
const judgements = await parallel(lenses.map((lens, i) => () =>
  agent(`You are ${lens}. Below are anonymous rewrites of four AI-generic sample texts; each label is one method (some labels may be the same method run more than once — judge each on its own). Score every label 1-10: plain, short, human-sounding, meaning intact, hedges preserved, no invented facts, no leftover AI tells, one consistent addressee, natural rhythm. Be harsh: 10 = you would not change a word. For each label say concretely what still keeps it from 10.

ORIGINALS:
${SAMPLE_BLOCK}

REWRITES en_marketing:
${blk('en_marketing')}

REWRITES en_explainer:
${blk('en_explainer')}

REWRITES tr_marketing:
${blk('tr_marketing')}

REWRITES tr_explainer:
${blk('tr_explainer')}`, { label: `judge:${i+1}`, phase: 'Judge', schema: JUDGE_SCHEMA })
))
const oj = judgements.filter(Boolean)
const agg = {}
for (const b of blind) agg[b.label] = { id: b.e.id, kind: b.e.kind, scores: [], notes: [] }
for (const j of oj) for (const r of (j.ranking||[])) if (agg[r.label]) { agg[r.label].scores.push(r.score); agg[r.label].notes.push(r.note) }
const table = Object.entries(agg).map(([label, v]) => ({ label, ...v, avg: v.scores.length ? +(v.scores.reduce((a,b)=>a+b,0)/v.scores.length).toFixed(2) : null })).sort((a,b)=>(b.avg||0)-(a.avg||0))
const mean = (arr) => arr.length ? +(arr.reduce((a,b)=>a+b,0)/arr.length).toFixed(2) : null
const byKind = {}
for (const t of table) { (byKind[t.kind] ||= []).push(t.avg) }
const summary = Object.fromEntries(Object.entries(byKind).map(([k, v]) => [k, { mean: mean(v), runs: v }]))
log('Kind means: ' + JSON.stringify(summary))
return { table, summary, judge_notes: oj.map(j => j.notes), sample_rewrites: ok.filter(e => e.kind==='v31').slice(0,2).map(e => e.rewrites) }
````
