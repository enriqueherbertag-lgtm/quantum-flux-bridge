# 📡 API Overview - Quantum-Flux Bridge

**Interfaces programáticas para integración y administración**

---

## 🎯 Visión General

Quantum-Flux Bridge expone múltiples interfaces API para diferentes casos de uso:
- **REST API**: Administración y monitoreo (humano y script)
- **gRPC**: Comunicación entre componentes (alta performance)
- **Configuration API**: Configuración dinámica en tiempo real
- **Metrics API**: Telemetría y observabilidad

Todas las APIs soportan autenticación via API Key o TLS mTLS.

---

## 🔌 REST API (HTTP/JSON)

### Base URL
```
http://localhost:9000/api/v1
```

### Endpoints Principales

#### 1. Salud del Sistema
```http
GET /health
```
**Respuesta:**
```json
{
  "status": "healthy",
  "version": "0.1.0",
  "uptime": "1245h32m16s",
  "components": {
    "core": "healthy",
    "routing": "healthy",
    "monitoring": "healthy"
  }
}
```

#### 2. Métricas del Sistema
```http
GET /metrics
```
**Formatos soportados:**
- `Accept: application/json` → JSON estructurado
- `Accept: text/plain` → Prometheus format
- `Accept: application/openmetrics-text` → OpenMetrics

**Ejemplo JSON:**
```json
{
  "throughput": {
    "current": "145.2 Mbps",
    "avg_5min": "132.8 Mbps",
    "peak_today": "210.5 Mbps"
  },
  "latency": {
    "current": "42 ms",
    "avg_5min": "38 ms",
    "percentile_95": "67 ms"
  },
  "packet_loss": {
    "current": "0.2%",
    "avg_5min": "0.3%"
  }
}
```

#### 3. Gestión de Enlaces
```http
GET /links
```
```http
GET /links/{link_id}
```
```http
PUT /links/{link_id}/weight
```
```http
POST /links/{link_id}/test
```

**Ejemplo: Listar enlaces**
```json
{
  "links": [
    {
      "id": "wan1",
      "interface": "eth0",
      "type": "ethernet",
      "status": "active",
      "weight": 70,
      "metrics": {
        "latency": "32ms",
        "jitter": "8ms",
        "packet_loss": "0.1%",
        "throughput": "95.2Mbps"
      }
    },
    {
      "id": "lte_backup",
      "interface": "ppp0",
      "type": "cellular",
      "status": "standby",
      "weight": 30,
      "metrics": {
        "latency": "125ms",
        "jitter": "45ms",
        "packet_loss": "1.2%",
        "throughput": "18.5Mbps"
      }
    }
  ]
}
```

#### 4. Gestión de Aplicaciones
```http
GET /applications
```
```http
POST /applications
```
```http
PUT /applications/{app_id}/priority
```
```http
DELETE /applications/{app_id}
```

**Ejemplo: Definir aplicación**
```json
{
  "name": "video_conferencing",
  "priority": "high",
  "traffic_patterns": [
    {
      "protocol": "UDP",
      "port_range": "50000-51000",
      "required_bandwidth": "2Mbps",
      "max_latency": "150ms",
      "max_jitter": "30ms"
    }
  ],
  "policies": {
    "prefer_low_latency": true,
    "allow_compression": false,
    "enable_fec": true
  }
}
```

#### 5. Configuración en Tiempo Real
```http
GET /config
```
```http
PATCH /config
```
```http
POST /config/validate
```

**Ejemplo: Actualizar configuración parcial**
```json
{
  "op": "update",
  "path": "/optimization/thresholds/packet_loss_warn",
  "value": "3%"
}
```

#### 6. Diagnóstico y Troubleshooting
```http
POST /diagnose
```
```http
GET /logs
```
```http
GET /events
```

---

## ⚡ gRPC API (Alta Performance)

### Servicios Definidos

#### 1. `LinkService`
```proto
service LinkService {
  rpc ListLinks(ListLinksRequest) returns (ListLinksResponse);
  rpc GetLinkMetrics(GetLinkMetricsRequest) returns (stream LinkMetrics);
  rpc UpdateLinkWeight(UpdateLinkWeightRequest) returns (UpdateLinkResponse);
  rpc TestLink(TestLinkRequest) returns (TestLinkResponse);
}
```

