# Parecer de Qualidade — LogiTrack Pro

**Responsável:** `Daniel Rannyeri da Silva Rocha`
**Testadores:** Daniel Rocha, Gabriel Passos
**Período de execução:** 27 e 28 de setembro de 2026
**Versão avaliada:** aplicação disponibilizada em https://logitrack.danieldiegosantana.me/

---

## 1. Escopo do ciclo

Foram planejados e executados **24 casos de teste** cobrindo cinco módulos: Acesso,
Veículos, Viagens, Manutenção e Dashboard. A suíte contempla caminhos felizes, casos
negativos, validações de campo e regras de negócio entre módulos.

A aplicação foi disponibilizada sem documentação de requisitos. Os resultados
esperados foram derivados das convenções da própria interface, das regras de
negócio do domínio de gestão de frotas e de boas práticas de integridade de dados,
conforme registrado no `README.md`. Onde uma regra foi inferida e não confirmada
pela interface, isso está indicado no caso de teste e no defeito correspondente.

### Fora de escopo

- **Segregação de permissões entre perfis.** Apenas uma credencial foi
  disponibilizada, o que impediu validar se existem níveis de acesso distintos.
- **Testes de carga e desempenho.** Não há informação sobre volumetria esperada em
  produção.
- **Testes de segurança aprofundados.** Exigem autorização formal e ambiente
  segregado. Registra-se que o defeito BUG-01 foi identificado em teste funcional
  de autenticação, e não em avaliação de segurança dedicada — o que sugere que uma
  análise específica é recomendável.
- **Validação em dispositivos móveis e em navegadores adicionais.**

---

## 2. Resultados

| Indicador | Valor |
|---|---|
| Casos planejados | 24 |
| Casos executados | 24 |
| Aprovados | 12 |
| Reprovados | 12 |
| Bloqueados | 0 |
| **Taxa de aprovação** | **50,0%** |
| Defeitos registrados | 8 |

### Distribuição por módulo

| Módulo | Executados | Aprovados | Reprovados | Taxa |
|---|---|---|---|---|
| Acesso | 3 | 2 | 1 | 66,7% |
| Veículos | 9 | 4 | 5 | 44,4% |
| Viagens | 5 | 1 | 4 | 20,0% |
| Manutenção | 4 | 4 | 0 | 100,0% |
| Dashboard | 3 | 1 | 2 | 33,3% |

### Defeitos por severidade

| Severidade | Quantidade | Defeitos |
|---|---|---|
| Crítica | 1 | BUG-01 |
| Alta | 3 | BUG-02, BUG-03, BUG-04 |
| Média | 4 | BUG-05, BUG-06, BUG-07, BUG-08 |
| Baixa | 0 | — |

---

## 3. Análise

### O defeito que define o ciclo

O BUG-01 — autenticação que aceita qualquer senha para um e-mail cadastrado —
está em categoria própria. Não se trata de uma falha de validação entre outras:
o controle de acesso do sistema está efetivamente ausente.

Qualquer pessoa que conheça ou deduza um endereço de e-mail cadastrado obtém acesso
integral à operação da frota, com permissão de leitura, alteração e exclusão. O
risco é ampliado pela existência de rota de cadastro público em `/register` e pela
previsibilidade dos endereços corporativos.

Há um agravante indireto: como a identidade não é comprovada no login, nenhuma ação
registrada no sistema pode ser atribuída com confiança a um usuário específico.
Isso compromete a rastreabilidade de toda a operação, independentemente de os
demais módulos funcionarem corretamente.

Chama atenção que o módulo Acesso apresente, simultaneamente, esse defeito e uma
proteção de rotas internas funcionando corretamente (CT-003): o sistema bloqueia
acesso por URL sem sessão, mas concede sessão sem verificar a senha. A camada de
autorização está implementada; a de autenticação, não.

### Padrão dos demais defeitos

Os sete defeitos restantes seguem um padrão único: **ausência de validação de dados
nos formulários de cadastro**. Campos numéricos sem limite mínimo (quilometragem,
ano de fabricação), campos sem obrigatoriedade definida (ano) e pares de datas sem
verificação de coerência (viagens).

O ponto central desta análise é que **não se trata de regras desconhecidas pela
equipe de desenvolvimento**. Duas evidências sustentam isso:

- O módulo **Manutenção** rejeita corretamente custo negativo (CT-019) e bloqueia
  data de finalização anterior à de início (CT-020), com mensagem específica. As
  mesmas regras estão ausentes em Viagens (BUG-03, BUG-04).
- No **mesmo formulário** de cadastro de veículo, a placa é validada corretamente —
  rejeita formatos inválidos, normaliza minúsculas, impede duplicidade — enquanto o
  campo Ano não recebe validação alguma (BUG-02).

A conclusão é que a validação foi implementada **campo a campo e tela a tela**, sem
camada comum. Isso tem consequência prática direta: corrigir os oito defeitos
listados resolve os sintomas conhecidos e deixa em aberto todos os campos e
formulários que não foram cobertos por esta suíte.

