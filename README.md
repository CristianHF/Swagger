# Swagger

* Es el lenguaje en el que se definen las APIs
* Es un lenguaje comprensible tanto para humanos como para máquinas
* Estructura de forma muy diferenciada las diferentes partes que componen un API
* Es un lenguaje para documentar
* Se guarda como fichero .yml o .yaml

### Estructura mínima en un formato Swagger
Como mínimo, un swagger debe tener obligatoriamente los siguientes elementos

* **swagger**: versión del lenguaje empleada
```yaml
swagger: "2.0"
```
* **info**: información básica del API, contempla 3 campos

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**title**: título del API

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**version**: número de versión del API

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**description**: descripción de lo que hace el API

* **paths**: lista de los paths y operaciones de un API
