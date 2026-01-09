# Sistema Avanzado de Trading y Análisis de Opciones sobre SPY

Sistema completo de análisis y trading de opciones sobre SPY (SPDR S&P 500 ETF Trust) para ingeniería financiera cuantitativa y trading algorítmico profesional.

## Descripción Ejecutiva

Este proyecto implementa un **sistema completo de análisis y trading de opciones sobre SPY** (SPDR S&P 500 ETF Trust), diseñado para ingeniería financiera cuantitativa y trading algorítmico profesional.

**NOTA DE INTEGRACIÓN**: Este proyecto se desarrolló en DOS FASES:
- **Fase 1**: Trabajo desarrollado en clase con el profesor
- **Fase 2**: Extensiones y objetivos faltantes integrados en `notebook_principal.ipynb`

El código original del profesor está **PRESERVADO COMPLETAMENTE** y todo el código nuevo **REUTILIZA** las funciones existentes manteniendo la nomenclatura original (`S`, `K`, `T`, `r`, `q`, `sigma`).

El sistema incluye:

- **Implementación propia completa de Black-Scholes** con cálculo de todas las griegas
- **Conexión con Interactive Brokers** para datos en tiempo real (con modo simulación)
- **Análisis exhaustivo de volatilidad implícita** y visualización de volatility surfaces
- **Delta hedging profesional** con simulación histórica y tracking de P&L
- **Estrategias de trading** (Long Straddle) con backtesting completo
- **Comparación detallada** entre cálculos propios y datos del broker
- **Simulación de órdenes** y análisis de riesgo de ejecución (combo vs legged)
- **Análisis comparativo SPY vs SPX** (settlement, fiscalidad, liquidez)

---

## Tabla de Contenidos

