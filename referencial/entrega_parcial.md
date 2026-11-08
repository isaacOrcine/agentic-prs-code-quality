# **1 Introdução**

A forma como o código é escrito mudou em poucos anos. Assistentes baseados em Modelos de Linguagem de Grande Escala (LLMs) passaram do autocompletar de linhas para **agentes de codificação** que planejam uma tarefa, editam vários arquivos, executam testes e abrem *pull requests* (PRs) sem intervenção humana a cada passo. Essa transição tem sido descrita como a chegada de uma “Engenharia de Software 3.0”, na qual agentes atuam como colegas de equipe \[1, 2\]. A adoção é ampla: na pesquisa Stack Overflow de 2025, 84% dos respondentes usam ou planejam usar ferramentas de IA, e cerca de metade dos desenvolvedores profissionais as usa diariamente \[3\]. No GitHub, o conjunto de dados AIDev registra 932.791 PRs de autoria de cinco agentes (OpenAI Codex, Devin, GitHub Copilot, Cursor e Claude Code) em 116.211 repositórios até agosto de 2025 \[4\]. Esse código não fica na fila de revisão: em um estudo com 567 PRs do Claude Code, 83,8% foram aceitos pelos mantenedores \[5\].

A questão deixa de ser se agentes produzem código que funciona e passa a ser o que acontece com o projeto depois que esse código entra. A maior parte do custo de um sistema está na sua manutenção, e problemas de projeto tendem a se acumular. *Code smells* aumentam a propensão de mudanças e defeitos nas classes afetadas \[6\] e, uma vez introduzidos, raramente são removidos \[7\]. Se agentes introduzem dívida técnica em ritmo maior do que equipes conseguem pagar, o ganho de velocidade de hoje vira custo de manutenção amanhã. Os próprios desenvolvedores já percebem esse risco: em 2025, 46% disseram desconfiar da precisão das ferramentas de IA, contra 33% que confiam \[3\].

As evidências disponíveis apontam nessa direção. No nível do PR, *smells* dominam os problemas introduzidos por PRs de correção de agentes já aceitos \[8\], e o código de agentes apresenta quase o dobro de redundância semântica do código humano, sem que os revisores reajam negativamente \[9\]. No nível do repositório, estudos de diferenças-em-diferenças mostram que a adoção do Cursor aumentou de forma persistente avisos de análise estática e complexidade \[10\], e que a adoção de agentes autônomos elevou os avisos em cerca de 18% e a complexidade cognitiva em cerca de 39% nos seis meses seguintes \[11\]. No nível do *commit*, 22,7% dos problemas introduzidos por assistentes de IA ainda sobrevivem na versão mais recente dos repositórios \[12\].

**O problema.** Sabemos que a qualidade medida no repositório cai depois da adoção de agentes, mas não sabemos **onde** essa degradação ocorre nem **o que acontece com ela depois**. Primeiro, as estimativas no nível do repositório \[10, 11\] não distinguem o código escrito por agentes do código mantido por humanos. Elas não dizem se a degradação está confinada aos arquivos que os agentes tocaram ou se se espalha para o restante do sistema. A diferença tem consequência prática: no primeiro caso, basta um portão de qualidade nos PRs de agentes; no segundo, a revisão desses PRs não é suficiente. Segundo, o efeito foi observado por apenas seis meses, e os próprios autores apontam a necessidade de horizontes mais longos \[11\]. Os estudos que acompanham o ciclo de vida da dívida tratam de *commits* de assistentes em geral, sem isolar PRs de agentes nem identificar quem paga a dívida \[12\]. Sem essas respostas, os mantenedores não têm base empírica para decidir onde concentrar revisão e refatoração quando passam a aceitar contribuições de agentes.

**Objetivo.** O objetivo geral deste trabalho é **analisar a origem e a persistência da degradação de manutenibilidade** em repositórios *open source* do GitHub que recebem PRs de agentes de codificação. Para alcançá-lo, os objetivos específicos são:

1.  **Selecionar**, a partir do subconjunto curado do AIDev (2.807 repositórios com mais de 100 estrelas \[4\]), uma amostra de repositórios com histórico suficiente antes e depois da adoção, e **estendê-la** com métricas mensais de qualidade.

2.  **Estimar** o efeito da adoção de agentes sobre complexidade, duplicação, *code smells*, avisos de análise estática e dívida técnica auto admitida (SATD) em um horizonte superior a 12 meses.

3.  **Comparar**, dentro de cada projeto, a evolução dos arquivos alterados por agentes com a dos arquivos alterados apenas por humanos.

4.  **Avaliar** se o efeito varia entre ferramentas e com a intensidade de uso dos agentes.

5.  **Analisar** o ciclo de vida dos problemas introduzidos por PRs de agentes: que fração é removida, em quanto tempo e por quem (humanos ou agentes).

Esses objetivos se desdobram em quatro questões de pesquisa (QP), apresentadas na Tabela 1.

**Tabela 1 – Questões de pesquisa**

| **QP** | **Pergunta**                                                                                                                                   | **Objetivo** |
|--------|------------------------------------------------------------------------------------------------------------------------------------------------|--------------|
| QP1    | O aumento de complexidade e de avisos após a adoção de agentes \[11\] se mantém, se acentua ou se reverte em um horizonte superior a 12 meses? | 1,2          |
| QP2    | A degradação se concentra nos arquivos tocados por agentes ou também atinge arquivos mantidos apenas por humanos?                              | 3            |
| QP3    | O efeito difere entre agentes e entre níveis de intensidade de uso?                                                                            | 4            |
| QP4    | Que fração dos problemas introduzidos por PRs de agentes é removida, em quanto tempo e por quem?                                               | 5            |

A QP1 é uma replicação deliberada de Agarwal et al. \[11\] em outra amostra e com horizonte maior. As QP2 e QP4 constituem a contribuição original do estudo.

**Contribuições previstas.** (i) Um conjunto de métricas mensais de qualidade para os repositórios selecionados do AIDev, com atribuição de cada arquivo a agentes ou humanos, publicado com o pacote de replicação (Seção 3). (ii) Uma decomposição do efeito da adoção por proveniência do código, que indica se a degradação está no código dos agentes ou se espalha (Seção 4, QP2). (iii) Uma caracterização de quanto da dívida introduzida por agentes é paga e por quem (Seção 4, QP4). (iv) Recomendações para mantenedores sobre onde aplicar portões de qualidade e revisão (Seção 5).

O restante do artigo está organizado da seguinte forma. A Seção 2 discute os trabalhos relacionados e posiciona este estudo. A Seção 3 descreve o desenho do estudo. A Seção 4 apresenta os resultados por questão de pesquisa. A Seção 5 discute implicações e ameaças à validade, e a Seção 6 conclui.

# **2 Trabalhos Relacionados**

Organizamos a literatura em quatro linhas, conforme a unidade de análise: o trecho de código gerado, o PR do agente, o repositório e o ciclo de vida da dívida. Cada subseção termina com o que este estudo acrescenta. A Tabela 2 resume a comparação.

## **2.1 Qualidade de código gerado por LLMs em ambientes controlados**

Os primeiros estudos avaliaram assistentes em problemas isolados. Yetiştiren et al. \[13\] mediram validade, corretude e eficiência do código do GitHub Copilot em problemas de programação. Nikolaidis et al. \[14\] compararam ChatGPT e Copilot em 60 problemas Python e observaram que as soluções costumam executar, mas trazem erros lógicos e exigem refinamento. Chen e Jiang \[15\] levaram a avaliação para agentes, analisando os *patches* de dez agentes em *issues* reais do SWE-bench com métricas de análise estática. Fu et al. \[16\] deram um passo em direção a projetos reais e identificaram fragilidades de segurança em código do Copilot presente em repositórios do GitHub.

Esses trabalhos estabelecem que código gerado por LLMs carrega problemas de qualidade, mas medem o código no momento em que é produzido. **Este estudo**, em vez disso, observa o código depois que ele é integrado a um projeto real e acompanha sua evolução por mais de 12 meses.

