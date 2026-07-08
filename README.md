# Meeting Digest Skill

**English** | [Português (Brasil)](#portugues-brasil)

Turn a meeting recording into a clean, verified digest. Maintained by ZaMpA.

This repo ships the same workflow for two agent harnesses:

| Harness | Skill path | Bias |
|---|---|---|
| Claude Code | [`claude/digest/SKILL.md`](claude/digest/SKILL.md) | Local-first shell workflow, Drive download support, transcript/audio/PDF pipeline |
| Codex | [`codex/digest/SKILL.md`](codex/digest/SKILL.md) | Connector-first workflow, video/audio/screenshot context, markdown-first outputs |

The skill is setup-agnostic: it does not assume a specific person, company,
Drive account, project, local path, or meeting provider.

## What It Produces

For each meeting, under a user-selected output directory. If the user does not
provide one, both skills create `meeting-digests/<PROJECT>/<DATE>` under the
current working directory.

| File or folder | Purpose |
|---|---|
| `source-metadata.json` | Non-secret source metadata used for reproducibility; excludes full URLs, file IDs, emails, tokens, and absolute paths |
| `transcript-raw.txt` | Raw transcript, captions, or ASR output |
| `transcript-clean.txt` | Cleaned transcript for reading and verification |
| `screenshots/` | Timestamped screenshots from visually relevant moments |
| `visual-references.md` | Screenshot index with timestamp and visual context |
| `digest.md` | Full digest: summary, decisions, action items, risks, notes |
| `executive-summary.md` | Short stakeholder-ready summary |
| `technical-analysis.md` | Evidence-backed analysis with source anchors |
| `handoff.md` | Numbered next-session plan |
| PDFs | Optional; generated only when requested or when the user's workflow requires them |

## Why This Is Not Just Summarization

- It prefers existing diarized transcripts when available, then captions, then
  audio transcription.
- It treats video and screenshots as first-class context, so UI demos, slides,
  charts, errors, and timestamps can be referenced accurately.
- It runs an adversarial verification pass against the raw transcript and visual
  references before finalizing the digest.
- It marks unclear claims as unconfirmed instead of inventing missing facts.
- It keeps real meeting outputs out of the repo by default.

## Install

Clone this repo from its GitHub page:

```bash
git clone <repo-url>
cd meeting-digest-skill
```

Install the harness you use, or install both.

For Claude Code:

```bash
mkdir -p ~/.claude/skills/digest
cp -R claude/digest/. ~/.claude/skills/digest/
```

For Codex:

```bash
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills/digest"
cp -R codex/digest/. "${CODEX_HOME:-$HOME/.codex}/skills/digest/"
```

On Windows (PowerShell), install for Claude Code with:

```powershell
New-Item -ItemType Directory -Force "$env:USERPROFILE\.claude\skills\digest" | Out-Null
Copy-Item -Recurse -Force claude\digest\* "$env:USERPROFILE\.claude\skills\digest\"
```

(For Codex on Windows, use `$env:CODEX_HOME` if set, otherwise `$env:USERPROFILE\.codex`.)

Restart the target harness so it discovers the skill.

You can also install the skill inside a project-local skills directory if your
harness supports project-scoped skills.

## Use

Ask naturally:

- Claude Code: "Use `/digest` on this Drive recording."
- Codex: "Use `$digest` on this Drive recording."
- "The meeting recording is in my Drive; create the digest."
- Paste a `drive.google.com/file/d/...` recording link.
- Provide a local recording, transcript, chat log, or screenshot folder.

The workflow runs autonomously and pauses only when a source cannot be accessed,
a required tool is missing, the output location is ambiguous, or a destructive
choice would be needed.

For a small smoke test before using Drive or video, create a local transcript
and ask the harness to digest it:

```bash
printf 'Alice: We decided to ship Friday.\nBob: I will update the docs.\n' > /tmp/meeting-digest-sample.txt
```

## Prerequisites

The exact requirements depend on the harness and source type:

- Google Drive access through the harness connector, browser session, or `rclone`
- `ffmpeg` and `ffprobe` for video/audio inspection and screenshots
- `pandoc`, `textutil`, or another document converter for `.docx` transcripts
- An available transcription path when no transcript or captions exist
- Chrome and `pdftotext` only if PDF generation is requested
- `python3` for small cleanup scripts when the agent chooses to use them

For local-first use, install the relevant tools with your system package
manager. Examples:

```bash
# macOS
brew install ffmpeg pandoc python rclone poppler

# Debian/Ubuntu
sudo apt-get install ffmpeg pandoc python3 rclone poppler-utils
```

```powershell
# Windows (winget); poppler/Chrome only if you generate PDFs
winget install Gyan.FFmpeg JohnMacFarlane.Pandoc Python.Python.3.12 Rclone.Rclone
# On Windows you can also use Chocolatey: choco install ffmpeg pandoc python rclone poppler
```

`source-metadata.json` should stay safe to inspect and share. Use labels,
source types, MIME types, durations, stream summaries, extraction methods, and
counts. Do not store full Drive URLs, full file IDs, personal emails, access
tokens, absolute local paths, or private account identifiers.

## Privacy Contract

This repo must remain reusable by anyone:

- Do not commit real recordings, transcripts, screenshots, chat logs, PDFs, or
  generated meeting digests.
- Do not add personal emails, private Drive links, local absolute paths, customer
  names, internal project names, or private setup details.
- Use placeholders in examples and documentation.
- If a screenshot contains secrets or sensitive third-party data, exclude it or
  redact it before referencing it.
- Keep public authorship as ZaMpA.

## Repo Layout

```text
claude/digest/SKILL.md        # Claude Code skill
codex/digest/SKILL.md         # Codex skill
codex/digest/agents/openai.yaml
examples/_style.css           # Optional PDF styling example
```

## Maintainer Validation

Before publishing a change, run privacy and syntax checks from the repo root:

```bash
touch /tmp/meeting-digest-private-denylist.txt
# Add your private names, emails, project names, local paths, account IDs,
# and any real Drive/Docs IDs you want to keep out of the public repo:
${EDITOR:-vi} /tmp/meeting-digest-private-denylist.txt
rg -n -f /tmp/meeting-digest-private-denylist.txt .
gitleaks dir . --redact --no-banner
gitleaks detect --source . --redact --no-banner --log-opts=--all
```

For Codex skill syntax, run the Codex skill validator if available in your setup.

## Contributing

Fork it, use it, make it yours. This skill is meant to be adapted to your own
setup, harness, and meeting workflow.

- **PRs are welcome** — bug fixes, a new harness variant, better prereq/install
  steps for your OS, or workflow improvements.
- **Open an issue** for ideas, questions, or something that broke.
- Keep every contribution **setup-agnostic and privacy-first**: no real names,
  emails, absolute local paths, private Drive links/file IDs, customer or
  internal project names, transcripts, or screenshots — see the Privacy Contract
  above and run the Maintainer Validation checks before opening a PR.
- Not comfortable opening a PR? Just fork and adapt it freely (Apache 2.0).

## License

Apache License 2.0. See [LICENSE](LICENSE) and [NOTICE](NOTICE).

<a id="portugues-brasil"></a>

## Português (Brasil)

[English](#meeting-digest-skill) | **Português (Brasil)**

Transforme uma gravação de reunião em um digest limpo e verificado. Mantido por
ZaMpA.

Este repo entrega o mesmo fluxo para dois harnesses de agente:

| Harness | Caminho da skill | Foco |
|---|---|---|
| Claude Code | [`claude/digest/SKILL.md`](claude/digest/SKILL.md) | Fluxo local-first via shell, suporte a download do Drive, pipeline de transcrição/audio/PDF |
| Codex | [`codex/digest/SKILL.md`](codex/digest/SKILL.md) | Fluxo connector-first, contexto de vídeo/audio/prints, saída principal em markdown |

A skill é setup-agnostic: não assume pessoa, empresa, conta de Drive, projeto,
caminho local ou provedor de reunião específico.

### O que ela gera

Para cada reunião, em um diretório escolhido pelo usuário. Se o usuário não
informar um diretório, as duas skills criam
`meeting-digests/<PROJECT>/<DATE>` dentro do diretório atual.

| Arquivo ou pasta | Finalidade |
|---|---|
| `source-metadata.json` | Metadados não secretos da fonte; exclui URLs completas, IDs de arquivo, emails, tokens e caminhos absolutos |
| `transcript-raw.txt` | Transcrição bruta, legendas ou ASR |
| `transcript-clean.txt` | Transcrição limpa para leitura e validação |
| `screenshots/` | Prints com minutagem de trechos visualmente relevantes |
| `visual-references.md` | Índice dos prints com timestamp e contexto visual |
| `digest.md` | Digest completo: resumo, decisões, ações, riscos, notas |
| `executive-summary.md` | Resumo curto para stakeholder |
| `technical-analysis.md` | Análise com evidências e referências à fonte |
| `handoff.md` | Plano numerado para a próxima sessão |
| PDFs | Opcional; gerado só quando solicitado ou quando o workflow do usuário exigir |

### Instalação

Clone este repo a partir da página dele no GitHub:

```bash
git clone <repo-url>
cd meeting-digest-skill
```

Instale o harness que você usa, ou instale os dois.

Para Claude Code:

```bash
mkdir -p ~/.claude/skills/digest
cp -R claude/digest/. ~/.claude/skills/digest/
```

Para Codex:

```bash
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills/digest"
cp -R codex/digest/. "${CODEX_HOME:-$HOME/.codex}/skills/digest/"
```

No Windows (PowerShell), instale para o Claude Code com:

```powershell
New-Item -ItemType Directory -Force "$env:USERPROFILE\.claude\skills\digest" | Out-Null
Copy-Item -Recurse -Force claude\digest\* "$env:USERPROFILE\.claude\skills\digest\"
```

(Para o Codex no Windows, use `$env:CODEX_HOME` se definido, senão `$env:USERPROFILE\.codex`.) Dependências no Windows: `winget install Gyan.FFmpeg JohnMacFarlane.Pandoc Python.Python.3.12 Rclone.Rclone`.

Reinicie o harness para que ele descubra a skill.

### Uso

Peça naturalmente:

- Claude Code: "Use `/digest` nesta gravação do Drive."
- Codex: "Use `$digest` nesta gravação do Drive."
- "A gravação da reunião está no meu Drive; faça o digest."
- Cole um link `drive.google.com/file/d/...`.
- Forneça uma gravação local, transcrição, chat ou pasta de prints.

O fluxo roda de forma autônoma e só pausa quando a fonte não pode ser acessada,
uma ferramenta obrigatória está ausente, o diretório de saída está ambíguo ou
uma decisão destrutiva seria necessária.

Para um smoke test pequeno antes de usar Drive ou vídeo, crie uma transcrição
local e peça para o harness gerar o digest:

```bash
printf 'Alice: We decided to ship Friday.\nBob: I will update the docs.\n' > /tmp/meeting-digest-sample.txt
```

### Contrato de privacidade

Este repo deve continuar reutilizável por qualquer pessoa:

- Não commite gravações, transcrições, prints, chats, PDFs ou digests reais.
- Não adicione emails pessoais, links privados do Drive, caminhos absolutos
  locais, nomes de clientes, nomes de projetos internos ou detalhes privados de setup.
- Use placeholders em exemplos e documentação.
- Se um print tiver segredos ou dados sensíveis de terceiros, exclua ou redija
  antes de referenciar.
- Mantenha a autoria pública como ZaMpA.

### Contribuindo

Faça fork, use, deixe do seu jeito. Esta skill existe pra ser adaptada ao seu
setup, harness e fluxo de reuniões.

- **PRs são bem-vindos** — correções, uma variante de harness nova, passos de
  instalação/pré-requisitos melhores pro seu SO, ou melhorias no fluxo.
- **Abra uma issue** pra ideias, dúvidas ou algo que quebrou.
- Mantenha toda contribuição **setup-agnostic e privacy-first**: sem nomes reais,
  emails, caminhos absolutos locais, links/IDs privados do Drive, nomes de
  clientes ou de projetos internos, transcrições ou prints — veja o Contrato de
  privacidade acima e rode os checks de validação antes de abrir o PR.
- Não quer abrir PR? Só faça fork e adapte livremente (Apache 2.0).

### Licença

Apache License 2.0. Veja [LICENSE](LICENSE) e [NOTICE](NOTICE).
