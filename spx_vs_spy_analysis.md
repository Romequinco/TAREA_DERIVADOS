# Reflexión: ¿Qué habría cambiado usando SPX en lugar de SPY?

## Introducción

A lo largo de este proyecto hemos utilizado opciones sobre **SPY** (SPDR S&P 500 ETF Trust) para implementar nuestra estrategia de long straddle y sus variantes delta-hedged. En esta sección reflexionamos sobre qué habría cambiado si hubiéramos utilizado opciones sobre **SPX** (S&P 500 Index).

---

## 1. Diferencias Fundamentales entre SPX y SPY

### Tabla Comparativa General

| Característica | SPX | SPY |
|----------------|-----|-----|
| **Tipo de instrumento** | Índice (no negociable) | ETF (negociable) |
| **Valor aproximado** | ~5,000 USD | ~500 USD (1/10 de SPX) |
| **Multiplicador** | 100 | 100 |
| **Valor nocional por contrato** | ~500,000 USD | ~50,000 USD |
| **Estilo de ejercicio** | Europeo (solo al vencimiento) | Americano (cualquier momento) |
| **Tipo de liquidación** | Cash settlement (efectivo) | Physical settlement (acciones ETF) |
| **Subyacente negociable** | No | Sí |
| **Tratamiento fiscal (USA)** | Sección 1256 (60/40) | Capital gains estándar |
| **Dividendos** | No (índice de precios) | Sí (ETF paga dividendos) |
| **Horario de trading** | Más amplio (opens antes) | Horario regular de mercado |

---

## 2. Implicaciones para la Estrategia Long Straddle

### 2.1 Tamaño de la Posición

**SPX:**
- 1 contrato de opciones SPX tiene aproximadamente la misma exposición que 10 contratos de opciones SPY
- Valor nocional: ~500,000 USD por straddle
- Requiere cuenta significativamente más grande
- Inversión típica en prima: 15,000 - 30,000 USD para un straddle ATM a 30 días
- **Público objetivo**: institucionales, fondos, cuentas grandes (>100,000 USD)

**SPY:**
- Valor nocional: ~50,000 USD por straddle
- Inversión típica en prima: 1,500 - 3,000 USD para un straddle ATM a 30 días
- **Público objetivo**: retail, cuentas más pequeñas, educación

**Conclusión**: SPY es más accesible y apropiado para cuentas retail y propósitos educativos.

---

### 2.2 Estilo de Ejercicio: Europeo vs Americano

**SPX (Europeo):**
- Las opciones solo pueden ejercerse en la fecha de vencimiento
- **Ventajas**:
  - Más predecible: sabes exactamente cuándo puede haber ejercicio
  - Sin pin risk: no hay riesgo de que te asignen antes del vencimiento
  - Modelos teóricos más precisos (Black-Scholes asume ejercicio europeo)
- **Desventajas**:
  - Menos flexibilidad para el comprador
  - No puedes ejercer anticipadamente si es muy favorable

**SPY (Americano):**
- Las opciones pueden ejercerse en cualquier momento antes del vencimiento
- **Ventajas**:
  - Flexibilidad para el comprador
  - Puedes capturar valor intrínseco anticipadamente si es óptimo
- **Desventajas**:
  - Pin risk: riesgo de asignación anticipada si las opciones están deep ITM
  - Para long straddle: si una pata se mete muy ITM, puede ser ejercida antes del vencimiento
  - Complica la gestión de la posición

**Impacto en nuestro proyecto:**
- Con SPY tuvimos que estar atentos al riesgo de ejercicio anticipado si el mercado se movía fuertemente en una dirección
- Con SPX esto no sería una preocupación: solo monitorizamos hasta el vencimiento
- Para una estrategia delta-hedged, el ejercicio anticipado de SPY puede complicar el rebalanceo

---

### 2.3 Liquidez y Spreads

**SPX:**
- Liquidez excelente en strikes ATM y cercanos
- Spreads bid-ask típicamente más estrechos en términos porcentuales
- Ejemplo: spread de 0.20 USD sobre opción de 80 USD = 0.25%
- Mejor para vencimientos mensuales estándar
- Market makers profesionales ofrecen buenos precios en combos

