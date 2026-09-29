# Estratégia de Testes Adicionais — LogiTrack Pro

## Diagnóstico que orienta a proposta

Os 12 casos reprovados neste ciclo apontam para duas causas distintas, com pesos
muito diferentes.

**A primeira é o controle de acesso.** O sistema concede sessão sem verificar a
senha (BUG-01). O defeito foi encontrado em um teste funcional simples de login —
não em avaliação de segurança dedicada. Isso é significativo para a estratégia: se
uma falha dessa gravidade estava ao alcance do primeiro caso negativo de
autenticação, a camada de segurança não foi submetida a verificação sistemática em
nenhum momento do desenvolvimento.

**A segunda é a validação de dados**, e o padrão é consistente: campos numéricos sem
limite mínimo, campos sem obrigatoriedade definida e pares de datas sem verificação
de coerência. O ponto relevante é que essas regras **já existem** em outras partes
do sistema — Manutenção valida datas e custo, e o cadastro de veículo valida placa
corretamente. A validação foi implementada tela a tela, sem camada comum.

Isso desloca a prioridade da estratégia. O problema central não é cobertura de fluxo
— os caminhos felizes funcionaram em todos os módulos — mas **confiabilidade do
controle de acesso e do dado**. Uma proposta que priorizasse carga ou
compatibilidade antes disso estaria otimizando a dimensão errada.

A ordem abaixo reflete essa leitura.

| # | Tipo de teste | Prioridade |
|---|---|---|
| 1 | Testes de segurança | Crítica |
| 2 | Testes de API | Alta |
| 3 | Testes unitários de validação | Alta |
| 4 | Testes de integração | Média |
| 5 | Testes automatizados de interface (E2E) | Média |
| 6 | Testes de acessibilidade | Média |
| 7 | Testes de compatibilidade entre navegadores e dispositivos | Baixa |

---

## 1. Testes de segurança

**Objetivo**
Verificar sistematicamente os mecanismos de autenticação, autorização e proteção de
dados da aplicação, com avaliação dedicada e não incidental.

**Parte do sistema a ser validada**
Prioritariamente o fluxo de autenticação: verificação efetiva de credenciais,
política de senhas, gestão e expiração de sessão, e comportamento diante de
tentativas repetidas de acesso. Em seguida, o controle de autorização por
endpoint, a pertinência da rota de cadastro público `/register` e a exposição de
dados operacionais a usuários não autenticados.

**Risco que ajuda a reduzir**
Acesso indevido à operação completa da frota. O defeito BUG-01 demonstra que o
risco não é hipotético: na versão avaliada, conhecer um endereço de e-mail
cadastrado é suficiente para obter permissão de leitura, alteração e exclusão em
todos os módulos. Reduz também o risco de repúdio — sem autenticação confiável,
nenhuma ação registrada pode ser atribuída com segurança a um usuário.

**Prioridade sugerida: Crítica**
É o único item desta lista que condiciona a disponibilização do sistema. Nenhuma
outra frente de qualidade tem valor prático enquanto o controle de acesso permanecer
inoperante. Recomenda-se, no mínimo, a verificação da autenticação antes de qualquer
liberação, e uma avaliação de segurança mais ampla em ciclo dedicado, com
autorização formal e ambiente segregado.

---

## 2. Testes de API

**Objetivo**
Validar contratos, códigos de resposta HTTP e tratamento de dados inválidos nos
endpoints, verificando se as regras de negócio são aplicadas no servidor e não
apenas na interface.

**Parte do sistema a ser validada**
Endpoints de criação, edição e exclusão de veículos, viagens e manutenções, com
foco nos campos que apresentaram falha de validação: quilometragem, ano e pares de
datas. Inclui a validação de autorização — verificar se os endpoints rejeitam
requisições sem token válido — e o comportamento da exclusão de registros com
vínculos.

**Risco que ajuda a reduzir**
Persistência de dados inválidos por requisições que não passam pela interface. As
validações ausentes identificadas neste ciclo podem existir apenas no front-end;
nesse caso, qualquer cliente que consuma a API contorna a regra e corrompe a base,
junto com todos os indicadores derivados dela.

Há um risco específico a verificar: o módulo Manutenção rejeitou corretamente custo
negativo e datas incoerentes pela interface, mas isso não comprova que a regra exista
no servidor. Testes de API responderiam se a validação correta observada em
Manutenção é de fato mais robusta ou apenas mais visível.

