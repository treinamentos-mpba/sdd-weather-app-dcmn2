# Backlog de Tarefas — Weather App

Tarefas derivadas de `plans/weather-app-plan.md`. A ordem dos IDs é topológica: as dependências listadas devem estar concluídas antes do início da tarefa. Cada tarefa tem um tipo principal e, em geral, um único arquivo de implementação ou teste.

## Entrega 1 — Contratos e dados

### T-01
- **Título:** Definir contratos meteorológicos
- **Descrição curta:** Criar os tipos compartilhados de domínio e serviço.
- **Critérios de aceite:** `Unit`, `City`, `CurrentWeather`, `ForecastDay`, `WeatherData`, `RequestState`, `WeatherError` e `WeatherService` estão definidos em TypeScript strict; temperaturas usam Celsius e ausência é `null`.
- **Dependências:** Nenhuma.
- **Arquivos prováveis:** `src/types/weather.ts`
- **Tipo:** Data

### T-02
- **Título:** Validar input de busca
- **Descrição curta:** Implementar trim, limites e preservação do termo Unicode.
- **Critérios de aceite:** Vazio é rejeitado; são aceitos 1–100 pontos de código; caracteres especiais válidos são preservados; valores acima de 100 são rejeitados sem rede.
- **Dependências:** Nenhuma.
- **Arquivos prováveis:** `src/lib/searchValidation.ts`
- **Tipo:** Data

### T-03
- **Título:** Implementar conversão de temperatura
- **Descrição curta:** Criar conversões Celsius/Fahrenheit e regra de arredondamento.
- **Critérios de aceite:** Usa `F = C × 9/5 + 32`; exibe uma casa decimal e, em empate, arredonda afastando de zero.
- **Dependências:** T-01.
- **Arquivos prováveis:** `src/lib/temperature.ts`
- **Tipo:** Data

### T-04
- **Título:** Implementar formatação pt-BR
- **Descrição curta:** Formatar números e datas civis sem deslocamento UTC.
- **Critérios de aceite:** Números seguem pt-BR; data `YYYY-MM-DD` permanece o mesmo dia ao ser apresentada; formatação recebe explicitamente o timezone quando necessário.
- **Dependências:** T-01.
- **Arquivos prováveis:** `src/lib/format.ts`
- **Tipo:** Data

### T-05
- **Título:** Mapear códigos meteorológicos
- **Descrição curta:** Converter códigos WMO em descrições pt-BR.
- **Critérios de aceite:** Códigos suportados retornam descrição localizada; código desconhecido retorna condição indisponível, sem lançar erro.
- **Dependências:** T-01.
- **Arquivos prováveis:** `src/lib/weatherCodes.ts`
- **Tipo:** Data

### T-06
- **Título:** Normalizar resultados de geocoding
- **Descrição curta:** Mapear itens de geocoding para `City`.
- **Critérios de aceite:** Mapeia ID, nome, região, país e coordenadas; região e país podem faltar; resposta malformada é rejeitada como inválida.
- **Dependências:** T-01.
- **Arquivos prováveis:** `src/lib/openMeteoMapper.ts`
- **Tipo:** Data

### T-07
- **Título:** Normalizar resposta de forecast
- **Descrição curta:** Mapear current e arrays diários para `WeatherData`.
- **Critérios de aceite:** Mapeia pelo índice/data; gera exatamente cinco datas civis esperadas a partir de hoje no timezone retornado; dados ausentes recebem `null`; arrays órfãos são ignorados sem inventar valores.
- **Dependências:** T-01, T-03, T-04.
- **Arquivos prováveis:** `src/lib/openMeteoMapper.ts`
- **Tipo:** Data

### T-08
- **Título:** Testar validação da busca
- **Descrição curta:** Cobrir trim, limites Unicode e caracteres especiais.
- **Critérios de aceite:** Testes Vitest cobrem input vazio, 1, 100 e 101 pontos de código, espaços externos e preservação de acentos/apóstrofos/hífens.
- **Dependências:** T-02.
- **Arquivos prováveis:** `tests/unit/searchValidation.test.ts`
- **Tipo:** Test

### T-09
- **Título:** Testar conversão de temperatura
- **Descrição curta:** Cobrir conversão, precisão e desempate.
- **Critérios de aceite:** Testes Vitest cobrem conversão nos dois sentidos, valores negativos, valores de referência 0 °C/32 °F e empates positivos e negativos.
- **Dependências:** T-03.
- **Arquivos prováveis:** `tests/unit/temperature.test.ts`
- **Tipo:** Test

