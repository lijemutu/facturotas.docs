
#### **Paso 1: Título y Descripción de la Operación**

  * **Nombre de la Operación (CamelCase):** `[nombreDeLaOperacion]`
  * **Descripción General:** `[Escribe aquí una descripción clara y concisa de lo que hace esta operación, para qué sirve y cuál es su principal beneficio. Similar a la sección "Descripción de la Operación" del ejemplo.]`

-----

#### **Paso 2: Parámetros de Entrada (Input)**

Proporciona los detalles de cada parámetro que recibe la operación.

  * **Parámetro 1:**
      * **Nombre:** `[nombre_param_1]`
      * **Tipo de Dato:** `[string/int/boolean]`
      * **Descripción:** `[Describe el propósito del parámetro. Incluye enlaces a otras secciones si es necesario, ej: [Solicita aquí](../../../#Requisitos)]`
  * **Parámetro 2:**
      * **Nombre:** `[nombre_param_2]`
      * **Tipo de Dato:** `[string]`
      * **Descripción:** `[Descripción del parámetro 2]`
  * **(Añade más parámetros según sea necesario)**

-----

#### **Paso 3: Parámetros de Salida (Output)**

Describe los atributos contenidos en el objeto de respuesta de la operación.

  * **Nombre del Objeto de Respuesta:** `[NombreRespuesta]`
  * **Descripción de la Respuesta:** `[Indica qué representa la respuesta. Puedes mencionar si es idéntica a otra operación, ej: "La respuesta de esta operación es idéntica a la de la operación 'timbrar'."]`
  * **Atributo 1:**
      * **Nombre:** `[nombre_atributo_1]`
      * **Tipo de Dato:** `[string/int]`
      * **Descripción:** `[Descripción del atributo 1]`
  * **Atributo 2:**
      * **Nombre:** `[nombre_atributo_2]`
      * **Tipo de Dato:** `[string]`
      * **Descripción:** `[Descripción del atributo 2]`
  * **(Añade más atributos según sea necesario)**

-----

#### **Paso 4: Estructura de Entrada (Opcional)**

Si la operación requiere un objeto complejo como entrada (ej. un JSON o XML), detállalo aquí.

  * **Formato de Entrada:** `[JSON/XML]`
  * **Nombre del Parámetro que lo Contiene:** `[ej: jsonB64]`
  * **Descripción de la Estructura:** `[Breve explicación sobre la estructura y cómo debe ser enviada. Menciona si debe ser codificada en Base64.]`
  * **Ejemplo de Estructura:**
    ```[json|xml]
    [Pega aquí el cuerpo completo del JSON o XML de ejemplo. Asegúrate de que esté bien formateado.]
    ```

-----

#### **Paso 5: Ejemplos de Código**

Proporciona ejemplos de implementación para los lenguajes de programación soportados.

  * **Lenguajes a incluir (separados por comas):** `C#,Java,Python,PHP`

  * **Para C\#:**

      * **Herramienta y Configuración:** `[Describe la herramienta (ej. svcutil) y el comando para generar el cliente SOAP. Especifica si es para DESARROLLO o PRODUCCIÓN.]`
      * **Código de Implementación:**
        ```csharp
        [Pega aquí el código completo de C#, incluyendo la construcción del objeto, la llamada al servicio y el manejo de la respuesta. Comenta el código para explicar los pasos clave.]
        ```

  * **Para Java:**

      * **Herramienta y Configuración:** `[Describe la herramienta (ej. wsimport) y el comando para generar las clases cliente.]`
      * **Código de Implementación:**
        ```java
        [Pega aquí el código completo de Java, siguiendo el estilo del ejemplo (uso de CompletableFuture, construcción del objeto, etc.).]`
        ```

  * **Para Python:**

      * **Herramienta y Configuración:** `[Describe la librería a usar (ej. Zeep) y cómo instalarla.]`
      * **Código de Implementación:**
        ```python
        [Pega aquí el código completo de Python, preferiblemente asíncrono, mostrando la creación del layout, la llamada y el manejo de la respuesta.]`
        ```

  * **Para PHP:**

      * **Herramienta y Configuración:** `[Describe la extensión necesaria (ej. php-soap) y cualquier configuración relevante.]`
      * **Código de Implementación:**
        ```php
        [Pega aquí el código completo de PHP, usando la clase SoapClient y mostrando cómo manejar la solicitud y la respuesta.]`
        ```

-----

#### **Paso 6: Ejemplo de Respuesta (SOAP)**

Muestra la estructura del sobre SOAP que el servicio devolverá.

  * **Descripción:** `[Breve descripción de la respuesta SOAP, ej: "La estructura de la respuesta SOAP es la misma que la de la operación 'timbrar'."]`
  * **XML de Respuesta:**
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <SOAP-ENV:Envelope ...>
        <SOAP-ENV:Body>
            <ns1:[nombreDeLaOperacion]Response xmlns:ns1="urn:ws_api">
                <return xsi:type="tns:[NombreRespuesta]">
                    <code xsi:type="xsd:string">CÓDIGO</code>
                    <message xsi:type="xsd:string">MENSAJE</message>
                    <data xsi:type="xsd:string"><![CDATA[[DATOS DE RESPUESTA]]]></data>
                </return>
            </ns1:[nombreDeLaOperacion]Response>
        </SOAP-ENV:Body>
    </SOAP-ENV:Envelope>
    ```

-----

#### **Paso 7: Códigos de Respuesta (Opcional)**

Si utilizas un shortcode de Hugo para mostrar códigos de respuesta estandarizados, especifícalo aquí.

  * **Shortcode de Hugo:** `[ej: {{< codigos-timbrado >}} o {{< codigos-cancelacion >}}]`