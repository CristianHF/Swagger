# Swagger

* Es el lenguaje en el que se definen las APIs
* Es un lenguaje comprensible tanto para humanos como para máquinas
* Estructura de forma muy diferenciada las diferentes partes que componen un API
* Es un lenguaje para documentar
* Se guarda como fichero .yml o .yaml

-----

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
```yaml
info:
  title: Accounts
  version: 1.0.0
  description: Allows get accounts list and account details.
```

* **paths**: lista de los paths y operaciones de un API
```yaml
paths:
  /accounts:
    get:
      responses:
        '200':
          description: OK
  /accounts/{account_id}:
    get:
      responses:
        '200':
          description: OK
      parameters:
        - name: account_id
          type: string
          in: path
          required: true
```

-----

### Objeto definitions
Se usa para definir los tipos de datos que pueden ser consumidos o producidos por el API. Luego en cada operación podemos referenciarlos para incluirlos en la petición o en la respuesta.

Por ejemplo, si definimos un objeto que de respuesta a la operación de listado de cuentas, una de sus propiedades debe ser de tipo lista (array) y además hay que indicar de que tipo serán los elementos de esa lista.
```yaml
definitions:
  accountsList:
    properties:
      accountsList:
        type: array
        items:
          $ref: '#/definitions/accountDetails'
```
Aunque en este punto el objeto **accountDetails** aún no está definido, no importa, se puede definir justo después o en cualquier otro sitio siempre dentro del objeto **definitions**
```yaml
accountDetails:
    properties:
      accountId:
        type: string
        description: Unique account identifier.
        example: '1234567890'
      alias:
        type: string
        description: Simple name for identify each account.
        example: MyAccount1
      mainBalance:
        $ref: '#/definitions/balance'
        description: "Account's main balance."
      lastUpdateDate:
        type: string
        description: Last time the account was updated.
        format: date
        example: 01/20/2020
```
Siempre que una propiedad de un objeto, sea otro objeto, debemos referenciarlo con **$ref**, como es el caso del objeto **balance**
```yaml
  balance:
    properties:
      amount:
        type: number
        description: Amount.
        format: double
        example: 10
      currency:
        type: string
        description: Currency.
        example: EUR
    description: "Object that represent account's balances. Includes amount and currency."
```
Para completar las definiciones de objetos, definimos la lista de errores y el error
```yaml
  errorList:
    type: object
    properties:
      errors:
        type: array
        items:
          $ref: '#/definitions/error'
  error:
    properties:
      code:
        type: string
        description: Error code.
        example: 'XXX'
      message:
        type: string
        description: Error message.
        example: Message
      level:
        type: string
        description: Error level.
        example: Error
      description:
        type: string
        description: Error description.
        example: Description
```
