# Análise de Experiência do Usuário — LogiTrack Pro

Oportunidades de melhoria identificadas durante a análise da aplicação. O desafio
solicita três; foram registradas seis, por terem naturezas distintas e não se
sobreporem: prevenção de erros, visibilidade de status, acesso ao conteúdo,
hierarquia da informação, eficiência de uso e consistência textual.

A ordem reflete o impacto na operação, não a facilidade de correção. Os três
primeiros itens afetam a capacidade de executar tarefas; os três últimos afetam a
eficiência e a confiança no sistema.

As justificativas referenciam as heurísticas de usabilidade de Nielsen. Onde a
oportunidade se relaciona a um defeito registrado, a referência ao
[`01-relatorio-de-bugs.md`](01-relatorio-de-bugs.md) está indicada.

---

## UX-01 — Campos obrigatórios não são sinalizados nos formulários

### Funcionalidade ou tela analisada

Formulários de cadastro dos três módulos: *Adicionar Veículo*, *Adicionar Viagem* e
*Agendar Manutenção*.

### Situação identificada

Nenhum dos formulários indica quais campos são obrigatórios. Não há asterisco,
rótulo auxiliar ou qualquer marcação visual que diferencie campo obrigatório de
campo opcional.

A execução confirmou que a obrigatoriedade **existe e é aplicada**, mas apenas na
submissão: em CT-009, o cadastro foi corretamente bloqueado sem placa e sem modelo.
O usuário só descobre a regra ao ser interrompido.

O problema é agravado pelos campos numéricos *Ano* e *Custo Estimado*, apresentados
pré-preenchidos com `0`. Um valor padrão visível comunica que o campo já está
resolvido, quando `0` é inválido para ambos — o usuário é induzido a ignorá-lo.
Esse é exatamente o mecanismo que permite o registro silencioso de anos de
fabricação inválidos, documentado em **BUG-02**.

### Alteração recomendada

1. Marcar os campos obrigatórios com asterisco e incluir a legenda
   `* campo obrigatório` no topo do formulário.
2. Substituir o valor padrão `0` dos campos *Ano* e *Custo Estimado* por placeholder
   ilustrativo, seguindo o padrão já adotado nos campos de texto
   (`Ex: Volvo FH 540`, `Ex: 400.5`), mantendo o campo efetivamente vazio.
3. Validar ao sair do campo, e não apenas na submissão, permitindo correção imediata.
4. Exibir mensagens de erro específicas por campo, junto ao campo correspondente.

### Justificativa da melhoria

Viola a heurística de **prevenção de erros**: a interface não fornece a informação
necessária ao preenchimento correto antes que o erro ocorra. Compromete também
**reconhecimento em vez de memorização**, ao exigir que o usuário deduza as regras
por tentativa e erro a cada novo cadastro.

O valor padrão `0` não é apenas questão de usabilidade — é a condição que torna
provável um defeito de dados já confirmado no sistema.

### Benefício esperado para o usuário

Redução do número de tentativas até o cadastro válido, menor frustração em
formulários de uso frequente, e diminuição direta da entrada de dados inválidos na
base — o que reduz retrabalho de correção posterior.

---

## UX-02 — Manutenções vencidas não são diferenciadas das agendadas

### Funcionalidade ou tela analisada

Dashboard, card *Cronograma de Manutenção*; e tela **Manutenção**, coluna *Status*.

### Situação identificada

Manutenções preventivas com data de início já ultrapassada permanecem com o status
`Pendente`, exibido com o mesmo tratamento visual neutro de uma manutenção agendada
para os próximos dias. No cronograma do Dashboard, registros de 23 de julho aparecem
com o rótulo `67 dias atrás` ao lado de um badge `Pendente` em cinza.

O sistema calcula e exibe o atraso, mas não o trata como condição distinta: cabe ao
usuário ler o número de dias de cada card e concluir por conta própria quais exigem
ação.

A execução de CT-021 confirmou que a transição de status funciona corretamente —
`Pendente`, `Em Realização` e `Concluída` avançam e persistem. A lacuna não é
técnica: é a ausência de um estado que represente o atraso.

### Alteração recomendada

1. Introduzir o estado **Atrasado**, aplicado automaticamente quando a data de
   início prevista for anterior à data atual e o status ainda for `Pendente`.
2. Atribuir a esse estado destaque visual próprio e posicionamento prioritário na
   ordenação do cronograma.
