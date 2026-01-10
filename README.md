# ☁️ Infraestructura Productiva a Costo Cero en GCP Free Tier

> Un caso de estudio sobre cómo optimizar recursos limitados para desplegar aplicaciones complejas en la nube sin incurrir en costos.

![Diagrama de Arquitectura](enlace_al_diagrama_que_genere_arriba.png)

## 🎯 El Desafío

El objetivo era desplegar un entorno de servidor propio en la nube para alojar proyectos personales, con una restricción estricta: **mantenerse 100% dentro de la Capa Gratuita (Free Tier) de Google Cloud Platform.**

El reto técnico principal fue ejecutar un **bot de Python que utiliza Selenium (Chromium Headless)**. Chromium es conocido por su alto consumo de memoria, y la instancia gratuita de GCP (`e2-micro`) solo ofrece **1 GB de RAM**, lo cual es insuficiente para esta tarea por defecto.

## 🏗️ La Solución Arquitectónica

Se diseñó una arquitectura basada en **microservicios contenerizados** sobre una máquina virtual Linux altamente optimizada.

### Componentes Clave

| Componente | Elección Tecnológica | Justificación |
| :--- | :--- | :--- |
| **Cloud Provider** | GCP Compute Engine | Uso de la instancia `e2-micro` elegible para el Free Tier en `us-central1`. |
| **OS** | Debian 12 (Bookworm) | Menor consumo de recursos base comparado con Ubuntu. |
| **Orquestación** | Docker & Docker Compose | Aislamiento de servicios, gestión de dependencias y reinicio automático (`restart: unless-stopped`). |
| **Almacenamiento** | Persistent Disk (30GB) | Maximización del almacenamiento gratuito permitido. |
| **Red** | VPC Firewall | Reglas estrictas permitiendo solo tráfico HTTP (80) y SSH (22). |

## 🛠️ Ingeniería y Optimizaciones (Lo más difícil)

Para hacer viable este entorno con recursos tan limitados, se aplicaron técnicas de ingeniería de sistemas:

### 1. Gestión de Memoria (The "Swap Trick")
Con 1GB de RAM, el proceso de construcción de la imagen de Docker con Chromium fallaba por *Out Of Memory (OOM)*.

* **Solución:** Se implementó un archivo de intercambio (**Swap File**) de **2 GB** en el disco persistente.
* **Ajuste:** Se configuró `vm.swappiness=10` en el kernel para priorizar el uso de la RAM real y usar el disco solo cuando sea absolutamente necesario, evitando la degradación excesiva del rendimiento.

### 2. Servicios Desplegados en el "Monorepo"

Se utiliza un enfoque de repositorio único para gestionar la infraestructura con `docker-compose.yml`.

* **Servicio A: Memos (Self-Hosted Notes)**
    * Alternativa open-source a Notion.
    * Expuesto directamente al puerto 80 para acceso web mediante IP pública.
    * Datos persistentes en volumen de Docker.

* **Servicio B: Subte Alerta Bot (Worker)**
    * Bot de Python que monitorea el estado del subte de Buenos Aires vía web scraping (Selenium) y notifica por Telegram.
    * Se ejecuta en segundo plano (headless) sin exponer puertos.
    * **Optimización de Logs:** Se configuró la rotación de logs de Docker (`max-size: "10m"`, `max-file: "3"`) para evitar que la salida de Selenium llene el disco de 30GB con el tiempo.

## 📈 Resultados y Evidencia

El despliegue fue exitoso. El servidor opera 24/7 de manera estable, manejando la carga de trabajo de Selenium gracias a la gestión de memoria virtual.

**Evidencia 1: El proceso de construcción exitoso (a pesar de tomar +19 minutos por el throttling de CPU):**

![Build Exitoso](enlace_a_tu_imagen_b43355.jpg)

**Evidencia 2: Estado de los servicios en producción:**

![Servicios UP](enlace_a_tu_imagen_a9528e.jpg)

---
*Este repositorio documenta la infraestructura. El código fuente de los servicios se mantiene en repositorios privados.*
