# Especificação de Produto — Weather App

## Overview

### Objetivo

Oferecer uma aplicação web simples para consultar o clima atual e a previsão de cinco dias de uma cidade, facilitando decisões cotidianas e o planejamento da semana. A interface da primeira versão será em português do Brasil.

### Público e necessidade

As personas deste documento são hipóteses de discovery, ainda não validadas por pesquisa: pessoa que planeja a semana, pessoa que decide o que fazer no dia e pessoa viajando. Todas precisam localizar uma cidade e interpretar informações meteorológicas sem confundir local, data ou unidade.

### Escopo da primeira versão

- Pesquisar e selecionar cidades.
- Consultar condições meteorológicas atuais e uma previsão de cinco dias.
- Alternar as temperaturas entre Celsius e Fahrenheit; iniciar em Celsius.
- Usar Open-Meteo como fonte de geocodificação e dados meteorológicos, sem exigir API key.
- Acessar sem conta e sem persistência de dados no servidor.
- Usar a interface em pt-BR e em dispositivos móveis.

As decisões acima foram fechadas no discovery. As regras e metas abaixo são o baseline da V1 para desenvolvimento. As verificações restantes em **Open Questions** são gates de lançamento, não bloqueios de implementação.

### Indicador de usabilidade para lançamento

Em teste moderado com 10 pessoas que não conhecem a interface, pelo menos 8 devem conseguir, sem ajuda, pesquisar e selecionar a cidade correta, consultar a previsão de cinco dias e alternar a unidade. Registrar conclusão por tarefa e erros de seleção; não coletar esses dados por telemetria na V1. A meta é um gate de lançamento.

## Functional Requirements

- **FR1 — Pesquisar cidades:** aceitar uma consulta de 1 a 100 caracteres Unicode após remover espaços nas extremidades; preservar acentos, espaços, apóstrofos e hífens e codificar o termo na requisição. Apresentar cidade e, quando fornecidos, região/estado e país.
- **FR2 — Selecionar cidade:** permitir selecionar um resultado e usar suas coordenadas em todas as consultas meteorológicas subsequentes. Para resultados ambíguos, exigir escolha explícita; respostas de buscas anteriores não podem substituir a seleção mais recente.
- **FR3 — Exibir clima atual:** mostrar temperatura, unidade, condição meteorológica em pt-BR e horário local disponibilizado pela fonte para a cidade selecionada. Identificar os dados como indisponíveis se a fonte não retornar os campos mínimos.
- **FR4 — Exibir previsão de cinco dias:** mostrar cinco datas locais consecutivas, de hoje até hoje + 4 dias, no fuso horário da cidade. Para cada data, mostrar condição em pt-BR e temperaturas mínima e máxima. Manter a data visível mesmo quando algum dado estiver ausente e marcar esse dado como indisponível.
- **FR5 — Alternar unidade de temperatura:** permitir alternar entre Celsius e Fahrenheit, aplicando a escolha às temperaturas atuais e a todas as temperaturas da previsão. Usar Celsius ao abrir a aplicação; a escolha vale até a página ser recarregada ou encerrada e não é persistida.
- **FR6 — Comunicar estados da consulta:** indicar carregamento, busca sem resultados, erro de serviço/rede e timeout. Para erro recuperável de consulta meteorológica, oferecer nova tentativa para a cidade selecionada. Busca vazia ou inválida deve ser tratada localmente, sem chamada à API.
- **FR7 — Acessar sem conta:** permitir os fluxos de busca e consulta sem cadastro, login, perfil ou armazenamento de histórico/preferência em servidor próprio.

## User Stories

