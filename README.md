# Módulo: Programación de Recursos de Cobranza (Scheduling)

## 1. Requisitos Funcionales Gerenciales (Definición de Parámetros)

Este módulo es crítico para establecer la política y la tarifa de cobranza, lo cual define los productos (servicios) ofrecidos,. Los parámetros son variables fijas que definen los escenarios posibles y la lógica del negocio.

| Caso de Uso (CU) | Nombre del Requisito Funcional | Detalle de la Función |
| :--- | :--- | :--- |
| **CU 1** | Configurar Tipologías y Parámetros de Servicio de Cobranza | Permitir al Gerente definir los diferentes productos o etapas de cobranza (e.g., Mora Temprana, Prejudicial, Judicial). La definición incluye: Rangos de Mora (días de atraso, e.g., Mora 1 a 10 días), Rangos de Monto (mínimo/máximo de la deuda), y si requiere Garantía o Aval (Codeudor, Hipoteca). |
| **CU 2** | Definir Protocolos y Canales de Gestión | Permitir asociar a cada tipología de cobranza (CU 1) su Protocolo específico de acciones. Esto incluye definir la Duración Máxima de la Campaña (e.g., 10 días), el Tipo y Frecuencia de Canales a utilizar (llamadas automáticas, emails, correos, visitas), y el tipo de Recurso Operativo requerido (operador simple, abogado, robot). |
| **CU 3** | Gestión de Recursos Operativos (Parque) | Registrar y mantener el inventario de recursos reales (operadores, robots de llamada, vehículos). Esto incluye definir la capacidad máxima diaria de cada operador (e.g., 50 llamadas preventivas al día) y los horarios de disponibilidad, ya que estos recursos se asignarán mediante tickets. |

## 2. Requisitos Funcionales de Proceso Masivo (Batch)

Los procesos de cobranza se manejan con grandes volúmenes de deudores, por lo que son típicamente procesos Batch.

| Caso de Uso (CU) | Nombre del Requisito Funcional | Detalle de la Función |
| :--- | :--- | :--- |
| **CU 4** | Recepción y Validación Masiva de Cartera | El sistema debe procesar un archivo de entrada con el lote de deudores (cartera) proporcionado por el cliente (sponsor). El proceso Batch debe validar la integridad y existencia de datos clave (e.g., DNI/RUC contra entidades externas). |
| **CU 5** | Clasificación Automática de Deudores y Asignación de Producto | El sistema, de forma automática, debe clasificar a cada deudor de la cartera (CU 4) y asignarle el producto de cobranza (Preventivo, Judicial, etc.). Esta clasificación se realiza comparando el monto y la mora del deudor contra los parámetros definidos en CU 1. |
| **CU 6** | Generación Masiva de Tickets de Gestión | Basándose en la cartera clasificada (CU 5) y la capacidad del Parque de Recursos (CU 3), el sistema debe generar automáticamente tickets que representan tareas específicas (e.g., "Llamar al deudor X el día Y a la hora Z"). Estos tickets se generan en lote para programar el trabajo futuro de los operadores/robots. |
| **CU 7** | Cambio Automático de Estado por Vencimiento (Gobernanza) | El sistema debe monitorear la duración de las campañas. Si un ticket o gestión de cobranza ha excedido su plazo máximo definido (ej. 10 días), el sistema debe cambiar automáticamente el estado del deudor al siguiente producto o etapa de cobranza (e.g., de Preventivo a Prejudicial), lo cual requerirá la generación de nuevos tickets. |

## 3. Requisitos Funcionales Operacionales (On-Line)

Estos requisitos se refieren a la interacción diaria de los operadores con el sistema y la provisión de información crítica en tiempo real.

| Caso de Uso (CU) | Nombre del Requisito Funcional | Detalle de la Función |
| :--- | :--- | :--- |
| **CU 8** | Consulta de Carga de Trabajo y Disponibilidad | Permitir a los operadores (o al sistema de asignación) consultar en tiempo real los tickets que le han sido asignados. También debe permitir a la gerencia consultar la disponibilidad futura de recursos para responder a nuevos requerimientos del cliente. |
| **CU 9** | Registro de Eventos de Cobranza (Data Entry) | Permitir al operador registrar la interacción con el deudor (llamada, email, visita, etc.), incluyendo el resultado, el compromiso de pago y comentarios en pantalla. Este proceso debe ser ágil, con navegación rápida entre pantallas para maximizar la productividad. |
| **CU 10** | Consulta Crítica de Deuda y Historial | Permitir a los gerentes de cuenta y operadores acceder a una Consulta Crítica de un deudor. Esta vista debe mostrar los datos completos del deudor, el monto adeudado (saldo), la mora actual, el historial de gestión y los compromisos de pago, accediendo a múltiples tablas de forma eficiente. |

---

## 4. Requisitos de Atributos de Calidad (No Funcionales)

Los atributos de calidad definen el comportamiento del sistema y son críticos para un módulo de alto volumen como Cobranzas.

| Atributo de Calidad | Descripción y Criterio Medible |
| :--- | :--- |
| **Rendimiento (Performance)** | 1. **Online (CU 9, CU 10):** Las transacciones críticas (Consulta Crítica, Registro de Gestión) deben ser altamente optimizadas (por ejemplo, mediante la creación de tablas suplementarias o redundancia controlada) para asegurar un tiempo de respuesta mínimo. El tiempo de respuesta aceptable debe ser menor a 2 segundos.<br>2. **Batch (CU 4, CU 6):** Los procesos masivos de actualización de cartera y generación de tickets deben ejecutarse dentro de la ventana de tiempo nocturna asignada (e.g., máximo 3 horas) para no interferir con las operaciones del día siguiente. |
| **Integridad y Seguridad** | El sistema debe garantizar la integridad de los saldos y la coherencia de los datos financieros. El proceso debe evitar la ruptura de Intervalos de Control (IC) o Áreas de Control (AC) en la base de datos para mantener la performance y la estabilidad física de las tablas maestras. El acceso a la información sensible del deudor debe ser restringido y auditado. |
| **Escalabilidad** | El módulo Batch debe estar diseñado para manejar un alto volumen de datos (varios miles o millones de deudores) sin degradar la performance. Debe soportar un crecimiento planificado en la cantidad de registros de movimiento (bitácora de gestión) que son voluminosos. |
| **Usabilidad (Online)** | La navegación entre pantallas del operador (CU 9) debe ser sencilla, idealmente siguiendo flujos de trabajo predeterminados para presentar automáticamente la información necesaria (nombre, teléfono, deuda). |