**SPY:**
- Liquidez excepcional en prácticamente todos los strikes
- Spreads muy estrechos, especialmente en strikes ATM
- Ejemplo: spread de 0.02-0.05 USD sobre opción de 8 USD = 0.25-0.6%
- Excelente liquidez también en vencimientos semanales
- Uno de los productos de opciones más líquidos del mundo

**Conclusión**: Ambos tienen liquidez excelente, pero SPY tiene ventaja en strikes más alejados y vencimientos no estándar.

---

## 3. Implicaciones para el Delta-Hedging

### 3.1 El Desafío con SPX

**Problema fundamental**: No puedes comprar o vender el índice SPX directamente (es solo un número calculado).

**Alternativas para hedgear delta con SPX:**

#### Opción 1: Futuros ES (E-mini S&P 500)
```
Ratio de conversión:
- 1 contrato futuro ES = 50 × Precio del índice S&P 500
- 1 contrato opción SPX = 100 × Precio del índice S&P 500
- Por tanto: 1 opción SPX ≈ 2 futuros ES

Ejemplo de hedge:
- Straddle SPX con Delta = +0.20
- Delta total = 0.20 × 100 = +20 "puntos de índice"
- Futuros ES necesarios = 20 / 50 = 0.4 contratos
- Problema: no puedes operar 0.4 contratos, mínimo es 1
- Resultado: overhedge o underhedge, no puedes ser preciso
```

**Desventajas**:
- Ratio complejo (2:1)
- Granularidad gruesa: no puedes ajustar con precisión
- Requiere cuenta de futuros
- Margin requirements diferentes
- Complejidad operativa adicional

#### Opción 2: Usar SPY como proxy
```
Ratio de conversión:
- SPX ≈ 10 × SPY
- 1 opción SPX ≈ 10 opciones SPY (en exposición)
- Para hedgear 1 opción SPX: usar 10× las acciones de SPY

Ejemplo de hedge:
- Straddle SPX con Delta = +0.20
- Acciones SPY necesarias = 0.20 × 100 × 10 = 200 acciones
```

**Desventajas**:
- Tracking error: SPY no replica perfectamente SPX (aunque es mínimo)
- Dividendos de SPY introducen pequeñas diferencias
- Expenses ratio del ETF (muy bajo, ~0.09%)

#### Opción 3: Usar otra opción SPX
- Vender calls/puts OTM como hicimos en el Prompt 5
- Ventaja: todo en el mismo producto
- Desventaja: afecta Gamma, Vega, Theta (no es hedge puro de delta)

---

### 3.2 La Simplicidad con SPY

**Hedge directo con el subyacente:**
```python
# Cálculo simple y directo
delta_straddle = delta_call + delta_put  # Ejemplo: +0.20
acciones_necesarias = -delta_straddle × 100  # -20 acciones

# Ejecución
if acciones_necesarias > 0:
    comprar(20, 'SPY')
else:
    vender(20, 'SPY')
```

**Ventajas:**
- Ratio 1:1 perfecto: delta de opciones SPY se cubre con acciones SPY
- Granularidad fina: puedes ajustar de 1 en 1 acción
- Simple de entender y ejecutar
- No requiere cuenta de futuros
- Tracking perfecto (mismo subyacente)

**Desventaja:**
- Requiere capital para comprar/vender acciones (no apalancado como futuros)
- Ejemplo: hedgear delta de 0.20 requiere ~20 × 500 = 10,000 USD de capital

---

### 3.3 Comparación Visual del Delta-Hedging

```
ESCENARIO: Straddle con Delta = +0.30, necesitas neutralizar

┌─────────────────────────────────────────────────────────────┐
│                        CON SPX                               │
├─────────────────────────────────────────────────────────────┤
│ Opción 1: Futuros ES                                        │
│   Delta a neutralizar: +30 puntos índice                    │
│   Futuros necesarios: 30/50 = 0.6 contratos                │
│   Problema: Solo puedes operar 1 contrato entero           │
│   Resultado: Overhedge de 66%                               │
│                                                              │
│ Opción 2: Usar SPY como proxy                               │
│   Delta a neutralizar: +30 puntos índice                    │
│   Acciones SPY: 30 × 10 = 300 acciones                     │
│   Capital necesario: 300 × 500 = 150,000 USD               │
│   Resultado: Posible pero capital intensivo                 │
│                                                              │
│ Opción 3: Vender calls SPX OTM                              │
│   Encuentra call con delta -0.30                            │
│   Vende 1 contrato                                          │
│   Resultado: Delta neutral PERO afecta Gamma/Vega/Theta    │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                        CON SPY                               │
├─────────────────────────────────────────────────────────────┤
│ Usar acciones SPY directamente                              │
│   Delta a neutralizar: +0.30                                │
│   Acciones necesarias: -0.30 × 100 = -30 acciones          │
│   Operación: Vender 30 acciones de SPY                     │
│   Capital: 30 × 500 = 15,000 USD                           │
│   Resultado: Hedge preciso, simple, directo                 │
└─────────────────────────────────────────────────────────────┘
```

