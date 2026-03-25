# Historial de cambios — MEGC

Todos los cambios relevantes de este proyecto se documentan en este archivo.

El formato sigue el estándar [Keep a Changelog](https://keepachangelog.com/es-ES/1.0.0/) y el versionado sigue [Semantic Versioning](https://semver.org/lang/es/).

---

## [1.0.0] — 2025-03-23

### Primera versión estable

Esta es la versión inicial del Modelo de Equilibrio General Computable (MEGC) implementado en Python, desarrollada como alternativa de código abierto a la versión en GAMS.

#### Funcionalidades incluidas

- Núcleo del modelo CGE con calibración automática desde la SAM
- Motor de simulación de escenarios de política configurables vía YAML
- Análisis de política fiscal: cambios tributarios, subsidios y gasto público
- Evaluación de proyectos de infraestructura y su impacto sectorial
- Generación automática de reportes en Excel y texto plano
- Ejemplos reproducibles: política fiscal e inversión en infraestructura
- Script de verificación de instalación

#### Notas técnicas

- Requiere Python 3.9 o superior
- Las bases de datos (SAM y datos sectoriales) se distribuyen por separado
- Compatible con Windows, macOS y Linux

---

## Próximas versiones (planificado)

### [1.1.0] — Fecha por confirmar

- Incorporación de módulo de dinámica temporal (modelo recursivo dinámico)
- Mejora en visualización de resultados con gráficos interactivos
- Soporte para múltiples años base

### [2.0.0] — Fecha por confirmar

- Módulo de análisis de sensibilidad automatizado
- Integración con bases de datos públicas (DANE, Banco Mundial)
- Interfaz de línea de comandos mejorada

---

*Para reportar errores o solicitar funcionalidades, utilice la sección [Issues](../../issues) del repositorio.*
