# Atividade - Aula 05

## Exercício 2:
`logger.js`
```
const fs = require("fs");
const path = require("path");

const logDir = path.join(__dirname, "logs");
if (!fs.existsSync(logDir)) {
  fs.mkdirSync(logDir, { recursive: true });
}

function log(level, message, extra = {}) {
  const entry = {
    timestamp: new Date().toISOString(),
    level,
    message,
    ...extra,
  };
  const line = JSON.stringify(entry);
  console.log(line);
  fs.appendFileSync(path.join(logDir, "app.log"), line + "\n");
}

const logger = {
  info:  (msg, extra) => log("INFO",  msg, extra),
  warn:  (msg, extra) => log("WARN",  msg, extra),
  error: (msg, extra) => log("ERROR", msg, extra),
};

logger.info("Aplicação INPE Alertas iniciada", { version: "1.0.0" });
logger.info("Novo alerta recebido", { tipo: "Queimada", regiao: "Amazônia", nivel: 92 });
logger.warn("Nível de alerta elevado", { tipo: "Inundação", nivel: 75, limite: 70 });
logger.error("Falha ao enviar notificação push", { usuarioId: 42, erro: "Timeout" });

module.exports = logger;
```

## Exercício 3:
`monitoring/fluentd/fluent.conf`
```
<source>
  @type tail
  path /logs/app.log
  pos_file /logs/app.log.pos
  tag inpe.alertas
  read_from_head true
  <parse>
    @type json
  </parse>
</source>

<filter inpe.alertas>
  @type grep
  <regexp>
    key level
    pattern /INFO|WARN|ERROR/
  </regexp>
</filter>

<match inpe.alertas>
  @type elasticsearch
  host elasticsearch
  port 9200
  logstash_format true
  logstash_prefix inpe-alertas
  flush_interval 5s
</match>
```

## Exercício 4:
`monitoring/prometheus/prometheus.yml`
```
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - "alertas.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - alertmanager:9093

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "inpe-alertas"
    static_configs:
      - targets: ["app:3000"]
    metrics_path: /metrics
```

## Exercício 5:
`monitoring/prometheus/alertas.yml`
```
groups:
  - name: inpe_alertas_criticos
    rules:

      - alert: AplicacaoIndisponivel
        expr: up{job="inpe-alertas"} == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "INPE Alertas está fora do ar"
          description: "A aplicação não responde há mais de 1 minuto."

      - alert: AltaTaxaDeErros
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.1
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Alta taxa de erros HTTP 5xx detectada"

      - alert: UsoDeMemoriaElevado
        expr: process_resident_memory_bytes{job="inpe-alertas"} > 200000000
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Uso de memória elevado na aplicação"

      - alert: LatenciaElevada
        expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 2
        for: 3m
        labels:
          severity: warning
        annotations:
          summary: "Latência P95 acima de 2 segundos"
```