**Conclusión clara**: Para delta-hedging con rebalanceo frecuente, SPY es significativamente más simple y práctico.

---

## 4. Aspectos Fiscales (Contexto USA)

### 4.1 Tratamiento Fiscal de SPX (Sección 1256)

Las opciones sobre índices amplios como SPX reciben tratamiento especial bajo la Sección 1256 del código fiscal estadounidense:

```
Regla 60/40:
- 60% de las ganancias/pérdidas se tratan como long-term capital gains
- 40% se tratan como short-term capital gains
- Independientemente del tiempo que mantengas la posición

Tasas aproximadas (2024):
- Long-term capital gains: 15-20% (según bracket)
- Short-term capital gains: hasta 37% (tasa ordinaria)

Tasa efectiva bajo 1256:
- 60% × 20% + 40% × 37% = 12% + 14.8% = 26.8%
```

**Ejemplo numérico:**
```
Ganancia en straddle SPX: 10,000 USD

Bajo Sección 1256:
- Long-term (60%): 6,000 USD × 20% = 1,200 USD
- Short-term (40%): 4,000 USD × 37% = 1,480 USD
- Total impuestos: 2,680 USD
- Tasa efectiva: 26.8%
- Ganancia neta: 7,320 USD
```

### 4.2 Tratamiento Fiscal de SPY (Estándar)

Las opciones sobre ETFs como SPY reciben tratamiento estándar de capital gains:

```
Depende del holding period:
- Menos de 1 año: short-term capital gains (tasa ordinaria, hasta 37%)
- Más de 1 año: long-term capital gains (15-20%)

Para trading activo de opciones (típico en straddles):
- Casi siempre short-term (posiciones < 1 año)
- Tasa: hasta 37%
```

**Mismo ejemplo:**
```
Ganancia en straddle SPY: 10,000 USD
Holding period: 30 días (típico)

Short-term capital gains:
- 10,000 USD × 37% = 3,700 USD
- Ganancia neta: 6,300 USD
```

### 4.3 Comparación Fiscal

```
Ganancia de 10,000 USD:
┌──────────────┬──────────────┬──────────────┬──────────────┐
│  Instrumento │   Impuestos  │ Ganancia Neta│  Diferencia  │
├──────────────┼──────────────┼──────────────┼──────────────┤
│     SPX      │   2,680 USD  │   7,320 USD  │      -       │
│     SPY      │   3,700 USD  │   6,300 USD  │  -1,020 USD  │
└──────────────┴──────────────┴──────────────┴──────────────┘

Ahorro fiscal con SPX: 1,020 USD (10.2% de la ganancia)
```

**Ventaja significativa de SPX para traders activos profesionales.**

---

## 5. Dividendos y su Impacto

### 5.1 SPX: Índice de Precios

**Concepto**: SPX es un índice de precios, NO incluye dividendos.

**Implicaciones:**
- Las empresas del S&P 500 pagan dividendos (~1.5% anual agregado)
- Cuando una empresa paga dividendo, su precio baja por el monto del dividendo (ex-dividend)
- El índice SPX refleja esta caída de precio
- Las opciones sobre SPX NO ajustan por dividendos

**Para Black-Scholes:**
```python
# Opciones SPX: usar dividend yield = 0 (o muy pequeño para ajustes)
precio_call_spx = black_scholes(S, K, T, r, sigma, q=0.0)
```

**Ventaja**: Modelos más limpios, sin complicaciones de dividendos.

### 5.2 SPY: ETF que Paga Dividendos

