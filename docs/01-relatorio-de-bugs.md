# Relatório de Defeitos — LogiTrack Pro

Defeitos identificados durante a execução da suíte de testes, em 27 e 28 de setembro
de 2026. Cada registro traz a descrição do problema, as condições de reprodução e o
impacto operacional.

**Severidade** mede a consequência técnica e operacional do defeito.
**Prioridade** mede a urgência de correção, considerando também custo de correção e
visibilidade para o usuário. As duas dimensões são independentes.

| ID | Título | Módulo | Severidade | Prioridade | Casos |
|---|---|---|---|---|---|
| BUG-01 | Autenticação aceita qualquer senha para e-mail cadastrado | Acesso | **Crítica** | Crítica | CT-002 |
| BUG-02 | Campo Ano sem validação de intervalo e sem obrigatoriedade | Veículos | Alta | Alta | CT-005, CT-006, CT-009 |
| BUG-03 | Campo de quilometragem aceita valor negativo | Viagens | Alta | Alta | CT-014 |
| BUG-04 | Viagem aceita data de chegada anterior à data de saída | Viagens | Alta | Alta | CT-015 |
| BUG-05 | Dashboard exige recarregamento manual para refletir novos registros | Dashboard | Média | Média | CT-024 |
| BUG-06 | Mesmo veículo pode ser alocado em viagens simultâneas | Viagens | Média | Média | CT-016 |
| BUG-07 | Veículo em manutenção é liberado para viagem | Viagens | Média | Média | CT-017 |
| BUG-08 | Filtro do Dashboard aplica-se a apenas um componente | Dashboard | Média | Média | CT-022 |

**Resumo por severidade:** 1 Crítica · 3 Altas · 4 Médias.

---

## BUG-01 — Autenticação aceita qualquer senha para e-mail cadastrado

| | |
|---|---|
| **Módulo** | Acesso |
| **Severidade** | **Crítica** |
| **Prioridade** | Crítica |
| **Casos relacionados** | CT-002 |
| **Ambiente** | `[navegador/versão]` · `[SO]` · `[resolução]` |

### Descrição

O sistema não valida a senha informada no login. Qualquer valor no campo Senha
concede acesso, desde que o e-mail corresponda a uma conta existente. A
autenticação está, na prática, reduzida à verificação de existência do e-mail.

### Pré-condições

Conhecer um endereço de e-mail cadastrado no sistema.

### Passos para reprodução

1. Acessar a tela de login da aplicação
2. Informar um e-mail cadastrado — por exemplo, `logap@teste.com`
3. Informar uma senha arbitrária e diferente da correta — por exemplo, `senhaErrada123`
4. Clicar em **Entrar**

### Resultado esperado

O acesso é negado, o sistema permanece na tela de login e exibe mensagem genérica
de credenciais inválidas.

### Resultado obtido

O usuário é autenticado e redirecionado para o Dashboard, com acesso integral a
todos os módulos do sistema.

### Impacto

Este é o defeito de maior gravidade do ciclo. O controle de acesso do sistema está
efetivamente ausente: qualquer pessoa que conheça ou deduza um endereço de e-mail
cadastrado obtém acesso total à operação da frota, com permissão de leitura,
alteração e exclusão de veículos, viagens e manutenções.

O risco é amplificado por dois fatores observados na aplicação:

- O e-mail costuma seguir padrão corporativo previsível, o que dispensa qualquer
  técnica de descoberta sofisticada.
- Existe rota de cadastro público em `/register`, o que sugere que a base de
  usuários pode ser ampliada por terceiros.

Além do risco de acesso indevido, o defeito compromete a rastreabilidade: nenhuma
ação registrada no sistema pode ser atribuída com confiança a um usuário
específico, já que a identidade não é comprovada no login.

Recomenda-se tratamento imediato, anterior a qualquer outra correção desta lista, e
antes de qualquer disponibilização da aplicação a usuários reais.

### Evidências

- `evidencias/CT-002_bypass-autenticacao.png` — Dashboard carregado após autenticação
  com senha incorreta

---

