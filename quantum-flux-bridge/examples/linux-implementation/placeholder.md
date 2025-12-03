# 🐧 Linux Implementation - Quantum-Flux Bridge

*En desarrollo - Próximamente disponible Q3 2024*

## 📋 Descripción
Implementación nativa de Quantum-Flux Bridge para sistemas Linux, funcionando como módulo del kernel o espacio de usuario.

## 🎯 Distribuciones Soportadas
- **Enterprise:** RHEL/CentOS 7+, Ubuntu 20.04+, Debian 10+
- **Cloud:** Amazon Linux 2, Ubuntu Server, Container Linux
- **Embedded:** OpenWRT, Yocto, Buildroot
- **Desktop:** Cualquier distribución con kernel ≥ 4.15

## 🏗️ Arquitecturas
- **Kernel Module:** Alto rendimiento, integración profunda
- **eBPF:** Flexible, seguro, sin recompilar kernel
- **Userspace Daemon:** Fácil deployment, menos privilegios

## 🚀 Métodos de Instalación

### Opción A: Paquete del Sistema
```bash
# Debian/Ubuntu
curl -s https://packages.quantumflux.io/key.gpg | sudo apt-key add -
echo "deb https://packages.quantumflux.io/ubuntu focal main" | sudo tee /etc/apt/sources.list.d/quantumflux.list
sudo apt update
sudo apt install quantumflux-bridge

# RHEL/CentOS
sudo yum install https://packages.quantumflux.io/rpm/quantumflux-bridge-latest.rpm
```

### Opción B: Docker
```bash
docker run -d \
  --name qfb \
  --network host \
  --cap-add=NET_ADMIN \
  --cap-add=NET_RAW \
  -v /etc/qfb:/etc/qfb \
  quantumflux/bridge:linux
```

### Opción C: Desde Código
```bash
git clone https://github.com/enriqueherbertag-igtm/quantum-flux-bridge.git
cd quantum-flux-bridge/linux
make
sudo make install
sudo systemctl enable qfb
sudo systemctl start qfb
```

## ⚙️ Configuración de Sistema

### systemd Service
```ini
# /etc/systemd/system/qfb.service
[Unit]
Description=Quantum-Flux Bridge
After=network-online.target
Wants=network-online.target

[Service]
Type=notify
ExecStart=/usr/bin/qfb --config=/etc/qfb/config.yaml
Restart=on-failure
CapabilityBoundingSet=CAP_NET_ADMIN CAP_NET_RAW
AmbientCapabilities=CAP_NET_ADMIN CAP_NET_RAW

[Install]
WantedBy=multi-user.target
```

### Configuración Netplan (Ubuntu 20.04+)
```yaml
# /etc/netplan/01-qfb.yaml
network:
  version: 2
  bridges:
    qfb0:
      interfaces: [eth0, wlan0]
      parameters:
        stp: false
      addresses: [192.168.100.1/24]
```

## 🔧 Ejemplos de Uso

### Gateway para Red Local
```bash
# 1. Habilitar IP forwarding
echo 1 > /proc/sys/net/ipv4/ip_forward

# 2. Configurar NAT
iptables -t nat -A POSTROUTING -o qfb0 -j MASQUERADE

# 3. Configurar clients para usar 192.168.100.1 como gateway
```

### Optimizador para Servidor Web
```yaml
# /etc/qfb/web-server.yaml
applications:
  - name: "nginx"
    ports: [80, 443]
    priority: "high"
    optimization:
      compression: "aggressive"
      fec: true
      multipath: true
  
  - name: "ssh"
    ports: [22]
    priority: "medium"
    optimization:
      latency: "minimize"
```

## 📊 Monitoreo y Debug

### Comandos Útiles
```bash
# Estado del servicio
sudo qfbctl status

# Métricas en tiempo real
sudo qfbctl metrics --live

# Debug de enlaces
sudo qfbctl debug links --verbose

# Generar reporte
sudo qfbctl diagnose --output=report.tar.gz
```

### Integración con Herramientas Existentes
```bash
# Prometheus metrics
curl http://localhost:9090/metrics

# JSON API
curl http://localhost:9090/api/v1/status

# Logs estructurados (JSON)
journalctl -u qfb -f -o json
```

## 🐛 Troubleshooting Común

### Problema: Módulo del kernel no carga
```bash
# Verificar dependencias
ldd /lib/modules/$(uname -r)/extra/qfb.ko

# Cargar manualmente
sudo modprobe qfb debug=1

# Ver logs del kernel
dmesg | grep qfb
```

### Problema: Interfaces no detectadas
```bash
# Listar interfaces disponibles
ip link show

# Forzar detección
sudo qfbctl detect --force

# Ver configuración de red
networkctl status
```

## 🗓️ Roadmap de Desarrollo

### v0.1.0 (Actual)
- ✅ Daemon userspace básico
- ✅ Configuración YAML
- ✅ API REST básica

### v0.2.0 (Q3 2024)
- 🔄 Módulo del kernel
- 🔄 Soporte eBPF
- 🔄 Integración netfilter

### v0.3.0 (Q4 2024)
- 🔄 High-availability clustering
- 🔄 Zero-config para redes comunes
- 🔄 GUI web administration

## 🤝 Contribuir
¿Desarrollador Linux/kernel? ¡Necesitamos ayuda!
- **Kernel hackers:** Módulo de rendimiento
- **Network engineers:** Configuraciones reales
- **DevOps:** Deployment automatizado

## 🔗 Recursos
- [Kernel Documentation](/docs/kernel.md)
- [Performance Tuning](/docs/tuning.md)
- [Security Hardening](/docs/security.md)

---

**"Linux en todas partes, optimizado en todas partes."**