**Concepto**: SPY es un ETF que recibe los dividendos de las acciones subyacentes y los distribuye a los holders del ETF.

**Implicaciones:**
- SPY paga dividendos trimestrales (~1.5% anual)
- Si posees acciones de SPY (al hacer delta-hedge), recibes estos dividendos
- Las opciones sobre SPY ajustan su precio por expectativa de dividendos

**Para Black-Scholes:**
```python
# Opciones SPY: incluir dividend yield esperado
precio_call_spy = black_scholes(S, K, T, r, sigma, q=0.015)
```

**Impacto en delta-hedging:**
```
Beneficio adicional al hedgear con acciones SPY:
- Tienes posición larga en SPY para cubrir delta
- Ejemplo: 100 acciones × 500 USD = 50,000 USD
- Dividend yield: 1.5% anual
- Dividendo anual esperado: 750 USD
- Para posición de 30 días: 750/12 ≈ 62 USD

Este ingreso adicional compensa parcialmente el theta decay del straddle.
```

**Complicación**: Necesitas ajustar el cálculo de griegas con el dividend yield correcto.

---

## 6. Cambios Necesarios en el Código

Si hubiéramos usado SPX en lugar de SPY, estos serían los principales cambios:

### 6.1 Ajuste de Tamaño de Contrato

```python
# CON SPY (código actual)
def construir_long_straddle(ticker='SPY', dias_vencimiento=30):
    precio_subyacente = obtener_precio(ticker)  # ~500 USD
    notional = precio_subyacente × 100  # ~50,000 USD
    # ... resto del código ...

# CON SPX (modificado)
def construir_long_straddle(ticker='SPX', dias_vencimiento=30):
    precio_subyacente = obtener_precio(ticker)  # ~5,000 USD
    notional = precio_subyacente × 100  # ~500,000 USD
    
    # Advertencia al usuario sobre tamaño
    print(f"ATENCIÓN: Valor nocional = {notional:,.0f} USD")
    print("Esto requiere cuenta apropiada para el tamaño")
    # ... resto del código ...
```

### 6.2 Delta-Hedging

```python
# CON SPY (código actual)
def aplicar_delta_hedge(info_straddle, precios_historicos):
    delta = calcular_delta_straddle(info_straddle)
    acciones_necesarias = -delta × 100
    
    # Ejecutar: comprar/vender acciones de SPY
    operar_acciones('SPY', acciones_necesarias)

# CON SPX (opción usando SPY como proxy)
def aplicar_delta_hedge(info_straddle, precios_historicos):
    delta = calcular_delta_straddle(info_straddle)
    
    # 1 SPX ≈ 10 SPY, entonces necesitamos 10× las acciones
    acciones_spy_necesarias = -delta × 100 × 10
    
    # Ejecutar: comprar/vender acciones de SPY (como proxy)
    operar_acciones('SPY', acciones_spy_necesarias)
    
    print(f"Nota: Usando SPY como proxy para hedgear SPX")

# CON SPX (opción usando futuros ES)
def aplicar_delta_hedge(info_straddle, precios_historicos):
    delta = calcular_delta_straddle(info_straddle)
    puntos_indice = delta × 100  # Delta en "puntos de índice"
    
    # 1 futuro ES = 50 puntos, 1 opción SPX = 100 puntos
    futuros_necesarios = puntos_indice / 50
    
    # Redondear (problema de granularidad)
    futuros_a_operar = round(futuros_necesarios)
    
    if abs(futuros_necesarios - futuros_a_operar) > 0.3:
        print(f"ADVERTENCIA: Error de hedge significativo")
        print(f"Óptimo: {futuros_necesarios:.2f}, Operando: {futuros_a_operar}")
    
    operar_futuros('ES', futuros_a_operar)
```

### 6.3 Cálculo de Griegas (Dividendos)

