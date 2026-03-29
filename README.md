# 🏠 Homelab Prod

![Public Docs](https://img.shields.io/badge/Public%20Docs-Yes-0A66C2?style=for-the-badge)
![Sanitized](https://img.shields.io/badge/Sanitized-Yes-2E8B57?style=for-the-badge)
![Bilingual](https://img.shields.io/badge/Bilingual-ES%20%7C%20EN-6A5ACD?style=for-the-badge)
![Infrastructure](https://img.shields.io/badge/Focus-Infrastructure-444444?style=for-the-badge)
![Security](https://img.shields.io/badge/Focus-Security-B22222?style=for-the-badge)

> **Documentación pública, profesional y sanitizada** de un homelab orientado a  
> **infraestructura · segmentación · seguridad · operación · observabilidad · recuperación**

---

## ✨ ¿Qué muestra este repositorio?

Este repositorio no busca impresionar por cantidad de herramientas.  
Busca demostrar algo más importante:

- **criterio de arquitectura**
- **segmentación de red con propósito**
- **controles de acceso razonables**
- **operación repetible**
- **observabilidad útil**
- **backups con enfoque de recuperación**
- **documentación defendible en entrevista técnica**

En otras palabras: **menos humo, más criterio**.

---

## 🎯 Objetivo

Este proyecto representa un entorno de laboratorio personal diseñado para practicar conceptos reales de:

- **infraestructura**
- **ciberseguridad**
- **networking**
- **monitoreo**
- **continuidad operativa**
- **documentación técnica**

La versión pública está pensada para mostrar **cómo está diseñado**, **cómo se opera** y **cómo se recupera**, sin exponer información sensible.

---

## 🧭 Principios del proyecto

| Principio | Enfoque |
|---|---|
| 🛡️ Seguridad por diseño | Minimizar exposición y reducir riesgo desde la arquitectura |
| ⚙️ Simplicidad operativa | Mantener un entorno entendible, operable y mantenible |
| 📊 Observabilidad | Obtener visibilidad útil, no solo dashboards decorativos |
| 🔒 Aislamiento de servicios | Separar roles y reducir lateralidad innecesaria |
| 📈 Capacidad de crecimiento | Permitir evolución sin perder control del entorno |

---

## 📚 Alcance del repositorio

### ✅ Incluye
- resumen ejecutivo del entorno
- arquitectura y segmentación
- modelo de seguridad y accesos
- runbook operativo público
- estrategia de backup y recuperación
- observabilidad y roadmap

### 🚫 No incluye
- secretos
- credenciales
- tokens
- claves
- endpoints reales de acceso remoto
- configuraciones sensibles completas
- logs o capturas que identifiquen el entorno real

---

## 🗂️ Mapa de lectura

### 🇪🇸 Español
- [01 · Resumen ejecutivo](docs/es/01-resumen-ejecutivo.md)
- [02 · Arquitectura y red](docs/es/02-arquitectura-y-red.md)
- [03 · Seguridad y accesos](docs/es/03-seguridad-y-accesos.md)
- [04 · Runbook operativo](docs/es/04-runbook-operativo.md)
- [05 · Backup y recuperación](docs/es/05-backup-y-recuperacion.md)
- [06 · Observabilidad y roadmap](docs/es/06-observabilidad-y-roadmap.md)

### 🇬🇧 English
- [01 · Executive summary](docs/en/01-executive-summary.md)
- [02 · Architecture and network](docs/en/02-architecture-and-network.md)
- [03 · Security and access](docs/en/03-security-and-access.md)
- [04 · Operations runbook](docs/en/04-operations-runbook.md)
- [05 · Backup and recovery](docs/en/05-backup-and-recovery.md)
- [06 · Observability and roadmap](docs/en/06-observability-and-roadmap.md)

---

## 🏗️ ¿Qué tipo de proyecto es?

Este homelab está documentado como un entorno que prioriza:

- **virtualización**
- **segmentación lógica**
- **DNS interno centralizado**
- **controles de acceso**
- **observabilidad**
- **backups y recuperación**
- **operación repetible**

No es un repositorio de instalación paso a paso de cada herramienta.  
Es un repositorio que busca mostrar **criterio técnico**, **orden operativo** y **madurez documental**.

---

## 🔎 Qué debería ver una persona técnica acá

Este repositorio está organizado para que, al recorrerlo, se entienda con rapidez:

1. **qué entorno se construyó**
2. **cómo está segmentado**
3. **cómo se controla el acceso**
4. **cómo se opera**
5. **cómo se respalda**
6. **cómo se piensa su evolución**

---

## 🧰 Estructura del repositorio

```text
Homelab/
├── README.md
├── LICENSE.md
├── docs/
│   ├── es/
│   │   ├── 01-resumen-ejecutivo.md
│   │   ├── 02-arquitectura-y-red.md
│   │   ├── 03-seguridad-y-accesos.md
│   │   ├── 04-runbook-operativo.md
│   │   ├── 05-backup-y-recuperacion.md
│   │   └── 06-observabilidad-y-roadmap.md
│   └── en/
│       ├── 01-executive-summary.md
│       ├── 02-architecture-and-network.md
│       ├── 03-security-and-access.md
│       ├── 04-operations-runbook.md
│       ├── 05-backup-and-recovery.md
│       └── 06-observability-and-roadmap.md
```