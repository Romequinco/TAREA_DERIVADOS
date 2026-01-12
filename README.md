# Sistema Avanzado de Trading y Análisis de Opciones sobre SPY

Sistema completo de análisis y trading de opciones sobre SPY (SPDR S&P 500 ETF Trust) para ingeniería financiera cuantitativa y trading algorítmico profesional.

## Descripción del Proyecto

Este proyecto está compuesto por **dos notebooks principales**:

1. **`EJERCICIO_MIAX_2025.ipynb`**: Notebook del profesor con la implementación base
   - Conexión a Interactive Brokers
   - Implementación de Black-Scholes
   - Cálculo de griegas y volatilidad implícita
   - Visualización de volatility smiles y surfaces

2. **`notebook_principal.ipynb`**: Notebook de la clase con todos los objetivos implementados
   - Extensiones y objetivos completados sobre el código base
   - Construcción y análisis de Long Straddle
   - Delta-hedging profesional
   - Backtesting completo de estrategias
   - Análisis exhaustivo de P&L histórico

El código del profesor está **PRESERVADO COMPLETAMENTE** y el código nuevo **REUTILIZA** todas las funciones existentes manteniendo la nomenclatura original.

---

## Estructura del Notebook Principal

El `notebook_principal.ipynb` implementa 6 objetivos principales:

1. **Construcción de Long Straddle sobre SPY**
   - Funciones para construir straddles ATM
   - Análisis de sensibilidad a diferentes strikes
   - Visualización de payoffs

2. **Versión Delta-Hedged del Straddle**
   - Implementación de delta-hedging diario
   - Rebalanceo automático usando SPY
   - Tracking completo de posiciones

3. **Análisis de P&L Histórico: Expuesta vs Delta-Hedged**
   - Cálculo exhaustivo de P&L
   - Descomposición por contribución de griegas
   - Comparación entre estrategias

4. **Backtesting de Estrategias**
   - Clases LongStraddleStrategy y DeltaHedgedStraddleStrategy
   - Métricas de performance completas
   - Generación automática de datos de simulación

5. **Comparación Estratégica**
   - Análisis comparativo entre straddle normal y delta-hedged
   - Métricas y visualizaciones comparativas

6. **Visualizaciones y Resultados**
   - Gráficos comparativos
   - Organización de outputs en `outputs/html/`

---

## Características Principales

### Funcionalidades Implementadas

- **Conexión a Interactive Brokers** con manejo robusto de errores y fallback a yfinance
- **Implementación propia de Black-Scholes** con cálculo completo de todas las griegas (Delta, Gamma, Theta, Vega, Rho)
- **Análisis de volatilidad implícita** con visualización de volatility smiles y surfaces
- **Construcción y análisis de Long Straddle** con funciones para construir straddles ATM
- **Delta hedging profesional** con simulación histórica y rebalanceo diario usando SPY
- **Backtesting completo** con clases para estrategias normales y delta-hedged
- **Análisis exhaustivo de P&L** con descomposición por contribución de griegas
- **Organización de outputs** con todos los archivos HTML en `outputs/html/` organizados por categoría

### Estructura de Outputs

```
outputs/
├── html/
│   ├── volatility_smiles/     # Gráficos de volatility smiles
│   ├── greeks/                 # Análisis de griegas
│   ├── volatility_surfaces/    # Superficies de volatilidad 3D
│   ├── historical_analysis/    # Análisis histórico
│   └── simulations/            # Resultados de backtesting
└── csv/
    ├── option_chains/          # Cadenas de opciones
    └── results/                # Resultados de análisis

images/
├── straddle_payoff.png         # Payoff del straddle
├── straddle_sensibilidad_strikes.png
└── pnl_analysis/               # Análisis de P&L
```

---

## Instalación Rápida

### Requisitos

- Python 3.8 o superior
- Interactive Brokers TWS o IB Gateway (opcional, para datos reales)

### Pasos de Instalación

