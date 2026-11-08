# **3 Metodologia**

Esta seção descreve o desenho completo do estudo, que prevê a análise dos 99 repositórios candidatos selecionados do AIDev. Os resultados da Seção 4 vêm de um **piloto com 5 desses repositórios**, que serviu para validar o *pipeline* de ponta a ponta; a amostra completa é trabalho em andamento. As análises do piloto são descritivas: não há grupo controle, testes de significância nem intervalos por *bootstrap*.

## **3.1 Materiais e Ferramentas**

A fonte de dados é a versão v3 do AIDev \[4\], restrita ao subconjunto curado de repositórios com mais de 100 estrelas. Usamos três tabelas: `repository` (2.807 repositórios, com nome, linguagem e estrelas), `pull_request` (33.596 PRs de agentes, com número, agente, datas de criação e de *merge* e repositório) e `pr_commits` (88.576 *commits*, com o SHA e o PR de origem). Nesse subconjunto, os PRs foram criados entre 24/12/2024 e 30/07/2025.

O histórico vem de clones do GitHub. Medimos avisos de análise estática com o Ruff \[24\], complexidade ciclomática \[26\] e linhas de código-fonte (SLOC) com o Radon \[25\] e comentários de dívida técnica autoadmitida (SATD) \[20\] com o módulo `tokenize` do Python. A sobrevivência dos avisos usa o estimador de Kaplan–Meier \[28\] da biblioteca lifelines \[27\]. A Tabela 3 lista as versões do ambiente em que todos os *notebooks* foram executados.

**Tabela 3 – Materiais e ferramentas**

| **Item**                       | **Versão ou especificação**                            | **Uso no estudo**                                       |
|--------------------------------|--------------------------------------------------------|---------------------------------------------------------|
| AIDev \[4\]                    | v3; tabelas `repository`, `pull_request`, `pr_commits` | Seleção, datas de adoção e atribuição por PR            |
| Git                            | 2.50.1 (Apple Git-155)                                 | Clones, histórico *first-parent*, *blobs* e renomeações |
| Python                         | 3.12.16                                                | Todos os *notebooks*                                    |
| Ruff \[24\]                    | 0.16.8                                                 | Avisos de análise estática                              |
| Radon \[25\]                   | 6.0.1                                                  | Complexidade ciclomática \[26\] e SLOC                  |
| lifelines \[27\]               | 0.30.3                                                 | Kaplan–Meier \[28\] e teste *log-rank*                  |
| pandas, NumPy, pyarrow         | 2.3.3; 2.5.4; 26.0.0                                   | Tabelas e arquivos Parquet                              |
| matplotlib, SciPy, statsmodels | 3.11.2; 1.18.1; 0.15.0                                 | Figuras; estatística prevista para a amostra completa   |
| Equipamento                    | MacBook Air, Apple M5, 16 GB de RAM, macOS 26.5.1      | Execução de todas as etapas                             |

Fonte: elaborado pelo autor.

O Ruff roda em modo `--isolated`, que ignora a configuração de cada projeto e aplica as mesmas regras a todos: F (Pyflakes), E (pycodestyle), B (bugbear), SIM (simplificações), C90 (complexidade de McCabe), PLE, PLW, PLR e PLC (regras de erro, aviso, refatoração e convenção derivadas do Pylint) e ERA (código comentado). Excluímos E501 (comprimento de linha), PLR2004 (números mágicos) e PLC0415 (*import* fora do topo). Os *code smells* do objetivo 2 são aproximados por essas regras, em especial PLR, SIM e C90. A duplicação de código, também citada no objetivo 2, **não foi medida** nesta versão do estudo.

## **3.2 Procedimentos**

O estudo é executado por seis *notebooks*, um por etapa (Figura 1), que gravam suas saídas em CSV ou Parquet. Um sétimo *notebook* (`06_artigo`) reúne os números e as figuras deste artigo sem alterar os anteriores. Código e saídas formam o pacote de replicação.

![](estudo/out/figs/fig1_pipeline.png)

Figura 1 – Etapas do estudo e *notebooks* correspondentes

Fonte: elaborado pelo autor.

**Seleção.** Filtramos os repositórios do subconjunto curado cuja linguagem principal é Python e contamos, em cada um, os PRs de agentes aceitos (`merged_at` preenchido). Ficaram como candidatos os repositórios com pelo menos 5 PRs aceitos (Tabela 4). O agente mais frequente nos candidatos é o OpenAI Codex em 59, o Devin em 17, o Copilot em 13, o Cursor em 9 e o Claude Code em 1.

**Tabela 4 – Seleção dos repositórios**

| **Etapa**                                             | **Repositórios** |
|-------------------------------------------------------|-----------------:|
| Subconjunto curado do AIDev v3 (mais de 100 estrelas) | 2.807            |
| Com linguagem principal Python                        | 530              |
| Em Python, com ao menos 1 PR de agente aceito         | 315              |
| Candidatos: ao menos 5 PRs de agente aceitos          | 99               |
| Piloto                                                | 5                |

Fonte: elaborado pelo autor.

**Escolha do piloto.** Escolhemos 5 dos 99 candidatos por conveniência, com três requisitos:

- projetos pequenos (de 29 a 261 arquivos Python no maior mês), para que cada medição leve minutos;
- pelo menos 12 meses de histórico antes da adoção;
- cobertura de quatro agentes diferentes (Claude Code, OpenAI Codex, Copilot e Devin).

A Tabela 5 resume a amostra. A escolha não é aleatória, e o piloto não representa os 99 candidatos.

**Tabela 5 – Amostra do piloto**

