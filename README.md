# MEGC — Modelo de Equilibrio General Computable en Python

[![Python](https://img.shields.io/badge/Python-3.9%2B-blue)](https://www.python.org/)
[![Licencia](https://img.shields.io/badge/Licencia-MIT-green)](LICENSE)
[![Estado](https://img.shields.io/badge/Estado-v1.0.0-brightgreen)](CHANGELOG.md)

Implementación en Python de un Modelo de Equilibrio General Computable (MEGC) orientado al **análisis de políticas económicas** y la **evaluación de proyectos de infraestructura**. Desarrollado como alternativa de código abierto.

---

## Tabla de contenidos

1. [¿Qué hace este modelo?](#qué-hace-este-modelo)
2. [Requisitos del sistema](#requisitos-del-sistema)
3. [Instalación](#instalación)
4. [Estructura del proyecto](#estructura-del-proyecto)
5. [Datos de entrada: SAM y bases de datos](#datos-de-entrada-sam-y-bases-de-datos)
6. [Cómo ejecutar el modelo](#cómo-ejecutar-el-modelo)
7. [Interpretación de resultados](#interpretación-de-resultados)
8. [Ejemplos incluidos](#ejemplos-incluidos)
9. [Equipo y contacto](#equipo-y-contacto)

---

## ¿Qué hace este modelo?

El MEGC permite simular los efectos económicos de intervenciones de política, incluyendo:

- **Análisis de política fiscal**: impacto de cambios tributarios, subsidios o gasto público sobre sectores productivos, hogares y empleo.
- **Evaluación de proyectos de infraestructura**: estimación de efectos macroeconómicos y sectoriales de inversiones en transporte, energía, conectividad, etc.
- **Shocks externos**: variaciones en precios internacionales, tipo de cambio o demanda agregada.

El modelo parte de una **Matriz de Contabilidad Social (SAM)** calibrada para la economía antioqueña y resuelve el sistema de equilibrio general mediante optimización numérica en Python.

---

## Requisitos del sistema

| Componente | Versión mínima |
|---|---|
| Python | 3.9 o superior |
| pip | 21.0 o superior |
| RAM recomendada | 8 GB |
| Sistema operativo | Windows 10, macOS 12, Ubuntu 20.04 |

### Dependencias de Python

Todas las dependencias se instalan automáticamente (ver sección de instalación):

```
numpy >= 1.23
scipy >= 1.9
pandas >= 1.5
openpyxl >= 3.0
matplotlib >= 3.6
```

---

## Instalación

### Paso 1: Clonar el repositorio

```bash
git clone https://github.com/tu-usuario/MEGC-modelo.git
cd MEGC-modelo
```

### Paso 2: Crear un entorno virtual (recomendado)

```bash
# En Windows
python -m venv venv
venv\Scripts\activate

# En macOS / Linux
python -m venv venv
source venv/bin/activate
```

### Paso 3: Instalar dependencias

```bash
pip install -r requirements.txt
```

### Paso 4: Verificar la instalación

```bash
python src/verificar_instalacion.py
```

Si todo está correcto, verás el mensaje: `✓ MEGC instalado correctamente`.

---

## Estructura del proyecto

```
MEGC-modelo/
│
├── src/                        # Código fuente del modelo
│   ├── modelo_megc.py          # Núcleo del modelo CGE
│   ├── calibracion.py          # Rutinas de calibración con la SAM
│   ├── simulacion.py           # Motor de simulación de escenarios
│   ├── resultados.py           # Generación de reportes y tablas
│   └── verificar_instalacion.py
│
├── config/                     # Parámetros de configuración
│   ├── parametros_base.yaml    # Parámetros por defecto del modelo
│   └── escenarios/             # Definición de escenarios de política
│       ├── escenario_base.yaml
│       └── ejemplo_infraestructura.yaml
│
├── docs/                       # Documentación técnica
│   ├── manual_usuario.md       # Guía de uso detallada
│   ├── descripcion_modelo.md   # Fundamentos teóricos
│   └── interpretacion_resultados.md
│
├── examples/                   # Casos de uso reproducibles
│   ├── ejemplo_politica_fiscal.py
│   └── ejemplo_infraestructura.py
│
├── requirements.txt            # Dependencias de Python
├── CHANGELOG.md                # Historial de versiones
└── README.md                   # Este archivo
```

> **Nota:** Las bases de datos (SAM y datos sectoriales) **no están incluidas** en este repositorio. Son proporcionadas por el equipo técnico de forma separada. Ver sección siguiente.

---

## Datos de entrada: SAM y bases de datos

El modelo requiere archivos de datos externos que **no forman parte del repositorio** por razones de confidencialidad y tamaño. Estos archivos son entregados por el equipo técnico responsable de su construcción.

### Archivos requeridos

| Archivo | Formato | Descripción |
|---|---|---|
| `SAM_base.xlsx` | Excel | Matriz de Contabilidad Social del año base |
| `elasticidades.xlsx` | Excel | Parámetros de elasticidad por sector |
| `datos_sectoriales.xlsx` | Excel | Datos complementarios de producción y empleo |

### Dónde ubicarlos

Coloque los archivos recibidos en la carpeta `data/` (créela si no existe):

```
MEGC-modelo/
└── data/               ← Cree esta carpeta y coloque los archivos aquí
    ├── SAM_base.xlsx
    ├── elasticidades.xlsx
    └── datos_sectoriales.xlsx
```

Si los archivos tienen nombres distintos, actualice las rutas en `config/parametros_base.yaml`.

---

## Cómo ejecutar el modelo

### Ejecución básica (escenario base)

```bash
python src/simulacion.py --escenario config/escenarios/escenario_base.yaml
```

### Ejecución de un escenario de política

```bash
python src/simulacion.py --escenario config/escenarios/ejemplo_infraestructura.yaml --output resultados/
```

### Opciones disponibles

| Opción | Descripción |
|---|---|
| `--escenario` | Ruta al archivo YAML del escenario |
| `--output` | Carpeta donde se guardan los resultados (por defecto: `resultados/`) |
| `--verbose` | Muestra información detallada del proceso de solución |
| `--graficos` | Genera gráficos automáticamente al finalizar |

---

## Interpretación de resultados

El modelo genera automáticamente una carpeta de resultados con:

- `equilibrio_macroeconomico.xlsx` — PIB, consumo, inversión, balanza comercial
- `resultados_sectoriales.xlsx` — Producción, empleo y precios por sector
- `distribucion_hogares.xlsx` — Impacto en ingreso por tipo de hogar
- `reporte_simulacion.txt` — Resumen ejecutivo del escenario simulado

Para una guía detallada de interpretación, consulte [`docs/interpretacion_resultados.md`](docs/interpretacion_resultados.md).

---

## Ejemplos incluidos

En la carpeta `examples/` encontrará dos casos de uso completamente documentados:

1. **`ejemplo_politica_fiscal.py`** — Simulación de un cambio en la tasa del IVA y su efecto sobre el consumo de los hogares y la recaudación.
2. **`ejemplo_infraestructura.py`** — Evaluación de una inversión en infraestructura vial y su impacto sobre la productividad sectorial y el empleo.

Ejecute cualquiera de ellos con:

```bash
python examples/ejemplo_politica_fiscal.py
```

---

## Equipo y contacto

Este modelo fue desarrollado por **[Diego Quintero / Universidad Eafit]**.

Para soporte técnico, reporte de errores o solicitud de los archivos de datos:

- **Correo:** daquinterr@eafit.edu.co
- **Issues:** Utilice la pestaña [Issues](../../issues) de este repositorio para reportar problemas

---

*Desarrollado en Python como alternativa de código abierto.*