### T-10
- **Título:** Testar formatação de datas e números
- **Descrição curta:** Validar localização e estabilidade de datas civis.
- **Critérios de aceite:** Testes Vitest cobrem formatação pt-BR e datas próximas à meia-noite/virada de mês sem deslocamento de dia.
- **Dependências:** T-04.
- **Arquivos prováveis:** `tests/unit/format.test.ts`
- **Tipo:** Test

### T-11
- **Título:** Testar mapeadores e códigos WMO
- **Descrição curta:** Cobrir normalização de ambas as APIs e fallback de códigos.
- **Critérios de aceite:** Testes Vitest cobrem payload válido, campos opcionais, arrays desalinhados, dados parciais, estruturas inválidas e código WMO desconhecido.
- **Dependências:** T-05, T-06, T-07.
- **Arquivos prováveis:** `tests/unit/openMeteoMapper.test.ts`
- **Tipo:** Test

### T-12
- **Título:** Implementar serviço de geocoding
- **Descrição curta:** Consultar Open-Meteo Geocoding e classificar suas respostas.
- **Critérios de aceite:** `searchCities` usa `URLSearchParams`, `count=10`, `language=pt`, `format=json`; lista vazia é distinta de erro; aplica `AbortSignal` e timeout de 10 s; produz `WeatherError` tipado.
- **Dependências:** T-01, T-02, T-06.
- **Arquivos prováveis:** `src/services/weatherService.ts`
- **Tipo:** Data

### T-13
- **Título:** Implementar serviço de forecast
- **Descrição curta:** Consultar condições atuais e cinco dias para uma cidade.
- **Critérios de aceite:** `getWeather` usa coordenadas, `current`, `daily`, Celsius, `timezone=auto` e `forecast_days=5`; aplica timeout/abort e retorna `WeatherData` normalizado.
- **Dependências:** T-01, T-03, T-04, T-07, T-12.
- **Arquivos prováveis:** `src/services/weatherService.ts`
- **Tipo:** Data

### T-14
- **Título:** Testar serviço de geocoding
- **Descrição curta:** Mockar `fetch` e verificar o contrato de busca.
- **Critérios de aceite:** Testes Vitest verificam URL/parâmetros, lista vazia, HTTP, rede, JSON inválido, timeout, cancelamento e `retryable`.
- **Dependências:** T-12.
- **Arquivos prováveis:** `tests/unit/weatherService.geocoding.test.ts`
- **Tipo:** Test

### T-15
- **Título:** Testar serviço de forecast
- **Descrição curta:** Mockar `fetch` e verificar a consulta meteorológica.
- **Critérios de aceite:** Testes Vitest verificam coordenadas/parâmetros, Celsius, cinco dias, timeout, resposta parcial/inválida, erro HTTP/rede e cancelamento.
- **Dependências:** T-13.
- **Arquivos prováveis:** `tests/unit/weatherService.forecast.test.ts`
- **Tipo:** Test

## Entrega 2 — Estado e componentes

### T-16
- **Título:** Implementar hook `useWeather`
- **Descrição curta:** Orquestrar busca, seleção, clima, retry e unidade.
- **Critérios de aceite:** Expõe estados `idle/loading/success/empty/error`; seleciona cidade explicitamente; descarta respostas antigas; retry usa termo/cidade ativos; unidade não faz request; recarga inicia em Celsius.
- **Dependências:** T-12, T-13.
- **Arquivos prováveis:** `src/hooks/useWeather.ts`
- **Tipo:** Data

### T-17
- **Título:** Testar estados do hook
- **Descrição curta:** Substituir serviços por mocks e validar transições.
- **Critérios de aceite:** Testes cobrem estados de busca e forecast, seleção, retry, troca de unidade sem request e resposta fora de ordem.
- **Dependências:** T-16.
- **Arquivos prováveis:** `tests/unit/useWeather.test.tsx`
- **Tipo:** Test

### T-18
- **Título:** Implementar formulário `SearchBar`
- **Descrição curta:** Criar busca controlada e seleção de resultados.
- **Critérios de aceite:** Envia explicitamente; input inválido mostra correção sem request; resultados são identificáveis e selecionáveis por teclado e toque.
- **Dependências:** T-02, T-16.
- **Arquivos prováveis:** `src/components/SearchBar.tsx`
- **Tipo:** UI

