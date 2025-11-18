## 📊 Métricas monitoreadas con Prometheus y Node Exporter

Para evaluar el estado y rendimiento del sistema Linux, se utilizaron tres métricas fundamentales obtenidas desde Node Exporter. Estas métricas permiten identificar comportamientos anómalos, detectar saturación de recursos y asegurar que el servidor funcione adecuadamente.

---

### 🧠 1. CPU Usage (%)

**Métrica utilizada:**  
`node_cpu_seconds_total`

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

### 💾 2. Memory Usage (%)

**Métrica utilizada:**  
`node_memory_MemAvailable_bytes`  
`node_memory_MemTotal_bytes`

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

### 📦 3. Disk Used (%)

**Métrica utilizada:**  
`node_filesystem_avail_bytes`  
`node_filesystem_size_bytes`

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

## 📝 Resumen

Estas tres métricas permiten una vista completa del estado del servidor:

| Métrica | Recurso | Qué permite detectar |
|--------|----------|-----------------------|
| CPU Usage (%) | Procesador | Sobrecarga, procesos pesados |
| Memory Usage (%) | RAM | Presión de memoria, uso de swap |
| Disk Used (%) | Disco | Falta de espacio, riesgo de fallos |

Con ellas, es posible mantener un monitoreo claro y confiable del rendimiento del sistema Linux utilizando **Prometheus + Node Exporter**.