## **2.2 Qualidade de PRs de agentes em repositórios reais**

Com o AIDev \[2, 4\] e conjuntos de dados semelhantes, tornou-se possível estudar agentes atuando em projetos reais. Watanabe et al. \[5\] analisaram 567 PRs do Claude Code e mostraram que 83,8% foram aceitos pelos mantenedores. Cynthia et al. \[8\] aplicaram o SonarQube antes e depois de 1.210 PRs de correção aceitos em repositórios Python e encontraram *smells* como o tipo de problema dominante. As diferenças entre agentes praticamente desaparecem quando o número de problemas é normalizado pelo tamanho da mudança. Huang et al. \[9\] mostraram que PRs de agentes contêm quase o dobro de redundância semântica dos PRs humanos (0,287 contra 0,153 em sua métrica, medida em um repositório), enquanto revisores reagem a eles de forma mais neutra ou positiva. Ezaki et al. \[17\] analisaram 525 comentários de SATD escritos por agentes e encontraram predominância de dívida de requisitos e de projeto. Horikawa et al. \[18\] estudaram refatorações feitas por agentes.

Esses estudos tiram uma fotografia de cada PR no momento do *merge*. Os próprios autores apontam a falta de uma linha de base humana pareada e de efeitos posteriores de manutenção como limitações \[8\]. **Este estudo** usa a mesma fonte de dados, mas acompanha os arquivos tocados por agentes nos meses seguintes ao *merge* e compara sua evolução com a de arquivos mantidos apenas por humanos no mesmo projeto (QP2).

## **2.3 Efeitos da adoção de IA no nível do repositório**

Uma terceira linha estima como a adoção de ferramentas de IA altera a trajetória de projetos inteiros. O relatório da GitClear \[19\], com 211 milhões de linhas alteradas entre 2020 e 2024, associa a popularização da IA ao aumento de duplicação e de *churn* e à queda de refatoração, mas com dados majoritariamente proprietários e análise correlacional. He et al. \[10\] usaram diferenças-em-diferenças com adoção escalonada para comparar cerca de 800 repositórios que adotaram o Cursor a um grupo controle pareado. A adoção trouxe aumento de velocidade grande e transitório, e aumento persistente de avisos de análise estática e de complexidade, que por sua vez explica parte da desaceleração posterior.

O trabalho mais próximo do nosso é o de Agarwal et al. \[11\], que aplicaram o mesmo desenho ao AIDev, definindo a adoção como o primeiro PR de agente. Com 518 repositórios tratados (mínimo de 10 estrelas e 10 PRs de agentes) e controles pareados, os autores estimaram, nos seis meses após a adoção, aumento de cerca de 18% nos avisos de análise estática e de cerca de 39% na complexidade cognitiva. O ganho de velocidade só apareceu em projetos que não usavam IA antes. Os autores tratam suas estimativas como efeitos de intenção de tratamento, reconhecem coeficientes pré-tratamento significativos isolados e recomendam acompanhar os projetos por horizontes mais longos.

Esses estudos medem o efeito no repositório como um todo e não separam o código escrito por agentes do código mantido por humanos. **Este estudo** replica o desenho de Agarwal et al. com horizonte superior a 12 meses (QP1) e, principalmente, decompõe o efeito por proveniência do arquivo, usando os diffs por arquivo do subconjunto curado do AIDev (QP2).

## **2.4 Ciclo de vida da dívida técnica e IA**

A literatura de evolução de software mostra que *smells* tendem a surgir cedo e, na maioria dos casos, não são removidos \[7\], e que comentários de SATD tornam visível a dívida que os próprios desenvolvedores reconhecem \[20\]. Liu et al. \[12\] levaram essa perspectiva para a IA: analisaram 302,6 mil *commits* de cinco assistentes em 6.299 repositórios, rastrearam cada problema introduzido até a versão mais recente e constataram que 89,3% dos problemas são *smells* e que 22,7% ainda sobrevivem. Popescu et al. \[21\] estimaram curvas de sobrevivência e *churn* de código de agentes e de humanos e observaram mais *churn* ao longo do tempo no código de agentes. Do lado da dívida autoadmitida, Al Mujahid e Imran \[22\] identificaram comentários em que desenvolvedores reconhecem dívida em código gerado por IA, e Xiao et al. \[23\] caracterizaram como projetos *open source* declaram o uso de IA generativa.