**Prioridade sugerida: Alta**
Ataca diretamente a causa raiz da maior parte dos defeitos encontrados, tem custo de
implementação baixo, executa em segundos e não depende de interface estável — o que
o torna o primeiro candidato a integrar o pipeline.

---

## 3. Testes unitários de validação

**Objetivo**
Verificar isoladamente as funções que implementam as regras de validação de cada
campo, garantindo comportamento correto nos limites e nos valores inválidos.

**Parte do sistema a ser validada**
Funções de validação de quilometragem (mínimo maior que zero), ano de fabricação
(intervalo plausível e obrigatoriedade), custo estimado (não negativo), formato de
placa e coerência entre pares de datas.

Recomenda-se que essas funções sejam extraídas para uma **camada comum de
validação**, compartilhada entre os formulários. O diagnóstico deste ciclo indica
que a duplicação de regras por tela é a origem das divergências encontradas: a mesma
verificação de datas existe em Manutenção e falta em Viagens.

**Risco que ajuda a reduzir**
Reintrodução de regras incorretas a cada alteração de código, e divergência entre
módulos que deveriam aplicar a mesma regra. Uma regra validada unitariamente
permanece testada mesmo quando o formulário que a utiliza é redesenhado. Reduz
também o custo de detecção: um defeito encontrado em teste unitário é ordens de
grandeza mais barato de corrigir do que o mesmo defeito em produção.

**Prioridade sugerida: Alta**
Não consta da lista sugerida no desafio, mas é a contramedida mais direta para o
padrão de defeitos observado. Executa em milissegundos, é o nível mais barato da
pirâmide de testes e sustenta os demais níveis.

---

## 4. Testes de integração

**Objetivo**
Validar a consistência dos dados ao atravessar módulos, verificando se as
agregações apresentadas correspondem aos registros de origem e se as operações em um
módulo produzem o efeito esperado nos demais.

**Parte do sistema a ser validada**
A cadeia que liga Viagens e Manutenção aos indicadores do Dashboard: soma de
quilometragem por veículo, contagem de viagens por categoria e composição do
cronograma. Inclui também o efeito da exclusão de um veículo sobre seus registros
dependentes, e a disponibilidade de veículo para viagem conforme o status de
manutenção.

**Risco que ajuda a reduzir**
Divergência entre o dado registrado e o dado apresentado, e comportamento
inconsistente entre módulos. Os defeitos BUG-07, BUG-08 e a observação sobre
exclusão de veículos (CT-011) são todos dessa natureza: cada módulo funciona
isoladamente, mas a composição entre eles produz resultado incorreto ou
operacionalmente impossível. É um risco que testes unitários não capturam por
definição.

**Prioridade sugerida: Média**
O Dashboard é a tela de leitura gerencial do sistema, e indicadores incorretos
comprometem a confiança em toda a aplicação. Fica abaixo dos itens anteriores apenas
porque parte dos defeitos de indicador deixa de existir quando a validação de
entrada é corrigida.

---

## 5. Testes automatizados de interface (end-to-end)

**Objetivo**
Automatizar a execução dos fluxos principais da aplicação, garantindo que permaneçam
funcionais após alterações.

**Parte do sistema a ser validada**
Autenticação — incluindo o caso negativo de senha incorreta, que deve permanecer
coberto permanentemente após a correção de BUG-01 — e os fluxos completos de criação,
edição e exclusão nos três módulos de cadastro.

Prova de conceito sugerida com Playwright ou Cypress, cobrindo login válido, login
com senha incorreta, cadastro com dados válidos e cadastro com dado inválido
verificando o bloqueio.

**Risco que ajuda a reduzir**
Regressão em funcionalidades estáveis. Os CRUDs são repetitivos e de comportamento
previsível — exatamente o perfil que torna a reexecução manual custosa e a automação
vantajosa. A suíte manual deste ciclo levou dois dias para 24 casos; automatizada,
executaria em minutos a cada alteração.

Há um argumento adicional específico deste sistema: o parecer recomenda reexecução
integral da suíte após as correções, não apenas dos casos reprovados. Automação
reduz diretamente o custo dessa reexecução.