| **Projeto**           | **Agente principal** | **PRs aceitos (AIDev)** | **Estrelas** | ***Commits*** | ***Commits* de agente** | **Adoção** | **Meses a partir da adoção** | **Intensidade** |
|-----------------------|----------------------|------------------------:|-------------:|--------------:|------------------------:|------------|-----------------------------:|----------------:|
| numbagg/numbagg       | Claude Code          | 12                      | 226          | 791           | 40                      | 17/06/2025 | 16                           | 0,284           |
| econ-ark/HARK         | OpenAI Codex         | 8                       | 352          | 1.063         | 15                      | 20/05/2025 | 17                           | 0,088           |
| NewFuture/DDNS        | Copilot              | 13                      | 4.528        | 596           | 81                      | 01/07/2025 | 15                           | 0,535           |
| lidofinance/diffyscan | OpenAI Codex         | 6                       | 139          | 193           | 6                       | 22/05/2025 | 17                           | 0,111           |
| airbytehq/PyAirbyte   | Devin                | 14                      | 281          | 588           | 243                     | 28/01/2025 | 21                           | 0,820           |

Fonte: elaborado pelo autor. Notas: *commits* do histórico *first-parent* do *branch* padrão; meses de observação do mês da adoção até setembro de 2026, inclusive; intensidade definida na Seção 3.3.

**Reconstrução do histórico.** Clonamos o *branch* padrão de cada projeto e extraímos o histórico *first-parent*: a sequência de *commits* da linha principal, com um *commit* por PR integrado. Guardamos SHA, pais, autor, e-mail, data em UTC e mensagem completa, com os *trailers* de coautoria. A data da coleta é a do último *commit* entre os clones: 27/10/2026.

**Atribuição de *commits* a agentes.** Um *commit* do histórico é atribuído a um agente por três critérios, aplicados nesta ordem:

1. **Número do PR.** O título traz `Merge pull request #N` ou termina em `(#N)`, N é um PR de agente aceito no AIDev naquele repositório, e a data do *commit* cai entre 1 dia antes da criação do PR e 2 dias depois do *merge*. A janela evita confundir números de PR repetidos.
2. **SHA.** O SHA do *commit* consta em `pr_commits` como parte de um PR de agente, o que cobre PRs integrados por *rebase*.
3. **Assinatura.** Autor, e-mail ou mensagem trazem a marca de um agente (por exemplo, `copilot-swe-agent`, `Claude <noreply@anthropic.com>`, `devin-ai-integration`, `chatgpt-codex-connector` ou `/codex/` no nome do *branch*), só a partir de 01/03/2024. O critério cobre PRs posteriores ao corte do AIDev. Marcas ambíguas, como a coautoria do Copilot em sugestões de revisão aceitas por humanos, não são usadas.

A **data de adoção** é a do primeiro *commit* atribuído a agente.

**Critérios de inclusão.** O repositório precisa ter pelo menos 12 meses de histórico antes da adoção, pelo menos 30 *commits* nesses 12 meses, ao menos um PR do AIDev localizado no histórico e adoção até 31/08/2025. Os cinco projetos cumprem todos, com 36 (diffyscan) a 246 (PyAirbyte) *commits* nos 12 meses anteriores à adoção.

**Medição mensal.** Para cada mês, de 12 meses antes do mês da adoção até setembro de 2026, tomamos o último *commit* do histórico *first-parent* anterior ao primeiro dia do mês seguinte. Listamos os arquivos `.py` da árvore com o respectivo *blob*, o objeto do Git que guarda o conteúdo de uma versão do arquivo, excluindo pastas de ambientes e de terceiros (`venv/`, `site-packages/`, `vendor/`, `build/`, `dist/` e semelhantes). Projetos com mais de 5.000 arquivos `.py` num mês seriam excluídos por custo, o que não ocorreu.

Como um arquivo que não muda mantém o mesmo *blob*, cada *blob* é medido uma única vez: avisos do Ruff (regra e texto da linha), complexidade ciclomática por função, número de funções, SLOC e comentários com TODO, FIXME, HACK ou XXX (SATD). Os arquivos de teste entram na medição. A janela cobriu 28 meses no numbagg, 29 no HARK e no diffyscan, 27 no DDNS e 33 no PyAirbyte.

**Rastreamento dos avisos introduzidos.** Para cada *commit* de agente posterior à adoção, listamos os arquivos `.py` alterados com os *blobs* de antes e de depois (`git log --first-parent --raw -M`). Cada aviso é representado pelo par (regra, texto da linha), e os avisos novos são a diferença entre as contagens desses pares depois e antes; um aviso que só muda de linha não conta como novo.

A linha de base humana é uma amostra de *commits* humanos posteriores à adoção que alteram arquivos `.py`, sorteada com semente fixa (42). O tamanho é o número de *commits* de agente (mínimo de 10), limitado aos *commits* humanos disponíveis, o que reduziu a amostra no DDNS (32 contra 38 de agente) e no PyAirbyte (39 contra 178).

Cada arquivo é seguido nos *commits* seguintes, com renomeações. Um aviso é removido quando a contagem do seu par cai ou o arquivo é apagado, e registramos a data e se o *commit* responsável é de agente. Avisos ainda presentes ficam censurados na data da coleta.

**Análises por QP.**

- **QP1.** Comparamos cada métrica mensal com o seu valor no mês −1, nas faixas de −6 a −2, 0 a 5, 6 a 11 e 12 a 15 meses relativos à adoção.
- **QP2.** Classificamos os arquivos em existentes no mês −1 e tocados por agente depois da adoção (A), existentes tocados só por humanos (H), existentes não tocados (U) e novos, criados por agente ou por humano. Os arquivos alterados entre os meses −12 e −1 formam a linha de base.
- **QP3.** Colocamos lado a lado a intensidade de uso e a variação das métricas e calculamos taxas por agente.
- **QP4.** Estimamos curvas de Kaplan–Meier por tipo de autor.

Para a amostra completa, o desenho prevê teste de Wilcoxon nas faixas da QP1, regressão segmentada com erros agrupados por repositório, o estimador de Callaway e Sant'Anna \[29\] com controles ainda não tratados, intervalos por *bootstrap* de repositórios e correlação de Spearman. Nada disso foi aplicado ao piloto, porque cinco repositórios não sustentam inferência.

