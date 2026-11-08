# Roteiro para reproduzir o estudo manualmente

Oct 26, 2026 · @Isaac Orcine

Este roteiro leva você da seleção dos repositórios no AIDev até os números de cada questão de pesquisa, fazendo cada etapa por conta própria. Ele começa por um **piloto com 5 projetos**, que valida o método e os notebooks em poucos dias, e termina mostrando como escalar para os 99 candidatos.

## Antes de começar

Seu MacBook Air M5, com 16 GB de RAM e 512 GB de SSD, dá conta das duas fases. O piloto ocupa menos de 2 GB e cada projeto é medido em minutos. A amostra completa precisa de cerca de 25 GB livres durante a coleta e de uma a duas semanas de trabalho, contando as verificações.

| Item | Para quê | Observação |
| --- | --- | --- |
| Espaço em disco (20 GB ou mais) | Clonar os repositórios | Os 99 candidatos ocupam cerca de 25 GB. Apague cada clone depois de medir e mantenha só as saídas |
| Python 3.11 ou mais recente | Toda a análise | Você já tem o Anaconda no Mac |
| Git | Histórico e versões dos arquivos | Já vem com o Xcode Command Line Tools |
| Ruff 0.16.8 e Radon 6.0.1 | Avisos e complexidade | Use essas versões para comparar com o artigo |
| pandas, pyarrow, statsmodels, SciPy, matplotlib, lifelines | Painéis, testes, gráficos e sobrevivência | Instalação na Etapa 1 |
| R com o pacote `did` (opcional) | Estimador de Callaway e Sant'Anna | Mais simples do que implementá-lo à mão |

Organize o trabalho em pastas e grave a saída de cada etapa em arquivo (CSV ou Parquet). Assim, cada etapa pode ser refeita sem repetir as anteriores:

```text
estudo/
  data/        tabelas do AIDev
  repos/       clones (temporários)
  out/<repo>/  saídas por repositório
  notebooks/   01_selecao, 02_historico, 03_metricas, 04_rastreamento, 05_analises
```

Crie um notebook por etapa e anote em um diário de pesquisa cada decisão que tomar (critérios, limiares, exceções). Esse diário vira a seção de metodologia.

## O piloto: 5 projetos

Os cinco projetos abaixo são pequenos, têm pelo menos 12 meses de histórico antes da adoção e cobrem quatro agentes diferentes. Os valores de referência vêm da rodada feita para o artigo: use-os só para conferir os seus, porque cada número tem que sair dos seus notebooks.

