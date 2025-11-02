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