```python
# CON SPY (incluir dividend yield)
def calcular_griegas_spy(S, K, T, r, sigma):
    q = 0.015  # Dividend yield de SPY (~1.5% anual)
    
    # Black-Scholes ajustado
    d1 = (np.log(S/K) + (r - q + 0.5*sigma**2)*T) / (sigma*np.sqrt(T))
    d2 = d1 - sigma*np.sqrt(T)
    
    # Griegas con dividendos
    delta_call = np.exp(-q*T) * norm.cdf(d1)
    # ... resto de griegas ...
    
    return {'delta': delta_call, ...}

# CON SPX (sin dividend yield o muy pequeño)
def calcular_griegas_spx(S, K, T, r, sigma):
    q = 0.0  # SPX es índice de precios
    
    # Black-Scholes estándar
    d1 = (np.log(S/K) + (r + 0.5*sigma**2)*T) / (sigma*np.sqrt(T))
    d2 = d1 - sigma*np.sqrt(T)
    
    delta_call = norm.cdf(d1)
    # ... resto de griegas ...
    
    return {'delta': delta_call, ...}
```

### 6.4 Gestión de Ejercicio Anticipado

```python
# CON SPY (monitorear ejercicio anticipado)
def monitorear_posicion(info_straddle, precio_actual):
    # Calcular moneyness
    call_itm = precio_actual - info_straddle['strike']
    put_itm = info_straddle['strike'] - precio_actual
    
    # Alertas de riesgo de ejercicio anticipado
    if call_itm > 20:  # Deep ITM
        print("ALERTA: Call muy ITM, riesgo de ejercicio anticipado")
        print("Considera cerrar posición o ajustar")
    
    if put_itm > 20:  # Deep ITM
        print("ALERTA: Put muy ITM, riesgo de ejercicio anticipado")
        print("Considera cerrar posición o ajustar")

# CON SPX (no necesario, ejercicio solo al vencimiento)
def monitorear_posicion(info_straddle, precio_actual):
    # Solo monitorear P&L y griegas
    # No preocuparse por ejercicio anticipado
    pass
```

### 6.5 Cálculo de P&L con Impuestos

```python
# CON SPY
def calcular_pnl_neto_impuestos(pnl_bruto, dias_holding):
    if dias_holding < 365:
        # Short-term capital gains
        tasa = 0.37  # Ejemplo: bracket más alto
    else:
        # Long-term capital gains
        tasa = 0.20
    
    impuestos = pnl_bruto × tasa
    pnl_neto = pnl_bruto - impuestos
    
    return pnl_neto, impuestos

# CON SPX (Sección 1256)
def calcular_pnl_neto_impuestos(pnl_bruto, dias_holding):
    # Independiente del holding period: siempre 60/40
    tasa_efectiva = 0.60 × 0.20 + 0.40 × 0.37
    # = 0.12 + 0.148 = 0.268
    
    impuestos = pnl_bruto × tasa_efectiva
    pnl_neto = pnl_bruto - impuestos
    
    return pnl_neto, impuestos
```

---

## 7. Resultados Esperados: ¿Cambiarían las Conclusiones?

### 7.1 Patrones de P&L

**Conceptualmente idénticos:**
- Mismo comportamiento de griegas (escalado 10×)
- Mismo patrón de gamma scalping
- Misma exposición a vega y theta
- Mismas conclusiones sobre delta-hedging vs sin hedge

**Ejemplo:**
```
Straddle SPY:
- Inversión: 1,500 USD
- Gamma: +0.05
- Movimiento de 1% en SPY: ~5 USD de precio
- Beneficio de gamma: ~0.05 × 25 × 100 = 125 USD (aprox)

Straddle SPX:
- Inversión: 15,000 USD (10× más)
- Gamma: +0.05 (mismo valor por "punto de índice")
- Movimiento de 1% en SPX: ~50 USD de precio
- Beneficio de gamma: ~0.05 × 2500 × 100 = 1,250 USD (10× más)

Retorno porcentual: Idéntico en ambos casos
```

### 7.2 Diferencias en Resultados Netos

**Factores que SÍ cambiarían el resultado final:**

1. **Costos de transacción** (no simulados en nuestro proyecto):
   - SPX: comisiones sobre 1 contrato grande
   - SPY: comisiones sobre 10 contratos pequeños (o 1 si ajustamos tamaño)
   - Impacto: menor con SPX para posiciones grandes

2. **Spreads bid-ask**:
   - SPX: spreads más estrechos porcentualmente en ATM
   - SPY: spreads muy competitivos pero ligeramente más amplios porcentualmente
   - Impacto: ~5-10 USD de diferencia en entrada/salida

