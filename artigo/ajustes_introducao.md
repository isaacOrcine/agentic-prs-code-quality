# Ajustes sugeridos na Introdução (e em dois pontos da Seção 2)

Pontos da entrega parcial que ficaram inconsistentes com o que foi feito nas Seções 3 e 4. O texto da Introdução não foi alterado.

## Introdução

1. **Objetivo 1 – amostra.** O texto fala em selecionar uma amostra a partir dos 2.807 repositórios do subconjunto curado e "estendê-la com métricas mensais". A seleção ficou restrita a repositórios **Python** (530 → 315 → 99 candidatos), e as métricas mensais existem só para os **5 projetos do piloto**. Convém dizer "repositórios Python" e que a extensão completa está em andamento.

2. **Objetivo 2 – "estimar o efeito".** O piloto é descritivo: sem grupo controle, sem testes e sem estimativa de efeito. Sugestão: "descrever a evolução... e, na amostra completa, estimar o efeito".

3. **Objetivo 2 – duplicação.** A duplicação **não foi medida**. Ou ela sai do objetivo, ou o objetivo deixa claro que será medida só na amostra completa (com ferramenta a definir).

4. **Objetivo 2 – *code smells*.** Não usamos um detector de *smells*: eles são aproximados pelas regras do Ruff (sobretudo PLR, SIM e C90). Vale dizer isso no objetivo ou na nota da Tabela 1.

5. **QP1 como "replicação deliberada" de Agarwal et al. [11].** O piloto não replica o desenho de [11]:
   - não usa diferenças-em-diferenças nem grupo controle;
   - usa o Ruff com regras próprias;
   - mede complexidade **ciclomática**, não cognitiva;
   - tem 5 projetos.

   A Tabela 1 também fala no "aumento de complexidade [11]", que lá é cognitiva. Sugestão: "replicação conceitual" ou "comparação com [11]", com a replicação do desenho prevista para a amostra completa.

6. **Objetivo 4 – "variar entre ferramentas".** No piloto, cada agente aparece praticamente em um só projeto, então agente e projeto se confundem. O objetivo pode ficar, mas o texto não deve prometer separar o efeito do agente sem a amostra completa.

7. **Objetivo 5 e QP4 – "problemas introduzidos por PRs de agentes".** O rastreamento parte de ***commits* atribuídos a agentes**, o que inclui *commits* identificados só por assinatura e posteriores ao corte do AIDev, e os "problemas" são avisos do Ruff. Sugestão: "problemas (avisos de análise estática) introduzidos por *commits* de agentes".

8. **Contribuição (i).** Ela promete "um conjunto de métricas mensais para os repositórios selecionados... publicado com o pacote de replicação". Hoje o conjunto cobre os 5 projetos do piloto.

9. **Contribuição (ii) – "decomposição do efeito".** O piloto decompõe **taxas descritivas** por proveniência, não um efeito estimado.

10. **Contribuição (iv) e último parágrafo – Seção 5.** A Introdução promete que a "Seção 5 discute implicações e ameaças à validade" e traz "recomendações para mantenedores (Seção 5)". Na versão atual:
    - a discussão e as ameaças estão nas **Seções 4.5 e 4.6**;
    - não há seção de recomendações;
    - a Seção 6 (conclusão) ainda não existe.

    É preciso ajustar o roteiro do artigo ou escrever as Seções 5 e 6.

11. **"Horizonte superior a 12 meses".** O piloto observa de 15 (DDNS) a 21 meses (PyAirbyte) a partir do mês da adoção, então a afirmação vale. Mas as faixas da QP1 só vão até o mês 15, e no DDNS até o 14. Convém dizer "até 15 meses nas comparações por faixa".

## Seção 2 (fora do escopo pedido, mas inconsistente)

12. **2.3, último parágrafo.** O texto diz que o estudo decompõe o efeito "usando os diffs por arquivo do subconjunto curado do AIDev". Na prática, os arquivos alterados vêm do **histórico Git** (`git log --first-parent --raw -M`) dos clones, e não de tabelas de diffs do AIDev.

13. **Tabela 2, linha "Este estudo".** "Sim, com autoria da remoção" e "Mais de 12 meses" valem para o piloto. A fonte, porém, é "AIDev (>100 estrelas)" restrita a **Python**, e a separação agente/humano é feita por atribuição de *commits* sem validação manual.