3. Incluir filtro por status na listagem de Manutenção, permitindo isolar os
   registros atrasados.
4. Exibir no Dashboard um contador de manutenções em atraso.

### Justificativa da melhoria

Viola a heurística de **visibilidade do status do sistema**: a informação crítica
existe, mas não recebe peso visual proporcional à sua gravidade. Compromete também
**reconhecimento em vez de memorização**, ao exigir leitura e interpretação
individual de cada card em vez de percepção imediata.

Em gestão de frotas, manutenção preventiva vencida não é um registro desatualizado:
é um veículo circulando fora das condições previstas de segurança.

### Benefício esperado para o usuário

Identificação imediata dos veículos que exigem ação, redução do risco operacional
associado a revisões não realizadas e menor esforço cognitivo do gestor na rotina
diária de acompanhamento.

---

## UX-03 — Layout não se adapta a telas estreitas e inviabiliza a navegação

### Funcionalidade ou tela analisada

Listagens de **Veículos**, **Viagens** e **Manutenção**.

### Situação identificada

Ao reduzir a largura da janela, as listagens não reorganizam o conteúdo. A tabela é
cortada: na tela de Veículos, a coluna *Ano* deixa de ser exibida, sem indicação de
que existe informação além da borda visível e sem rolagem horizontal que permita
alcançá-la.

O controle de paginação sofre o mesmo efeito, com consequência mais séria: os
números das páginas transbordam o contêiner e ficam parcialmente inacessíveis. Em
uma listagem com 260 veículos distribuídos em 26 páginas, o usuário perde a
capacidade de navegar entre os registros.

A distinção entre os dois sintomas importa. A coluna oculta é perda de informação —
contornável por outra tela. A paginação quebrada é **perda de função**: não existe
caminho alternativo para alcançar o registro que está na página 14.

### Alteração recomendada

1. Aplicar rolagem horizontal às tabelas em larguras reduzidas, ou adotar layout
   alternativo em cartões para telas estreitas, garantindo que nenhuma coluna
   desapareça silenciosamente.
2. Corrigir o contêiner da paginação para que se reorganize em larguras menores —
   com quebra de linha, agrupamento por reticências (`1 2 3 … 26`) ou navegação
   simplificada por anterior/próxima.
3. Definir e verificar formalmente os pontos de quebra de layout para as resoluções
   efetivamente utilizadas.

### Justificativa da melhoria

A perda da paginação viola a heurística de **controle e liberdade do usuário**: o
sistema retira a capacidade de navegar pelos próprios dados em função de uma
condição — a largura da janela — sem relação com a operação. A coluna que
desaparece sem aviso compromete a **visibilidade do status do sistema**, já que a
interface não sinaliza haver conteúdo oculto.

Não se trata de preferência estética: em determinada largura, uma tarefa que o
sistema oferece deixa de ser executável.

### Benefício esperado para o usuário

Acesso a todos os registros e a todas as colunas independentemente do tamanho da
janela ou do dispositivo, e viabilização do uso em notebooks de tela reduzida ou em
janelas lado a lado — cenário comum em rotina administrativa.

---

## UX-04 — Hierarquia visual do Dashboard não corresponde à densidade da informação

### Funcionalidade ou tela analisada

**Dashboard** — cards *Total de KM Percorrido*, *Volume por Categoria* e *Cronograma
de Manutenção*.

### Situação identificada

A distribuição do espaço na tela é inversa à relevância analítica de cada
componente.

O card *Total de KM Percorrido* ocupa a largura integral da tela para apresentar um
único valor numérico e a placa do veículo. Abaixo dele, o gráfico *Volume por
Categoria* e o *Cronograma de Manutenção* — que concentram a informação
interpretável da tela — dividem o espaço restante.

Dentro do próprio card do gráfico a desproporção se repete: o donut é renderizado em
dimensão reduzida no canto superior, com a legenda ao lado, deixando a maior parte
da área do card vazia. O elemento que carrega a informação é o menor da tela, e o
espaço reservado para ele é majoritariamente não utilizado.

O efeito prático é que a leitura do Dashboard começa pelo dado menos denso. O
usuário percorre a tela na ordem inversa à utilidade.

### Alteração recomendada

1. Reduzir a área do card de valor único, convertendo-o em indicador compacto —
   formato que permitiria exibir outros indicadores relevantes na mesma faixa:
   total de veículos, viagens do período, manutenções em aberto.