## 4. Diseño Conceptual del Módulo de Cobranza

### 4.1. Representación Gráfica (Modelo Conceptual – Notación de Chen)

El modelo se centra en la relación crítica entre el cliente deudor, la estrategia de cobranza asignada (Tipología) y el registro detallado de la tarea programada (Ticket/Gestión).



```mermaid
erDiagram
    CLIENTE ||--|{ TIPOLOGIA_COBRANZA : "Asignación de Estrategia"
    TIPOLOGIA_COBRANZA ||--|{ PROTOCOLO : "Definición de Reglas"
    TIPOLOGIA_COBRANZA ||--o{ RECURSO : "Recursos Habilitados"
    CLIENTE ||--|{ GESTION : "Tareas Programadas"
    RECURSO ||--|{ GESTION : "Asignación de Recurso"
    CARTERA ||--|{ CLIENTE : "Lote de Deudores"

    CLIENTE {
        string CodCliente PK
        string NroDocumento
        string Nombre
        float MontoPendiente
        int DiasMora
        string ClasificacionRiesgo
    }

    TIPOLOGIA_COBRANZA {
        string CodTipologia PK
        string Descripcion
        float MontoMin
        float MontoMax
        int MoraMin
        int MoraMax
    }

    PROTOCOLO {
        string CodProtocolo PK
        int DuracionMaxDias
        string CanalPreferente
        string SecuenciaAcciones
        string CodTipologia FK
    }

    RECURSO {
        string CodRecurso PK
        string TipoRecurso
        int CapacidadDiaria
        string HorarioDisp
    }
    
    GESTION {
        string IDTicket PK
        datetime FechaHoraProgramada
        string EstadoGestion
        string Resultado
        string CodCliente FK
        string CodRecurso FK
    }
    
    CARTERA {
        string CodLote PK
        string ClienteSponsor
        date FechaRecepcion
    }
```

### 4.2. Diccionario de Datos (Fichas de Tipos de Entidad)

#### Entidad: CLIENTE (Deudor)

| Nombre | Descripción | Propósito | Reglas de Negocio Relevantes |
| :--- | :--- | :--- | :--- |
| **CLIENTE** | Representa al deudor cuya cuenta está siendo gestionada por el módulo de cobranza. | Centralizar la información financiera y demográfica relevante para la gestión de la deuda. | **RF 10:** Debe contar con un código único (Cuenta o Código de Cliente). |

| Atributo | Descripción | Propósito | Dominio de Valores | Obligatoriedad | Unicidad | Multivaluado |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **CodCliente (PK)** | Código único del cliente en el sistema. | Clave principal y enlace a todas las gestiones y movimientos. | Texto (ej. CHAR 10) | Sí | Sí | No |
| **NroDocumento** | Número de identificación (DNI, RUC). | Validación de identidad (CU 4) y búsqueda (Consulta Crítica). | Texto (ej. CHAR 15) | Sí | Sí | No |
| **MontoPendiente** | Saldo total actual que el cliente adeuda. | Determinar la tipología de cobranza aplicable (CU 5) y el riesgo. | Dinero (Decimal 18,2) | Sí | No | No |
| **DiasMora** | Cantidad de días de atraso en el pago (mora actual). | Clave para la reclasificación automática (CU 7) y asignación de Tipología (CU 5). | Número (Entero) | Sí | No | No |
| **ClasificacionRiesgo** | Nivel de riesgo crediticio o de pérdida asociado al cliente. | Apoyo a la toma de decisiones gerenciales. | Enumeración (Bajo, Medio, Alto) | Sí | No | No |

#### Entidad: TIPOLOGIA\_COBRANZA (Producto)

| Nombre | Descripción | Propósito | Reglas de Negocio Relevantes |
| :--- | :--- | :--- | :--- |
| **TIPOLOGIA\_COBRANZA** | Define la estrategia o producto de gestión de deuda, basándose en rangos de monto y mora. | **RF 1, RF 2:** Define los escenarios posibles y la lógica del negocio para la programación (parámetros). | **CU 5:** Un deudor se clasifica automáticamente en una tipología según su mora y monto. |

| Atributo | Descripción | Propósito | Dominio de Valores | Obligatoriedad | Unicidad | Multivaluado |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **CodTipologia (PK)** | Código único que identifica el tipo de servicio (ej. 'P' para Preventivo). | Clave principal para asociar reglas y recursos a la estrategia. | Texto (ej. CHAR 5) | Sí | Sí | No |
| **Descripcion** | Nombre descriptivo del producto de cobranza. | Visualización en interfaces gerenciales. | Texto | Sí | No | No |
| **MontoMin** | Monto mínimo de deuda para aplicar esta tipología. | Definición de rango (parámetro). | Dinero | Sí | No | No |
| **MoraMax** | Mora máxima (en días) que puede tener un deudor para permanecer en esta tipología. | Disparador para el cambio de estado (CU 7). | Número (Entero) | Sí | No | No |

#### Entidad: RECURSO (Parque Operativo)

| Nombre | Descripción | Propósito | Reglas de Negocio Relevantes |
| :--- | :--- | :--- | :--- |
| **RECURSO** | Entidad que representa la capacidad física y humana disponible para ejecutar las tareas. | **RF 3:** Gestionar el inventario (parque) y la disponibilidad para la generación de tickets. | **CU 6:** La generación de tickets depende de la CapacidadDiaria de cada recurso. |

| Atributo | Descripción | Propósito | Dominio de Valores | Obligatoriedad | Unicidad | Multivaluado |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **CodRecuro (PK)** | Identificador único del recurso (ej. Operador 001, Robot 005). | Programación de tareas y monitoreo de productividad. | Texto (ej. CHAR 8) | Sí | Sí | No |
| **TipoRecurso** | Rol o naturaleza del recurso asignable. | Clasificación para las reglas de asignación (e.g., solo abogados para Judicial). | Enumeración (Operador, Abogado, Robot, Vehículo) | Sí | No | No |
| **CapacidadDiaria** | Número máximo de tickets o tareas que el recurso puede manejar por día. | Parámetro clave para la Generación Batch de Tickets (CU 6). | Número (Entero) | Sí | No | No |
| **HorarioDisp** | Período de tiempo en que el recurso está disponible para ser programado. | Control de asignación de tareas. | Texto (ej. HH:MM-HH:MM) | Sí | No | No |

#### Entidad: GESTION (Ticket)

