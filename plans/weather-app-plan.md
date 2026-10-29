# Plano Técnico — Weather App

## Architecture

Aplicação SPA client-side, sem backend próprio. O fluxo segue uma direção única: `App → components + hooks → services → lib/types`. `lib` e `types` não dependem de React nem de APIs de rede. Não criar camada genérica de repositório, cache ou state store.

- **Apresentação (`components`):** componentes controlados por props, responsáveis por renderizar dados, estados e ações acessíveis. Não fazem `fetch`, não conhecem URLs e não mantêm cópia do estado global do fluxo.
- **Composição (`App.tsx`):** instancia `useWeather`, liga estado e callbacks aos componentes e define a composição da tela; não contém protocolo HTTP nem regras de conversão.
- **Orquestração (`hooks`):** `useWeather` mantém resultados e estados de busca/forecast, cidade selecionada e unidade; coordena `searchCities`/`getWeather`, retry, cancelamento e descarte de respostas antigas.
- **Acesso a dados (`services`):** `weatherService` monta chamadas Open-Meteo, aplica timeout, trata status/JSON e entrega modelos internos. Não importa React nem componentes.
- **Funções puras (`lib`):** validação de input, mapeamento dos payloads Open-Meteo, conversão de temperatura, mapeamento WMO e formatação. Recebem valores e retornam valores; não fazem I/O nem alteram estado.
- **Contratos (`types`):** tipos compartilhados de cidade, clima, previsão, unidade e erros; não contêm comportamento de runtime.

Não há autenticação, API key, servidor da aplicação, persistência ou analytics na V1. Cada camada possui uma fronteira de teste: funções puras com testes determinísticos; serviços com `fetch` mockado; hook com serviço substituído; componentes com props e interações simuladas; E2E cobre o fluxo integrado.

## Tech Stack

- React 19 e React DOM, conforme dependências existentes.
- TypeScript 6 em modo strict.
- Vite 8 para desenvolvimento e build.
- Tailwind CSS 3 para apresentação responsiva.
- `fetch`, `URLSearchParams` e `AbortController` nativos para integração HTTP e timeout; nenhuma biblioteca de estado ou HTTP adicional.
- Vitest, Testing Library e `user-event` para testes unitários e de componentes; Playwright para E2E.
- Biome para lint e formatação; pnpm como gerenciador; Node.js 22 ou superior.
- Para axe automatizado no Playwright, adicionar `@axe-core/playwright` como dependência de desenvolvimento, pois não consta atualmente no `package.json`.

## Project Structure

Manter os diretórios convencionados no repositório; cada componente visual em seu próprio arquivo. Dependências seguem as setas definidas em **Architecture**; não criar barrel files ou subcamadas sem necessidade demonstrada.

```text
src/
  App.tsx
  main.tsx
  components/
    # Apresentação; recebe dados e callbacks por props
    SearchBar.tsx
    CurrentWeather.tsx
    ForecastList.tsx
    ForecastCard.tsx
    UnitToggle.tsx
    states/
      LoadingState.tsx
      EmptyState.tsx
      ErrorState.tsx
  hooks/
    # Estado e orquestração do fluxo da tela
    useWeather.ts
  services/
    # Fronteira HTTP com Open-Meteo
    weatherService.ts
  lib/
    # Funções puras, sem React ou I/O
    format.ts
    temperature.ts
    weatherCodes.ts
    openMeteoMapper.ts
    searchValidation.ts
  types/
    # Contratos compartilhados
    weather.ts
  styles/
    index.css
```

**Por que essa separação facilita testes:**

- `lib/`: testar limites, códigos e conversões com entradas/saídas determinísticas, sem mocks de React ou rede.
- `services/`: substituir `fetch` por respostas controladas para verificar URLs, parâmetros, timeout e mapeamento de erro.
- `hooks/`: substituir `weatherService` para testar transições de estado, retry e concorrência sem depender da Open-Meteo.
- `components/`: renderizar props e testar teclado, acessibilidade e estados sem montar o hook nem fazer rede.
- E2E: testar somente os fluxos críticos com Playwright e rotas de API interceptadas; reservar chamadas reais ao provedor para verificação manual/sintética separada.

