# Discovery — Aplicação de Previsão do Tempo

## Contexto

A empresa solicitou uma aplicação de previsão do tempo para que usuários consultem condições meteorológicas de cidades escolhidas por eles. O escopo informado inclui busca de cidades, consulta do clima atual, previsão para cinco dias, alternância entre Celsius e Fahrenheit e uso em dispositivos móveis.

O briefing define as capacidades principais, mas ainda não determina a fonte e a cobertura dos dados, o significado exato de "cinco dias", o conteúdo exibido em cada consulta nem os critérios de desempenho e acessibilidade. Essas decisões precisam ser esclarecidas antes de se tornarem critérios de aceite.

## Requisitos Funcionais

- **RF1 — Buscar cidade:** permitir que o usuário pesquise uma cidade e selecione o local desejado. A forma de tratar nomes iguais ou resultados múltiplos precisa ser definida.
- **RF2 — Consultar clima atual:** exibir as condições atuais para a cidade selecionada. Quais atributos serão apresentados ainda precisam ser especificados.
- **RF3 — Consultar previsão:** exibir a previsão para cinco dias para a cidade selecionada. É necessário esclarecer se o período inclui o dia atual e quais informações serão mostradas por dia.
- **RF4 — Alternar unidade:** permitir alternar entre Celsius e Fahrenheit e refletir a unidade escolhida nas temperaturas exibidas, tanto no clima atual quanto na previsão.
- **RF5 — Comunicar estados da consulta:** informar quando uma busca ou consulta estiver carregando, não encontrar resultados ou falhar, com orientação para o próximo passo quando aplicável. Este requisito de suporte é uma recomendação para tornar o fluxo utilizável e deve ser validado.

## Requisitos Não-Funcionais

- **RNF1 — Responsividade:** a interface deve ser utilizável em dispositivos móveis, conforme solicitado no briefing, e adaptar o conteúdo a diferentes tamanhos de tela. Faixas de viewport e dispositivos suportados precisam ser acordados.
- **RNF2 — Usabilidade:** busca, leitura das condições e alternância de unidade devem ser compreensíveis e operáveis em telas móveis, inclusive por toque. Critérios de avaliação ainda precisam ser definidos.
- **RNF3 — Acessibilidade:** controles e informações meteorológicas devem ser acessíveis por teclado e tecnologias assistivas, com semântica e contraste adequados. O nível de conformidade esperado precisa ser acordado.
- **RNF4 — Desempenho:** busca e apresentação de dados devem ocorrer em tempo aceitável para o usuário. Metas de latência e condições de rede para medi-las estão em aberto.
- **RNF5 — Resiliência:** a experiência deve comunicar falhas de rede ou indisponibilidade dos dados sem deixar o usuário sem contexto. Comportamento de nova tentativa e uso de dados em cache precisam ser definidos.

## Riscos

| Risco | Probabilidade | Impacto | Mitigação sugerida |
| --- | --- | --- | --- |
| Fonte meteorológica indisponível, lenta ou com limites de uso | Média; confirmar após escolha da fonte | Alto: consultas podem falhar ou demorar | Avaliar cobertura, limites e disponibilidade antes da escolha; prever tratamento de falha e, se apropriado, cache. |
| Busca retornar cidades homônimas ou localização imprecisa | Alta em localidades com nomes repetidos | Alto: dados podem ser exibidos para o lugar errado | Apresentar informações de desambiguação, como estado/região e país, antes da seleção. |
| Definição ambígua de cinco dias ou fuso horário | Média | Médio/alto: datas e período podem divergir da expectativa | Definir período, timezone e apresentação das datas; validar exemplos com usuários. |
| Interface ou controles difíceis de usar em telas pequenas | Média | Alto: compromete o requisito explícito de uso móvel | Projetar e validar em viewports móveis representativos, incluindo telas estreitas e testes em dispositivo. |
| Conversão de temperatura ou unidades inconsistente entre telas | Baixa/média | Alto: informação meteorológica incorreta ou confusa | Definir unidade como preferência consistente e validar conversões e exibição em testes. |
| Conectividade instável impedir a consulta | Média | Médio/alto: usuário pode não obter os dados | Comunicar estado e permitir nova tentativa; decidir se dados em cache devem ser exibidos e identificados como desatualizados. |

## Perguntas em Aberto

1. **Qual fonte de dados será usada e qual cobertura geográfica e frequência de atualização são necessárias?** Impacto: define disponibilidade, precisão, limites de uso e possíveis custos.
2. **O período de cinco dias inclui hoje ou os cinco dias seguintes? A previsão será diária ou incluirá intervalos horários?** Impacto: altera conteúdo, datas e layout.
3. **Quais atributos devem aparecer no clima atual e em cada dia da previsão?** Impacto: define o conteúdo essencial e a densidade da interface.
4. **Como o usuário deve escolher entre cidades com o mesmo nome? Quais localidades/regiões precisam ser suportadas?** Impacto: afeta precisão da busca e prevenção de consultas ao local errado.
5. **Qual será a unidade inicial e a escolha deve persistir entre visitas?** Impacto: afeta a primeira consulta, preferências e comportamento após recarregar a aplicação.
6. **Quais idiomas e formatos de data/hora devem ser suportados?** Impacto: afeta textos, localização e legibilidade dos dados.
7. **A aplicação deve solicitar geolocalização ou sempre exigir busca manual?** Impacto: altera o fluxo inicial, permissões e tratamento de privacidade.
8. **Qual comportamento é esperado sem conexão ou durante falha da fonte de dados? É necessário cache?** Impacto: afeta resiliência, atualização e complexidade de implementação.
9. **Quais navegadores, tamanhos de tela e critérios de acessibilidade são requisitos de aceite?** Impacto: define a matriz de compatibilidade e o esforço de validação.
10. **Há necessidade de login, cidades favoritas ou histórico de buscas?** Impacto: pode introduzir persistência, dados pessoais e escopo adicional.

## Suposições

- A aplicação será acessada pela web; o briefing não especifica aplicativo nativo.
- O usuário escolhe uma cidade por meio de busca, pois não foi mencionada geolocalização automática.
- As consultas dependem de uma fonte de dados meteorológicos externa e de conectividade, até que outra arquitetura seja definida.
- Celsius e Fahrenheit são as duas unidades de temperatura necessárias na primeira versão, conforme o briefing.
- Não se assume autenticação, persistência de preferências, favoritos, histórico ou funcionamento offline; esses pontos permanecem em aberto.

## Personas iniciais

Estas personas são arquétipos para orientar descoberta posterior, não segmentos confirmados por pesquisa.

- **Pessoa que planeja a semana:** consulta a previsão de cinco dias para organizar compromissos e atividades.
- **Pessoa que decide o que fazer hoje:** consulta rapidamente o clima atual, muitas vezes pelo celular, para planejar o dia.

## Diretrizes para a próxima etapa

As escolhas abaixo são propostas de trabalho, não decisões aprovadas no briefing:

- Avaliar uma fonte meteorológica sem chave de API, verificando cobertura e limites antes de selecioná-la.
- Definir se os cinco dias correspondem ao dia atual mais quatro dias ou aos cinco dias seguintes.
- Confirmar Celsius como unidade inicial e se a preferência deve ser persistida.
- Confirmar idioma, política de geolocalização, necessidade de cache e requisitos de conta antes de fechar a especificação.