| Nombre | Descripción | Propósito | Reglas de Negocio Relevantes |
| :--- | :--- | :--- | :--- |
| **GESTION** | Unidad atómica de trabajo o tarea programada (ticket). | **CU 6, RF 9:** Registrar la asignación de la tarea a un recurso y el evento histórico de la gestión (Data Entry). | **RF 9:** Un ticket debe ser único e identificable con un recurso y un deudor específicos. |

| Atributo | Descripción | Propósito | Dominio de Valores | Obligatoriedad | Unicidad | Multivaluado |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **IDTicket (PK)** | Identificador único de la tarea de cobranza. | Clave principal para el seguimiento operativo y auditoría. | Texto | Sí | Sí | No |
| **FechaHoraProgramada** | Fecha y hora en la que el recurso debe ejecutar la tarea (programación). | Control del flujo de trabajo de los operadores. | Fecha/Hora | Sí | No | No |
| **EstadoGestion** | Estado actual de la tarea. | Monitoreo operativo (Consulta Crítica). | Enumeración (Pendiente, En Ejecución, Finalizado, Cancelado, Compromiso Pago) | Sí | No | No |
| **Resultado** | Resultado final de la interacción con el deudor (registrado vía Data Entry). | Alimentación del archivo bitácora (Diario de Operaciones). | Texto | No | No | No |
| **CodCliente (FK)** | Referencia al deudor a quien se dirige la gestión. | Enlace a la cuenta (Consulta Crítica). | Texto (CHAR 10) | Sí | No | No |
| **CodRecurso (FK)** | Referencia al recurso asignado para realizar la tarea. | Enlace al operador responsable (Reporte Disputed Items/Collector). | Texto (CHAR 8) | Sí | No | No |

### 4.3. Diccionario de Datos (Fichas de Tipos de Relación)

#### Relación: ASIGNADO\_A

| Nombre | Tipos de Entidad Participantes | Cardinalidades (min..max) | Justificación de Cardinalidades (Reglas de Negocio) |
| :--- | :--- | :--- | :--- |
| **ASIGNADO\_A** | CLIENTE, TIPOLOGIA\_COBRANZA | Cliente (1,1); Tipología (0,N) | **CU 5:** Un cliente debe estar asignado actualmente a una y solo una tipología de cobranza (según sus reglas de monto/mora). Una tipología puede tener cero o muchos clientes asignados en un momento dado. |

#### Relación: ES\_ASIGNADO

| Nombre | Tipos de Entidad Participantes | Cardinalidades (min..max) | Justificación de Cardinalidades (Reglas de Negocio) |
| :--- | :--- | :--- | :--- |
| **ES\_ASIGNADO** | RECURSO, GESTION | Recurso (1,N); Gestión (1,1) | Un ticket (gestión) debe ser asignado a uno y solo un recurso (operario, robot, abogado) para su ejecución. Un recurso puede tener asignadas muchas gestiones. |


# Diseño Lógico y Esquema Relacional del Módulo Programación de Recursos de Cobranza

## 1. Esquema Relacional (Representación Gráfica)

El esquema relacional para la programación de recursos de cobranza se compone de tablas que definen los tipos de servicio, los recursos físicos y humanos disponibles, las fechas de programación (calendario) y la tabla asociativa que relaciona los recursos con las unidades de tiempo reservado (tickets).



[Image of database relational schema diagram]


**Relaciones principales (Tablas):**
1.  `TIPO_COBRANZA`
2.  `RECURSO`
3.  `CALENDARIO` (Tabla de referencia para fechas)
4.  `TICKET` (Unidad de tiempo programada)
5.  `ASIGNACION_RECURSO_TICKET` (Tabla asociativa)

**Relaciones entre tablas:**
* `TICKET` se relaciona con `TIPO_COBRANZA` (1:N): Un tipo de cobranza puede generar muchos tickets.
* `TICKET` se relaciona con `CALENDARIO` (1:N): Una fecha puede tener muchos tickets programados.
* `TICKET` se relaciona con `RECURSO` (N:M): Un recurso puede ser asignado a muchos tickets, y un ticket (actividad) puede requerir varios recursos (ej. un operador y una herramienta). Esta relación se resuelve mediante la tabla `ASIGNACION_RECURSO_TICKET`.

---

## 2. Diccionario de Datos

Para cada relación (tabla), se detallan el nombre, la descripción, el propósito, las reglas de negocio relevantes, las claves y las restricciones obligatorias.

### T1: TIPO\_COBRANZA

| Aspecto | Detalle | Fuente(s) |
| :--- | :--- | :--- |
| **Nombre de la Relación** | TIPO\_COBRANZA | |
| **Descripción** | Define las características y parámetros de los diferentes servicios de cobranza ofrecidos (e.g., preventiva, prejudicial, judicial). | |
| **Propósito** | Permite al sistema clasificar automáticamente a los deudores y determinar qué tipo de gestión (y por ende, qué recursos) se debe aplicar en función de la mora y el monto. | |
| **Reglas de Negocio Relevantes** | La gestión de cobranza se realiza generalmente cuando la mora es mayor a 0 días. La transición entre tipos de cobranza (ej. preventiva a prejudicial) debe ser automática según el parámetro de días de mora. | |
| **Clave Primaria (PK)** | `ID_TIPO_COBRANZA` | |
| **Claves Foráneas (FK)** | N/A | |
| **Unicidad (UNIQUE)** | `NOMBRE_TIPO` | |

**Tabla de Atributos:**
| Nombre de la Columna | Descripción | Propósito | Tipo de Dato | Obligatoriedad | Restricciones |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `ID_TIPO_COBRANZA` | Identificador único del tipo de servicio (ej. 'P01', 'J01'). | PK. | `CHAR(4)` | `NOT NULL` | PK |
| `NOMBRE_TIPO` | Nombre descriptivo del tipo de cobranza (ej. Preventiva, Judicial). | Identificación. | `VARCHAR(50)` | `NOT NULL` | `UNIQUE` |
| `MORA_MIN_DIAS` | Mínimo de días de mora para aplicar este tipo de cobranza. | Regla de clasificación. | `NUMERIC(3)` | `NOT NULL` | `CHECK (> 0)` |
| `MORA_MAX_DIAS` | Máximo de días de mora para aplicar este tipo de cobranza (Duración de la campaña). | Regla de clasificación y duración máxima. | `NUMERIC(3)` | `NOT NULL` | |
| `MONTO_MIN` | Monto mínimo de la deuda para destinar recursos a esta gestión. | Regla de optimización de recursos. | `DECIMAL(14, 2)` | `NOT NULL` | |
| `MONTO_MAX` | Monto máximo de la deuda. | Regla de clasificación. | `DECIMAL(14, 2)` | `NULL` | |
| `REQUIERE_GARANTIA` | Indicador si este tipo de cobranza requiere la existencia de un aval o garantía (ej. Judicial). | Regla de clasificación. | `CHAR(1)` | `NOT NULL` | `CHECK ('S', 'N')` |
| `PROTOCOLO_ID` | Código del protocolo de acciones asociado (ej. secuencia de llamadas, emails, cartas). | FK (implícita a tabla de protocolos). | `CHAR(10)` | `NOT NULL` | |