## **3.3 Métricas de Avaliação**

A Tabela 6 define as métricas. Nas fórmulas, $p$ é o projeto, $m$ o mês, $f$ o arquivo e $i$ uma alteração (par *commit*, arquivo). Todas as densidades usam SLOC, e uma KLOC equivale a mil linhas de código-fonte.

**Tabela 6 – Métricas, fórmulas e questões de pesquisa**

| **Métrica**                                  | **Definição**                                                                                                                                                                      | **QP**   |
|----------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|
| Avisos/KLOC                                  | $1000 \cdot \sum_f \text{avisos}_{f,m} / \sum_f \text{SLOC}_{f,m}$                                                                                                                 | QP1, QP3 |
| Avisos de complexidade/KLOC                  | Idem, só com as regras C901 e PLR09xx                                                                                                                                              | QP1      |
| Complexidade média por função                | $\sum_f \text{CC}_{f,m} / \sum_f \text{funções}_{f,m}$, com CC a complexidade ciclomática \[26\]                                                                                   | QP1, QP3 |
| SATD/KLOC                                    | $1000 \cdot \sum_f \text{SATD}_{f,m} / \sum_f \text{SLOC}_{f,m}$                                                                                                                   | QP1, QP3 |
| Variação na faixa $[a,b]$                    | $100 \cdot (\overline{x}_{p,\,a..b} / x_{p,-1} - 1)$, média mensal da faixa contra o mês −1                                                                                        | QP1, QP3 |
| Variação de avisos por KLOC adicionada       | $1000 \cdot \sum_f \Delta\text{avisos}_f / \sum_f \Delta\text{SLOC}_f$, só arquivos com $\Delta\text{SLOC}_f > 0$; $\Delta$ entre o mês −1 (ou a criação) e o último mês do arquivo | QP2      |
| Variação de complexidade por KLOC adicionada | Idem, com $\Delta\text{CC}_f$                                                                                                                                                      | QP2      |
| Avisos novos por KLOC adicionada             | $1000 \cdot \sum_i \text{novos}_i / \sum_i \max(0, \text{SLOC}^{\text{depois}}_i - \text{SLOC}^{\text{antes}}_i)$                                                                  | QP3      |
| Intensidade de uso                           | *Commits* de agente que alteram `.py` após a adoção / *commits* que alteram `.py` após a adoção                                                                                    | QP3      |
| $S(t)$ de Kaplan–Meier \[28\]                | $\hat S(t) = \prod_{t_j \le t} (1 - d_j / n_j)$, com $d_j$ avisos removidos no dia $t_j$ e $n_j$ avisos ainda em risco                                                             | QP4      |
| Fração removida e fração ainda presente      | Avisos removidos até a coleta / avisos introduzidos, e o complemento                                                                                                               | QP4      |
| Quem removeu                                 | Avisos removidos por *commit* de agente / avisos removidos                                                                                                                         | QP4      |

Fonte: elaborado pelo autor.

Duas métricas têm nomes parecidos. A da QP2 é uma **variação líquida** do arquivo entre dois estados e pode ser negativa; no *notebook* 05 ela aparece como "avisos novos por KLOC". A da QP3 conta só os avisos **novos** de cada alteração, sem descontar os removidos.

**Critérios de sucesso do piloto.** Como não houve validação manual da atribuição, o piloto é considerado bem-sucedido se o *pipeline* roda do início ao fim e passa nas verificações automáticas da Tabela 7. Todas foram cumpridas, com uma ressalva: 5 PRs do AIDev do PyAirbyte não foram localizados no histórico *first-parent*. Nenhum dos 2 a 13 *commits* de cada um desses PRs aparece no histórico, o que sugere integração em outro *branch*; a causa não foi verificada.

**Tabela 7 – Verificações automáticas do piloto**

| **Verificação**                                                                   | **Resultado**                                                                  |
|-----------------------------------------------------------------------------------|--------------------------------------------------------------------------------|
| O funil reproduz as contagens do subconjunto curado                               | 2.807, 530, 315 e 99; `candidatos.csv` com 99 linhas                           |
| *Commits* no log = `git rev-list --first-parent --count HEAD` = linhas atribuídas | Iguais nos 5 projetos (791, 1.063, 596, 193 e 588)                             |
| Assinatura e AIDev não discordam no mesmo *commit*                                | 0 discordâncias nos 5 projetos                                                 |
| Nenhuma atribuição por assinatura anterior a 01/03/2024                           | A mais antiga é de 16/05/2025 (PyAirbyte)                                      |
| A adoção coincide com o primeiro PR aceito no AIDev                               | Mesmo dia nos 5 projetos                                                       |
| Fração de *commits* de agente plausível                                           | De 1,4% (HARK) a 41,3% (PyAirbyte)                                             |
| PRs do AIDev localizados no histórico                                             | 48 de 53 (5 do PyAirbyte não localizados)                                      |
| Arquivos inalterados (U) não mudam de avisos                                      | Soma da variação = 0 nos 135 arquivos U                                        |
| Os avisos introduzidos fecham com os totais por autor e por destino               | 324 + 235 = 559; 431 presentes + 120 corrigidos + 8 em arquivos apagados = 559 |
| A QP2 recalculada de forma independente no *notebook* 06 é igual à do 05          | Igual                                                                          |
| Arquivos com erro de sintaxe                                                      | 4 registros (arquivo, mês), 2 arquivos de documentação do HARK                 |

Fonte: elaborado pelo autor.

# **4 Resultados e Discussão**

Os resultados a seguir descrevem os cinco projetos do piloto, lado a lado sempre que há tabela por projeto. Um aviso vale para todas as subseções: o PyAirbyte responde por 871 das 1.609 alterações rastreadas (54%) e por 38,469 das 64,122 KLOC adicionadas nessas alterações. Por isso, os totais agregados refletem sobretudo esse projeto.

