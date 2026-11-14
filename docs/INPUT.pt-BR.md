# Guia de input (PT-BR)

🇬🇧 English: [INPUT.en.md](./INPUT.en.md)

Este documento explica **cada campo** e **cada grupo** de entrada do Actor, para desenvolvedores, agências e empresas.

**Regra prática:** informe pelo menos um local em `places` e/ou `placeItems`. O resto tem default seguro.

---

## Visão rápida dos grupos

| Grupo | Campos | Para quê |
| --- | --- | --- |
| Locais | `places`, `placeItems` | Quais estabelecimentos monitorar |
| Modo | `runMode` | Monitorar, importar histórico ou resetar baseline |
| Orçamento | `maxBudgetUsd`, `maxPlacesPerRun` | Limitar cobrança PPE nesta run |
| Coleta | `maxReviewsPerPlace`, `sortBy`, `reviewSource`, `locale`, `mapsHost` | Como / quanto coletar no Maps |
| Filtros | `minRating`, `maxRating`, `languages`, `onlyWithText`, `onlyWithoutOwnerResponse`, `publishedWithinDays` | Recortar o que entra no diff |
| Prioridade & histórico | `negativeRatingThreshold`, `maxPriorityActionsPerPlace`, `emitBaselineAsNew`, `trackPossibleRemovals`, `consecutiveMissThreshold`, `dedupeEmittedChanges` | Como priorizar e detectar remoções |
| Saída | `includeUnchangedReviews`, `includeFlatExport`, `includeRunDigest` | O que grava no Dataset |
| Persistência | `historyStoreName` | Memória entre agendas |
| Confiabilidade | `maxConcurrency`, `delayBetweenPlacesMs`, `navigationTimeoutSecs`, `maxRetriesPerPlace`, `failRunOnAnyPlaceError` | Estabilidade / custo de plataforma |
| Chaves do cliente | `customerGoogleApiKey` | API Google do **cliente** (nunca do desenvolvedor do Actor) |
| Notificações | `webhookUrls`, `alertEmails`, `emailMode`, `notifyOn`, `minPriorityToNotify`, `webhookSlackCompatible` | Slack/API e e-mail |
| Proxy / debug | `proxyConfiguration`, `debug` | Proxy Apify do usuário e logs |

---

## 1. Locais a monitorar

### `places` (lista de strings)

- **O quê:** URLs do Google Maps ou Place IDs (`ChIJ…`).
- **Quando usar:** lista simples, sem metadados.
- **Exemplos válidos:**
  - `ChIJJ3SpfQsZTIYRIcqOUQVKF1M`
  - `https://www.google.com/maps/place/?q=place_id:ChIJ…`
  - URL completa de um estabelecimento no Maps
- **Obrigatório?** Não sozinho — mas precisa haver **pelo menos um** item em `places` **ou** `placeItems`.

### `placeItems` (lista de objetos)

Para agências / multi-cliente. Cada item:

| Subcampo | Obrigatório | Função |
| --- | --- | --- |
| `place` | sim | URL ou Place ID |
| `label` | não | Nome interno da unidade (ex.: “Campinas Centro”) |
| `clientName` | não | Nome do cliente/conta (aparece em Dataset, webhook e e-mail) |
| `tags` | não | Tags para relatório (ex.: `sp`, `flagship`) |

`places` e `placeItems` são **mesclados**. Duplicatas do mesmo local são tratadas pelo Place ID / chave interna.

---

## 2. Modo de execução (`runMode`)

| Valor | Use quando | Profundidade | Agenda frequente? |
| --- | --- | --- | --- |
| `monitor` (**padrão**) | Feed diário/semanal de mudanças | limitada (máx. 40 reviews) | **Sim** |
| `import_history` | Importação pontual mais profunda | até 100 reviews | **Não** |
| `reset_baseline` | Apagar histórico dos locais e recriar baseline | caps de monitor | Só sob demanda |
| `export_snapshot` | Alias de `import_history` (compatibilidade) | até 100 | **Não** |

**Primeira run em `monitor`:** cria baseline (não flooda “review nova”). Negativas sem resposta ainda podem virar ação prioritária.

---

## 3. Orçamento (PPE)

Só o evento `place-monitored` é cobrado (local **verificado com sucesso**). Mudanças, ações, webhook e e-mail estão **inclusos**.

### `maxBudgetUsd`

- Teto soft em dólares desta run (planejado a **US$ 0,10** por local com sucesso).
- Ex.: `5` ≈ até ~50 locais cobrados.
- Também configure o **max total charge** da run na Apify.