### T2: RECURSO

| Aspecto | Detalle | Fuente(s) |
| :--- | :--- | :--- |
| **Nombre de la Relación** | RECURSO | |
| **Descripción** | Lista de recursos reales (operarios, abogados, autómatas, herramientas tecnológicas) disponibles para la gestión de cobranza. | |
| **Propósito** | Asegurar que la programación de tickets se base en recursos existentes y determinar su capacidad y disponibilidad. | |
| **Reglas de Negocio Relevantes** | Los recursos son programados en base a su capacidad para maximizar la eficiencia y evitar que un operador tenga una carga de trabajo desbalanceada. | |
| **Clave Primaria (PK)** | `ID_RECURSO` | |
| **Claves Foráneas (FK)** | N/A | |
| **Unicidad (UNIQUE)** | `CODIGO_RECURSO` | |

**Tabla de Atributos:**
| Nombre de la Columna | Descripción | Propósito | Tipo de Dato | Obligatoriedad | Restricciones |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `ID_RECURSO` | Identificador único del recurso (ej. ID de empleado, ID de robot, ID de laptop). | PK. | `NUMERIC(10)` | `NOT NULL` | PK |
| `CODIGO_RECURSO` | Código alfanumérico único. | Identificación rápida. | `VARCHAR(20)` | `NOT NULL` | `UNIQUE` |
| `TIPO_RECURSO` | Clasificación del recurso (ej. Humano, Tecnológico, Físico, Financiero). | Permite filtrar al programar. | `CHAR(2)` | `NOT NULL` | `CHECK (CÓDIGO)` |
| `DESCRIPCION` | Descripción detallada del recurso (ej. Operador Call Center N° 3, Robot de Envío de Emails). | Detalle del bien/persona. | `VARCHAR(100)` | `NOT NULL` | |
| `CAPACIDAD_DIARIA` | Cantidad máxima de tareas que puede manejar en un día (ej. 50 llamadas, 10 visitas). | Control de carga. | `NUMERIC(4)` | `NOT NULL` | |
| `ESTADO` | Estado actual del recurso (ej. Disponible, Ocupado, Mantenimiento). | Control de disponibilidad. | `CHAR(1)` | `NOT NULL` | `CHECK (CÓDIGO)` |

### T3: CALENDARIO (Lookup Table / Tabla de Referencia)

| Aspecto | Detalle | Fuente(s) |
| :--- | :--- | :--- |
| **Nombre de la Relación** | CALENDARIO | |
| **Descripción** | Define las fechas y sus características (días laborables, feriados, etc.), esencial para la programación de tickets. | |
| **Propósito** | Proporcionar una base temporal para la generación de tickets y gestionar la disponibilidad en base a días especiales (ej. feriados que pueden afectar tarifas o disponibilidad). | |
| **Reglas de Negocio Relevantes** | Los tickets se programan por un periodo definido (e.g., semanal o mensual). | |
| **Clave Primaria (PK)** | `FECHA` | |
| **Claves Foráneas (FK)** | N/A | |
| **Unicidad (UNIQUE)** | N/A | |
| **Lookup Data** | Contiene todas las fechas relevantes. | |