## Data Model

Os tipos internos são independentes do formato Open-Meteo, com nomes dos campos de origem indicados nos comentários. Temperaturas são guardadas em Celsius para evitar conversões repetidas e inconsistências; a unidade de apresentação é estado de UI.

```ts
type Unit = "celsius" | "fahrenheit";

interface City {
  id?: number; // Open-Meteo geocoding result id.
  name: string; // Open-Meteo `name`.
  region?: string; // Open-Meteo `admin1`, when provided.
  country?: string; // Open-Meteo `country`, when provided.
  latitude: number; // Open-Meteo `latitude`; used in forecast requests.
  longitude: number; // Open-Meteo `longitude`; used in forecast requests.
}

interface CurrentWeather {
  temperatureC: number | null; // Open-Meteo `current.temperature_2m`, normalized to Celsius.
  weatherCode: number | null; // Open-Meteo `current.weather_code`; mapped to a pt-BR description.
  observedAt: string | null; // Open-Meteo `current.time`, local to the returned timezone.
}

interface ForecastDay {
  date: string; // Open-Meteo `daily.time`, as local YYYY-MM-DD (not UTC).
  weatherCode: number | null; // Open-Meteo `daily.weather_code`.
  minimumC: number | null; // Open-Meteo `daily.temperature_2m_min`, normalized to Celsius.
  maximumC: number | null; // Open-Meteo `daily.temperature_2m_max`, normalized to Celsius.
}

interface WeatherData {
  city: City; // Selected geocoding result used for this response.
  timezone: string; // Open-Meteo forecast response `timezone` for this location.
  current: CurrentWeather; // Current observation returned for the selected city.
  forecast: ForecastDay[]; // Five local calendar days, today through today + 4.
}

type RequestState<T> =
  | { status: "idle" | "loading"; data: null; error: null }
  | { status: "success"; data: T; error: null }
  | { status: "empty"; data: null; error: null }
  | { status: "error"; data: null; error: WeatherError };

type WeatherErrorCode = "timeout" | "network" | "http" | "invalid-response";

interface WeatherError {
  source: "geocoding" | "forecast";
  code: WeatherErrorCode;
  retryable: boolean;
}
```

`City` guarda as coordenadas selecionadas como identidade para consultas, mesmo quando o nome não é único. `ForecastDay.date` é uma data civil (`YYYY-MM-DD`), não um instante UTC. `WeatherData.forecast` é normalizado para cinco posições; campos faltantes recebem `null`, nunca valor interpolado. `weatherCode` é traduzido para pt-BR por uma função pura; código desconhecido produz condição indisponível.

Contrato de alto nível do serviço:

```ts
interface WeatherService {
  searchCities(query: string, signal?: AbortSignal): Promise<City[]>;
  getWeather(city: City, signal?: AbortSignal): Promise<WeatherData>;
}
```

Contrato do hook para a UI: expor `searchState`, `weatherState`, `selectedCity`, `unit: Unit`, `search(query)`, `selectCity(city)`, `setUnit(unit: Unit)` e `retryWeather()`. Componentes não acessam Open-Meteo diretamente.

## Data Flow

1. A pessoa informa a cidade; a UI remove espaços externos e valida o limite de 1–100 caracteres Unicode. Campo vazio ou termo acima do limite não inicia rede.
2. O hook inicia geocodificação; o serviço codifica o termo com `URLSearchParams`, inicia timeout de 10 s e converte os resultados para `City[]`.
3. A UI apresenta os resultados com os campos de localização fornecidos. A pessoa seleciona explicitamente um resultado; não há seleção automática de homônimos.
4. A seleção dispara `getWeather(city)`. O serviço consulta a previsão para latitude/longitude, solicita valores Celsius e `timezone=auto`, e normaliza clima atual, timezone e cinco dias.
5. O hook publica o snapshot. Componentes exibem a condição traduzida, datas locais e temperaturas formatadas na unidade corrente.
6. Alternar a unidade recalcula apenas a apresentação a partir dos valores Celsius normalizados. Recarregar a página restaura Celsius.
7. Uma resposta de operação cancelada ou antiga é descartada. Retry manual repete a consulta para a cidade selecionada.