## BUG-02 — Campo Ano sem validação de intervalo e sem obrigatoriedade

| | |
|---|---|
| **Módulo** | Veículos |
| **Severidade** | Alta |
| **Prioridade** | Alta |
| **Casos relacionados** | CT-005, CT-006, CT-009 |
| **Ambiente** | `[navegador/versão]` · `[SO]` · `[resolução]` |

### Descrição

O campo *Ano* do cadastro de veículo não aplica nenhuma validação. Foram aceitos:

- valores negativos (`-2000`);
- o valor `0`, que é também o valor padrão exibido no campo;
- valores fora de qualquer intervalo plausível (`3000`);
- a submissão sem preenchimento do campo.

Os demais campos do mesmo formulário são validados corretamente: a placa rejeita
formatos inválidos e normaliza minúsculas, e placa e modelo são obrigatórios. A
ausência de validação é específica do campo Ano.

### Passos para reprodução

1. Acessar a tela **Veículos**
2. Clicar em **Adicionar Veículo**
3. Preencher placa e modelo com dados válidos
4. Informar `-2000` no campo *Ano*
5. Clicar em **Criar**
6. Repetir o fluxo com o valor `3000` e, em seguida, mantendo o valor padrão `0`

### Resultado esperado

O cadastro é rejeitado com mensagem indicando que o ano deve estar dentro de um
intervalo válido, e o campo é tratado como obrigatório.

### Resultado obtido

Todos os cadastros são concluídos com sucesso. O veículo passa a constar na
listagem com o ano informado, inclusive negativo.

### Impacto

O ano de fabricação é insumo do cálculo de idade média da frota, indicador usado em
decisões de renovação, em políticas de manutenção preventiva e em requisitos
contratuais de transporte.

O valor padrão `0` agrava o defeito de forma relevante: um cadastro submetido sem
atenção ao campo grava silenciosamente um ano inválido. O erro deixa de ser pontual
e passa a ser sistemático — tende a se acumular na base proporcionalmente ao volume
de cadastros.

### Evidências

- `evidencias/CT-005_ano-negativo.png` — listagem de Veículos exibindo o registro
  cadastrado com ano `-2000`

---

## BUG-03 — Campo de quilometragem aceita valor negativo

| | |
|---|---|
| **Módulo** | Viagens |
| **Severidade** | Alta |
| **Prioridade** | Alta |
| **Casos relacionados** | CT-014 |
| **Ambiente** | `[navegador/versão]` · `[SO]` · `[resolução]` |

### Descrição

O campo *Quilometragem Percorrida (KM)* do cadastro de viagem não aplica validação
de valor mínimo, aceitando distâncias negativas.

### Passos para reprodução

1. Acessar a tela **Viagens**
2. Clicar em **Nova Viagem**
3. Selecionar um veículo e preencher origem, destino e datas com dados válidos
4. Informar `-525` no campo *Quilometragem Percorrida (KM)*
5. Clicar em **Adicionar**

### Resultado esperado

O cadastro é rejeitado com mensagem indicando que a quilometragem deve ser maior
que zero.

### Resultado obtido

A viagem é cadastrada e exibida na listagem com `-525 km`.

### Impacto

A quilometragem alimenta o indicador *Total de KM Percorrido* do Dashboard, que
sustenta decisões de manutenção preventiva, cálculo de custo por quilômetro e
renovação de frota.

A validação de soma realizada em CT-023 confirmou que o indicador reflete
fielmente a soma das viagens — o que, neste contexto, é agravante: o Dashboard
propaga corretamente um dado incorreto. Um único registro negativo reduz o total do
veículo e pode mascarar uso acima do previsto, sem qualquer sinalização de
inconsistência.

### Evidências

- `evidencias/CT-014_km-negativa.png` — listagem de Viagens exibindo o registro com
  `-525 km`

---

## BUG-04 — Viagem aceita data de chegada anterior à data de saída

| | |
|---|---|
| **Módulo** | Viagens |
| **Severidade** | Alta |
| **Prioridade** | Alta |
| **Casos relacionados** | CT-015 |
| **Ambiente** | `[navegador/versão]` · `[SO]` · `[resolução]` |