```bash
# 1. Crear entorno virtual (recomendado)
python -m venv venv

# Windows
venv\Scripts\activate

# Linux/Mac
source venv/bin/activate

# 2. Instalar dependencias
pip install -r requirements.txt

# 3. Verificar instalación
python -c "import numpy, pandas, scipy, matplotlib, seaborn, plotly; print('OK')"
```

### Configurar Interactive Brokers (Opcional)

1. Descarga e instala [TWS](https://www.interactivebrokers.com/en/index.php?f=16042) o [IB Gateway](https://www.interactivebrokers.com/en/index.php?f=16457)
2. Configura el puerto API (por defecto: 7497 para paper trading)
3. Habilita "Enable ActiveX and Socket Clients" en Configuración → API

> **Nota**: Si no tienes acceso a IB, el sistema funcionará en modo simulación con datos sintéticos realistas.

---

## Uso

### Ejecutar el Notebook Principal

**Importante**: El `notebook_principal.ipynb` es completamente autónomo e incluye todas las funciones necesarias. Puede ejecutarse independientemente o en el mismo kernel donde se ejecutó el notebook del profesor.

---

## Objetivos Implementados

### Del Código Original (EJERCICIO_MIAX_2025.ipynb)

1. Conexión a Interactive Brokers desde Python
2. Definir contratos de opciones sobre SPY y obtener cadenas
3. Estimación de volatilidad implícita (método bisección)
4. Visualización de volatility smiles y surfaces
5. Cálculo de griegas (Delta, Gamma, Theta, Vega, Rho)

### Extensiones en notebook_principal.ipynb

1. **Análisis de sensibilidad de Long Straddle** a diferentes strikes (ATM-2% a ATM+2%)
2. **Backtesting de estrategias** con clase LongStraddleStrategy
3. **Delta-hedging profesional** con función aplicar_delta_hedge y clase DeltaHedgedStraddleStrategy
4. **Análisis exhaustivo de P&L** con funciones calcular_pnl_sin_hedge y calcular_pnl_con_hedge
5. **Comparación estratégica** entre straddle normal y delta-hedged

---

## Principios de Integración

1. **PRESERVAR**: Todo el código original se mantiene intacto
2. **REUTILIZAR**: Las funciones existentes se usan en lugar de reescribirlas
3. **EXTENDER**: Nuevas funcionalidades se añaden sin modificar las existentes
4. **CONSISTENCIA**: Nomenclatura original se mantiene (`S`, `K`, `T`, `r`, `q`, `sigma`)
5. **DOCUMENTAR**: Cada sección nueva explica qué reutiliza del código original

---

## Estructura del Proyecto

```
TAREA_DERIVADOS/
├── EJERCICIO_MIAX_2025.ipynb      # Notebook del profesor (código base)
├── notebook_principal.ipynb        # Notebook de la clase (extensiones)
├── requirements.txt                # Dependencias del proyecto
├── README.md                       # Este archivo
├── SOLUCION_ERROR_10089.md         # Guía para error de datos de mercado
├── images/                         # Imágenes y gráficos
│   ├── straddle_payoff.png
│   ├── pnl_analysis/
│   └── volatility_surfaces/
└── outputs/                        # Resultados organizados
    ├── html/                       # Gráficos HTML interactivos
    └── csv/                        # Datos en formato CSV
```

---

## Aprendizajes Principales

- **Reutilización de código**: Las funciones del código original (bs_price, bs_greeks_manual) son robustas y garantizan coherencia
- **Delta Hedging**: Transforma el straddle de apuesta direccional en apuesta pura sobre volatilidad
- **Análisis de sensibilidad**: Los straddles ATM ofrecen el mejor balance entre coste y sensibilidad
- **Backtesting**: La generación de datos de simulación permite validar estrategias sin datos históricos
- **Descomposición de P&L**: El análisis por contribución de griegas permite entender qué factores contribuyen más al resultado

---