### T-19
- **Título:** Implementar aviso `ProviderNotice`
- **Descrição curta:** Informar o envio de termo/coordenadas ao Open-Meteo.
- **Critérios de aceite:** Aviso em pt-BR fica visível junto à busca, explica o envio necessário ao provedor e não exige consentimento adicional.
- **Dependências:** Nenhuma.
- **Arquivos prováveis:** `src/components/ProviderNotice.tsx`
- **Tipo:** UI

### T-20
- **Título:** Implementar clima atual
- **Descrição curta:** Renderizar `CurrentWeather` em componente controlado por props.
- **Critérios de aceite:** Exibe temperatura/unidade, condição localizada e horário local; valores nulos são indicados como indisponíveis.
- **Dependências:** T-01, T-03, T-04, T-05.
- **Arquivos prováveis:** `src/components/CurrentWeather.tsx`
- **Tipo:** UI

### T-21
- **Título:** Implementar card diário
- **Descrição curta:** Renderizar uma data e suas métricas meteorológicas.
- **Critérios de aceite:** Exibe condição, mínima e máxima; cada campo nulo aparece como indisponível sem valor estimado.
- **Dependências:** T-01, T-03, T-04, T-05.
- **Arquivos prováveis:** `src/components/ForecastCard.tsx`
- **Tipo:** UI

### T-22
- **Título:** Implementar lista de cinco dias
- **Descrição curta:** Apresentar os cinco `ForecastDay` recebidos.
- **Critérios de aceite:** Renderiza cinco datas em ordem local; dias com dados parciais permanecem visíveis; cada item usa `ForecastCard`.
- **Dependências:** T-21.
- **Arquivos prováveis:** `src/components/ForecastList.tsx`
- **Tipo:** UI

### T-23
- **Título:** Implementar seletor de unidade
- **Descrição curta:** Criar controle acessível para Celsius/Fahrenheit.
- **Critérios de aceite:** Permite escolher exatamente uma unidade, comunica seleção por acessibilidade e chama apenas `setUnit`.
- **Dependências:** T-01.
- **Arquivos prováveis:** `src/components/UnitToggle.tsx`
- **Tipo:** UI

### T-24
- **Título:** Implementar estado de carregamento
- **Descrição curta:** Apresentar loading perceptível e acessível.
- **Critérios de aceite:** Mensagem/status é anunciado por tecnologia assistiva e não depende apenas de cor.
- **Dependências:** Nenhuma.
- **Arquivos prováveis:** `src/components/states/LoadingState.tsx`
- **Tipo:** UI

### T-25
- **Título:** Implementar estado vazio
- **Descrição curta:** Apresentar busca sem resultados ou dados indisponíveis.
- **Critérios de aceite:** Mensagem distingue ausência de resultado de indisponibilidade meteorológica e oferece ação adequada.
- **Dependências:** Nenhuma.
- **Arquivos prováveis:** `src/components/states/EmptyState.tsx`
- **Tipo:** UI

### T-26
- **Título:** Implementar estado de erro
- **Descrição curta:** Apresentar falhas recuperáveis com retry.
- **Critérios de aceite:** Mensagem acessível não expõe detalhes técnicos e retry chama o callback recebido por props.
- **Dependências:** T-01.
- **Arquivos prováveis:** `src/components/states/ErrorState.tsx`
- **Tipo:** UI

### T-27
- **Título:** Compor tela principal
- **Descrição curta:** Conectar `useWeather` aos componentes sem lógica de rede em `App`.
- **Critérios de aceite:** `App` passa estado e callbacks; não acessa services diretamente; apresenta aviso, busca, clima, previsão, unidade e estados correspondentes.
- **Dependências:** T-16, T-18, T-19, T-20, T-22, T-23, T-24, T-25, T-26.
- **Arquivos prováveis:** `src/App.tsx`
- **Tipo:** UI

### T-28
- **Título:** Configurar entrada da aplicação
- **Descrição curta:** Montar `App` no elemento raiz React.
- **Critérios de aceite:** `main.tsx` importa estilos globais e monta `App` uma única vez no root configurado.
- **Dependências:** T-27.
- **Arquivos prováveis:** `src/main.tsx`
- **Tipo:** UI

### T-29
- **Título:** Aplicar estilos responsivos
- **Descrição curta:** Implementar layout mobile-first para os fluxos principais.
- **Critérios de aceite:** Busca, clima atual, cinco dias e unidade são utilizáveis a partir de 320 CSS px; não há rolagem horizontal na área principal nos viewports do plano.
- **Dependências:** T-28.
- **Arquivos prováveis:** `src/styles/index.css`
- **Tipo:** UI

