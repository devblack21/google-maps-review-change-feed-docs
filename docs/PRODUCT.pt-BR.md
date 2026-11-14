# Google Maps Review Change Feed — Guia do Produto (PT-BR)

**Seu radar de reputação no Google Maps.**  
Monitora vários estabelecimentos. Mostra o que mudou. Diz o que responder primeiro.

🇬🇧 English version: [PRODUCT.en.md](./PRODUCT.en.md)

---

## O problema (bem direto)

Se você cuida de **restaurantes, clínicas, lojas, barbearias, hotéis, franquias ou clientes de SEO local**, as avaliações do Google Maps não param.

Abrir o Maps um por um todo dia **não escala**.

O que você precisa de verdade é:

> “O que mudou desde ontem — e o que eu preciso responder agora?”

É exatamente isso que este Actor entrega.

---

## O que o produto faz

1. Coleta **avaliações públicas do Google Maps** dos seus locais (qualquer país)
2. Guarda um **histórico** entre execuções agendadas
3. Detecta **reviews novas** e **alterações** (nota, texto, resposta do dono, fotos)
4. Monta **ações prioritárias** (principalmente negativas sem resposta)
5. Grava resultados estruturados no **Dataset** do Apify
6. Opcionalmente avisa no **Slack / Make / Zapier / sua API** via webhooks

Não é só um scraper de reviews.  
É um **feed de mudanças + fila de ação** para times de reputação.

---

## Para quem é

### Brasil
- Agências de marketing local / SEO local
- Redes de restaurantes, clínicas, academias, varejo
- Franquias com dezenas de unidades
- Operações que precisam responder rápido no Google

### Internacional
- Marcas multi-local (EUA, Europa, LATAM, APAC)
- Redes de franquia
- Agências de reputation management
- Times de experiência do cliente

---

## Onde o aviso aparece

| Canal | O que você recebe |
| --- | --- |
| **Dataset do Apify** | Ações prioritárias, mudanças, resumos por local, linhas CSV |
| **RUN_SUMMARY** | Totais e saúde da execução |
| **Webhooks** | Push para Slack / automações quando algo precisa de atenção |
| **Agendamento** | Monitoramento automático diário/semanal |

---

## Exemplos reais

### Exemplo A — Rede de padarias (Campinas + São Paulo)

**Input**

```json
{
  "placeItems": [
    {
      "place": "ChIJxxxxxxxxxxxxxxxxxxxxxx",
      "label": "Campinas Centro",
      "clientName": "Padaria Aurora",
      "tags": ["sp", "interior"]
    },
    {
      "place": "https://www.google.com.br/maps/place/?q=place_id:ChIJyyyyyyyyyyyyyyyyyyyy",
      "label": "Paulista",
      "clientName": "Padaria Aurora",
      "tags": ["sp", "capital"]
    }
  ],
  "runMode": "monitor",
  "maxReviewsPerPlace": 20,
  "maxPlacesPerRun": 50,
  "maxBudgetUsd": 5,
  "negativeRatingThreshold": 2,
  "historyStoreName": "aurora-maps-history",
  "webhookUrls": ["https://hooks.slack.com/services/XXX/YYY/ZZZ"],
  "alertEmails": ["reputacao@padariaaurora.com.br"],
  "emailMode": "digest",
  "notifyOn": "priority_actions",
  "minPriorityToNotify": "high",
  "webhookSlackCompatible": true,
  "locale": "pt-BR",
  "mapsHost": "www.google.com.br"
}
```

**Dia 1 (baseline)**  
O histórico é criado. Negativas sem resposta já viram ações prioritárias.

**Dia 2 (monitor)**  
O Dataset pode mostrar:

```json
{
  "recordType": "priority_action",
  "priority": "critical",
  "actionType": "new_negative",
  "reason": "New 1-star review without owner response",
  "clientName": "Padaria Aurora",
  "placeName": "Padaria Aurora — Paulista",
  "review": {
    "rating": 1,
    "authorName": "Carla",
    "text": "Fila enorme e pão frio",
    "hasOwnerResponse": false,
    "reviewUrl": "https://..."
  }
}
```

O Slack recebe um alerta curto com link do local e o motivo.

### Exemplo B — Clínica odontológica multi-unidades (EUA)

```json
{
  "placeItems": [
    {
      "place": "ChIJzzzzzzzzzzzzzzzzzzzz",
      "label": "Miami Lakes",
      "clientName": "BrightSmile Dental",
      "tags": ["usa", "fl"]
    }
  ],
  "locale": "en-US",
  "mapsHost": "www.google.com",
  "sortBy": "newest",
  "notifyOn": "priority_and_changes"
}
```

Alertas úteis:
- novas reviews 1–2 estrelas
- queda de nota
- resposta do dono apareceu (caso resolvido)
- possível remoção após várias ausências (investigar — nunca assumir exclusão)

### Exemplo C — Agência com 40 clientes

Use `clientName` + `label` + `tags` para cada linha ficar pronta para CRM:

- filtrar por cliente no Sheets
- rotear alertas Slack por canal
- exportar só `priority_action` para a reunião da manhã

---

## Por que o valor é alto

| Sem o Actor | Com o Actor |
| --- | --- |
| Conferir Maps na mão | Agenda automática |
| Review ruim passa batido | Fila de prioridade |
| Sem memória entre dias | Histórico durável |
| Dump bruto de reviews | Feed acionável de mudanças |
| Achismo | Evidência + links + motivo |

Uma review 1 estrela sem resposta pode custar agenda.  
Pegar no mesmo dia é o produto.

---

## Princípios de confiança

- Sumir de uma coleta **≠ excluída**
- `possible_removal` só depois de várias ausências
- Baseline não inundar com “tudo é novo”
- Falha em um local não derruba a run inteira

---

## Ritmo sugerido de operação

1. Primeira run → baseline  
2. Agendar diário no horário local da manhã  
3. Manter o mesmo `historyStoreName`  
4. Mandar `priority_actions` para o Slack  
5. Exportar `flat_export` semanalmente para relatório

---

## Realidade de preço (Apify Store)

**Pay per event + usage cobrado do usuário** (toggle da Console obrigatório). Único evento custom pago: `place-monitored` a **US$ 0,10** por verificação com sucesso. Mudanças, ações, webhooks e e-mail inclusos. Falhas/retries grátis. Teto de segurança 50 locais/run sem orçamento.

Frequência (30 locais, só eventos do Actor): **1×/dia → US$ 90 / 30 dias**; **4×/dia → US$ 360**, mais o uso da plataforma Apify na sua conta. Detalhes: [PRICING.pt-BR.md](./PRICING.pt-BR.md).

Detalhes: [PRICING.pt-BR.md](./PRICING.pt-BR.md)

---

## Docs relacionados

- [Como usar + Agenda (PT)](./USAGE.pt-BR.md)
- [Guia de campos de input (PT)](./INPUT.pt-BR.md)
- [Precificação PPE (PT)](./PRICING.pt-BR.md)
- [Guia de Webhooks (PT)](./WEBHOOKS.pt-BR.md)
- [Alertas por e-mail (PT)](./EMAILS.pt-BR.md)
- English: [PRODUCT.en.md](./PRODUCT.en.md) · [USAGE.en.md](./USAGE.en.md)
