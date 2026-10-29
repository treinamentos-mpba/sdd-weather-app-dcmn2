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

As probabilidades abaixo são avaliações preliminares, pois a fonte de dados, o público e os critérios de operação ainda não foram definidos.

| Categoria | Risco | Probabilidade | Impacto | Estratégia de mitigação |
| --- | --- | --- | --- | --- |
| Técnico | Fonte meteorológica indisponível, lenta ou sujeita a limites de uso | Média | Alto: consultas podem falhar, atrasar ou exceder limites/custos | Avaliar cobertura, SLA, limites e custos antes da escolha; tratar falhas, monitorar dependência e definir cache ou alternativa quando viável. |
| Técnico e produto | Dados desatualizados, imprecisos ou sem cobertura para a cidade buscada | Média | Alto: decisões do usuário podem se basear em informação incorreta ou indisponível | Validar cobertura e frequência de atualização da fonte; exibir local e horário da observação e comunicar indisponibilidade de dados. |
| Produto | Cidades homônimas ou geocodificação incorreta levam à previsão do local errado | Média | Alto: reduz confiança e pode induzir decisões inadequadas | Mostrar cidade, região/estado e país nos resultados; exigir seleção clara quando houver ambiguidades e validar exemplos de busca. |
| Técnico e produto | Interpretação divergente dos cinco dias, datas ou fuso horário | Média | Médio/alto: previsão exibida para datas inesperadas | Definir período e granularidade; usar o fuso do local consultado e testar limites de dia e mudanças de horário. |
| Técnico | Conversão ou apresentação de unidades inconsistente entre clima atual e previsão | Baixa | Alto: valores podem ser interpretados incorretamente | Centralizar conversões, exibir unidade junto aos valores e cobrir ambas as unidades com testes automatizados. |
| Técnico e produto | Rede instável ou falha da API deixa a tela sem dados ou sem orientação | Média | Alto: interrompe o principal fluxo do produto | Implementar estados de carregamento/erro e nova tentativa; definir política de cache e indicar claramente quando dados podem estar desatualizados. |
| Produto | Experiência móvel inadequada ou incompatível com dispositivos/navegadores relevantes | Média | Alto: impede o uso no contexto explicitamente pedido | Definir matriz mínima de navegadores e larguras; testar busca, leitura e controles em telas pequenas e com toque. |
| Técnico e produto | Acessibilidade insuficiente impede alguns usuários de concluir consultas | Média | Médio/alto: exclui usuários e pode gerar barreiras de conformidade | Adotar critérios de acessibilidade verificáveis, testar teclado e leitor de tela e corrigir contraste e semântica dos controles. |
| Produto | Escopo e critérios de sucesso indefinidos geram entregas desalinhadas ou expansão contínua | Alta | Alto: atrasos, retrabalho e divergência sobre o que significa concluir a primeira versão | Aprovar público prioritário, escopo de MVP, métricas e critérios de aceite antes do desenvolvimento; tratar extras como decisões explícitas. |
| Técnico e produto | Coleta de localização ou histórico sem política clara de privacidade | Baixa/média | Alto: perda de confiança e possíveis obrigações legais | Minimizar coleta e retenção; pedir permissão quando necessário, documentar uso dos dados e validar requisitos de privacidade aplicáveis. |
| Produto | Aplicação não atende às necessidades reais e tem baixa adoção | Média | Alto: baixo valor entregue apesar da implementação correta | Validar personas e fluxos com usuários; observar métricas de busca concluída e retorno, revisando prioridades com base em evidências. |

## Perguntas em Aberto