## **4.1 QP1 – Trajetória após a adoção**

A Tabela 8 compara cada métrica com o mês anterior à adoção, e a Figura 2 mostra as trajetórias mensais divididas pelo valor do mês −1. Entre o mês −1 e a faixa de 12 a 15 meses, a densidade de avisos caiu nos cinco projetos, de −7% (numbagg) a −67% (diffyscan). A complexidade média por função variou pouco em quatro projetos (de −3% a +8%) e caiu −43% no DDNS. O SATD por KLOC caiu nos quatro projetos que tinham SATD no mês −1; o DDNS não tem comentário SATD nesse mês, e a razão não existe (traço na tabela). Já os avisos de complexidade por KLOC subiram em quatro projetos, mas sobre poucos avisos no mês −1: 7 no numbagg e 2 no DDNS.

**Tabela 8 – Variação de cada métrica em relação ao mês −1, por faixa de meses relativos à adoção**

| **Métrica**                   | **Projeto** | **Mês −1** | **−6 a −2** | **0 a 5** | **6 a 11** | **12 a 15** |
|-------------------------------|-------------|-----------:|------------:|----------:|-----------:|------------:|
| Avisos/KLOC                   | numbagg     | 14,80      | −0%         | −4%       | −4%        | −7%         |
|                               | HARK        | 27,98      | −1%         | +3%       | −3%        | −11%        |
|                               | DDNS        | 14,48      | +34%        | −13%      | −13%       | −18%        |
|                               | diffyscan   | 42,42      | +2%         | −35%      | −54%       | −67%        |
|                               | PyAirbyte   | 7,45       | −4%         | −1%       | −5%        | −10%        |
| Avisos de complexidade/KLOC   | numbagg     | 1,95       | +1%         | +52%      | +54%       | +51%        |
|                               | HARK        | 10,99      | −1%         | +2%       | −1%        | +6%         |
|                               | DDNS        | 1,32       | +10%        | +395%     | +389%      | +320%       |
|                               | diffyscan   | 9,36       | −6%         | −7%       | −21%       | −31%        |
|                               | PyAirbyte   | 2,06       | −28%        | +1%       | +33%       | +64%        |
| Complexidade média por função | numbagg     | 3,27       | +1%         | +6%       | +5%        | +4%         |
|                               | HARK        | 2,42       | −0%         | +6%       | +2%        | −3%         |
|                               | DDNS        | 4,07       | −3%         | −48%      | −48%       | −43%        |
|                               | diffyscan   | 3,21       | +1%         | +1%       | −0%        | −0%         |
|                               | PyAirbyte   | 2,74       | −1%         | +0%       | +4%        | +8%         |
| SATD/KLOC                     | numbagg     | 2,51       | +6%         | −21%      | −33%       | −35%        |
|                               | HARK        | 1,08       | −5%         | −7%       | −36%       | −49%        |
|                               | DDNS        | 0,00       | –           | –         | –          | –           |
|                               | diffyscan   | 0,62       | −80%        | −21%      | −100%      | −100%       |
|                               | PyAirbyte   | 1,46       | +30%        | −5%       | −23%       | −38%        |

Fonte: elaborado pelo autor. Nota: o DDNS só tem meses até o 14; na sua faixa de 12 a 15 entram os meses 12 a 14.

![](estudo/out/figs/qp1_trajetorias.png)

Figura 2 – Trajetória mensal de avisos por KLOC, complexidade média por função e SATD por KLOC, em razão do valor do mês −1

Fonte: elaborado pelo autor.

Antes de interpretar os saltos das séries, examinamos os *commits* dos meses em que eles ocorrem. Três têm causa identificada: dois vêm de PRs humanos de correção de avisos, e um de mudança na composição do código.

**Queda da complexidade no DDNS (mês 0, julho de 2025).** O projeto passou de 17 para 64 arquivos `.py`, e os 33 arquivos novos de teste, com 775 funções de complexidade média 1,47, derrubaram a média. Fora dos testes, a complexidade média por função subiu de 4,07 para 4,39, a densidade de avisos de 14,48 para 40,02 por KLOC e os avisos de complexidade de 2 para 74. No mês, *commits* humanos criaram ou renomearam 36 arquivos `.py`, 34 deles em dois PRs de reestruturação, e *commits* do Copilot, 18.

**Queda de avisos no diffyscan (mês 0, maio de 2025).** Em 08/05/2025, duas semanas antes do primeiro *commit* do Codex, um PR humano intitulado `fix/linter-errors` reduziu os avisos do projeto de 68 para 44.

**Queda de avisos no PyAirbyte (mês −9, abril de 2024).** Um PR humano intitulado "Update ruff rules and fix exceptions" reduziu os avisos de 132 para 87, nove meses antes da adoção. Ele contribui para a queda da razão de cerca de 2, no início da série na Figura 2, para perto de 1.

As quedas de SATD/KLOC precisam ser lidas como contagens (Tabela 9). No numbagg (9 comentários no início e no fim) e no PyAirbyte (de 22 para 21), a queda vem do crescimento do SLOC. Só o HARK reduziu a contagem, de 32 para 22. No diffyscan, o −100% é um único comentário, removido por um PR humano em outubro de 2025.

Como a fração do SLOC em testes subiu em todos os projetos (de 0% para 69% no DDNS e para 47% no diffyscan), repetimos a comparação sem os testes. Sem eles, a variação de avisos/KLOC na faixa de 12 a 15 meses muda de sinal em três projetos: +1% no numbagg, +92% no DDNS e +11% no PyAirbyte. Restam quedas no HARK (−2%) e no diffyscan (−50%). A complexidade média por função sem testes sobe nos cinco projetos, de +5% a +24%.

**Tabela 9 – Composição do código e variação na faixa de 12 a 15 meses com e sem arquivos de teste**