**Tabla de Atributos:**
| Nombre de la Columna | Descripción | Propósito | Tipo de Dato | Obligatoriedad | Restricciones |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `FECHA` | Día específico. | PK. | `DATE` | `NOT NULL` | PK |
| `DIA_SEMANA` | Día de la semana (L, M, X, J, V, S, D). | Referencia de horarios. | `CHAR(1) | `NOT NULL` | |
| `ES_FERIADO` | Indica si la fecha es un día festivo. | Afecta la tarifa y disponibilidad. | `CHAR(1)` | `NOT NULL` | `CHECK ('S', 'N')` |
| `TURNO_OPERATIVO` | Indica el turno disponible (Mañana, Tarde, Noche). | Detalle de franjas horarias. | `CHAR(1)` | `NOT NULL` | |

### T4: TICKET

| Aspecto | Detalle | Fuente(s) |
| :--- | :--- | :--- |
| **Nombre de la Relación** | TICKET | |
| **Descripción** | Unidad de tiempo reservada para realizar una actividad de cobranza (cupo). | |
| **Propósito** | Gestionar la disponibilidad del servicio. Permite al sistema determinar si puede aceptar nuevos requerimientos y programar recursos. | |
| **Reglas de Negocio Relevantes** | Un cliente puede adquirir el servicio de cobranza si existe un ticket disponible en la fecha requerida. El ticket se genera en procesos Batch basados en la disponibilidad y capacidad. | |
| **Clave Primaria (PK)** | `ID_TICKET` | |
| **Claves Foráneas (FK)** | `ID_TIPO_COBRANZA`, `FECHA` | |
| **Unicidad (UNIQUE)** | N/A | |

**Tabla de Atributos:**
| Nombre de la Columna | Descripción | Propósito | Tipo de Dato | Obligatoriedad | Restricciones |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `ID_TICKET` | Identificador único del ticket programado. | PK. | `NUMERIC(15)` | `NOT NULL` | PK |
| `ID_TIPO_COBRANZA` | Identificador del tipo de cobranza al que aplica este ticket (FK a T1). | Define el protocolo a seguir. | `CHAR(4)` | `NOT NULL` | FK a `TIPO_COBRANZA` |
| `FECHA` | Fecha de la gestión programada (FK a T3). | Define el día. | `DATE` | `NOT NULL` | FK a `CALENDARIO` |
| `HORA_INICIO` | Hora de inicio de la ventana de tiempo del ticket. | Control de turno. | `TIME` | `NOT NULL` | |
| `HORA_FIN` | Hora de finalización de la ventana de tiempo del ticket. | Control de turno. | `TIME` | `NOT NULL` | |
| `ESTADO_TICKET` | Indica si el ticket está 'Disponible', 'Reservado' (Vendido) o 'Usado'. | Control de inventario de cupos. | `CHAR(1)` | `NOT NULL` | `CHECK (CÓDIGO)` |

### T5: ASIGNACION\_RECURSO\_TICKET (Tabla Asociativa)

| Aspecto | Detalle | Fuente(s) |
| :--- | :--- | :--- |
| **Nombre de la Relación** | ASIGNACION\_RECURSO\_TICKET | |
| **Descripción** | Resuelve la relación N:M entre TICKET y RECURSO, detallando qué recurso específico está asignado a qué tarea dentro de un ticket. | |
| **Propósito** | Relaciona la unidad de trabajo (ticket) con el recurso real que lo ejecutará, permitiendo asignar cargas de trabajo a operadores individuales. | |
| **Reglas de Negocio Relevantes** | Asegura que se respete la capacidad del recurso (`CAPACIDAD_DIARIA` en T2) y que los tickets solo puedan ser asignados si el recurso está disponible. | |
| **Clave Primaria (PK)** | `ID_TICKET`, `ID_RECURSO` | |
| **Claves Foráneas (FK)** | `ID_TICKET`, `ID_RECURSO` | |

**Tabla de Atributos:**
| Nombre de la Columna | Descripción | Propósito | Tipo de Dato | Obligatoriedad | Restricciones |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `ID_TICKET` | Clave primaria compuesta (FK a TICKET). | Componente de la PK. | `NUMERIC(15)` | `NOT NULL` | PK, FK a `TICKET` |
| `ID_RECURSO` | Clave primaria compuesta (FK a RECURSO). | Componente de la PK. | `NUMERIC(10)` | `NOT NULL` | PK, FK a `RECURSO` |
| `TAREA_ASIGNADA` | Detalle de la acción específica que debe realizar el recurso (ej. Primer contacto telefónico, Envío de carta automatizada). | Detalle de la ejecución del protocolo. | `VARCHAR(100)` | `NOT NULL` | |
| `ESTADO_TAREA` | Estado de la tarea asignada (ej. Pendiente, En proceso, Finalizada, Cancelada). | Seguimiento de la bitácora. | `CHAR(1A)` | `NOT NULL` | |



## 6. Diseño Físico del Módulo Programación de Recursos de Cobranza

### 6.1. Selección y Justificación de Tablas

Según las instrucciones, debemos escoger al menos dos tablas que no sean catálogos o `lookup tables` y que tengan un `movimiento significativo` a lo largo de la vida del sistema, dando preferencia a aquellas que representan transacciones.

Hemos seleccionado las dos principales tablas transaccionales generadas en este módulo:

| Tabla | Nombre Lógico | Justificación (Volumen, Criticidad y Relevancia) |
| :--- | :--- | :--- |
| **T4** | **TICKET** | Esta tabla representa la `unidad de cupo` de gestión programada. Es `crítica` porque gestiona la disponibilidad del servicio y el inventario de cupos. Los tickets se generan en procesos `Batch`, lo que implica un `alto volumen de datos` inicial y periódico (masivo). |
| **T5** | **ASIGNACION\_RECURSO\_TICKET** | Es la tabla asociativa que registra la asignación de un recurso a un ticket, funcionando como una `bitácora de hechos` (`Data entry` de eventos en tiempo real). Su volumen de registros será el más alto del módulo, ya que documenta cada tarea específica y su estado, volviéndola crítica para cualquier `consulta de explotación de datos` o reportes históricos. |

### 6.2. Estimación y Proyección de Registros (3 Años)

La proyección se basa en supuestos explícitos sobre la operación del negocio, un requisito fundamental para fundamentar la estimación.

**Supuestos Operativos:**
1.  **Días operativos:** El sistema opera 260 días al año (5 días/semana).
2.  **Capacidad de Programación (Ticket):** Se estima una capacidad inicial de `8,000 tickets diarios` (basada en 100 recursos activos x 8 horas/día x 10 tickets/hora).
3.  **Ratio de Asignación (Recursos):** Se asume que cada ticket requiere un promedio de `1.5 asignaciones` de recursos (T5).
4.  **Crecimiento:** Se proyecta un crecimiento anual del `15%` en el volumen de tickets debido a la expansión de servicios de cobranza (ej. nuevos tipos de cobranza definidos en T1).

**T4: TICKET (Proyección de 3 Años)**

* **Cálculo Base (Día):** `8,000` registros/día.

| Periodo | Registros Iniciales (RL) | Base de Cálculo (RL) | Proyección Anual (RL) |
| :--- | :--- | :--- | :--- |
| **Carga Inicial (Año 0)** | 1,040,000 | `8,000 * 130 días` (6 meses) | **1,040,000** |
| **Año 1** | `1,040,000 × 1.15` | `8,000 × 260 días` | **2,080,000** |
| **Año 2** | `2,080,000 × 1.15` | `2,080,000 × 1.15` | **2,392,000** |
| **Año 3** | `2,392,000 × 1.15` | `2,392,000 × 1.15` | **2,750,800** |
| **Total Proyectado (Años 1-3)** | | | **7,222,800** |

**T5: ASIGNACION\_RECURSO\_TICKET (Proyección de 3 Años)**

* **Cálculo Base (Día):** `8,000 × 1.5 = 12,000` registros/día.

| Periodo | Registros Iniciales (RL) | Base de Cálculo (RL) | Proyección Anual (RL) |
| :--- | :--- | :--- | :--- |
| **Carga Inicial (Año 0)** | 1,560,000 | `12,000 * 130 días` (6 meses) | **1,560,000** |
| **Año 1** | `1,560,000 × 1.15` | `12,000 × 260 días` | **3,120,000** |
| **Año 2** | `3,120,000 × 1.15` | `3,120,000 × 1.15` | **3,588,000** |
| **Año 3** | `3,588,000 × 1.15` | `3,588,000 × 1.15` | **4,126,200** |
| **Total Proyectado (Años 1-3)** | | | **10,834,200** |

### 6.3. Parámetros Físicos y Dimensionamiento

El diseño físico debe definir cómo se almacenarán, recuperarán y transmitirán los datos. Utilizaremos el modelo de organización `indexada`, ya que permite el acceso aleatorio y es el enfoque típico para tablas transaccionales maestras.

| Parámetro Físico | Valor Seleccionado | Justificación según la Fuente |
| :--- | :--- | :--- |
| **Organización** | Indexada (Árbol B / Indexación Denso) | Permite acceso aleatorio (por clave). Necesario para consultas en línea que acceden por clave. |
| **Tamaño de Bloque (IC)** | 8 KB (8192 Bytes) | El Intervalo de Control (IC) es la unidad de transferencia entre disco y memoria. Se selecciona un tamaño grande (8KB) para procesos `Batch` (como la generación de tickets). |
| **Tamaño de Extent (AC)** | 1 Cilindro | Para archivos voluminosos como las tablas transaccionales, se recomienda que el Área de Control (AC) se fije al tamaño de un cilindro para evitar la fragmentación. |
| **Espacio Libre (%IC)** | 20% | Este porcentaje se reserva para inserciones o crecimiento de registros, evitando rupturas y fragmentación de la base de datos. |
| **Tipo de Información** | Movimiento (Evento) | Los tickets y las asignaciones son eventos generados en tiempo real. |

#### 6.3.1. T4: TICKET - Estructura y Cálculo de Tamaño

**1. Estructura del Registro Lógico (RL) (Moda):**

| Atributo | Tipo Lógico | Tamaño Estimado (Bytes) |
| :--- | :--- | :--- |
| `ID_TICKET` (PK) | `NUMERIC(15)` | 8 (Compresión para claves) |
| `ID_TIPO_COBRANZA` (FK) | `CHAR(4)` | 4 |
| `FECHA` (FK) | `DATE` | 7 |
| `HORA_INICIO`, `HORA_FIN` | `TIME` | 6 (3 bytes c/u, simplificado) |
| `ESTADO_TICKET` | `CHAR(1)` | 1 |
| **Longitud del Registro Lógico (RL)** | | **26 Bytes** |

**2. Cálculo del Espacio Total Estimado (Proyección Total):**
* **Capacidad por IC (8192 Bytes):** El IC almacena RLs, Campos Descriptores de Registro (`CDR` ≈ 3 bytes) y Campo de IC (`CIC` ≈ 4 bytes).
* **Espacio útil por IC (80%):** `8192 × 0.80 = 6553.6` bytes.
* **Registros por IC:** `6553.6 / (26 RL + 3 CDR) ≈ 226` RL/IC.
* **Total de RL (Años 1-3):** `7,222,800` registros.
* **Total de ICs requeridos:** `7,222,800 / 226 ≈ 31,950` ICs.

**Taman ˜ o Total Estimado (Data) = 31,950 ICs × 8192 Bytes/IC ≈ 261.7 MB**

Nota: Este cálculo no incluye el espacio requerido para el Index, que sería un componente separado, compacto y cargado en memoria para optimizar la búsqueda.

#### 6.3.2. T5: ASIGNACION\_RECURSO\_TICKET - Estructura y Cálculo de Tamaño

**1. Estructura del Registro Lógico (RL) (Moda):**

| Atributo | Tipo Lógico | Tamaño Estimado (Bytes) |
| :--- | :--- | :--- |
| `ID_TICKET` (PK/FK) | `NUMERIC(15)` | 8 |
| `ID_RECURSO` (PK/FK) | `NUMERIC(10)` | 6 |
| `TAREA_ASIGNADA` | `VARCHAR(100)` | 50 (Moda/Longitud más típica) |
| `ESTADO_TAREA` | `CHAR(1)` | 1 |
| **Longitud del Registro Lógico (RL)** | | **65 Bytes** |

**2. Cálculo del Espacio Total Estimado (Proyección Total):**
* **Capacidad por IC (8192 Bytes):**
* **Espacio útil por IC (80%):** `6553.6` bytes.
* **Registros por IC:** `6553.6 / (65 RL + 3 CDR) ≈ 96` RL/IC.
* **Total de RL (Años 1-3):** `10,834,200` registros.
* **Total de ICs requeridos:** `10,834,200 / 96 ≈ 112,856` ICs.

**Taman ˜ o Total Estimado (Data) = 112,856 ICs × 8192 Bytes/IC ≈ 924.5 MB**

**Perspectiva del Diseño Físico**

El diseño físico enfatiza que el `tiempo de respuesta` depende críticamente del tiempo de recuperación de datos. Dado el `alto volumen de datos` y la naturaleza transaccional de ambas tablas, es imperativo que la `fragmentación` física sea mínima.

La decisión de reservar el `20%` de Espacio Libre a nivel de IC (Intervalo de Control) y fijar el tamaño de Extent (AC) al tamaño de un Cilindro es una `medida de optimización` que permite el crecimiento ordenado y evita los reacomodos de páginas y cilindros (fragmentación). La fragmentación puede degradar seriamente la `performance`, volviendo al sistema lento, incluso si la consulta es sencilla.

La planificación de la capacidad física asegura que el sistema esté preparado para el crecimiento agresivo sin colapsar. Esto es un ejemplo de cómo el `Diseño Interno` (físico y lógico) debe orientarse a la eficiencia, utilizando el mínimo de recursos con el mejor rendimiento.


---

## 7. Creación de Tablas y Poblamiento de Datos

### 7.1. Creación de Tablas (DDL - Data Definition Language)

* **Forma de Generación:** La generación de las sentencias Data Definition Language (DDL) se realiza de forma Manual, aplicando las reglas de conversión del Modelo Relacional desarrollado en la fase anterior.
* **Lineamientos sobre los Scripts:** Para cumplir con los lineamientos de diseño físico, y asegurar que los scripts puedan reprocesarse, se incluyen sentencias de eliminación (DROP) al inicio.
* Se crea un esquema específico (`prog_cobranza`) para el módulo, y todas las tablas se alojan dentro de este esquema, evitando el esquema `public`.

**Sentencias DDL Consolidadas (Ejemplo General de Estructura):**

```sql
-- 1) Eliminar el esquema si existe (Permite volver a ejecutar) [3]
DROP SCHEMA IF EXISTS prog_cobranza CASCADE; 