### T-30
- **Título:** Testar busca e aviso de privacidade
- **Descrição curta:** Cobrir interação de busca e divulgação ao provedor.
- **Critérios de aceite:** Testing Library verifica input inválido sem rede, seleção/teclado e texto visível do aviso em pt-BR.
- **Dependências:** T-18, T-19.
- **Arquivos prováveis:** `tests/unit/SearchBar.test.tsx`, `tests/unit/ProviderNotice.test.tsx`
- **Tipo:** Test

### T-31
- **Título:** Testar componente de clima atual
- **Descrição curta:** Verificar sucesso e dados incompletos no card atual.
- **Critérios de aceite:** Testes verificam temperatura/unidade, condição, horário e campos indisponíveis acessíveis.
- **Dependências:** T-20.
- **Arquivos prováveis:** `tests/unit/CurrentWeather.test.tsx`
- **Tipo:** Test

### T-32
- **Título:** Testar componentes da previsão
- **Descrição curta:** Verificar composição e conteúdo dos cinco cards diários.
- **Critérios de aceite:** Testes verificam ordem e número de dias, condições localizadas e campos nulos sem valores fabricados.
- **Dependências:** T-21, T-22.
- **Arquivos prováveis:** `tests/unit/ForecastCard.test.tsx`, `tests/unit/ForecastList.test.tsx`
- **Tipo:** Test

### T-33
- **Título:** Testar seletor de unidade
- **Descrição curta:** Verificar teclado, seleção única e callback do controle.
- **Critérios de aceite:** Testing Library confirma seleção Celsius/Fahrenheit, estado acessível e chamada de `setUnit` sem rede.
- **Dependências:** T-23.
- **Arquivos prováveis:** `tests/unit/UnitToggle.test.tsx`
- **Tipo:** Test

### T-34
- **Título:** Testar estado de carregamento
- **Descrição curta:** Verificar conteúdo e semântica acessível do loading.
- **Critérios de aceite:** Teste confirma status anunciado e indicação não dependente apenas de cor.
- **Dependências:** T-24.
- **Arquivos prováveis:** `tests/unit/LoadingState.test.tsx`
- **Tipo:** Test

### T-35
- **Título:** Testar estados vazio e erro
- **Descrição curta:** Verificar mensagens, ações e acessibilidade dos estados.
- **Critérios de aceite:** Testes distinguem vazio/erro, verificam orientação e retry pelo callback, e confirmam ausência de detalhes internos.
- **Dependências:** T-25, T-26.
- **Arquivos prováveis:** `tests/unit/EmptyState.test.tsx`, `tests/unit/ErrorState.test.tsx`
- **Tipo:** Test

## Entrega 3 — Integração e lançamento

### T-36
- **Título:** Testar fluxo principal com Playwright
- **Descrição curta:** Validar busca, seleção, dados e unidade com APIs interceptadas.
- **Critérios de aceite:** Fluxo completo chega ao clima e previsão; cidade ativa é correta; alternar unidade atualiza valores sem nova chamada; aviso de privacidade está visível.
- **Dependências:** T-28, T-29.
- **Arquivos prováveis:** `tests/e2e/weather-happy-path.spec.ts`
- **Tipo:** Test

### T-37
- **Título:** Testar erros e edge cases com Playwright
- **Descrição curta:** Cobrir vazio, falhas, timeout e resposta parcial.
- **Critérios de aceite:** Testes interceptados cobrem cidade homônima, geocoding vazio, input vazio, falha HTTP/rede, timeout, retry bem-sucedido e forecast parcial.
- **Dependências:** T-28, T-29, T-36.
- **Arquivos prováveis:** `tests/e2e/weather-error-states.spec.ts`
- **Tipo:** Test

### T-38
- **Título:** Adicionar dependência axe para Playwright
- **Descrição curta:** Instalar suporte à auditoria automatizada de acessibilidade.
- **Critérios de aceite:** `@axe-core/playwright` consta como dependência de desenvolvimento e instalação pnpm reproduz a lockfile.
- **Dependências:** T-28.
- **Arquivos prováveis:** `package.json`, `pnpm-lock.yaml`
- **Tipo:** Infra

### T-39
- **Título:** Auditar fluxos com axe
- **Descrição curta:** Executar verificações automatizadas nos estados principais.
- **Critérios de aceite:** Testes axe cobrem tela inicial, resultados e erro; violações críticas/altas bloqueiam a suíte ou têm exceção documentada.
- **Dependências:** T-36, T-38.
- **Arquivos prováveis:** `tests/e2e/accessibility.spec.ts`
- **Tipo:** Test

