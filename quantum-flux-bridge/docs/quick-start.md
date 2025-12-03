# 🚀 Inicio Rápido - Quantum-Flux Bridge

**Primeros pasos en 10 minutos**  
*Para desarrolladores, administradores de red y evaluadores técnicos*

---

## ⚡ Instalación Express

### Opción 1: Docker (Recomendado para evaluación)
```bash
# 1. Obtener la imagen
docker pull quantumflux/bridge:latest

# 2. Ejecutar contenedor de prueba
docker run -d \
  --name qfb-test \
  -p 8080:8080 \
  -p 9000:9000 \
  quantumflux/bridge:latest \
  --mode=standalone \
  --config=/etc/qfb/minimal.yaml

# 3. Verificar estado
docker logs qfb-test
```

### Opción 2: Binarios Pre-compilados
```bash
# Linux x86_64
wget https://releases.quantumflux.io/v0.1.0/qfb-linux-amd64
chmod +x qfb-linux-amd64
./qfb-linux-amd64 --help

# macOS
wget https://releases.quantumflux.io/v0.1.0/qfb-darwin-amd64
chmod +x qfb-darwin-amd64
./qfb-darwin-amd64 --help
```

### Opción 3: Desde Código Fuente (Desarrolladores)
```bash
# Prerrequisitos: Go 1.21+, make, git
git clone https://github.com/enriqueherbertag-igtm/quantum-flux-bridge.git
cd quantum-flux-bridge

# Compilar
make build

# Ejecutar pruebas básicas
make test-simple

# Ejecutar servidor de desarrollo
./bin/qfb dev --port=8080
```

---

## 🎯 Ejemplo Básico: Proxy de Optimización

### Escenario: Mejorar conexión HTTP inestable

**1. Configuración (`config.yaml`):**
```yaml
version: "1.0"
mode: "transparent-proxy"

interfaces:
  - name: "wan1"
    type: "ethernet"
    address: "192.168.1.100"
  
  - name: "lte_backup"
    type: "cellular"
    address: "dynamic"

optimization:
  enabled: true
  algorithms:
    - "multipath"
    - "compression"
    - "fec"
  
  thresholds:
    packet_loss_warn: 5%    # >5% activa FEC
    latency_warn: 150ms     # >150ms activa compresión
    jitter_warn: 50ms       # >50ms activa buffering

logging:
  level: "info"
  output: "stdout"
```

**2. Iniciar servicio:**
```bash
./qfb start --config=config.yaml
```

**3. Probar optimización:**
```bash
# Sin QFB (línea base)
curl --max-time 10 https://api.example.com/data

# Con QFB a través del proxy
curl --max-time 10 --proxy http://localhost:8080 https://api.example.com/data

# Ver métricas en tiempo real
curl http://localhost:9000/metrics
```

---

## 📊 Monitoreo Inmediato

### Panel Web (puerto 9000)
```
http://localhost:9000/dashboard
```

### Métricas vía API:
```bash
# Salud del servicio
curl http://localhost:9000/health

# Métricas en formato Prometheus
curl http://localhost:9000/metrics

# Estadísticas de enlaces
curl http://localhost:9000/api/v1/links

# Rendimiento por aplicación
curl http://localhost:9000/api/v1/applications
```

### Logs en tiempo real:
```bash
# Ver logs
tail -f /var/log/qfb.log

# Filtrar por severidad
grep "WARN\|ERROR" /var/log/qfb.log

# Métricas clave a monitorear
watch -n 5 'curl -s http://localhost:9000/metrics | grep "packet_loss\|throughput"'
```

---

## 🔧 Configuración de Ejemplo: Dos Enlaces

**Caso:** Internet por fibra + LTE como backup