2. Ampliar o gráfico *Volume por Categoria* para ocupar a área disponível do seu
   card, eliminando o espaço vazio.
3. Exibir os valores absolutos diretamente sobre os segmentos do gráfico, reduzindo
   a dependência da legenda lateral.
4. Reservar a posição superior da tela aos indicadores de ação — manutenções
   atrasadas, veículos indisponíveis — em vez de a um acumulado histórico.

### Justificativa da melhoria

Viola o princípio de **design estético e minimalista**, cujo critério não é
simplicidade visual mas proporcionalidade: cada elemento deve competir por atenção
na medida de sua relevância. Um indicador de valor único ocupando a maior área da
tela desloca a atenção do conteúdo analítico.

Compromete também o **reconhecimento em vez de memorização**: um gráfico pequeno com
legenda lateral exige associação entre cor e rótulo, enquanto valores exibidos sobre
os próprios segmentos são lidos diretamente.

### Benefício esperado para o usuário

Leitura mais rápida da situação da frota, com a informação de maior valor decisório
ocupando posição e área compatíveis com seu peso, e melhor aproveitamento da tela
para exibir indicadores hoje ausentes.

---

## UX-05 — Listagens oferecem apenas busca textual, sem filtros

### Funcionalidade ou tela analisada

Listagens de **Veículos**, **Viagens** e **Manutenção**. Na data da análise, a base
continha 260 veículos, 96 viagens e 81 manutenções.

### Situação identificada

As três telas oferecem exclusivamente busca por texto livre, com paginação fixa de
10 linhas. Não há filtro por tipo de veículo, por status de manutenção, por período
de viagem, nem ordenação por coluna.

A busca em si funciona bem: CT-012 confirmou filtragem correta por placa e por
modelo, indiferente a maiúsculas e minúsculas. O problema não é a qualidade da
busca, é o fato de ela ser o único recurso disponível.

Os critérios também são inconsistentes entre as telas: Veículos busca por placa e
modelo, Manutenção por placa, modelo e serviço, e Viagens apenas por origem e
destino. A consequência prática é que **não é possível localizar as viagens de um
veículo específico pela listagem** — o critério mais natural de consulta no módulo,
e justamente o que foi necessário durante a execução dos testes de Dashboard.

Com 260 veículos distribuídos em 26 páginas, responder a perguntas operacionais
comuns — quais veículos pesados existem, quais manutenções estão pendentes, quais
viagens ocorreram em determinado mês — exige navegação página a página.

### Alteração recomendada

1. Incluir filtros por coluna em cada listagem: tipo e ano em Veículos; status e
   período em Manutenção; veículo e período em Viagens.
2. Padronizar o escopo da busca textual para incluir a placa do veículo em todas as
   telas.
3. Habilitar ordenação por clique no cabeçalho das colunas.
4. Permitir ajuste do número de linhas por página com valores maiores que 10.

### Justificativa da melhoria

Viola a heurística de **flexibilidade e eficiência de uso**: a interface trata todo
volume de dados da mesma forma, sem oferecer mecanismos de recorte ao usuário
experiente. Compromete também **consistência e padrões**, pela divergência dos
critérios de busca entre telas equivalentes.

O impacto cresce com o uso: a limitação é tolerável em base de demonstração e
inviabiliza a operação em uma frota real.

### Benefício esperado para o usuário

Redução expressiva do tempo para localizar registros, viabilização de consultas
operacionais hoje impossíveis pela interface, e consistência de comportamento entre
telas equivalentes — o que reduz a curva de aprendizado.

---

## UX-06 — Inconsistências de texto e de idioma na interface

### Funcionalidade ou tela analisada

Menu lateral, tela de login, listagens de **Viagens** e **Manutenção**, e mensagens
de erro do cadastro de veículos.

### Situação identificada

Foram identificadas seis ocorrências de inconsistência textual, distribuídas por
telas diferentes — o que caracteriza padrão, e não erro isolado.

| Ocorrência | Onde | Tipo |
|---|---|---|
| `Veiculos` no menu lateral, enquanto o título da própria página exibe `Veículos` | Menu lateral | Acentuação, com divergência interna |
| `Ainda nao tem uma conta?` | Tela de login | Acentuação |
| `Revisao preventiva` | Manutenção e Dashboard | Acentuação |
| `Joao Pessoa` | Viagens | Acentuação, em nome próprio |
| `Teste Usuario 1` | Cabeçalho, identificação do usuário | Acentuação |
| `Veiculo with placa 'JVC-1234' already exists` | Cadastro de veículo, ao tentar duplicar placa | Mistura de idiomas e exposição de terminologia interna |