### Descrição

O cadastro de viagem não valida a coerência cronológica entre as datas de saída e
chegada. Uma viagem que termina antes de começar é aceita, e o sistema ainda exibe
mensagem de confirmação de sucesso.

**Observação relevante:** a mesma regra está corretamente implementada no módulo
Manutenção. Em CT-020, o agendamento com data de finalização anterior à de início
foi bloqueado com a mensagem *"Data de finalização não pode ser anterior à data de
início"*. Trata-se, portanto, de regra já especificada e já implementada no sistema,
porém ausente em Viagens.

### Passos para reprodução

1. Acessar a tela **Viagens**
2. Clicar em **Nova Viagem**
3. Selecionar um veículo e preencher origem e destino
4. Informar data de saída `26/07/2026` e data de chegada `25/07/2026`
5. Clicar em **Adicionar**

### Resultado esperado

O cadastro é bloqueado com mensagem indicando a inconsistência entre as datas, a
exemplo do comportamento já adotado no módulo Manutenção.

### Resultado obtido

A viagem é cadastrada e o sistema exibe a mensagem *"Viagem atualizada com
sucesso!"*.

### Impacto

Compromete qualquer análise temporal da operação: tempo médio de viagem,
disponibilidade de veículo por período e detecção de sobreposição de alocação.

O agravante está na mensagem de sucesso: o sistema não apenas aceita o dado
inconsistente, mas confirma ativamente ao usuário que a operação foi bem-sucedida,
eliminando a chance de percepção do erro no momento do cadastro.

A comparação com o módulo Manutenção indica que a causa não é ausência de regra
definida, mas validação implementada de forma isolada por tela, sem camada comum —
o que sugere que outros formulários podem apresentar lacunas semelhantes.

### Evidências

- `evidencias/CT-015_datas-invertidas-sucesso.png` — mensagem de confirmação exibida
  após o cadastro da viagem com cronologia invertida
- `evidencias/CT-020_manutencao-bloqueio-datas.png` — bloqueio correto da mesma regra
  no módulo Manutenção, para contraste

---

## BUG-05 — Dashboard exige recarregamento manual para refletir novos registros

| | |
|---|---|
| **Módulo** | Dashboard |
| **Severidade** | Média |
| **Prioridade** | Média |
| **Casos relacionados** | CT-024 |
| **Ambiente** | `[navegador/versão]` · `[SO]` · `[resolução]` |

### Descrição

O gráfico *Volume por Categoria* não incorpora viagens cadastradas durante a mesma
sessão de navegação. Ao retornar ao Dashboard após registrar novas viagens, os
valores permanecem inalterados. Os dados são atualizados apenas com o recarregamento
manual da página.

O cálculo no servidor, portanto, está correto: o defeito está na atualização dos
dados no cliente, que não são requisitados novamente ao voltar para a tela.

### Passos para reprodução

1. Acessar o **Dashboard** e anotar os valores do gráfico *Volume por Categoria*
2. Acessar a tela **Viagens** e cadastrar uma ou mais viagens válidas
3. Confirmar que os registros aparecem na listagem
4. Retornar ao **Dashboard** pelo menu lateral e comparar os valores do gráfico
5. Recarregar a página e comparar novamente

### Resultado esperado

O gráfico reflete o total atualizado de viagens ao ser acessado, sem exigir ação
adicional do usuário.

### Resultado obtido

Os valores permanecem inalterados na navegação pelo menu. Após o recarregamento
manual da página, o gráfico passa a exibir os dados corretos.

### Impacto

O usuário que cadastra registros e consulta o Dashboard na sequência — fluxo natural
de uso — visualiza indicadores desatualizados sem qualquer indicação de defasagem.
Não há data de referência, aviso de dados em cache ou botão de atualização.

O impacto é limitado por os dados corretos estarem a um recarregamento de distância
e por a informação se corrigir sozinha em acessos posteriores. Ainda assim,
compromete a confiança na tela: um indicador que às vezes está certo e às vezes está
defasado, sem sinalizar qual é o caso, exige que o usuário desconfie de todos.