```yaml
# config-dual-wan.yaml
links:
  primary:
    interface: "eth0"
    gateway: "192.168.1.1"
    weight: 70
    monitoring:
      target: "8.8.8.8"
      interval: "5s"
      timeout: "2s"
  
  backup:
    interface: "ppp0"
    gateway: "dynamic"
    weight: 30
    cost_multiplier: 2.0  # LTE es más caro, usar menos

routing:
  strategy: "performance-weighted"
  failover_threshold: 3    # 3 fallas consecutivas = failover
  restore_check_interval: "30s"

applications:
  - name: "video_conference"
    priority: "high"
    required_bandwidth: "1Mbps"
    max_latency: "150ms"
    
  - name: "file_backup"
    priority: "low"
    required_bandwidth: "500Kbps"
    schedule: "02:00-06:00"
```

**Iniciar con configuración dual:**
```bash
./qfb start --config=config-dual-wan.yaml --log-level=debug
```

---

## 🐛 Solución de Problemas Comunes

### Problema: Servicio no inicia
```bash
# 1. Verificar puertos
netstat -tulpn | grep :8080
netstat -tulpn | grep :9000

# 2. Verificar permisos
ls -la /var/log/qfb.log

# 3. Modo debug
./qfb start --config=config.yaml --log-level=debug --dry-run
```

### Problema: No detecta interfaces
```bash
# Listar interfaces disponibles
ip link show

# Forzar interfaz específica
./qfb start --interface=eth0 --fallback-interface=wlan0
```

### Problema: Alto uso de CPU
```bash
# Reducir complejidad de algoritmos
./qfb start --simple-mode --disable-fec --compression-level=1

# Limitar recursos
./qfb start --max-cpu=50% --max-memory=512M
```

---

## 📈 Verificar Mejora de Rendimiento

### Test de línea base (sin QFB):
```bash
# Instalar herramientas de test (si no las tienes)
sudo apt install iperf3 netperf

# 1. Medir ancho de banda
iperf3 -c server.example.com -t 30 -i 5

# 2. Medir pérdida de paquetes
ping -c 100 server.example.com | grep "packet loss"

# 3. Medir latencia y jitter
mtr --report --report-cycles=100 server.example.com
```

### Test con QFB activo:
```bash
# Configurar QFB como proxy transparente
export http_proxy="http://localhost:8080"
export https_proxy="http://localhost:8080"

# Repetir tests
iperf3 -c server.example.com -t 30 -i 5
ping -c 100 server.example.com
```

### Comparar resultados:
```bash
# Script de comparación simple
echo "=== SIN QFB ==="
cat baseline_results.txt
echo ""
echo "=== CON QFB ==="
cat optimized_results.txt
echo ""
echo "=== MEJORA ==="
python3 -c "
import sys
baseline = 45.2  # Mbps sin QFB
optimized = 67.8  # Mbps con QFB
improvement = ((optimized - baseline) / baseline) * 100
print(f'Throughput: +{improvement:.1f}%')
"
```

---

## 🚀 Siguientes Pasos

### Para Evaluación:
1. **Probar 24 horas** en entorno no crítico
2. **Monitorear** logs y métricas
3. **Documentar** comportamiento observado
4. **Probar failover** (desconectar enlace principal)

### Para Desarrollo:
1. **Revisar** `/examples/` para más escenarios
2. **Explorar** API en `http://localhost:9000/swagger`
3. **Modificar** configuración y observar efectos
4. **Contribuir** mejoras vía Pull Requests

### Para Producción:
1. **Leer** [Guía de Despliegue](deployment-guide.md)
2. **Configurar** monitoreo externo (Prometheus, Grafana)
3. **Establecer** backup de configuración
4. **Planear** ventana de mantenimiento

---

## 📞 Soporte Rápido

### Primeros 15 minutos de problemas:
1. ✅ ¿Servicio corriendo? `systemctl status qfb`
2. ✅ ¿Puertos abiertos? `ss -tulpn | grep 8080`
3. ✅ ¿Logs recientes? `tail -50 /var/log/qfb.log`
4. ✅ ¿Configuración válida? `./qfb validate --config=config.yaml`
5. ✅ ¿Recursos suficientes? `free -h && top -b -n1 | grep qfb`

### Si persiste el problema:
```bash
# Generar reporte de diagnóstico
./qfb diagnose --output=report.zip

# Subir a (enlace para reportes) o
# Abrir issue en GitHub con el reporte
```
