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

As decisões acima foram fechadas no discovery. Os detalhes ainda não decididos estão listados em **Open Questions** e não devem ser tratados como compromissos aprovados.

### Indicadores de sucesso propostos

- Uma pessoa consegue encontrar e selecionar a cidade pretendida sem confundir homônimos.
- Uma pessoa consegue consultar o clima atual e as cinco datas da previsão para a cidade selecionada.
- Uma pessoa consegue identificar a unidade de temperatura e alternar entre Celsius e Fahrenheit.
- Os principais fluxos permanecem utilizáveis em uma tela móvel estreita.

Os indicadores e tempos-alvo precisam ser validados com usuários e stakeholders antes de virarem metas de lançamento.

## Functional Requirements

- **FR1 — Pesquisar cidades:** o usuário deve poder informar o nome de uma cidade e iniciar uma busca. Os resultados devem permitir identificar o local; quando disponíveis, devem apresentar cidade, região/estado e país.
- **FR2 — Selecionar cidade:** o usuário deve poder escolher um resultado de busca. A seleção deve definir a cidade usada nas consultas de clima atual e previsão.
- **FR3 — Exibir clima atual:** para a cidade selecionada, o produto deve exibir pelo menos a temperatura atual e uma descrição da condição meteorológica, com a unidade indicada.
- **FR4 — Exibir previsão de cinco dias:** para a cidade selecionada, o produto deve exibir cinco períodos diários correspondentes à data local de hoje e aos quatro dias seguintes. Como hipótese de escopo para esta versão, cada período apresenta data, condição e temperaturas mínima e máxima; esse conjunto de campos deve ser confirmado.
- **FR5 — Alternar unidade de temperatura:** o usuário deve poder alternar entre Celsius e Fahrenheit. A unidade escolhida deve ser aplicada à temperatura atual e a todas as temperaturas da previsão visível.
- **FR6 — Comunicar estados da consulta:** durante busca e carregamento de dados, o produto deve comunicar que a operação está em andamento. Deve também informar busca sem resultados e falha de consulta, oferecendo nova tentativa quando aplicável.
- **FR7 — Acessar sem conta:** o usuário deve conseguir usar as consultas principais sem criar conta ou autenticar-se.

## User Stories

- Como Planejadora da semana, quero pesquisar e selecionar uma cidade para consultar a previsão do local correto. (FR1, FR2)
- Como Decisor do dia a dia, quero ver a temperatura e a condição atuais da cidade selecionada para decidir o que vestir ou como me deslocar. (FR3)
- Como Planejadora da semana, quero consultar os cinco períodos diários da previsão para organizar compromissos e atividades. (FR4)
- Como Decisor do dia a dia, quero alternar entre Celsius e Fahrenheit para interpretar as temperaturas na unidade que conheço. (FR5)
- Como Pessoa viajando, quero distinguir cidades com nomes semelhantes pelos dados de localização para consultar o clima do destino correto. (FR1, FR2)
- Como Pessoa viajando, quero saber quando uma busca não encontra resultados ou uma consulta falha e poder tentar novamente para continuar meu planejamento. (FR6)
- Como Decisor do dia a dia, quero consultar o clima sem criar uma conta para obter a informação rapidamente. (FR7)

## Acceptance Criteria

Os critérios abaixo são verificáveis por testes de interface ou integração com respostas controladas da fonte. Os dados exibidos devem corresponder às respostas recebidas, sem inventar valores.

### AC1 — Busca com resultados (FR1)

- **Given** que o serviço de busca retorna uma ou mais cidades para o termo informado
- **When** a pessoa envia a busca
- **Then** a interface apresenta os resultados como opções selecionáveis e mostra os campos de localização fornecidos pela fonte; região/estado e país são exibidos quando disponíveis.

### AC2 — Seleção da cidade (FR2)

