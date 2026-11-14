# Java — get started

Java 17 or newer. Uses `java.net.http` from the JDK. No Maven dependency.

Contract and field list: [api.md](api.md).

## Use cases

- Daily monitor: `runMode` `monitor` and the same `historyStoreName`.
- Agency: `placeItems` with `label`, `clientName`, and `tags`.
- Your app reads `priority_action` rows after the run.
- Cap spend with `maxBudgetUsd` and `maxPlacesPerRun`. A successful place is USD 0.10.

## Run

Save as `MonitorReviews.java`.

```bash
APIFY_TOKEN=apify_api_xxx java MonitorReviews.java
```

```java
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.time.Duration;
import java.util.Set;

public class MonitorReviews {
    static final String ACTOR = "basta_compilar~google-maps-review-change-feed";
    static final Set<String> TERMINAL = Set.of("SUCCEEDED", "FAILED", "ABORTED", "TIMED-OUT");

    public static void main(String[] args) throws Exception {
        String token = System.getenv("APIFY_TOKEN");
        if (token == null || token.isBlank()) {
            throw new IllegalStateException("Set APIFY_TOKEN");
        }
        String input = """
                {
                  "placeItems": [{
                    "place": "https://www.google.com/maps/search/Ifome+Burguer+Hortol%C3%A2ndia",
                    "label": "Hortolândia",
                    "clientName": "Demo Brand",
                    "tags": ["brasil"]
                  }],
                  "runMode": "monitor",
                  "locale": "pt-BR",
                  "mapsHost": "www.google.com.br",
                  "sortBy": "newest",
                  "reviewSource": "auto",
                  "maxReviewsPerPlace": 20,
                  "maxPlacesPerRun": 50,
                  "maxBudgetUsd": 5,
                  "historyStoreName": "google-maps-review-history",
                  "includeFlatExport": true,
                  "includeRunDigest": true,
                  "notifyOn": "priority_actions",
                  "minPriorityToNotify": "high",
                  "emailMode": "digest"
                }
                """;

        HttpClient http = HttpClient.newBuilder().connectTimeout(Duration.ofSeconds(30)).build();
        String started = call(http, token, "POST", "/acts/" + ACTOR + "/runs", input);
        String runId = field(started, "id");
        String datasetId = field(started, "defaultDatasetId");
        String status = field(started, "status");

        for (int i = 0; i < 120 && !TERMINAL.contains(status); i++) {
            Thread.sleep(5000);
            status = field(call(http, token, "GET", "/actor-runs/" + runId, null), "status");
        }

        String items = call(http, token, "GET", "/datasets/" + datasetId + "/items?format=json&clean=true", null);
        int actions = count(items, "\"recordType\":\"priority_action\"");
        System.out.println("runId=" + runId + " status=" + status + " priorityActions=" + actions);
        System.out.println(items);
        if (!"SUCCEEDED".equals(status)) {
            System.exit(1);
        }
    }

    static String call(HttpClient http, String token, String method, String path, String body) throws Exception {
        HttpRequest.Builder builder = HttpRequest.newBuilder()
                .uri(URI.create("https://api.apify.com/v2" + path))
                .timeout(Duration.ofSeconds(60))
                .header("Authorization", "Bearer " + token)
                .header("Content-Type", "application/json");
        if ("POST".equals(method)) {
            builder.POST(HttpRequest.BodyPublishers.ofString(body));
        } else {
            builder.GET();
        }
        HttpResponse<String> response = http.send(builder.build(), HttpResponse.BodyHandlers.ofString());
        if (response.statusCode() / 100 != 2) {
            throw new IllegalStateException(response.statusCode() + " " + response.body());
        }
        return response.body();
    }

    static String field(String json, String key) {
        String needle = "\"" + key + "\":\"";
        int start = json.indexOf(needle);
        if (start < 0) {
            return "";
        }
        start += needle.length();
        return json.substring(start, json.indexOf('"', start));
    }

    static int count(String json, String needle) {
        int found = 0;
        for (int from = 0; (from = json.indexOf(needle, from)) >= 0; from += needle.length()) {
            found++;
        }
        return found;
    }
}
```

Keep `historyStoreName` stable for that portfolio. The first successful run is the baseline.

Full field list: [INPUT.en.md](../docs/INPUT.en.md). Pricing: [PRICING.en.md](../docs/PRICING.en.md).