### `maxPlacesPerRun`

- Teto duro de cobranças com sucesso nesta run.
- Prefill sugerido: `50`.
- Se **os dois** (`maxBudgetUsd` e `maxPlacesPerRun`) ficarem vazios, o Actor aplica teto de segurança de **50**.

Falhas de coleta **não** geram PPE.

---

## 4. Coleta

### `maxReviewsPerPlace`

- Quantas reviews públicas mais recentes analisar por local.
- Default: **20**.
- Cap: **40** em `monitor` / `reset_baseline`; **100** em `import_history`.

### `sortBy`

Ordem das avaliações solicitada antes de coletar:

| Valor | Significado |
| --- | --- |
| `newest` (padrão) | Mais recentes — ideal para monitoramento |
| `relevant` | Mais relevantes |
| `highest` | Melhores notas |
| `lowest` | Piores notas |

### `reviewSource`

| Valor | Significado |
| --- | --- |
| `auto` (padrão) | Lê a **lista pública de avaliações da Busca do Google** do local (ordenável, paginada, mesmos IDs de avaliação do Maps). Se a Busca não abrir, usa o painel do Maps. |
| `maps_panel` | Somente o painel do Google Maps (comportamento antigo). |

**Por quê:** sessões anônimas do Google Maps recebem uma *visualização limitada* — cerca de 5 avaliações "mais relevantes", e tanto *Ordenar* quanto *Ver mais avaliações* abrem uma tela de login. Um monitor `newest` usando só o painel do Maps não consegue ver as avaliações mais recentes.

Quando a amostra está incompleta (`limited_public_view` ou `sort_not_applied`), a execução continua com sucesso, mas:

- os contadores de possível remoção **não** avançam (sem `possible_removal` falso);
- avaliações nunca vistas publicadas antes da base são armazenadas sem gerar `new`;
- `place_summary.summary.collection` informa `source`, `issues`, `sampleIncomplete` e `removalTracking`, e o e-mail explica o ocorrido.

### `locale`

Locale do browser **e idioma da interface do Google** (enviado como `hl=`; sem isso o Google escolhe o idioma pelo IP). Ex.: `pt-BR`, `en-US`. Também define o idioma do template de e-mail (`pt*` → português).

### `mapsHost`

Host ao montar URL a partir de Place ID. Ex.: `www.google.com.br` para Brasil, `www.google.com` global.

---

## 5. Filtros (após a coleta)

Aplicados **depois** de raspar. Não mudam o que o Maps mostra; só o que entra no diff / ações.

| Campo | Função | Default |
| --- | --- | --- |
| `minRating` | Mantém só nota ≥ valor | off |
| `maxRating` | Mantém só nota ≤ valor | off |
| `languages` | Prefixos BCP-47 (`pt`, `en`…). Review sem idioma conhecido **é mantida** | `[]` |
| `onlyWithText` | Descarta review só com estrela, sem texto | `false` |
| `onlyWithoutOwnerResponse` | Só reviews sem resposta do dono | `false` |
| `publishedWithinDays` | Janela aproximada de publicação; data desconhecida **é mantida** | off |

---

## 6. Ações prioritárias e histórico

### `negativeRatingThreshold` (default `2`)

Notas ≤ este valor contam como “negativa” na fila de prioridade.

### `maxPriorityActionsPerPlace` (default `50`)

Limite de registros `priority_action` por estabelecimento nesta run.

### `emitBaselineAsNew` (default `false`)

- `false`: primeira coleta só cria histórico (recomendado).
- `true`: trata baseline como se fossem reviews “new” (pode gerar flood).

### `trackPossibleRemovals` (default `true`)

Se uma review conhecida sumir do painel em várias coletas **com sucesso**, emite `possible_removal` (não apaga o histórico).

### `consecutiveMissThreshold` (default `3`)

Quantas coletas bem-sucedidas seguidas sem ver a review antes de `possible_removal`.

### `dedupeEmittedChanges` (default `true`)

Evita reemitir a mesma mudança (mesmo fingerprint) no Dataset.

---

## 7. Saída no Dataset

| Campo | Default | Efeito |
| --- | --- | --- |
| `includeUnchangedReviews` | `false` | Se `true`, grava também reviews que não mudaram |
| `includeFlatExport` | `true` | Linhas `flat_export` (CSV/Sheets) |
| `includeRunDigest` | `true` | Um `run_digest` com totais da run |

Tipos comuns de registro: `priority_action`, `change`, `place_summary`, `flat_export`, `run_digest`, `place_error`.

---