| **Projeto** | **SLOC (mês −1 → último)** | **% do SLOC em testes** | **SATD (comentários)** | **Avisos/KLOC: com testes** | **Avisos/KLOC: sem testes** | **CC/função: com testes** | **CC/função: sem testes** |
|-------------|---------------------------:|------------------------:|-----------------------:|----------------------------:|----------------------------:|--------------------------:|--------------------------:|
| numbagg     | 3.582 → 5.924              | 56 → 61                 | 9 → 9                  | −7%                         | +1%                         | +4%                       | +5%                       |
| HARK        | 29.560 → 42.895            | 16 → 26                 | 32 → 22                | −11%                        | −2%                         | −3%                       | +5%                       |
| DDNS        | 1.519 → 22.982             | 0 → 69                  | 0 → 0                  | −18%                        | +92%                        | −43%                      | +18%                      |
| diffyscan   | 1.603 → 7.531              | 0 → 47                  | 1 → 0                  | −67%                        | −50%                        | 0%                        | +24%                      |
| PyAirbyte   | 15.027 → 51.196            | 26 → 39                 | 22 → 21                | −10%                        | +11%                        | +8%                       | +10%                      |

Fonte: elaborado pelo autor. Nota: arquivos de teste são os caminhos em `tests/`, `test/` ou `testing/` e os arquivos `test_*.py`, `*_test.py` e `conftest.py`.

**Resposta à QP1.** No piloto, após a adoção, observou-se queda da densidade de avisos nos cinco projetos e estabilidade da complexidade média por função em quatro deles. Essas variações, porém, dependem da entrada de testes e de PRs humanos de correção: sem os testes, a densidade de avisos subiu em três projetos. O piloto não indica um padrão comum de manutenção, acentuação ou reversão e não permite confirmar nem contestar o aumento estimado por Agarwal et al. \[11\].

## **4.2 QP2 – Proveniência da degradação**

A Figura 3 e a Tabela 10 apresentam a variação de avisos e de complexidade por KLOC adicionada em cada grupo de arquivos. No agregado, os arquivos existentes tocados por agentes tiveram variação de 8,1 avisos por KLOC adicionada, contra 0,6 nos existentes mantidos só por humanos. Entre os arquivos novos, os criados por agentes tiveram 5,0 e os criados por humanos, 13,1, e a linha de base pré-adoção registrou 36,7. A complexidade nova por KLOC ficou entre 137,4 e 199,7 nos cinco grupos, sem ordem clara.

![](estudo/out/figs/qp2_taxas.png)

Figura 3 – Variação de avisos e de complexidade por KLOC adicionada, por grupo de arquivos (agregado dos cinco projetos)

Fonte: elaborado pelo autor.

A linha de base pré-adoção (1,52 KLOC, das quais 1,16 do HARK) e o grupo H (1,75 KLOC) têm denominadores pequenos, e suas taxas não sustentam conclusão.

Por projeto, os grupos A e H existem juntos em quatro projetos. O grupo A tem taxa maior em três deles: numbagg (13,2 contra 0,0), diffyscan (15,9 contra −8,7) e PyAirbyte (8,9 contra 0,0). No HARK, tem taxa menor (−8,8 contra 5,6). Nos quatro, o grupo H soma no máximo 1,08 KLOC adicionada, e no DDNS ele não tem arquivo que tenha crescido.

Entre os arquivos novos, a taxa dos criados por agentes é maior no numbagg, no DDNS e no diffyscan e menor no PyAirbyte, que concentra 79% da KLOC desse grupo; o agregado de 5,0 reflete sobretudo esse projeto. Testes respondem por 54% da KLOC dos arquivos novos de agentes e por 62% da dos arquivos novos de humanos.

**Tabela 10 – Variação de avisos por KLOC adicionada por grupo e por projeto (KLOC adicionada entre parênteses)**

| **Grupo**                       | **Arquivos** | **Agregado** | **numbagg** | **HARK**     | **DDNS**     | **diffyscan** | **PyAirbyte** |
|---------------------------------|-------------:|-------------:|------------:|-------------:|-------------:|--------------:|--------------:|
| Pré-adoção: alterados           | 79           | 36,7 (1,52)  | 0,0 (0,03)  | 50,2 (1,16)  | – (0,00)     | – (0,00)      | −5,9 (0,34)   |
| Pós: existentes, só humanos (H) | 71           | 0,6 (1,75)   | 0,0 (0,04)  | 5,6 (1,08)   | – (0,00)     | −8,7 (0,58)   | 0,0 (0,06)    |
| Pós: existentes, com agente (A) | 128          | 8,1 (8,36)   | 13,2 (0,91) | −8,8 (1,24)  | 21,5 (0,23)  | 15,9 (1,26)   | 8,9 (4,72)    |
| Pós: novos, criados por humanos | 168          | 13,1 (42,96) | 0,0 (0,23)  | 20,1 (18,34) | 9,9 (15,21)  | 5,1 (3,93)    | 4,6 (5,24)    |
| Pós: novos, criados por agente  | 156          | 5,0 (39,38)  | 10,3 (1,17) | – (0,00)     | 10,4 (6,71)  | 5,9 (0,34)    | 3,6 (31,15)   |

Fonte: elaborado pelo autor. Nota: "–" indica grupo sem arquivos que cresceram no projeto (taxa indefinida).

Nos 135 arquivos que nenhum *commit* pós-adoção alterou (grupo U), a soma da variação de avisos foi zero, o que confirma a consistência entre atribuição e medição.

**Resposta à QP2.** No piloto, os arquivos existentes tocados por agentes acumularam mais variação de avisos por KLOC adicionada (8,1) do que os mantidos só por humanos (0,6), e essa ordem aparece em três dos quatro projetos em que os dois grupos existem. O grupo H, porém, soma apenas 1,75 KLOC no agregado e no máximo 1,08 KLOC por projeto. Entre os arquivos novos, a taxa dos criados por agentes foi menor (5,0 contra 13,1), um valor determinado pelo PyAirbyte. Os dados do piloto não permitem dizer se a degradação se concentra nos arquivos de agentes.