- **US1** — Como Planejadora da semana, quero pesquisar e selecionar uma cidade para consultar a previsão do local correto. (FR1, FR2)
- **US2** — Como Decisor do dia a dia, quero ver a temperatura e a condição atuais da cidade selecionada para decidir o que vestir ou como me deslocar. (FR3)
- **US3** — Como Planejadora da semana, quero consultar os cinco períodos diários da previsão para organizar compromissos e atividades. (FR4)
- **US4** — Como Decisor do dia a dia, quero alternar entre Celsius e Fahrenheit para interpretar as temperaturas na unidade que conheço. (FR5)
- **US5** — Como Pessoa viajando, quero distinguir cidades com nomes semelhantes pelos dados de localização para consultar o clima do destino correto. (FR1, FR2)
- **US6** — Como Pessoa viajando, quero saber quando uma busca não encontra resultados ou uma consulta falha e poder tentar novamente para continuar meu planejamento. (FR6)
- **US7** — Como Decisor do dia a dia, quero consultar o clima sem criar uma conta para obter a informação rapidamente. (FR7)

## Rastreabilidade

| User Story | Acceptance Criteria | Requisitos não-funcionais relevantes |
| --- | --- | --- |
| **US1 — Pesquisar e selecionar cidade** | AC1, AC2, AC4, AC6, AC8, AC11 | NFR1, NFR2, NFR3, NFR4, NFR5, NFR6, NFR7, NFR8, NFR9 |
| **US2 — Consultar clima atual** | AC5, AC8, AC9 | NFR1, NFR2, NFR3, NFR4, NFR5, NFR6, NFR7, NFR9 |
| **US3 — Consultar previsão de cinco dias** | AC6, AC8, AC9 | NFR1, NFR2, NFR3, NFR4, NFR5, NFR6, NFR7, NFR9 |
| **US4 — Alternar unidade** | AC7, AC10 | NFR1, NFR2, NFR3, NFR7, NFR8 |
| **US5 — Distinguir cidades homônimas** | AC2, AC4, AC11 | NFR1, NFR2, NFR3, NFR4, NFR5, NFR7, NFR8, NFR9 |
| **US6 — Recuperar-se de busca vazia ou falha** | AC1, AC3, AC8, AC9 | NFR2, NFR3, NFR4, NFR5, NFR6, NFR8, NFR9 |
| **US7 — Usar sem criar conta** | AC10 | NFR8 |

## Acceptance Criteria

Os cenários usam respostas controladas para validar a interface e a integração. Valores exibidos devem corresponder à resposta; dados ausentes nunca são inventados.

### AC1 — Input inválido (FR1, FR6)

- **Given** que o campo de busca está vazio ou contém apenas espaços
- **When** a pessoa envia o formulário
- **Then** a aplicação não chama o serviço, mantém o foco no campo e apresenta uma instrução de correção acessível.

### AC2 — Busca válida (FR1)

- **Given** que o campo contém de 1 a 100 caracteres após trim
- **When** a pessoa envia a busca
- **Then** a aplicação envia o termo corretamente codificado e apresenta os resultados selecionáveis com cidade e os campos de região/estado e país que a fonte retornar.
- **Given** que o termo inclui acentos, apóstrofos ou hífens válidos
- **When** a busca é enviada
- **Then** o termo é preservado e a requisição é válida.

### AC3 — Busca sem resultados (FR1, FR6)

- **Given** que o serviço de geocodificação retorna uma lista vazia
- **When** a busca termina
- **Then** a aplicação informa que não encontrou a cidade, mantém o termo editável e não solicita dados meteorológicos.

### AC4 — Seleção e desambiguação (FR2)

- **Given** que a busca retorna um ou mais locais
- **When** a pessoa seleciona um resultado
- **Then** esse local é identificado como ativo e suas coordenadas são usadas para clima atual e previsão.
- **Given** que duas cidades têm o mesmo nome
- **When** os resultados são exibidos
- **Then** cada opção mostra os dados geográficos disponíveis e nenhuma é selecionada automaticamente.

### AC5 — Clima atual (FR3)

- **Given** que a fonte retorna temperatura, condição e horário para a cidade ativa
- **When** o carregamento termina
- **Then** a interface mostra esses valores, unidade, descrição em pt-BR e horário local.

### AC6 — Previsão diária (FR4)

- **Given** que a fonte fornece uma previsão para a cidade e seu fuso horário
- **When** a previsão é renderizada
- **Then** são mostradas cinco datas locais consecutivas, de hoje até hoje + 4 dias, cada uma com condição e mínima/máxima rotuladas.
- **Given** que faltam campos ou dias na resposta
- **When** a previsão é renderizada
- **Then** as cinco datas permanecem identificadas, campos indisponíveis são marcados como tal e a previsão é indicada como incompleta; nenhum valor é estimado pela aplicação.