## 8. Persistência e agenda

### `historyStoreName` (default `google-maps-review-history`)

Nome do Key-Value Store da Apify com o histórico entre runs.

**Importante:** use o **mesmo nome** em todas as runs agendadas do mesmo conjunto de locais. Trocar o nome = “amnésia” (nova baseline).

---

## 9. Confiabilidade (custo / estabilidade)

| Campo | Default | Notas |
| --- | --- | --- |
| `maxConcurrency` | `1` | Locais em paralelo (máx. 3). Prefira 1 sem proxy residencial |
| `delayBetweenPlacesMs` | `1500` | Pausa entre locais quando concorrência = 1 |
| `navigationTimeoutSecs` | `35` | Timeout de carregamento da página (máx. 60) |
| `maxRetriesPerPlace` | `0` | Tentativas extras após falha (máx. 1). Não gera PPE, mas gasta infra |
| `failRunOnAnyPlaceError` | `false` | `false` = falha isolada por local; `true` = run inteira falha |

---

## 10. Chaves do cliente

### `customerGoogleApiKey` (opcional, secreto)

Chave **do cliente** (Places API). Se o scrape não trouxer nome/endereço, o Actor pode enriquecer via Places Details com **essa** chave. Nunca usa chave do desenvolvedor do Actor. Proxy também é do usuário (`proxyConfiguration`).

---

## 11. Notificações

Webhook e e-mail usam os **mesmos** filtros `notifyOn` / `minPriorityToNotify`. Inclusos no preço `place-monitored`.

### `webhookUrls`

Lista de URLs HTTP que recebem JSON (Slack Incoming Webhook, Make, Zapier, n8n, API própria).

### `alertEmails`

Destinatários HTML/texto. Envio direto via Gmail (padrão, sem domínio), SMTP personalizado ou Resend. Configure segredos no ambiente do Actor. Veja [configuração de e-mail](./EMAILS.pt-BR.md).

### `emailMode`

| Valor | Comportamento |
| --- | --- |
| `digest` (padrão) | 1 e-mail no fim da run, seção por estabelecimento |
| `per_place` | 1 e-mail por local com alerta |

### `notifyOn`

| Valor | Envia |
| --- | --- |
| `none` | Nada |
| `priority_actions` (padrão) | Só ações prioritárias |
| `changes` | Só mudanças |
| `priority_and_changes` | Ambos |
| `all` | Sempre (mesmo sem itens) |

### `minPriorityToNotify` (default `high`)

Ignora ações abaixo deste nível: `critical` → `high` → `medium` → `low`.

### `webhookSlackCompatible` (default `false`)

Se `true`, inclui campos `text` / `blocks` úteis para Slack.

Detalhes: [WEBHOOKS.pt-BR.md](./WEBHOOKS.pt-BR.md) · [EMAILS.pt-BR.md](./EMAILS.pt-BR.md)

---

## 12. Proxy e debug

### `proxyConfiguration`

Editor de proxy da Apify (grupos, país, etc.). Recomendado em listas grandes ou agenda frequente. Custo de proxy é do **usuário**.

### `debug` (default `false`)

Logs verbosos de scrape/diff. Use só para diagnóstico.

---

## Exemplo completo (Brasil, diário)

```json
{
  "placeItems": [
    {
      "place": "ChIJxxxxxxxxxxxxxxxxxxxxxx",
      "label": "Campinas Centro",
      "clientName": "Padaria Aurora",
      "tags": ["brasil", "sp"]
    }
  ],
  "runMode": "monitor",
  "maxReviewsPerPlace": 20,
  "maxPlacesPerRun": 50,
  "maxBudgetUsd": 5,
  "sortBy": "newest",
  "locale": "pt-BR",
  "mapsHost": "www.google.com.br",
  "negativeRatingThreshold": 2,
  "historyStoreName": "aurora-maps-history",
  "includeFlatExport": true,
  "alertEmails": ["reputacao@padariaaurora.com.br"],
  "emailMode": "digest",
  "webhookUrls": ["https://hooks.slack.com/services/XXX/YYY/ZZZ"],
  "webhookSlackCompatible": true,
  "notifyOn": "priority_actions",
  "minPriorityToNotify": "high",
  "maxConcurrency": 1,
  "maxRetriesPerPlace": 0
}
```

---

## Relacionados

- [Produto](./PRODUCT.pt-BR.md)
- [Precificação](./PRICING.pt-BR.md)
- [Webhooks](./WEBHOOKS.pt-BR.md)
- [E-mail](./EMAILS.pt-BR.md)