### T-40
- **Título:** Configurar projetos Playwright por browser
- **Descrição curta:** Preparar execução da matriz cross-browser da spec.
- **Critérios de aceite:** `playwright.config.ts` define projetos para Chromium/Chrome, Edge, Firefox e WebKit e documenta a estratégia para versões estável atual/anterior suportadas no CI.
- **Dependências:** T-36.
- **Arquivos prováveis:** `playwright.config.ts`
- **Tipo:** Test

### T-41
- **Título:** Registrar smoke test em dispositivos reais
- **Descrição curta:** Definir execução e evidência para Safari/iOS e Chrome/Android.
- **Critérios de aceite:** Checklist registra dispositivo/versão, viewport, fluxo principal e resultado; WebKit desktop não é aceito como substituto de Safari iOS real.
- **Dependências:** T-40.
- **Arquivos prováveis:** `docs/browser-compatibility.md`
- **Tipo:** Test

### T-42
- **Título:** Medir desempenho do fluxo principal
- **Descrição curta:** Criar execução sintética no perfil estabelecido pela NFR4.
- **Critérios de aceite:** 30 execuções em staging com perfil de viewport/rede definido registram latência; pelo menos 95% mostram clima e previsão em até 3 s com resposta controlada do provedor ≤500 ms.
- **Dependências:** T-36.
- **Arquivos prováveis:** `tests/e2e/performance.spec.ts`
- **Tipo:** Test

### T-43
- **Título:** Documentar probes de disponibilidade
- **Descrição curta:** Definir como medir frontend e provedor separadamente.
- **Critérios de aceite:** Documento especifica probe a cada minuto, sinal de sucesso do frontend, cálculo mensal de 99,5%, alertas e métrica separada para Open-Meteo.
- **Dependências:** T-28.
- **Arquivos prováveis:** `docs/monitoring.md`
- **Tipo:** Infra

### T-44
- **Título:** Preparar protocolo de usabilidade
- **Descrição curta:** Definir tarefas e registro para o gate de 10 participantes.
- **Critérios de aceite:** Protocolo inclui seleção da cidade correta, consulta de cinco dias e troca de unidade; define conclusão sem ajuda e coleta de erros por tarefa sem telemetria.
- **Dependências:** T-36.
- **Arquivos prováveis:** `docs/usability-test-protocol.md`
- **Tipo:** Test

### T-45
- **Título:** Executar gate de usabilidade
- **Descrição curta:** Conduzir o teste moderado e registrar resultados.
- **Critérios de aceite:** 10 pessoas não familiarizadas completam as tarefas; resultados registram conclusão e erros por tarefa; pelo menos 8 de 10 concluem o fluxo sem ajuda.
- **Dependências:** T-44.
- **Arquivos prováveis:** `docs/usability-test-results.md`
- **Tipo:** Test

### T-46
- **Título:** Validar cobertura e termos Open-Meteo
- **Descrição curta:** Confirmar adequação do provedor para a geografia e uso previstos.
- **Critérios de aceite:** Cobertura, atualização, limites e adequação dos termos ao uso comercial são verificados e registrados antes do lançamento.
- **Dependências:** T-12, T-13.
- **Arquivos prováveis:** `docs/provider-validation.md`
- **Tipo:** Infra

### T-47
- **Título:** Revisar aviso e política de privacidade
- **Descrição curta:** Aprovar o texto que informa o envio de busca/coordenadas ao provedor.
- **Critérios de aceite:** Áreas responsáveis aprovam a comunicação; aviso documentado corresponde ao `ProviderNotice`; não introduz coleta, cookies ou analytics fora do escopo.
- **Dependências:** T-19, T-46.
- **Arquivos prováveis:** `docs/privacy-notice.md`
- **Tipo:** Infra

### T-48
- **Título:** Configurar workflow de CI
- **Descrição curta:** Executar lint, build e suítes automatizadas nos pull requests.
- **Critérios de aceite:** Workflow instala dependências/browsers e executa `pnpm lint`, `pnpm build`, `pnpm test` e `pnpm test:e2e`; qualquer falha bloqueia a verificação.
- **Dependências:** T-14, T-15, T-17, T-30, T-31, T-32, T-33, T-34, T-35, T-37, T-39, T-40.
- **Arquivos prováveis:** `.github/workflows/ci.yml`
- **Tipo:** Infra