O mesmo padrão aparece fora da validação de campos. Ao excluir um veículo (CT-011),
as viagens vinculadas são removidas enquanto as manutenções permanecem órfãs — duas
políticas diferentes de integridade referencial convivendo na mesma operação.

### Impacto sobre os indicadores gerenciais

O Dashboard não valida os dados que agrega. A verificação de CT-023 confirmou que o
card *Total de KM Percorrido* reflete fielmente a soma das viagens do veículo — o
que, no contexto de BUG-03, é agravante: o indicador propaga corretamente um dado
incorreto, exibindo quilometragem total negativa sem qualquer sinalização.

Somam-se a isso dois defeitos de escopo e atualização: o filtro de veículo aplica-se
a apenas um dos três componentes da tela (BUG-08), e os dados só são atualizados com
recarregamento manual da página (BUG-05). Nenhum dos dois é individualmente grave,
mas ambos afetam a mesma tela — a de leitura executiva do sistema — e produzem o
mesmo efeito: o usuário não tem como saber se o que está vendo corresponde ao estado
real da operação.

### Pontos positivos

O ciclo identificou um volume relevante de defeitos, mas o sistema não é
uniformemente frágil:

- **O módulo Manutenção concluiu o ciclo sem nenhum defeito**, com validação de
  valor mínimo, verificação de coerência entre datas, mensagens de erro específicas
  e transição de status persistida corretamente. É a referência de implementação
  que os demais módulos deveriam seguir.
- **O cadastro de veículos apresenta a validação de placa bem construída**:
  rejeita formatos inválidos, normaliza automaticamente para maiúsculas, aceita os
  dois padrões brasileiros e bloqueia duplicidade com mensagem explícita.
- **Todos os fluxos de caminho feliz funcionaram.** Cadastro, edição e busca operam
  corretamente nos três módulos, com persistência confirmada após recarregamento.
- **A proteção de rotas internas está implementada** e bloqueia acesso por URL sem
  sessão ativa.
- **A navegação e a paginação são consistentes** entre as telas, e a aplicação não
  apresentou instabilidade ou erro não tratado durante a execução.

Isso é relevante para o encaminhamento: a base funcional está construída. O que
falta é a camada de consistência de dados e a correção do controle de acesso.

---

## 4. Conclusão

**A aplicação não está apta a ser disponibilizada aos usuários na versão avaliada.**

O fator determinante é o BUG-01. Um sistema que concede acesso sem verificar a senha
não pode ser exposto a usuários reais em nenhuma hipótese, independentemente da
qualidade dos demais módulos. Trata-se de bloqueador absoluto, não de item a ser
ponderado contra o restante do ciclo.

Ainda que esse defeito fosse corrigido isoladamente, a taxa de aprovação de 50% e a
concentração de falhas de validação em Veículos e Viagens indicam que a aplicação
requer um ciclo de correção antes da liberação. A natureza sistemática dos defeitos
— validação implementada por tela, sem camada comum — significa que correções
pontuais não oferecem garantia sobre os campos não testados.

### Encaminhamento recomendado

**1. Imediato, bloqueador**
Corrigir a verificação de senha no login (BUG-01) e avaliar a pertinência da rota
de cadastro público `/register` em um sistema de gestão interna. Enquanto não
corrigido, o acesso à aplicação deve permanecer restrito.

**2. Antes da liberação**
Corrigir os defeitos de severidade Alta (BUG-02, BUG-03, BUG-04) e, em vez de
tratá-los isoladamente, **estabelecer uma camada comum de validação** aplicada a
todos os formulários — replicando o padrão já adotado no módulo Manutenção. Definir
política única de integridade referencial para exclusão de registros com vínculos.

**3. Ciclo seguinte**
Corrigir os defeitos de severidade Média (BUG-05 a BUG-08), que afetam
principalmente a confiabilidade percebida dos indicadores do Dashboard. Implementar
as oportunidades de melhoria descritas em
[`02-analise-ux.md`](02-analise-ux.md), com prioridade para a sinalização de campos
obrigatórios, diretamente relacionada a BUG-02.

**4. Evolução do processo**
Adotar as recomendações de [`03-estrategia-testes.md`](03-estrategia-testes.md),
com prioridade para testes de API e testes unitários de validação. Ambos atacam
diretamente a causa raiz identificada e evitam a reintrodução do mesmo padrão de
defeito em funcionalidades futuras.

### Recomendação de revalidação

Após as correções, recomenda-se reexecução integral da suíte — e não apenas dos 12
casos reprovados. Alterações na camada de validação afetam formulários que hoje
aprovam, e a regressão precisa ser verificada.

---

## 5. Documentos relacionados

| Documento | Conteúdo |
|---|---|
| `Casos_de_Teste_LogiTrack.xlsx` | Suíte completa, execução e indicadores |
| [`01-relatorio-de-bugs.md`](01-relatorio-de-bugs.md) | Os 8 defeitos, com impacto e reprodução |
| [`02-analise-ux.md`](02-analise-ux.md) | Oportunidades de melhoria na experiência |
| [`03-estrategia-testes.md`](03-estrategia-testes.md) | Proposta de testes adicionais |
| `evidencias/` | Capturas de tela dos defeitos |