A primeira ocorrência é a mais reveladora: o mesmo termo aparece com e sem acento na
mesma tela, em elementos adjacentes. Indica que os textos são definidos ponto a
ponto, sem fonte única.

A última é de natureza distinta das demais. A validação funciona corretamente — o
sistema bloqueia a placa duplicada, conforme verificado em CT-008 —, mas a mensagem
devolvida ao usuário é a mensagem interna do sistema, em inglês, com o nome do campo
técnico exposto.

### Alteração recomendada

1. Revisar os textos estáticos da interface, corrigindo a acentuação.
2. Centralizar os rótulos em um arquivo único de textos, em vez de defini-los
   diretamente nos componentes, evitando divergência entre telas.
3. Substituir as mensagens de erro técnicas por linguagem natural orientada à ação.
   No caso citado: *"Já existe um veículo cadastrado com a placa JVC-1234."*
4. Verificar se o padrão de mensagem em inglês se repete em outras validações do
   sistema — as mensagens em português observadas em Manutenção sugerem tratamento
   inconsistente entre módulos.

### Justificativa da melhoria

A acentuação ausente viola **consistência e padrões**, especialmente no caso do menu
lateral, em que o mesmo termo é grafado de duas formas na mesma tela.

A mensagem técnica em inglês viola a heurística de **ajuda aos usuários a reconhecer,
diagnosticar e corrigir erros**, que recomenda linguagem natural em vez de códigos
internos. Viola também **correspondência entre o sistema e o mundo real**: o usuário
de um sistema em português não deve precisar interpretar terminologia de banco de
dados em outro idioma.

Este item é o de menor severidade do conjunto, mas o de menor custo de correção — e
o de maior visibilidade, por afetar telas de uso diário.

### Benefício esperado para o usuário

Percepção de acabamento e confiabilidade do produto, compreensão imediata das
mensagens de erro sem necessidade de interpretação, e eliminação da ambiguidade
causada por grafias divergentes do mesmo termo.

---

## Observações complementares

Achados registrados para completude, sem desenvolvimento individual.

| Achado | Natureza | Onde |
|---|---|---|
| Divergência de padrão numérico: quilometragem em `220.5` (ponto decimal) e valores monetários em `R$ 350,00` (vírgula decimal) | Consistência | Viagens e Manutenção |
| Nomenclatura divergente entre botão e modal: *Nova Manutenção* abre *Agendar Manutenção*; *Nova Viagem* abre *Adicionar Viagem* | Consistência e padrões | Manutenção, Viagens |
| Dashboard não informa o período de referência dos indicadores exibidos | Visibilidade do status do sistema | Dashboard |
| Seletor de veículo do Dashboard trunca o texto das opções | Legibilidade | Dashboard |
| Ícones de ação (editar e excluir) sem rótulo textual ou indicação acessível | Acessibilidade | Todas as listagens |
| Exclusão de registro executada sem diálogo de confirmação explícito sobre vínculos | Prevenção de erros | Todas as listagens |
| Cadastro público disponível em `/register` em sistema de gestão interna de frota | Controle de acesso | Tela de registro |

O último item deixa de ser questão de usabilidade à luz de **BUG-01**: com a
autenticação aceitando qualquer senha, a existência de cadastro público amplia
significativamente a superfície de acesso indevido. O tratamento recomendado está
descrito no [`04-parecer-de-qualidade.md`](04-parecer-de-qualidade.md).

---

## Nota sobre o ambiente de análise

A base apresentava, no momento da análise, registros originados de execuções
anteriores de terceiros — veículos nomeados `Teste API Ano`, `Veículo Teste Cypress`
e `Modelo QA Automatizado`, além de viagens com origem e destino preenchidos
automaticamente no padrão `Origem QGC3J32 - Destino QGC3J32`.

O fato não compromete as observações registradas neste documento, que dizem respeito
ao comportamento da interface e não ao conteúdo dos dados. Fica o registro de que
contagens absolutas não foram utilizadas como critério em nenhuma das análises,
justamente por essa razão.