# Reporte de Rendimiento y Benchmarks de pulso

Este documento detalla el consumo de recursos, los requisitos mínimos de hardware y el impacto operativo del agente de monitoreo **pulso**. Al ser un software diseñado para ejecutarse en segundo plano (daemon), su arquitectura se enfoca en la eficiencia extrema para garantizar un impacto nulo sobre los servicios principales del sistema.

---

## Consumo de recursos del agente

Bajo condiciones normales de operación en entornos de producción Linux, el comportamiento del agente se mantiene dentro de los siguientes parámetros estables:

* **CPU Promedio:** < 0.5% de un solo núcleo en procesadores modernos x86_64 / ARM64.
* **Memoria RAM:** Entre 12 MB y 18 MB de memoria RSS (Resident Set Size). La fluctuación depende del número de métricas activas en los formateadores.
* **Ancho de banda de red:** < 2 KB/s durante los intervalos de raspado de métricas (scraping). El tráfico es local si Prometheus reside en la misma instancia.

---

## Requisitos mínimos de hardware

Para garantizar el correcto despliegue de **pulso** sin comprometer la estabilidad del sistema anfitrión, se especifican las siguientes especificaciones base:

| Componente | Requisito Mínimo | Notas |
| :--- | :--- | :--- |
| **CPU** | 1 núcleo virtual (vCPU) a 1.0 GHz | Compatible con arquitecturas x86_64, i386 y ARMv7/ARM64 |
| **Memoria RAM** | 32 MB libres dedicados | El espacio incluye el búfer interno para formateadores |
| **Almacenamiento**| 15 MB de espacio en disco | Espacio para el binario estático y archivo `pulso.toml` |
| **Sistema Operativo**| Linux Kernel 4.15 o superior | Soporte nativo para lectura de `/proc` y `/sys` |

---

## Impacto por intervalo de muestreo

El consumo de ciclos de CPU está directamente ligado a la frecuencia con la que el agente lee los archivos del sistema operativo. A continuación se detallan las mediciones reales obtenidas en un procesador Intel Xeon de 2.4 GHz:

| interval_ms | Intervalo Equivalente | Consumo CPU Promedio | Impacto en RAM |
| :--- | :--- | :--- | :--- |
| `1000ms` | 1 segundo | 1.2% | 16.2 MB |
| `5000ms` | 5 segundos | 0.4% | 15.8 MB |
| `10000ms`| 10 segundos | 0.1% | 15.5 MB |
| `30000ms`| 30 segundos | < 0.05% | 15.5 MB |

---

## Cómo medir el rendimiento del agente

Para verificar el consumo en tiempo real del agente en tu propia distribución, puedes ejecutar el comando interactivo `top` o `htop` filtrando directamente por el nombre del proceso. 

Ejecute la siguiente línea en su terminal para aislar las métricas del proceso:

```bash
top -b -n 1 | grep pulso

