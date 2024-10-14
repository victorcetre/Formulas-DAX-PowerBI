# Fórmulas de KPI en DAX

Indicadores clave de rendimiento y métricas de negocio.

## Tasa de Crecimiento de Ventas

**Descripción:** Calcula la tasa de crecimiento de las ventas entre dos periodos.

```DAX
Tasa_Crecimiento = DIVIDE([Ventas Actual], [Ventas Periodo Anterior], 0) - 1
