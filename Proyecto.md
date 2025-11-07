# 3.2. Módulo 2 - Programación de Recursos Internos

## Descripción del Módulo
El módulo de programación se centra en gestionar la oferta de servicios de gestión de deuda, procesar carteras masivas de deudores (Batch) y asignar tareas específicas (tickets) a los recursos humanos y tecnológicos.

## Requerimientos funcionales
Para que este módulo funcione correctamente se debe de cumplir los siguientes requerimientos funcionales.

| Código                 |Requerimiento                              |
|------------------------|-------------------------------------------|
|RF201                   |Validación de la cartera de cliente        |
|RF202                   |Clasificación de los deudores              |
|RF203                   |Asignación de recursos                     |  
|RF204                   |Cambio de estado por vencimiento           | 
|RF205                   |Consultar la programación                  |
|RF206                   |Registros de eventos de cobranza           |
|RF207                   |Consultar el historial de los clientes     |

---

## **Caso de uso #1: Validación de la cartera de cliente**
| Item | Descripción |
| :--- | :--- |
| **Actor(es) involucrado(s)** | Sistema |
| **Objetivo** | Procesar un archivo masivo de deudores (cartera) y validar la integridad y existencia de datos clave. |
| **Precondiciones**|1. La cartera de deudores debe estar disponible en un archivo plano de entrada con el formato previamente definido y esperado.<br> 2. El Cliente (sponsor) debe haber adquirido el servicio de cobranza, autorizando el procesamiento de los datos de su cartera.                                             |
| **Disparador o evento inicial** | El proceso Batch inicia su ejecución programada. |
| **Flujo principal de eventos** | 1. El proceso Batch carga los datos del archivo plano de entrada.<br>2. El sistema aplica procesos de validación de datos clave.<br>3. Los registros que pasan la validación se preparan para el siguiente paso (Casos de uso 2).<br>4. Se emiten estadísticas de control. |
| **Flujos alternativos** | **A1: Rechazo por Dato Inválido:** Los registros con formatos inválidos o errores de integridad son excluidos y registrados en un archivo de rechazo (Bad File o Discard File). |
| **Postcondiciones** | La cartera se encuentra validada y limpia. Se ha generado un lote de deudores listo para ser clasificado. |
| **Excepciones** | **E1: Interrupción de Proceso:** El proceso Batch debe ser re-ejecutable desde un Punto de Control (Checkpoint) para evitar reprocesar todo desde el inicio. |


## **Caso de uso #2: Clasificación de los deudores**
| Item | Descripción |
| :--- | :--- |
| **Actor(es) involucrado(s)** | Sistema  |
| **Precondiciones**             |1. La cartera debe haber sido validada y estar lista para el procesamiento .<br> 2. El Catálogo de Tipologías y Parámetros debe estar completamente configurado para que el sistema pueda asignar un producto|
| **Objetivo** | Clasificar automáticamente a cada deudor de la cartera y asignarle el producto de cobranza (Preventivo, Judicial, etc.). |
| **Disparador o evento inicial** | La finalización exitosa del proceso de Validación Masiva de Cartera. |
| **Flujo principal de eventos** | 1. El sistema lee secuencialmente la cartera validada.<br>2. Consulta los parámetros de tipología.<br>3. Compara la mora y el monto del deudor contra los rangos paramétricos.<br>4. Asigna el producto de cobranza que corresponde.<br>5. Los registros se actualizan con el producto asignado. |
| **Flujos alternativos** | N/A |
| **Postcondiciones** | La cartera de deudores está clasificada, y cada deudor tiene asignado un producto/protocolo de cobranza. |
| **Excepciones** | **E1: Sin Clasificación Correspondiente:** Si un deudor no cumple con ningún rango definido en los parámetros, el sistema debe asignarle una tipología por defecto o registrarlo como excepción. |