3. **Impuestos** (diferencia significativa):
   - SPX: ~26.8% de tasa efectiva
   - SPY: ~37% si short-term
   - En ganancia de 10,000 USD: diferencia de ~1,000 USD

4. **Dividendos** (si hedgeamos con SPY):
   - SPY: ~750 USD anuales por cada 50,000 USD de hedge
   - Para 30 días: ~60 USD de ingreso adicional
   - Beneficio exclusivo de usar SPY para hedge

### 7.3 Conclusiones Invariantes

**Las siguientes conclusiones serían IDÉNTICAS con SPX o SPY:**

1. Delta-hedging reduce volatilidad del P&L pero no necesariamente aumenta el retorno total
2. La estrategia es una apuesta por volatilidad realizada vs volatilidad implícita
3. Gamma scalping genera beneficios en mercados volátiles
4. Theta decay es el coste constante de la estrategia
5. Vega exposure beneficia de aumentos de IV
6. Riesgo de legging al ejecutar patas sueltas
7. Neutralizar delta con opciones afecta gamma, vega y theta

---

## 8. Decisión Final: ¿Por qué Elegimos SPY?

### Justificación para Este Proyecto Académico

1. **Accesibilidad pedagógica**:
   - Tamaño manejable para estudiantes y cuentas retail
   - Más fácil de visualizar mentalmente (500 USD vs 5,000 USD)
   - Requiere menos capital para replicar en cuenta real

2. **Simplicidad del delta-hedging**:
   - Hedgear con el ETF directamente es más didáctico
   - Ratio 1:1 es más intuitivo
   - Evita complejidad de futuros o ratios 10:1
   - Permite enfocarse en conceptos sin complicaciones operativas

3. **Disponibilidad de datos**:
   - SPY tiene datos históricos más accesibles
   - APIs y brokers tienen mejor soporte para SPY en cuentas retail
   - Más fácil de obtener datos intradiarios y de opciones

4. **Generalización de conceptos**:
   - Todo lo aprendido con SPY es directamente aplicable a SPX (escalado)
   - Los conceptos de griegas, hedging, gamma scalping son idénticos
   - La complejidad adicional de SPX no añade valor pedagógico significativo

5. **Flexibilidad**:
   - Permite trabajar con cantidades fraccionales de capital
   - Fácil de escalar hacia arriba o abajo según necesidad
   - Ideal para backtesting con diferentes tamaños de cuenta

### Cuándo Deberías Usar SPX en la Práctica

Considera SPX si:

- Cuenta de trading > 100,000 USD
- Buscas eficiencia fiscal (Section 1256)
- Quieres evitar riesgo de ejercicio anticipado
- Operas con volúmenes grandes (institucional)
- Prefieres liquidación en cash vs acciones
- Te enfocas en vencimientos mensuales estándar
- Tienes acceso a futuros ES para hedging eficiente

Mantén SPY si:

- Cuenta de trading < 100,000 USD
- Prefieres simplicidad operativa
- Quieres hedgear con el subyacente directamente
- Operas frecuentemente con vencimientos semanales
- Beneficias de dividendos al mantener hedge
- Necesitas granularidad fina en el hedge
- Estás aprendiendo o enseñando conceptos

---

## 9. Resumen Ejecutivo

| Aspecto | SPX | SPY | Ganador para este proyecto |
|---------|-----|-----|----------------------------|
| Tamaño de posición | Grande (500k) | Pequeño (50k) | SPY |
| Delta-hedging | Complejo (futuros/proxy) | Simple (directo) | SPY |
| Fiscalidad | Ventajosa (26.8%) | Estándar (37%) | SPX |
| Ejercicio anticipado | No existe | Posible | SPX |
| Accesibilidad | Institucional | Retail | SPY |
| Liquidez | Excelente | Excelente | Empate |
| Dividendos | No | Sí (beneficio en hedge) | SPY |
| Complejidad operativa | Alta | Baja | SPY |
| Valor pedagógico | Igual | Igual | Empate |

**Conclusión**: Para un proyecto académico enfocado en aprender los conceptos de opciones, griegas, delta-hedging y gamma scalping, **SPY es la elección superior** por su simplicidad, accesibilidad y facilidad de implementación, sin sacrificar valor educativo.

Para trading profesional con cuentas grandes, **SPX ofrece ventajas fiscales y operativas** que justifican la complejidad adicional.