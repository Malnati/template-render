# Template Render Pro

Action de marketplace para renderizar templates a partir de variáveis de ambiente, usando envsubst.
Pensada para fluxos de auditoria, PR bots, notificações e qualquer cenário em que você queira montar mensagens dinâmicas a partir de arquivos de template (.md, .txt, .yml, etc.).

⸻

Visão geral

O Template Render Pro recebe:
	•	Um arquivo de template (com placeholders ${VAR}).
	•	As variáveis de ambiente já definidas no job.
	•	Opcionalmente, um caminho de saída.

E devolve:
	•	Um arquivo renderizado com os placeholders substituídos.
	•	O conteúdo renderizado em memória, pronto para ser usado em outras Actions (por exemplo, como corpo de comentário em PR).

É uma Action genérica, desacoplada de qualquer fluxo específico, para você reutilizar em múltiplos repositórios e pipelines.

⸻

Exemplo rápido de uso (v1.0.0)

# .github/workflows/example-template-render-pro.yml
name: "Example Template Render Pro"

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
        uses: Malnati/template-render-pro@v1.0.0
        with:
          template: .github/workflows/hardcode-timeline.md
          output: /tmp/hardcode-timeline.md

      - name: Exibir conteúdo renderizado
        shell: bash
        env:
          BODY: ${{ steps.render_timeline.outputs.body }}
        run: |
          set -euo pipefail
          printf '%s\n' "$BODY"


⸻

Casos de uso

Alguns cenários em que a Action se encaixa bem:
	•	Comentários padronizados em Pull Requests, combinando com Actions como Malnati/pr-comment.
	•	Mensagens de timeline de auditoria (hardcode, segurança, qualidade).
	•	Geração de arquivos .md ou .txt parametrizados para relatórios.
	•	Templates de body de PR, issues ou notificações internas.
	•	Qualquer fluxo em que você queira separar conteúdo (template) de dados dinâmicos (variáveis de ambiente).

⸻

Como funciona
	1.	Você define variáveis de ambiente no job (por exemplo, ATTACHED_FILE_PATH, BRANCH_CONVENTION, COMMIT_SHA, PR_URL).
	2.	O arquivo de template usa placeholders no formato ${VAR} (padrão envsubst).
	3.	A Action lê o template, aplica envsubst com o ambiente atual do job e grava o resultado em um arquivo.
	4.	O caminho do arquivo e o corpo renderizado são expostos em outputs para encadear com outras Actions.

⸻

Inputs

Input	Obrigatório	Descrição
template	Sim	Caminho do arquivo de template a ser renderizado (por exemplo, .github/workflows/hardcode-timeline.md).
output	Não	Caminho do arquivo de saída. Se vazio, será usado um caminho temporário em /tmp.


⸻

Outputs

Output	Descrição
path	Caminho absoluto do arquivo renderizado.
body	Conteúdo renderizado completo como string.


⸻

Exemplo com comentário em PR (modo raw)

Integração direta com Malnati/pr-comment usando o body gerado pelo Template Render Pro.

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
        uses: Malnati/template-render-pro@v1.0.0
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


⸻

Exemplo de template

<!-- .github/workflows/hardcode-timeline.md -->
A branch do arquivo ${ATTACHED_FILE_PATH} é ${BRANCH_CONVENTION} e o commit referente à inclusão deste arquivo é ${COMMIT_SHA}, então o Pull Request para integrar as mudanças desta branch é ${PR_URL}.

> [!TIP]
> Você pode modificar esta mensagem alterando o arquivo de template usado pelo fluxo.


⸻

Boas práticas
	•	Manter os templates versionados no repositório (.github/workflows/*.md, templates/*.md).
	•	Usar nomes de variáveis em maiúsculas para facilitar a identificação (REPORT_FILE, REPORT_BRANCH, etc.).
	•	Garantir que todas as variáveis usadas no template estejam definidas em env: antes da chamada da Action.
	•	Evitar lógica complexa dentro do template; deixar a lógica no workflow e apenas interpolar valores.

⸻

Limitações
	•	A Action depende de envsubst (disponível por padrão na imagem ubuntu-latest do GitHub Actions).
	•	O formato de placeholder suportado é ${VAR} (não há suporte para lógica condicional ou laços).
	•	Não há validação automática de variáveis ausentes: placeholders sem variável definida permanecem vazios ou substituídos por string vazia.

⸻

Desenvolvimento
	•	Repositório: https://github.com/Malnati/template-render-pro
	•	Tipo: Composite Action (action.yml)
	•	Plataforma alvo: ubuntu-latest (com envsubst disponível no ambiente)

Sugestões de evolução:
	•	Adicionar suporte opcional a validação de variáveis obrigatórias.
	•	Adicionar modo de “dry-run” com log das variáveis aplicadas.
	•	Expandir exemplos oficiais de integração com outras Actions (PR bots, linters, scanners).

⸻

Licença

Distribuído sob licença MIT.
Consulte o arquivo LICENSE no repositório para detalhes.

⸻

Para usar no Marketplace, crie a tag v1.0.0, configure a Action com o arquivo action.yml atual, adicione este README ao repositório e registre a Action na GitHub Marketplace apontando para Malnati/template-render-pro@v1.0.0.