| Projeto | Agente no AIDev | PRs de agentes aceitos (AIDev) | Adoção (referência) | Arquivos Python (máx.) |
| --- | --- | --- | --- | --- |
| [lidofinance/diffyscan](https://github.com/lidofinance/diffyscan) | OpenAI Codex | 6 | mai/2025 | 34 |
| [numbagg/numbagg](https://github.com/numbagg/numbagg) | Claude Code | 12 | jun/2025 | 29 |
| [econ-ark/HARK](https://github.com/econ-ark/HARK) | OpenAI Codex | 8 | mai/2025 | 151 |
| [NewFuture/DDNS](https://github.com/NewFuture/DDNS) | Copilot | 13 | jul/2025 | 108 |
| [airbytehq/PyAirbyte](https://github.com/airbytehq/PyAirbyte) | Devin | 14 | jan/2025 | 261 |

O PyAirbyte adotou bem antes dos outros, então serve de controle "ainda não tratado" nos primeiros meses dos demais. Com só 5 projetos, porém, esse tipo de comparação é apenas ilustrativo.

## Arquivos: links e o que fazer com cada um

| Arquivo | Link | Onde salvar | O que fazer |
| --- | --- | --- | --- |
| `repository.parquet` | [baixar (AIDev v3)](https://huggingface.co/datasets/hao-li/AIDev/resolve/v3/repository.parquet) | `estudo/data/` | Ler na Etapa 2 para obter linguagem, estrelas e nome de cada repositório |
| `pull_request.parquet` | [baixar (AIDev v3)](https://huggingface.co/datasets/hao-li/AIDev/resolve/v3/pull_request.parquet) | `estudo/data/` | Ler na Etapa 2 (contagem de PRs aceitos) e na Etapa 3 (número, datas e agente de cada PR) |
| `pr_commits.parquet` | [baixar (AIDev v3)](https://huggingface.co/datasets/hao-li/AIDev/resolve/v3/pr_commits.parquet) | `estudo/data/` | Usar na Etapa 3 para achar PRs integrados por *rebase* (SHA no histórico) |
| Os 5 repositórios | links da tabela acima | `estudo/repos/` | Clonar na Etapa 3 com `git clone`, não pelo botão de download do site, porque o ZIP não traz o histórico |
| Extensão Python do VS Code | [Marketplace](https://marketplace.visualstudio.com/items?itemName=ms-python.python) | Instalar no VS Code | Permite escolher o ambiente `agentes` como interpretador |
| Extensão Jupyter do VS Code | [Marketplace](https://marketplace.visualstudio.com/items?itemName=ms-toolsai.jupyter) | Instalar no VS Code | Roda os notebooks `.ipynb` dentro do VS Code |
| [Documentação do AIDev](https://huggingface.co/datasets/hao-li/AIDev/tree/v3) | consulta | não precisa salvar | Ler `data_table.md` para o significado de cada coluna |

Se o navegador salvar algum arquivo com outro nome, renomeie para o nome da primeira coluna. Os notebooks vão procurar exatamente esses nomes.

## Etapa 1 – Preparar o ambiente

Você vai trabalhar no VS Code, com um ambiente Python só para o estudo e as versões das ferramentas fixadas. Versões diferentes do Ruff mudam o conjunto de regras e, com isso, os números.

1. **Crie a pasta do estudo** com as subpastas `data`, `repos`, `out` e `notebooks` (estrutura acima). No VS Code, use *File › Open Folder* e abra a pasta `estudo`.
2. **Instale as extensões** Python e Jupyter da Microsoft (links na tabela de arquivos).
3. **Crie o ambiente.** Abra o terminal integrado (*Terminal › New Terminal*) e rode:

```bash
conda create -n agentes python=3.12 -y
conda activate agentes
pip install pandas pyarrow "ruff==0.16.8" "radon==6.0.1" statsmodels scipy matplotlib lifelines ipykernel
python -m ipykernel install --user --name agentes --display-name "Python (agentes)"
ruff --version && radon --version && git --version
```

4. **Ligue os notebooks ao ambiente.** Crie `notebooks/01_selecao.ipynb`, clique em *Select Kernel* no canto superior direito e escolha **Python (agentes)**. Faça o mesmo em cada notebook novo.
5. **Teste.** Rode, na primeira célula, `import pandas, statsmodels, lifelines; print('ok')`. Rode também `!ruff --version`: o `!` executa o comando no terminal do ambiente. Se aparecer `ok` e a versão 0.16.8, está tudo pronto.

Anote as versões que aparecerem, porque elas entram na Tabela 3 do artigo. Nos notebooks, você pode rodar comandos do Git e do Ruff tanto com `!` quanto com `subprocess.run` do Python. O `subprocess` é melhor quando precisar ler a saída dentro do código.

## Etapa 2 – Obter o AIDev v3 e selecionar candidatos

Use a versão v3 do AIDev (corte em agosto de 2025), que é a citada no artigo. A versão atual no Hugging Face é outra e tem números diferentes.

1. Baixe pelo navegador, para a pasta `data/`, as tabelas [repository.parquet](https://huggingface.co/datasets/hao-li/AIDev/resolve/v3/repository.parquet), [pull\_request.parquet](https://huggingface.co/datasets/hao-li/AIDev/resolve/v3/pull_request.parquet) e [pr\_commits.parquet](https://huggingface.co/datasets/hao-li/AIDev/resolve/v3/pr_commits.parquet).
2. Abra as três com pandas e confira as colunas que vai usar. Em `repository`: `id`, `full_name`, `language` e `stars`. Em `pull_request`: `number`, `agent`, `created_at`, `merged_at` e `repo_url`. Em `pr_commits`: `sha` e `pr_id`.
3. Filtre os repositórios Python e conte os PRs de agentes aceitos (com `merged_at` preenchido) de cada um. Fique com os que têm pelo menos 5:

```python
import pandas as pd
repo = pd.read_parquet('../data/repository.parquet')
pr = pd.read_parquet('../data/pull_request.parquet')
aceitos = pr[pr.merged_at.notna()]
py = repo[repo.language == 'Python']
cont = aceitos[aceitos.repo_id.isin(py.id)].groupby('repo_id').agg(
    n_prs=('id', 'size'), primeiro=('merged_at', 'min'),
    agente=('agent', lambda s: s.value_counts().idxmax()))
cand = cont[cont.n_prs >= 5].join(py.set_index('id')[['full_name', 'stars']])
cand.to_csv('../out/candidatos.csv')  # caminhos relativos a notebooks/
```

4. Confira as contagens: 2.807 repositórios no total, 530 em Python, 315 com ao menos um PR aceito e 99 candidatos. Se os números forem outros, verifique se baixou mesmo a v3.
5. Grave a lista do piloto em `out/piloto.csv`, filtrando os 99 candidatos pelos cinco projetos da tabela do piloto. Os notebooks seguintes devem ler esse arquivo. Assim, para escalar, basta trocar `piloto.csv` por `candidatos.csv`.

```python
piloto = ['lidofinance/diffyscan', 'numbagg/numbagg', 'econ-ark/HARK',
          'NewFuture/DDNS', 'airbytehq/PyAirbyte']
sel = cand[cand.full_name.isin(piloto)]
sel.to_csv('../out/piloto.csv')
sel  # devem aparecer 5 linhas
```

## Etapa 3 – Clonar, reconstruir o histórico e atribuir commits a agentes

Esta é a etapa que mais influencia os resultados. Um erro de atribuição muda a data de adoção e o grupo de cada arquivo, então valide à mão antes de seguir.

**3.1 Clonar.** Clone só o ramo padrão, um repositório por vez:

```bash
git clone --single-branch --no-tags https://github.com/OWNER/REPO repos/OWNER__REPO
```

No piloto, clone os cinco de uma vez numa célula do notebook `02_historico.ipynb`. Os cinco juntos ocupam menos de 2 GB:

```python
import pandas as pd, subprocess, os
for nome in pd.read_csv('../out/piloto.csv').full_name:
    destino = '../repos/' + nome.replace('/', '__')
    if not os.path.exists(destino):
        subprocess.run(['git', 'clone', '--single-branch', '--no-tags',
                        f'https://github.com/{nome}', destino], check=True)
```

Se preferir clonar pelo terminal do VS Code, rode a partir da pasta `estudo/`. O nome depois da URL é a pasta de destino, com `__` no lugar da barra, que é o nome que os passos seguintes esperam. Não use `--depth`: o estudo precisa do histórico completo.

```bash
cd estudo/repos
git clone --single-branch --no-tags https://github.com/lidofinance/diffyscan.git lidofinance__diffyscan
git clone --single-branch --no-tags https://github.com/numbagg/numbagg.git numbagg__numbagg
git clone --single-branch --no-tags https://github.com/econ-ark/HARK.git econ-ark__HARK
git clone --single-branch --no-tags https://github.com/NewFuture/DDNS.git NewFuture__DDNS
git clone --single-branch --no-tags https://github.com/airbytehq/PyAirbyte.git airbytehq__PyAirbyte
ls   # devem aparecer as 5 pastas
```

`--single-branch` baixa só o ramo padrão (`main` ou `master`), e `--no-tags` dispensa as *tags*, que não são usadas.

**3.2 Extrair o histórico *first-parent*.** É a sequência de *commits* que entrou na linha principal, com um *commit* por PR integrado:

```bash
git -C repos/OWNER__REPO log --first-parent --format='%x1e%H%x1f%P%x1f%an%x1f%ae%x1f%cI%x1f%B' > out/OWNER__REPO/log.txt
```

O separador `%x1e` divide os *commits*, e `%x1f` divide os campos. Assim a mensagem completa, com os *trailers* de coautoria, não quebra a leitura no pandas. Converta todas as datas para UTC.

No comando acima, `OWNER__REPO` é um marcador: troque pelo nome da pasta clonada, por exemplo `lidofinance__diffyscan`. A pasta de saída precisa existir antes, senão o redirecionamento `>` falha. Os campos gravados são:

| Código | Conteúdo |
| --- | --- |
| `%x1e` | início de cada *commit* |
| `%H` | hash do *commit* |
| `%P` | hashes dos pais |
| `%an`, `%ae` | nome e e-mail do autor |
| `%cI` | data do *commit* em ISO 8601 |
| `%B` | mensagem completa |
| `%x1f` | separador de campos |

No piloto, rode os cinco de uma vez no terminal, **a partir da pasta `estudo/`** (os caminhos são relativos a ela, não a `notebooks/`):

```bash
cd estudo
for r in lidofinance__diffyscan numbagg__numbagg econ-ark__HARK NewFuture__DDNS airbytehq__PyAirbyte; do
  mkdir -p out/$r
  git -C repos/$r log --first-parent --format='%x1e%H%x1f%P%x1f%an%x1f%ae%x1f%cI%x1f%B' > out/$r/log.txt
done
```

**Conferência.** O número de marcadores `%x1e` do arquivo deve ser igual ao número de *commits* do ramo principal. Nos dois últimos números de cada linha abaixo, eles têm de coincidir (no numbagg, o resultado de referência é 791 nos dois):

```bash
for r in lidofinance__diffyscan numbagg__numbagg econ-ark__HARK NewFuture__DDNS airbytehq__PyAirbyte; do
  echo $r $(tr -cd '\036' < out/$r/log.txt | wc -c) $(git -C repos/$r rev-list --first-parent --count HEAD)
done
```

O arquivo parece ilegível se aberto no editor, com caracteres estranhos entre os campos. Isso é esperado: a Etapa 3.3 o lê em Python e separa os *commits* por esses caracteres.

**3.3 Atribuir pelo AIDev.** Para cada PR de agente aceito daquele repositório, procure o *commit* correspondente no histórico:

- título com `Merge pull request #N` (merge comum) ou terminando em `(#N)` (*squash*), **e** data entre a criação do PR (menos 1 dia) e o *merge* (mais 2 dias);
- ou SHA listado em `pr_commits` que aparece no histórico *first-parent* (PRs integrados por *rebase*).

**3.4 Atribuir por assinatura.** Depois de agosto de 2025, o AIDev não cobre mais nada, então use as marcas que os próprios agentes deixam. Ignore qualquer assinatura anterior a 01/03/2024.

| Agente | Considere | Não considere |
| --- | --- | --- |
| Copilot | `copilot-swe-agent`, e-mail `198982749+Copilot@users.noreply.github.com`, *branch* `/copilot/` | `175728472+Copilot`, que marca sugestão de revisão aceita por humano |
| Claude Code | `Generated with [Claude Code]`, `Claude <noreply@anthropic.com>`, *branch* `/claude/` | `Co-Authored-By: Claude` sem o e-mail, que pega pessoas chamadas Claude |
| Devin | `devin-ai-integration` |  |
| OpenAI Codex | `chatgpt-codex-connector`, *branch* `/codex/` |  |
| Cursor | `cursoragent@cursor.com`, `Co-authored-by: Cursor Agent`, *branch* `/cursor/` |  |
| Jules | `google-labs-jules` |  |

As marcas de *branch* aparecem na mensagem de merge, como em "Merge pull request #12 from user/codex/...".

**3.5 Definir a adoção e aplicar os critérios de inclusão.** A data de adoção é a do primeiro *commit* de agente no ramo padrão. Mantenha o repositório só se:

1. o primeiro *commit* do projeto for pelo menos 12 meses anterior à adoção;
2. houver pelo menos 30 *commits* nesses 12 meses;
3. ao menos um PR do AIDev for encontrado no ramo padrão;
4. a adoção for até 31/08/2025.

Anote quantos repositórios cada critério exclui: isso forma a Tabela 4 do artigo.

**3.6 Validar à mão (opcional).** Sorteie 30 *commits* atribuídos a agentes e 30 atribuídos a humanos, abra cada um no GitHub e classifique você mesmo. A precisão obtida (por exemplo, 29 de 30 corretos) vai para as ameaças à validade e é o melhor argumento para a banca. **Se você não for validar,** declare isso nas ameaças à validade: a atribuição não terá precisão medida, e o leitor não saberá quantos *commits* foram classificados errado. Nesse caso, faça ao menos estas verificações automáticas e registre os números:

- quantos *commits* cada critério atribuiu (AIDev e assinatura) e quantos casos os dois critérios classificaram de forma diferente;
- a proporção de *commits* de agentes em cada projeto, para achar casos absurdos (por exemplo, 90% de um projeto antigo);
- que nenhuma atribuição por assinatura é anterior a 01/03/2024.

**3.7 Fazer 3.3 e 3.4 no notebook (piloto).** Crie `estudo/notebooks/02_atribuicao.ipynb`, com o *kernel* Python (agentes), e use duas células. Ele lê o `log.txt` do passo 3.2 (rode antes o laço dos cinco projetos), atribui cada *commit* a um agente ou a um humano pelos critérios 3.3 (AIDev) e 3.4 (assinatura) e confere os critérios de inclusão do 3.5. A célula 1 só define as funções:

```python
import os, re, warnings
import pandas as pd

warnings.filterwarnings('ignore', category=DeprecationWarning)   # aviso interno do pandas com o NumPy novo

ERA_AGENTES = pd.Timestamp('2024-03-01', tz='UTC')   # nenhum agente de codificação existia antes
ASSINATURAS = [   # 3.4: marcas que cada agente deixa em autor, e-mail, mensagem ou nome do branch do merge
    ('Copilot', re.compile(r'copilot-swe-agent|198982749\+Copilot@users\.noreply|\bfrom [^\s]+/copilot/', re.I)),
    ('Devin', re.compile(r'devin-ai-integration|\bfrom [^\s]+/devin/', re.I)),
    ('OpenAI_Codex', re.compile(r'chatgpt-codex-connector|\bfrom [^\s]+/codex[/-]', re.I)),
    ('Cursor', re.compile(r'cursoragent@cursor\.com|Co-authored-by: Cursor Agent|\bfrom [^\s]+/cursor/', re.I)),
    ('Claude_Code', re.compile(r'Generated with \[Claude Code\]|Claude <noreply@anthropic\.com>|\bfrom [^\s]+/claude/', re.I)),
    ('Jules', re.compile(r'google-labs-jules', re.I)),
]

pr = pd.read_parquet('../data/pull_request.parquet')
pc = pd.read_parquet('../data/pr_commits.parquet')


def ler_historico(chave, base='..'):
    """Lê o log.txt gerado no passo 3.2."""
    texto = open(f'{base}/out/{chave}/log.txt', encoding='utf-8', errors='replace').read()
    linhas = []
    for rec in texto.split('\x1e')[1:]:
        sha, pais, autor, email, data, msg = rec.split('\x1f', 5)
        linhas.append(dict(sha=sha, autor=autor, email=email, msg=msg,
                           data=pd.Timestamp(data).tz_convert('UTC')))
    return pd.DataFrame(linhas).sort_values('data').reset_index(drop=True)


def assinatura(c):
    """3.4: agente indicado pela assinatura do commit, ou None."""
    if c.data < ERA_AGENTES:
        return None
    blob = f'{c.autor} <{c.email}>\n{c.msg}'
    for agente, rx in ASSINATURAS:
        if rx.search(blob):
            return agente
    return None


def atribuir(nome, base='..'):
    chave = nome.replace('/', '__')
    h = ler_historico(chave, base)
    # PRs aceitos do AIDev neste repositório
    prs = pr[(pr.repo_url.str.lower() == f'https://api.github.com/repos/{nome}'.lower()) & pr.merged_at.notna()]
    num2agente = dict(zip(prs.number, prs.agent))
    # o commit no ramo principal deve cair entre 1 dia antes da criação do PR e 2 dias depois do merge
    criado = pd.to_datetime(prs.created_at, utc=True) - pd.Timedelta('1D')
    aceito = pd.to_datetime(prs.merged_at, utc=True) + pd.Timedelta('2D')
    janela = dict(zip(prs.number, zip(criado, aceito)))
    shas_pr = pc[pc.pr_id.isin(prs.id)].merge(prs[['id', 'agent']], left_on='pr_id', right_on='id')
    sha2agente = dict(zip(shas_pr.sha, shas_pr.agent))
    no_historico = set(h.sha)

    agente, fonte, assin = [], [], []
    for c in h.itertuples():
        a_sig = assinatura(c)
        assin.append(a_sig)
        titulo = c.msg.split('\n', 1)[0]
        m = re.search(r'Merge pull request #(\d+)', titulo) or re.search(r'\(#(\d+)\)\s*$', titulo)
        if m and int(m.group(1)) in num2agente and janela[int(m.group(1))][0] <= c.data <= janela[int(m.group(1))][1]:
            agente.append(num2agente[int(m.group(1))]); fonte.append('aidev')            # 3.3 merge ou squash
        elif c.sha in sha2agente and c.sha in no_historico:
            agente.append(sha2agente[c.sha]); fonte.append('aidev')                      # 3.3 rebase
        elif a_sig:
            agente.append(a_sig); fonte.append('assinatura')                             # 3.4
        else:
            agente.append(None); fonte.append(None)
    h['agente'], h['fonte'], h['assinatura'] = agente, fonte, assin
    h['is_agente'] = h.agente.notna()
    h['titulo'] = h.msg.str.split('\n').str[0]
    h.drop(columns='msg').to_csv(f'{base}/out/{chave}/commits_atribuidos.csv', index=False)
    return h, len(prs)


def resumo(nome, adocao_aidev, base='..'):
    h, n_prs = atribuir(nome, base)
    ag = h[h.is_agente]
    adocao = ag.data.min() if len(ag) else None
    pre12 = int(((h.data >= adocao - pd.DateOffset(months=12)) & (h.data < adocao)).sum()) if adocao is not None else 0
    inclui = bool(adocao is not None and h.data.min() <= adocao - pd.DateOffset(months=12) and pre12 >= 30)
    # 3.6 (verificações automáticas): onde a assinatura discorda do AIDev
    ai = h[h.fonte == 'aidev']
    discorda = int((ai.assinatura.notna() & (ai.assinatura != ai.agente)).sum())
    return dict(projeto=nome, commits=len(h), prs_aidev=n_prs,
                agente_aidev=int((h.fonte == 'aidev').sum()), agente_assinatura=int((h.fonte == 'assinatura').sum()),
                pct_agente=round(100 * len(ag) / len(h), 1),
                agente_principal=ag.agente.value_counts().idxmax() if len(ag) else None,
                adocao=adocao.strftime('%Y-%m-%d') if adocao is not None else None,
                mes_adocao_aidev=pd.Timestamp(adocao_aidev).strftime('%Y-%m'),
                commits_12m_antes=pre12, inclui=inclui, aidev_x_assinatura_discordam=discorda)
```

A célula 2 roda os cinco projetos e grava um resumo em `out/atribuicao_resumo.csv`; o código de cada projeto fica em `out/<projeto>/commits_atribuidos.csv`:

```python
piloto = pd.read_csv('../out/piloto.csv')
tabela = pd.DataFrame([resumo(p.full_name, p.primeiro) for _, p in piloto.iterrows()])
tabela.to_csv('../out/atribuicao_resumo.csv', index=False)
tabela
```

**Referência do piloto** (valores da minha execução; o número de *commits* pode ser um pouco maior no seu, porque os projetos continuam recebendo *commits*):

| Projeto | *Commits* | Agente (AIDev) | Agente (assinatura) | Adoção | Incluído |
| --- | --- | --- | --- | --- | --- |
| numbagg | 791 | 12 | 28 | 2025-06-17 | sim |
| HARK | 1.063 | 8 | 7 | 2025-05-20 | sim |
| DDNS | 596 | 13 | 68 | 2025-07-01 | sim |
| diffyscan | 193 | 6 | 0 | 2025-05-22 | sim |
| PyAirbyte | 588 | 9 | 234 | 2025-01-28 | sim |

A adoção é a data do primeiro *commit* atribuído a agente. Ela deve cair no mesmo mês da coluna `mes_adocao_aidev` (primeiro PR aceito no AIDev). Se cair em outro mês, a janela de 12 meses da Etapa 4 ficou deslocada: rode a Etapa 4 de novo com a data nova. A coluna `aidev_x_assinatura_discordam` conta os *commits* em que a assinatura aponta um agente diferente do AIDev; no piloto, o valor é 0 em todos. Esses números fazem parte das verificações automáticas do 3.6.

## Etapa 4 – Medir a qualidade mês a mês

O objetivo é uma tabela com uma linha por (repositório, mês, arquivo), cobrindo de 12 meses antes da adoção até setembro de 2026.

**4.1 Encontrar o *commit* de cada mês.** O estado no fim de um mês é o último *commit* antes do dia 1º do mês seguinte:

```bash
git -C repos/OWNER__REPO rev-list -1 --first-parent --before="2025-07-01T00:00:00Z" HEAD
```

**4.2 Listar os arquivos Python e suas versões.** O `ls-tree` mostra o identificador (*blob*) de cada versão de arquivo:

```bash
git -C repos/OWNER__REPO ls-tree -r <sha> | grep '\.py$'
```

Exclua caminhos com `venv/`, `.venv/`, `site-packages/`, `node_modules/`, `vendor/`, `third_party/`, `build/`, `dist/` e `.tox/`. Se um repositório passar de um limite que você definir (o artigo usou 5.000 arquivos Python), exclua-o por custo computacional e declare isso.

**4.3 Medir cada versão uma única vez.** Um arquivo que não mudou tem o mesmo *blob* em todos os meses. Então junte os *blobs* distintos, grave cada um como arquivo e meça todos de uma vez:

```bash
git -C repos/OWNER__REPO cat-file -p <blob> > tmp/<blob>.py
```

**4.4 Rodar o Ruff** em modo isolado, com as mesmas regras do artigo:

```bash
ruff check --isolated --no-cache --exit-zero --output-format json \
  --select F,E,B,SIM,C90,PLE,PLW,PLR,PLC,ERA \
  --ignore E501,PLR2004,PLC0415 tmp/ > ruff.json
```

Para cada aviso, guarde a regra (`code`) e o texto da linha indicada em `location.row`, sem espaços nas pontas. Você vai precisar do texto na Etapa 5.

**4.5 Complexidade e tamanho com o Radon.** `radon cc -j tmp/` dá a complexidade de cada função e método, e `radon raw -j tmp/` dá as linhas de código-fonte (SLOC). Por arquivo, guarde a soma da complexidade, o número de funções e o SLOC.

**4.6 SATD.** Conte os comentários com TODO, FIXME, HACK ou XXX. Use o módulo `tokenize` para olhar só comentários, sem pegar *strings*.

**4.7 Montar a tabela final.** Ligue as métricas de volta a (mês, caminho) pelo *blob*. Colunas sugeridas: `repo, mes, caminho, blob, sloc, cc_soma, n_funcoes, avisos, avisos_complexidade, satd`. Os `avisos_complexidade` são as regras C901 e PLR09xx. Acrescente, para cada caminho, a data do primeiro toque de agente e a do primeiro toque humano após a adoção. Elas saem de `git log --first-parent --raw -M`, que lista os arquivos alterados em cada *commit*.

**4.8 Fazer as consultas 4.1 a 4.7 no notebook (piloto).** Os comandos manuais acima servem para entender cada passo. Para rodar os cinco projetos, crie o notebook `03_medicao.ipynb` em `estudo/notebooks/`, com o *kernel* Python (agentes). Cole o código abaixo em duas células. A primeira só define as funções: cada mês, o último *commit* do *first-parent* (4.1), os arquivos `.py` sem as pastas excluídas (4.2), a leitura de cada *blob* uma única vez (4.3) e as medições de Ruff, Radon e SATD (4.4 a 4.6). A janela vai de 12 meses antes da adoção até setembro de 2026, com datas em UTC.

```python
import importlib.util, io, json, os, re, subprocess, sys, tempfile, tokenize
import pandas as pd
from radon.complexity import cc_visit
from radon.raw import analyze

RUFF_SELECT = 'F,E,B,SIM,C90,PLE,PLW,PLR,PLC,ERA'
RUFF_IGNORE = 'E501,PLR2004,PLC0415'
EXCL = re.compile(r'(^|/)(\.venv|venv|env|site-packages|node_modules|vendor|third_party|build|dist|\.tox)/')
SATD = re.compile(r'\b(TODO|FIXME|HACK|XXX)\b')
RUFF = [sys.executable, '-m', 'ruff'] if importlib.util.find_spec('ruff') else ['ruff']
LIMITE_ARQUIVOS = 5000          # acima disso o repositório é excluído por custo
FIM = pd.Timestamp('2026-10-01', tz='UTC')   # observa até setembro de 2026


def git(repo, *args, entrada=None):
    r = subprocess.run(['git', '-C', repo, *args], capture_output=True, input=entrada)
    if r.returncode != 0:
        raise RuntimeError(r.stderr.decode('utf-8', 'replace')[:300])
    return r.stdout


def meses_da_janela(adocao):
    """Fronteiras (1º dia de cada mês, UTC): de 12 meses antes da adoção até out/2026."""
    a = adocao.tz_convert('UTC')
    ini = pd.Timestamp(a.year, a.month, 1, tz='UTC') - pd.DateOffset(months=12)
    return list(pd.date_range(ini + pd.DateOffset(months=1), FIM, freq='MS', tz='UTC'))


def snapshots(repo, fronteiras):
    """4.1 e 4.2: para cada mês, o último commit do first-parent e os .py (caminho, blob)."""
    linhas = []
    for f in fronteiras:
        sha = git(repo, 'rev-list', '-1', '--first-parent', f'--before={f.strftime("%Y-%m-%dT%H:%M:%SZ")}', 'HEAD').decode().strip()
        if not sha:
            continue                       # ainda não havia commits
        mes = (f - pd.DateOffset(months=1)).strftime('%Y-%m')
        for l in git(repo, 'ls-tree', '-r', '-z', sha).decode('utf-8', 'replace').split('\0'):
            if not l:
                continue
            meta, caminho = l.split('\t', 1)
            tipo, blob = meta.split()[1:3]
            if tipo == 'blob' and caminho.endswith('.py') and not EXCL.search(caminho):
                linhas.append((mes, sha, caminho, blob))
    return pd.DataFrame(linhas, columns=['mes', 'sha', 'arquivo', 'blob'])


def ler_blobs(repo, blobs):
    """4.3: lê o conteúdo de cada blob distinto, uma só vez."""
    saida = git(repo, 'cat-file', '--batch', entrada=('\n'.join(blobs) + '\n').encode())
    conteudo, i = {}, 0
    while i < len(saida):
        fim = saida.index(b'\n', i)
        sha, _, tam = saida[i:fim].decode().split()
        tam = int(tam)
        conteudo[sha] = saida[fim + 1: fim + 1 + tam].decode('utf-8', 'replace')
        i = fim + 1 + tam + 1
    return conteudo


def medir_blobs(conteudo):
    """4.4: Ruff (avisos), Radon (complexidade, SLOC) e SATD de cada blob."""
    med, avisos = {}, []
    with tempfile.TemporaryDirectory() as tmp:
        for b, src in conteudo.items():
            with open(os.path.join(tmp, b + '.py'), 'w', encoding='utf-8') as f:
                f.write(src)
        r = subprocess.run([*RUFF, 'check', '--isolated', '--no-cache', '--exit-zero',
                            '--output-format', 'json', '--select', RUFF_SELECT, '--ignore', RUFF_IGNORE, tmp],
                           capture_output=True, text=True)
        if r.returncode != 0:
            raise RuntimeError('Ruff falhou: ' + r.stderr[:300])
        achados = json.loads(r.stdout or '[]')
    for a in achados:
        b = os.path.basename(a['filename'])[:-3]
        if a['code'] is None:                       # erro de sintaxe, não é aviso de qualidade
            med.setdefault(b, {})['erro_sintaxe'] = True
            continue
        linhas = conteudo[b].splitlines()
        row = a['location']['row']
        texto = linhas[row - 1].strip() if 0 < row <= len(linhas) else ''
        avisos.append((b, a['code'], row, texto))
    for b, src in conteudo.items():
        m = med.setdefault(b, {})
        try:
            blocos = cc_visit(src)
            cc = [x.complexity for x in blocos]
            m['sloc'] = analyze(src).sloc
            m['n_funcoes'] = len(cc)
            m['cc_media'] = sum(cc) / len(cc) if cc else 0.0
            m['cc_max'] = max(cc, default=0)
            m['cc_total'] = sum(cc)
        except Exception:
            m['erro_sintaxe'] = True
        satd = 0
        try:
            for t in tokenize.generate_tokens(io.StringIO(src).readline):
                if t.type == tokenize.COMMENT and SATD.search(t.string):
                    satd += 1
        except (tokenize.TokenError, IndentationError, SyntaxError):
            pass
        m['satd'] = satd
    df = pd.DataFrame.from_dict(med, orient='index').rename_axis('blob').reset_index()
    av = pd.DataFrame(avisos, columns=['blob', 'regra', 'linha', 'texto'])
    df['avisos'] = df.blob.map(av.blob.value_counts()).fillna(0).astype(int)
    compl = av[av.regra.str.match(r'C901|PLR09')]          # avisos de complexidade
    df['avisos_complexidade'] = df.blob.map(compl.blob.value_counts()).fillna(0).astype(int)
    return df, av


def medir_repo(nome, adocao, base='..'):
    chave = nome.replace('/', '__')
    repo, saida = f'{base}/repos/{chave}', f'{base}/out/{chave}'
    os.makedirs(saida, exist_ok=True)
    snap = snapshots(repo, meses_da_janela(pd.Timestamp(adocao)))
    if snap.groupby('mes').size().max() > LIMITE_ARQUIVOS:
        print(f'{nome}: mais de {LIMITE_ARQUIVOS} arquivos .py, excluído por custo'); return None
    conteudo = ler_blobs(repo, snap.blob.unique())
    med, avisos = medir_blobs(conteudo)
    tab = snap.merge(med, on='blob')
    tab.to_parquet(f'{saida}/arquivos_mensais.parquet')
    avisos.to_parquet(f'{saida}/avisos_por_blob.parquet')
    print(f'{nome}: {snap.mes.nunique()} meses, {len(snap)} linhas, {snap.blob.nunique()} versões medidas, '
          f'{len(avisos)} avisos')
    return tab
```

A segunda célula roda os cinco projetos lendo `out/piloto.csv`. Ela usa a data do primeiro PR aceito (coluna `primeiro`) como data de adoção:

```python
piloto = pd.read_csv('../out/piloto.csv')
for _, p in piloto.iterrows():
    medir_repo(p.full_name, p.primeiro)
```

Cada projeto gera, em `out/<projeto>/`, dois arquivos: `arquivos_mensais.parquet` (uma linha por mês e arquivo, com `blob`, `sloc`, `n_funcoes`, `cc_media`, `cc_max`, `cc_total`, `avisos`, `avisos_complexidade` e `satd`) e `avisos_por_blob.parquet` (regra, linha e texto de cada aviso, para a Etapa 5). A tabela final com a data do primeiro toque de agente e humano (4.7) sai da Etapa 5, que lê `git log --first-parent --raw -M`. Referência: o numbagg dá 28 meses, 669 linhas, 141 versões medidas e 561 avisos distintos, em cerca de 10 segundos. Um projeto que passe de 5.000 arquivos `.py` em algum mês é pulado e declarado como excluído por custo.

## Etapa 5 – Rastrear os avisos introduzidos

Aqui você monta os dados da QP4: quais avisos cada alteração introduziu e quando, e por quem, eles saíram do código.

1. **Liste as alterações.** Para cada *commit* de agente após a adoção, liste os arquivos `.py` alterados com o *blob* de antes e o de depois. As duas informações vêm na saída de `git log --first-parent --raw -M --no-abbrev`.
2. **Monte a linha de base humana.** Sorteie, com semente fixa, *commits* humanos posteriores à adoção, no mesmo número dos *commits* de agentes de cada repositório, e faça o mesmo com eles.
3. **Encontre os avisos novos.** Rode o Ruff nas versões de antes e de depois e represente cada aviso pelo par (regra, texto da linha). Os avisos novos são a diferença entre as contagens: `Counter(depois) - Counter(antes)`.
4. **Siga o arquivo.** Percorra os *commits* seguintes que alteram o arquivo (`git log --first-parent --raw -M <sha>..HEAD`), acompanhando renomeações (status `R`). O aviso sai quando a contagem do seu par cai. Se o arquivo for apagado, registre a remoção como "arquivo removido".
5. **Registre quem removeu.** Guarde a data e o autor do *commit* que removeu o aviso, classificado como agente ou humano pelas regras da Etapa 3. Avisos que continuam no código ficam censurados na data da sua coleta.

Gere duas tabelas:

| Tabela | Uma linha por | Colunas principais |
| --- | --- | --- |
| `alteracoes` | (*commit*, arquivo) rastreado | `repo, sha, data, autor_agente, agente, caminho, sloc_antes, sloc_depois` |
| `avisos_introduzidos` | aviso novo | `repo, sha, data, autor_agente, agente, caminho, regra, data_remocao, como, removido_por_agente` |

A primeira serve de denominador (linhas adicionadas, número de alterações), e a segunda guarda os eventos. Uma limitação para declarar no artigo: editar a linha de um aviso conta como remoção.

**5.1 Fazer a Etapa 5 no notebook (piloto).** Crie `estudo/notebooks/04_avisos.ipynb`, com o *kernel* Python (agentes). Ele precisa de `out/<projeto>/commits_atribuidos.csv`, gerado no 3.7. O código lista as alterações (`git log --first-parent --raw -M`), monta a linha de base humana com semente 42, roda o Ruff antes e depois de cada alteração, segue o arquivo nos *commits* seguintes e registra quem removeu cada aviso. A célula 1 só define as funções:

```python
import bisect, importlib.util, json, os, re, subprocess, sys, tempfile
from collections import Counter, defaultdict
import pandas as pd
from radon.raw import analyze

RUFF_SELECT = 'F,E,B,SIM,C90,PLE,PLW,PLR,PLC,ERA'
RUFF_IGNORE = 'E501,PLR2004,PLC0415'
EXCL = re.compile(r'(^|/)(\.venv|venv|env|site-packages|node_modules|vendor|third_party|build|dist|\.tox)/')
RUFF = [sys.executable, '-m', 'ruff'] if importlib.util.find_spec('ruff') else ['ruff']
SEMENTE = 42


def git(repo, *args, entrada=None):
    r = subprocess.run(['git', '-C', repo, *args], capture_output=True, input=entrada)
    if r.returncode != 0:
        raise RuntimeError(r.stderr.decode('utf-8', 'replace')[:300])
    return r.stdout


def eh_py(p):
    return p.endswith('.py') and not EXCL.search(p)


def sloc_de(src):
    try:
        return analyze(src).sloc
    except Exception:
        return sum(1 for l in src.splitlines() if l.strip() and not l.strip().startswith('#'))


def alteracoes_por_commit(repo, desde):
    """Passo 1: para cada commit do first-parent depois de `desde`, os arquivos alterados (com blob antes/depois)."""
    bruto = git(repo, 'log', '--first-parent', '-M', '--raw', '--no-abbrev', '--format=%x1e%H',
                f'{desde}..HEAD' if desde else 'HEAD').decode('utf-8', 'replace')
    mud = {}
    for rec in bruto.split('\x1e')[1:]:
        linhas = rec.strip().splitlines()
        sha, lista = linhas[0].strip(), []
        for l in linhas[1:]:
            if not l.startswith(':'):
                continue
            meta, *caminhos = l.split('\t')
            f = meta.split()
            antes, depois, st = f[2], f[3], f[4]
            velho = caminhos[0]
            novo = caminhos[1] if len(caminhos) > 1 else caminhos[0]
            lista.append((st[0], velho, novo, None if set(antes) == {'0'} else antes,
                          None if set(depois) == {'0'} else depois))
        mud[sha] = lista
    return mud


def ler_blobs(repo, blobs):
    blobs = sorted(blobs)
    saida = git(repo, 'cat-file', '--batch', entrada=('\n'.join(blobs) + '\n').encode())
    conteudo, i = {}, 0
    while i < len(saida):
        fim = saida.index(b'\n', i)
        sha, _, tam = saida[i:fim].decode().split()
        tam = int(tam)
        conteudo[sha] = saida[fim + 1: fim + 1 + tam].decode('utf-8', 'replace')
        i = fim + 1 + tam + 1
    return conteudo


def avisos_por_blob(conteudo):
    """Ruff em todos os blobs; cada aviso vira o par (regra, texto da linha)."""
    res = defaultdict(list)
    with tempfile.TemporaryDirectory() as tmp:
        for b, src in conteudo.items():
            with open(os.path.join(tmp, b + '.py'), 'w', encoding='utf-8') as f:
                f.write(src)
        r = subprocess.run([*RUFF, 'check', '--isolated', '--no-cache', '--exit-zero', '--output-format', 'json',
                            '--select', RUFF_SELECT, '--ignore', RUFF_IGNORE, tmp], capture_output=True, text=True)
        if r.returncode != 0:
            raise RuntimeError('Ruff falhou: ' + r.stderr[:300])
        for a in json.loads(r.stdout or '[]'):
            if a['code'] is None:
                continue
            b = os.path.basename(a['filename'])[:-3]
            linhas = conteudo[b].splitlines()
            row = a['location']['row']
            res[b].append((a['code'], linhas[row - 1].strip() if 0 < row <= len(linhas) else ''))
    return res


def rastrear(nome, base='..'):
    chave = nome.replace('/', '__')
    repo, saida = f'{base}/repos/{chave}', f'{base}/out/{chave}'
    h = pd.read_csv(f'{saida}/commits_atribuidos.csv')
    h['data'] = pd.to_datetime(h.data, utc=True)
    h = h.sort_values('data').reset_index(drop=True)
    adocao = h[h.is_agente].data.min()
    antes_adocao = h[h.data < adocao]
    mud = alteracoes_por_commit(repo, antes_adocao.iloc[-1].sha if len(antes_adocao) else None)
    pos = {s: i for i, s in enumerate(h.sha)}
    data_de, ag_de, nome_ag = dict(zip(h.sha, h.data)), dict(zip(h.sha, h.is_agente)), dict(zip(h.sha, h.agente))

    # (commit, arquivo) alterados depois da adoção
    linhas = []
    for c in h[h.data >= adocao].itertuples():
        for st, velho, novo, antes, depois in mud.get(c.sha, []):
            if eh_py(novo) and st != 'D':
                linhas.append((c.sha, c.data, novo, bool(c.is_agente), c.agente, antes, depois))
    toque = pd.DataFrame(linhas, columns=['sha', 'data', 'caminho', 'is_agente', 'agente', 'antes', 'depois'])
    # 2. linha de base humana: mesmo número de commits, sorteados com semente fixa
    ag_shas = toque[toque.is_agente].sha.unique()
    hum = toque[~toque.is_agente]
    hum_shas = pd.Series(hum.sha.unique())
    if len(hum_shas):
        hum_shas = hum_shas.sample(n=min(len(hum_shas), max(len(ag_shas), 10)), random_state=SEMENTE)
    sel = pd.concat([toque[toque.is_agente], hum[hum.sha.isin(set(hum_shas))]]).drop_duplicates(['sha', 'caminho'])
    rastro = [(t.sha, t.caminho, t.antes if isinstance(t.antes, str) else None,
               t.depois if isinstance(t.depois, str) else None) for t in sel.itertuples()]

    # 4. seguir o arquivo nos commits seguintes (com renomeações)
    eventos = defaultdict(list)
    for sha, lista in mud.items():
        if sha in pos:
            for st, velho, novo, antes, depois in lista:
                eventos[velho].append((pos[sha], st, novo, depois, sha))
    for k in eventos:
        eventos[k].sort()

    def seguir(sha_intro, caminho):
        atual, i, saida = caminho, pos[sha_intro], []
        while True:
            ev = eventos.get(atual, [])
            j = bisect.bisect_right([e[0] for e in ev], i)
            mudou = False
            for pi, st, novo, depois, sha in ev[j:]:
                if st == 'D':
                    saida.append((sha, None)); return saida
                if st == 'R' and novo != atual:
                    saida.append((sha, depois)); atual, i, mudou = novo, pi, True
                    break
                saida.append((sha, depois)); i = pi
            if not mudou:
                return saida

    seg = {(s, p): seguir(s, p) for s, p, _, d in rastro if d}
    todos = {b for r in rastro for b in r[2:] if b} | {b for v in seg.values() for _, b in v if b}
    conteudo = ler_blobs(repo, todos)
    av = avisos_por_blob(conteudo)

    # 3 e 5. avisos novos e quem os removeu
    introduzidos, alter = [], []
    for sha, caminho, antes, depois in rastro:
        ca = Counter(av[depois]) if depois else Counter()
        cb = Counter(av[antes]) if antes else Counter()
        alter.append(dict(repo=nome, sha=sha, data=data_de[sha], autor_agente=bool(ag_de[sha]), agente=nome_ag[sha],
                          caminho=caminho, sloc_antes=sloc_de(conteudo[antes]) if antes else 0,
                          sloc_depois=sloc_de(conteudo[depois]) if depois else 0,
                          avisos_antes=sum(cb.values()), avisos_depois=sum(ca.values())))
        for chave_av, n in (ca - cb).items():
            for k in range(n):
                rem, como = None, None
                for s, b in seg.get((sha, caminho), []):
                    if b is None:
                        rem, como = s, 'arquivo removido'; break
                    if Counter(av[b])[chave_av] < ca[chave_av] - k:
                        rem, como = s, 'aviso corrigido'; break
                introduzidos.append(dict(
                    repo=nome, sha=sha, data=data_de[sha], autor_agente=bool(ag_de[sha]), agente=nome_ag[sha],
                    caminho=caminho, regra=chave_av[0], data_remocao=data_de.get(rem), como=como,
                    removido_por_agente=ag_de.get(rem) if rem else None))
    pd.DataFrame(alter).to_csv(f'{saida}/alteracoes.csv', index=False)
    pd.DataFrame(introduzidos).to_csv(f'{saida}/avisos_introduzidos.csv', index=False)
    print(f'{nome}: {len(alter)} alterações rastreadas ({len(ag_shas)} commits de agente), '
          f'{len(introduzidos)} avisos introduzidos')
    return pd.DataFrame(alter), pd.DataFrame(introduzidos)
```

A célula 2 roda os cinco projetos, grava as duas tabelas de cada um em `out/<projeto>/` (`alteracoes.csv` e `avisos_introduzidos.csv`) e as versões juntas em `out/alteracoes.csv` e `out/avisos_introduzidos.csv`. Leva cerca de 1 a 2 minutos:

```python
piloto = pd.read_csv('../out/piloto.csv')
res = [rastrear(p.full_name) for _, p in piloto.iterrows()]
alteracoes = pd.concat([a for a, _ in res], ignore_index=True)
avisos_introduzidos = pd.concat([b for _, b in res], ignore_index=True)
alteracoes.to_csv('../out/alteracoes.csv', index=False)
avisos_introduzidos.to_csv('../out/avisos_introduzidos.csv', index=False)
avisos_introduzidos.groupby(['repo', 'autor_agente']).size().unstack(fill_value=0)
```

**Referência do piloto** (avisos introduzidos por *commits* de agentes e de humanos):

| Projeto | Por agentes | Por humanos |
| --- | --- | --- |
| numbagg | 34 | 13 |
| HARK | 2 | 41 |
| DDNS | 122 | 149 |
| diffyscan | 4 | 21 |
| PyAirbyte | 162 | 11 |

Os números devem bater exatamente, tanto os de **agentes** quanto os de **humanos**. Os de humanos vêm de uma amostra sorteada com semente fixa (42), então o resultado se repete em outras máquinas, desde que o clone tenha os mesmos *commits*. Se os projetos receberem *commits* novos, a amostra humana pode mudar. Declare a semente no artigo. A coluna `como` traz `aviso corrigido`, `arquivo removido` ou vazio quando o aviso continua no código (censurado na data da coleta). Nos cinco projetos, 120 avisos foram corrigidos, 8 saíram porque o arquivo foi apagado e 431 continuam.

## Etapa 6 – Análises de QP1 a QP4

Todas as análises partem das tabelas das Etapas 4 e 5. Normalize tudo pelo tamanho (por KLOC ou por função) e use o repositório como unidade de reamostragem nos intervalos de confiança.

**No piloto, com 5 projetos,** trate os resultados como descritivos. Mostre os valores de cada projeto lado a lado, em vez de só a mediana, e não tire conclusões dos valores-p. Um *bootstrap* com 5 repositórios dá intervalos pouco confiáveis, e o estimador de Callaway e Sant'Anna precisa de mais coortes: deixe os dois para a amostra completa. O objetivo do piloto é garantir que cada notebook roda do início ao fim e que os números fazem sentido quando você olha os *commits* no GitHub.

**QP1 – Trajetória.**

1. Some a tabela mensal por (repositório, mês) e calcule avisos/KLOC, avisos de complexidade/KLOC, CC média por função e SATD/KLOC.
2. Crie `rel` = mês menos o mês da adoção e divida cada métrica pelo seu valor em `rel = -1`.
3. Faça o gráfico da mediana e do intervalo interquartil por `rel`. Use a mediana porque, com poucos repositórios, alguns valores extremos puxam a média.
4. Compare as médias de cada repositório nos intervalos −6 a −2, 0 a 5, 6 a 11 e 12 a 15 com o mês −1, usando o teste de Wilcoxon.
5. Ajuste a regressão segmentada:

```python
import statsmodels.formula.api as smf
# d = painel (projeto, mês) da célula 1 do item 6.1, só com rel >= -12 (rode essa célula antes)
d = P[P.rel >= -12].dropna(subset=['avisos_kloc']).copy()
d['post'] = (d.rel >= 0).astype(int)
d['t_post'] = d.rel.clip(lower=0)
m = smf.ols('avisos_kloc ~ C(repo) + rel + post + t_post', data=d).fit(
    cov_type='cluster', cov_kwds={'groups': d.repo})
print(m.params[['rel', 'post', 't_post']], m.conf_int().loc[['post', 't_post']])
```

6. Repita com `log(cc_soma)` controlando por `log(sloc)`. Esse modelo mostra se a complexidade cresce mais rápido que o tamanho.
7. Para o curto prazo, use o pacote `did` do R, com `att_gt(..., control_group = "notyettreated")`. Como quase todos adotaram entre maio e julho de 2025, espere poucos controles a partir do mês 2.

**QP2 – Proveniência.**

1. Classifique os arquivos: existentes alterados por agente (A), existentes alterados só por humanos (H), não alterados (U), e novos criados por agente ou por humano.
2. Para cada grupo, calcule a soma da variação de avisos dividida pela soma das linhas adicionadas, vezes mil.
3. Obtenha o intervalo de confiança reamostrando repositórios mil vezes, com reposição.
4. Calcule a linha de base com os arquivos alterados entre os meses −12 e −1.
5. Rode a regressão por arquivo `variacao_avisos ~ A + variacao_kloc + C(repo)`. Como verificação, os arquivos U devem ter variação perto de zero.

**QP3 – Agente e intensidade.**

1. A intensidade é a fração dos *commits* pós-adoção que alteram arquivos Python feitos por agentes.
2. Calcule a correlação de Spearman entre a intensidade e a variação percentual de cada métrica (média dos meses 9 a 12 contra o mês −1).
3. Na tabela `alteracoes`, calcule os avisos novos por KLOC adicionada de cada agente e dos humanos, com intervalo por *bootstrap* de repositórios.

**QP4 – Persistência.**

```python
from lifelines import KaplanMeierFitter
from lifelines.statistics import logrank_test
# dur = dias até a remoção ou até a coleta; evento = 1 se removido
for g, x in df.groupby('autor_agente'):
    KaplanMeierFitter().fit(x.dur, x.evento, label=str(g)).plot_survival_function()
print(logrank_test(a.dur, h.dur, a.evento, h.evento).p_value)
```

Relate S(90), S(365), a fração ainda presente no fim, a mediana de dias até a remoção, a divisão entre correção e arquivo apagado e quem removeu. Repita o cálculo por família de regra: PLR, F, ERA, C90 e B.

**6.1 Fazer a Etapa 6 no notebook (piloto).** Crie `estudo/notebooks/05_analises.ipynb`, com o *kernel* Python (agentes). Ele usa as saídas das etapas anteriores (`arquivos_mensais.parquet`, `commits_atribuidos.csv`, `alteracoes.csv`, `avisos_introduzidos.csv`, `atribuicao_resumo.csv`) e os clones em `repos/`. Como são 5 projetos, o código só descreve: mostra o valor de cada projeto e os totais, sem testes de significância, *bootstrap* nem regressão. Use uma célula para cada bloco abaixo, na ordem.

**Célula 1 – carregar os dados.** Lê as tabelas mensais, calcula o mês relativo à adoção (`rel = 0` é o mês da adoção) e monta o painel (projeto, mês) com avisos, complexidade e SATD por tamanho:

```python
import os, re, subprocess
import numpy as np, pandas as pd
import matplotlib.pyplot as plt

os.makedirs('../out/figs', exist_ok=True)
piloto = pd.read_csv('../out/piloto.csv')
REPOS = list(piloto.full_name)
resumo = pd.read_csv('../out/atribuicao_resumo.csv').set_index('projeto')
adocao = pd.to_datetime(resumo.adocao, utc=True)            # primeiro commit de agente, por projeto
chave = lambda n: n.replace('/', '__')
curto = lambda n: n.split('/')[1]

# arquivos por mês (Etapa 4) com o mês relativo à adoção: rel = 0 é o mês da adoção
F = []
for n in REPOS:
    f = pd.read_parquet(f'../out/{chave(n)}/arquivos_mensais.parquet')
    f['repo'] = n
    a = adocao[n]
    f['rel'] = (f.mes.str[:4].astype(int) * 12 + f.mes.str[5:].astype(int)) - (a.year * 12 + a.month)
    F.append(f)
F = pd.concat(F, ignore_index=True)

# painel (projeto, mês): somas e densidades
P = F.groupby(['repo', 'rel']).agg(sloc=('sloc', 'sum'), avisos=('avisos', 'sum'), compl=('avisos_complexidade', 'sum'),
                                   satd=('satd', 'sum'), cc=('cc_total', 'sum'), nf=('n_funcoes', 'sum')).reset_index()
P['avisos_kloc'] = P.avisos / P.sloc * 1000
P['compl_kloc'] = P.compl / P.sloc * 1000
P['satd_kloc'] = P.satd / P.sloc * 1000
P['cc_func'] = P.cc / P.nf.replace(0, np.nan)
METRICAS = {'avisos_kloc': 'Avisos por KLOC', 'cc_func': 'Complexidade média por função', 'satd_kloc': 'SATD por KLOC'}
print('Projetos:', len(REPOS), '| meses por projeto:', P.groupby('repo').size().to_dict())
```

**Célula 2 – QP1, trajetória.** Gera a tabela da variação de cada métrica em quatro faixas de meses, em relação ao mês −1, e o gráfico `out/figs/qp1_trajetorias.png`, com uma linha por projeto. O DDNS não tem nenhum SATD no mês −1, então a razão não existe e aparece como `-`:

```python
base = P[P.rel == -1].set_index('repo')
faixas = [(-6, -2, '-6 a -2'), (0, 5, '0 a 5'), (6, 11, '6 a 11'), (12, 15, '12 a 15')]
linhas = []
for y, nome in METRICAS.items():
    for repo in REPOS:
        d = P[P.repo == repo]
        linha = {'métrica': nome, 'projeto': curto(repo), 'mês -1': round(base.loc[repo, y], 2)}
        for lo, hi, rot in faixas:
            m = d[(d.rel >= lo) & (d.rel <= hi)][y].mean()
            linha[rot] = f'{(m / base.loc[repo, y] - 1) * 100:+.0f}%' if pd.notna(m) and base.loc[repo, y] > 0 else '-'
        linhas.append(linha)
qp1 = pd.DataFrame(linhas)
qp1.to_csv('../out/qp1_faixas.csv', index=False)

fig, eixos = plt.subplots(1, 3, figsize=(12, 3.2), sharex=True)
cor = dict(zip(REPOS, plt.rcParams['axes.prop_cycle'].by_key()['color']))   # uma cor fixa por projeto
for ax, (y, nome) in zip(eixos, METRICAS.items()):
    for repo in REPOS:
        d = P[(P.repo == repo) & (P.rel >= -12) & (P.rel <= 15)]
        if base.loc[repo, y] > 0:                      # sem valor de base (zero), a razão não existe
            ax.plot(d.rel, d[y] / base.loc[repo, y], label=curto(repo), lw=1.4, color=cor[repo])
    ax.axvline(-0.5, color='gray', ls=':'); ax.axhline(1, color='black', lw=0.6)
    ax.set_title(nome); ax.set_xlabel('Meses desde a adoção')
eixos[0].set_ylabel('Razão em relação ao mês -1'); eixos[0].legend(frameon=False, fontsize=8)
fig.tight_layout(); fig.savefig('../out/figs/qp1_trajetorias.png', dpi=200)
qp1
```

**Célula 3 – QP2, proveniência.** Classifica cada arquivo como existente alterado por agente (A), existente alterado só por humanos (H), inalterado (U) ou novo, criado por agente ou por humano, e calcula os avisos novos por KLOC adicionada. Usa `git log --raw` para saber quem tocou cada arquivo. A soma da variação nos arquivos U deve ser zero, porque arquivo que não muda não muda de avisos:

```python
def toques(repo_dir, desde):
    """Arquivos .py alterados em cada commit do first-parent depois de `desde` (git log --raw -M)."""
    r = subprocess.run(['git', '-C', repo_dir, 'log', '--first-parent', '-M', '--raw', '--no-abbrev', '--format=%x1e%H',
                        f'{desde}..HEAD' if desde else 'HEAD'], capture_output=True)
    saida = {}
    for rec in r.stdout.decode('utf-8', 'replace').split('\x1e')[1:]:
        linhas = rec.strip().splitlines()
        caminhos = []
        for l in linhas[1:]:
            if l.startswith(':'):
                meta, *c = l.split('\t')
                if meta.split()[4][0] != 'D':
                    caminhos.append(c[-1])
        saida[linhas[0].strip()] = caminhos
    return saida

EXCL = re.compile(r'(^|/)(\.venv|venv|env|site-packages|node_modules|vendor|third_party|build|dist|\.tox)/')
linhas, intensidade = [], {}
for repo in REPOS:
    h = pd.read_csv(f'../out/{chave(repo)}/commits_atribuidos.csv')
    h['data'] = pd.to_datetime(h.data, utc=True)
    h = h.sort_values('data').reset_index(drop=True)
    antes = h[h.data < adocao[repo]]
    mud = toques(f'../repos/{chave(repo)}', antes.iloc[-1].sha if len(antes) else None)
    pos = h[h.data >= adocao[repo]]
    ag_path, hu_path, n_py, n_ag = {}, {}, 0, 0
    for c in pos.itertuples():
        py = [p for p in mud.get(c.sha, []) if p.endswith('.py') and not EXCL.search(p)]
        if not py:
            continue
        n_py += 1; n_ag += int(c.is_agente)
        for p in py:
            (ag_path if c.is_agente else hu_path).setdefault(p, c.data)
    intensidade[repo] = dict(commits_py=n_py, commits_agente=n_ag, intensidade=round(n_ag / n_py, 3) if n_py else np.nan)

    f = F[F.repo == repo]
    pre = f[f.rel == -1].set_index('arquivo')
    ult = f.sort_values('rel').groupby('arquivo').tail(1).set_index('arquivo')
    for caminho, l in ult.iterrows():
        ag, hu = caminho in ag_path, caminho in hu_path
        if caminho in pre.index:
            b = pre.loc[caminho]
            grupo = 'A' if ag else ('H' if hu else 'U')
            tipo = 'pre-existente'
            d = dict(d_sloc=l.sloc - b.sloc, d_avisos=l.avisos - b.avisos, d_cc=l.cc_total - b.cc_total)
        elif l.rel >= 0:
            grupo = 'A' if (ag and (not hu or ag_path[caminho] <= hu_path[caminho])) else 'H'
            tipo = 'novo'
            d = dict(d_sloc=l.sloc, d_avisos=l.avisos, d_cc=l.cc_total)
        else:
            continue
        linhas.append(dict(repo=repo, arquivo=caminho, tipo=tipo, grupo=grupo, **d))
Q2 = pd.DataFrame(linhas)

# linha de base pré-adoção: arquivos que existiam em rel=-12 e -1 e mudaram (só humanos, por definição)
pre_l = []
for repo in REPOS:
    f = F[F.repo == repo]
    ini = f[f.rel == f.rel.min()].set_index('arquivo'); fim = f[f.rel == -1].set_index('arquivo')
    comum = ini.index.intersection(fim.index)
    mud_ = comum[(fim.loc[comum, 'blob'] != ini.loc[comum, 'blob']).values]
    d = (fim.loc[mud_, ['sloc', 'avisos', 'cc_total']] - ini.loc[mud_, ['sloc', 'avisos', 'cc_total']])
    d.columns = ['d_sloc', 'd_avisos', 'd_cc']; d['repo'] = repo; d['grupo'] = 'pre'
    pre_l.append(d)
PRE = pd.concat(pre_l)

def taxa(df, col):
    """variação por KLOC adicionada: soma da variação / soma das linhas adicionadas (só arquivos que cresceram)."""
    d = df[df.d_sloc > 0]
    return round(d[col].sum() / d.d_sloc.sum() * 1000, 1) if d.d_sloc.sum() else np.nan

grupos = [('Pré-adoção: alterados', PRE), ('Pós: existentes, só humanos (H)', Q2[(Q2.tipo == 'pre-existente') & (Q2.grupo == 'H')]),
          ('Pós: existentes, tocados por agente (A)', Q2[(Q2.tipo == 'pre-existente') & (Q2.grupo == 'A')]),
          ('Pós: novos, criados por humanos', Q2[(Q2.tipo == 'novo') & (Q2.grupo == 'H')]),
          ('Pós: novos, criados por agente', Q2[(Q2.tipo == 'novo') & (Q2.grupo == 'A')])]
qp2 = pd.DataFrame([{'grupo': nome, 'arquivos': len(d), 'KLOC adicionada': round(d[d.d_sloc > 0].d_sloc.sum() / 1000, 2),
                     'avisos novos por KLOC': taxa(d, 'd_avisos'), 'complexidade nova por KLOC': taxa(d, 'd_cc')}
                    for nome, d in grupos])
por_proj = pd.DataFrame({curto(r): {nome: taxa(d[d.repo == r], 'd_avisos') for nome, d in grupos} for r in REPOS})
print('Sanidade: soma da variação de avisos nos arquivos U (deve ser perto de 0):',
      Q2[Q2.grupo == 'U'].d_avisos.sum())
qp2.to_csv('../out/qp2_taxas.csv', index=False); por_proj.to_csv('../out/qp2_por_projeto.csv')
display(qp2); display(por_proj)
```

**Célula 4 – QP3, agente e intensidade.** A intensidade é a fração dos *commits* depois da adoção que alteram arquivos Python e foram feitos por agente. A tabela mostra, para cada projeto, a intensidade e a variação das métricas nos meses 9 a 12 em relação ao mês −1. Com 5 projetos, não calcule a correlação de Spearman. Em seguida, a taxa de avisos novos por KLOC adicionada de cada agente e dos humanos:

```python
I = pd.DataFrame(intensidade).T
for y, nome in METRICAS.items():
    fim = P[(P.rel >= 9) & (P.rel <= 12)].groupby('repo')[y].mean()
    I[f'var % {y}'] = ((fim / base[y] - 1) * 100).round(0)
I['agente principal'] = resumo.agente_principal
I.index = [curto(r) for r in I.index]
I.to_csv('../out/qp3_intensidade.csv')
display(I)   # com 5 projetos, observe os pares lado a lado; não calcule correlação de Spearman

A = pd.read_csv('../out/alteracoes.csv'); N = pd.read_csv('../out/avisos_introduzidos.csv')
novos = N.groupby(['repo', 'sha', 'caminho']).size().rename('novos').reset_index()
A = A.merge(novos, on=['repo', 'sha', 'caminho'], how='left').fillna({'novos': 0})
A['ator'] = np.where(A.autor_agente, A.agente, 'Humano')
A['d_sloc'] = (A.sloc_depois - A.sloc_antes).clip(lower=0)
qp3 = A.groupby('ator').agg(alteracoes=('novos', 'size'), avisos_novos=('novos', 'sum'), kloc_adicionada=('d_sloc', lambda s: s.sum() / 1000))
qp3['avisos novos por KLOC'] = (qp3.avisos_novos / qp3.kloc_adicionada).round(1)
qp3.to_csv('../out/qp3_por_agente.csv')
qp3
```

**Célula 5 – QP4, permanência.** Curva de Kaplan–Meier dos avisos introduzidos por agentes e por humanos (`out/figs/qp4_sobrevivencia.png`), com a fração ainda presente em 90 e 365 dias, a mediana de dias até a remoção, a divisão entre correção e arquivo apagado e quem removeu. A data de coleta é a data do último *commit* dos clones:

```python
from lifelines import KaplanMeierFitter
from lifelines.statistics import logrank_test

N['data'] = pd.to_datetime(N.data, utc=True)
N['data_remocao'] = pd.to_datetime(N.data_remocao, utc=True)
COLETA = max(pd.read_csv(f'../out/{chave(r)}/commits_atribuidos.csv', usecols=['data']).data.pipe(pd.to_datetime, utc=True).max()
             for r in REPOS)                                  # data da coleta: o último commit dos clones
N['dias'] = ((N.data_remocao.fillna(COLETA) - N.data).dt.total_seconds() / 86400).clip(lower=0)
N['removido'] = N.data_remocao.notna().astype(int)
N['familia'] = N.regra.str.extract(r'^([A-Z]+)')[0]
print('Coleta em', COLETA.date())

fig, ax = plt.subplots(figsize=(5, 3.2))
linhas = []
for rot, g in [('Introduzidos por agentes', N[N.autor_agente]), ('Introduzidos por humanos', N[~N.autor_agente])]:
    km = KaplanMeierFitter().fit(g.dias, g.removido, label=f'{rot} (n={len(g)})')
    km.plot_survival_function(ax=ax, ci_show=False)
    rem = g[g.removido == 1]
    linhas.append({'grupo': rot, 'avisos': len(g), 'projetos': g.repo.nunique(), 'removidos': len(rem),
                   'ainda presentes': f'{(1 - g.removido.mean()) * 100:.0f}%',
                   'S(90 dias)': round(float(km.predict(90)), 2), 'S(365 dias)': round(float(km.predict(365)), 2),
                   'mediana de dias (removidos)': round(rem.dias.median(), 0) if len(rem) else None,
                   '% corrigidos (resto: arquivo apagado)': f'{(rem.como == "aviso corrigido").mean() * 100:.0f}%' if len(rem) else None,
                   '% removidos por agente': f'{rem.removido_por_agente.astype(float).mean() * 100:.0f}%' if len(rem) else None})
ax.set_xlabel('Dias desde a introdução'); ax.set_ylabel('Fração ainda presente'); ax.set_ylim(0, 1.02)
fig.tight_layout(); fig.savefig('../out/figs/qp4_sobrevivencia.png', dpi=200)
qp4 = pd.DataFrame(linhas)
qp4.to_csv('../out/qp4_resumo.csv', index=False)
a, h = N[N.autor_agente], N[~N.autor_agente]
print('Log-rank (valor-p NÃO interpretável com 5 projetos, só para conferir que roda):',
      round(logrank_test(a.dias, h.dias, a.removido, h.removido).p_value, 4))
display(qp4)
display(pd.crosstab(N.familia, N.autor_agente.map({True: 'agente', False: 'humano'})).sort_values('agente', ascending=False).head(8))
```

**Referência do piloto** (minha execução, com os mesmos clones; os números devem repetir, exceto a data de coleta e o QP4 se você atualizou os clones depois).

*QP2 – avisos novos por KLOC adicionada:*

| Grupo | Arquivos | KLOC adicionada | Avisos novos por KLOC | Complexidade nova por KLOC |
| --- | --- | --- | --- | --- |
| Pré-adoção: alterados | 79 | 1,52 | 36,7 | 150,8 |
| Pós: existentes, só humanos (H) | 71 | 1,75 | 0,6 | 137,4 |
| Pós: existentes, tocados por agente (A) | 128 | 8,36 | 8,1 | 184,4 |
| Pós: novos, criados por humanos | 168 | 42,96 | 13,1 | 199,7 |
| Pós: novos, criados por agente | 156 | 39,38 | 5,0 | 166,2 |

*QP3 – intensidade (fração dos* commits *de Python feita por agente):* numbagg 0,28; HARK 0,09; DDNS 0,54; diffyscan 0,11; PyAirbyte 0,82.

*QP3 – avisos novos por KLOC adicionada, por autor:*

| Autor | Alterações | Avisos novos | KLOC adicionada | Avisos novos por KLOC |
| --- | --- | --- | --- | --- |
| Humano | 443 | 235 | 17,77 | 13,2 |
| Devin | 776 | 162 | 34,89 | 4,6 |
| Copilot | 279 | 122 | 8,43 | 14,5 |
| Claude Code | 86 | 35 | 2,90 | 12,1 |
| OpenAI Codex | 19 | 5 | 0,11 | 46,3 |
| Cursor | 6 | 0 | 0,02 | 0,0 |

*QP4 – permanência dos avisos introduzidos:*

| Grupo | Avisos | Removidos | Ainda presentes | S(90) | S(365) | Mediana de dias | Corrigidos | Removidos por agente |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Agentes | 324 | 83 | 74% | 0,78 | 0,74 | 7 | 90% | 64% |
| Humanos | 235 | 45 | 81% | 0,81 | 0,77 | 12 | 100% | 31% |

**Como ler estes números.** O piloto serve para ver se o método roda e se os números fazem sentido; não dá para concluir nada com 5 projetos. O PyAirbyte (Devin) responde por 871 das 1.609 alterações rastreadas, e por isso domina os totais; olhe sempre a tabela por projeto. A taxa do OpenAI Codex (46,3) vem de só 0,11 KLOC adicionada e não é estável. O valor-p do log-rank (0,15) só confirma que a conta roda, não deve ser interpretado. As comparações estatísticas ficam para os 99 projetos.

## Etapa 7 – Checklist de verificações

Marque cada item antes de escrever os resultados.

- [ ] As contagens da seleção batem: 2.807, 530, 315 e 99 repositórios.
- [ ] Validei à mão 30 *commits* de agentes e 30 humanos e anotei a precisão.
- [ ] Nenhuma data de adoção é anterior a março de 2024 ou posterior a agosto de 2025.
- [ ] Investiguei todos os repositórios com adoção fora de maio a julho de 2025.
- [ ] A série mensal de SLOC não tem saltos inexplicados. Se tiver, procurei por código de terceiros importado.
- [ ] Os arquivos não alterados têm variação de avisos perto de zero.
- [ ] Abri no GitHub 10 avisos marcados como removidos e confirmei a remoção no *diff*.
- [ ] Fixei as sementes aleatórias e anotei as versões de Git, Ruff, Radon e Python.
- [ ] Rodei tudo de novo do zero no piloto e obtive os mesmos números.

## Etapa 8 – Dos resultados ao texto

Cada saída da análise tem um lugar no artigo:

| Saída | Vai para |
| --- | --- |
| Exclusões por critério | Tabela 4 (seleção da amostra) |
| Medianas no mês −1 e no fim da janela | Tabela 6 (evolução dos repositórios) |
| Gráfico de medianas por mês relativo | Figura 2 (QP1) |
| Coeficientes da regressão segmentada | Tabela 7 (QP1) |
| Taxas por proveniência | Figura 3 (QP2) |
| Avisos novos por KLOC, por agente | Figura 4 (QP3) |
| Curvas de Kaplan–Meier e indicadores | Figura 5 e Tabela 8 (QP4) |
| Precisão da validação manual (se feita) e decisões do diário de pesquisa | Seção 3.2 e ameaças à validade |

Termine cada QP com uma resposta de uma ou duas frases e um número. Só use linguagem causal ("a adoção causou") quando o desenho permitir. Nas séries interrompidas, prefira "após a adoção, a inclinação mudou". Se seus números ficarem longe dos do pacote de replicação, o problema costuma estar na atribuição (Etapa 3) ou na janela de meses (Etapa 4).

## Do piloto para os 99 projetos

Quando os cinco projetos passarem por todas as etapas e pelo checklist, a escala muda o volume de trabalho, mas não o método:

| Aspecto | Piloto (5) | Amostra completa (99 candidatos) |
| --- | --- | --- |
| Lista lida pelos notebooks | `out/piloto.csv` | `out/candidatos.csv` |
| Disco | menos de 2 GB | cerca de 25 GB durante a coleta. Apague cada clone depois de medir |
| Tempo de máquina | minutos | algumas horas. Repositórios grandes levam de 10 a 30 minutos cada |
| Execução | células do notebook | mova a medição para um script `.py` rodado no terminal, um projeto por vez, gravando `out/<repo>/` e pulando os que já têm saída |
| Critérios de inclusão | todos os 5 já os atendem | aplique os quatro critérios da Etapa 3.5 e anote as exclusões |
| Estatística | descritiva | regressão segmentada, *bootstrap* e Callaway e Sant'Anna |
| Validação manual | todos os *commits* de agentes dos 5 | amostra de 30 de agentes e 30 humanos |

No artigo, descreva o piloto na metodologia como a etapa que validou o *pipeline*, e informe as correções que ele motivou.

## Armadilhas que encontrei e como evitar

Todos estes problemas apareceram na primeira versão do estudo e mudaram resultados antes de serem corrigidos.

- **Coautoria "Copilot" sem ser agente:** quando um humano aceita uma sugestão de revisão do Copilot, o *commit* ganha a coautoria `175728472+Copilot`. Isso antecipava a adoção em dezenas de projetos.
- **Pessoas chamadas Claude:** `Co-Authored-By: Claude` aparece em projetos antigos, como no Django. Exija o e-mail `noreply@anthropic.com`.
- **Número de PR repetido:** em um *fork* do Django, um *commit* de 2012 tinha no título o mesmo número de um PR de agente. O filtro de datas da Etapa 3.3 resolve isso.
- **Merge por *squash*:** o PR inteiro vira um *commit* atribuído ao agente, inclusive ajustes feitos por humanos dentro dele. Declare isso como limitação.
- **Memória e tempo:** repositórios grandes, como mlflow e airbyte, levam de 10 a 30 minutos e podem passar de 4 GB de RAM. Meça um por vez e defina um limite de tempo.
- **Clones em paralelo:** dois processos clonando o mesmo repositório corrompem a pasta. Clone e meça em sequência, um repositório por vez.
- **Configuração do projeto:** sem `--isolated`, o Ruff lê o `pyproject.toml` de cada projeto, e as medições deixam de ser comparáveis.
- **Fusos horários:** converta todas as datas para UTC antes de comparar ou agrupar por mês.
- **Versão do AIDev:** fixe a v3. Versões mais novas mudam a lista de repositórios e de PRs.
- **Hugging Face bloqueado:** se o download por script falhar, baixe os três arquivos pelo navegador, como na Etapa 2.

## Fontes

- [AIDev v3 no Hugging Face](https://huggingface.co/datasets/hao-li/AIDev/tree/v3)
- [Repositório do AIDev com notebooks de exemplo](https://github.com/SAILResearch/AI_Teammates_in_SE3)
- [Ruff](https://github.com/astral-sh/ruff)
- [Radon](https://github.com/rubik/radon)
- [lifelines (análise de sobrevivência)](https://github.com/CamDavidsonPilon/lifelines)
- [Pacote `did` do R (Callaway e Sant'Anna)](https://github.com/bcallaway11/did)
