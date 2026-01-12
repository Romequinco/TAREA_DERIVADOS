# Solución para Error 10089: Datos de Mercado Requieren Suscripciones Adicionales

## Problema
El error `Error 10089: Los datos de mercado solicitados requieren suscripciones adicionales para API` ocurre cuando Interactive Brokers requiere suscripciones de pago para datos de mercado, incluso para datos delayed.

## Soluciones Prácticas

### Solución 1: Usar yfinance como Fallback (RECOMENDADA)

Esta es la solución más práctica si no tienes suscripciones de IBKR. Puedes usar `yfinance` para obtener precios del mercado de forma gratuita.

#### Código Helper a Agregar:

```python
def get_spy_price_with_fallback(ib=None, use_yfinance=True):
    """
    Obtiene el precio de SPY con fallback a yfinance si IBKR falla.
    
    Args:
        ib: Instancia de IB (opcional)
        use_yfinance: Si True, usa yfinance si IBKR falla
    
    Returns:
        Precio de SPY (float)
    """
    # Intentar con IBKR si está disponible
    if ib and ib.isConnected():
        try:
            # Configurar datos delayed ANTES de solicitar
            ib.reqMarketDataType(3)  # 3 = Delayed
            
            from ib_insync import Stock
            spy = Stock('SPY', 'SMART', 'USD')
            ib.qualifyContracts(spy)
            
            ticker = ib.reqMktData(spy, '', False, False)
            ib.sleep(2)  # Esperar datos
            
            if ticker.last and ticker.last > 0:
                return float(ticker.last)
            if ticker.close and ticker.close > 0:
                return float(ticker.close)
        except Exception as e:
            error_str = str(e)
            if '10089' in error_str or '10089' in repr(e):
                print(f"⚠️  Error 10089 detectado. Usando yfinance como fallback...")
            else:
                print(f"⚠️  Error con IBKR: {e}. Usando yfinance como fallback...")
    
    # Fallback a yfinance
    if use_yfinance:
        try:
            import yfinance as yf
            spy_ticker = yf.Ticker("SPY")
            hist = spy_ticker.history(period="1d", interval="1m")
            if not hist.empty:
                price = float(hist['Close'].iloc[-1])
                print(f"✓ Precio SPY obtenido de yfinance: ${price:.2f}")
                return price
        except Exception as e:
            print(f"❌ Error con yfinance: {e}")
    
    raise RuntimeError("No se pudo obtener precio de SPY (ni IBKR ni yfinance funcionaron)")


def get_market_data_safe(ib, contract, generic_tick_list='', snapshot=False):
    """
    Solicita datos de mercado de forma segura con manejo de error 10089.
    
    Args:
        ib: Instancia de IB
        contract: Contrato de IB
        generic_tick_list: Lista de ticks genéricos
        snapshot: Si True, solicita snapshot
    
    Returns:
        Ticker object o None si falla
    """
    try:
        # Asegurar que estamos usando datos delayed
        if ib.isConnected():
            ib.reqMarketDataType(3)  # 3 = Delayed
        
        ticker = ib.reqMktData(contract, generic_tick_list, snapshot, False)
        ib.sleep(1)  # Esperar un momento
        return ticker
    except Exception as e:
        error_str = str(e) + repr(e)
        if '10089' in error_str:
            print(f"⚠️  Error 10089: No hay suscripción de datos de mercado para {contract.symbol}")
            print(f"   Sugerencia: Usa yfinance o datos históricos como alternativa")
        return None
```

### Solución 2: Usar reqHistoricalData() en lugar de reqMktData()

Los datos históricos generalmente NO requieren suscripciones adicionales. Puedes obtener el último precio usando datos históricos:

```python
def get_price_from_historical(ib, contract):
    """
    Obtiene el precio usando datos históricos (no requiere suscripciones).
    """
    try:
        bars = ib.reqHistoricalData(
            contract,
            endDateTime='',
            durationStr='1 D',
            barSizeConfig='1 min',
            whatToShow='TRADES',
            useRTH=True,
            formatDate=1
        )
        if bars:
            return float(bars[-1].close)
    except Exception as e:
        print(f"Error obteniendo datos históricos: {e}")
    return None
```

### Solución 3: Configurar reqMarketDataType(3) ANTES de cada solicitud

**IMPORTANTE**: Debes llamar `ib.reqMarketDataType(3)` ANTES de cualquier `reqMktData()`, y debe hacerse DESPUÉS de conectar pero ANTES de solicitar datos.

```python
# Ejemplo correcto:
ib = IB()
ib.connect('127.0.0.1', 7497, clientId=1)

# ⚠️ IMPORTANTE: Configurar datos delayed ANTES de solicitar datos
ib.reqMarketDataType(3)  # 3 = Delayed data

# Ahora puedes solicitar datos
spy = Stock('SPY', 'SMART', 'USD')
ib.qualifyContracts(spy)
ticker = ib.reqMktData(spy, '', False, False)
```

### Solución 4: Suscribirse a Datos de Mercado en IBKR (Si tienes cuenta PRO)

Si tienes una cuenta IBKR PRO con saldo suficiente:

1. Ve al Portal del Cliente de IBKR
2. Navega a "Configuración" > "Datos de Mercado"
3. Suscríbete a "US Equity and Options Add-On Streaming Bundle" (gratis para cuentas activas)
4. O suscríbete a datos específicos según tus necesidades

**Nota**: Las cuentas PAPER/DEMO generalmente no pueden suscribirse a datos de mercado.

## Recomendación Final

Para desarrollo y pruebas, **usa la Solución 1 (yfinance como fallback)**. Es gratuita, confiable, y funciona bien para la mayoría de casos de uso. Solo necesitarás datos de IBKR si requieres:
- Datos en tiempo real exactos
- Griegas del modelo de IBKR
- Datos de opciones en tiempo real

Para análisis y backtesting, yfinance es perfectamente adecuado.

## Cómo Implementar

1. Agrega las funciones helper (`get_spy_price_with_fallback`, `get_market_data_safe`) al inicio de tu notebook
2. Reemplaza llamadas a `ib.reqMktData()` por `get_market_data_safe()` donde sea posible
3. Usa `get_spy_price_with_fallback()` para obtener el precio de SPY

## Ejemplo de Uso

```python
# En lugar de:
# spy = Stock('SPY', 'SMART', 'USD')
# ib.qualifyContracts(spy)
# ticker = ib.reqMktData(spy, '', False, False)
# S = ticker.last

# Usa:
S = get_spy_price_with_fallback(ib, use_yfinance=True)
print(f"Precio SPY: ${S:.2f}")
```