**Prioridade sugerida: Média**
Alto valor de longo prazo, com custo inicial de implementação e manutenção maior que
o dos níveis anteriores. Recomenda-se implementar após a estabilização das
validações, para evitar automatizar comportamento que ainda será corrigido.

---

## 6. Testes de acessibilidade

**Objetivo**
Identificar barreiras que dificultem ou impeçam o uso do sistema por pessoas com
deficiência, verificando conformidade com as diretrizes WCAG 2.1 nível AA.

**Parte do sistema a ser validada**
Contraste de cores nos badges de status e textos auxiliares, navegação completa por
teclado nos formulários modais, associação de rótulos aos campos, textos
alternativos nos ícones de ação — hoje representados apenas por ícone de lápis e
lixeira — e anúncio de mensagens de erro por leitores de tela.

**Risco que ajuda a reduzir**
Exclusão de usuários com deficiência visual ou motora e exposição a não
conformidade legal — a Lei Brasileira de Inclusão estabelece requisitos de
acessibilidade digital. Os ícones de ação sem rótulo textual e a dependência de cor
para comunicar status são os pontos de atenção mais evidentes na aplicação atual.

**Prioridade sugerida: Média**
Custo de avaliação inicial baixo, com ferramentas automatizadas cobrindo parte
relevante dos critérios. A prioridade sobe para Alta caso a aplicação venha a ser
usada por equipe ampla ou esteja sujeita a exigência contratual de acessibilidade.

---

## 7. Testes de compatibilidade entre navegadores e dispositivos

**Objetivo**
Verificar o comportamento consistente da aplicação nos navegadores e resoluções
efetivamente utilizados pelos usuários.

**Parte do sistema a ser validada**
Renderização das tabelas com rolagem horizontal, comportamento dos modais em telas
pequenas, funcionamento dos campos de data — que dependem de controle nativo do
navegador e variam entre implementações — e legibilidade do Dashboard em resolução
reduzida.

**Risco que ajuda a reduzir**
Indisponibilidade funcional para parte dos usuários. O risco se concentra nos campos
de data, cuja interface e formato de entrada diferem entre navegadores, e que já são
origem de defeitos de validação neste sistema.

**Prioridade sugerida: Baixa**
Sistema de uso interno, com parque tecnológico previsivelmente controlado. A
prioridade sobe caso haja uso em campo por motoristas em dispositivos móveis —
cenário plausível para o domínio, mas não evidenciado na aplicação atual.

---

## Tipo de teste fora do escopo desta proposta

| Tipo | Motivo |
|---|---|
| Testes de carga e desempenho | A aplicação não apresentou degradação perceptível no volume atual (249 veículos, 96 viagens, 81 manutenções), e não há informação sobre volumetria esperada em produção. Recomenda-se reavaliar quando houver projeção de uso, ou caso o sistema passe a receber dados telemétricos de frota, que alterariam significativamente o perfil de carga. |

---

## Recomendações de processo

Complementarmente aos tipos de teste, quatro ajustes de processo teriam prevenido
boa parte dos defeitos identificados.

**Critérios de aceite explícitos por funcionalidade.**
A ausência de especificação obrigou a inferir as regras de negócio a partir da
interface — e duas regras deste ciclo (BUG-06 e BUG-07) permanecem registradas como
premissas, por não constarem de nenhuma fonte formal. Definir, antes do
desenvolvimento, quais valores cada campo aceita e rejeita transforma validação em
requisito verificável, em vez de interpretação posterior.

**Camada comum de validação.**
O diagnóstico central deste ciclo é a duplicação de regras por tela. Extrair as
validações para um módulo compartilhado elimina a classe inteira de defeitos em que
a mesma regra existe em um formulário e falta em outro.

**Validação dupla, em interface e servidor.**
Adotar como padrão de projeto que toda regra de negócio seja aplicada nas duas
camadas, tratando a validação de interface como conveniência de usabilidade e a de
servidor como garantia de integridade.

**Testes integrados ao pipeline.**
Executar unitários e de API a cada commit, com bloqueio de merge em caso de falha,
desloca a detecção para o ponto mais barato do ciclo e impede a reintrodução de
defeitos já corrigidos. No caso de BUG-01, um único teste automatizado de login com
senha incorreta teria impedido que o defeito chegasse à fase de validação.