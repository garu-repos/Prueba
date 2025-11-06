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
|RF206                   |Consultar el historial de los clientes     |

---


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
---

## Requisitos de atributos de calidad

|**Atributos de calidad**      |Descripción                                                                                      |
|------------------------------|-------------------------------------------------------------------------------------------------|
|Seguridad                     |Solamento los usuarios de alta jerarquía son autorizados para registrar o modificar datos.       |
|Disponibilidad                |El módulo debe estar disponible en el horario laboral del 8:00 a 20:00 horas.                    |
|Escalabilidad                 |El módulo debe mostrar 100 clientes deudores con un tiempo de respuesta de a lo mucho 3 segundos.|
