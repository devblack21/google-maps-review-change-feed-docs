# C# — get started

.NET 8 or newer. Uses `HttpClient` and `System.Text.Json` from the shared framework. No NuGet package.

Contract and field list: [api.md](api.md).

## Use cases

- Daily monitor: `runMode` `monitor` and the same `historyStoreName`.
- Agency: `placeItems` with `label`, `clientName`, and `tags`.
- Your app reads `priority_action` rows after the run.
- Cap spend with `maxBudgetUsd` and `maxPlacesPerRun`. A successful place is USD 0.10.

## Run

Save as `Program.cs` in an empty folder.

```bash
dotnet new console --name ReviewFeed --framework net8.0
cp Program.cs ReviewFeed/Program.cs
cd ReviewFeed
APIFY_TOKEN=apify_api_xxx dotnet run
```

Replace the generated `Program.cs` with:

```csharp
using System.Net.Http.Headers;
using System.Text;
using System.Text.Json;

var token = Environment.GetEnvironmentVariable("APIFY_TOKEN")
    ?? throw new InvalidOperationException("Set APIFY_TOKEN");
const string actor = "basta_compilar~google-maps-review-change-feed";
var input = new
{
    placeItems = new[]
    {
        new
        {
            place = "https://www.google.com/maps/search/Ifome+Burguer+Hortol%C3%A2ndia",
            label = "Hortolândia",
            clientName = "Demo Brand",
            tags = new[] { "brasil" },
        },
    },
    runMode = "monitor",
    locale = "pt-BR",
    mapsHost = "www.google.com.br",
    sortBy = "newest",
    reviewSource = "auto",
    maxReviewsPerPlace = 20,
    maxPlacesPerRun = 50,
    maxBudgetUsd = 5,
    historyStoreName = "google-maps-review-history",
    includeFlatExport = true,
    includeRunDigest = true,
    notifyOn = "priority_actions",
    minPriorityToNotify = "high",
    emailMode = "digest",
};

using var http = new HttpClient { BaseAddress = new Uri("https://api.apify.com/v2/") };
http.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", token);

async Task<JsonDocument> Apify(HttpMethod method, string path, object? body = null)
{
    using var request = new HttpRequestMessage(method, path);
    if (body is not null)
    {
        request.Content = new StringContent(JsonSerializer.Serialize(body), Encoding.UTF8, "application/json");
    }
    using var response = await http.SendAsync(request);
    var text = await response.Content.ReadAsStringAsync();
    if (!response.IsSuccessStatusCode) throw new InvalidOperationException($"{(int)response.StatusCode} {text}");
    return JsonDocument.Parse(text);
}

using var started = await Apify(HttpMethod.Post, $"acts/{actor}/runs", input);
var runId = started.RootElement.GetProperty("data").GetProperty("id").GetString();
var datasetId = started.RootElement.GetProperty("data").GetProperty("defaultDatasetId").GetString();
var status = started.RootElement.GetProperty("data").GetProperty("status").GetString();
var terminal = new HashSet<string> { "SUCCEEDED", "FAILED", "ABORTED", "TIMED-OUT" };

for (var i = 0; i < 120 && !terminal.Contains(status!); i++)
{
    await Task.Delay(TimeSpan.FromSeconds(5));
    using var polled = await Apify(HttpMethod.Get, $"actor-runs/{runId}");
    status = polled.RootElement.GetProperty("data").GetProperty("status").GetString();
}

using var items = await Apify(HttpMethod.Get, $"datasets/{datasetId}/items?format=json&clean=true");
var actions = items.RootElement.EnumerateArray()
    .Where(item => item.GetProperty("recordType").GetString() == "priority_action")
    .Select(item => item.GetRawText());
Console.WriteLine(JsonSerializer.Serialize(new { runId, status, actions }));
if (status != "SUCCEEDED") Environment.ExitCode = 1;
```

Keep `historyStoreName` stable for that portfolio. The first successful run is the baseline.

Full field list: [INPUT.en.md](../docs/INPUT.en.md). Pricing: [PRICING.en.md](../docs/PRICING.en.md).