## **Caso de uso #3: Asignación de recursos**
| **ID**                         | RF203                                                                      |
|--------------------------------|----------------------------------------------------------------------------|
| **Actor(es)**                  |Jefe y supervisor de cobranza                                               |
| **Objetivo**                   |Asignar a un asesor de cobranza una cobranza programada.                    |
| **Precondiciones**             |Existe una cobranza programada.                                             |
| **Flujo Principal**            |1. El usuario selecciona una cobranza programada.<br>2. El sistema muestra la información de la cobranza.<br>3. El usuario selecciona el responsable de una lista de asesores de cobranza registrados.<br>4. El sistema guarda la asignación.|
| **Postcondiciones**            |Se genera el ticket de cobranza con un asesor encargado.                    |
| **Excepciones**                |Error en la asignación o fallo de conexión.                                 |

## Prototipo inicial
![](Prototipo/P020200.png)

1.Seleccion de cobranza programado
![](Prototipo/P020201.png)

2.Mustra información
![](Prototipo/P020202.png)

3.Asigna a un responsable
![](Prototipo/P020203.png)

4.Guardar la asignación
![](Prototipo/P020204.png)

## **Caso de uso #4: Cambio de estado por vencimiento**
| Item | Descripción |
| :--- | :--- |
| **Actor(es) involucrado(s)** | Sistema |
| **Objetivo** | Cambiar automáticamente el estado de un deudor a la siguiente etapa de cobranza si ha excedido su plazo máximo definido. |
| **Precondiciones**             |1. El proceso debe estar programado para ejecutarse en la ventana de tiempo Batch.<br> 2. Debe existir una cartera activa de deudores cuya gestión actual está siendo monitoreada en función de la Duración Máxima de la Campaña.|
| **Disparador o evento inicial** | Ejecución periódica para monitorear los plazos de vencimiento. |
| **Flujo principal de eventos** | 1. El sistema monitorea los tickets o gestiones activas y compara la fecha de inicio con la Duración Máxima de la Campaña.<br>2. Si se excede el plazo, el sistema cambia automáticamente el estado del deudor.<br>3. Este cambio desencadena la generación de nuevos tickets para el nuevo protocolo. |
| **Flujos alternativos** | N/A |
| **Postcondiciones** | El deudor ha sido promovido a una nueva tipología de cobranza, y el sistema ha generado las tareas (tickets) correspondientes a su nueva fase de gestión. |
| **Excepciones** | **E1: Plazo no Vencido:** Si el tiempo de gestión aún está dentro del límite (CU 2), el sistema ignora el deudor y continúa el barrido. |


## **Caso de uso #5: Consultar la programación**
| **ID**                         | RF205                                                                                                  |
|--------------------------------|--------------------------------------------------------------------------------------------------------|
| **Actor(es)**                  |                                                                                                        |
| **Objetivo**                   |Revisar las tickets vigentes de la cobranza mostrando resumenes del cliente, deuda y el asesor asignado.|
| **Precondiciones**             |Existe ticket registrados en el sistema.                                                                |
| **Flujo Principal**            |1. El usuario selecciona un ticket vigente.<br>2. El sistema busca y muestra todas los tickets programados. |
| **Postcondiciones**            |El usuario obtiene información actualizada del ticket.                                                  |
| **Excepciones**                |Fallo en la recuperación de la información.                                                             |

## Prototipo
![](Prototipo/P020400.png)


## **Caso de uso #6: Registro de eventos de cobranza**
| Item | Descripción |
| :--- | :--- |
| **Actor(es) involucrado(s)** | Operador. |
| **Objetivo** | Registrar la interacción con el deudor (llamada, email, visita, etc.), incluyendo el resultado, el compromiso de pago y comentarios en pantalla. |
| **Precondiciones**             |1. El sistema debe estar operando en modalidad On-Line para la captura transaccional de datos.<br> 2. El Operador debe haber seleccionado o estar gestionando un Ticket o tarea asignada, lo cual requiere que el ticket ya exista.<br> 3. El sistema debe garantizar que la operación será segura ante cualquier falla para evitar la pérdida o corrupción de datos. |
| **Disparador o evento inicial** | El operador finaliza una gestión asignada por un ticket y necesita registrar la evidencia. |
| **Flujo principal de eventos** | 1. El operador selecciona el ticket gestionado (Casos de uso 5).<br>2. Ingresa los datos de la interacción (ejemplo, "Deudor se compromete a pagar el día X").<br>3. El sistema registra el evento como un Movimiento.<br>4. El sistema actualiza la cuenta maestra del deudor.<br>5. El ticket se marca como cerrado o gestionado. |
| **Flujos alternativos** | **A1: Registro de Compromiso de Pago:** Si el deudor se compromete a pagar, el sistema registra la fecha y monto esperado, lo cual se convierte en un evento futuro de monitoreo. |
| **Postcondiciones** | El evento de cobranza queda registrado, y los saldos/estados de la cuenta del deudor se actualizan en tiempo real (Transaccional). |
| **Excepciones** | **E1: Error de Grabación:** En un proceso transaccional, debe asegurarse la seguridad ante cualquier falla y garantizar la reversa de operaciones (Rollback) si ocurre un error. |