### Evidências

- `evidencias/CT-024_dashboard-desatualizado.png` — gráfico *Volume por Categoria*
  inalterado após o cadastro de novas viagens, antes do recarregamento

---

## BUG-06 — Mesmo veículo pode ser alocado em viagens simultâneas

| | |
|---|---|
| **Módulo** | Viagens |
| **Severidade** | Média |
| **Prioridade** | Média |
| **Casos relacionados** | CT-016 |
| **Ambiente** | `[navegador/versão]` · `[SO]` · `[resolução]` |

### Descrição

O sistema permite registrar duas ou mais viagens para o mesmo veículo em períodos
sobrepostos, sem qualquer alerta de conflito de alocação.

### Passos para reprodução

1. Identificar uma viagem já cadastrada e o período que ela ocupa
2. Clicar em **Nova Viagem** e selecionar o mesmo veículo
3. Informar um período que se sobreponha ao da viagem existente
4. Clicar em **Adicionar**

### Resultado esperado

O sistema impede o cadastro e informa que o veículo já possui viagem registrada no
período, ou ao menos alerta sobre a sobreposição antes de confirmar.

### Resultado obtido

Ambas as viagens são cadastradas e coexistem na listagem.

### Impacto

Um veículo físico não pode realizar duas viagens simultaneamente. O sistema permite
registrar um estado operacionalmente impossível, o que compromete o planejamento de
alocação da frota e distorce indicadores de utilização — um veículo pode aparentar
disponibilidade ou uso incompatíveis com a realidade.

> Registro de premissa: esta regra foi inferida do domínio de gestão de frotas e não
> consta explicitamente da interface. Caso a operação admita registros paralelos por
> razão não identificada nesta análise, o item deve ser reclassificado de defeito
> para oportunidade de melhoria.

### Evidências

- `evidencias/CT-016_viagens-sobrepostas.png` — listagem exibindo duas viagens do
  mesmo veículo em período coincidente

---

## BUG-07 — Veículo em manutenção é liberado para viagem

| | |
|---|---|
| **Módulo** | Viagens |
| **Severidade** | Média |
| **Prioridade** | Média |
| **Casos relacionados** | CT-017 |
| **Ambiente** | `[navegador/versão]` · `[SO]` · `[resolução]` |

### Descrição

Veículos com manutenção não concluída permanecem disponíveis no seletor do cadastro
de viagem, e o sistema aceita a alocação sem alerta.

### Passos para reprodução

1. Consultar na tela **Manutenção** um veículo com manutenção de status `Pendente`
   ou `Em Realização`
2. Clicar em **Nova Viagem** e selecionar esse veículo
3. Cadastrar uma viagem no mesmo período da manutenção
4. Clicar em **Adicionar**

### Resultado esperado

O veículo em manutenção não é oferecido no seletor, ou o sistema impede o cadastro
informando a indisponibilidade.

### Resultado obtido

A viagem é cadastrada normalmente, sem qualquer indicação de conflito.

### Impacto

Compromete a finalidade de os módulos de Manutenção e Viagens operarem de forma
integrada. Operacionalmente, um veículo pode ser despachado enquanto está em
oficina, gerando conflito de agenda real. Do ponto de vista de segurança
operacional, permite alocar veículo cuja revisão preventiva está em aberto.

> Registro de premissa: assim como em BUG-06, esta regra foi inferida do domínio e
> não está explicitada na interface. Caso a empresa opere com manutenções que não
> imobilizam o veículo, o item deve ser reclassificado.

### Evidências

- `evidencias/CT-017_viagem-veiculo-em-manutencao.png` — viagem cadastrada para
  veículo com manutenção em aberto

---

## BUG-08 — Filtro do Dashboard aplica-se a apenas um componente

| | |
|---|---|
| **Módulo** | Dashboard |
| **Severidade** | Média |
| **Prioridade** | Média |
| **Casos relacionados** | CT-022 |
| **Ambiente** | `[navegador/versão]` · `[SO]` · `[resolução]` |