#### 2. `RoutingService`
```proto
service RoutingService {
  rpc GetRoute(GetRouteRequest) returns (RouteResponse);
  rpc UpdateRoutingTable(UpdateRoutingRequest) returns (UpdateRoutingResponse);
  rpc StreamRoutingEvents(Empty) returns (stream RoutingEvent);
}
```

#### 3. `TelemetryService`
```proto
service TelemetryService {
  rpc StreamMetrics(StreamMetricsRequest) returns (stream MetricUpdate);
  rpc GetHealth(HealthRequest) returns (HealthStatus);
  rpc GetStatistics(StatisticsRequest) returns (StatisticsResponse);
}
```

### Ejemplo de Cliente Go
```go
package main

import (
    "context"
    "log"
    "time"
    
    pb "github.com/enriqueherbertag-igtm/quantum-flux-bridge/proto"
    "google.golang.org/grpc"
)

func main() {
    conn, err := grpc.Dial("localhost:9090", grpc.WithInsecure())
    if err != nil {
        log.Fatal(err)
    }
    defer conn.Close()
    
    client := pb.NewLinkServiceClient(conn)
    
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()
    
    resp, err := client.ListLinks(ctx, &pb.ListLinksRequest{})
    if err != nil {
        log.Fatal(err)
    }
    
    for _, link := range resp.Links {
        log.Printf("Link: %s, Status: %s, Latency: %v", 
            link.Id, link.Status, link.Metrics.Latency)
    }
}
```

---

## ⚙️ Configuration API (YAML/JSON)

### Configuración Principal
```yaml
api:
  enabled: true
  interfaces:
    - name: "rest"
      port: 9000
      enable_cors: true
      authentication:
        method: "api_key"
        required: true
    
    - name: "grpc"
      port: 9090
      enable_tls: true
      authentication:
        method: "mtls"
        required: true
  
  rate_limiting:
    enabled: true
    requests_per_minute: 100
    burst_size: 20
  
  logging:
    level: "info"
    format: "json"
    file: "/var/log/qfb-api.log"
```

### Configuración por Entorno
```bash
# Desarrollo
QFB_API_PORT=9000
QFB_API_AUTH=false
QFB_LOG_LEVEL=debug

# Producción
QFB_API_PORT=443
QFB_API_AUTH=true
QFB_API_TLS_CERT=/etc/ssl/certs/qfb.crt
QFB_API_TLS_KEY=/etc/ssl/private/qfb.key
```

---

## 📊 Metrics API (Observabilidad)

### Métricas Expuestas

#### Métricas de Red
```
qfb_link_latency_ms{link="wan1"} 32.5
qfb_link_jitter_ms{link="wan1"} 8.2
qfb_link_packet_loss{link="wan1"} 0.001
qfb_link_throughput_bps{link="wan1"} 95200000
```

#### Métricas de Sistema
```
qfb_cpu_usage_percent 45.2
qfb_memory_usage_bytes 128450560
qfb_active_connections 42
qfb_routing_decisions_total 12458
```

#### Métricas de Aplicación
```
qfb_app_throughput_bps{app="video_conference"} 1500000
qfb_app_latency_ms{app="video_conference"} 85
qfb_app_packet_loss{app="video_conference"} 0.005
```

### Integración con Prometheus
```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'quantum-flux'
    static_configs:
      - targets: ['qfb-host:9000']
    metrics_path: '/metrics'
    scheme: 'http'
    scrape_interval: 15s
```

### Dashboard Grafana
Importar dashboard template:
```
Dashboard ID: 14784
URL: https://grafana.com/grafana/dashboards/14784
```

---

## 🔐 Autenticación y Seguridad

### Métodos Soportados

#### 1. API Keys
```bash
curl -H "X-API-Key: qfb_sk_live_1234567890abcdef" \
  http://localhost:9000/api/v1/health
```

#### 2. TLS mTLS
```bash
curl --cert client.crt --key client.key \
  https://qfb.example.com/api/v1/health
```

#### 3. JWT Tokens
```bash
curl -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..." \
  http://localhost:9000/api/v1/health
```