- **Given** que a interface apresenta mais de um resultado
- **When** a pessoa seleciona um resultado específico
- **Then** o nome desse local é identificado como cidade ativa e as consultas de clima atual e previsão usam as coordenadas desse resultado.

### AC3 — Clima atual (FR3)

- **Given** que a fonte retorna temperatura e condição atual para a cidade ativa
- **When** o carregamento dos dados termina
- **Then** a interface apresenta a temperatura, sua unidade e a descrição da condição recebida para essa cidade.

### AC4 — Previsão de cinco dias (FR4)

- **Given** que a fonte retorna dados diários para a cidade ativa e seu fuso horário
- **When** a previsão é apresentada
- **Then** existem exatamente cinco períodos, com datas locais consecutivas de hoje até hoje + 4 dias, e cada período exibe condição, temperaturas mínima e máxima e unidades identificáveis.

### AC5 — Unidade padrão e alternância (FR5)

- **Given** que a pessoa abre uma consulta sem ter selecionado outra unidade
- **When** clima atual e previsão são exibidos
- **Then** todas as temperaturas aparecem em Celsius.
- **Given** que há temperaturas atuais e previstas exibidas em Celsius
- **When** a pessoa seleciona Fahrenheit
- **Then** todas as temperaturas visíveis são convertidas para Fahrenheit e todos os rótulos correspondentes são atualizados.
- **Given** que a fonte fornece uma temperatura de 0 °C
- **When** a pessoa seleciona Fahrenheit
- **Then** o valor correspondente exibido é 32 °F.

### AC6 — Estado de carregamento (FR6)

- **Given** que uma busca ou consulta foi iniciada e a resposta ainda não chegou
- **When** a operação permanece pendente
- **Then** a interface apresenta um estado de carregamento perceptível e acessível.

### AC7 — Busca sem resultados (FR1, FR6)

- **Given** que o serviço de busca retorna uma lista vazia
- **When** a busca termina
- **Then** a interface informa que não encontrou resultados e permite iniciar outra busca.

### AC8 — Falha e nova tentativa (FR6)

- **Given** que a consulta de clima atual ou previsão retorna erro ou falha de rede
- **When** a falha é recebida pela aplicação
- **Then** a interface informa que os dados não puderam ser carregados, não apresenta dados inválidos como atuais e oferece uma ação de nova tentativa.
- **Given** que a interface exibe a ação de nova tentativa após uma falha
- **When** a pessoa a aciona
- **Then** uma nova consulta é iniciada para a mesma cidade selecionada.

### AC9 — Uso sem conta (FR7)

- **Given** que uma pessoa abre a aplicação em uma sessão nova
- **When** pesquisa e seleciona uma cidade
- **Then** consegue consultar o clima atual e a previsão sem cadastro, login ou criação de conta.

### AC10 — Ausência de persistência no servidor (NFR7)

- **Given** que uma pessoa realiza buscas e altera a unidade durante o uso
- **When** as consultas são concluídas
- **Then** nenhum perfil, histórico de buscas ou preferência é gravado em servidor próprio do produto; as requisições necessárias ao Open-Meteo não contam como persistência do produto.

## Non-Functional Requirements

Os limites numéricos identificados como **propostos** são metas iniciais para discussão. Devem ser aprovados e ter ambiente de medição definido antes de serem usados como critérios contratuais de lançamento.