### AC7 — Unidade e arredondamento (FR5)

- **Given** que a aplicação acabou de ser aberta
- **When** os dados são exibidos
- **Then** todas as temperaturas usam Celsius.
- **Given** que a pessoa seleciona Fahrenheit
- **When** o clima atual e a previsão estão visíveis
- **Then** todas as temperaturas são convertidas pela fórmula F = C × 9/5 + 32 e exibidas com uma casa decimal, arredondada ao décimo mais próximo; em empate, arredondar afastando de zero. Os rótulos também são atualizados.
- **Given** que o valor de origem é 0 °C ou 32 °F
- **When** a pessoa alterna a unidade
- **Then** o valor equivalente é 32 °F ou 0 °C, respectivamente.
- **Given** que a pessoa recarrega a página
- **When** os dados são carregados novamente
- **Then** a unidade volta para Celsius.

### AC8 — Carregamento acessível (FR6)

- **Given** que uma busca ou consulta está pendente
- **When** a aplicação aguarda resposta
- **Then** mostra um estado de carregamento que não depende apenas de cor e é anunciado por tecnologia assistiva; o formulário não inicia duplicatas por submissão repetida enquanto a mesma requisição está pendente.

### AC9 — Erro, timeout e recuperação (FR6)

- **Given** que uma requisição falha ou não responde em até 10 segundos
- **When** o erro ou timeout ocorre
- **Then** o carregamento termina, uma mensagem acessível explica que os dados não estão disponíveis e uma ação de nova tentativa é apresentada; nenhum dado ausente é exibido como atual.
- **Given** que a pessoa aciona nova tentativa
- **When** a consulta termina com sucesso
- **Then** os dados da cidade ativa são apresentados e a mensagem de erro é removida.

### AC10 — Acesso sem conta e sem persistência própria (FR7)

- **Given** que uma pessoa abre uma sessão nova
- **When** pesquisa e consulta uma cidade
- **Then** consegue completar o fluxo sem cadastro/login e a aplicação não grava conta, histórico ou preferência em servidor próprio.

### AC11 — Concorrência de buscas (FR1, FR2)

- **Given** que a pessoa inicia uma segunda busca antes da primeira terminar
- **When** as respostas chegam fora de ordem
- **Then** apenas resultados e dados associados à busca/seleção mais recente são exibidos.

## Non-Functional Requirements

Os limites abaixo são os baselines adotados para a V1 e devem ser usados nos testes de implementação. Mudanças exigem decisão explícita de escopo; o cumprimento será verificado antes do lançamento.

- **NFR1 — Responsividade:** suportar viewport de 320 CSS px ou maior, sem rolagem horizontal na área principal. Validar busca, consulta e troca de unidade em 320, 390, 768 e 1280 CSS px.
- **NFR2 — Compatibilidade:** suportar as duas versões estáveis mais recentes de Chrome, Edge, Firefox e Safari, incluindo Safari em iOS e Chrome em Android. Validar os fluxos principais em cada motor suportado.
- **NFR3 — Acessibilidade:** atender WCAG 2.2 AA nos fluxos principais. Combinar verificação automatizada com axe-core e verificação manual de teclado, foco visível e leitor de tela; automação isolada não conta como comprovação de conformidade.
- **NFR4 — Desempenho:** em 30 execuções sintéticas de staging com viewport 390 × 844 CSS px, rede configurada para 10 Mbps down/2 Mbps up e RTT de 100 ms, pelo menos 95% devem mostrar clima atual e previsão até 3 segundos após selecionar a cidade. Usar resposta controlada do provedor em até 500 ms; medir dependência real separadamente.
- **NFR5 — Timeout e resiliência:** cancelar cada requisição à fonte, incluindo geocodificação e consultas meteorológicas, após 10 segundos; finalizar loading, anunciar erro acessível e permitir retry manual quando aplicável. Não fazer retry automático nem apresentar dados armazenados como atuais; a V1 não oferece modo offline/cache persistente.
- **NFR6 — Disponibilidade:** meta de 99,5% mensal para carregamento e interatividade do frontend, verificada por probes sintéticos a cada minuto. Medir a disponibilidade do Open-Meteo separadamente e reportar indisponibilidade da dependência sem atribuí-la ao SLO do frontend.
- **NFR7 — Idioma e datas:** interface, mensagens e condições meteorológicas em pt-BR; formatar números e datas segundo pt-BR. Interpretar e exibir datas da previsão no fuso retornado para a cidade selecionada.
- **NFR8 — Privacidade:** não criar conta, perfil, histórico ou preferência persistida em servidor próprio; manter a unidade apenas em memória durante a página aberta. Informar que termo de busca/coordenadas necessários são enviados ao Open-Meteo. Não incluir analytics ou cookies de rastreamento na V1.
- **NFR9 — Segurança e integridade:** usar HTTPS para aplicação e chamadas ao provedor; codificar input como parâmetro de URL e nunca interpretá-lo como HTML. Associar cada resposta à cidade selecionada e não substituir dados de uma seleção mais recente por resposta atrasada.

