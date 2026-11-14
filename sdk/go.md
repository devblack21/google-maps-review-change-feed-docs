# Go — get started

Go 1.22 or newer. Uses `net/http` and `encoding/json` from the standard library. No third-party module.

Contract and field list: [api.md](api.md).

## Use cases

- Daily monitor: `runMode` `monitor` and the same `historyStoreName`.
- Agency: `placeItems` with `label`, `clientName`, and `tags`.
- Your service reads `priority_action` rows after the run.
- Cap spend with `maxBudgetUsd` and `maxPlacesPerRun`. A successful place is USD 0.10.

## Run

Save as `main.go`.

```bash
go mod init reviewfeed
APIFY_TOKEN=apify_api_xxx go run .
```

```go
package main

import (
	"bytes"
	"encoding/json"
	"fmt"
	"io"
	"net/http"
	"os"
	"time"
)

const actor = "basta_compilar~google-maps-review-change-feed"

type runEnvelope struct {
	Data struct {
		ID               string `json:"id"`
		Status           string `json:"status"`
		DefaultDatasetID string `json:"defaultDatasetId"`
	} `json:"data"`
}

type datasetItem struct {
	RecordType string `json:"recordType"`
	PlaceName  string `json:"placeName"`
	Priority   string `json:"priority"`
	Reason     string `json:"reason"`
}

func main() {
	token := os.Getenv("APIFY_TOKEN")
	if token == "" {
		panic("Set APIFY_TOKEN")
	}
	input := map[string]any{
		"placeItems": []map[string]any{{
			"place":      "https://www.google.com/maps/search/Ifome+Burguer+Hortol%C3%A2ndia",
			"label":      "Hortolândia",
			"clientName": "Demo Brand",
			"tags":       []string{"brasil"},
		}},
		"runMode":              "monitor",
		"locale":               "pt-BR",
		"mapsHost":             "www.google.com.br",
		"sortBy":               "newest",
		"reviewSource":         "auto",
		"maxReviewsPerPlace":   20,
		"maxPlacesPerRun":      50,
		"maxBudgetUsd":         5,
		"historyStoreName":     "google-maps-review-history",
		"includeFlatExport":    true,
		"includeRunDigest":     true,
		"notifyOn":             "priority_actions",
		"minPriorityToNotify":  "high",
		"emailMode":            "digest",
	}

	var started runEnvelope
	apify(token, http.MethodPost, "/acts/"+actor+"/runs", input, &started)
	status := started.Data.Status
	terminal := map[string]bool{"SUCCEEDED": true, "FAILED": true, "ABORTED": true, "TIMED-OUT": true}
	for i := 0; i < 120 && !terminal[status]; i++ {
		time.Sleep(5 * time.Second)
		var polled runEnvelope
		apify(token, http.MethodGet, "/actor-runs/"+started.Data.ID, nil, &polled)
		status = polled.Data.Status
	}

	var items []datasetItem
	apify(token, http.MethodGet, "/datasets/"+started.Data.DefaultDatasetID+"/items?format=json&clean=true", nil, &items)
	var actions []datasetItem
	for _, item := range items {
		if item.RecordType == "priority_action" {
			actions = append(actions, item)
		}
	}
	out, _ := json.MarshalIndent(map[string]any{
		"runId": started.Data.ID, "status": status, "actions": actions,
	}, "", "  ")
	fmt.Println(string(out))
	if status != "SUCCEEDED" {
		os.Exit(1)
	}
}

func apify(token, method, path string, body any, dest any) {
	var reader io.Reader
	if body != nil {
		raw, err := json.Marshal(body)
		if err != nil {
			panic(err)
		}
		reader = bytes.NewReader(raw)
	}
	req, err := http.NewRequest(method, "https://api.apify.com/v2"+path, reader)
	if err != nil {
		panic(err)
	}
	req.Header.Set("Authorization", "Bearer "+token)
	req.Header.Set("Content-Type", "application/json")
	res, err := http.DefaultClient.Do(req)
	if err != nil {
		panic(err)
	}
	defer res.Body.Close()
	raw, _ := io.ReadAll(res.Body)
	if res.StatusCode/100 != 2 {
		panic(fmt.Sprintf("%d %s", res.StatusCode, raw))
	}
	if err := json.Unmarshal(raw, dest); err != nil {
		panic(err)
	}
}
```

Keep `historyStoreName` stable for that portfolio. The first successful run is the baseline.

Full field list: [INPUT.en.md](../docs/INPUT.en.md). Pricing: [PRICING.en.md](../docs/PRICING.en.md).