## **4.3 QP3 – Diferenças entre agentes e intensidade de uso**

A Tabela 11 coloca lado a lado a intensidade de uso, de 0,088 (HARK) a 0,820 (PyAirbyte), e a variação das métricas entre o mês −1 e a média dos meses 9 a 12. Os pares não mostram ordenação: o diffyscan tem a maior queda de avisos/KLOC (−63%) com intensidade de 0,111 (4 *commits* de agente), e o PyAirbyte, a maior intensidade, tem −6%. Com cinco pontos, não calculamos correlação. A queda de −48% na complexidade do DDNS vem da entrada dos testes (Seção 4.1).

**Tabela 11 – Intensidade de uso e variação das métricas entre o mês −1 e a média dos meses 9 a 12**

| **Projeto** | **Agente principal** | ***Commits* com `.py`** | ***Commits* de agente** | **Intensidade** | **Avisos/KLOC** | **CC/função** | **SATD/KLOC** |
|-------------|----------------------|------------------------:|------------------------:|----------------:|----------------:|--------------:|--------------:|
| numbagg     | Claude Code          | 88                      | 25                      | 0,284           | −4%             | +5%           | −32%          |
| HARK        | OpenAI Codex         | 159                     | 14                      | 0,088           | −5%             | −0%           | −45%          |
| DDNS        | Copilot              | 71                      | 38                      | 0,535           | −13%            | −48%          | –             |
| diffyscan   | OpenAI Codex         | 36                      | 4                       | 0,111           | −63%            | −3%           | −100%         |
| PyAirbyte   | Devin                | 217                     | 178                     | 0,820           | −6%             | +5%           | −31%          |

Fonte: elaborado pelo autor.

A Figura 4 mostra os avisos novos por KLOC adicionada por autor. Entre os que têm mais de 1 KLOC adicionada, as taxas foram 4,6 para o Devin (34,892 KLOC), 12,1 para o Claude Code (2,904), 13,2 para os humanos (17,770) e 14,5 para o Copilot (8,433). As taxas do OpenAI Codex (46,3, com 5 avisos em 0,108 KLOC) e do Cursor (0,0 em 0,015 KLOC) vêm de 19 e 6 alterações e não sustentam conclusão.

![](estudo/out/figs/qp3_por_agente.png)

Figura 4 – Avisos novos por KLOC adicionada, por agente e na amostra humana

Fonte: elaborado pelo autor.

A comparação entre agentes se confunde com a comparação entre projetos: o Devin só aparece no PyAirbyte, 8,397 das 8,433 KLOC do Copilot estão no DDNS, e o HARK reúne quatro agentes (OpenAI Codex, Cursor, Copilot e Claude Code). Dentro de cada projeto (Tabela 12), os agentes têm taxa maior que os humanos no DDNS (14,5 contra 13,2), no PyAirbyte (4,6 contra 3,1) e no diffyscan (48,2 contra 15,3, sobre 0,083 KLOC). Têm taxa menor no HARK (3,1 contra 38,2) e no numbagg (14,6 contra 28,5, sobre 0,456 KLOC humanas). A taxa baixa do Devin no agregado acompanha o projeto: no PyAirbyte, a amostra humana também tem taxa baixa.

**Tabela 12 – Avisos novos por KLOC adicionada, por projeto e tipo de autor**

| **Projeto** | **Autor** | **Alterações** | ***Commits*** | **Avisos novos** | **KLOC adicionada** | **Avisos novos/KLOC** |
|-------------|-----------|---------------:|--------------:|-----------------:|--------------------:|----------------------:|
| numbagg     | agente    | 81             | 25            | 34               | 2,329               | 14,6                  |
|             | humano    | 44             | 25            | 13               | 0,456               | 28,5                  |
| HARK        | agente    | 25             | 14            | 2                | 0,651               | 3,1                   |
|             | humano    | 48             | 14            | 41               | 1,072               | 38,2                  |
| DDNS        | agente    | 275            | 38            | 122              | 8,397               | 14,5                  |
|             | humano    | 220            | 32            | 149              | 11,296              | 13,2                  |
| diffyscan   | agente    | 9              | 4             | 4                | 0,083               | 48,2                  |
|             | humano    | 36             | 10            | 21               | 1,369               | 15,3                  |
| PyAirbyte   | agente    | 776            | 178           | 162              | 34,892              | 4,6                   |
|             | humano    | 95             | 39            | 11               | 3,577               | 3,1                   |

Fonte: elaborado pelo autor.

**Resposta à QP3.** No piloto, não há ordenação visível entre intensidade de uso e variação das métricas. As taxas por agente com denominador razoável (de 4,6 a 14,5 avisos novos por KLOC) acompanham o projeto em que cada agente atuou, e não é possível separar o efeito do agente do efeito do projeto.

## **4.4 QP4 – Permanência e remoção dos avisos introduzidos**

Foram rastreados 324 avisos introduzidos por agentes e 235 pela amostra humana (Tabela 13 e Figura 5). Até a coleta, 74% dos de agentes e 81% dos humanos continuavam no código. As curvas caem nos primeiros dias e depois se estabilizam: $S(90)$ foi 0,78 e 0,81, e $S(365)$, 0,74 e 0,77. Entre os removidos, a mediana foi de 7 e 12 dias, e 90% e 100% foram correções do aviso (o resto saiu com o arquivo apagado). O *log-rank* foi executado só para confirmar que o procedimento roda; seu valor-p (0,1544) não deve ser interpretado com cinco projetos.

**Tabela 13 – Permanência dos avisos introduzidos**

