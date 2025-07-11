
# consultarCfdisRelacionados

Esta sección detalla la operación para consultar la lista de CFDI relacionados a un comprobante específico a partir de su Folio Fiscal (UUID).

### Descripción de la Operación

Esta operación permite obtener una lista de los folios fiscales (UUID) de los comprobantes que están relacionados con un CFDI en particular. Para realizar la consulta, es necesario proporcionar las credenciales de la API, los archivos de certificado (.cer y .key) en formato PEM, el RFC del emisor y el UUID del comprobante del cual se desea obtener los CFDI relacionados.

La respuesta incluye un código de estado, un mensaje descriptivo y, en caso de éxito, una estructura JSON con la lista de los documentos relacionados.

### Parámetros de Entrada (Input)

| Parámetro   | Tipo de Dato | Descripción                                                               |
| :---------- | :----------- | :------------------------------------------------------------------------ |
| `apikey`    | `string`     | Credencial de acceso al servicio ([Solicita aquí](../../../#Requisitos)).    |
| `keyPEM`    | `string`     | Contenido del archivo de llave privada (.key) en formato PEM.             |
| `cerPEM`    | `string`     | Contenido del archivo de certificado (.cer) en formato PEM.               |
| `uuid`      | `string`     | Folio Fiscal (UUID) del CFDI del cual se consultarán los relacionados.    |
| `rfcEmisor` | `string`     | RFC del contribuyente emisor del CFDI.                                    |

### Parámetros de Salida (Output) - RespuestaRelacionados

| Atributo  | Tipo de Dato | Descripción                                                                 |
| :-------- | :----------- | :-------------------------------------------------------------------------- |
| `code`    | `string`     | Código de respuesta que indica el resultado de la operación.                |
| `message` | `string`     | Mensaje descriptivo sobre el resultado de la operación.                     |
| `data`    | `string`     | Estructura en formato JSON que contiene la lista de los CFDI relacionados.  |

### Ejemplo de Código

#### Solicitud (Request)

{{< tabs items="C#,Java,Python,PHP" >}}

  {{< tab >}}
  ### Implementación
  
  **Herramienta y Configuración:** Se utiliza la herramienta `svcutil` de .NET para generar el cliente a partir del WSDL.
  *   **Desarrollo:** `dotnet svcutil https://dev.facturaloplus.com/ws/servicio.do?wsdl`
  *   **Producción:** `dotnet svcutil https://app.facturaloplus.com/ws/servicio.do?wsdl`

  ```csharp
  using System;
  using System.Threading.Tasks;

   public class ConsultarRelacionadosRequest
   {
        public string Apikey { get; set; }
        public string KeyPEM { get; set; }
        public string CerPEM { get; set; }
        public string Uuid { get; set; }
        public string RfcEmisor { get; set; }
   }

   public async Task<RespuestaRelacionados> ConsultarRelacionadosAsync(ConsultarRelacionadosRequest request)
    {
        using var client = new ServicioTimbradoWSPortTypeClient("ServicioTimbradoWSPort");
        try
        {
            var response = await client.consultarCfdisRelacionadosAsync(request.Apikey, request.KeyPEM, request.CerPEM, request.Uuid, request.RfcEmisor);
            return response;
        }
        catch (Exception ex)
        {
           Console.WriteLine($"Error al consultar CFDI relacionados: {ex.Message}");
           throw;
        }
    }

   // Ejemplo de uso
   public async Task EjemploUsoConsultarRelacionadosAsync()
   {
       var request = new ConsultarRelacionadosRequest
       {
           Apikey = "TU_API_KEY_AQUI",
           KeyPEM = "CONTENIDO_KEY_PEM",
           CerPEM = "CONTENIDO_CER_PEM",
           Uuid = "5FD4E09E-52F4-4A0E-8E4D-39E6B8E47168",
           RfcEmisor = "ABC010101XYZ"
       };

       var resultado = await ConsultarRelacionadosAsync(request);

       if (resultado != null && resultado.code == "200")
       {
           Console.WriteLine("¡Consulta Exitosa!");
           Console.WriteLine($"Mensaje: {resultado.message}");
           Console.WriteLine($"Datos: {resultado.data}");
       }
       else
       {
           Console.WriteLine($"Error: {resultado?.message}");
       }
   }
  ```
  {{< /tab >}}

  {{< tab >}}
  ### Implementación

  **Herramienta y Configuración:** Se utiliza la herramienta `wsimport` del JDK para generar las clases de cliente.
  *   **Desarrollo:** `wsimport -keep -verbose https://dev.facturaloplus.com/ws/servicio.do?wsdl`
  *   **Producción:** `wsimport -keep -verbose https://app.facturaloplus.com/ws/servicio.do?wsdl`

  ```java
  import java.util.concurrent.CompletableFuture;

  public class ConsultarRelacionadosService {

    // ... (Definición de ExecutorService, Logger, etc.)

    public CompletableFuture<RespuestaRelacionados> consultarRelacionadosAsync(String apiKey, String keyPEM, String cerPEM, String uuid, String rfcEmisor) {
        return CompletableFuture.supplyAsync(() -> {
            try {
                ServicioTimbradoWS service = new ServicioTimbradoWS();
                ServicioTimbradoWSPortType port = service.getServicioTimbradoWSPort();
                return port.consultarCfdisRelacionados(apiKey, keyPEM, cerPEM, uuid, rfcEmisor);
            } catch (Exception ex) {
                throw new RuntimeException("Error al consultar CFDI relacionados", ex);
            }
        }, executor);
    }

    // Ejemplo de uso
    public static void main(String[] args) {
        ConsultarRelacionadosService service = new ConsultarRelacionadosService();
        String apiKey = "TU_API_KEY_AQUI";
        String keyPEM = "CONTENIDO_KEY_PEM";
        String cerPEM = "CONTENIDO_CER_PEM";
        String uuid = "5FD4E09E-52F4-4A0E-8E4D-39E6B8E47168";
        String rfcEmisor = "ABC010101XYZ";

        service.consultarRelacionadosAsync(apiKey, keyPEM, cerPEM, uuid, rfcEmisor)
            .whenComplete((resultado, ex) -> {
                if (ex != null) {
                    System.err.println("Error: " + ex.getMessage());
                } else if ("200".equals(resultado.getCode())) {
                    System.out.println("¡Consulta Exitosa!");
                    System.out.println("Mensaje: " + resultado.getMessage());
                    System.out.println("Datos: " + resultado.getData());
                } else {
                    System.err.println("Error en la respuesta: " + resultado.getMessage());
                }
                service.shutdown();
            }).join();
    }
  }
  ```
  {{< /tab >}}

  {{< tab >}}
  ### Implementación

  **Herramienta y Configuración:** Se utiliza la librería `zeep`. Para instalarla, ejecuta: `pip install zeep`

  ```python
  import asyncio
  from zeep.asyncio import AsyncClient

  class ConsultarRelacionadosService:
      def __init__(self, wsdl_url: str):
          self.wsdl_url = wsdl_url
          self.async_client = AsyncClient(self.wsdl_url)

      async def consultar_relacionados_async(self, **kwargs) -> dict:
          response = await self.async_client.service.consultarCfdisRelacionados(**kwargs)
          return response

  async def main():
      service = ConsultarRelacionadosService("https://dev.facturaloplus.com/ws/servicio.do?wsdl")
      
      params = {
          "apikey": "TU_API_KEY_AQUI",
          "keyPEM": "CONTENIDO_KEY_PEM",
          "cerPEM": "CONTENIDO_CER_PEM",
          "uuid": "5FD4E09E-52F4-4A0E-8E4D-39E6B8E47168",
          "rfcEmisor": "ABC010101XYZ"
      }

      resultado = await service.consultar_relacionados_async(**params)

      if resultado and resultado['code'] == '200':
          print("¡Consulta Exitosa!")
          print(f"Mensaje: {resultado['message']}")
          print(f"Datos: {resultado['data']}")
      else:
          print(f"Error al realizar la consulta: {resultado['message'] if resultado else 'Sin respuesta'}")

  if __name__ == "__main__":
      asyncio.run(main())
  ```
  {{< /tab >}}

  {{< tab >}}
  ### Implementación

  **Herramienta y Configuración:** Se requiere la extensión `php-soap`. Generalmente viene incluida con PHP, pero si no, se puede instalar con `apt-get install php-soap` (Debian/Ubuntu) o `yum install php-soap` (CentOS/RHEL). Asegúrate de que esté habilitada en tu `php.ini`.

  ```php
  <?php
  class ConsultarRelacionadosService {
      private string $wsdlUrl;

      public function __construct(string $wsdlUrl) {
          $this->wsdlUrl = $wsdlUrl;
      }

      public function consultarRelacionados(array $params): ?object {
          try {
              $soapClient = new SoapClient($this->wsdlUrl, ['trace' => 1, 'exceptions' => true]);
              $response = $soapClient->consultarCfdisRelacionados($params);
              return $response->return ?? null;
          } catch (Exception $e) {
              echo "Error: " . $e->getMessage();
              return null;
          }
      }
  }

  // Ejemplo de uso
  $service = new ConsultarRelacionadosService("https://dev.facturaloplus.com/ws/servicio.do?wsdl");
  
  $params = [
      'apikey'    => "TU_API_KEY_AQUI",
      'keyPEM'    => "CONTENIDO_KEY_PEM",
      'cerPEM'    => "CONTENIDO_CER_PEM",
      'uuid'      => "5FD4E09E-52F4-4A0E-8E4D-39E6B8E47168",
      'rfcEmisor' => "ABC010101XYZ"
  ];

  $resultado = $service->consultarRelacionados($params);

  header('Content-Type: application/json');
  if ($resultado && $resultado->code == '200') {
      echo json_encode([
          'status' => 'success',
          'message' => $resultado->message,
          'data' => json_decode($resultado->data)
      ], JSON_PRETTY_PRINT);
  } else {
      echo json_encode([
          'status' => 'error',
          'message' => $resultado->message ?? 'Error al realizar la consulta.'
      ], JSON_PRETTY_PRINT);
  }
  ?>
  ```
  {{< /tab >}}

{{< /tabs >}}

#### Respuesta (Response)

La respuesta SOAP contiene el resultado de la consulta de CFDI relacionados.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<SOAP-ENV:Envelope SOAP-ENV:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/"
xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/"
xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns:SOAP-ENC="http://schemas.xmlsoap.org/soap/encoding/"
xmlns:tns="urn:ws_api">
    <SOAP-ENV:Body>
        <ns1:consultarCfdisRelacionadosResponse xmlns:ns1="urn:ws_api">
            <return xsi:type="tns:RespuestaRelacionados">
                <code xsi:type="xsd:string">200</code>
                <message xsi:type="xsd:string">OK</code>
                <data xsi:type="xsd:string">JSON DE CFDIs RELACIONADOS</data>
            </return>
        </ns1:consultarCfdisRelacionadosResponse>
    </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

##### Códigos de respuesta

Los códigos de respuesta de la operación (éxito o error en la llamada) se pueden consultar en la sección general de códigos. El detalle específico de los CFDI relacionados se encuentra en el campo `data` de la respuesta.