## External APIs

### Geocoding — Open-Meteo

- **URL:** `GET https://geocoding-api.open-meteo.com/v1/search?name=Lisbon&count=10&language=pt&format=json`. Construir query strings com `URLSearchParams`, sem concatenar input diretamente.
- **Parâmetros:** `name` obrigatório; `count=10` limita sugestões; `language=pt` pede nomes/localizações em português quando disponíveis; `format=json` solicita JSON. Este endpoint não usa `current`, `daily` ou `timezone`.
- **Resposta resumida:**

```json
{
  "results": [
    {
      "id": 2267057,
      "name": "Lisbon",
      "latitude": 38.71667,
      "longitude": -9.13333,
      "admin1": "Lisbon",
      "country": "Portugal"
    }
  ],
  "generationtime_ms": 0.2
}
```

- **Mapeamento:** `results[i].id → City.id`; `name → name`; `admin1 → region`; `country → country`; `latitude → latitude`; `longitude → longitude`. `admin1` e `country` podem não vir e permanecem opcionais. Não converter nome ou coordenadas; armazenar o resultado selecionado.
- **Sem resultados e erros:** `results: []` representa busca sem correspondência. Campo `results` ausente/malformado, HTTP não-2xx, erro JSON ou timeout são erro de serviço, não lista vazia.

### Forecast — Open-Meteo

- **URL:** `GET https://api.open-meteo.com/v1/forecast?latitude=38.71667&longitude=-9.13333&current=temperature_2m%2Cweather_code&daily=weather_code%2Ctemperature_2m_min%2Ctemperature_2m_max&temperature_unit=celsius&timezone=auto&forecast_days=5`.
- **Parâmetros:** `latitude` e `longitude` vêm do `City` selecionado; `current=temperature_2m,weather_code` pede condições atuais; `daily=weather_code,temperature_2m_min,temperature_2m_max` pede os campos diários; `temperature_unit=celsius` fixa a unidade interna; `timezone=auto` pede o timezone das coordenadas; `forecast_days=5` pede hoje e os quatro dias seguintes.
- **Resposta resumida:**

```json
{
  "latitude": 38.7167,
  "longitude": -9.1333,
  "timezone": "Europe/Lisbon",
  "current": {
    "time": "2026-10-29T12:00",
    "temperature_2m": 18.4,
    "weather_code": 1
  },
  "daily": {
    "time": ["2026-10-29", "2026-10-30", "2026-10-31", "2026-11-01", "2026-11-02"],
    "weather_code": [1, 3, 61, 2, 0],
    "temperature_2m_min": [13.2, 14.0, 15.1, 12.8, 12.0],
    "temperature_2m_max": [21.5, 20.0, 18.7, 19.2, 20.1]
  }
}
```

- **Mapeamento para `WeatherData`:** associar o `City` selecionado a `city`; `timezone → timezone`; `current.temperature_2m → current.temperatureC`; `current.weather_code → current.weatherCode`; `current.time → current.observedAt`. Para cada índice `i`, criar um `ForecastDay`: `daily.time[i] → date`, `daily.weather_code[i] → weatherCode`, `daily.temperature_2m_min[i] → minimumC` e `daily.temperature_2m_max[i] → maximumC`.
- **Validação/normalização:** arrays diários são alinhados pelo índice, não por ordenação independente. Resposta completa gera cinco dias. Se um campo ou data estiver ausente, normalizar o dado ausente para `null` e manter as cinco datas esperadas; não extrapolar nem inventar valores. Validar `timezone`, tipos numéricos e `current` antes de produzir `WeatherData`.
- Ambas as APIs são chamadas diretamente pelo browser, sem API key. Cobertura, limites de uso e termos devem ser verificados para a geografia de lançamento antes da publicação.

## State Management

Usar estado React local, sem Redux, Context global ou cache de servidor. `App` chama `useWeather` e passa valores/callbacks aos componentes; `SearchBar` mantém apenas o texto ainda não submetido. O hook é a única fonte de verdade para busca e clima:

- `searchState`: `idle` antes da primeira busca; `loading` durante geocoding; `success` com `City[]` não vazia; `empty` quando geocoding retorna lista vazia; `error` para falha de rede, HTTP, timeout ou resposta inválida.
- `weatherState`: `idle` sem cidade selecionada; `loading` enquanto consulta a cidade ativa; `success` com `WeatherData`, inclusive resposta parcial com campos `null`; `empty` quando a resposta válida não contém nenhum dado meteorológico utilizável; `error` para falha de rede, HTTP, timeout ou resposta inválida.
- `selectedCity` e `unit` (`Unit`) pertencem ao hook. A unidade começa como `celsius`, permanece em memória enquanto a página está aberta e não é persistida.
- `AbortController` por operação cancela chamadas substituídas; um identificador incremental descarta respostas antigas mesmo se o cancelamento não impedir que cheguem.

Ao iniciar uma busca, limpar resultados anteriores e mudar `searchState` para `loading`. Ao selecionar uma cidade, definir a cidade ativa e mudar `weatherState` para `loading`; não exibir snapshot da cidade anterior como se fosse da nova. Retry manual repete a operação correspondente, preservando termo ou cidade.

As temperaturas normalizadas permanecem em Celsius no `WeatherData`. A apresentação deriva cada valor durante o render: Celsius usa o valor base; Fahrenheit usa `C × 9/5 + 32`, arredondado a uma casa decimal com empate afastando de zero. `setUnit` altera apenas o estado React `unit`; não chama `weatherService`, não invalida `WeatherData` e não faz novo request. Recarregar restaura Celsius conforme FR5.

## Error Handling

- **Input inválido:** exibir mensagem pt-BR junto ao campo e manter foco; não chamar serviço nem alterar `searchState` para `loading`.
- **Geocoding vazio:** definir `searchState = empty`, preservar o termo editável e não consultar forecast.
- **Falha de rede:** classificar rejeição de `fetch` (exceto cancelamento intencional) como `network`; exibir mensagem genérica e retry manual. Não expor detalhes internos.
- **Erro HTTP/API:** tratar status não-2xx como `http`; 429 e 5xx são retryable manualmente, enquanto erros 4xx de requisição não são. Não interpretar resposta de erro do provedor como resultado vazio nem exibir seu corpo bruto.
- **Timeout:** usar timeout de 10 s em cada chamada de geocoding ou forecast. Classificar abort causado pelo timer como `timeout`; terminar loading, apresentar retry manual e descartar resposta tardia. Abort causado por nova busca/seleção é silencioso e não deve mostrar erro.
- **JSON ou estrutura inválida:** erro de parse, tipos inesperados ou arrays incompatíveis tornam a operação `error` com código `invalid-response`; nunca substituir por `empty`.
- **Resposta parcial válida:** manter `weatherState = success`; preencher campos ausentes com `null`, preservar a data diária e marcar cada campo/seção indisponível. Não interpolar, copiar de outro dia ou tratar dado ausente como zero. Se a resposta válida não trouxer nenhum dado meteorológico utilizável, usar `weatherState = empty`.
- **Retry:** sem tentativas automáticas. Retry de geocoding reutiliza o último termo válido; retry meteorológico reutiliza a `selectedCity`. Ao sucesso, limpar erro e atualizar estado/dados.
- **Concorrência:** abortar operação substituída e conferir o identificador antes de atualizar estado. Resposta de cidade ou termo antigo nunca deve substituir dados da operação mais recente.

## Testing Strategy

### Vitest e Testing Library

- **Funções puras (`lib`):** testar trim e limites Unicode (0, 1, 100 e 101 caracteres), caracteres especiais, conversão C↔F, arredondamento de empate positivo/negativo, formatação pt-BR, códigos WMO conhecidos/desconhecidos e datas civis sem deslocamento UTC.
- **Normalizadores (`lib/openMeteoMapper`):** testar payload geocoding/forecast válido, arrays desalinhados, campos opcionais ausentes, payload parcial, JSON com tipo inesperado e ausência de dados utilizáveis. Asserir que `null` representa ausência e que nenhum valor é interpolado.
- **Serviços (`services`, `fetch` mockado):** verificar endpoint e parâmetros de geocoding; coordenadas, `current`, `daily`, Celsius, timezone e cinco dias no forecast; respostas vazias; HTTP 4xx/429/5xx; falha de rede; JSON inválido; timeout aos 10 s; cancelamento intencional e resposta tardia. Asserir os códigos `WeatherError` e `retryable` corretos.
- **Hook (`useWeather`, serviço mockado):** verificar transições `idle → loading → success/empty/error`, seleção de cidade, retry, unidade sem nova chamada ao serviço, erro limpo após sucesso e descarte de resposta fora de ordem.
- **Componentes (Testing Library + `user-event`):** renderizar loading, erro, vazio e sucesso de forma independente; verificar busca/seleção, input inválido sem rede, troca de unidade sem nova consulta, retry, foco, teclado e anúncio acessível de status/erro. Componentes recebem props/callbacks e não exigem mock de `fetch`.

