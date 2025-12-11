<!-- README.md -->
<div align="center">

<h1>Template Render Pro</h1>

<p>GitHub Action para renderizar templates usando variáveis de ambiente.</p>

<p>
  <a href="https://github.com/marketplace/actions/template-render">
    <img alt="GitHub Marketplace" src="https://img.shields.io/badge/GitHub%20Marketplace-Template%20Render%20Pro-blue.svg">
  </a>
  <a href="https://github.com/Malnati/template-render/releases">
    <img alt="Release" src="https://img.shields.io/github/v/release/Malnati/template-render?label=release">
  </a>
  <a href="https://github.com/Malnati/template-render/blob/main/LICENSE">
    <img alt="License" src="https://img.shields.io/github/license/Malnati/template-render">
  </a>
</p>

</div>

---

## Visão geral

O **Template Render Pro** é uma GitHub Action genérica para **preencher templates** a partir de variáveis de ambiente, usando `envsubst`.

Você define:

- Um arquivo de **template** (por exemplo, `.md`, `.txt`, `.yml`);
- As **variáveis de ambiente** no workflow (`REPORT_FILE`, `PR_URL`, `COMMIT_SHA`, etc.);
- Opcionalmente, um **caminho de saída**.

A Action devolve:

- Um **arquivo renderizado** com os placeholders substituídos;
- O **conteúdo renderizado em memória**, pronto para ser encadeado em outras Actions (comentários em PR, bodies de issues, mensagens de auditoria, etc.).

---

## Por que usar

<ul>
  <li><strong>Desacoplado do fluxo</strong>: funciona com qualquer pipeline (auditoria, segurança, qualidade, release notes, etc.).</li>
  <li><strong>Sem mágica proprietária</strong>: usa <code>envsubst</code>, padrão em <code>ubuntu-latest</code>.</li>
  <li><strong>Pronto para PR bots</strong>: combine com Actions como <code>Malnati/pr-comment</code> para criar comentários ricos.</li>
  <li><strong>Reutilizável</strong>: centralize textos em templates versionados e reaproveite em múltiplos repositórios.</li>
</ul>

---

## Como funciona

1. Você define variáveis em `env:` no job (ex.: `ATTACHED_FILE_PATH`, `BRANCH_CONVENTION`, `COMMIT_SHA`, `PR_URL`).
2. O template usa placeholders no formato `${VAR}` (padrão `envsubst`).
3. A Action aplica `envsubst` e grava o resultado em um arquivo.
4. O caminho do arquivo (`path`) e o conteúdo (`body`) são expostos como `outputs`.

---

## Uso rápido (v1.0.0)

```yaml
# .github/workflows/example-template-render.yml
name: "Example Template Render"

on:
  workflow_dispatch:

jobs:
  demo:
    runs-on: ubuntu-latest
    env:
      ATTACHED_FILE_PATH: reports/example.md
      BRANCH_CONVENTION: ops/files/demo
      COMMIT_SHA: ${{ github.sha }}
      PR_URL: https://github.com/owner/repo/pull/1

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Render template
        id: render_timeline
        uses: Malnati/template-render@v1.0.0
        with:
          template: .github/workflows/hardcode-timeline.md
          output: /tmp/hardcode-timeline.md

      - name: Exibir conteúdo renderizado
        shell: bash
        env:
          BODY: ${{ steps.render_timeline.outputs.body }}
        run: |
          set -euo pipefail
          printf '%s
' "$BODY"
```

---

## Inputs

<table>
  <thead>
    <tr>
      <th>Input</th>
      <th>Obrigatório</th>
      <th>Descrição</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>template</code></td>
      <td><strong>Sim</strong></td>
      <td>Caminho do arquivo de template a ser renderizado (ex.: <code>.github/workflows/hardcode-timeline.md</code>).</td>
    </tr>
    <tr>
      <td><code>output</code></td>
      <td>Não</td>
      <td>Caminho do arquivo de saída. Se vazio, será usado um caminho temporário em <code>/tmp</code>.</td>
    </tr>
  </tbody>
</table>

---

## Outputs

<table>
  <thead>
    <tr>
      <th>Output</th>
      <th>Descrição</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>path</code></td>
      <td>Caminho absoluto do arquivo renderizado.</td>
    </tr>
    <tr>
      <td><code>body</code></td>
      <td>Conteúdo renderizado completo como string.</td>
    </tr>
  </tbody>
</table>

---

## Exemplo com comentário em Pull Request

Integração com <code>Malnati/pr-comment</code> usando o `body` gerado pelo Template Render Pro.

```yaml
# .github/workflows/hardcode-timeline.yml
name: "Hardcode Timeline"

on:
  workflow_dispatch:

jobs:
  timeline:
    runs-on: ubuntu-latest
    env:
      ATTACHED_FILE_PATH: .reports/20250101-1200_hardcode.md
      BRANCH_CONVENTION: ops/files/main
      COMMIT_SHA: ${{ github.sha }}
      PR_URL: https://github.com/owner/repo/pull/1

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Render timeline template
        id: timeline
        uses: Malnati/template-render@v1.0.0
        with:
          template: .github/workflows/hardcode-timeline.md
          output: /tmp/hardcode-timeline.md

      - name: Publicar comentário na PR
        uses: Malnati/pr-comment@v8.0.0
        env:
          BODY_MESSAGE: ${{ steps.timeline.outputs.body }}
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          pr_number: 1
          use_raw_body: "true"
          template_path: ""
          message_id: hardcode-timeline
```

---

## Exemplo de template

```markdown
<!-- .github/workflows/hardcode-timeline.md -->
A branch do arquivo ${ATTACHED_FILE_PATH} é ${BRANCH_CONVENTION} e o commit referente à inclusão deste arquivo é ${COMMIT_SHA}, então o Pull Request para integrar as mudanças desta branch é ${PR_URL}.

> [!TIP]
> Você pode modificar esta mensagem alterando o arquivo de template usado pelo fluxo.
```

---

## Boas práticas

<ul>
  <li>Versione seus templates no repositório (<code>.github/workflows/*.md</code>, <code>templates/*.md</code>, etc.).</li>
  <li>Use variáveis em maiúsculas para facilitar leitura (<code>REPORT_FILE</code>, <code>REPORT_BRANCH</code>, <code>PR_URL</code>).</li>
  <li>Garanta que todas as variáveis usadas no template estejam definidas em <code>env:</code> antes da chamada da Action.</li>
  <li>Mantenha a lógica no workflow e use o template apenas para interpolar valores.</li>
</ul>

---

## Compatibilidade

- Runner: <code>ubuntu-latest</code> (ou qualquer ambiente com <code>envsubst</code> disponível).
- Tipo: Composite Action (<code>action.yml</code> na raiz do repositório).

---

## Roadmap

<ul>
  <li>Adicionar modo opcional de validação de variáveis obrigatórias.</li>
  <li>Adicionar modo <em>dry-run</em> com log das variáveis aplicadas.</li>
  <li>Incluir mais exemplos oficiais de integração (auditoria de código, segurança, relatórios de qualidade).</li>
</ul>

---

## Licença

Distribuído sob licença MIT.  
Consulte o arquivo <code>LICENSE</code> no repositório para detalhes.

---

<div align="center">
  <sub>Repositório oficial: <a href="https://github.com/Malnati/template-render">Malnati/template-render</a></sub>
</div>