### Ejemplo de Middleware de Autenticación
```go
func AuthMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        apiKey := r.Header.Get("X-API-Key")
        
        if !isValidAPIKey(apiKey) {
            http.Error(w, "Unauthorized", http.StatusUnauthorized)
            return
        }
        
        next.ServeHTTP(w, r)
    })
}
```

---

## 🧪 Testing la API

### Suite de Pruebas
```bash
# Instalar herramienta de testing
go get github.com/steinfletcher/apitest

# Ejecutar tests
cd api && go test -v ./...

# Tests específicos
go test -run TestHealthEndpoint
go test -run TestLinkManagement
```

### Ejemplo de Test
```go
func TestHealthEndpoint(t *testing.T) {
    handler := setupAPI()
    
    apitest.New().
        Handler(handler).
        Get("/api/v1/health").
        Expect(t).
        Status(http.StatusOK).
        Body(`{"status":"healthy"}`).
        End()
}
```

### Pruebas de Carga
```bash
# Usar k6 para pruebas de carga
k6 run --vus 100 --duration 30s api-load-test.js
```

```javascript
// api-load-test.js
import http from 'k6/http';
import { check, sleep } from 'k6';

export default function() {
  let res = http.get('http://localhost:9000/api/v1/health');
  check(res, {
    'status is 200': (r) => r.status === 200,
    'response time < 50ms': (r) => r.timings.duration < 50
  });
  sleep(0.1);
}
```

---

## 🚀 Quick Start para Desarrolladores

### 1. Iniciar API Local
```bash
# Modo desarrollo
./qfb api --dev --port=9000

# Verificar
curl http://localhost:9000/api/v1/health
```

### 2. Generar Cliente SDK
```bash
# Generar cliente Go
protoc --go_out=. --go-grpc_out=. proto/*.proto

# Generar cliente Python
python -m grpc_tools.protoc -I./proto --python_out=. --grpc_python_out=. proto/*.proto

# Generar cliente TypeScript
protoc --plugin=protoc-gen-ts=./node_modules/.bin/protoc-gen-ts \
  --ts_out=. proto/*.proto
```

### 3. Ejemplo de Integración
```python
# client.py
import requests
import json

class QFBClient:
    def __init__(self, base_url, api_key):
        self.base_url = base_url
        self.headers = {'X-API-Key': api_key}
    
    def get_health(self):
        response = requests.get(
            f"{self.base_url}/health",
            headers=self.headers
        )
        return response.json()
    
    def update_link_weight(self, link_id, weight):
        response = requests.put(
            f"{self.base_url}/links/{link_id}/weight",
            headers=self.headers,
            json={'weight': weight}
        )
        return response.json()

# Uso
client = QFBClient('http://localhost:9000/api/v1', 'my-api-key')
print(client.get_health())
```

---

## 📚 Recursos Adicionales

### Documentación Completa
- [API Reference](https://docs.quantumflux.io/api) (cuando exista)
- [gRPC Proto Definitions](/proto/)
- [Postman Collection](/docs/postman-collection.json)

### Herramientas Recomendadas
- **Postman/Insomnia**: Testing REST API
- **grpcurl**: Testing gRPC endpoints
- **BloomRPC**: GUI para gRPC
- **Prometheus + Grafana**: Monitoreo

### Soporte
- **Issues de GitHub**: Reportar bugs en API
- **Discord/Slack**: Soporte en tiempo real (futuro)
- **Stack Overflow**: Tag `quantum-flux-bridge`

---

## 📝 Notas de Versión API

### v0.1.0 (Actual)
- ✅ REST API básica completa
- ✅ gRPC service definitions
- ✅ Autenticación via API Key
- ✅ Métricas Prometheus

### v0.2.0 (Próximo)
- 🔄 WebSocket para updates en tiempo real
- 🔄 GraphQL endpoint alternativo
- 🔄 OpenAPI 3.0 specification
- 🔄 SDKs oficiales (Python, Go, JavaScript)

### v0.3.0 (Planificado)
- 🔄 Admin UI completa
- 🔄 Plugin system para extensiones
- 🔄 Audit logging completo
- 🔄 Rate limiting avanzado

---

**"Una API bien diseñada es como un buen contrato: claro, consistente y justo para todas las partes."**
