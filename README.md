# /watch

**DÃª ao Claude a capacidade de assistir qualquer vÃ­deo.**

> Cole uma URL ou um arquivo local e o Claude *assiste* â€” **extraÃ§Ã£o de frames por corte de cena** (um frame por corte, nÃ£o a cada N segundos), um **microscÃ³pio do hook 0-10s** (frames densos + Whisper word-level na abertura, onde todo vÃ­deo ganha ou perde sua atenÃ§Ã£o), **mÃ©tricas editoriais de pacing com motion por shot** e **auto-save opcional no Obsidian**, pra um vÃ­deo assistido virar uma entrada conectada no wiki sem copia-e-cola.

Claude Code:
```
/plugin marketplace add inematds/claude-watch
/plugin install watch@claude-watch
```

claude.ai (web): [baixe o `watch.skill`](https://github.com/inematds/claude-watch/releases/latest) e solte em Settings â†’ Capabilities â†’ Skills.

Codex / skills genÃ©ricas:
```bash
git clone https://github.com/inematds/claude-watch.git ~/.codex/skills/watch
```

Zero config pra comeÃ§ar â€” `yt-dlp` e `ffmpeg` se instalam no primeiro run via `brew` no macOS (Linux/Windows imprimem os comandos exatos). Captions cobrem a maioria dos vÃ­deos pÃºblicos de graÃ§a. Chave da API Whisper sÃ³ Ã© necessÃ¡ria quando o vÃ­deo nÃ£o tem legenda. Aponte `$WATCH_VAULT_DIR` pro seu vault Obsidian pra ligar o auto-save, ou deixe sem setar que a skill pula a etapa de ingest em silÃªncio.

## O que tem dentro

- **ExtraÃ§Ã£o de frames por corte de cena** â€” `scripts/frames.py` pega um frame por shot detectado via `select=gt(scene,...)` do ffmpeg, nÃ£o um tick uniforme a cada N segundos. O custo de token fica plano em vÃ­deo longo porque o nÃºmero de frames Ã© limitado pela quantidade de cortes, nÃ£o pela duraÃ§Ã£o.
- **MicroscÃ³pio do hook 0-10s** â€” `scripts/hook.py` roda uma passada mais densa a 2 fps nos primeiros 10 segundos + um transcript Whisper word-level, pro relatÃ³rio dizer o que estava na tela *conforme cada palavra caÃ­a*. Os 10 primeiros segundos sÃ£o onde todo vÃ­deo ganha ou perde sua atenÃ§Ã£o.
- **MÃ©tricas editoriais de pacing, motion e cÃ¢mera** â€” `scripts/pacing.py` calcula cortes/min, duraÃ§Ã£o mÃ©dia/mediana de shot, **motion por shot via ffmpeg `signalstats`** (delta de luma YDIF) e **movimento de cÃ¢mera por shot** (pan / tilt / zoom / estÃ¡tico / handheld via ffmpeg `vidstabdetect`) â€” tudo sem opencv. O shot mais agitado tambÃ©m guia a escolha das hero frames. DÃ¡ pra raciocinar sobre ritmo como um editor faz.
- **`report.md` estruturado com marcadores de preenchimento** â€” `scripts/report.py` emite um relatÃ³rio de esquema fixo (TL;DR, momentos-chave, breakdown do hook, perfil editorial, citaÃ§Ãµes, entidades, conceitos, transcript) onde as seÃ§Ãµes narrativas sÃ£o marcadores explÃ­citos `<!-- pending Claude fill: ... -->`. O Claude tem uma lista de tarefas pra percorrer antes do ingest, nÃ£o um doc em branco.
- **Auto-save opcional no Obsidian** â€” o Step 4.4 monta o relatÃ³rio em `$VAULT_DIR/raw/watched/<slug>/` e abre via URL scheme `obsidian://`. O Step 4.5 oferece o ingest no wiki do vault. Os dois passos pulam limpo quando nenhum vault Ã© detectado. O caminho do vault vem de `$WATCH_VAULT_DIR` ou Ã© auto-detectado em `~/Second brain/`, `~/Documents/Obsidian/`, `~/Obsidian/`.

O pipeline-base â€” download yt-dlp, frames ffmpeg, backends Groq/OpenAI Whisper, o modo focado `--start`/`--end`, o hook de SessionStart e a instalaÃ§Ã£o multi-superfÃ­cie â€” vem do projeto original `claude-video` e funciona sem mudanÃ§a (ver [CrÃ©ditos](#crÃ©ditos)).

---

O Claude lÃª uma pÃ¡gina, roda um script, navega um repo. O que ele nÃ£o faz, de fÃ¡brica, Ã© *assistir um vÃ­deo*. VocÃª cola um link do YouTube e ele tem que adivinhar pelo tÃ­tulo ou puxar um transcript que perde 90% do que estÃ¡ na tela.

Com o `/watch` vocÃª cola uma URL ou um caminho local, faz uma pergunta, e o Claude baixa o vÃ­deo, extrai frames numa taxa auto-escalada, puxa um transcript com timestamp (captions grÃ¡tis quando hÃ¡, Whisper como fallback) e `Read` cada frame como imagem. Na hora de responder, ele *viu* o vÃ­deo e *ouviu* o Ã¡udio.

```
/watch https://youtu.be/dQw4w9WgXcQ o que acontece aos 30 segundos?
```

## Por que isso existe

ConstruÃ­ isso porque vivo usando vÃ­deo pra acompanhar conteÃºdo. Se vejo um vÃ­deo do YouTube bombando, quero saber como o criador estruturou o gancho â€” o que estÃ¡ na tela nos 3 primeiros segundos, o que ele falou, por que funcionou. Isso significava assistir na mÃ£o com um bloco de notas. Agora eu sÃ³ colo a URL e pergunto.

A outra metade Ã© resumo. A maioria dos vÃ­deos nÃ£o merece 20 minutos da minha atenÃ§Ã£o. Eu passo a URL pro Claude, ele puxa o transcript e me diz o que de fato aconteceu. Se o visual importa, os frames vÃªm junto. Se Ã© um podcast ou talking head, o transcript basta.

O Claude Ã© Ã³timo em ler e sintetizar â€” mas atÃ© agora vÃ­deo era o Ãºnico input que eu nÃ£o conseguia passar pra ele. Colar um link do YouTube nÃ£o dava nada Ãºtil. O `/watch` fecha essa lacuna.

## Pra que as pessoas usam

**Analisar conteÃºdo de outra pessoa.** `/watch https://youtu.be/<video-viral> que gancho eles abriram?` O Claude olha os primeiros frames, lÃª o transcript de abertura, quebra a estrutura. O mesmo pra ad creative, lanÃ§amento de concorrente, intro de podcast â€” qualquer coisa onde o *como* importa tanto quanto o *quÃª*.

**Diagnosticar um bug por vÃ­deo.** Te mandam um screen recording de algo quebrado. `/watch bug-repro.mov o que estÃ¡ dando errado?` O Claude assiste a gravaÃ§Ã£o, acha o frame onde o problema aparece, descreve o que estÃ¡ na tela e muitas vezes pega a causa sem vocÃª nunca abrir o arquivo.

**Resumir um vÃ­deo.** `/watch https://youtu.be/<coisa-longa> resume isso` faz o Ã³bvio â€” puxa a estrutura, os momentos-chave, o que foi realmente dito e mostrado. Mais rÃ¡pido que assistir em 2x.

## Como funciona

1. **VocÃª cola um vÃ­deo e uma pergunta.** URL (tudo que o yt-dlp suporta â€” YouTube, Loom, TikTok, X, Instagram, e mais umas centenas) ou um caminho local (`.mp4`, `.mov`, `.mkv`, `.webm`).
2. **O `yt-dlp` baixa.** Pra URLs, num diretÃ³rio de trabalho temporÃ¡rio. Pra arquivos locais, sem download â€” sÃ³ Ã© sondado no lugar.
3. **O `ffmpeg` extrai frames numa taxa auto-escalada.** O orÃ§amento de frames Ã© consciente da duraÃ§Ã£o: â‰¤30s pega ~30 frames, 30-60s pega ~40, 1-3min pega ~60, 3-10min pega ~80, mais longo pega 100 esparso. Tetos rÃ­gidos: 2 fps, 100 frames. JPEGs a 512px de largura por padrÃ£o â€” sobe com `--resolution 1024` se o Claude precisar ler texto na tela.
4. **O transcript vem de um de dois lugares.** Primeira tentativa: `yt-dlp` puxa captions nativos (manual ou auto-gerados) da fonte. GrÃ¡tis, instantÃ¢neo, razoavelmente preciso. Fallback: extrai um clipe de Ã¡udio mono 16 kHz e manda pro Whisper â€” `whisper-large-v3` da Groq (preferido â€” mais barato e rÃ¡pido) ou `whisper-1` da OpenAI.
5. **Frames + transcript sÃ£o entregues ao Claude.** O script imprime os caminhos dos frames com marcadores `t=MM:SS` e o transcript com timestamps. O Claude `Read` cada frame em paralelo â€” JPEGs renderizam direto como imagem no contexto dele.
6. **O Claude responde fundamentado no que estÃ¡ de fato na tela e no Ã¡udio.** NÃ£o "com base na descriÃ§Ã£o" ou "segundo o tÃ­tulo". Ele viu os frames. Ouviu o transcript. Responde como quem assistiu o vÃ­deo.
7. **Limpeza.** O script imprime um diretÃ³rio de trabalho no fim. Se vocÃª nÃ£o vai fazer follow-up, o Claude remove.

## O objetivo final: o vÃ­deo vira uma nota conectada

O `/watch` nÃ£o para na resposta. O objetivo de verdade Ã© transformar o vÃ­deo numa **nota estruturada e conectada no seu "Second Brain" do Obsidian** â€” enquadrada pelo *porquÃª* vocÃª assistiu (o `--intent`). Ele grava em duas camadas:

**1. O artefato bruto â€” `report.md`** (Step 4.4), montado em `raw/watched/<slug>/` junto com as **hero frames** escolhidas. Esquema fixo: frontmatter (source, title, duration, watched_at, intent, hero_frames, transcript_source), **TL;DR** pela lente do seu intent, **momentos-chave**, o **microscÃ³pio do hook 0-10s** (frame-a-frame + transcript word-level + o padrÃ£o de gancho), **perfil editorial** (cortes/min, duraÃ§Ã£o de shot, motion por shot, movimento de cÃ¢mera, fingerprint de estilo), **citaÃ§Ãµes**, **entidades** (pessoas / empresas / ferramentas como `[[wikilinks]]`), **conceitos** e o **transcript** completo.

**2. IngestÃ£o no wiki** (Step 4.5, sÃ³ com o seu consentimento). Se o vault tem um `CLAUDE.md` com uma Ingest op, ela roda contra o relatÃ³rio e grava/atualiza:
- `wiki/entities/` â€” pÃ¡ginas das pessoas, empresas e ferramentas mencionadas
- `wiki/concepts/` â€” os frameworks e modelos mentais que apareceram
- `wiki/sources/` â€” a pÃ¡gina do prÃ³prio vÃ­deo com TL;DR + citaÃ§Ãµes
- uma linha no `log.md`

Sem um `CLAUDE.md` no vault, ele cai num ingest genÃ©rico: a linha no `log.md` mais o relatÃ³rio montado.

**O objetivo unificador:** o vÃ­deo vira um nÃ³ de primeira classe no seu grafo de conhecimento â€” as pessoas, ferramentas e conceitos dele ligados via `[[wikilink]]` Ã s notas que vocÃª jÃ¡ tem, enquadrados pelo seu motivo de assistir. Detalhe: o script *gera* o `report.md` sozinho, mas o wiki sÃ³ Ã© populado se o seu vault definir a Ingest op â€” a skill delega esse passo ao contrato do seu Second Brain, em vez de trazer um pronto.

## Ler o estilo cinematogrÃ¡fico (perfil editorial)

Um vÃ­deo curto â€” de atÃ© ~2-3 minutos â€” Ã© o ponto ideal pra "ler o estilo": o orÃ§amento dÃ¡ ~60-80 frames (um por corte), o transcript inteiro e o perfil editorial completo. O que sai Ã© um **brief de estilo**, parte **medido**, parte **inferido**:

**Medido (nÃºmeros reais, sem opencv):**
- **Ritmo** â€” cortes/min + duraÃ§Ã£o mÃ©dia/mediana de shot. Ã‰ o fingerprint de pacing: "cortes secos, shot mÃ©dio 1,8s" (estilo MTV/Fireship) vs "shots de 8s" (contemplativo).
- **Motion** â€” `signalstats` por shot: quais planos sÃ£o agitados vs parados.
- **CÃ¢mera** â€” rÃ³tulo por shot (pan / tilt / zoom / estÃ¡tico / handheld) + o movimento dominante, via `vidstabdetect`.

**Inferido pelo Claude (lendo os frames):**
- Paleta / look de cor, enquadramento e composiÃ§Ã£o (close vs wide, simetria), iluminaÃ§Ã£o (high/low-key), uso de texto na tela, transiÃ§Ãµes e lente aparente (grande-angular vs tele).
- Um **fingerprint de estilo de uma linha** (jÃ¡ Ã© uma seÃ§Ã£o do `report.md`).

> **Densidade:** o motion e o classificador de cÃ¢mera amostram o vÃ­deo **inteiro** (4-8 fps), nÃ£o sÃ³ os frames que o Claude vÃª. Movimento *dentro* de um shot Ã© capturado mesmo o Claude vendo um frame por plano.

**Limites honestos:**
- Grading/LUT, lente, profundidade de campo e temperatura de cor sÃ£o **inferidos dos frames, nÃ£o medidos**.
- O *sentido* do movimento (esquerda/direita) Ã© heurÃ­stico â€” confiÃ¡vel sÃ³ pro **eixo** (pan vs tilt vs zoom vs handheld vs estÃ¡tico).
- Ãudio: sÃ³ a fala (transcript). NÃ£o analisa trilha/mÃºsica/mixagem.

Como rodar â€” o `--intent` molda o report pra esse eixo:
```
/watch seu-video.mp4 estilo cinematogrÃ¡fico e linguagem de cÃ¢mera
```

O brief resultante (pacing + cÃ¢mera + look + fingerprint) serve de direÃ§Ã£o pronta pra ferramentas de geraÃ§Ã£o/motion de vÃ­deo.

**Exemplo real:** [`examples/avatar-reel-cinematic-style.md`](examples/avatar-reel-cinematic-style.md) â€” diagnÃ³stico de estilo de um reel do Avatar, saÃ­da de uma execuÃ§Ã£o real do `/watch`.

## OrÃ§amento de frames â€” por que importa

O custo de token Ã© dominado pelos frames. Cada frame Ã© uma imagem; tokens de imagem somam rÃ¡pido. A lÃ³gica de auto-fps existe pra vocÃª nÃ£o estourar o orÃ§amento de contexto num scan esparso de um vÃ­deo de 30 minutos que teria sido melhor respondido por uma janela focada de 30 segundos.

| DuraÃ§Ã£o | OrÃ§amento de frames padrÃ£o | O que vocÃª ganha |
|---------|----------------------------|------------------|
| â‰¤30 s | ~30 frames | Denso â€” basicamente todo momento-chave |
| 30 s - 1 min | ~40 frames | Ainda denso |
| 1 - 3 min | ~60 frames | ConfortÃ¡vel |
| 3 - 10 min | ~80 frames | Esparso mas utilizÃ¡vel |
| > 10 min | 100 frames | Aviso de "scan esparso" â€” re-rode focado |

Quando o usuÃ¡rio nomeia um momento ("por volta de 2:30", "os Ãºltimos 30 segundos", "de 0:45 a 1:00"), passe `--start` / `--end`. O modo focado usa orÃ§amentos por segundo mais densos, limitados a 2 fps. Bem mais Ãºtil que uma passada esparsa pelo vÃ­deo inteiro.

## Instalar

| SuperfÃ­cie | InstalaÃ§Ã£o |
|------------|------------|
| **Claude Code** | `/plugin marketplace add inematds/claude-watch` e depois `/plugin install watch@claude-watch` |
| **claude.ai** (web) | [Baixe o `watch.skill`](https://github.com/inematds/claude-watch/releases/latest) â†’ Settings â†’ Capabilities â†’ Skills â†’ `+` |
| **Codex** | `git clone https://github.com/inematds/claude-watch.git ~/.codex/skills/watch` |
| **Manual / dev** | `git clone https://github.com/inematds/claude-watch.git ~/.claude/skills/watch` |
| **ConfiguraÃ§Ã£o** | Opcional: `export WATCH_VAULT_DIR=/caminho/do/seu/vault/obsidian` pra ligar o auto-save. Auto-detecta `~/Second brain/`, `~/Documents/Obsidian/`, `~/Obsidian/`. |

### Claude Code

```
/plugin marketplace add inematds/claude-watch
/plugin install watch@claude-watch
```

Atualize depois com `/plugin update watch@claude-watch`.

### claude.ai (web)

1. [Baixe o `watch.skill`](https://github.com/inematds/claude-watch/releases/latest) do Ãºltimo release.
2. VÃ¡ em Settings â†’ Capabilities â†’ Skills.
3. Clique `+` e solte o arquivo.

Ative "Code execution and file creation" em Capabilities antes â€” a skill chama `ffmpeg` e `yt-dlp`, entÃ£o nÃ£o roda sem isso.

### Codex

```bash
git clone https://github.com/inematds/claude-watch.git ~/.codex/skills/watch
```

### Manual (desenvolvedor)

```bash
git clone https://github.com/inematds/claude-watch.git ~/.claude/skills/watch
```

## Primeiro run

Na primeira chamada `/watch`, a skill roda `scripts/setup.py --check`. Se `ffmpeg` / `yt-dlp` nÃ£o estiverem no PATH, ou nenhuma chave Whisper estiver setada, ela te guia pra resolver:

- **macOS** â€” auto-roda `brew install ffmpeg yt-dlp`.
- **Linux** â€” imprime os comandos exatos `apt` / `dnf` / `pipx`.
- **Windows** â€” imprime os comandos `winget` / `pip`.
- **Chave de API** â€” cria `~/.config/watch/.env` (modo `0600`) com placeholders comentados pra `GROQ_API_KEY` (preferido) e `OPENAI_API_KEY`.

Depois do setup, o preflight Ã© silencioso e o `/watch` simplesmente funciona. O check Ã© um lookup de sub-100ms, entÃ£o nÃ£o te atrasa nos runs seguintes.

## Use suas prÃ³prias chaves

Captions cobrem a maioria dos vÃ­deos pÃºblicos de graÃ§a. O fallback Whisper sÃ³ entra quando o vÃ­deo genuinamente nÃ£o tem trilha de legenda â€” tipicamente arquivos locais, TikToks, alguns Vimeos e o eventual upload de YouTube sem legenda.

| Capacidade | O que vocÃª precisa | Custo |
|------------|--------------------|-------|
| Download + captions nativos | `yt-dlp` + `ffmpeg` | GrÃ¡tis |
| Fallback Whisper (preferido) | [Chave Groq](https://console.groq.com/keys) â€” `whisper-large-v3` | Barato, rÃ¡pido |
| Fallback Whisper (alt) | [Chave OpenAI](https://platform.openai.com/api-keys) â€” `whisper-1` | PreÃ§o padrÃ£o |
| Desligar o Whisper de vez | `--no-whisper` | GrÃ¡tis, sÃ³ frames quando nÃ£o hÃ¡ captions |

## Uso

```
/watch https://youtu.be/dQw4w9WgXcQ o que acontece aos 30 segundos?
/watch https://www.tiktok.com/@user/video/123 resume isso
/watch ~/Movies/screen-recording.mp4 quando a UI quebra?
/watch https://vimeo.com/123 que ferramentas ela menciona?
```

Focado numa seÃ§Ã£o especÃ­fica â€” orÃ§amento de frames mais denso, menos token:
```
/watch https://youtu.be/abc --start 2:15 --end 2:45
/watch video.mp4 --start 50 --end 60
/watch "$URL" --start 1:12:00            # de 1h12m atÃ© o fim
```

Outros botÃµes (passados pro `scripts/watch.py`):

- `--max-frames N` â€” baixa o teto de frames pra um orÃ§amento de token mais apertado.
- `--resolution W` â€” sobe a largura do frame pra 1024 px quando o Claude precisa ler texto na tela (slides, terminais, cÃ³digo).
- `--fps F` â€” sobrescreve o cÃ¡lculo de auto-fps (ainda limitado a 2 fps).
- `--whisper groq|openai` â€” forÃ§a um backend Whisper especÃ­fico.
- `--no-whisper` â€” desliga a transcriÃ§Ã£o de vez; sÃ³ frames.
- `--out-dir DIR` â€” guarda os arquivos de trabalho num lugar especÃ­fico (padrÃ£o: tmp auto-gerado).

## Limites

- **Melhor acurÃ¡cia: abaixo de 10 minutos.** Acima disso o script imprime um aviso de "scan esparso" â€” re-rode focado na parte que importa com `--start`/`--end`.
- **Tetos rÃ­gidos: 2 fps, 100 frames.** A contagem de frames dirige o custo de token; o script forÃ§a isso mesmo quando a conta de auto-fps implicaria mais.
- **Limite de upload do Whisper: 25 MB.** A mono 16 kHz isso dÃ¡ uns 50 minutos de Ã¡udio. VÃ­deos maiores precisam de captions ou de `--start`/`--end` pra uma janela menor.
- **Sem plataformas privadas.** A skill nÃ£o loga em nada. SÃ³ URLs pÃºblicas e arquivos locais. Se o yt-dlp nÃ£o alcanÃ§a sem auth, o `/watch` tambÃ©m nÃ£o.

## Estrutura

```
.
â”œâ”€â”€ SKILL.md                 # contrato da skill â€” carregado pelas trÃªs superfÃ­cies
â”œâ”€â”€ scripts/
â”‚   â”œâ”€â”€ watch.py             # entry point â€” orquestra download â†’ frames â†’ transcript
â”‚   â”œâ”€â”€ download.py          # wrapper do yt-dlp
â”‚   â”œâ”€â”€ frames.py            # frames ffmpeg + auto-fps + motion (signalstats) + cÃ¢mera (vidstabdetect)
â”‚   â”œâ”€â”€ pacing.py            # mÃ©tricas editoriais (cortes/min, shot length, motion, movimento de cÃ¢mera)
â”‚   â”œâ”€â”€ hook.py              # microscÃ³pio do hook 0-10s
â”‚   â”œâ”€â”€ report.py            # emissor do report.md estruturado
â”‚   â”œâ”€â”€ transcribe.py        # parse de VTT + dedupe + orquestraÃ§Ã£o Whisper
â”‚   â”œâ”€â”€ whisper.py           # clients Groq / OpenAI (pure stdlib)
â”‚   â”œâ”€â”€ setup.py             # preflight + instalador
â”‚   â””â”€â”€ build-skill.sh       # monta dist/watch.skill pro upload no claude.ai
â”œâ”€â”€ hooks/                   # hook de status no SessionStart (sÃ³ Claude Code)
â”œâ”€â”€ .claude-plugin/          # plugin.json + marketplace.json (Claude Code)
â”œâ”€â”€ .codex-plugin/           # empacotamento codex
â””â”€â”€ .github/workflows/       # release.yml â€” auto-monta watch.skill no push de tag
```

## Desenvolver

```bash
# Monta o bundle de upload do claude.ai:
bash scripts/build-skill.sh      # â†’ dist/watch.skill

# Roda os testes (stdlib unittest, sem dependÃªncia de pytest):
python3 -m pytest scripts/tests/ -q   # ou: python3 -m unittest discover scripts/tests
```

Releasing: tag `vX.Y.Z`, push da tag. O workflow monta `dist/watch.skill` e anexa ao release do GitHub.

Veja o [CHANGELOG.md](CHANGELOG.md) pro histÃ³rico de versÃµes.

## CrÃ©ditos

O `/watch` Ã© construÃ­do sobre o **[claude-video](https://github.com/bradautomates/claude-video)** do **[Bradley Bonanno](https://github.com/bradautomates)**. A skill `/watch` original â€” o download yt-dlp, o pipeline ffmpeg, os backends Groq/OpenAI Whisper, o fluxo de instalaÃ§Ã£o e o hook de SessionStart â€” Ã© trabalho dele, lanÃ§ado sob a licenÃ§a MIT. Este repo estende com extraÃ§Ã£o de frames por corte de cena, o microscÃ³pio do hook 0-10s, o motion por shot via signalstats, o `report.md` estruturado e o auto-save no Obsidian.

Autor original e detentor do copyright: **Bradley Bonanno** (ver [LICENSE](LICENSE)). Contribuidores estÃ£o listados em [AUTHORS.md](AUTHORS.md).

## Open source

LicenÃ§a MIT.

ConstruÃ­do sobre `yt-dlp`, `ffmpeg` e a `Read` multimodal do Claude. TranscriÃ§Ã£o Whisper via [Groq](https://groq.com) ou [OpenAI](https://openai.com).

---

[github.com/inematds/claude-watch](https://github.com/inematds/claude-watch) Â· [CrÃ©ditos](#crÃ©ditos) Â· [LICENSE](LICENSE)

---

Projeto adaptado da comunidade INEMA TDS - Marca Arckhe