# Ruby — get started

Ruby 3.0 or newer. Uses `net/http` and `json` from the standard library. No gem install.

Contract and field list: [api.md](api.md).

## Use cases

- Daily monitor: `runMode` `monitor` and the same `historyStoreName`.
- Agency: `placeItems` with `label`, `clientName`, and `tags`.
- Your app reads `priority_action` rows after the run.
- Cap spend with `maxBudgetUsd` and `maxPlacesPerRun`. A successful place is USD 0.10.

## Run

Save as `monitor.rb`.

```bash
APIFY_TOKEN=apify_api_xxx ruby monitor.rb
```

```ruby
require 'json'
require 'net/http'
require 'uri'

token = ENV.fetch('APIFY_TOKEN')
actor = 'basta_compilar~google-maps-review-change-feed'
input = {
  placeItems: [
    {
      place: 'https://www.google.com/maps/search/Ifome+Burguer+Hortol%C3%A2ndia',
      label: 'Hortolândia',
      clientName: 'Demo Brand',
      tags: ['brasil']
    }
  ],
  runMode: 'monitor',
  locale: 'pt-BR',
  mapsHost: 'www.google.com.br',
  sortBy: 'newest',
  reviewSource: 'auto',
  maxReviewsPerPlace: 20,
  maxPlacesPerRun: 50,
  maxBudgetUsd: 5,
  historyStoreName: 'google-maps-review-history',
  includeFlatExport: true,
  includeRunDigest: true,
  notifyOn: 'priority_actions',
  minPriorityToNotify: 'high',
  emailMode: 'digest'
}

def apify(method, path, token, body = nil)
  uri = URI("https://api.apify.com/v2#{path}")
  request = method == :post ? Net::HTTP::Post.new(uri) : Net::HTTP::Get.new(uri)
  request['Authorization'] = "Bearer #{token}"
  request['Content-Type'] = 'application/json'
  request.body = JSON.generate(body) if body
  response = Net::HTTP.start(uri.host, uri.port, use_ssl: true, read_timeout: 60) do |http|
    http.request(request)
  end
  raise "#{response.code} #{response.body}" unless response.is_a?(Net::HTTPSuccess)

  JSON.parse(response.body)
end

started = apify(:post, "/acts/#{actor}/runs", token, input)
run_id = started.dig('data', 'id')
dataset_id = started.dig('data', 'defaultDatasetId')
status = started.dig('data', 'status')
terminal = %w[SUCCEEDED FAILED ABORTED TIMED-OUT]

until terminal.include?(status)
  sleep 5
  status = apify(:get, "/actor-runs/#{run_id}", token).dig('data', 'status')
end

items = apify(:get, "/datasets/#{dataset_id}/items?format=json&clean=true", token)
actions = items.select { |item| item['recordType'] == 'priority_action' }
puts JSON.pretty_generate({ runId: run_id, status: status, actions: actions })
exit 1 unless status == 'SUCCEEDED'
```

Keep `historyStoreName` stable for that portfolio. The first successful run is the baseline.

Full field list: [INPUT.en.md](../docs/INPUT.en.md). Pricing: [PRICING.en.md](../docs/PRICING.en.md).