Liu et al. estudam *commits* de assistentes em geral, e Popescu et al. medem a estabilidade de linhas, não problemas de qualidade. Nenhum dos dois identifica **quem** remove a dívida. **Este estudo** rastreia os problemas introduzidos especificamente por PRs de agentes e identifica se sua remoção é feita por humanos ou por outros agentes (QP4).

## **2.5 Posicionamento**

**Tabela 2 – Comparação com os trabalhos mais próximos**

| **Trabalho**          | **Unidade de análise**    | **Fonte**                  | **Horizonte após a integração** | **Separa código de agente e humano no mesmo projeto?** | **Rastreia remoção da dívida?**       |
|-----------------------|---------------------------|----------------------------|---------------------------------|--------------------------------------------------------|---------------------------------------|
| Cynthia et al. \[8\]  | PR de correção            | AIDev                      | No *merge*                      | Não                                                    | Não                                   |
| Huang et al. \[9\]    | PR                        | AIDev                      | No *merge*                      | Parcial (compara PRs)                                  | Não                                   |
| He et al. \[10\]      | Repositório (Cursor)      | GitHub                     | Cerca de 6 meses                | Não                                                    | Não                                   |
| Agarwal et al. \[11\] | Repositório (agentes)     | AIDev                      | 6 meses                         | Não                                                    | Não                                   |
| Liu et al. \[12\]     | *Commit* de assistente    | GitHub                     | Até a última versão             | Não                                                    | Sobrevivência, sem autoria da remoção |
| Popescu et al. \[21\] | Linhas de PRs             | Dataset próprio            | Longitudinal                    | Sim (*churn*), sem métricas de qualidade               | Não                                   |
| **Este estudo**       | **Repositório e arquivo** | **AIDev (\>100 estrelas)** | **Mais de 12 meses**            | **Sim**                                                | **Sim, com autoria da remoção**       |

Em síntese, a literatura já mostrou que agentes introduzem problemas de manutenibilidade no momento do PR e que a qualidade agregada dos repositórios piora após a adoção. Falta ligar os dois níveis: saber se a piora do repositório vem do código que os agentes escreveram ou se se espalha para o código humano, e se essa dívida é paga ao longo do tempo. Este estudo preenche essa lacuna combinando três elementos que ainda não aparecem juntos: atribuição por arquivo, horizonte superior a 12 meses e rastreamento de quem remove a dívida.

# **Referências**

\[1\] HASSAN, A. E. et al. Towards AI-Native Software Engineering (SE 3.0): A Vision and a Challenge Roadmap. arXiv:2410.06107, 2024. Disponível em: <https://arxiv.org/abs/2410.06107>.

\[2\] LI, H.; ZHANG, H.; HASSAN, A. E. The Rise of AI Teammates in Software Engineering (SE) 3.0: How Autonomous Coding Agents Are Reshaping Software Engineering. arXiv:2507.15003, 2025. Disponível em: <https://arxiv.org/abs/2507.15003>.

\[3\] STACK OVERFLOW. 2025 Developer Survey: AI. 2025. Disponível em: <https://survey.stackoverflow.co/2025/ai>.

\[4\] LI, H.; ZHANG, H.; HASSAN, A. E. AIDev: Studying AI Coding Agents on GitHub. In: International Conference on Mining Software Repositories (MSR), Rio de Janeiro, 2026. Disponível em: <https://arxiv.org/abs/2602.09185>.

\[5\] WATANABE, M. et al. On the use of agentic coding: An empirical study of pull requests on GitHub. arXiv:2509.14745, 2025.

