# Desafio Técnico — LogiTrack Pro | Estágio em Qualidade

Análise de qualidade do **LogiTrack Pro**, sistema web de gestão de frotas
(veículos, viagens, manutenções e indicadores operacionais), realizada em 27 e 28 de
setembro de 2026.

**Candidato:** `Daniel Rannyeri da Silva Rocha`
**Vaga:** Estágio em Qualidade — LogAp I.T. Solutions

---

## Resumo da análise

| Indicador | Resultado |
|---|---|
| Casos de teste planejados | 24 |
| Casos executados | 24 |
| Aprovados | 12 |
| Reprovados | 12 |
| Taxa de aprovação | 50,0% |
| Defeitos registrados | 8 |
| Oportunidades de UX identificadas | 6 |

**Defeitos por severidade:** 1 Crítica · 3 Altas · 4 Médias

**Conclusão:** a aplicação **não está apta** a ser disponibilizada aos usuários na
versão avaliada. O fator determinante é o defeito BUG-01, que permite autenticação
com qualquer senha para um e-mail cadastrado. A análise completa está em
[`docs/04-parecer-de-qualidade.md`](docs/04-parecer-de-qualidade.md).

---

## Por onde começar

Se você tiver pouco tempo, leia nesta ordem:

1. [`docs/04-parecer-de-qualidade.md`](docs/04-parecer-de-qualidade.md) — resultados,
   análise de causa e conclusão sobre a aptidão do sistema
2. [`docs/01-relatorio-de-bugs.md`](docs/01-relatorio-de-bugs.md) — os 8 defeitos,
   com impacto e passos de reprodução
3. `Casos_de_Teste_LogiTrack.xlsx` — a suíte completa e os indicadores de execução

---

## Organização dos materiais

```
.
├── README.md
├── Casos_de_Teste_LogiTrack.xlsx        Suíte de testes, execução e indicadores
├── docs/
│   ├── 01-relatorio-de-bugs.md          Defeitos, impacto e reprodução
│   ├── 02-analise-ux.md                 Oportunidades de melhoria na experiência
│   ├── 03-estrategia-testes.md          Proposta de testes adicionais
│   └── 04-parecer-de-qualidade.md       Conclusão sobre a qualidade da aplicação
└── evidencias/                          Capturas de tela dos defeitos
```

### Atendimento aos itens do desafio

| Item solicitado | Onde está |
|---|---|
| A — Planejamento e execução dos cenários de teste | `Casos_de_Teste_LogiTrack.xlsx`, aba *Casos de Teste* |
| A — Descrição dos comportamentos incorretos, impacto e reprodução | [`docs/01-relatorio-de-bugs.md`](docs/01-relatorio-de-bugs.md) |
| B — Análise de experiência do usuário | [`docs/02-analise-ux.md`](docs/02-analise-ux.md) |
| C — Estratégia de testes adicionais | [`docs/03-estrategia-testes.md`](docs/03-estrategia-testes.md) |
| Conclusões sobre a qualidade da aplicação | [`docs/04-parecer-de-qualidade.md`](docs/04-parecer-de-qualidade.md) |
| Evidências | `evidencias/` |

### A planilha

| Aba | Conteúdo |
|---|---|
| `Instrucoes` | Premissas, ambiente e convenções de preenchimento |
| `Testadores` | Equipe da rodada, módulos atribuídos e ambiente de cada testador |
| `Casos de Teste` | Suíte completa, agrupada por módulo |
| `Resumo da Execução` | Indicadores e gráficos calculados automaticamente |

Cada caso contém os nove itens exigidos no desafio: identificação, objetivo,
pré-condições, dados utilizados, passos, resultado esperado, resultado obtido,
status e evidência. Foram acrescentados módulo, prioridade, testador e data de
execução, para permitir a leitura dos indicadores por recorte.

---

## Ferramentas utilizadas

| Ferramenta | Uso |
|---|---|
| Navegador + DevTools | Execução manual dos casos, inspeção de console e verificação de responsividade |
| Microsoft Excel | Documentação da suíte, controle de execução e indicadores |
| Markdown | Relatório de defeitos, análise de UX, estratégia e parecer |
| Ferramenta de captura de tela | Registro das evidências |

### Ambiente de execução

- **URL:** https://logitrack.danieldiegosantana.me/
- **Usuário:** logap@teste.com
- **Navegador:** Chrome 129.0.6668.101
- **Sistema operacional:** Windows 11 Pro 23H2
- **Resolução:** 1920x1080
- **Período:** 27 e 28 de setembro de 2026

---

## Premissas consideradas

A aplicação foi disponibilizada sem documentação de requisitos, especificação
funcional ou critérios de aceite. Os **resultados esperados** de cada caso foram
derivados de três fontes, nesta ordem de precedência:

1. **Convenções da própria interface** — placeholders, máscaras, rótulos e opções
   dos formulários. Exemplo: o placeholder `ABC-1234 ou ABC1D23` no campo Placa
   define os dois formatos aceitos pelo sistema.
2. **Comportamento já implementado em outros módulos.** Onde uma regra existe em um
   módulo e falta em outro, adotou-se o comportamento implementado como referência
   do esperado. Exemplo: a validação de coerência entre datas, presente em
   Manutenção e ausente em Viagens.
3. **Regras de negócio do domínio de gestão de frotas e boas práticas de
   integridade de dados** — distância percorrida sempre positiva, ano de fabricação
   dentro de intervalo plausível, campos obrigatórios sinalizados.

As premissas específicas de cada cenário estão registradas na coluna
**Pré-condições** do caso correspondente. Dois defeitos — BUG-06 e BUG-07 — decorrem
de regras inferidas e não confirmadas pela interface; ambos trazem registro
explícito dessa condição e da possibilidade de reclassificação.

---

## Critério de registro de evidências

Foram registradas nove capturas: **uma por defeito**, escolhida por demonstrar o
comportamento incorreto sem depender de contexto adicional — em geral o estado
final, que comprova a persistência do dado inválido.

Uma delas refere-se a CT-020, caso aprovado, por evidenciar que a regra de coerência
entre datas está implementada em Manutenção e ausente em Viagens, o que sustenta a
análise de causa registrada em BUG-04.

Os demais casos executados estão documentados na planilha, com resultado obtido,
status, testador e data de execução.

---

## Testes automatizados

Não foram implementados testes automatizados neste ciclo. O esforço disponível foi
direcionado à cobertura manual e à documentação da análise.

A proposta de automação — com escopo sugerido, ferramentas, prioridade e
justificativa — está descrita em
[`docs/03-estrategia-testes.md`](docs/03-estrategia-testes.md), itens 2, 3 e 5.

---

## Limitações do ciclo

- **Perfil único de acesso.** Apenas uma credencial foi disponibilizada, o que
  impediu validar segregação de permissões entre perfis de usuário.
- **Ambiente compartilhado.** A base contém massa de dados de execuções anteriores
  de terceiros — registros nomeados `Modelo QA Automatizado`, `Teste API Ano` e
  `Veículo Teste Cypress`. Por essa razão, contagens absolutas não foram utilizadas
  como critério de aprovação em nenhum caso.
- **Escopo funcional.** Testes de carga, desempenho e segurança aprofundada não
  foram realizados, conforme justificado no parecer.