-- 2) Crear esquema [2, 3]
CREATE SCHEMA prog_cobranza;

-- Elimina la tabla si existe [3]
DROP TABLE IF EXISTS prog_cobranza.TIPO_COBRANZA CASCADE;

-- 4) Crear tabla [4]
CREATE TABLE prog_cobranza.TIPO_COBRANZA (
    ID_TIPO_COBRANZA CHAR(4) PRIMARY KEY, -- PK
    NOMBRE_TIPO VARCHAR(50) NOT NULL UNIQUE, -- Restricción de unicidad
    MORA_MIN_DIAS NUMERIC(3) NOT NULL,
    MORA_MAX_DIAS NUMERIC(3) NOT NULL,
    MONTO_MIN DECIMAL(14, 2) NOT NULL,
    MONTO_MAX DECIMAL(14, 2),
    REQUIERE_GARANTIA CHAR(1) NOT NULL CHECK (REQUIERE_GARANTIA IN ('S', 'N')), -- Restricción de dominio
    PROTOCOLO_ID CHAR(10) NOT NULL
);

-- Elimina la tabla si existe [3]
DROP TABLE IF EXISTS prog_cobranza.TICKET;

CREATE TABLE prog_cobranza.TICKET (
    ID_TICKET BIGSERIAL PRIMARY KEY, -- PK, usa BIGSERIAL para autoincremento en PostgreSQL
    ID_TIPO_COBRANZA CHAR(4) NOT NULL,
    FECHA DATE NOT NULL,
    HORA_INICIO TIME NOT NULL,
    HORA_FIN TIME NOT NULL,
    ESTADO_TICKET CHAR(1) NOT NULL CHECK (ESTADO_TICKET IN ('D', 'R', 'U')), -- D: Disponible, R: Reservado, U: Usado
    
    -- Claves Foráneas [4]
    CONSTRAINT fk_tipo_cobranza FOREIGN KEY (ID_TIPO_COBRANZA)
        REFERENCES prog_cobranza.TIPO_COBRANZA (ID_TIPO_COBRANZA),
    
    CONSTRAINT fk_fecha_calendario FOREIGN KEY (FECHA)
        REFERENCES prog_cobranza.CALENDARIO (FECHA)
);
```

---
### 7.2. Poblamiento Inicial de Datos (DML - Data Manipulation Language)

* **Datos Utilizados:** Se utilizarán datos sintéticos que reflejen la realidad del negocio, asegurando que los catálogos y las referencias (como tipos de cobranza y recursos) contengan información con coherencia.
* **Orden de Ejecución:** Es crucial asegurar el orden de ejecución de las sentencias `INSERT` para respetar la integridad referencial. Las tablas maestras y de referencia (TIPO\_COBRANZA, RECURSO, CALENDARIO) deben poblarse antes que las tablas transaccionales que las referencian (TICKET, ASIGNACION\_RECURSO\_TICKET).

A continuación, se presentan ejemplos de sentencias `INSERT` para TIPO\_COBRANZA y RECURSO.

#### Ejemplo 1: Poblamiento de TIPO\_COBRANZA

Insertamos dos tipos de cobranza que definen los escenarios del negocio.

```sql
-- Cobranza Preventiva (Menor mora, menor monto)
INSERT INTO prog_cobranza.TIPO_COBRANZA (
    ID_TIPO_COBRANZA, NOMBRE_TIPO, MORA_MIN_DIAS, MORA_MAX_DIAS, 
    MONTO_MIN, MONTO_MAX, REQUIERE_GARANTIA, PROTOCOLO_ID
) VALUES (
    'P001', 'Preventiva Telefónica', 1, 30, 50.00, 1000.00, 'N', 'PROT_P01'
);