| **Introduzidos por** | **Avisos** | **Removidos** | **Ainda presentes** | **S(90)** | **S(365)** | **Mediana de dias (removidos)** | **Corrigidos** | **Removidos por agente** |
|----------------------|-----------:|--------------:|--------------------:|----------:|-----------:|--------------------------------:|---------------:|-------------------------:|
| Agentes              | 324        | 83            | 74%                 | 0,78      | 0,74       | 7                               | 90%            | 64%                      |
| Humanos (amostra)    | 235        | 45            | 81%                 | 0,81      | 0,77       | 12                              | 100%           | 31%                      |

Fonte: elaborado pelo autor.

![](estudo/out/figs/qp4_sobrevivencia.png)

Figura 5 – Curvas de Kaplan–Meier da fração de avisos ainda presentes, por tipo de autor da introdução

Fonte: elaborado pelo autor.

Dos avisos de agentes removidos, 64% saíram em *commits* de agentes, contra 31% dos humanos. Esse resultado se concentra em dois projetos (Tabela 14). Dos 53 avisos de agentes removidos por agentes, 24 estão no PyAirbyte, onde todas as remoções desses avisos foram feitas pelo Devin, e 19 no DDNS; são os projetos de maior intensidade. No HARK e no diffyscan, nenhum aviso foi removido por agente. A autoria da remoção parece acompanhar a fração de *commits* de agente do projeto.

**Tabela 14 – Avisos introduzidos e removidos, por projeto e tipo de autor da introdução**

| **Projeto** | **Introduzidos por** | **Avisos** | **Removidos** | **% removidos** | **Removidos por agente** |
|-------------|----------------------|-----------:|--------------:|----------------:|-------------------------:|
| numbagg     | agentes              | 34         | 18            | 53%             | 10                       |
|             | humanos              | 13         | 0             | 0%              | 0                        |
| HARK        | agentes              | 2          | 0             | 0%              | 0                        |
|             | humanos              | 41         | 11            | 27%             | 0                        |
| DDNS        | agentes              | 122        | 39            | 32%             | 19                       |
|             | humanos              | 149        | 21            | 14%             | 14                       |
| diffyscan   | agentes              | 4          | 2             | 50%             | 0                        |
|             | humanos              | 21         | 11            | 52%             | 0                        |
| PyAirbyte   | agentes              | 162        | 24            | 15%             | 24                       |
|             | humanos              | 11         | 2             | 18%             | 0                        |

Fonte: elaborado pelo autor.

Por família, a PLR (refatoração, derivada do Pylint) é a mais frequente entre os avisos de agentes (178 de 324, dos quais 106 da regra PLR0913, excesso de argumentos) e entre os humanos (84 de 235). A família B (bugbear) tem mais ocorrências humanas (50) que de agentes (9). As contagens por família são pequenas demais para curvas separadas.

**Resposta à QP4.** No piloto, cerca de três quartos dos avisos introduzidos por agentes (74%) permanecem no código até a coleta, uma fração próxima à da amostra humana (81%). Quando removidos, os avisos de agentes saem principalmente em *commits* de agentes (64%), mas esse resultado vem do DDNS e do PyAirbyte, os projetos em que os agentes fazem mais *commits*.

## **4.5 Discussão**

**Atendimento dos objetivos.** A Tabela 15 resume o que o piloto alcançou. Todos os procedimentos rodaram e passaram nas verificações, mas nenhum objetivo foi atingido por completo, porque as respostas são descritivas e limitadas a cinco projetos.

**Tabela 15 – Atendimento dos objetivos específicos no piloto**

| **Objetivo**                            | **Situação no piloto** | **O que foi feito e o que falta**                                                                                                               |
|-----------------------------------------|------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------|
| 1. Selecionar e estender a amostra      | Parcial                | 99 candidatos selecionados; métricas mensais só para 5 projetos                                                                                 |
| 2. Estimar o efeito da adoção           | Parcial                | Trajetórias descritas por até 21 meses após a adoção; sem estimativa de efeito; duplicação não medida; *smells* aproximados pelas regras do Ruff |
| 3. Comparar arquivos A e H              | Parcial                | Procedimento validado (grupo U com variação zero); grupos com poucas KLOC e resultado não consistente entre projetos                            |
| 4. Variação entre agentes e intensidade | Parcial                | Taxas por agente e intensidade calculadas; agente confundido com projeto; correlação não calculada                                              |
| 5. Ciclo de vida dos problemas          | Parcial                | Fração removida, tempo e autoria da remoção medidos; resultado concentrado em dois projetos                                                     |

Fonte: elaborado pelo autor.

O principal avanço é metodológico. O piloto mostra que é possível combinar os três elementos que a Tabela 2 aponta como ausentes nos trabalhos mais próximos: atribuição por arquivo, observação por mais de 12 meses após a adoção e autoria da remoção. Ele também revelou dois cuidados para a amostra completa:

- **Composição do código.** A entrada de testes alterou o sinal das variações de três projetos, e os resultados devem ser apresentados com e sem testes.
- **Eventos humanos concorrentes.** PRs humanos de correção de *lint* produziram as maiores quedas de avisos, que uma análise sem grupo controle atribuiria ao período pós-adoção.

**Comparação com os trabalhos relacionados.** Agarwal et al. \[11\] estimaram, com diferenças-em-diferenças, 518 repositórios tratados e controles pareados, aumentos de cerca de 18% nos avisos e de cerca de 39% na complexidade cognitiva nos seis meses após a adoção. O piloto usa outra ferramenta e outras regras, mede complexidade ciclomática, normaliza por KLOC contra o mês −1, não tem controle e tem cinco projetos. Com essas diferenças, a queda de avisos/KLOC observada não contradiz \[11\], mas também não o confirma: sem testes, a densidade subiu em três projetos, e a variação na faixa de 0 a 5 meses foi pequena em três deles (de −4% a +3%). O mesmo vale para He et al. \[10\], que estudaram o Cursor em cerca de 800 repositórios; no piloto, o Cursor aparece em apenas 6 alterações.