## **Caso de uso #7: Consultar el historial de los clientes**
| Item | Descripción |
| :--- | :--- |
| **Actor(es) involucrado(s)** | Gerentes de cuenta y Operadores. |
| **Objetivo** | Proporcionar una Consulta Crítica del deudor, mostrando datos completos, saldo, mora, historial de gestión y compromisos de pago, accediendo a múltiples tablas de forma eficiente. |
| **Precondiciones**             |1. La Tabla de Consulta Crítica (tabla suplementaria/redundante) debe haber sido previamente generada y poblada por un proceso Batch.<br> 2. La consulta debe ingresar un identificador único del deudor para garantizar un tiempo de respuesta óptimo.<br> 3. El sistema debe asegurar que, a pesar de ser una consulta que requiere acceso a múltiples tablas, mantenga un rendimiento aceptable.|
| **Disparador o evento inicial** | El usuario requiere ver la información consolidada de un deudor. |
| **Flujo principal de eventos** | 1. El usuario ingresa el identificador del deudor.<br>2. El sistema accede a una tabla suplementaria/redundante pre-elaborada para evitar múltiples accesos a tablas operativas.<br>3. La vista consolida los datos del cliente, la relación de cuentas y sus saldos, y el historial de gestión en una única pantalla. |
| **Flujos alternativos** | N/A |
| **Postcondiciones** | La información completa y consolidada del deudor se muestra con un tiempo de respuesta mínimo. |
| **Excepciones** | **E1: Deudor No Encontrado:** El sistema informa que el identificador no existe o no corresponde a un deudor en la cartera activa. |


---

## Requisitos de atributos de calidad

|**Atributos de calidad**      |Descripción                                                                                      |
|------------------------------|-------------------------------------------------------------------------------------------------|
|Seguridad                     |Solamento los usuarios de alta jerarquía son autorizados para registrar o modificar datos.       |
|Disponibilidad                |El módulo debe estar disponible en el horario laboral del 8:00 a 20:00 horas.                    |
|Escalabilidad                 |El módulo debe mostrar 100 clientes deudores con un tiempo de respuesta de a lo mucho 3 segundos.|















<!---

---
## CU 6: Generación Masiva de Tickets de Gestión

| Item | Descripción |
| :--- | :--- |
| **Componente del Caso de Uso** | Definición y Soporte en las Fuentes |
| **Actor(es) involucrado(s)** | Sistema (Batch). |
| **Objetivo** | Generar automáticamente tickets que representan tareas específicas para la gestión de la deuda, programando el trabajo futuro de los recursos. |
| **Disparador o evento inicial** | La finalización exitosa de la Clasificación Automática de Deudores (CU 5). |
| **Flujo principal de eventos** | 1. El sistema lee la cartera clasificada.<br>2. Consulta el Protocolo asociado a la tipología del deudor (CU 2) para determinar las acciones.<br>3. Consulta el Parque de Recursos (CU 3) para verificar la disponibilidad.<br>4. Crea un ticket (tarea programada) asignando la acción al recurso disponible en la fecha programada.<br>5. Actualiza el compromiso del recurso. |
| **Flujos alternativos** | **A1: Capacidad Excedida:** Si el recurso alcanza su capacidad máxima diaria, la tarea se programa para la siguiente fecha disponible. |
| **Postcondiciones** | Se ha creado un lote de Tickets de Gestión asignados a los Recursos Operativos con fechas de ejecución futuras. |
| **Excepciones** | **E1: Interrupción:** Debe usar Punto de Control (Checkpoint) para permitir el re-arranque desde el último commit si el proceso es interrumpido. |




