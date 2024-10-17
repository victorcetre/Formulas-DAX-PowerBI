
# Informe Detallado para la Gestión de Disponibilidad de Consultores en Power BI

## Introducción

Este informe presenta un enfoque detallado para gestionar la **disponibilidad de consultores** en Power BI, integrando cálculos personalizados que contemplan ausencias parciales (como permisos médicos) y completas (como vacaciones). El objetivo es facilitar la planificación y asignación de recursos, y permitir el análisis de la disponibilidad de los consultores y roles en función de sus horas asignadas, sus ausencias y las excepciones, como festivos que afectan los días laborables.

## Objetivo

El principal objetivo es diseñar un sistema robusto en Power BI que permita:

1. Controlar la disponibilidad de los consultores para sus distintos roles y proyectos.
2. Gestionar las ausencias (vacaciones, permisos) y excepciones de días laborables.
3. Facilitar la toma de decisiones sobre la contratación de nuevos recursos cuando los roles están sobrecargados.
4. Visualizar claramente los periodos en los que los consultores no estarán disponibles.

## Proceso General

### 1. Disponibilidad Base Semanal

Cada consultor tiene una **disponibilidad base estándar de 40 horas por semana**. Esta cantidad puede variar dependiendo de la asignación de horas a diferentes proyectos y roles, y de las ausencias registradas (vacaciones, permisos).

### 2. Registro de Asignaciones por Proyecto y Rol

Se registra semanalmente cuántas horas están asignadas a cada consultor para sus diferentes roles. Por ejemplo:

- **Consultor A** tiene asignadas 15 horas en el Rol 1 y 10 horas en el Rol 2.
- La **disponibilidad restante** del consultor se calcula restando las horas asignadas de las 40 horas base:
  
  \[
  	ext{Disponibilidad restante} = 40 - (	ext{Horas Rol 1} + 	ext{Horas Rol 2})
  \]

### 3. Registro de Ausencias

Se lleva un registro de las ausencias del consultor, ya sea por permisos o vacaciones. Este registro incluye los siguientes campos:

- **Consultor**: Nombre o ID.
- **Tipo de Ausencia**: Vacaciones, permiso médico, cita, etc.
- **Fecha de Inicio y Fin**: El periodo de la ausencia.
- **Horas Perdidas**: En caso de ausencias parciales (como citas médicas).
- **Comentarios**: Detalles adicionales del permiso.

#### Ejemplo de tabla de ausencias:

| Consultor     | Tipo de Ausencia | Fecha de Inicio | Fecha de Fin | Horas Perdidas | Comentarios       |
|---------------|------------------|-----------------|--------------|----------------|-------------------|
| Consultor X   | Vacaciones        | 14-Oct          | 18-Oct       | N/A            | Vacaciones anuales|
| Consultor Y   | Cita médica       | 15-Oct          | 15-Oct       | 2 horas        | De 2 PM a 4 PM    |
| Consultor Z   | Permiso personal  | 13-Oct          | 13-Oct       | 1 hora         | Cita personal     |

### 4. Cálculos de Disponibilidad

La disponibilidad semanal total de cada consultor se calcula restando las horas asignadas y las horas de ausencia de las 40 horas base.

- **Fórmula de Disponibilidad Semanal**:

  \[
  	ext{Disponibilidad semanal} = 40 - (	ext{Horas asignadas a proyectos} + 	ext{Horas de ausencias})
  \]

### 5. Cálculos de Disponibilidad por Roles Múltiples

Si un consultor desempeña varios roles, se suman las horas asignadas a cada uno y luego se calcula la disponibilidad restante. Si el consultor tiene 35 horas asignadas en total, solo le quedarían 5 horas disponibles para asignar a otros roles.

---

## Implementación en Power BI

### 1. Calendario de Días Laborales y Festivos

Se crea un calendario en Power BI usando DAX que incluye información de días laborables y festivos. Esto es fundamental para ajustar los cálculos de disponibilidad en función de los días no laborables y cualquier excepción.

#### **DAX para el Calendario de Días Laborables con Festivos**:

```DAX
Dim_Calendar = ADDCOLUMNS (
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
```

### 2. Días Festivos y Lógica de Excepciones

Se incluye una columna calculada que determina si un día es laboral teniendo en cuenta festivos. Además, se considera una excepción: si el festivo cae en un miércoles o viernes, ese día se convierte en laboral y el lunes siguiente es no laborable.

#### DAX para la columna `EsDíaLaboral`:

```DAX
EsDíaLaboral = 
VAR EsFestivo = NOT ( ISBLANK ( RELATED ( Tabla_Festivos[Fecha Festivo] ) ) )
VAR EsMiércolesOViernes = WEEKDAY ( [Date], 2 ) = 3 || WEEKDAY ( [Date], 2 ) = 5
VAR EsLunesSiguiente = WEEKDAY ( [Date] - 3, 2 ) = 1 && NOT ( ISBLANK ( RELATED ( Tabla_Festivos[Fecha Festivo] ) ) )
RETURN 
IF (
    (EsFestivo && EsMiércolesOViernes),
    "Sí",  // Si es festivo y cae miércoles o viernes, se considera laboral
    IF (
        (EsLunesSiguiente),
        "No",  // Si es el lunes después de un festivo en miércoles o viernes, no se trabaja
        IF (
            WEEKDAY ( [Date], 2 ) <= 5 && NOT ( EsFestivo ),
            "Sí",  // Caso normal, día laboral de lunes a viernes sin ser festivo
            "No"  // Caso general no laboral (fines de semana y festivos regulares)
        )
    )
)
```

---

## Visualización y Cálculos en Power BI

### 1. Tabla o Matriz de Disponibilidad

Se puede crear una tabla que muestre las horas disponibles por semana para cada consultor, excluyendo los días festivos y las ausencias parciales. Además, esta tabla puede filtrar los días laborables en función de la columna `EsDíaLaboral`.

### 2. Calendario de Ausencias

Utilizando un gráfico de Gantt o un calendario visual, es posible mostrar las ausencias y días laborables. Los días festivos que caen en miércoles o viernes se mostrarán como laborables, y los lunes siguientes como no laborables.

### 3. KPI de Disponibilidad

Para visualizar rápidamente cuántas horas disponibles tiene un consultor en una semana específica, se pueden utilizar KPIs en Power BI. Estos KPIs filtrarán las horas disponibles excluyendo fines de semana y festivos, según la lógica implementada.

---

## Conclusión

Este informe presenta un sistema flexible y robusto para gestionar la disponibilidad de consultores en Power BI, considerando tanto asignaciones de proyectos como ausencias parciales y festivos. Con la implementación de este calendario personalizado y las visualizaciones adecuadas, los líderes de proyectos podrán planificar mejor los recursos y gestionar la carga de trabajo de los consultores de manera eficiente.


Informe de Propuesta para la Gestión de Disponibilidad de Consultores en Power BI
Introducción
Este informe detalla la estrategia propuesta para gestionar la disponibilidad de consultores en proyectos, incorporando ausencias parciales (como permisos médicos) y ausencias completas (como vacaciones). La solución tiene como objetivo optimizar la asignación de recursos, mejorar la planificación de proyectos y facilitar la toma de decisiones en la gestión de equipos.

1. Objetivo
El objetivo principal es crear un sistema eficiente de registro y cálculo de la disponibilidad de los consultores en función de:

Asignaciones de proyectos y roles.
Ausencias parciales y completas, como permisos médicos, citas y vacaciones.
El sistema permitirá:

Determinar si un consultor está disponible para asignaciones en función de su carga de trabajo y ausencias.
Controlar la necesidad de contratación de nuevos recursos según las demandas de los roles.
Visualizar la disponibilidad y las ausencias del equipo de manera clara y accesible.
2. Proceso General
2.1. Disponibilidad Base Semanal
Cada consultor tiene una disponibilidad base estándar de 40 horas por semana, que puede variar en función de asignaciones de roles y proyectos, así como de ausencias registradas (vacaciones, permisos).

2.2. Registro de Asignaciones por Proyecto y Rol
La asignación de horas a proyectos y roles se registra semanalmente para cada consultor. Por ejemplo:

Consultor A tiene asignadas 15 horas para el Rol 1 y 10 horas para el Rol 2.
La disponibilidad restante del consultor se calcula restando las horas asignadas de las 40 horas base:
Disponibilidad restante
=
40
−
(
Horas Rol 1
+
Horas Rol 2
)
Disponibilidad restante=40−(Horas Rol 1+Horas Rol 2)
2.3. Registro de Ausencias
Para gestionar ausencias parciales (citas médicas, permisos por horas) y ausencias completas (vacaciones), se propone una tabla de ausencias centralizada que contiene los siguientes campos:

Consultor: Nombre o ID.
Tipo de Ausencia: Vacaciones, cita médica, permiso personal, etc.
Fecha de Inicio y Fin: El período de la ausencia.
Horas Perdidas: En el caso de ausencias parciales, se registra cuántas horas el consultor no estará disponible.
Comentarios Adicionales: Para especificar detalles adicionales del permiso.
Ejemplo de la tabla de ausencias:

Consultor	Tipo de Ausencia	Fecha de Inicio	Fecha de Fin	Horas Perdidas	Comentarios
Consultor X	Vacaciones	14-Oct	18-Oct	N/A	Vacaciones anuales
Consultor Y	Cita médica	15-Oct	15-Oct	2 horas	De 2 PM a 4 PM
Consultor Z	Permiso personal	13-Oct	13-Oct	1 hora	Cita personal
3. Cálculos y Procedimientos
3.1. Disponibilidad Total Semanal
La disponibilidad semanal total de cada consultor se calcula restando las horas asignadas y las horas de ausencia de las 40 horas base.

Fórmula de Disponibilidad Semanal:

Disponibilidad semanal
=
40
−
(
Horas asignadas a proyectos
+
Horas de ausencias
)
Disponibilidad semanal=40−(Horas asignadas a proyectos+Horas de ausencias)
3.2. Gestión de Roles Múltiples
Si un consultor tiene asignaciones en múltiples roles, el sistema debe sumar las horas asignadas a cada rol y luego calcular la disponibilidad restante. Si un consultor ya tiene 35 horas asignadas a dos roles, por ejemplo, solo tendrá 5 horas disponibles para cualquier otro rol adicional.

3.3. Incorporación de Ausencias
Las ausencias parciales o totales se restan del total de horas disponibles. Si el consultor está de vacaciones por una semana completa, su disponibilidad será 0 horas. Si tiene un permiso de 2 horas en un día, esa cantidad de tiempo se resta de las horas disponibles ese día o semana.

Ejemplo:

Disponibilidad base: 40 horas.
Horas asignadas a proyectos: 35 horas.
Permiso médico: 2 horas (cita médica).
Disponibilidad restante
=
40
−
35
−
2
=
3
 
horas disponibles en la semana
.
Disponibilidad restante=40−35−2=3horas disponibles en la semana.
4. Visualización y Análisis en Power BI
4.1. Visualización de Disponibilidad por Consultor
En Power BI, se implementará una tabla o matriz que muestre la disponibilidad de cada consultor por semana, junto con las ausencias registradas, lo que permite a los líderes de proyectos ver fácilmente cuándo los consultores estarán disponibles o ausentes.

Ejemplo de matriz en Power BI:

Consultor	Semana del 14-Oct	Semana del 21-Oct	Semana del 28-Oct
Consultor X	0 horas (vacaciones)	40 horas	40 horas
Consultor Y	38 horas (2h cita)	40 horas	40 horas
Consultor Z	39 horas (1h permiso)	40 horas	40 horas
4.2. Calendario de Ausencias
Se puede visualizar un calendario interactivo o un gráfico de Gantt para representar las ausencias de los consultores de manera visual y clara. Cada día o semana con ausencias completas (vacaciones) o parciales (permisos médicos) puede destacarse en colores, facilitando la planificación de recursos.

4.3. Indicadores de Disponibilidad
Se pueden crear KPIs o indicadores en Power BI para mostrar el total de horas disponibles de cada consultor, permitiendo una visión rápida de cuántas horas quedan disponibles para asignaciones en cada semana.

5. Recomendaciones Finales
La estructuración propuesta permite un control eficiente de la disponibilidad de los consultores, ya que combina el seguimiento de ausencias con las asignaciones de proyectos y roles, sin necesidad de llevar un registro manual diario. Esta solución también permite identificar de manera proactiva las necesidades de contratación cuando no haya consultores suficientes para cubrir los roles.

La implementación de esta solución en Power BI ofrece a los líderes de proyectos una visión clara y en tiempo real de la disponibilidad del equipo, facilitando la planificación de recursos y la toma de decisiones.

#### **DAX para el Calendario de Días Laborables con Festivos**:

```DAX
Dias_Festivos = 
VAR AnosDisponibles = DISTINCT ( SELECTCOLUMNS ( Calendario, "Año", YEAR ( Calendario[Date] ) ) ) -- Obtenemos los años únicos de la tabla Calendario
RETURN
    GENERATE (
        AnosDisponibles,
        ADDCOLUMNS (
            'Festivos',
            "Fecha Completa", DATE ( VALUE ( [Año] ), 'Festivos'[Mes], 'Festivos'[Dia] ), -- Crea la fecha completa utilizando el año del calendario
            "Festivo", 'Festivos'[Conmemoración] -- Asigna el nombre del festivo
        )
    )