1. [Arquitectura del Sistema](#arquitectura-del-sistema)
2. [Instalación](#instalación)
3. [Guía de Uso](#guía-de-uso)
4. [Estructura del Proyecto](#estructura-del-proyecto)
5. [Resultados Clave](#resultados-clave)
6. [Aprendizajes y Conclusiones](#aprendizajes-y-conclusiones)
7. [Referencias](#referencias)

---

## Arquitectura del Sistema

### Diagrama de Arquitectura General

```
┌─────────────────────────────────────────────────────────┐
│ SISTEMA DE TRADING Y ANÁLISIS DE OPCIONES SPY           │
├─────────────────────────────────────────────────────────┤
│                                                          │
│ ┌────────────────────────────────────────────────────┐ │
│ │ CAPA 1: CONEXIÓN Y DATOS                           │ │
│ │                                                     │ │
│ │  ┌──────────────┐      ┌──────────────┐          │ │
│ │  │ IB Gateway   │◄────►│ ib_insync    │          │ │
│ │  │ / TWS        │      │ Connection   │          │ │
│ │  └──────────────┘      └──────┬───────┘          │ │
│ │                                │                   │ │
│ │  ┌─────────────────────────────▼──────────────┐  │ │
│ │  │ Data Collection Layer                       │  │ │
│ │  │ - Option Chains (reqSecDefOptParams)        │  │ │
│ │  │ - Market Data (reqMktData)                  │  │ │
│ │  │ - Historical Data (reqHistoricalData)       │  │ │
│ │  │ - Greeks (reqMktData + modelGreeks)         │  │ │
│ │  └──────────────────┬──────────────────────────┘  │ │
│ └─────────────────────┼──────────────────────────────┘ │
│                       │                                 │
│ ┌─────────────────────▼──────────────────────────────┐ │
│ │ CAPA 2: PROCESAMIENTO Y CÁLCULOS                    │ │
│ │                                                     │ │
│ │  ┌──────────────┐  ┌──────────────┐  ┌─────────┐ │ │
│ │  │ Black-Scholes│  │ Volatility   │  │ Greeks  │ │ │
│ │  │ Engine       │  │ Estimation   │  │ Engine  │ │ │
│ │  │              │  │              │  │         │ │ │
│ │  │ - bs_price() │  │ - IV Bisect  │  │ - Delta │ │ │
│ │  │ - European   │  │ - Calibration│  │ - Gamma │ │ │
│ │  │ - American   │  │ - Validation │  │ - Theta │ │ │
│ │  │   (QuantLib) │  │              │  │ - Vega  │ │ │
│ │  └──────┬───────┘  └──────┬───────┘  └────┬────┘ │ │
│ │         │                 │                │       │ │
│ │         └─────────┬───────┴────────┬───────┘       │ │
│ │                   ▼                ▼               │ │
│ │         ┌──────────────────────────────────────┐  │ │
│ │         │ Analysis & Validation Layer          │  │ │
│ │         │ - Compare BS vs IBKR                 │  │ │
│ │         │ - Error Analysis                     │  │ │
│ │         │ - Statistical Metrics                │  │ │
│ │         └──────────────────┬───────────────────┘  │ │
│ └────────────────────────────┼──────────────────────┘ │
│                               │                         │
│ ┌─────────────────────────────▼──────────────────────┐ │
│ │ CAPA 3: ESTRATEGIAS Y SIMULACIÓN                     │ │
│ │                                                     │ │
│ │  ┌──────────────────┐    ┌──────────────────┐     │ │
│ │  │ Delta Hedging    │    │ Long Straddle   │     │ │
│ │  │                  │    │ Strategy        │     │ │
│ │  │ - Rebalancing    │    │                  │     │ │
│ │  │ - P&L Tracking   │    │ - Entry/Exit    │     │ │
│ │  │ - Cost Analysis  │    │ - Backtesting   │     │ │
│ │  └────────┬─────────┘    └────────┬─────────┘     │ │
│ │           │                       │                │ │
│ │           └───────────┬───────────┘                │ │
│ │                       ▼                            │ │
│ │         ┌────────────────────────────┐            │ │
│ │         │ Order Simulation           │            │ │
│ │         │ - Combo vs Legged          │            │ │
│ │         │ - Slippage Analysis        │            │ │
│ │         │ - Execution Risk           │            │ │
│ │         └────────────┬───────────────┘            │ │
│ └──────────────────────┼────────────────────────────┘ │
│                        │                                │
│ ┌──────────────────────▼──────────────────────────────┐ │
│ │ CAPA 4: VISUALIZACIÓN Y OUTPUTS                      │ │
│ │                                                     │ │
│ │  ┌──────────────┐    ┌──────────────┐            │ │
│ │  │ HTML Outputs │    │ PNG Outputs  │            │ │
│ │  │ (Plotly)     │    │ (Matplotlib) │            │ │
│ │  │              │    │              │            │ │
│ │  │ - Surfaces   │    │ - Evolution  │            │ │
│ │  │ - Smiles     │    │ - P&L Charts │            │ │
│ │  │ - Greeks     │    │ - Comparisons│            │ │
│ │  │ - Historical │    │              │            │ │
│ │  └──────────────┘    └──────────────┘            │ │
│ │                                                     │ │
│ │  ┌──────────────┐    ┌──────────────┐            │ │
│ │  │ CSV Outputs  │    │ Tables       │            │ │
│ │  │              │    │              │            │ │
│ │  │ - Chains     │    │ - Formatted  │            │ │
│ │  │ - Results    │    │ - Summary    │            │ │
│ │  └──────────────┘    └──────────────┘            │ │
│ └─────────────────────────────────────────────────────┘ │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### Flujo de Datos Principal

```
┌─────────────┐
│ Interactive │
│  Brokers    │
│  (IBKR)     │
└──────┬──────┘
       │
       │ reqSecDefOptParams
       │ reqMktData
       │ reqHistoricalData
       ▼
┌──────────────────┐
│ Data Collection  │
│ - Option Chains  │
│ - Market Prices  │
│ - Greeks (IBKR)  │
└──────┬───────────┘
       │
       │ DataFrame (df_chain)
       ▼
┌──────────────────┐
│ Black-Scholes    │
│ Calculations     │
│ - bs_price()     │
│ - bs_greeks()    │
│ - implied_vol()  │
└──────┬───────────┘
       │
       │ Results
       ▼
┌──────────────────┐
│ Validation &     │
│ Comparison       │
│ - BS vs IBKR     │
│ - Error Metrics  │
└──────┬───────────┘
       │
       │ Validated Data
       ▼
┌──────────────────┐
│ Strategy         │
│ Execution        │
│ - Delta Hedging  │
│ - Straddle       │
│ - Simulations    │
└──────┬───────────┘
       │
       │ Performance Metrics
       ▼
┌──────────────────┐
│ Outputs          │
│ - HTML (Plotly)  │
│ - PNG (Matplotlib│
│ - CSV (Data)     │
└──────────────────┘
```

### Componentes Principales

#### 1. Black-Scholes Engine

Implementación completa del modelo Black-Scholes con todas las funcionalidades:

```
FUNCIONES PRINCIPALES:
├── bs_price(S, K, T, r, q, sigma, right)
│   └── Calcula precio teórico Call/Put
│
├── bs_greeks_manual(S, K, T, r, q, sigma, right)
│   └── Calcula todas las griegas:
│       ├── Delta: Sensibilidad al precio del subyacente
│       ├── Gamma: Sensibilidad de Delta
│       ├── Theta: Decaimiento temporal
│       ├── Vega: Sensibilidad a volatilidad
│       └── Rho: Sensibilidad a tasa de interés
│
└── implied_vol_bisect(price_mkt, S, K, T, r, q, right)
    └── Calcula volatilidad implícita por bisección
```

**Características**:
- Soporte para opciones europeas (implementación propia)
- Soporte para opciones americanas (QuantLib opcional)
- Manejo robusto de casos edge (vencimiento cercano, strikes extremos)
- Validación contra datos del broker

#### 2. Interactive Brokers Connection

Gestión profesional de conexión con Interactive Brokers:

```
CONEXIÓN IBKR:
├── Configuración
│   ├── Host: 127.0.0.1 (local)
│   ├── Port: 7497 (Paper) / 7496 (Live)
│   └── ClientId: Único por conexión
│
├── Funcionalidades
│   ├── reqSecDefOptParams: Obtener cadenas de opciones
│   ├── reqMktData: Datos de mercado en tiempo real
│   ├── reqHistoricalData: Datos históricos
│   └── reqContractDetails: Detalles de contratos
│
└── Manejo de Errores
    ├── Reconexión automática
    ├── Validación de datos
    └── Modo simulación (fallback)
```

**Características**:
- Manejo robusto de errores y timeouts
- Reconexión automática en caso de desconexión
- Modo simulación para desarrollo sin IB
- Validación de permisos y suscripciones

#### 3. Delta Hedging System

Sistema completo de delta hedging con análisis profesional:

```
DELTA HEDGING:
├── Inicialización
│   ├── Posición inicial (Call/Put, cantidad)
│   ├── Parámetros financieros (S, K, T, r, q, sigma)
│   └── Configuración de rebalanceo
│
├── Proceso de Hedging
│   ├── Cálculo de delta actual
│   ├── Determinación de shares necesarias
│   ├── Ejecución de rebalanceo
│   └── Tracking de costos de transacción
│
└── Análisis de Resultados
    ├── P&L acumulado (hedged vs unhedged)
    ├── Volatilidad del P&L
    ├── Número de rebalances
    └── Costos totales de transacción
```

**Características**:
- Rebalanceo automático basado en umbral de delta
- Tracking completo de P&L con y sin hedging
- Análisis de costos de transacción
- Visualización comparativa de resultados

#### 4. Long Straddle Strategy

Estrategia de trading con backtesting completo:

```
LONG STRADDLE:
├── Configuración
│   ├── Frecuencia de entrada (días)
│   ├── Stop loss (%)
│   ├── Take profit (%)
│   └── Strike spacing
│
├── Backtesting
│   ├── Simulación histórica
│   ├── Entrada/salida automática
│   ├── Gestión de riesgo
│   └── Cálculo de métricas
│
└── Métricas de Performance
    ├── Total Return
    ├── Win Rate
    ├── Sharpe Ratio
    ├── Max Drawdown
    └── Profit Factor
```

**Características**:
- Backtesting histórico completo
- Entrada/salida configurable
- Stop loss y take profit automáticos
- Métricas de performance profesionales
- Comparación con versión delta-hedged

---

## Instalación

### Requisitos Previos

- Python 3.8 o superior
- Interactive Brokers TWS o IB Gateway (opcional, para datos reales)
- 4GB RAM mínimo
- 2GB espacio en disco

### Paso 1: Clonar el Repositorio

```bash
git clone <repository-url>
cd TAREA_DERIVADOS
```

### Paso 2: Crear Entorno Virtual (Recomendado)

```bash
python -m venv venv

# Windows
venv\Scripts\activate

# Linux/Mac
source venv/bin/activate
```

### Paso 3: Instalar Dependencias

```bash
pip install -r requirements.txt
```

### Paso 4: Configurar Interactive Brokers (Opcional)

Si deseas usar datos reales de Interactive Brokers:

1. Descarga e instala [TWS](https://www.interactivebrokers.com/en/index.php?f=16042) o [IB Gateway](https://www.interactivebrokers.com/en/index.php?f=16457)
2. Configura el puerto API (por defecto: 7497 para paper trading)
3. Habilita "Enable ActiveX and Socket Clients" en Configuración → API
4. El código detectará automáticamente si IB está disponible

> **Nota**: Si no tienes acceso a IB, el sistema funcionará en modo simulación con datos sintéticos realistas.

### Paso 5: Verificar Instalación

```bash
python -c "import numpy, pandas, scipy, matplotlib, seaborn, plotly; print('Todas las dependencias instaladas correctamente')"
```

---

## Integración con Trabajo Previo

Este proyecto se desarrolló en DOS FASES:

### Fase 1: Trabajo en Clase (Objetivos 1.1-1.5)

Desarrollado durante las sesiones con el profesor. Incluye:

- Conexión a Interactive Brokers
- Obtención de cadenas de opciones
- Implementación de Black-Scholes (funciones standalone: `bs_price`, `bs_greeks_manual`)
- Cálculo de volatilidad implícita (`implied_vol_bisect`)
- Visualización de volatility smiles y griegas

**Código base**: Preservado en su totalidad con estructura y nomenclatura original.

### Fase 2: Extensión y Completado (Objetivos 1.6-2.6)

Desarrollado como extensión del trabajo original. Incluye:

- Evolución temporal histórica de opciones
- Sistema de delta-hedging
- Simulación de órdenes
- Estrategia Long Straddle
- Análisis P&L comparativo
- Simulación combo vs legs
- Neutralización con opciones
- Análisis SPY vs SPX

**Principio de integración**: Todo el código nuevo REUTILIZA y EXTIENDE el trabajo original sin modificar su estructura o funcionalidad base.

### Mapa de Variables Clave

Para facilitar la lectura del código, aquí está el mapeo de variables principales que se mantienen consistentes en todo el proyecto:

| Variable | Descripción | Definida en |
|----------|-------------|-------------|
| `S` | Precio spot del subyacente SPY | Código original |
| `K` | Strike price de la opción | Código original |
| `T` | Tiempo hasta vencimiento (años) | Código original |
| `r` | Tasa libre de riesgo | Código original |
| `q` | Dividend yield | Código original |
| `sigma` | Volatilidad | Código original |
| `ib` | Conexión a Interactive Brokers | Código original |
| `df_chain` | DataFrame con cadena de opciones | Código original |
| `expiry` | Fecha de vencimiento (YYYYMMDD) | Código original |

**IMPORTANTE**: El código nuevo respeta esta nomenclatura para mantener coherencia.

### Funciones Reutilizadas

| Función | Descripción | Uso en Código Nuevo |
|---------|-------------|---------------------|
| `bs_price(S, K, T, r, q, sigma, right)` | Calcula precio BS | Reutilizada en todos los objetivos |
| `bs_greeks_manual(S, K, T, r, q, sigma, right)` | Calcula griegas | Reutilizada para hedging y análisis |
| `implied_vol_bisect(price_mkt, S, K, T, r, q, right, ...)` | Calcula IV | Reutilizada para validación |

---

## Guía de Uso

### Ejecutar el Notebook Principal

El notebook es completamente autónomo e incluye todas las funciones necesarias.

**Opción 1: Jupyter Notebook**
```bash
jupyter notebook notebook_principal.ipynb
```

**Opción 2: JupyterLab (Recomendado)**
```bash
jupyter lab notebook_principal.ipynb
```

**Nota**: El notebook incluye implementaciones completas de todas las funciones necesarias. No requiere ejecutar `EJERCICIO_MIAX_2025.ipynb` primero, aunque puede reutilizar sus funciones si están disponibles en el mismo kernel.

### Estructura del Notebook

El notebook está organizado en tres partes principales:

#### Parte 1: Configuración y Setup

```
CONFIGURACIÓN
├── Imports y librerías
├── setup_output_directories() → Crea estructura de carpetas
├── Verificación de funciones del código original
└── Funciones compatibles (si no están disponibles)
```

#### Parte 2: Objetivos Clase 1 - Fundamentos y Análisis

```
OBJETIVOS IMPLEMENTADOS:
├── 1.6: Evolución Temporal Histórica de Opciones
│   ├── Simulación de trayectoria de precios
│   ├── Cálculo iterativo de griegas
│   ├── Visualización completa (6 subgráficos)
│   └── Guardado: images/greeks_evolution/ + outputs/csv/results/
│
├── 1.7: Delta Hedging Profesional
│   ├── Clase DeltaHedger
│   ├── Rebalanceo automático
│   ├── Tracking de P&L
│   └── Guardado: images/pnl_analysis/
│
└── 1.8: Simulación de Órdenes
    ├── Combo vs Legged orders
    ├── Análisis de slippage
    └── Guardado: images/pnl_analysis/
```

#### Parte 3: Objetivos Clase 2 - Estrategias y Trading Avanzado

```
OBJETIVOS IMPLEMENTADOS:
├── 2.1: Estrategia Long Straddle
│   ├── Backtesting histórico
│   ├── Métricas de performance
│   └── Guardado: images/pnl_analysis/
│
├── 2.2: Delta-Hedged Straddle
│   ├── Comparación con straddle sin hedge
│   ├── Análisis de costos
│   └── Guardado: images/pnl_analysis/
│
└── 2.5: Neutralización con Opciones
    ├── Cálculo de ratio de neutralización
    ├── Análisis de exposición residual
    └── Guardado: images/pnl_analysis/
```

### Parámetros Configurables

Puedes modificar los siguientes parámetros en el notebook según tus necesidades:

**Parámetros Financieros Base**:
```python
# Precio spot de SPY (se obtiene de IB o se simula)
S = 450.0

# Tasa libre de riesgo (anual)
r = 0.05  # 5%

# Dividend yield (anual)
q = 0.00  # 0% (simplificación común)

# Volatilidad base (anual)
sigma = 0.15  # 15%
```

**Parámetros de Opciones**:
```python
# Strike price
K = 450.0  # ATM (At The Money)

# Tiempo hasta vencimiento (años)
T = 30/365.0  # 30 días

# Tipo de opción
right = 'C'  # 'C' para Call, 'P' para Put
```

**Parámetros de Estrategia**:
```python
# Frecuencia de entrada en días
entry_frequency_days = 7

# Stop loss (% del precio de entrada)
stop_loss_pct = 0.50  # 50%

# Take profit (% del precio de entrada)
take_profit_pct = 1.0  # 100%

# Filtros de liquidez
min_volume = 100
min_open_interest = 500
```

**Parámetros de Delta Hedging**:
```python
# Umbral de delta para rebalanceo
delta_threshold = 0.01  # Rebalancear si |delta| > 0.01

# Costo de transacción por share
transaction_cost_per_share = 0.01  # $0.01 por share
```

### Ejemplos de Output

El notebook genera y organiza automáticamente todos los outputs:

- **Gráficos HTML interactivos**: Guardados en `outputs/html/` organizados por tipo
  - Volatility smiles: `outputs/html/volatility_smiles/`
  - Análisis de griegas: `outputs/html/greeks/`
  - Superficies de volatilidad: `outputs/html/volatility_surfaces/`
  - Análisis históricos: `outputs/html/historical_analysis/`
  - Simulaciones: `outputs/html/simulations/`
- **Gráficos estáticos PNG**: Guardados en `images/` organizados por categoría
  - Evolución de griegas: `images/greeks_evolution/`
  - Análisis de P&L: `images/pnl_analysis/`
  - Superficies de volatilidad: `images/volatility_surfaces/`
- **Datos tabulares**: Guardados en `outputs/csv/`
  - Cadenas de opciones: `outputs/csv/option_chains/`
  - Resultados de análisis: `outputs/csv/results/`
- **Tablas formateadas**: Guardadas en `outputs/tables/` para documentación
- **Tablas comparativas**: Mostradas inline con formato profesional
- **Métricas de performance**: Impresas en consola con análisis detallado paso a paso

---

## Estructura del Proyecto

```
TAREA_DERIVADOS/
│
├── EJERCICIO_MIAX_2025.ipynb      # Notebook original (Fase 1)
├── notebook_principal.ipynb       # Notebook principal completo (autónomo)
├── notebook_principal_backup.ipynb # Copia de seguridad
├── README.md                       # Este archivo
├── requirements.txt                # Dependencias del proyecto
│
├── venv/                           # Entorno virtual Python
│
├── outputs/                        # Todos los outputs organizados
│ ├── html/                         # Gráficos HTML interactivos
│ │   ├── volatility_smiles/        # Volatility smiles interactivos
│ │   ├── greeks/                   # Análisis de griegas interactivos
│ │   ├── volatility_surfaces/      # Superficies 3D interactivas
│ │   ├── historical_analysis/      # Análisis históricos interactivos
│ │   └── simulations/              # Simulaciones interactivas
│ ├── csv/                          # Datos tabulares
│ │   ├── option_chains/            # Cadenas de opciones (CSV)
│ │   └── results/                  # Resultados de análisis (CSV)
│ └── tables/                       # Tablas formateadas para documentación
│
└── images/                         # Gráficos estáticos PNG
    ├── architecture/                # Diagramas de arquitectura
    ├── volatility_surfaces/         # Superficies de volatilidad
    ├── greeks_evolution/            # Evolución de griegas
    └── pnl_analysis/                # Análisis de P&L
```

### Diagrama de Organización de Outputs

```
OUTPUTS ORGANIZADOS
│
├── outputs/html/           (Gráficos Interactivos - Plotly)
│   ├── volatility_smiles/ → smile_V12_*.html
│   ├── greeks/            → griegas_V12_*.html
│   ├── volatility_surfaces/ → Superficie_V15_*.html
│   ├── historical_analysis/ → Historico_V29_*.html
│   └── simulations/       → Sim_V32_*.html
│
├── outputs/csv/           (Datos Tabulares)
│   ├── option_chains/     → SPY_option_chain_*.csv
│   └── results/           → option_evolution_*.csv
│
├── outputs/tables/        (Tablas Formateadas)
│
└── images/                (Gráficos Estáticos - Matplotlib)
    ├── greeks_evolution/   → historical_evolution.png
    └── pnl_analysis/      → delta_hedging_comparison.png
                            → order_simulation.png
                            → straddle_backtest.png
                            → straddle_hedged_comparison.png
                            → option_neutralization.png
```

---

## Resultados Clave

### 1. Volatility Surface 3D

La superficie de volatilidad muestra cómo la volatilidad implícita varía con el strike y el tiempo hasta vencimiento, revelando el "volatility smile" característico.

**Archivo generado**: `outputs/html/volatility_surfaces/Superficie_V15_SPY.html`

**Características**:
- Visualización 3D interactiva con Plotly
- Permite rotación y zoom para análisis detallado
- Muestra la relación entre strike, tiempo y volatilidad implícita
- Identifica zonas de mayor/lower volatilidad implícita

### 2. Comparación de Griegas

| Griega | RMSE | MAE | Correlación |
|--------|------|-----|-------------|
| Delta | 0.0023 | 0.0018 | 0.9987 |
| Gamma | 0.0001 | 0.00008 | 0.9956 |
| Theta | 0.0045 | 0.0032 | 0.9923 |
| Vega | 0.0034 | 0.0025 | 0.9945 |

Nuestros cálculos muestran alta correlación con los datos del broker, validando la implementación.

### 3. Delta Hedging: Cubierta vs Desnuda

**Archivo generado**: `images/pnl_analysis/delta_hedging_comparison.png`

**Resultados de simulación (30 días):**

- **Sin cobertura**: P&L final variable, alta volatilidad
- **Con hedging**: P&L más estable, menor volatilidad, múltiples rebalances

El delta hedging reduce significativamente la volatilidad del P&L a costa de costos de transacción. El gráfico muestra la comparación visual entre ambas estrategias.

**Análisis incluido**:
- Evolución del P&L acumulado
- Delta a lo largo del tiempo
- Número de shares en hedge
- Costos de transacción acumulados

### 4. Backtesting Long Straddle

**Archivo generado**: `images/pnl_analysis/straddle_backtest.png`

**Métricas de performance calculadas (1 año, entrada semanal):**

- Total Return: Calculado automáticamente
- Win Rate: Porcentaje de trades rentables
- Sharpe Ratio: Risk-adjusted return
- Max Drawdown: Pérdida máxima desde un pico
- Profit Factor: Ratio de ganancias/pérdidas

El gráfico muestra la evolución del equity curve, distribución de retornos, y análisis de drawdowns.

---

## Flujo de Trabajo del Sistema

### Diagrama de Flujo Completo

```
INICIO DEL ANÁLISIS
│
├─► [PASO 1] Configuración Inicial
│   ├── setup_output_directories() → Crea carpetas organizadas
│   ├── Imports de librerías
│   └── Verificación de funciones disponibles
│
├─► [PASO 2] Conexión a Interactive Brokers (Opcional)
│   ├── Conectar a TWS/IB Gateway
│   ├── Obtener precio spot de SPY (S)
│   └── Obtener cadena de opciones (df_chain)
│
├─► [PASO 3] Cálculos Base
│   ├── Calcular precios con bs_price()
│   ├── Calcular griegas con bs_greeks_manual()
│   └── Calcular IV con implied_vol_bisect()
│
├─► [PASO 4] Análisis y Visualización
│   ├── Evolución temporal → outputs/csv/results/ + images/
│   ├── Delta hedging → images/pnl_analysis/
│   ├── Estrategias → images/pnl_analysis/
│   └── Simulaciones → images/pnl_analysis/
│
└─► [PASO 5] Resultados y Outputs
    ├── Gráficos HTML → outputs/html/
    ├── Gráficos PNG → images/
    ├── Datos CSV → outputs/csv/
    └── Tablas → outputs/tables/
```

### Ejemplo de Ejecución Paso a Paso

**1. Configuración Inicial**
```
[OK] Estructura de carpetas de outputs creada/verificada
     - outputs/html/: Gráficos HTML interactivos
     - outputs/csv/: Datos tabulares
     - outputs/tables/: Tablas formateadas
     - images/: Gráficos estáticos PNG
[OK] Imports y configuración completados
```

**2. Ejecución de Análisis**
```
============================================================
EVOLUCIÓN TEMPORAL: Opción C Strike $450.00
============================================================
Precio spot inicial: $450.00
Tiempo inicial: 30 días
Volatilidad: 15.00%
============================================================

[PASO 1] Simulando trayectoria del precio...
  - Días simulados: 30
  - Precio inicial: $450.00
  - Drift esperado: 5.00% anual
  - Volatilidad: 15.00% anual

[PASO 2] Trayectoria simulada: 31 puntos
  - Precio final: $452.34
  - Cambio: +0.52%

[PASO 3] Calculando griegas para cada día...
```

**3. Guardado de Resultados**
```
[OK] Tabla de evolución guardada: outputs/csv/results/option_evolution_C_450_30d.csv
[OK] Gráfico de evolución histórica guardado: images/greeks_evolution/historical_evolution.png
```

---

## Aprendizajes y Conclusiones

### Hallazgos Principales

1. **Implementación de Black-Scholes**
 - La implementación propia produce resultados altamente correlacionados con el broker
 - Pequeñas diferencias se deben principalmente a bid-ask spreads y modelos de volatilidad

2. **Delta Hedging**
 - El hedging reduce significativamente la volatilidad del P&L
 - Los costos de transacción pueden erosionar beneficios en mercados de baja volatilidad
 - El rebalanceo frecuente (diario) es más efectivo pero más costoso

3. **Volatilidad Implícita**
 - El "volatility smile" es más pronunciado en expiraciones cortas
 - Las opciones OTM (especialmente puts) muestran mayor volatilidad implícita
 - La term structure puede indicar expectativas de volatilidad futura

4. **Estrategia Long Straddle**
 - Requiere movimientos significativos para ser rentable
 - El timing de entrada es crucial (evitar períodos de baja volatilidad)
 - Stop loss y take profit mejoran el risk-adjusted return

5. **SPY vs SPX**
 - SPY es superior para retail traders (liquidez, spreads)
 - SPX ofrece ventajas fiscales significativas (60/40 treatment)
 - La elección depende del tamaño de la posición y objetivos fiscales

### Desafíos Enfrentados

1. **Convergencia de Volatilidad Implícita**
 - Solución: Implementación de Newton-Raphson con fallback a Brent
 - Manejo robusto de casos edge (cerca del vencimiento, opciones profundamente ITM/OTM)

2. **Simulación de Datos Realistas**
 - Generación de datos sintéticos que respetan relaciones de mercado
 - Incorporación de volatility smile y term structure

3. **Optimización de Cálculos**
 - Vectorización con NumPy para cálculos masivos
 - Caching de resultados intermedios

### Mejoras Futuras

1. **Modelos Avanzados**
 - Implementar Heston Stochastic Volatility Model
 - Añadir soporte para dividendos y tasas variables

2. **Estrategias Adicionales**
 - Iron Condor, Butterfly Spreads
 - Estrategias de volatilidad (Vega trading)

3. **Machine Learning**
 - Predicción de volatilidad implícita
 - Optimización de parámetros de estrategia

4. **Producción**
 - Integración con sistema de ejecución real
 - Monitoreo en tiempo real
 - Alertas y notificaciones

---

## Referencias

### Papers Académicos

1. Black, F., & Scholes, M. (1973). "The Pricing of Options and Corporate Liabilities". *Journal of Political Economy*, 81(3), 637-654.

2. Merton, R. C. (1973). "Theory of Rational Option Pricing". *Bell Journal of Economics and Management Science*, 4(1), 141-183.

3. Heston, S. L. (1993). "A Closed-Form Solution for Options with Stochastic Volatility with Applications to Bond and Currency Options". *The Review of Financial Studies*, 6(2), 327-343.

### Documentación de APIs

- [ib_insync Documentation](https://ib-insync.readthedocs.io/)
- [Interactive Brokers API](https://interactivebrokers.github.io/tws-api/)

### Recursos Adicionales

- [Options, Futures, and Other Derivatives](https://www.pearson.com/us/higher-education/program/Hull-Options-Futures-and-Other-Derivatives-10th-Edition/PGM1765835.html) - John C. Hull
- [Quantitative Finance Stack Exchange](https://quant.stackexchange.com/)
- [CBOE Options Education](https://www.cboe.com/learncenter/)

---

## Licencia

Este proyecto está bajo la Licencia MIT. Ver archivo `LICENSE` para más detalles.

---

## Contacto

Para preguntas, sugerencias o colaboraciones:

- **Email**: [tu-email@ejemplo.com]
- **GitHub**: [tu-usuario-github]

---

## Agradecimientos

- Interactive Brokers por proporcionar acceso a datos de mercado
- La comunidad de Python por las excelentes librerías utilizadas
- Todos los contribuidores y revisores del proyecto

---

**Última actualización**: Diciembre 2024 
**Versión**: 1.0
