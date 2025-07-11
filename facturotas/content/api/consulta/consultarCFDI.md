# consultarCFDI

Esta sección detalla la operación para consultar el XML de un CFDI previamente timbrado con el servicio.

### Descripción de la Operación

Esta operación permite obtener el XML de un comprobante fiscal que ha sido timbrado utilizando la misma cuenta de la API. La consulta está limitada a CFDI cuya fecha de timbrado no exceda los 3 meses desde la fecha de la solicitud.

Para realizar la consulta, solo es necesario proporcionar la `apikey` y el `uuid` del comprobante.

### Parámetros de Entrada (Input)

| Parámetro | Tipo de Dato | Descripción                                                                 |
| :-------- | :----------- | :-------------------------------------------------------------------------- |
| `apikey`  | `string`     | Credencial de acceso al servicio ([Solicita aquí](../../../#Requisitos)).      |
| `uuid`    | `string`     | Folio Fiscal (UUID) del CFDI que se desea consultar.                        |

### Parámetros de Salida (Output) - RespuestaConsultarCFDI

| Atributo  | Tipo de Dato | Descripción                                                                 |
| :-------- | :----------- | :-------------------------------------------------------------------------- |
| `code`    | `string`     | Código de respuesta que indica el resultado de la operación.                |
| `message` | `string`     | Mensaje descriptivo sobre el resultado de la operación.                     |
| `data`    | `string`     | Contenido del XML del CFDI consultado.                                      |

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

   public class ConsultarCFDIRequest
   {
        public string Apikey { get; set; }
        public string Uuid { get; set; }
   }

   public async Task<RespuestaConsultarCFDI> ConsultarCFDIAsync(ConsultarCFDIRequest request)
    {
        using var client = new ServicioTimbradoWSPortTypeClient("ServicioTimbradoWSPort");
        try
        {
            var response = await client.consultarCFDIAsync(request.Apikey, request.Uuid);
            return response;
        }
        catch (Exception ex)
        {
           Console.WriteLine($"Error al consultar el CFDI: {ex.Message}");
           throw;
        }
    }

   // Ejemplo de uso
   public async Task EjemploUsoConsultarCFDIAsync()
   {
       var request = new ConsultarCFDIRequest
       {
           Apikey = "TU_API_KEY_AQUI",
           Uuid = "5FD4E09E-52F4-4A0E-8E4D-39E6B8E47168"
       };

       var resultado = await ConsultarCFDIAsync(request);

       if (resultado != null && resultado.code == "200")
       {
           Console.WriteLine("¡Consulta Exitosa!");
           Console.WriteLine($"Mensaje: {resultado.message}");
           Console.WriteLine("XML del CFDI:");
           Console.WriteLine(resultado.data);
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

  public class ConsultarCFDIService {

    // ... (Definición de ExecutorService, Logger, etc.)

    public CompletableFuture<RespuestaConsultarCFDI> consultarCFDIAsync(String apiKey, String uuid) {
        return CompletableFuture.supplyAsync(() -> {
            try {
                ServicioTimbradoWS service = new ServicioTimbradoWS();
                ServicioTimbradoWSPortType port = service.getServicioTimbradoWSPort();
                return port.consultarCFDI(apiKey, uuid);
            } catch (Exception ex) {
                throw new RuntimeException("Error al consultar el CFDI", ex);
            }
        }, executor);
    }

    // Ejemplo de uso
    public static void main(String[] args) {
        ConsultarCFDIService service = new ConsultarCFDIService();
        String apiKey = "TU_API_KEY_AQUI";
        String uuid = "5FD4E09E-52F4-4A0E-8E4D-39E6B8E47168";

        service.consultarCFDIAsync(apiKey, uuid)
            .whenComplete((resultado, ex) -> {
                if (ex != null) {
                    System.err.println("Error: " + ex.getMessage());
                } else if ("200".equals(resultado.getCode())) {
                    System.out.println("¡Consulta Exitosa!");
                    System.out.println("Mensaje: " + resultado.getMessage());
                    System.out.println("XML del CFDI:");
                    System.out.println(resultado.getData());
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

  class ConsultarCFDIService:
      def __init__(self, wsdl_url: str):
          self.wsdl_url = wsdl_url
          self.async_client = AsyncClient(self.wsdl_url)

      async def consultar_cfdi_async(self, **kwargs) -> dict:
          response = await self.async_client.service.consultarCFDI(**kwargs)
          return response

  async def main():
      service = ConsultarCFDIService("https://dev.facturaloplus.com/ws/servicio.do?wsdl")
      
      params = {
          "apikey": "TU_API_KEY_AQUI",
          "uuid": "5FD4E09E-52F4-4A0E-8E4D-39E6B8E47168"
      }

      resultado = await service.consultar_cfdi_async(**params)

      if resultado and resultado['code'] == '200':
          print("¡Consulta Exitosa!")
          print(f"Mensaje: {resultado['message']}")
          print("XML del CFDI:")
          print(resultado['data'])
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
  class ConsultarCFDIService {
      private string $wsdlUrl;

      public function __construct(string $wsdlUrl) {
          $this->wsdlUrl = $wsdlUrl;
      }

      public function consultarCFDI(array $params): ?object {
          try {
              $soapClient = new SoapClient($this->wsdlUrl, ['trace' => 1, 'exceptions' => true]);
              $response = $soapClient->consultarCFDI($params);
              return $response->return ?? null;
          } catch (Exception $e) {
              echo "Error: " . $e->getMessage();
              return null;
          }
      }
  }

  // Ejemplo de uso
  $service = new ConsultarCFDIService("https://dev.facturaloplus.com/ws/servicio.do?wsdl");
  
  $params = [
      'apikey'    => "TU_API_KEY_AQUI",
      'uuid'      => "5FD4E09E-52F4-4A0E-8E4D-39E6B8E47168"
  ];

  $resultado = $service->consultarCFDI($params);

  header('Content-Type: text/plain');
  if ($resultado && $resultado->code == '200') {
      echo "¡Consulta Exitosa!\n";
      echo "Mensaje: {$resultado->message}\n";
      echo "XML del CFDI:\n";
      echo $resultado->data;
  } else {
      echo "Error al realizar la consulta: " . ($resultado->message ?? 'Error desconocido');
  }
  ?>
  ```
  {{< /tab >}}

{{< /tabs >}}

#### Respuesta (Response)

La respuesta SOAP contiene el XML del CFDI consultado.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<SOAP-ENV:Envelope SOAP-ENV:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/"
xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/"
xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns:SOAP-ENC="http://schemas.xmlsoap.org/soap/encoding/"
xmlns:tns="urn:ws_api">
    <SOAP-ENV:Body>
        <ns1:consultarCFDIResponse xmlns:ns1="urn:ws_api">
            <return xsi:type="tns:RespuestaConsultarCFDI">
                <code xsi:type="xsd:string">200</code>
                <message xsi:type="xsd:string">OK</code>
                <data xsi:type="xsd:string"><![CDATA[<?xml version="1.0" encoding="utf-8"?>
<cfdi:Comprobante ...> ... </cfdi:Comprobante>]]></data>
            </return>
        </ns1:consultarCFDIResponse>
    </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

##### Códigos de respuesta

Los códigos de respuesta de la operación (éxito o error en la llamada) se pueden consultar en la sección general de códigos. El XML del comprobante se encuentra en el campo `data` de la respuesta.
