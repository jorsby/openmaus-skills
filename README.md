# openmaus-skills

Jorsby'ye özel üç OpenMaus skill'i. Her klasörde tek bir `SKILL.md` var; kaynak burasıdır, düzenleme elle yapılır.

**Bota yüklemek:** OpenMaus → bot → Skills → klasör URL'sini yapıştır.

```
https://github.com/jorsby/openmaus-skills/tree/main/skills/cinematic-video-production
https://github.com/jorsby/openmaus-skills/tree/main/skills/faceless-video-production
https://github.com/jorsby/openmaus-skills/tree/main/skills/jstudio
```

| Skill | Ne anlatır |
|---|---|
| `cinematic-video-production` | Sinematik video üretim hattı |
| `faceless-video-production` | Faceless video üretim hattı |
| `jstudio` | Jorsby Studio v2 ile proje kurma, üretim, inceleme, teslim (play'ler ve op listesi gömülü) |

OpenMaus yalnız `SKILL.md` dosyasını kurar; ek dosyalar gelmez. Bu yüzden referans metinleri dosyanın sonunda "Paket eki" başlıkları altında gömülüdür.

**Hazır skill'ler** bu repoda tutulmaz, sahibinin reposundan yüklenir. Örnek:

- Supabase: https://github.com/supabase/agent-skills/tree/main/skills/supabase
- Wrangler: https://github.com/cloudflare/skills/tree/main/skills/wrangler
- Workers best practices: https://github.com/cloudflare/skills/tree/main/skills/workers-best-practices

**Güncelleme:** burada düzenle, commit + push; botta eskisini sil, yeniden yükle. OpenMaus aynı adla ikinci yüklemeyi kabul etmez.