1. **Quem são os usuários prioritários e que decisões esperam tomar com o app?** Impacto: orienta prioridades, conteúdo e fluxos principais.
2. **"Usar em dispositivos móveis" significa site responsivo, aplicativo nativo ou ambos? Quais sistemas e navegadores devem ser suportados?** Impacto: altera arquitetura, distribuição, esforço e plano de testes.
3. **A busca aceita nomes parciais, acentos e diferentes idiomas? Deve sugerir resultados enquanto o usuário digita?** Impacto: afeta usabilidade, comportamento da busca e cobertura geográfica necessária.
4. **Como distinguir cidades homônimas? Devem aparecer estado/região e país? Quais países ou localidades são relevantes?** Impacto: uma seleção ambígua pode exibir dados do local errado.
5. **O app deve detectar localização automaticamente, pedir permissão ou exigir busca manual?** Impacto: altera o fluxo inicial e envolve permissões, privacidade e tratamento de localização negada.
6. **De onde virão as observações e previsões? Qual cobertura geográfica, custo e limite de uso são aceitáveis?** Impacto: determina viabilidade, precisão, disponibilidade e dependências externas.
7. **"Clima atual" significa observação recente ou condição estimada? Com que frequência os dados precisam ser atualizados?** Impacto: afeta precisão percebida, latência, custo e exibição do horário da medição.
8. **Os cinco dias incluem hoje ou os cinco dias seguintes? A previsão será diária, por hora ou ambas? Qual fuso horário define a data?** Impacto: altera o período consultado, o layout e a interpretação das datas.
9. **Quais informações devem aparecer no clima atual e na previsão, além da temperatura?** Impacto: define conteúdo, densidade visual e necessidades da fonte de dados.
10. **Celsius/Fahrenheit se aplica apenas à temperatura? Quais serão as unidades de vento, precipitação e pressão? Qual unidade será inicial?** Impacto: afeta consistência, compreensão dos dados e conversões.
11. **A unidade escolhida deve continuar após recarregar ou voltar ao app?** Impacto: define se é preciso armazenar preferências e onde fazê-lo.
12. **Quais idiomas, formatos de data/hora e convenções regionais serão suportados?** Impacto: afeta textos, busca, formatação dos dados e esforço de localização.
13. **O que deve acontecer se não houver resultados, a consulta falhar ou a fonte estiver indisponível? Deve haver nova tentativa ou dados em cache?** Impacto: sem esse comportamento, falhas de rede podem deixar o usuário sem explicação ou próximo passo.
14. **Quais metas de disponibilidade e tempo de resposta são aceitáveis, e em quais condições serão medidas?** Impacto: sem metas, não há critério objetivo para avaliar qualidade ou negociar limites da fonte externa.
15. **Qual nível de acessibilidade é esperado? Quais larguras de tela e métodos de interação precisam ser validados?** Impacto: pode exigir decisões de design e implementação que são difíceis de corrigir no fim do projeto.
16. **O usuário poderá salvar cidades, consultar histórico ou sincronizar preferências? É necessário login?** Impacto: pode ampliar significativamente o escopo e introduzir persistência e tratamento de dados pessoais.
17. **Quais dados de busca ou localização serão coletados, armazenados e por quanto tempo?** Impacto: afeta consentimento, segurança, política de privacidade e obrigações legais.
18. **Como será medido o sucesso do produto e quais fluxos precisam estar concluídos para considerar a primeira versão pronta?** Impacto: sem métricas e critérios de aceite, prioridades e conclusão ficam sujeitas a interpretações diferentes.

## Suposições

- A aplicação será acessada pela web; o briefing não especifica aplicativo nativo.
- O usuário escolhe uma cidade por meio de busca, pois não foi mencionada geolocalização automática.
- As consultas dependem de uma fonte de dados meteorológicos externa e de conectividade, até que outra arquitetura seja definida.
- Celsius e Fahrenheit são as duas unidades de temperatura necessárias na primeira versão, conforme o briefing.
- Não se assume autenticação, persistência de preferências, favoritos, histórico ou funcionamento offline; esses pontos permanecem em aberto.

## Personas iniciais

Estas personas são hipóteses para orientar a descoberta; devem ser validadas com usuários reais.

| Persona | Objetivo principal | Contexto de uso | Métrica de sucesso para a pessoa |
| --- | --- | --- | --- |
| **Planejadora da semana** | Consultar os próximos dias para escolher quando realizar compromissos ou atividades. | Principalmente desktop, ao organizar a agenda; consulta ocasional pelo celular. | Encontra a cidade e entende a previsão dos cinco dias em até um minuto, sem precisar consultar outra fonte para interpretar as datas. |
| **Decisor do dia a dia** | Ver rapidamente as condições atuais para decidir o que vestir ou como se deslocar. | Principalmente mobile, em consultas rápidas antes de sair de casa ou durante o dia. | Identifica a temperatura atual, a condição do tempo e a unidade escolhida em até 30 segundos. |
| **Pessoa viajando** | Pesquisar uma cidade que pode não conhecer e avaliar as condições durante a viagem. | Mobile em trânsito; desktop ao planejar antes da viagem. | Seleciona o local correto entre resultados semelhantes e acessa a previsão sem confundir cidade, região ou fuso horário. |

## Diretrizes para a próxima etapa

As escolhas abaixo são propostas de trabalho, não decisões aprovadas no briefing:

- Avaliar uma fonte meteorológica sem chave de API, verificando cobertura e limites antes de selecioná-la.
- Definir se os cinco dias correspondem ao dia atual mais quatro dias ou aos cinco dias seguintes.
- Confirmar Celsius como unidade inicial e se a preferência deve ser persistida.
- Confirmar idioma, política de geolocalização, necessidade de cache e requisitos de conta antes de fechar a especificação.
