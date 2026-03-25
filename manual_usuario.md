# Manual de usuario — MEGC

**Modelo de Equilibrio General Computable en Python**  
Versión 1.0.0 — Para análisis de políticas y proyectos de infraestructura

---

## Índice

1. [Introducción al modelo](#1-introducción-al-modelo)
2. [Preparación de los datos de entrada](#2-preparación-de-los-datos-de-entrada)
3. [Configuración de un escenario](#3-configuración-de-un-escenario)
4. [Ejecución del modelo](#4-ejecución-del-modelo)
5. [Lectura e interpretación de resultados](#5-lectura-e-interpretación-de-resultados)
6. [Casos de uso frecuentes](#6-casos-de-uso-frecuentes)
7. [Supuestos del modelo y limitaciones](#7-supuestos-del-modelo-y-limitaciones)
8. [Solución de problemas comunes](#8-solución-de-problemas-comunes)

---

## 1. Introducción al modelo

El MEGC es una herramienta de análisis económico que simula el funcionamiento completo de una economía: cómo interactúan productores, consumidores, gobierno y sector externo, y cómo esa interacción cambia cuando se introduce una política o un proyecto.

A diferencia de modelos de equilibrio parcial (que analizan un sector en aislamiento), el MEGC captura los **efectos indirectos y de segunda vuelta**: por ejemplo, una inversión en vías no solo afecta al sector construcción, sino también al transporte, la agricultura, el comercio y el empleo en toda la economía.

### ¿Cuándo usar este modelo?

El MEGC es apropiado para responder preguntas como:

- ¿Cuál es el impacto macroeconómico de una reforma tributaria sobre el PIB y el empleo?
- ¿Cómo afecta una inversión en infraestructura eléctrica a la competitividad de los sectores productivos?
- ¿Qué sectores ganan y cuáles pierden ante un shock de precios internacionales?
- ¿Cómo se distribuye el impacto de una política entre distintos tipos de hogares?

---

## 2. Preparación de los datos de entrada

### 2.1 La Matriz de Contabilidad Social (SAM)

La SAM es el punto de partida del modelo. Representa el flujo circular de la economía en un año base: quién produce qué, quién consume qué, cómo se distribuyen los ingresos y cómo interactúa la economía con el exterior.

**Formato requerido del archivo `SAM_base.xlsx`:**

- Primera fila y primera columna: nombres de las cuentas (sectores, factores, instituciones)
- Valores en millones de pesos corrientes del año base
- La SAM debe estar balanceada: la suma de cada fila debe ser igual a la suma de su columna correspondiente

Si detecta desbalances, el modelo lo notificará con el mensaje `⚠ SAM desbalanceada` al iniciar. En ese caso, contacte al equipo técnico.

### 2.2 Parámetros de elasticidad

El archivo `elasticidades.xlsx` contiene los parámetros de sustitución y transformación que determinan cuán flexibles son los agentes económicos ante cambios de precios. Estos valores fueron estimados o calibrados por el equipo técnico y **no deben modificarse** salvo para análisis de sensibilidad específicos.

### 2.3 Verificación de los datos

Antes de ejecutar cualquier simulación, verifique que los archivos están correctamente ubicados:

```bash
python src/verificar_instalacion.py
```

La salida esperada es:

```
✓ Python 3.x detectado
✓ Dependencias instaladas
✓ SAM_base.xlsx encontrada y legible
✓ elasticidades.xlsx encontrada y legible
✓ MEGC listo para ejecutar
```

---

## 3. Configuración de un escenario

Un escenario define qué política o proyecto desea simular. Se configura mediante un archivo YAML en la carpeta `config/escenarios/`.

### 3.1 Estructura de un archivo de escenario

```yaml
# config/escenarios/mi_escenario.yaml

nombre: "Aumento del IVA general al 21%"
descripcion: "Simulación del impacto de elevar la tasa general del IVA"
año_base: 2022

# Tipo de cierre macroeconómico
cierre:
  ahorro_inversion: "ahorro_dirigido"   # ahorro_dirigido | inversion_dirigida
  mercado_laboral: "salario_flexible"   # salario_flexible | desempleo
  sector_externo: "tipo_cambio_flexible"

# Perturbaciones (la política que se simula)
perturbaciones:
  impuestos:
    IVA_general: 0.21         # Nueva tasa (era 0.19)
  
  gasto_publico:
    infraestructura_vial: 0   # Sin cambio en este escenario

# Opciones de reporte
reporte:
  tablas_excel: true
  grafico_sectorial: true
  resumen_texto: true
```

### 3.2 Tipos de perturbaciones disponibles

| Categoría | Variable | Descripción |
|---|---|---|
| `impuestos` | `IVA_general`, `IVA_reducido`, `renta_empresas` | Tasas impositivas (valor entre 0 y 1) |
| `gasto_publico` | `infraestructura_vial`, `infraestructura_energetica`, `educacion`, `salud` | Variación porcentual respecto al año base |
| `precios_externos` | `exportaciones`, `importaciones` | Índice de precios (1.0 = sin cambio) |
| `productividad` | Nombre del sector | Factor multiplicativo de productividad total |

### 3.3 Opciones de cierre macroeconómico

El cierre define qué variables se ajustan para que el modelo alcance el equilibrio. La elección del cierre depende del contexto de política:

- **`ahorro_dirigido`**: La inversión se ajusta al ahorro disponible. Apropiado para análisis de corto plazo.
- **`inversion_dirigida`**: El ahorro se ajusta a una inversión exógena. Apropiado para evaluar proyectos de inversión pública.
- **`salario_flexible`**: El salario se ajusta hasta que el mercado laboral se vacía (pleno empleo). Apropiado para largo plazo.
- **`desempleo`**: El salario es rígido y el desempleo varía. Apropiado para análisis de corto plazo con rigideces.

---

## 4. Ejecución del modelo

### 4.1 Comando básico

```bash
python src/simulacion.py --escenario config/escenarios/mi_escenario.yaml
```

### 4.2 Con carpeta de salida personalizada

```bash
python src/simulacion.py \
  --escenario config/escenarios/mi_escenario.yaml \
  --output resultados/politica_iva_2025/ \
  --verbose \
  --graficos
```

### 4.3 Qué ocurre al ejecutar

El modelo realiza los siguientes pasos automáticamente:

1. **Carga de datos**: lee la SAM y los parámetros de elasticidad
2. **Calibración**: ajusta los parámetros del modelo para replicar el año base
3. **Aplicación de la perturbación**: introduce el cambio de política definido en el YAML
4. **Solución del sistema**: encuentra el nuevo equilibrio mediante iteración numérica
5. **Cálculo de variaciones**: compara el nuevo equilibrio con el año base
6. **Generación de reportes**: exporta los resultados a Excel y texto

El proceso tarda típicamente entre 30 segundos y 3 minutos dependiendo del tamaño de la SAM y la complejidad del escenario.

---

## 5. Lectura e interpretación de resultados

### 5.1 Archivos generados

Al finalizar, encontrará en la carpeta de salida:

| Archivo | Contenido |
|---|---|
| `equilibrio_macroeconomico.xlsx` | PIB, consumo privado, inversión, exportaciones, importaciones, índice de precios |
| `resultados_sectoriales.xlsx` | Producción, valor agregado, empleo y precios por sector |
| `distribucion_hogares.xlsx` | Variación en el ingreso real por tipo de hogar |
| `reporte_simulacion.txt` | Resumen ejecutivo con los principales hallazgos |

### 5.2 Cómo leer las variaciones

Todos los resultados se expresan como **variación porcentual respecto al escenario base** (año base sin perturbación), excepto donde se indica explícitamente.

Ejemplo de lectura:

```
Sector construcción:  +3.2%   →  La producción del sector aumenta 3.2%
Sector agropecuario: -0.8%   →  La producción del sector cae 0.8%
PIB real:            +1.1%   →  El PIB total crece 1.1%
```

### 5.3 Indicadores macroeconómicos clave

| Indicador | Interpretación |
|---|---|
| **PIB real** | Medida agregada del efecto sobre la actividad económica |
| **Índice de precios al consumidor** | Si sube, la política es inflacionaria |
| **Tipo de cambio real** | Si deprecia, mejora la competitividad exportadora |
| **Balanza comercial** | Variación en el saldo de exportaciones menos importaciones |
| **Recaudación fiscal** | Cambio en los ingresos del gobierno |

### 5.4 Resultados sectoriales

La hoja `resultados_sectoriales.xlsx` muestra para cada sector:

- **Producción**: volumen físico de bienes y servicios generado
- **Valor agregado**: contribución del sector al PIB
- **Empleo**: número de empleos equivalentes (en miles)
- **Precio del bien doméstico**: índice de precios del sector

Los sectores con mayor variación positiva son los principales ganadores de la política simulada; los de mayor variación negativa, los más afectados.

---

## 6. Casos de uso frecuentes

### Caso 1: Evaluar una reforma tributaria

1. Copie `config/escenarios/escenario_base.yaml` y renómbrelo
2. En la sección `perturbaciones > impuestos`, cambie la tasa correspondiente
3. En `cierre`, use `ahorro_dirigido` y `salario_flexible` para análisis de largo plazo
4. Ejecute y compare el impacto en `equilibrio_macroeconomico.xlsx` y `distribucion_hogares.xlsx`

### Caso 2: Evaluar un proyecto de infraestructura

1. En la sección `perturbaciones > gasto_publico`, aumente la categoría correspondiente (ej. `infraestructura_vial: 0.15` para un aumento del 15%)
2. En `perturbaciones > productividad`, puede incluir la mejora de productividad esperada del proyecto en los sectores beneficiados
3. Use el cierre `inversion_dirigida` para modelar que el proyecto es exógeno al ahorro privado
4. Los resultados en `resultados_sectoriales.xlsx` mostrarán qué sectores se benefician más

### Caso 3: Análisis de sensibilidad

Para verificar la robustez de los resultados ante cambios en parámetros:

```bash
# Ejecute el mismo escenario con diferentes valores de elasticidad
python src/simulacion.py --escenario mi_escenario.yaml --elasticidades config/elasticidades_alternativas.yaml
```

---

## 7. Supuestos del modelo y limitaciones

### Supuestos principales

- **Competencia perfecta** en todos los mercados (los productores son tomadores de precios)
- **Rendimientos constantes a escala** en la función de producción
- **Economía pequeña y abierta**: los precios internacionales son exógenos (el país no afecta precios mundiales)
- **Año base representativo**: la SAM refleja una situación de equilibrio en el año base
- **Análisis comparativo estático**: el modelo compara dos equilibrios (antes y después de la política) sin modelar explícitamente la trayectoria de transición

### Limitaciones importantes

- El modelo no captura **efectos de corto plazo** ni fricciones de ajuste (rigideces de precios, costos de transacción)
- No modela **comportamientos estratégicos** ni poder de mercado sectorial
- La precisión de los resultados depende directamente de la **calidad de la SAM** y los **parámetros de elasticidad**
- No es apropiado para simular **crisis financieras** ni fenómenos monetarios
- Los resultados son sensibles al **cierre macroeconómico** elegido; se recomienda probar más de uno

---

## 8. Solución de problemas comunes

### El modelo no converge

**Síntoma:** El proceso de solución se detiene con el mensaje `No se encontró solución en N iteraciones`.

**Posibles causas y soluciones:**
- La perturbación es demasiado grande: pruebe con un cambio más pequeño y escale gradualmente
- El cierre macroeconómico no es apropiado para el escenario: revise la sección 3.3
- Hay inconsistencias en la SAM: ejecute `python src/verificar_instalacion.py` con la opción `--check-sam`

### Error al leer los archivos de datos

**Síntoma:** `FileNotFoundError: No se encontró SAM_base.xlsx`

**Solución:** Verifique que los archivos están en la carpeta `data/` y que los nombres coinciden exactamente con los configurados en `config/parametros_base.yaml`.

### Los resultados parecen extremos

**Síntoma:** Variaciones superiores al 50% en sectores individuales.

**Posible causa:** Elasticidades muy altas para el tamaño de la perturbación, o la SAM presenta sectores muy pequeños con alta sensibilidad relativa.

**Solución:** Revise los valores de elasticidad del sector afectado en `elasticidades.xlsx` y consulte con el equipo técnico.

---

*Para soporte adicional, abra un issue en el repositorio o contacte al equipo técnico.*