## Edge Cases

| Cenário | Comportamento esperado |
| --- | --- |
| Cidade inexistente | Se a busca não encontrar correspondência para a cidade informada, mostrar uma mensagem de que a cidade não foi encontrada, manter o campo disponível para correção e não solicitar previsão para uma cidade diferente. |
| Input vazio ou só com espaços | Ao enviar um campo vazio após trim, não chamar a API; manter o foco no campo e apresentar instrução de correção. |
| Caracteres especiais no nome | Preservar acentos, espaços, apóstrofos e hífens e codificar o termo como parâmetro. Rejeitar localmente termos acima de 100 caracteres Unicode, com mensagem de limite. |
| Falha de API | Em resposta de erro ou falha de rede, encerrar o carregamento, informar que os dados não puderam ser obtidos e oferecer nova tentativa. Não apresentar valores ausentes ou antigos como condições atuais. |
| Timeout | Após 10 segundos sem resposta, cancelar a requisição, terminar loading, informar indisponibilidade temporária e oferecer retry manual. Ignorar qualquer resposta tardia após cancelamento ou nova seleção. |
| Geocoding sem resultados | Tratar como cidade não encontrada: informar, manter o termo editável e não iniciar consulta meteorológica. Erro/timeout do próprio serviço de geocodificação deve usar o estado de falha de API. |
| Resposta parcial | Manter as cinco datas da previsão; marcar cada campo ausente como indisponível e indicar que a previsão está incompleta. Não inventar valores, copiar valores de outro dia ou substituir a cidade selecionada. |
| Cidades homônimas ou identificação incompleta | Apresentar os dados de região/estado e país quando disponíveis; exigir seleção explícita se houver mais de uma opção e não escolher silenciosamente um resultado ambíguo. |
| Cidade sem dados atuais ou previsão | Mostrar a seção disponível e indicar indisponibilidade na seção sem dados, sem substituir a cidade selecionada ou exibir valores de outro local. |
| Mudança de dia ou fuso horário diferente | Calcular os cinco períodos pela data local e timezone retornados para a cidade, inclusive em virada de mês ou ano. |
| Unidade alterada durante carregamento | Aplicar a unidade selecionada aos dados existentes e recebidos depois; recarregar a página restaura Celsius. |
| Conversão com arredondamento | Converter com F = C × 9/5 + 32 e exibir uma casa decimal; em empate no arredondamento, afastar de zero em todas as telas. |
| Busca nova antes da anterior terminar | Exibir somente a resposta ligada à busca/seleção mais recente; descartar respostas atrasadas. |
| Viewport mínimo ou navegação assistiva | Manter os fluxos principais utilizáveis a partir de 320 CSS px e operáveis por teclado e leitor de tela. |

## Assumptions