### Playwright

- Interceptar Open-Meteo com `page.route`; E2E não depende de disponibilidade ou dados ao vivo do provedor.
- Cobrir o fluxo principal: submeter busca, escolher cidade, verificar clima atual e cinco datas diárias, alternar unidade e confirmar que a troca não dispara nova requisição.
- Cobrir cidade homônima com escolha explícita, busca sem resultados, input vazio, erro de API, timeout, retry bem-sucedido e resposta parcial sem valores inventados.
- Executar os fluxos principais nos viewports de 320 × 720, 390 × 844, 768 × 1024 e 1280 × 800 CSS px; verificar ausência de rolagem horizontal e controles utilizáveis em mobile.
- Executar a matriz de browsers/dispositivos de NFR2 no CI. Rodar axe nos estados inicial, resultados e erro; complementar com revisão manual de teclado/leitor de tela conforme NFR3.
- Manter testes com chamadas reais à Open-Meteo fora dos testes de aceitação determinísticos; usar smoke test sintético separado para disponibilidade e dependência externa.

### Critérios de manutenção

- Cada caso automatizado deve referenciar o AC/US que cobre.
- Preferir asserts de comportamento observável e parâmetros/contratos públicos; evitar asserts de estrutura interna de componentes ou implementação do hook.
- CI deve executar unitários, componentes, E2E mockado, lint e build. Desempenho, disponibilidade do provedor e validação manual de acessibilidade são gates separados descritos nos NFRs.

## Risks & Trade-offs

| Tema | Decisão do plano | Alternativa considerada | Trade-off |
| --- | --- | --- | --- |
| Transporte HTTP | Usar `fetch`, `URLSearchParams` e `AbortController` nativos. | Axios ou Ky. | Evita dependência e abstração para duas APIs; exige padronizar manualmente parsing, timeout e erros no serviço. |
| Estado e dados remotos | Estado local em `useWeather`, sem store global. | Redux/Zustand ou TanStack Query. | Menos conceitos e configuração para um único fluxo; não oferece cache, deduplicação global nem sincronização entre telas, que estão fora da V1. |
| Integração meteorológica | Browser chama Open-Meteo diretamente, sem backend/proxy. | Backend intermediário ou proxy serverless. | Mantém deploy simples e sem API key; disponibilidade, CORS, termos e limites ficam ligados ao provedor e precisam de validação antes do lançamento. |
| Cache e persistência | Não guardar resposta nem preferência após recarga. | Cache HTTP/app, IndexedDB ou localStorage. | Reduz retenção e evita apresentar dados obsoletos; offline e preferência persistente não são suportados. |
| Unidade interna | Normalizar e manter temperaturas em Celsius; derivar Fahrenheit na apresentação. | Solicitar ao provedor a unidade selecionada ou converter e armazenar ambas. | Fonte única evita deriva e troca de unidade não faz request; exige testes da fórmula e do arredondamento. |
| Testes E2E | Interceptar APIs com `page.route`; medir o provedor separadamente. | Chamar Open-Meteo real em cada E2E. | Testes determinísticos e rápidos; smoke separado é necessário para detectar indisponibilidade ou mudança real do contrato externo. |
| Descrição meteorológica | Mapear códigos WMO localmente para pt-BR. | Serviço externo de tradução/descrição. | Sem dependência ou latência adicional; o mapeamento local precisa de manutenção e fallback para códigos desconhecidos. |