\[6\] PALOMBA, F. et al. On the diffuseness and the impact on maintainability of code smells: a large scale empirical investigation. In: International Conference on Software Engineering (ICSE), 2018.

\[7\] TUFANO, M. et al. When and why your code starts to smell bad (and whether the smells go away). *IEEE Transactions on Software Engineering*, v. 43, n. 11, p. 1063–1088, 2017.

\[8\] CYNTHIA, S. T.; MUTTAKIN, A.; ROY, B. Beyond Bug Fixes: An Empirical Investigation of Post-Merge Code Quality Issues in Agent-Generated Pull Requests. In: MSR 2026 (Mining Challenge). Disponível em: <https://arxiv.org/abs/2601.20109>.

\[9\] HUANG, H. et al. More Code, Less Reuse: Investigating Code Quality and Reviewer Sentiment towards AI-generated Pull Requests. In: MSR 2026 (Mining Challenge). Disponível em: <https://arxiv.org/abs/2601.21276>.

\[10\] HE, H.; MILLER, C.; AGARWAL, S.; KÄSTNER, C.; VASILESCU, B. Speed at the Cost of Quality: How Cursor AI Increases Short-Term Velocity and Long-Term Complexity in Open-Source Projects. In: MSR 2026. Disponível em: <https://arxiv.org/abs/2511.04427>.

\[11\] AGARWAL, S.; HE, H.; VASILESCU, B. AI IDEs or Autonomous Agents? Measuring the Impact of Coding Agents on Software Development. In: MSR 2026. Disponível em: <https://arxiv.org/abs/2601.13597>.

\[12\] LIU, Y. et al. Debt Behind the AI Boom: A Large-Scale Empirical Study of AI-Generated Code in the Wild. arXiv:2603.28592, 2026. Disponível em: <https://arxiv.org/abs/2603.28592>.

\[13\] YETIŞTIREN, B.; ÖZSOY, I.; TÜZÜN, E. Assessing the quality of GitHub Copilot’s code generation. In: International Conference on Predictive Models and Data Analytics in Software Engineering (PROMISE), 2022, p. 62–71.

\[14\] NIKOLAIDIS, N. et al. A Comparison of the Effectiveness of ChatGPT and Co-Pilot for Generating Quality Python Code Solutions. In: SANER Companion, 2024, p. 93–101.

\[15\] CHEN, Z.; JIANG, L. Evaluating software development agents: Patch patterns, code quality, and issue complexity in real-world GitHub scenarios. In: SANER, 2025, p. 657–668.

\[16\] FU, Y. et al. Security weaknesses of Copilot-generated code in GitHub projects: An empirical study. *ACM Transactions on Software Engineering and Methodology*, v. 34, n. 8, 2025.

\[17\] EZAKI et al. Characterizing Self-Admitted Technical Debt Generated by AI Coding Agents. In: MSR 2026 (Mining Challenge).

\[18\] HORIKAWA, K. et al. Agentic Refactoring: An Empirical Study of AI Coding Agents. arXiv:2511.04824, 2025.

\[19\] GITCLEAR. AI Copilot Code Quality: 2025 Research. Relatório técnico, 2025. Disponível em: <https://www.gitclear.com/ai_assistant_code_quality_2025_research>.

\[20\] POTDAR, A.; SHIHAB, E. An exploratory study on self-admitted technical debt. In: International Conference on Software Maintenance and Evolution (ICSME), 2014.

\[21\] POPESCU, R. M. et al. Investigating Autonomous Agent Contributions in the Wild: Activity Patterns and Code Change over Time. In: MSR 2026. Disponível em: <https://arxiv.org/abs/2604.00917>.

\[22\] AL MUJAHID, A.; IMRAN, M. M. “TODO: Fix the Mess Gemini Created”: Towards Understanding GenAI-Induced Self-Admitted Technical Debt. In: International Conference on Technical Debt (TechDebt), 2026. Disponível em: <https://arxiv.org/abs/2601.07786>.

\[23\] XIAO, T. et al. Self-Admitted GenAI Usage in Open-Source Software. arXiv:2507.10422, 2025.