-- Cobranza Judicial (Mayor mora, mayor monto, requiere garantía)
INSERT INTO prog_cobranza.TIPO_COBRANZA (
    ID_TIPO_COBRANZA, NOMBRE_TIPO, MORA_MIN_DIAS, MORA_MAX_DIAS, 
    MONTO_MIN, MONTO_MAX, REQUIERE_GARANTIA, PROTOCOLO_ID
) VALUES (
    'J001', 'Judicial Litigio', 91, 365, 5000.00, 9999999.99, 'S', 'PROT_J01'
);


-- Recurso Humano (Operador de Call Center)
INSERT INTO prog_cobranza.RECURSO (
    ID_RECURSO, CODIGO_RECURSO, TIPO_RECURSO, DESCRIPCION, 
    CAPACIDAD_DIARIA, ESTADO
) VALUES (
    1001, 'OPCC_45', 'H', 'Operador Senior Call Center Turno Tarde', 
    60, 'D' -- D: Disponible
);

-- Recurso Tecnológico (Robot para envío masivo de correos)
INSERT INTO prog_cobranza.RECURSO (
    ID_RECURSO, CODIGO_RECURSO, TIPO_RECURSO, DESCRIPCION, 
    CAPACIDAD_DIARIA, ESTADO
) VALUES (
    5001, 'ROB_EMAIL_1', 'T', 'Robot Automatizado Envío Emails', 
    5000, 'M' -- M: Mantenimiento (No disponible para programación)
);
```


### 8.1. Búsqueda de Texto (Índices Invertidos)

#### Escenario Relevante

Se requiere una funcionalidad de búsqueda rápida que permita a los supervisores encontrar recursos (`RECURSO`) o tareas asignadas (`ASIGNACION_RECURSO_TICKET.TAREA_ASIGNADA`) mediante palabras clave específicas (ej., "abogado", "senior", "protocolo judicial") contenidas en campos de texto libre.

#### Fundamentación

Si esta consulta se realiza de manera rutinaria utilizando comandos estándar como `LIKE '%palabra%'` sobre un campo extenso (`VARCHAR`), el sistema se vería obligado a barrer (scan) toda la tabla. Dado que esta consulta podría ser muy solicitada (alta demanda) y su costo es alto (barrido completo), se clasifica como una consulta crítica. Este tipo de búsqueda consume mucho tiempo, lo que degrada la performance del sistema.

La solución técnica para búsquedas de texto es el uso de Listas Invertidas (Índices Invertidos).  Esta técnica permite la búsqueda por palabras (tipo Google) y resuelve la consulta previamente, minimizando el tiempo de respuesta.

#### Diseño de la Solución e Integración

La solución consiste en crear un índice invertido (Full Text Search Index en PostgreSQL) sobre la columna `DESCRIPCION` en la tabla `prog_cobranza.RECURSO`. Este índice transformará el texto en tokens (palabras clave) que pueden ser consultados de forma rápida.

**Sentencias SQL (Ejemplo Conceptual):**

```sql
-- DDL: Creación de la columna y el índice invertido (asumiendo PostgreSQL)
ALTER TABLE prog_cobranza.RECURSO
    ADD COLUMN TS_DESCRIPCION TSVECTOR;

-- Actualización del índice (generalmente en Batch)
UPDATE prog_cobranza.RECURSO
    SET TS_DESCRIPCION = TO_TSVECTOR('spanish', DESCRIPCION);