Cynthia et al. \[8\] usaram o SonarQube no *merge* de 1.210 PRs de correção, encontraram *smells* como problema dominante e viram as diferenças entre agentes desaparecerem com a normalização pelo tamanho da mudança. No piloto, com outra ferramenta e outra unidade (a alteração por arquivo seguida no tempo), as regras de refatoração também predominam nos avisos de agentes (178 de 324), e as diferenças entre agentes acompanham as diferenças entre projetos. Os resultados apontam na mesma direção, sem serem comparáveis diretamente. Huang et al. \[9\] mediram redundância semântica, e a duplicação não foi medida aqui, o que impede a comparação.

Liu et al. \[12\] observaram que 22,7% dos problemas introduzidos por assistentes sobrevivem até a versão mais recente; no piloto, 74% dos avisos de agentes permaneciam na coleta. Os dois estudos diferem em ferramenta, unidade (*commits* de assistentes em geral contra alterações de agentes), horizonte (até a versão mais recente contra no máximo 21 meses), critério de remoção e amostra (6.299 repositórios contra 5), e os números não são comparáveis. A alta permanência está mais próxima do padrão de *smells* raramente removidos descrito por Tufano et al. \[7\], em outra linguagem e com outra ferramenta.

Popescu et al. \[21\] observaram mais *churn* no código de agentes medindo a estabilidade de linhas. O piloto mede avisos, e a fração removida foi maior entre os de agentes (83 de 324) que entre os humanos (45 de 235), diferença que não pode ser interpretada com cinco projetos. Nenhum desses trabalhos identifica quem remove a dívida. No piloto, a autoria da remoção acompanhou a intensidade de uso, o que indica que essa análise deve controlar a intensidade na amostra completa.

## **4.6 Ameaças à validade**

**Validade interna.**

- **Atribuição sem validação manual.** *Commits* de agentes sem marca reconhecida contam como humanos, e *merges* por *squash* atribuem ao agente os ajustes humanos feitos no PR. Cinco PRs do AIDev do PyAirbyte não foram localizados. As verificações automáticas (Tabela 7) reduzem, mas não eliminam, esse risco.
- **Sem grupo controle.** Eventos concorrentes podem aparecer como efeito do período pós-adoção, como os PRs humanos de *lint* no diffyscan e no PyAirbyte.
- **Remoção.** Editar a linha de um aviso conta como remoção, o que pode superestimar as correções.
- **Amostra humana.** Sorteada com semente 42, ficou limitada nos projetos com poucos *commits* humanos (39 no PyAirbyte).

**Validade externa.**

- **Amostra.** Os cinco projetos foram escolhidos por conveniência entre os 99 candidatos: são pequenos e escritos em Python, com mais de 100 estrelas.
- **Peso do PyAirbyte.** Um único projeto responde por 54% das alterações rastreadas e domina os totais.
- **Agentes.** Cada agente aparece praticamente em um único projeto.
- **Dados.** Os resultados se referem ao AIDev v3, com PRs até 30/07/2025, e a projetos Python, e não se generalizam para outras linguagens nem para o conjunto dos candidatos.

**Validade de construção.**

- **Avisos do Ruff.** São uma aproximação de qualidade: não medem defeitos, e as regras selecionadas são apenas um recorte dos *code smells*.
- **Configuração dos projetos.** Quatro dos cinco projetos mencionam o Ruff nos seus arquivos de configuração, de modo que o modo `--isolated` pode contar avisos que os mantenedores decidiram ignorar.
- **Complexidade e SATD.** A complexidade é ciclomática, não cognitiva como em \[11\]. O SATD depende de quatro palavras-chave e, no piloto, soma poucos comentários por projeto (de 0 a 32 no mês −1).
- **Duplicação.** Não foi medida.
- **Testes e adoção.** Os testes entram nas métricas e mudam o sinal de parte dos resultados. A adoção é definida pelo primeiro *commit* de agente, que coincidiu com o primeiro PR aceito no AIDev nos cinco projetos.

**Validade de conclusão.**

- **Análise descritiva.** Não há testes de significância nem intervalos de confiança, e o valor-p do *log-rank* não deve ser interpretado.
- **Denominadores pequenos.** Várias taxas se apoiam em menos de 1 KLOC adicionada e em poucas alterações (OpenAI Codex, com 19, e Cursor, com 6) ou em menos de 2 KLOC (grupos H e pré-adoção da QP2).
- **Valores de base pequenos.** As variações percentuais dependem de um único mês de base, o que amplia oscilações quando a base é pequena, como em 2 avisos de complexidade no DDNS ou 1 comentário SATD no diffyscan.
- **Agregados.** Misturam projetos de tamanhos muito diferentes; as respostas às QP valem só como descrição do piloto.

# **Referências (novas)**

\[24\] ASTRAL. Ruff: an extremely fast Python linter and code formatter. Versão 0.16.8. Disponível em: <https://github.com/astral-sh/ruff>. Acesso em: 28 out. 2026.

\[25\] RADON: various code metrics for Python code. Versão 6.0.1. Disponível em: <https://github.com/rubik/radon>. Acesso em: 28 out. 2026.

\[26\] McCABE, T. J. A complexity measure. *IEEE Transactions on Software Engineering*, v. SE-2, n. 4, p. 308–320, 1976.

\[27\] DAVIDSON-PILON, C. lifelines: survival analysis in Python. *Journal of Open Source Software*, v. 4, n. 40, p. 1317, 2019.

\[28\] KAPLAN, E. L.; MEIER, P. Nonparametric estimation from incomplete observations. *Journal of the American Statistical Association*, v. 53, n. 282, p. 457–481, 1958.

\[29\] CALLAWAY, B.; SANT'ANNA, P. H. C. Difference-in-Differences with multiple time periods. *Journal of Econometrics*, v. 225, n. 2, p. 200–230, 2021.
