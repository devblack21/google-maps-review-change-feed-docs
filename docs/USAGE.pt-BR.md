# Como usar este Actor (PT-BR)

🇬🇧 English: [USAGE.en.md](./USAGE.en.md)

Guia para **quem usa** o Google Maps Review Change Feed na Apify.

---

## O que o Actor faz (simples)

1. Você passa os estabelecimentos do Google Maps (URLs ou Place IDs).  
2. Clica em **Run** (ou deixa um **agendamento** rodar).  
3. Ele lê as reviews públicas, compara com a última execução e guarda o histórico.  
4. Se algo importante mudou, pode avisar por **webhook** (Slack etc.) e/ou **e-mail**.

Ele **não** fica 24h olhando o Google sozinho. Só verifica quando uma execução acontece.

---

## Passo 1 — Primeira execução (baseline)

1. Abra o Actor no Console da Apify.  
2. Preencha o input (pelo menos um local). Começo recomendado:

```json
{
  "placeItems": [
    {
      "place": "ChIJxxxxxxxxxxxxxxxxxxxxxx",
      "label": "Loja principal",
      "clientName": "Minha Marca"
    }
  ],
  "runMode": "monitor",
  "maxReviewsPerPlace": 20,
  "maxPlacesPerRun": 10,
  "maxBudgetUsd": 1,
  "historyStoreName": "meu-historico-maps-reviews",
  "notifyOn": "priority_actions",
  "minPriorityToNotify": "high",
  "includeFlatExport": true,
  "locale": "pt-BR",
  "mapsHost": "www.google.com.br"
}
```

3. Clique em **Start**.  
4. Espere o sucesso. Veja o **Dataset** e o Key-value store `RUN_SUMMARY`.

**Primeira run = memória.** Ele guarda as reviews atuais. Em geral **não** manda alerta de “tudo é novo”. Negativas sem resposta ainda podem virar ação prioritária.

---

## Passo 2 — Agendar verificação automática (Apify)

Para monitorar sem clicar todo dia:

1. No Console da Apify, abra este Actor (ou salve uma **Task** com o seu input).  
2. Vá em **Schedules** (ou crie um Schedule que rode a Task/Actor).  
3. Escolha a frequência, por exemplo:
   - **Diário** às 08:00 — recomendado para a maioria  
   - A cada 6 horas — só se precisar de alerta mais rápido (custa mais)  
4. Salve o agendamento.

**Crítico:** use o **mesmo** `historyStoreName` em todas as runs agendadas.  
Se mudar o nome, o Actor “esquece” o histórico e começa uma baseline nova.

---

## Passo 3 — Alertas

### Dataset (sempre)
Toda run grava registros estruturados para CSV, Sheets ou CRM.

### Webhook (opcional)
Preencha `webhookUrls` (Slack, Make, Zapier, n8n, sua API).  
Guia: [WEBHOOKS.pt-BR.md](./WEBHOOKS.pt-BR.md)

### E-mail (opcional)
Preencha `alertEmails` e `emailMode` (`digest` recomendado).  
Guia: [EMAILS.pt-BR.md](./EMAILS.pt-BR.md)

### O que dispara notificação?
Controlado por:

| Campo | Significado |
| --- | --- |
| `notifyOn` | `none` · `priority_actions` (padrão) · `changes` · `priority_and_changes` · `all` |
| `minPriorityToNotify` | Ignora ações abaixo desse nível (`high` é um bom padrão) |

Sem evento que bata no filtro → a run conclui, **sem spam** de alerta.

---

## Passo 4 — Como funciona a cobrança (visão do usuário)

| O quê | Custo |
| --- | --- |
| Cada local **verificado com sucesso** na run | **US$ 0,10** (`place-monitored`) |
| Mudanças detectadas, ações, webhook, e-mail | **Incluso** (sem evento extra) |
| Local que falhou / retry | **Sem** cobrança do evento custom |
| Uso de plataforma Apify (compute, proxy, storage) | Cobrado pela Apify na **sua** conta |

Exemplo: 10 locais, 1×/dia → cerca de **US$ 1,00/dia** em eventos do Actor (+ uso de plataforma).

Detalhes: [PRICING.pt-BR.md](./PRICING.pt-BR.md)

---

## Setup recomendado em produção

1. Primeira run manual → baseline  
2. Agenda **diária** de manhã  
3. Mesmo `historyStoreName` sempre para aquele portfólio  
4. `notifyOn`: `priority_actions`  
5. Webhook Slack + e-mail digest  
6. Defina `maxBudgetUsd` / `maxPlacesPerRun` para não estourar gasto  
7. Use Proxy da Apify (sua conta) em listas grandes  

Mais contexto de produto: [PRODUCT.pt-BR.md](./PRODUCT.pt-BR.md)  
Todos os campos: [INPUT.pt-BR.md](./INPUT.pt-BR.md)
