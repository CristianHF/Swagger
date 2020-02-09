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
          schema:
            $ref: '#/definitions/accountsList'
        '500':
          description: Error
          schema:
            $ref: '#/definitions/errorList'
  /accounts/{account_id}:
    get:
      responses:
        '200':
          description: OK
          schema:
            $ref: '#/definitions/accountDetails'
        '500':
          description: Error
          schema:
            $ref: '#/definitions/errorList'
      parameters:
        - name: account_id
          type: string
          in: path
          required: true
```
En esta parte se deben definir los recursos en un primer nivel y luego el verbo que pueden utilizar en el siguiente (get, post, put, patch, delete,...). Para cada operación se definen además de las diferentes respuestas que puede devolver, indicándolo en la etiqueta **schema**, también los parámetros de entrada que conforman la llamada, ya sean de tipo path, query string o body en la etiqueta **in** y la obligatoriedad en la etiqueta **required**.

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

-----

### Otros objetos Swagger

* **basePath**: si todas las operaciones tienen una parte común al inicio, se puede extraer e indicarlo aquí. Por defecto es una barra.
```yaml
basePath: /accounts
```
* **securityDefinitions**: aquí se puede indicar el flujo de seguridad OAuth que usa el API para poder consumirla, los campos obligatorios son el tipo, tipo de flujo y la URL que se usaría para obtener el token de acceso, opcionalmente se pueden indicar los scopes en el caso de que cada operacion necesite una autorización diferente. También aquí se puede indicar el API Key.
```yaml
securityDefinitions:
  Client Credentials:
    type: oauth2
    description: Client credentials security flow.
    flow: application
    scopes:
      accounts_list.read: Accounts list scope.
      account_details.read: Account details scope.
    tokenUrl: 'https://localhost.com/oauth/token'
  X-Client-Id:
    type: apiKey
    in: header
    name: X-Client-Id
    description: Client id header
```

* **security**: cuando está en primer nivel, lo que se hace es indicar cuál va a ser la definición de seguridad común que se va a aplicar a todas las operaciones, en este caso, lo más básico para llamar a un API es el API Key.
```yaml
security:
  - X-Client-Id: []
```
Si lo incluimos dentro de la definición de una operación, indicamos el scope para el que se tiene que validar el access token.
```yaml
  /accounts:
    get:
      responses:
        '200':
          description: OK
          schema:
            $ref: '#/definitions/accountsList'
        '500':
          description: Internal Server Error
          schema:
            $ref: '#/definitions/errorList'
      summary: Accounts list.
      description: List of accounts.
      security:
        - Client Credentials:
            - accounts_list.read
```
* **parameters**: si tenemos algún parámetro de entrada que se repita en varias operaciones, podemos definirlo aquí y referenciarlo cada vez que lo necesitemos, así sólo se define una vez. 
```yaml
parameters:
  Authorization:
    name: Authorization
    type: string
    required: true
    in: header
    description: Authorization header.
```
Esta cabecera la podemos referenciar en cada operación para tener una definición de seguridad completa
```yaml
paths:
  /accounts:
    get:
      responses:
        '200':
          description: OK
          schema:
            $ref: '#/definitions/accountsList'
        '500':
          description: Internal Server Error
          schema:
            $ref: '#/definitions/errorList'
      summary: Accounts list.
      description: List of accounts.
      parameters:
        - $ref: '#/parameters/Authorization'
      security:
        - Client Credentials:
            - accounts_list.read
  '/accounts/{account_id}':
    get:
      responses:
        '200':
          description: OK
          schema:
            $ref: '#/definitions/accountDetails'
        '404':
          description: Not Found
          schema:
            $ref: '#/definitions/errorList'
        '500':
          description: Internal Server Error
          schema:
            $ref: '#/definitions/errorList'
      summary: Account details.
      description: Details of an account.
      parameters:
        - $ref: '#/parameters/Authorization'
        - name: account_id
          type: string
          required: true
          in: path
          description: Unique account identifier.
      security:
        - Client Credentials:
            - account_details.read
```
