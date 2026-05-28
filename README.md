# 🖥️ Homelab — server-m73

> Infraestructura personal de servidores sobre un Lenovo ThinkCentre M73, orientada al aprendizaje de DevOps, redes, monitoreo y servicios en la nube local.

---

## 📦 Hardware

| Componente | Detalle |
|---|---|
| **Máquina** | Lenovo ThinkCentre M73 |
| **CPU** | Intel Core i7-4790 @ 3.6GHz (4 núcleos / 8 hilos) |
| **RAM** | 12 GB DDR3 1600MHz |
| **Almacenamiento** | 500 GB SSD |
| **Sistema Operativo** | Ubuntu 24.04 LTS |
| **Red** | IP pública + Tailscale VPN |

---

## 🏗️ Arquitectura General

```
Internet / Red local
        │
        ▼
  Tailscale VPN
  (subnet routing 192.168.1.0/24)
        │
        ▼
  server-m73 (192.168.1.75)
        │
        ├── Portainer        (gestión de contenedores)
        ├── Homepage         (dashboard de servicios)
        ├── Grafana          (visualización de métricas)
        ├── Prometheus       (recolección de métricas)
        ├── Node Exporter    (métricas del host)
        ├── cAdvisor         (métricas de contenedores)
        ├── Nextcloud        (nube personal)
        ├── Floci            (emulador AWS local)
        └── Minecraft Server (servidor de juego)
```

---

## 🛠️ Stack de Tecnologías

| Tecnología | Uso |
|---|---|
| **Docker + Docker Compose** | Contenedores y orquestación |
| **Portainer** | Gestión visual de contenedores |
| **Tailscale** | VPN privada con subnet routing |
| **Prometheus** | Recolección y almacenamiento de métricas |
| **Grafana** | Dashboards de monitoreo |
| **Node Exporter** | Métricas de CPU, RAM y disco |
| **cAdvisor** | Métricas de contenedores Docker |
| **Nextcloud** | Nube personal (almacenamiento, notas, calendario) |
| **Floci** | Emulador local de servicios AWS (S3, Lambda, DynamoDB, etc.) |
| **AWS CLI v2** | Interacción con servicios de Floci |
| **MySQL 8** | Base de datos para Nextcloud |
| **Redis** | Caché para Nextcloud |

---

## 🔐 Acceso Remoto — Tailscale

El acceso remoto al homelab se gestiona mediante **Tailscale**, configurado con subnet routing para exponer toda la red local (`192.168.1.0/24`) a los dispositivos autorizados.

```bash
# Anunciar subnet en el servidor
sudo tailscale up --advertise-routes=192.168.1.0/24

# Aceptar rutas en el cliente
tailscale up --accept-routes
```

**Ventajas sobre VPN tradicional:**
- NAT traversal automático
- Sin necesidad de IP pública estática
- Funciona detrás de CGNAT
- Gestión centralizada desde el panel web de Tailscale

---

## 📊 Monitoreo — Prometheus + Grafana

Stack de observabilidad completo que monitorea el host y todos los contenedores en tiempo real.

**Dashboards activos:**
- **Node Exporter Full** (ID: 1860) — CPU, RAM, disco, red del servidor
- **cAdvisor** (ID: 19792) — uso de recursos por contenedor

**Retención de datos:** 30 días

```yaml
# Acceso
Grafana:    http://192.168.1.75:3001
Prometheus: http://192.168.1.75:9090
```

---

## ☁️ Emulación AWS local — Floci

**Floci** es un emulador open-source de AWS que reemplaza a LocalStack. Permite desarrollar y probar aplicaciones que usan servicios de AWS sin costo ni dependencia de la nube real.

**Servicios disponibles:** S3, SQS, Lambda, DynamoDB, RDS, ElastiCache, API Gateway, Cognito, KMS, Step Functions, CloudFormation, EventBridge y más (45 servicios en total).

```bash
# Configuración del cliente AWS
aws configure
# Access Key: test | Secret Key: test | Region: us-east-1

# Ejemplo — crear un bucket S3
aws --endpoint-url http://192.168.1.75:4566 s3 mb s3://mi-bucket

# Subir un archivo
aws --endpoint-url http://192.168.1.75:4566 s3 cp archivo.txt s3://mi-bucket/
```

**Ventajas sobre LocalStack:**
- Sin token de autenticación (LocalStack lo requiere desde marzo 2026)
- Inicia en ~24ms vs ~3.3s de LocalStack
- 91% menos memoria en reposo (~13 MiB vs ~143 MiB)
- Licencia MIT sin restricciones

---

## ☁️ Nube Personal — Nextcloud

Alternativa self-hosted a Google Drive / Notion. Almacenamiento de archivos, notas, calendario y contactos propios sin depender de servicios externos.

```yaml
# Acceso
Nextcloud: http://192.168.1.75:8081
```

**Stack:** Nextcloud + MySQL 8 + Redis (caché)

---

## 🎮 Servidor de Minecraft

Servidor de Minecraft accesible desde la red local y a través de Tailscale.

---

## 📁 Estructura del Repositorio

```
homelab/
├── README.md
└── stacks/
    ├── monitoring/
    │   ├── docker-compose.yml
    │   └── prometheus.yml
    ├── nextcloud/
    │   └── docker-compose.yml
    ├── floci/
    │   └── docker-compose.yml
    └── minecraft/
        └── docker-compose.yml
```

---

## 💡 Aprendizajes Clave

- **Docker y contenedores:** gestión de stacks multi-servicio, volúmenes, redes y dependencias entre contenedores.
- **Redes y VPN:** diferencias entre WireGuard puro y Tailscale, NAT traversal, subnet routing, y exposición segura de servicios.
- **Observabilidad:** arquitectura de monitoreo con Prometheus (pull model), exporters y visualización en Grafana.
- **Servicios en la nube local:** emulación de AWS con Floci para desarrollar sin costos ni dependencia de internet.
- **Troubleshooting real:** diagnóstico de permisos en Linux, configuración de certificados SSL, resolución de problemas de conectividad en red.
- **Seguridad:** principio de mínima exposición — servicios internos solo accesibles por VPN, sin puertos abiertos innecesarios a internet.

---

## 🚀 Próximos Pasos

- [ ] Pipeline CI/CD con Gitea + Woodpecker
- [ ] Dominio propio con Cloudflare + NPM (subdominios con HTTPS)
- [ ] Minecraft Exporter para Prometheus
- [ ] Bot de Discord/Telegram con notificaciones del homelab
- [ ] Explorar k3s (Kubernetes ligero) en el server-m73

---

## 👤 Autor

**Santiago Rome**
Estudiante de Ingeniería de Sistemas
Medellin, Colombia