- A V1 é uma aplicação web responsiva; aplicativos nativos e geolocalização automática não fazem parte do escopo.
- Open-Meteo fornece geocodificação e dados meteorológicos sem API key; cobertura, limites e termos de uso devem ser confirmados antes do lançamento.
- Cinco dias significa hoje e os quatro dias seguintes no timezone da cidade; a previsão V1 é diária com condição e mínima/máxima.
- Celsius é o padrão. A escolha de unidade dura somente enquanto a página está aberta; recarga restaura Celsius.
- Não há autenticação, backend próprio, persistência local, analytics, cookies de rastreamento, cache offline, favoritos ou histórico na V1.
- A descrição de condição será localizada em pt-BR a partir dos códigos retornados pela fonte.
- Os valores de viewport, compatibilidade, desempenho, timeout, acessibilidade e disponibilidade definidos em NFRs são os alvos de aceite da V1.

## Risks

| Risco | Probabilidade | Impacto | Mitigação |
| --- | --- | --- | --- |
| Open-Meteo indisponível, lento ou sujeito a limites | Média | Consultas falham, atrasam ou ficam limitadas | Validar limites, cobertura e disponibilidade; tratar falhas, monitorar dependência e acordar estratégia de cache/contingência. |
| Dados desatualizados ou sem cobertura | Média | Usuário toma decisões com informação incompleta ou indisponível | Verificar cobertura e atualização; informar cidade e contexto temporal dos dados quando a fonte fornecer essa informação. |
| Cidade homônima ou seleção geográfica incorreta | Média | Previsão de local errado e perda de confiança | Apresentar contexto geográfico nos resultados e tornar a seleção explícita. |
| Datas, fuso ou definição de cinco dias inconsistentes | Média | Período exibido diverge da expectativa do usuário | Usar a data local da cidade e testar mudança de dia, limites de mês/ano e fusos distintos. |
| Conversão de unidade inconsistente | Baixa | Temperatura incorreta ou confusa | Centralizar conversão, rotular unidades e validar com valores de referência. |
| Uso móvel ou acessibilidade inadequados | Média | Parte dos usuários não consegue concluir o fluxo | Definir critérios de aceite, testar viewports representativos, teclado e tecnologia assistiva. |
| Termos, cobertura ou limites do provedor incompatíveis com o lançamento | Média | Necessidade de trocar fonte ou limitar localidades após implementação | Validar termos, cobertura e limites do Open-Meteo para o público-alvo antes do lançamento. |
| Consultas a provedor externo sem comunicação clara de privacidade | Baixa/média | Perda de confiança e possíveis obrigações de conformidade | Minimizar dados enviados, informar o uso do provedor e validar requisitos de privacidade aplicáveis. |

## Out of Scope

Itens explicitamente fora da V1; inclusão posterior exige revisão de escopo:

- Aplicativos nativos para iOS ou Android.
- Login, contas, perfis ou sincronização entre dispositivos.
- Persistência local ou em servidor de histórico, favoritos ou preferência de unidade; a unidade volta a Celsius ao recarregar a página.
- Geolocalização automática.
- Funcionamento offline, cache de dados meteorológicos ou uso de dados antigos durante falha.
- Autocomplete e sugestões de busca antes do envio do formulário; a busca é iniciada explicitamente pela pessoa.
- Exibição de vento, precipitação, pressão, umidade, índice UV, visibilidade ou outras métricas além dos campos definidos nos FR3 e FR4.
- Previsões horárias, previsões além dos cinco dias definidos ou histórico meteorológico.
- Comparação simultânea de várias cidades.
- Alertas, notificações, mapas, radar meteorológico ou histórico climático.
- Interface em idiomas além de pt-BR.
- Recomendações automatizadas de vestuário, saúde, viagem ou decisão baseadas na previsão.
- Analytics, publicidade e cookies de rastreamento.

## Open Questions

Não há perguntas de produto que bloqueiem o início da implementação com o baseline definido nesta spec. Antes do lançamento, concluir estes gates:

1. Validar cobertura na geografia de lançamento, limites de uso e termos do Open-Meteo, inclusive adequação ao uso comercial.
2. Aprovar com as áreas responsáveis o aviso de privacidade sobre o envio de termo de busca/coordenadas ao provedor.
3. Executar o teste moderado definido no Overview e atingir a meta de pelo menos 8 de 10 participantes concluindo os fluxos sem ajuda.