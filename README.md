# openmaus-skills

Jorsby'nin kendi yazdığı OpenMaus skill'leri. Her klasör bir `SKILL.md` taşır; skill'in kaynağı burasıdır.

**Bota yüklemek:** OpenMaus → bot → Skills → şu URL'yi yapıştır:

```
https://github.com/jorsby/openmaus-skills/tree/main/skills/<skill-adi>
```

## Skill'ler

| Skill | Ne yapar | Kimde durur |
|---|---|---|
| `calisma-duzeni` | Serhat ile iletişim, onaylı işin planı, görev devri, sonuç raporu | CEO |
| `company-rules` | Şirket kuralları, roller, kapsam sınırları, skill kaynağı | Yöneticiler |
| `is-plani` | Onaylı işi adımlara ayırma, atama, doğrulanmış ilerleme | Yöneticiler |
| `plain-writing` | Sade, kısa, insan gibi yazım (her dilde) | Yöneticiler, creator'lar |
| `dev-flow` | Kod işi: kapsam, branch, doğrulama, PR, inceleme akışı | Geliştiriciler |
| `review-ilkeleri` | Kod/PR incelemesinde doğruluk, kapsam, kanıt ölçütleri | Geliştiriciler |

## Dışarıdan alınan skill'ler

Bunlar bu repoda **tutulmaz**; sahipleri güncelledikçe doğrudan kendi repolarından yüklenir.

| Skill | Yapıştırılacak URL | Kimde durur |
|---|---|---|
| `supabase` | https://github.com/supabase/agent-skills/tree/main/skills/supabase | Geliştiriciler |
| `wrangler` | https://github.com/cloudflare/skills/tree/main/skills/wrangler | Studio geliştiricileri |
| `workers-best-practices` | https://github.com/cloudflare/skills/tree/main/skills/workers-best-practices | Studio geliştiricileri |

Not: OpenMaus yalnız skill klasöründeki düz `.md` dosyalarını alır. `references/` ve `assets/` alt klasörleri gelmez; ayrıntı için botlar Cloudflare ve Supabase dokümantasyon araçlarını kullanır.

## Kurallar

- Skill düzenlemesi bu repoda yapılır: commit + push, sonra botta sil + yeniden yükle. OpenMaus aynı adla ikinci yüklemeyi kabul etmez.
- Klasör adı = skill adı: kebab-case, sürüm eki yok.
- Ürün bilgisi skill'e değil ürün repo'sunun `AGENTS.md` dosyasına yazılır.
