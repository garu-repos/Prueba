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
