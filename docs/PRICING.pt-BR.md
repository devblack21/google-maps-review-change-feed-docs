# Preço (PT-BR)

🇬🇧 English: [PRICING.en.md](./PRICING.en.md)

Você paga **por local verificado com sucesso**, mais o uso normal da plataforma Apify na sua conta.

## O que o Actor cobra

| Evento | Quando cobra | Preço |
| --- | --- | --- |
| `place-monitored` | Cada local **verificado com sucesso** numa execução | **USD 0,10** |

Incluso nesse preço: mudanças detectadas, ações prioritárias, webhook e e-mail.

Não cobra como evento do Actor:

- local em que a coleta falhou
- novas tentativas
- reenvio de webhook ou e-mail
- locais pulados porque `maxBudgetUsd` ou `maxPlacesPerRun` estourou

Um dia sem avaliação nova ainda cobra **USD 0,10** por local que foi verificado com sucesso. A verificação é o produto.

## O que a Apify cobra além do evento

Compute, proxy e storage são uso da plataforma na **sua** conta. São separados dos USD 0,10.

## Exemplos (só eventos do Actor, 30 dias)

| Cenário | Total aproximado |
| --- | --- |
| 10 locais × 1 vez por dia | ~USD 30 |
| 30 locais × 1 vez por dia | ~USD 90 |
| 30 locais × 4 vezes por dia | ~USD 360 |

Limite uma execução com `maxBudgetUsd` e `maxPlacesPerRun`.

## Relacionado

- [Como usar](./USAGE.pt-BR.md)
- [Campos](./INPUT.pt-BR.md)