---

# 3. Casos de Uso Operacionales (On-Line)

*Estos requisitos se refieren a la interacción diaria de los operadores con el sistema en tiempo real.*

## CU 8: Consulta de Carga de Trabajo y Disponibilidad

| Item | Descripción |
| :--- | :--- |
| **Componente del Caso de Uso** | Definición y Soporte en las Fuentes |
| **Actor(es) involucrado(s)** | Operadores y Gerencia. |
| **Objetivo** | Permitir a los operadores consultar en tiempo real los tickets que le han sido asignados. Permitir a la gerencia consultar la disponibilidad futura de recursos (cupos). |
| **Disparador o evento inicial** | Un operador inicia sesión o la gerencia accede al módulo de planificación para responder a requerimientos de clientes. |
| **Flujo principal de eventos** | 1. El usuario accede al módulo de consultas (On-Line).<br>2. Si es operador, el sistema muestra la lista de tickets asignados (CU 6).<br>3. Si es Gerente, el sistema presenta un reporte de carga de trabajo y cupos disponibles en el Parque de Recursos (CU 3). |
| **Flujos alternativos** | N/A |
| **Postcondiciones** | El usuario obtiene una vista inmediata (tiempo de respuesta mínimo) de las tareas asignadas o la capacidad planificada. |
| **Excepciones** | **E1: Baja Performance:** Si la consulta tarda más de lo aceptable (ej. > 2 segundos), puede indicar un problema de diseño, ya que es una operación crítica en línea. |
-->

---
# 4.2. Diseño Conceptual

### 4.2. Diccionario de Datos (Fichas de Tipos de Entidad)

#### Entidad: CLIENTE (Deudor)

| Nombre | Descripción | Propósito | Reglas de Negocio Relevantes |
| :--- | :--- | :--- | :--- |
| **CLIENTE** | Representa al deudor cuya cuenta está siendo gestionada por el módulo de cobranza. | Centralizar la información financiera y demográfica relevante para la gestión de la deuda. | Debe contar con un código único (Cuenta o Código de Cliente). |

| Atributo | Descripción | Propósito | Dominio de Valores | Obligatoriedad | Unicidad | Multivaluado |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **CodCliente (PK)** | Código único del cliente en el sistema. | Clave principal y enlace a todas las gestiones y movimientos. | Texto | Sí | Sí | No |
| **NroDocumento** | Número de identificación | Validación de identidad y búsqueda (Consulta Crítica). | Texto | Sí | Sí | No |
| **MontoPendiente** | Saldo total actual que el cliente adeuda. | Determinar la tipología de cobranza aplicable y el riesgo. | Dinero | Sí | No | No |
| **DiasMora** | Cantidad de días de atraso en el pago (mora actual). | Clave para la reclasificación automática y asignación de Tipología. | Número | Sí | No | No |
| **ClasificacionRiesgo** | Nivel de riesgo crediticio o de pérdida asociado al cliente. | Apoyo a la toma de decisiones gerenciales. | Enumeración (Bajo, Medio, Alto) | Sí | No | No |

#### Entidad: TIPOLOGIA\_COBRANZA (Producto)

| Nombre | Descripción | Propósito | Reglas de Negocio Relevantes |
| :--- | :--- | :--- | :--- |
| **TIPOLOGIA\_COBRANZA** | Define la estrategia o producto de gestión de deuda, basándose en rangos de monto y mora. | Define los escenarios posibles y la lógica del negocio para la programación (parámetros). | Un deudor se clasifica automáticamente en una tipología según su mora y monto. |

| Atributo | Descripción | Propósito | Dominio de Valores | Obligatoriedad | Unicidad | Multivaluado |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **CodTipologia (PK)** | Código único que identifica el tipo de servicio. | Clave principal para asociar reglas y recursos a la estrategia. | Texto | Sí | Sí | No |
| **Descripcion** | Nombre descriptivo del producto de cobranza. | Visualización en interfaces gerenciales. | Texto | Sí | No | No |
| **MontoMin** | Monto mínimo de deuda para aplicar esta tipología. | Definición de rango (parámetro). | Dinero | Sí | No | No |
| **MoraMax** | Mora máxima (en días) que puede tener un deudor para permanecer en esta tipología. | Disparador para el cambio de estado. | Número (Entero) | Sí | No | No |

