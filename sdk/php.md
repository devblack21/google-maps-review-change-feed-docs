# PHP — get started

PHP 8.0 or newer, with OpenSSL enabled (the usual build). Uses `file_get_contents` and `json_decode`. No Composer package.

Contract and field list: [api.md](api.md).

## Use cases

- Daily monitor: `runMode` `monitor` and the same `historyStoreName`.
- Agency: `placeItems` with `label`, `clientName`, and `tags`.
- Your app reads `priority_action` rows after the run.
- Cap spend with `maxBudgetUsd` and `maxPlacesPerRun`. A successful place is USD 0.10.

## Run

Save as `monitor.php`.

```bash
APIFY_TOKEN=apify_api_xxx php monitor.php
```

```php
<?php
$token = getenv('APIFY_TOKEN');
if (!$token) {
    fwrite(STDERR, "Set APIFY_TOKEN\n");
    exit(1);
}

$actor = 'basta_compilar~google-maps-review-change-feed';
$input = [
    'placeItems' => [[
        'place' => 'https://www.google.com/maps/search/Ifome+Burguer+Hortol%C3%A2ndia',
        'label' => 'Hortolândia',
        'clientName' => 'Demo Brand',
        'tags' => ['brasil'],
    ]],
    'runMode' => 'monitor',
    'locale' => 'pt-BR',
    'mapsHost' => 'www.google.com.br',
    'sortBy' => 'newest',
    'reviewSource' => 'auto',
    'maxReviewsPerPlace' => 20,
    'maxPlacesPerRun' => 50,
    'maxBudgetUsd' => 5,
    'historyStoreName' => 'google-maps-review-history',
    'includeFlatExport' => true,
    'includeRunDigest' => true,
    'notifyOn' => 'priority_actions',
    'minPriorityToNotify' => 'high',
    'emailMode' => 'digest',
];

function apify(string $method, string $path, string $token, ?array $body = null): array
{
    $headers = "Authorization: Bearer {$token}\r\nAccept: application/json\r\n";
    $content = '';
    if ($body !== null) {
        $headers .= "Content-Type: application/json\r\n";
        $content = json_encode($body, JSON_UNESCAPED_UNICODE);
    }
    $context = stream_context_create([
        'http' => [
            'method' => $method,
            'header' => $headers,
            'content' => $content,
            'ignore_errors' => true,
            'timeout' => 60,
        ],
    ]);
    $raw = file_get_contents('https://api.apify.com/v2' . $path, false, $context);
    $status = 0;
    foreach ($http_response_header ?? [] as $line) {
        if (preg_match('#HTTP/\S+\s+(\d+)#', $line, $m)) {
            $status = (int) $m[1];
        }
    }
    if ($status < 200 || $status >= 300) {
        throw new RuntimeException($status . ' ' . $raw);
    }
    return json_decode($raw, true);
}

$started = apify('POST', "/acts/{$actor}/runs", $token, $input);
$runId = $started['data']['id'];
$datasetId = $started['data']['defaultDatasetId'];
$status = $started['data']['status'];
$terminal = ['SUCCEEDED', 'FAILED', 'ABORTED', 'TIMED-OUT'];

while (!in_array($status, $terminal, true)) {
    sleep(5);
    $status = apify('GET', "/actor-runs/{$runId}", $token)['data']['status'];
}

$items = apify('GET', "/datasets/{$datasetId}/items?format=json&clean=true", $token);
$actions = array_values(array_filter(
    $items,
    fn ($item) => ($item['recordType'] ?? '') === 'priority_action'
));
echo json_encode(['runId' => $runId, 'status' => $status, 'actions' => $actions], JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE), PHP_EOL;
if ($status !== 'SUCCEEDED') {
    exit(1);
}
```

Keep `historyStoreName` stable for that portfolio. The first successful run is the baseline.

Full field list: [INPUT.en.md](../docs/INPUT.en.md). Pricing: [PRICING.en.md](../docs/PRICING.en.md).