-- Creación del Índice GIN (implementación de Listas Invertidas)
CREATE INDEX idx_recurso_descripcion_ts
    ON prog_cobranza.RECURSO USING GIN (TS_DESCRIPCION);

-- DML: Ejemplo de uso del Índice Invertido (Búsqueda por palabras)
SELECT ID_RECURSO, DESCRIPCION
FROM prog_cobranza.RECURSO
WHERE TS_DESCRIPCION @@ TO_TSQUERY('spanish', 'Operador & Senior');
```

### 8.2. Manejo de Grandes Volúmenes de Datos (Particionamiento)

#### Escenario Relevante

La tabla `ASIGNACION_RECURSO_TICKET` (T5), que registra cada tarea específica asignada a un recurso, se proyecta con un volumen superior a 10 millones de registros en 3 años. La mayoría de las consultas operativas y gerenciales (ej. performance diaria o auditoría del último trimestre) solo acceden a los datos más recientes.

#### Fundamentación

El alto volumen de datos en T5 afecta directamente el rendimiento de las consultas. Para evitar que las consultas lentas "atrasen" el sistema, se debe aplicar una estrategia de Particionamiento. El particionamiento divide la data en partes más pequeñas (segmentos físicos) por rangos de clave. 

#### Diseño de la Estrategia de Particionamiento

Se seleccionará la columna `FECHA` (que se deriva del `ID_TICKET` asociado) como clave de particionamiento, ya que las consultas se dan generalmente por periodo de tiempo (recurrencia).

* **Tabla seleccionada:** `prog_cobranza.ASIGNACION_RECURSO_TICKET` (T5).
* **Campo de particionamiento:** `FECHA`.
* **Criterio:** Particionamiento por Rango (mensual o trimestral), ya que el tiempo es continuo y predecible. Particionar por meses es adecuado para reportes de gestión que suelen ser mensuales.

**Sentencias SQL (Ejemplo Conceptual de Particionamiento Mensual):**

```sql
-- DDL: Creación de la tabla maestra particionada (ejemplo de PostgreSQL 11+)
CREATE TABLE prog_cobranza.ASIGNACION_RECURSO_TICKET (
    ID_TICKET NUMERIC(15) NOT NULL,
    ID_RECURSO NUMERIC(10) NOT NULL,
    FECHA DATE NOT NULL,
    -- ... otros atributos
    PRIMARY KEY (ID_TICKET, ID_RECURSO, FECHA)
) PARTITION BY RANGE (FECHA);

-- DDL: Creación de particiones hijas (ejemplo para un mes)
CREATE TABLE prog_cobranza.ASIGNACION_RECURSO_TICKET_202401
    PARTITION OF prog_cobranza.ASIGNACION_RECURSO_TICKET
    FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

-- DML: Ejemplo de consulta que accede solo a una partición
SELECT *
FROM prog_cobranza.ASIGNACION_RECURSO_TICKET
WHERE FECHA >= '2024-03-01' AND FECHA < '2024-04-01'; 
-- El gestor solo revisa la partición de marzo, ignorando los millones de registros anteriores.
```

### 8.3. Actualización Periódica de Datos (Procesos Batch)

#### Escenario Relevante

La generación de nuevos tickets (cupos) disponibles para la venta o asignación requiere evaluar masivamente la capacidad diaria de todos los recursos (T2) y la disponibilidad del calendario (T3). Asimismo, la clasificación de deudores o la liquidación de las tareas completadas deben realizarse al final del día.

#### Fundamentación

La Actualización de Base de Datos a grandes volúmenes (como la generación de millones de tickets disponibles) o el Recálculo de Indicadores (estadísticas, saldos) son procesos Batch.  Ejecutar estas operaciones en línea (*On-Line*) paralizaría el sistema por el alto consumo de recursos, por lo que deben ser ejecutadas de manera nocturna o periódica. Estos procesos se resuelven de manera óptima utilizando la técnica de Matching, que requiere que los archivos de entrada estén secuenciales y ordenados por clave.

#### Diseño del Flujo de Ejecución (Generación de Tickets)

El proceso Batch debe ser re-ejecutable y utilizar un esquema de cuadre de totales (Checkpoint) para asegurar la integridad de la información generada.

**Flujo de Ejecución (Pasos):**

| Paso | Módulo (Función) | Entrada | Proceso | Salida |
| :--- | :--- | :--- | :--- | :--- |
| P1 | ORDENA-DATOS | RECURSO (Tabla), CALENDARIO (Tabla) | Descarga y Ordenamiento secuencial de Recursos y Días operativos. | FICHERO\_RECURSO\_SEQ, FICHERO\_CALENDARIO\_SEQ (Archivos Planos). |
| P2 | PROGRAMA (Match) | FICHERO\_RECURSO\_SEQ, FICHERO\_CALENDARIO\_SEQ | Cruza (Match) la capacidad de los recursos con los días disponibles para generar los cupos (TICKET temporal). | FICHERO\_TICKET\_NEW (Archivo Plano de Movimiento). |
| P3 | CARGA-BD | FICHERO\_TICKET\_NEW | Carga Masiva (LOAD o COPY) de los nuevos tickets a la tabla TICKET (T4). | prog\_cobranza.TICKET (Tabla Indexada). |
| P4 | CUADRE-CONTROL | FICHERO\_TICKET\_NEW, prog\_cobranza.TICKET | Verifica que el total de tickets generados en P2 sea igual al total de registros insertados en P3. | LOG |

**Sentencias SQL (Ejemplo de Cuadre de Control)**

La verificación de totales es una práctica indispensable para garantizar la confiabilidad del sistema.

```sql
-- Verificar que los totales del proceso (FICHERO_TICKET_NEW) 
-- coincidan con la carga final en la tabla TICKET (T4).

-- 1. Obtener el total de registros cargados en el proceso Batch (asumido por variable)
SELECT COUNT(*) INTO total_cargado FROM prog_cobranza.TICKET WHERE FECHA = CURRENT_DATE;

-- 2. Obtener el total de tickets que debieron generarse según la capacidad diaria total (asumido por variable)
SELECT SUM(CAPACIDAD_DIARIA) INTO capacidad_total_esperada FROM prog_cobranza.RECURSO WHERE ESTADO = 'D';

-- 3. Comparación y alerta si hay descuadre
IF total_cargado != capacidad_total_esperada THEN
    RAISE EXCEPTION 'ERROR DE CUADRE: La cantidad de tickets cargados (%) no coincide con la capacidad total esperada (%).', 
        total_cargado, capacidad_total_esperada;
END IF;
```