#### Entidad: RECURSO (Parque Operativo)

| Nombre | Descripción | Propósito | Reglas de Negocio Relevantes |
| :--- | :--- | :--- | :--- |
| **RECURSO** | Entidad que representa la capacidad física y humana disponible para ejecutar las tareas. | Gestionar el inventario y la disponibilidad para la generación de tickets. | La generación de tickets depende de la CapacidadDiaria de cada recurso. |

| Atributo | Descripción | Propósito | Dominio de Valores | Obligatoriedad | Unicidad | Multivaluado |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **CodRecuro (PK)** | Identificador único del recurso. | Programación de tareas y monitoreo de productividad. | Texto | Sí | Sí | No |
| **TipoRecurso** | Rol o naturaleza del recurso asignable. | Clasificación para las reglas de asignación. | Enumeración (Operador, Abogado, Robot, Vehículo) | Sí | No | No |
| **CapacidadDiaria** | Número máximo de tickets o tareas que el recurso puede manejar por día. | Parámetro clave para la Generación Batch de Tickets. | Número | Sí | No | No |
| **HorarioDisp** | Período de tiempo en que el recurso está disponible para ser programado. | Control de asignación de tareas. | Texto (HH:MM-HH:MM) | Sí | No | No |

#### Entidad: GESTION (Ticket)

| Nombre | Descripción | Propósito | Reglas de Negocio Relevantes |
| :--- | :--- | :--- | :--- |
| **GESTION** | Unidad atómica de trabajo o tarea programada (ticket). | Registrar la asignación de la tarea a un recurso y el evento histórico de la gestión (Data Entry). | Un ticket debe ser único e identificable con un recurso y un deudor específicos. |

| Atributo | Descripción | Propósito | Dominio de Valores | Obligatoriedad | Unicidad | Multivaluado |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **IDTicket (PK)** | Identificador único de la tarea de cobranza. | Clave principal para el seguimiento operativo y auditoría. | Texto | Sí | Sí | No |
| **FechaHoraProgramada** | Fecha y hora en la que el recurso debe ejecutar la tarea (programación). | Control del flujo de trabajo de los operadores. | Fecha/Hora | Sí | No | No |
| **EstadoGestion** | Estado actual de la tarea. | Monitoreo operativo (Consulta Crítica). | Enumeración (Pendiente, En Ejecución, Finalizado, Cancelado, Compromiso Pago) | Sí | No | No |
| **Resultado** | Resultado final de la interacción con el deudor (registrado vía Data Entry). | Alimentación del archivo bitácora (Diario de Operaciones). | Texto | No | No | No |
| **CodCliente (FK)** | Referencia al deudor a quien se dirige la gestión. | Enlace a la cuenta (Consulta Crítica). | Texto | Sí | No | No |
| **CodRecurso (FK)** | Referencia al recurso asignado para realizar la tarea. | Enlace al operador responsable (Reporte Disputed Items/Collector). | Texto | Sí | No | No |

### 4.3. Diccionario de Datos (Fichas de Tipos de Relación)

#### Relación: ASIGNADO\_A

| Nombre | Tipos de Entidad Participantes | Cardinalidades (min..max) | Justificación de Cardinalidades (Reglas de Negocio) |
| :--- | :--- | :--- | :--- |
| **ASIGNADO\_A** | CLIENTE, TIPOLOGIA\_COBRANZA | Cliente (1,1); Tipología (0,N) | Un cliente debe estar asignado actualmente a una y solo una tipología de cobranza (según sus reglas de monto/mora). Una tipología puede tener cero o muchos clientes asignados en un momento dado. |

#### Relación: ES\_ASIGNADO

| Nombre | Tipos de Entidad Participantes | Cardinalidades (min..max) | Justificación de Cardinalidades (Reglas de Negocio) |
| :--- | :--- | :--- | :--- |
| **ES\_ASIGNADO** | RECURSO, GESTION | Recurso (1,N); Gestión (1,1) | Un ticket (gestión) debe ser asignado a uno y solo un recurso para su ejecución. Un recurso puede tener asignadas muchas gestiones. |