- **NFR1 — Responsividade:** os fluxos de busca, leitura do clima e alternância de unidade devem funcionar em larguras de viewport a partir de 320 CSS px, sem rolagem horizontal na área principal. A largura mínima é uma meta proposta.
- **NFR2 — Acessibilidade:** propor conformidade WCAG 2.2 nível AA para as telas e fluxos principais, incluindo operação por teclado, foco visível, nomes acessíveis para controles e leitura por tecnologia assistiva. O nível final requer aprovação.
- **NFR3 — Desempenho:** propor que, no perfil de dispositivo/rede a ser acordado e com a fonte disponível, pelo menos 95% das consultas concluídas apresentem clima atual e previsão em até 3 segundos após a seleção da cidade. O perfil de medição e a meta precisam ser confirmados.
- **NFR4 — Disponibilidade:** definir e monitorar a disponibilidade da aplicação separadamente da disponibilidade do Open-Meteo. Meta inicial proposta para a aplicação: 99,5% mensal; escopo, janela de medição e exclusões precisam ser aprovados.
- **NFR5 — Resiliência:** falhas de rede, ausência de resultados ou indisponibilidade da fonte não devem deixar a interface em carregamento indefinido nem ser apresentadas como dados atuais válidos. O produto deve comunicar o estado e oferecer recuperação quando possível.
- **NFR6 — Idioma e formatos:** textos da interface devem estar em pt-BR. Datas e números devem ser apresentados em formato compatível com pt-BR; convenções exatas de data, horário e fuso ainda precisam ser acordadas.
- **NFR7 — Privacidade e dados:** a primeira versão não deve exigir autenticação nem persistir dados do usuário em servidor próprio. Consultas devem enviar ao provedor externo apenas os dados necessários à busca e à obtenção da previsão.
- **NFR8 — Integridade de informação:** cada temperatura exibida deve trazer ou estar claramente associada à sua unidade; datas da previsão devem corresponder ao local selecionado e ao período especificado.

## Edge Cases

- O termo pesquisado não corresponde a nenhuma cidade.
- A busca retorna várias cidades com o mesmo nome ou sem informações suficientes para distingui-las.
- A fonte não fornece região/estado, país ou coordenadas para um resultado.
- A cidade encontrada não tem dados atuais ou previsão disponíveis.
- A fonte retorna dados incompletos, malformados ou com datas faltando.
- O dia local muda durante uma consulta, ou a cidade está em outro fuso horário; a previsão deve continuar cobrindo a data local de hoje e os quatro dias seguintes.
- A conexão cai durante a busca ou durante o carregamento da previsão.
- A fonte está indisponível, responde com erro ou limita temporariamente as consultas.
- O usuário alterna unidade enquanto uma consulta está carregando; dados novos e já exibidos devem usar a mesma unidade selecionada.
- O resultado convertido exige arredondamento ou pode produzir pequenas diferenças de precisão; o rótulo da unidade não pode divergir do valor apresentado.
- O usuário inicia uma nova busca antes de a anterior terminar; a cidade e os dados exibidos não devem ficar associados a consultas diferentes.
- A tela tem largura mínima suportada ou o usuário navega apenas por teclado/leitor de tela.

## Assumptions

- A primeira versão é uma aplicação web responsiva, não um aplicativo nativo; isso é hipótese até a decisão de plataforma.
- A busca será iniciada pelo usuário. Geolocalização automática não faz parte do fluxo especificado enquanto não for aprovada.
- Open-Meteo será usado sem API key para geocodificação e dados meteorológicos; cobertura, limites e disponibilidade ainda precisam ser verificados.
- A interpretação aprovada de cinco dias é hoje + quatro dias, usando a data local da cidade selecionada.
- A hipótese funcional para a previsão é uma entrada diária com condição e mínima/máxima; granularidade e campos ainda precisam de validação.
- A unidade inicial será Celsius; não está decidido se a preferência será lembrada localmente entre visitas.
- Não haverá autenticação ou persistência de servidor na primeira versão. Isso não decide cache ou armazenamento local no navegador.
- O idioma inicial será pt-BR; a interface não terá outro idioma na primeira versão, salvo decisão posterior.

## Risks

