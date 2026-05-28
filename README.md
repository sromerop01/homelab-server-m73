# 🖥️ Homelab — server-m73

> Infraestructura personal de servidores sobre un Lenovo ThinkCentre M73, orientada al aprendizaje de DevOps, redes, monitoreo y servicios en la nube local.

---

## 📦 Hardware

| Componente | Detalle |
|---|---|
| **Máquina** | Lenovo ThinkCentre M73 |
| **CPU** | Intel Core i7-4790 @ 3.60GHz (4 núcleos / 8 hilos) |
| **RAM** | 12 GB — 9.8 GB disponibles en reposo |
| **Almacenamiento** | 457 GB — 100 GB usados (23%) |
| **Sistema Operativo** | Ubuntu 24.04.4 LTS (Noble Numbat) |
| **Kernel** | 6.8.0-117-generic |
| **Red** | IP pública + Tailscale VPN mesh |

---

## 🏗️ Arquitectura General

```
Internet / Red local
        │
        ▼
  Tailscale VPN mesh
  (subnet routing 192.168.1.0/24 — exit node)
        │
        ▼
  server-m73 (192.168.1.75)
        │
        ├── Nginx Proxy Manager  (reverse proxy, SSL/TLS — puertos 80, 443, 81)
        │
        ├── Portainer            (gestión de contenedores)
        ├── Homepage             (dashboard de servicios — puerto 3000)
        │
        ├── Prometheus           (métricas — puerto 9090)
        ├── Grafana              (dashboards — puerto 3001)
        ├── Node Exporter        (métricas del host)
        ├── cAdvisor             (métricas de contenedores)
        │
        ├── Nextcloud            (nube personal — puerto 8081)
        ├── Floci                (emulador AWS — puerto 4566)
        ├── API REST Laravel     (backend con MySQL 8.4)
        ├── Open WebUI + Ollama  (LLMs locales — detenido)
```

---

## 🛠️ Stack de Tecnologías

### Containerización y Orquestación

| Tecnología | Versión | Uso |
|---|---|---|
| **Docker CE** | 29.2.1 | Motor de contenedores |
| **Docker Compose** | v5 | Orquestación de stacks multi-servicio |
| **Portainer CE** | latest | Gestión visual de contenedores vía web |

### Red y Acceso Remoto

| Tecnología | Uso |
|---|---|
| **Tailscale** | VPN mesh — subnet routing, exit node, acceso desde macOS / Android / Windows |
| **Nginx Proxy Manager** | Reverse proxy con SSL/TLS automático |

### Monitoreo y Observabilidad

| Tecnología | Uso |
|---|---|
| **Prometheus** | Recolección y almacenamiento de métricas (pull model) |
| **Grafana** | Dashboards de visualización |
| **Node Exporter** | Métricas del host: CPU, RAM, disco, red |
| **cAdvisor** | Métricas de contenedores Docker en tiempo real |

### Servicios Desplegados

| Servicio | Stack | Puerto |
|---|---|---|
| **Homepage** | Dashboard | 3000 |
| **Nextcloud** | Nextcloud + MySQL 8 + Redis | 8081 |
| **Floci** | Emulador AWS | 4566 |
| **API REST Laravel** | Laravel + MySQL 8.4 | — |
| **Open WebUI + Ollama** | LLMs locales | detenido |

---

## 🔐 Acceso Remoto — Tailscale

Tailscale está configurado como **exit node** y con **subnet routing**, permitiendo acceso completo a la red local desde cualquier dispositivo autorizado (macOS, Android, Windows).

```bash
# Anunciar subnet y habilitar exit node en el servidor
sudo tailscale up --advertise-routes=192.168.1.0/24 --advertise-exit-node

# Aceptar rutas en el cliente
tailscale up --accept-routes
```

**Dispositivos conectados:** MacBook, Android, Windows, server-m73

---

## 📊 Monitoreo — Prometheus + Grafana

Stack de observabilidad completo que monitorea el host y todos los contenedores en tiempo real.

**Dashboards activos:**
- **Node Exporter Full** (ID: 1860) — CPU, RAM, disco, red del servidor
- **cAdvisor** (ID: 19792) — uso de recursos por contenedor

**Retención de datos:** 30 días

```
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

# Crear un bucket S3
aws --endpoint-url http://192.168.1.75:4566 s3 mb s3://mi-bucket

# Subir un archivo
aws --endpoint-url http://192.168.1.75:4566 s3 cp archivo.txt s3://mi-bucket/
```

**Por qué Floci sobre LocalStack:**
- Sin token de autenticación (LocalStack lo requiere desde marzo 2026)
- Inicia en ~24ms vs ~3.3s de LocalStack
- 91% menos memoria en reposo (~13 MiB vs ~143 MiB)
- Licencia MIT sin restricciones

---

## ☁️ Nube Personal — Nextcloud

Alternativa self-hosted a Google Drive. Almacenamiento de archivos, notas, calendario y contactos propios sin depender de servicios externos.

```
Nextcloud: http://192.168.1.75:8081
```

**Stack:** Nextcloud + MySQL 8 + Redis (caché)

---

## 📁 Estructura del Repositorio

```
homelab/
├── README.md
├── .gitignore
└── stacks/
    ├── monitoring/
    │   ├── docker-compose.yml
    │   └── prometheus.yml
    ├── nextcloud/
    │   └── docker-compose.yml
    └── floci/
        └── docker-compose.yml
```

---

## 💡 Aprendizajes Clave

- **Docker y contenedores:** gestión de stacks multi-servicio, volúmenes, redes internas y dependencias entre contenedores.
- **Redes y VPN:** configuración de Tailscale como exit node y subnet router, NAT traversal, y exposición segura de servicios.
- **Observabilidad:** arquitectura de monitoreo con Prometheus (pull model), exporters y visualización en Grafana.
- **Reverse proxy:** configuración de Nginx Proxy Manager con SSL/TLS para enrutar tráfico por subdominios.
- **Servicios en la nube local:** emulación de 45 servicios AWS con Floci para desarrollar sin costos ni dependencia de internet.
- **Backend:** despliegue de API REST con Laravel + MySQL en entorno containerizado.
- **Troubleshooting:** diagnóstico de permisos en Linux, certificados SSL, resolución de problemas de conectividad en red.
- **Seguridad:** principio de mínima exposición — servicios internos accesibles solo por VPN.

---

## 🚀 Próximos Pasos

- [ ] Pipeline CI/CD con Gitea + Woodpecker
- [ ] Dominio propio con Cloudflare + NPM (subdominios con HTTPS)
- [ ] Minecraft Exporter para Prometheus
- [ ] Bot de Discord/Telegram con notificaciones del homelab
- [ ] Explorar k3s (Kubernetes ligero) en el server-m73
- [ ] Activar Open WebUI + Ollama para LLMs locales

---

## 👤 Autor

**Santiago Rome**
Estudiante de Ingeniería de Sistemas
Medellin, Colombia