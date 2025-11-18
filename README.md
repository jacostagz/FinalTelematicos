# Proyecto de Monitoreo con Prometheus, Node Exporter y Grafana

Este proyecto implementa un sistema completo de monitoreo utilizando **Prometheus**, **Node Exporter** y **Grafana**, desplegado en dos máquinas virtuales (Servidor y Cliente) creadas con Vagrant.  
Incluye métricas del sistema, reglas de alerta, dashboards personalizados y visualizaciones en tiempo real.

---

## 📋 Tabla de Contenidos

- [Monitoreo con Prometheus y Node Exporter](#monitoreo-con-prometheus-y-node-exporter)
  - [Instalación de Prometheus](#instalación-de-prometheus-servidor)
  - [Configuración de prometheus.yml](#configuración-de-prometheusyml)
  - [Instalación de Node Exporter](#instalación-de-node-exporter-cliente)
  - [Reglas de Alerta](#reglas-de-alerta-en-prometheus)
  - [Métricas Monitoreadas](#documentación-de-métricas-monitoreadas)
- [Visualización con Grafana](#visualización-con-grafana)
  - [Instalación](#instalación-de-grafana-servidor)
  - [Dashboard Personalizado](#dashboard-creado-con-3-paneles)
- [Entrega de Resultados](#entrega-de-resultados)
- [Conclusión Técnica](#conclusión-técnica)

---

## Monitoreo con Prometheus y Node Exporter

### ✔ Instalación de Prometheus (Servidor)

Prometheus fue instalado en el servidor en la siguiente estructura de directorios:

```
/usr/local/bin/prometheus
/etc/prometheus/prometheus.yml
/etc/prometheus/alert.rules.yml
```

El servicio se habilitó manualmente usando:

```bash
sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl start prometheus
```

Prometheus quedó accesible en:

```
http://192.168.50.3:9090
```

---

### ✔ Configuración de prometheus.yml

El archivo recolecta:
- Métricas locales del propio Prometheus
- Métricas del cliente que ejecuta Node Exporter

```yaml
global:
  scrape_interval: 5s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node_exporter'
    static_configs:
      - targets: ['192.168.50.2:9100']
```

---

### ✔ Instalación de Node Exporter (Cliente)

Node Exporter fue instalado en:

```
/usr/local/bin/node_exporter
```

Servicio configurado con:

```bash
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
```

Verificado con:

```bash
curl http://localhost:9100/metrics
```

Firewall configurado:

```bash
sudo ufw allow from 192.168.50.3 to any port 9100 proto tcp
```

---

### ✔ Reglas de Alerta en Prometheus

Se creó el archivo:

```
/etc/prometheus/alert.rules.yml
```

Con una alerta básica de CPU:

```yaml
groups:
  - name: system_alerts
    rules:
      - alert: HighCPUUsage
        expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[2m])) * 100) > 80
        for: 30s
        labels:
          severity: warning
        annotations:
          summary: "Uso alto de CPU"
          description: "La CPU ha superado el 80% de uso en {{ $labels.instance }}"
```

Prometheus quedó cargando las reglas:

```bash
sudo systemctl restart prometheus
```

---

### 📊 Métricas Monitoreadas con Prometheus y Node Exporter

Para evaluar el estado y rendimiento del sistema Linux, se utilizaron tres métricas fundamentales obtenidas desde Node Exporter. Estas métricas permiten identificar comportamientos anómalos, detectar saturación de recursos y asegurar que el servidor funcione adecuadamente.

---

#### 🧠 1. CPU Usage (%)

**Métrica utilizada:**  
`node_cpu_seconds_total`

**Consulta PromQL:**
```promql
100 - (avg by(instance)(rate(node_cpu_seconds_total{mode="idle"}[2m])) * 100)
```

**Descripción:**  
Esta métrica reporta la cantidad de segundos que la CPU ha pasado en cada estado:
- *idle* (inactiva)
- *user* (procesos de usuario)
- *system* (procesos del kernel)
- *iowait*, *irq*, *softirq*, etc.

A partir de esta métrica se calcula el **porcentaje real de uso de CPU**, midiendo cuánto tiempo estuvo *ocupada* versus *inactiva*.

**Utilidad en monitoreo:**  
Permite detectar:
- Sobrecarga del procesador
- Procesos que consumen demasiado CPU
- Problemas de rendimiento bajo alta demanda

Es una de las métricas más importantes para analizar el estado del sistema.

---

#### 💾 2. Memory Usage (%)

**Métricas utilizadas:**  
`node_memory_MemAvailable_bytes`  
`node_memory_MemTotal_bytes`

**Consulta PromQL:**
```promql
(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100
```

**Descripción:**  
Estas métricas informan la cantidad total de memoria RAM y la cantidad de memoria verdaderamente disponible para nuevas aplicaciones sin necesidad de usar swap.

Se usa para calcular el **porcentaje de memoria utilizada** en el sistema.

**Utilidad en monitoreo:**  
Ayuda a identificar:
- Falta de memoria disponible
- Posibles cambios de rendimiento debido a uso de swap
- Presión de memoria que puede afectar procesos críticos

Un uso alto de memoria puede causar lentitud, swapping y degradación del servicio.

---

#### 📦 3. Disk Used (%)

**Métricas utilizadas:**  
`node_filesystem_avail_bytes`  
`node_filesystem_size_bytes`

**Consulta PromQL:**
```promql
(node_filesystem_size_bytes{fstype!="tmpfs"} - node_filesystem_free_bytes{fstype!="tmpfs"}) / node_filesystem_size_bytes{fstype!="tmpfs"} * 100
```

**Descripción:**  
Estas métricas muestran el espacio disponible y el tamaño total del disco o sistema de archivos monitoreado.

Mediante estos valores se calcula el **porcentaje de espacio utilizado en disco**.

**Utilidad en monitoreo:**  
Permite prevenir situaciones críticas como:
- Quedarse sin espacio en disco
- Fallos en servicios que requieren escritura
- Corrupción de datos y caídas del sistema

Es especialmente importante en servidores web, bases de datos, logs, y servicios que generan archivos constantemente.

---

#### 📝 Resumen de Métricas

Estas tres métricas permiten una vista completa del estado del servidor:

| Métrica | Recurso | Qué permite detectar |
|---------|---------|----------------------|
| CPU Usage (%) | Procesador | Sobrecarga, procesos pesados |
| Memory Usage (%) | RAM | Presión de memoria, uso de swap |
| Disk Used (%) | Disco | Falta de espacio, riesgo de fallos |

Con ellas, es posible mantener un monitoreo claro y confiable del rendimiento del sistema Linux utilizando **Prometheus + Node Exporter**.

---

## Visualización con Grafana

### ✔ Instalación de Grafana (Servidor)

Grafana se instaló y quedó corriendo en:

```
http://192.168.50.3:3000
Usuario: admin
Contraseña: admin
```

Servicio verificado con:

```bash
sudo systemctl status grafana-server
```

---

### ✔ Configuración de la Fuente de Datos Prometheus

En Grafana:

1. Ir a **Configuration → Data sources**
2. Seleccionar **Prometheus**
3. Configurar URL:
   ```
   http://localhost:9090
   ```
4. Guardar y verificar (OK)

---

### ✔ Dashboard Creado con 3 Paneles

#### 1. CPU Usage (%) – Gráfico de líneas

**Consulta:**
```promql
100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[2m])) * 100)
```

---

#### 2. Memory Usage – Gráfico de líneas

**Consulta:**
```promql
(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100
```

---

#### 3. Disk Usage – Gauge

**Consulta:**
```promql
(node_filesystem_size_bytes{fstype!="tmpfs"} - node_filesystem_free_bytes{fstype!="tmpfs"}) / node_filesystem_size_bytes{fstype!="tmpfs"} * 100
```

---

### ✔ Panel Preconfigurado Importado

Se importó el dashboard oficial de Grafana:

**ID: 1860** (Node Exporter Full)

Incluye:
- CPU
- RAM
- Disco
- Networking
- Load average

---

## 📁 Entrega de Resultados

Este repositorio incluye:

### ✔ Archivos de Configuración:
- `/etc/prometheus/prometheus.yml`
- `/etc/prometheus/alert.rules.yml`
- Servicios systemd del servidor y cliente

### ✔ Scripts usados en la instalación
### ✔ Dashboards exportados en JSON
### ✔ Capturas de pantalla del despliegue
### ✔ Este archivo README.md

**Para evaluación:**  
Subir el repositorio público y entregar el enlace.

---

## 📌 Conclusión Técnica

### ✔ ¿Qué aprendí al integrar Prometheus, Node Exporter y Grafana?

Aprendí cómo funciona un sistema de monitoreo moderno basado en **pull**, cómo usar Node Exporter para obtener métricas del sistema y cómo centralizarlas en Prometheus para analizarlas y crear reglas de alerta.

Grafana permitió visualizar métricas de forma clara y profesional.

---

### ✔ ¿Qué fue lo más desafiante y cómo lo resolvería?

**Lo más desafiante fue:**
- Configurar correctamente los servicios systemd
- Exponer puertos entre servidor y cliente
- Lograr que Prometheus detectara el Node Exporter

---

### ✔ ¿Qué beneficio aporta la observabilidad en DevOps?

La observabilidad permite:
- Detectar fallas antes de que afecten usuarios
- Medir rendimiento real del sistema
- Tomar decisiones basadas en datos
- Reducir tiempo de diagnóstico (MTTR)

**Es esencial en infraestructura moderna.**

---

## 🚀 Autor

Krsna Gutierrez

**Proyecto Parcial**  
Implementación de Prometheus, Node Exporter y Grafana
