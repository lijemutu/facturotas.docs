# consultarAutorizacionesPendientes

Esta sección detalla la operación para que un contribuyente receptor consulte las solicitudes de cancelación de CFDI que están pendientes de su autorización.

### Descripción de la Operación

Cuando un emisor intenta cancelar un CFDI que requiere la aceptación del receptor, esta operación permite al receptor consultar cuáles son esas solicitudes pendientes. Para autenticarse ante el SAT y obtener esta lista, el receptor debe proporcionar su Certificado de Sello Digital (CSD) en formato PEM.

La respuesta incluirá una lista de los folios fiscales (UUIDs) de los comprobantes que esperan una acción (aceptación o rechazo) por parte del receptor.

### Parámetros de Entrada (Input)

| Parámetro | Tipo de Dato | Descripción |
| :--- | :--- | :--- |
| `apikey` | `string` | Credencial de acceso al servicio ([Solicita aquí](../../../#Requisitos)). |
| `keyPEM` | `string` | Contenido del archivo de la llave privada (`.key`) del **receptor** en formato PEM. |
| `cerPEM` | `string` | Contenido del archivo del certificado de llave pública (`.cer`) del **receptor** en formato PEM. |

### Parámetros de Salida (Output) - RespuestaPendientes

| Atributo | Tipo de Dato | Descripción |
| :--- | :--- | :--- |
| `code` | `string` | Código de respuesta de la operación. |
| `message` | `string` | Mensaje detallado de la respuesta. |
| `data` | `string` | Lista de UUIDs de los CFDI pendientes de autorización, separados por comas. |

### Ejemplo de Código

#### Solicitud (Request)

{{< tabs items="C#,Java,Python,PHP" >}}

  {{< tab >}}
  ### Implementación
  
  **Herramienta y Configuración:** Se utiliza la herramienta `svcutil` de .NET para generar el cliente a partir del WSDL.
  *   **Desarrollo:** `dotnet svcutil https://dev.facturaloplus.com/ws/servicio.do?wsdl`
  *   **Producción:** `dotnet svcutil https://app.facturaloplus.com/ws/servicio.do?wsdl`

  ```csharp
  // Lee el contenido de los archivos .key.pem y .cer.pem
  string keyPem = File.ReadAllText("ruta/al/archivo.key.pem");
  string cerPem = File.ReadAllText("ruta/al/archivo.cer.pem");

  // Crea la solicitud para el servicio
  var request = new consultarAutorizacionesPendientesRequest
  {
      apikey = "TU_API_KEY",
      keyPEM = keyPem,
      cerPEM = cerPem
  };

  // Llama a la operación del servicio web
  var response = await client.consultarAutorizacionesPendientesAsync(request);

  // Procesa la respuesta
  Console.WriteLine($"Código: {response.return.code}");
  Console.WriteLine($"Mensaje: {response.return.message}");
  Console.WriteLine($"UUIDs Pendientes: {response.return.data}");
  ```
  {{< /tab >}}

  {{< tab >}}
  ### Implementación

  **Herramienta y Configuración:** Se utiliza la herramienta `wsimport` del JDK para generar las clases de cliente.
  *   **Desarrollo:** `wsimport -keep -verbose https://dev.facturaloplus.com/ws/servicio.do?wsdl`
  *   **Producción:** `wsimport -keep -verbose https://app.facturaloplus.com/ws/servicio.do?wsdl`

  ```java
  // Lee el contenido de los archivos PEM
  String keyPem = new String(Files.readAllBytes(Paths.get("ruta/al/archivo.key.pem")));
  String cerPem = new String(Files.readAllBytes(Paths.get("ruta/al/archivo.cer.pem")));

  // Prepara la petición
  ConsultarAutorizacionesPendientes peticion = new ConsultarAutorizacionesPendientes();
  peticion.setApikey("TU_API_KEY");
  peticion.setKeyPEM(keyPem);
  peticion.setCerPEM(cerPem);

  // Llama al servicio
  RespuestaPendientes respuesta = port.consultarAutorizacionesPendientes(peticion);

  // Procesa la respuesta
  System.out.println("Código: " + respuesta.getCode());
  System.out.println("Mensaje: " + respuesta.getMessage());
  System.out.println("UUIDs Pendientes: " + respuesta.getData());
  ```
  {{< /tab >}}

  {{< tab >}}
  ### Implementación

  **Herramienta y Configuración:** Se utiliza la librería `zeep`. Para instalarla, ejecuta: `pip install zeep`

  ```python
  from zeep import Client

  # Inicializa el cliente con la URL del WSDL
  client = Client('URL_DEL_WSDL_AQUI')

  # Lee el contenido de los archivos PEM
  with open('ruta/al/archivo.key.pem', 'r') as f:
      key_pem_content = f.read()
  with open('ruta/al/archivo.cer.pem', 'r') as f:
      cer_pem_content = f.read()

  # Llama al servicio con los parámetros requeridos
  response = client.service.consultarAutorizacionesPendientes(
      apikey='TU_API_KEY',
      keyPEM=key_pem_content,
      cerPEM=cer_pem_content
  )

  # Imprime la respuesta
  print(f"Código: {response.code}")
  print(f"Mensaje: {response.message}")
  print(f"UUIDs Pendientes: {response.data}")
  ```
  {{< /tab >}}

  {{< tab >}}
  ### Implementación

  **Herramienta y Configuración:** Se requiere la extensión `php-soap`. Generalmente viene incluida con PHP, pero si no, se puede instalar con `apt-get install php-soap` (Debian/Ubuntu) o `yum install php-soap` (CentOS/RHEL). Asegúrate de que esté habilitada en tu `php.ini`.

  ```php
  // Inicializa el cliente SOAP
  $client = new SoapClient('URL_DEL_WSDL_AQUI');

  // Lee el contenido de los archivos PEM
  $key_pem = file_get_contents('ruta/al/archivo.key.pem');
  $cer_pem = file_get_contents('ruta/al/archivo.cer.pem');

  // Prepara los parámetros para la solicitud
  $params = [
      'apikey' => 'TU_API_KEY',
      'keyPEM' => $key_pem,
      'cerPEM' => $cer_pem
  ];

  // Llama a la operación del servicio
  $response = $client->consultarAutorizacionesPendientes($params);

  // Muestra la respuesta
  echo "Código: " . $response->return->code . "\n";
  echo "Mensaje: " . $response->return->message . "\n";
  echo "UUIDs Pendientes: " . $response->return->data . "\n";
  ```
  {{< /tab >}}

{{< /tabs >}}

#### Respuesta (Response)

La respuesta SOAP contiene la lista de UUIDs pendientes.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/" xmlns:ns1="urn:ws_api" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:tns="urn:ws_api">
    <SOAP-ENV:Body>
        <ns1:consultarAutorizacionesPendientesResponse>
            <return xsi:type="tns:RespuestaPendientes">
                <code xsi:type="xsd:string">200</code>
                <message xsi:type="xsd:string">OK</message>
                <data xsi:type="xsd:string"><![CDATA[UUID_1,UUID_2,UUID_3]]></data>
            </return>
        </ns1:consultarAutorizacionesPendientesResponse>
    </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

##### Códigos de respuesta

{{< codigos-consulta-autorizaciones >}}