### Descrição

O seletor de veículo do Dashboard afeta somente o card *Total de KM Percorrido*. O
gráfico *Volume por Categoria* e o card *Cronograma de Manutenção* permanecem com o
escopo da frota completa, sem qualquer indicação de que não seguem o filtro.

### Passos para reprodução

1. Acessar o **Dashboard**
2. Anotar os valores dos três componentes da tela
3. Selecionar um veículo específico no seletor
4. Comparar novamente os valores dos três componentes

### Resultado esperado

Todos os componentes refletem o escopo definido pelo filtro, ou os componentes de
escopo global são identificados como tal na interface.

### Resultado obtido

Apenas o card *Total de KM Percorrido* é filtrado. Os demais componentes continuam
exibindo dados de toda a frota.

### Impacto

O usuário interpreta uma tela filtrada como um conjunto coerente. Componentes com
escopos distintos e sem sinalização levam à leitura incorreta dos indicadores: o
gestor pode atribuir a um veículo específico um volume de viagens ou uma agenda de
manutenção que pertencem à frota inteira.

### Evidências

- `evidencias/CT-022_filtro-dashboard.png` — Dashboard com veículo selecionado no
  filtro, com os demais componentes mantendo o escopo da frota completa

---

## Observações adicionais

### Exclusão de veículo com registros vinculados (CT-011)

A exclusão de um veículo produz tratamento divergente entre os módulos dependentes.
Ao excluir o veículo `JVC-1234`, as viagens vinculadas foram removidas junto com o
cadastro, enquanto as manutenções permaneceram na listagem do módulo Manutenção,
referenciando um veículo que já não existe.

O comportamento é inconsistente em si mesmo: nenhuma das duas políticas possíveis —
remoção em cascata ou preservação do histórico — foi aplicada integralmente. O
resultado é a perda do histórico de viagens somada à permanência de registros de
manutenção órfãos.

O item não foi registrado como defeito numerado neste ciclo por não ter evidência
capturada, mas está documentado na planilha de casos de teste e é relevante para a
análise de causa: assim como em BUG-04, indica regras implementadas de forma
isolada por tela, sem política comum de integridade referencial.

---

## Critério de registro de evidências

As evidências foram registradas de forma seletiva: **uma captura por defeito**,
escolhida por demonstrar o comportamento incorreto sem depender de contexto
adicional — em geral o estado final, que comprova a persistência do dado inválido.

Foram registradas nove capturas. Acrescenta-se, entre elas, uma de contraste
referente a CT-020, caso aprovado, por
evidenciar que a regra de coerência entre datas está implementada no módulo
Manutenção e ausente em Viagens, o que sustenta a análise de causa registrada em
BUG-04.

Os demais casos executados estão documentados na planilha
`Casos_de_Teste_LogiTrack_Pro.xlsx`, com resultado obtido, status, testador e data
de execução.

---

## Anexo — Comportamentos validados com sucesso

Registrados para dimensionar corretamente o estado da aplicação. Nem todo módulo
apresentou defeitos.

| Comportamento | Caso |
|---|---|
| Proteção de rotas internas contra acesso sem sessão ativa | CT-003 |
| Validação de formato de placa, com normalização automática para maiúsculas | CT-007 |
| Bloqueio de placa duplicada | CT-008 |
| Obrigatoriedade de placa e modelo no cadastro de veículo | CT-009 |
| Edição de veículo com persistência correta | CT-010 |
| Busca por placa e por modelo, indiferente a maiúsculas e minúsculas | CT-012 |
| Rejeição de custo estimado negativo em Manutenção | CT-019 |
| Bloqueio de datas incoerentes em Manutenção, com mensagem específica | CT-020 |
| Transição de status da manutenção, com persistência | CT-021 |
| Consistência entre o total de KM do Dashboard e a soma das viagens | CT-023 |

O módulo **Manutenção** foi o único a concluir o ciclo sem defeitos, com validação
de valor mínimo e de coerência entre datas — exatamente as regras ausentes em
Veículos e Viagens.