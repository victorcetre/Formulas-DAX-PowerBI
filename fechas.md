# Fórmulas para Fechas en DAX

Aquí encontrarás fórmulas relacionadas con fechas, como la creación de tablas de calendario y la segmentación por periodos.

## Dim_Calendar DAX

**Descripción:** Esta fórmula crea una tabla de calendario para segmentar datos por año, trimestre, mes, semana, etc.

```DAX
Dim_Calendar DAX = ADDCOLUMNS (
  CALENDAR ( DATE( YEAR ( MIN ( Tabla_Proyectos[Fecha de Inicio] )), 01, 01), DATE( YEAR( MAX(Tabla_Proyectos[Fecha Fin] ) ), 12, 31 ) ),
  "FechaSK", FORMAT ( [Date], "YYYYMMDD" ),
  "#Año", YEAR ( [Date] ),
  "#Trimestre", QUARTER ( [Date] ),
  "Mes", FORMAT ( [Date], "MMMM" ),
  ...
)
