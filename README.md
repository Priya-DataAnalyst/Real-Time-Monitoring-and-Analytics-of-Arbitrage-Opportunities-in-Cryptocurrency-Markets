# real-time crypto arbitrage analytics

## overview

a real-time pipeline that watches multiple exchanges and highlights potential **arbitrage** (buy low on one exchange, sell higher on another). it streams trades, filters duplicates, protects privacy, clusters price behavior, and surfaces signals on a live dashboard with optional email alerts.

## key features

* **multi-exchange ingestion**: bitstamp, bitfinex, kraken via websocket + kafka topics.
* **stream processing**: apache spark for low-latency transforms.
* **duplicate control**: bloom filter to skip repeated uuids.
* **privacy**: differential privacy (noise + hashing) for anonymized ids.
* **pattern discovery**: k-means for price clusters, lsh for fast similarity lookup.
* **storage**: influxdb (time-series) + mongodb (analysis).
* **visuals**: grafana dashboard (min buy, max sell, latest trades).
* **alerts**: optional email notifications on profitable spreads.
* **fairness checks**: coverage by exchange/symbol, cluster-level variance (anova).

## architecture (high level)

1. exchanges → kafka topics
2. spark streaming → de-dupe (bloom), privacy, features → k-means + lsh
3. write to influxdb/mongodb
4. grafana panels + email alerts
   *(add your diagram image at `docs/architecture.png`)*

## tech stack

kafka • spark • python • mongodb • influxdb • grafana • docker (local)

## quick start

> prerequisites: docker, docker compose, python 3.10+

```bash
# 1) clone
git clone <your-repo-url>
cd <your-repo-name>

# 2) environment
cp .env.example .env
# edit api keys, db urls, smtp creds

# 3) start infra (kafka, zookeeper, mongodb, influxdb, grafana)
docker compose up -d

# 4) run stream producer (coinapi/websocket -> kafka)
python apps/streaming/kafka_producer.py

# 5) run spark job (consume -> transform -> write)
python apps/streaming/spark_stream.py

# 6) run etl/analysis (k-means, lsh, plots)
python apps/analysis/run_etl.py

# 7) (optional) start email alert worker
python apps/alerts/alert_worker.py
```

## configuration

* **.env**

  * `COINAPI_KEY=...` (or other feed keys)
  * `KAFKA_BROKER=...`
  * `MONGO_URI=...` / `MONGO_DB=...`
  * `INFLUX_URL=...` / `INFLUX_TOKEN=...` / `INFLUX_BUCKET=...`
  * `SMTP_SERVER=...` / `SMTP_PORT=...` / `SMTP_USER=...` / `SMTP_PASS=...` *(app password as needed)*

## repo structure

```
notebooks/
spark.ipynb # spark streaming jobs and transformations
kafka.ipynb # kafka producer/consumer setup for trade streams
mongodbscript.ipynb # load and manage trade data in mongodb
mongodb_data_analysis.ipynb # analyze clusters and price spreads
influxdb_script.ipynb # store and query time-series data
arbitragedetection.ipynb # detect arbitrage opportunities using k-means + lsh
EmailAlertscript.ipynb # automated email notifications for detected spreads
Script_copy_files_to_temp...ipynb # utility script for local file management
```

## data flow & methods

* **bloom filter**: fast membership test to skip duplicate uuids in stream.
* **differential privacy**: laplace noise + hashing → anonymized ids.
* **k-means**: cluster trades by price/derived features; inspect price-diff spread.
* **lsh**: bucket similar patterns for fast lookup of near matches.

## dashboards

grafana shows:

* **min buy** per exchange
* **max sell** per exchange
* **latest buy/sell tickers**
  refresh every 5 minutes (flux queries to influxdb). add screenshots in `docs/screenshots/`.

## alerts

the alert worker scans recent trades (e.g., last 10 minutes) in mongodb, computes spreads, and emails details when thresholds hit.

## evaluation

* latency & throughput of the stream
* correctness of arbitrage detection (precision of flagged events)
* anova on cluster price-diffs to confirm significant variance.

## results (summary)

* combined **k-means + lsh** improved speed of finding similar price behavior.
* **bloom + privacy** cut duplicates and safeguarded identities.
* grafana delivered near-real-time visibility for decisions.

## known limits / future work

* reduce stream latency further; add more exchanges/feeds
* sentiment/news signals; anomaly detection for manipulation
* stronger error handling; optional automation hooks for trades.

## fairness checklist

* balanced ingestion by symbol/exchange, cluster-level checks, and statistical tests to avoid bias toward specific times or markets.

## team & credits

contributors followed the **CRediT taxonomy** (conceptualization, methodology, software, validation, analysis, investigation, writing, visualization).

## license

This project is licensed under the [MIT License](./LICENSE).

