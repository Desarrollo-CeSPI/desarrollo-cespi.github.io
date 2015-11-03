---
layout: page
title: Nuevo Proyecto
permalink: /procedimientos/nuevo_proyecto
---

* Al inicio del proyecto, se debe conformar un grupo de al menos dos personas. Nunca el proyecto se iniciará por un único recurso
  * (GP) Se tratará de definir los recursos que participarán a lo largo del proyecto, pero esto puede cambiar en el tiempo de vida del mismo
  * (GP|PL) Definir roles de los integrantes seleccionados
* Crear proyectos
  * (PL) Redmine
  * (PL) Gitlab
  * (PL) Crear el proyecto en File Server (Own Cloud)
* Documentación:
  * (AF) Identificar la problemática actual
  * (AF) Discutir acerca del dominio del proyecto
  * (AF) De los puntos anteriores más entrevistas con el cliente, generar documento de casos de uso / Tickets en REDMINE a partir de entrevistas con el cliente
  * (AF|ARQ|LT) Plantear tests de aceptación, usando la técnica de Outside In Development
* Antes de desarrollar
  * (ARQ|LT) Definir tecnología a usar: frameworks, motores de DB y herramientas para lograr objetivos generales
  * (ARQ|LT|D) Discutir diseño y arquitectura general
  * (ARQ|LT|D) Tratar de generar a mano alzada (o herramienta) diagramas de clases o DB. Lo que ayude a enteder el diseño
* Inicialización del código
  * (ARQ|LT|D) Utilizar template de aplicaciones: dependiendo de si usa: sso, integrador, es para UNLP o no habrán combinaciones de librerías a usar.
  * (LT|D) Cuando se deba utilizar una librería, se deberá consultar el inventario de librerías usadas. Si no existe la librería necesaria, realizar una evaluación y agregar al inventario de librerías
  * (D) Usar comandos de scaffolding y no tanto desarrollo manual, considerando que DEBE EXISTIR UNA PERSONALIZACION que es parte del template

* Documentación que debe quedar siempre accesible
  * (ALL) Diagrama de la DB (rails-erd)
  * (ALL) Seed de datos para desarrollo
  * (ALL) Tests de aceptación
  * (PL) README de la aplicación con una breve descripción de qué cubre, con links a:
    * La documentación mencioanda arriba
    * Al Proyecto en GIT/Redmine
    * Al entorno de pruebas y producción
  * (LT) INSTALL con información a considerarse para su instalación en los diferentes entornos:
    * Especificar dependencias (librerías), servicios, vms docker/vagrant, etc según ambientes

* Capacitaciones
  * (ALL) Cuando se integra una nueva herramienta tratar de capacitar al equipo (del proyecto y tal vez a todo el equipo)

* Workflows de trabajo
  * Uso de redmine
    * (PL|D) Los tickets en redmine deben crearse con estimación en horas.
    * (PL|D) Cuando un ticket es asignado, debe indicarse fecha de fin.
  * SCRUM
    * (ALL) Estar preparados para las reuniones de inico y cierre de sprint. Todos debemos saber cuándo son
    * (PL) Mantener con la mayor frecuencia posible las reuniones diarias. Tartar que sean diarias
  * Desarrollo
    * Herramientas y documentación interna que explique como usarlas:
       * pry + binding / better errors
       * Editores / IDEs
    * (LT|D) Cada vez que se desarrolla nueva funcionalidad, implementar tests de unidad
    * (LT|D) Documentar solo el código necesario
    * (ALL) Describir la forma en se obtiene un DUMP de ambientes de testing/producción
  * Reuniones con el cliente
  * (PL|AF) Mantener minutas en un directorio en el File Server
  * (ALL) Tratar de usar CI
  * (ALL) Que los deploys ppuedan ser parte de CD