| Risco | Probabilidade | Impacto | Mitigação |
| --- | --- | --- | --- |
| Open-Meteo indisponível, lento ou sujeito a limites | Média | Consultas falham, atrasam ou ficam limitadas | Validar limites, cobertura e disponibilidade; tratar falhas, monitorar dependência e acordar estratégia de cache/contingência. |
| Dados desatualizados ou sem cobertura | Média | Usuário toma decisões com informação incompleta ou indisponível | Verificar cobertura e atualização; informar cidade e contexto temporal dos dados quando a fonte fornecer essa informação. |
| Cidade homônima ou seleção geográfica incorreta | Média | Previsão de local errado e perda de confiança | Apresentar contexto geográfico nos resultados e tornar a seleção explícita. |
| Datas, fuso ou definição de cinco dias inconsistentes | Média | Período exibido diverge da expectativa do usuário | Usar a data local da cidade e testar mudança de dia, limites de mês/ano e fusos distintos. |
| Conversão de unidade inconsistente | Baixa | Temperatura incorreta ou confusa | Centralizar conversão, rotular unidades e validar com valores de referência. |
| Uso móvel ou acessibilidade inadequados | Média | Parte dos usuários não consegue concluir o fluxo | Definir critérios de aceite, testar viewports representativos, teclado e tecnologia assistiva. |
| Escopo ou métricas de sucesso indefinidos | Alta | Retrabalho, atrasos e divergência sobre conclusão da V1 | Validar questões abertas e metas propostas antes de aprovar a especificação para implementação. |
| Consultas a provedor externo sem comunicação clara de privacidade | Baixa/média | Perda de confiança e possíveis obrigações de conformidade | Minimizar dados enviados, informar o uso do provedor e validar requisitos de privacidade aplicáveis. |

## Out of Scope

Itens não previstos para a primeira versão, sujeitos a revisão se as perguntas em aberto mudarem o escopo:

- Aplicativos nativos para iOS ou Android.
- Login, contas, perfis ou sincronização entre dispositivos.
- Persistência de histórico, cidades favoritas ou preferências em servidor.
- Geolocalização automática.
- Funcionamento offline ou garantia de dados em cache.
- Alertas, notificações, mapas, radar meteorológico ou histórico climático.
- Previsão para além do período de cinco dias.
- Interface em idiomas além de pt-BR.

## Open Questions

1. A primeira versão será exclusivamente web responsiva ou também exigirá aplicativo nativo? Quais navegadores e versões serão suportados?
2. Quais países e localidades precisam de cobertura? Como tratar localidades sem região/estado nos resultados de busca?
3. A busca deve oferecer autocomplete, aceitar nomes parciais e tolerar acentos ou termos em outros idiomas?
4. Geolocalização automática deve ser considerada para uma versão futura? Como será tratada a permissão e a recusa?
5. Quais são os limites, frequência de atualização e cobertura confirmados do Open-Meteo para o público esperado?
6. “Clima atual” representa observação ou estimativa? Quais campos, além de temperatura e condição, são essenciais e como será indicada a hora da observação?
7. A previsão diária proposta é suficiente? Quais atributos por dia são obrigatórios e qual fuso horário deve reger as datas?
8. Celsius/Fahrenheit se aplica apenas à temperatura? Quais unidades serão usadas para vento, precipitação e pressão?
9. A escolha de unidade deve persistir localmente entre visitas? É permitido cache local, e por quanto tempo?
10. Qual comportamento de nova tentativa é esperado? Dados em cache podem ser apresentados durante uma falha e, em caso afirmativo, como serão identificados como desatualizados?
11. Quais metas de desempenho e disponibilidade serão aprovadas, em qual ambiente serão medidas e como a indisponibilidade do provedor será contabilizada?
12. WCAG 2.2 AA, viewport mínima de 320 CSS px e metas de toque são critérios de aceite aprovados?
13. Quais formatos de data/hora e convenções regionais serão usados na UI pt-BR?
14. Quais métricas de produto e fluxos concluídos determinarão o sucesso e a prontidão para lançamento da primeira versão?
15. Favoritos ou histórico mantidos apenas no navegador são desejados, apesar da ausência de persistência de servidor?