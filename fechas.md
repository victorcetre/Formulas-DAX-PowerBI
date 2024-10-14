# Fórmulas para Fechas en DAX

Aquí encontrarás fórmulas relacionadas con fechas, como la creación de tablas de calendario y la segmentación por periodos.

## Dim_Calendar DAX

**Descripción:** Esta fórmula DAX crea una tabla de calendario que abarca desde la primera fecha registrada en una columna de fechas iniciales hasta la última fecha registrada en una columna de fechas finales en tu modelo de datos. La tabla generada permite segmentar los datos por distintos periodos de tiempo, como año, semestre, trimestre, mes, semana, etc., y facilita la creación de análisis basados en el tiempo. Cada columna adicional proporciona detalles específicos relacionados con las fechas, como el año, semestre, trimestre, mes, semana, día, y más.

```DAX
Dim_Calendar DAX = ADDCOLUMNS (
  CALENDAR (
    DATE( YEAR ( MIN ( [Fecha Inicial] )), 01, 01),   -- Fecha de inicio basada en la columna con las fechas más antiguas
    DATE( YEAR ( MAX ( [Fecha Final] )), 12, 31)      -- Fecha de fin basada en la columna con las fechas más recientes
  ),
  "FechaSK", FORMAT ( [Date], "YYYYMMDD" ),            -- Clave de fecha (formato YYYYMMDD)
  "#Año", YEAR ( [Date] ),                             -- Año
  "#Semestre", IF (MONTH([Date]) <= 6, 1, 2),          -- Semestre (1 para enero-junio, 2 para julio-diciembre)
  "#Trimestre", QUARTER ( [Date] ),                    -- Trimestre
  "#Mes", MONTH ( [Date] ),                            -- Mes (en formato numérico)
  "#Día", DAY ( [Date] ),                              -- Día del mes
  "Semestre", "S" & IF (MONTH([Date]) <= 6, 1, 2),     -- Semestre (formato "S1" o "S2")
  "Trimestre", "T" & FORMAT ( [Date], "Q" ),           -- Trimestre (formato "T1", "T2", etc.)
  "Mes", FORMAT ( [Date], "MMMM" ),                    -- Mes (nombre completo del mes)
  "MesCorto", FORMAT ( [Date], "MMM" ),                -- Mes abreviado (ejemplo: Ene, Feb, etc.)
  "#DíaSemana", WEEKDAY ( [Date], 2 ),                 -- Día de la semana (numérico, 1 para lunes)
  "#SemanaAño", WEEKNUM ( [Date], 2 ),                 -- Semana del año (numérico)
  "CierreSemana", ( [Date] + 7 - WEEKDAY( [Date], 2 ) ),-- Fecha de cierre de semana (próximo domingo)
  "Día", FORMAT ( [Date], "DDDD" ),                    -- Día de la semana (nombre completo: lunes, martes, etc.)
  "DíaCorto", FORMAT ( [Date], "DDD" ),                -- Día de la semana abreviado (Lun, Mar, etc.)
  "AñoSemestre", FORMAT([Date], "YYYY") & "/S" & IF(MONTH([Date]) <= 6, 1, 2),  -- Año y semestre combinados
  "AñoTrimestre", FORMAT ( [Date], "YYYY" ) & "/T" & FORMAT ( [Date], "Q" ), -- Año y trimestre combinados
  "Año#Mes", FORMAT ( [Date], "YYYY/MM" ),             -- Año y mes combinados
  "AñoMesCorto", FORMAT ( [Date], "YYYY/mmm" ),        -- Año y mes abreviado combinados
  "InicioMes", EOMONTH( [Date], -1) + 1,               -- Primer día del mes
  "FinMes", EOMONTH( [Date], 0)                        -- Último día del mes
)
```

```DAX Sin Comentarios
Dim_Calendar DAX = ADDCOLUMNS (
  CALENDAR (
    DATE( YEAR ( MIN ( [Fecha Inicial] )), 01, 01),   
    DATE( YEAR ( MAX ( [Fecha Final] )), 12, 31)),
  "FechaSK", FORMAT ( [Date], "YYYYMMDD" ),            
  "#Año", YEAR ( [Date] ),                             
  "#Semestre", IF (MONTH([Date]) <= 6, 1, 2),          
  "#Trimestre", QUARTER ( [Date] ),                    
  "#Mes", MONTH ( [Date] ),                            
  "#Día", DAY ( [Date] ),                              
  "Semestre", "S" & IF (MONTH([Date]) <= 6, 1, 2),     
  "Trimestre", "T" & FORMAT ( [Date], "Q" ),           
  "Mes", FORMAT ( [Date], "MMMM" ),                    
  "MesCorto", FORMAT ( [Date], "MMM" ),                
  "#DíaSemana", WEEKDAY ( [Date], 2 ),                 
  "#SemanaAño", WEEKNUM ( [Date], 2 ),                 
  "CierreSemana", ( [Date] + 7 - WEEKDAY( [Date], 2 ) ),
  "Día", FORMAT ( [Date], "DDDD" ),                    
  "DíaCorto", FORMAT ( [Date], "DDD" ),                
  "AñoTrimestre", FORMAT ( [Date], "YYYY" ) & "/T" & FORMAT ( [Date], "Q" ), 
  "Año#Mes", FORMAT ( [Date], "YYYY/MM" ),             
  "AñoMesCorto", FORMAT ( [Date], "YYYY/mmm" ),        
  "AñoSemestre", FORMAT([Date], "YYYY") & "/S" & IF(MONTH([Date]) <= 6, 1, 2),  
  "InicioMes", EOMONTH( [Date], -1) + 1,               
  "FinMes", EOMONTH( [Date], 0))
