# Kotlin — get started

Kotlin/JVM on JDK 11 or newer. Uses `java.net.http` from the JDK. No Gradle dependency. The same program runs on a server or as the network call inside an Android-free Kotlin service. An Android app should call your own backend or the panel API later; do not ship `APIFY_TOKEN` inside a mobile binary.

Contract and field list: [api.md](api.md).

## Use cases

- Daily monitor: `runMode` `monitor` and the same `historyStoreName`.
- Agency: `placeItems` with `label`, `clientName`, and `tags`.
- Your service reads `priority_action` rows after the run.
- Cap spend with `maxBudgetUsd` and `maxPlacesPerRun`. A successful place is USD 0.10.

## Run

Save as `MonitorReviews.kt` next to a Kotlin compiler, or paste into an existing JVM project with no extra libraries.

```bash
kotlinc MonitorReviews.kt -include-runtime -d monitor.jar
APIFY_TOKEN=apify_api_xxx java -jar monitor.jar
```

```kotlin
import java.net.URI
import java.net.http.HttpClient
import java.net.http.HttpRequest
import java.net.http.HttpResponse
import java.time.Duration

private const val ACTOR = "basta_compilar~google-maps-review-change-feed"
private val terminal = setOf("SUCCEEDED", "FAILED", "ABORTED", "TIMED-OUT")

fun main() {
    val token = System.getenv("APIFY_TOKEN") ?: error("Set APIFY_TOKEN")
    val input = """
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
    """.trimIndent()

    val http = HttpClient.newBuilder().connectTimeout(Duration.ofSeconds(30)).build()
    val started = call(http, token, "POST", "/acts/$ACTOR/runs", input)
    val runId = field(started, "id")
    val datasetId = field(started, "defaultDatasetId")
    var status = field(started, "status")
    repeat(120) {
        if (status in terminal) return@repeat
        Thread.sleep(5000)
        status = field(call(http, token, "GET", "/actor-runs/$runId", null), "status")
    }
    val items = call(http, token, "GET", "/datasets/$datasetId/items?format=json&clean=true", null)
    val actions = items.split("\"recordType\":\"priority_action\"").size - 1
    println("runId=$runId status=$status priorityActions=$actions")
    println(items)
    if (status != "SUCCEEDED") kotlin.system.exitProcess(1)
}

private fun call(http: HttpClient, token: String, method: String, path: String, body: String?): String {
    val builder = HttpRequest.newBuilder()
        .uri(URI.create("https://api.apify.com/v2$path"))
        .timeout(Duration.ofSeconds(60))
        .header("Authorization", "Bearer $token")
        .header("Content-Type", "application/json")
    val request = if (method == "POST") {
        builder.POST(HttpRequest.BodyPublishers.ofString(body)).build()
    } else {
        builder.GET().build()
    }
    val response = http.send(request, HttpResponse.BodyHandlers.ofString())
    if (response.statusCode() / 100 != 2) error("${response.statusCode()} ${response.body()}")
    return response.body()
}

private fun field(json: String, key: String): String {
    val needle = "\"$key\":\""
    val start = json.indexOf(needle).takeIf { it >= 0 } ?: return ""
    val from = start + needle.length
    return json.substring(from, json.indexOf('"', from))
}
```

Keep `historyStoreName` stable for that portfolio. The first successful run is the baseline.

Full field list: [INPUT.en.md](../docs/INPUT.en.md). Pricing: [PRICING.en.md](../docs/PRICING.en.md).
