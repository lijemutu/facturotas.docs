# consultarEstadoSAT

Esta sección detalla la operación para consultar el estado actual de un CFDI (Comprobante Fiscal Digital por Internet) directamente en los registros del SAT.

### Descripción de la Operación

Esta operación permite verificar si un CFDI está vigente o cancelado. Proporciona información clave sobre el estado del comprobante, incluyendo si es cancelable y el estatus del proceso de cancelación, en caso de que se haya iniciado.

Para realizar la consulta, es necesario proporcionar los datos exactos del comprobante: el folio fiscal (UUID), los RFC del emisor y receptor, y el monto total.

### Parámetros de Entrada (Input)

| Parámetro     | Tipo de Dato | Descripción                                                              |
| :------------ | :----------- | :----------------------------------------------------------------------- |
| `apikey`      | `string`     | Credencial de acceso al servicio ([Solicita aquí](../../../#Requisitos)).   |
| `uuid`        | `string`     | Folio Fiscal (UUID) del CFDI que se desea consultar.                     |
| `rfcEmisor`   | `string`     | RFC del contribuyente emisor del CFDI.                                   |
| `rfcReceptor` | `string`     | RFC del contribuyente receptor del CFDI.                                 |
| `total`       | `string`     | Monto total exacto del CFDI, como cadena de texto.                       |

### Parámetros de Salida (Output) - consultarEstadoSATResponse

| Atributo             | Tipo de Dato | Descripción                                                                                                                                 |
| :------------------- | :----------- | :------------------------------------------------------------------------------------------------------------------------------------------ |
| `CodigoEstatus`      | `string`     | Indica el estado de la consulta. Un valor común es `S - Comprobante obtenido satisfactoriamente`.                                           |
| `EsCancelable`       | `string`     | Indica si el CFDI se puede cancelar. Valores: `Cancelable con aceptación`, `Cancelable sin aceptación`, `No cancelable`.                     |
| `Estado`             | `string`     | Describe el estado actual del CFDI. Valores: `Vigente`, `Cancelado`.                                                                        |
| `EstatusCancelacion` | `string`     | Detalla el proceso de cancelación. Valores: `(null)`, `En proceso`, `Plazo vencido`, `Solicitud rechazada`, `Cancelado sin aceptación`, `Cancelado con aceptación`. |

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

   public class ConsultarEstadoSATRequest
   {
        public string Apikey { get; set; }
        public string Uuid { get; set; }
        public string RfcEmisor { get; set; }
        public string RfcReceptor { get; set; }
        public string Total { get; set; }
   }

   public async Task<consultarEstadoSATResponse> ConsultarEstadoAsync(ConsultarEstadoSATRequest request)
    {
        using var client = new ServicioTimbradoWSPortTypeClient("ServicioTimbradoWSPort");
        try
        {
            var response = await client.consultarEstadoSATAsync(request.Apikey, request.Uuid, request.RfcEmisor, request.RfcReceptor, request.Total);
            return response;
        }
        catch (Exception ex)
        {
           Console.WriteLine($"Error al consultar estado en SAT: {ex.Message}");
           throw;
        }
    }

    public class consultarEstadoSATResponse
   {
      public string? Estado { get; set; }
      public string? EsCancelable { get; set; }
      public string? EstatusCancelacion { get; set; }
   }

   // Ejemplo de uso
   public async Task EjemploUsoConsultarEstadoAsync()
   {
       var request = new ConsultarEstadoSATRequest
       {
           Apikey = "TU_API_KEY_AQUI",
           Uuid = "5FD4E09E-52F4-4A0E-8E4D-39E6B8E47168",
           RfcEmisor = "ABC010101XYZ",
           RfcReceptor = "XAXX010101000",
           Total = "116.00"
       };

       var resultado = await ConsultarEstadoAsync(request);

       if (resultado != null)
       {
           Console.WriteLine("¡Consulta Exitosa!");
           Console.WriteLine($"Estado: {resultado.Estado}");
           Console.WriteLine($"Es Cancelable: {resultado.EsCancelable}");
           Console.WriteLine($"Estatus Cancelación: {resultado.EstatusCancelacion}");
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

  public class ConsultarEstadoSATService {

    // ... (Definición de ExecutorService, Logger, etc.)

    public CompletableFuture<ConsultarEstadoSATResponse> consultarEstadoSATAsync(String apiKey, String uuid, String rfcEmisor, String rfcReceptor, String total) {
        return CompletableFuture.supplyAsync(() -> {
            try {
                ServicioTimbradoWS service = new ServicioTimbradoWS();
                ServicioTimbradoWSPortType port = service.getServicioTimbradoWSPort();
                return port.consultarEstadoSAT(apiKey, uuid, rfcEmisor, rfcReceptor, total);
            } catch (Exception ex) {
                throw new RuntimeException("Error al consultar estado en SAT", ex);
            }
        }, executor);
    }

    // Ejemplo de uso
    public static void main(String[] args) {
        ConsultarEstadoSATService service = new ConsultarEstadoSATService();
        String apiKey = "TU_API_KEY_AQUI";
        String uuid = "5FD4E09E-52F4-4A0E-8E4D-39E6B8E47168";
        String rfcEmisor = "ABC010101XYZ";
        String rfcReceptor = "XAXX010101000";
        String total = "116.00";

        service.consultarEstadoSATAsync(apiKey, uuid, rfcEmisor, rfcReceptor, total)
            .whenComplete((resultado, ex) -> {
                if (ex != null) {
                    System.err.println("Error: " + ex.getMessage());
                } else {
                    System.out.println("¡Consulta Exitosa!");
                    System.out.println("Estado: " + resultado.getEstado());
                    System.out.println("Es Cancelable: " + resultado.getEsCancelable());
                    System.out.println("Estatus Cancelación: " + resultado.getEstatusCancelacion());
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

  class ConsultarEstadoSATService:
      def __init__(self, wsdl_url: str):
          self.wsdl_url = wsdl_url
          self.async_client = AsyncClient(self.wsdl_url)

      async def consultar_estado_async(self, **kwargs) -> dict:
          response = await self.async_client.service.consultarEstadoSAT(**kwargs)
          return response

  async def main():
      service = ConsultarEstadoSATService("https://dev.facturaloplus.com/ws/servicio.do?wsdl")
      
      params = {
          "apikey": "TU_API_KEY_AQUI",
          "uuid": "5FD4E09E-52F4-4A0E-8E4D-39E6B8E47168",
          "rfcEmisor": "ABC010101XYZ",
          "rfcReceptor": "XAXX010101000",
          "total": "116.00"
      }

      resultado = await service.consultar_estado_async(**params);

      if resultado:
          print("¡Consulta Exitosa!")
          print(f"Estado: {resultado['Estado']}")
          print(f"Es Cancelable: {resultado['EsCancelable']}")
          print(f"Estatus Cancelación: {resultado['EstatusCancelacion']}")
      else:
          print("Error al realizar la consulta.")

  if __name__ == "__main__":
      asyncio.run(main())
  ```
  {{< /tab >}}

  {{< tab >}}
  ### Implementación

  **Herramienta y Configuración:** Se requiere la extensión `php-soap`. Generalmente viene incluida con PHP, pero si no, se puede instalar con `apt-get install php-soap` (Debian/Ubuntu) o `yum install php-soap` (CentOS/RHEL). Asegúrate de que esté habilitada en tu `php.ini`.

  ```php
  <?php
  class ConsultarEstadoSATService {
      private string $wsdlUrl;

      public function __construct(string $wsdlUrl) {
          $this->wsdlUrl = $wsdlUrl;
      }

      public function consultarEstado(array $params): ?object {
          try {
              $soapClient = new SoapClient($this->wsdlUrl, ['trace' => 1, 'exceptions' => true]);
              $response = $soapClient->consultarEstadoSAT($params);
              return $response->return ?? null;
          } catch (Exception $e) {
              echo "Error: " . $e->getMessage();
              return null;
          }
      }
  }

  // Ejemplo de uso
  $service = new ConsultarEstadoSATService("https://dev.facturaloplus.com/ws/servicio.do?wsdl");
  
  $params = [
      'apikey'      => "TU_API_KEY_AQUI",
      'uuid'        => "5FD4E09E-52F4-4A0E-8E4D-39E6B8E47168",
      'rfcEmisor'   => "ABC010101XYZ",
      'rfcReceptor' => "XAXX010101000",
      'total'       => "116.00"
  ];

  $resultado = $service->consultarEstado($params);

  header('Content-Type: text/plain');
  if ($resultado) {
      echo "¡Consulta Exitosa!\n";
      echo "Estado: {$resultado->Estado}\n";
      echo "Es Cancelable: {$resultado->EsCancelable}\n";
      echo "Estatus Cancelación: {$resultado->EstatusCancelacion}\n";
  } else {
      echo "Error al realizar la consulta.\n";
  }
  ?>
  ```
  {{< /tab >}}

{{< /tabs >}}

#### Respuesta (Response)

La respuesta SOAP contiene el estado del CFDI consultado.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<SOAP-ENV:Envelope SOAP-ENV:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/"
xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/"
xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-
instance" xmlns:SOAP-ENC="http://schemas.xmlsoap.org/soap/encoding/"
xmlns:tns="urn:ws_api">
    <SOAP-ENV:Body>
        <ns1:consultarEstadoSATResponse xmlns:ns1="urn:ws_api">
            <return xsi:type="tns:RespuestaEstatusSAT">
                <CodigoEstatus xsi:type="xsd:string">CÓDIGO SAT</CodigoEstatus>
                <EsCancelable xsi:type="xsd:string">RESPUESTA SAT</EsCancelable>
                <Estado xsi:type="xsd:string">RESPUESTA SAT</Estado>
                <EstatusCancelacion xsi:type="xsd:string">RESP. SAT</EstatusCancelacion>
            </return>
        </ns1:consultarEstadoSATResponse>
    </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

##### Códigos de respuesta

Los códigos de respuesta de la operación en sí (éxito o error en la llamada) se pueden consultar en la sección general de códigos. El estado específico del CFDI se encuentra dentro de los campos `Estado`, `EsCancelable` y `EstatusCancelacion` de la respuesta.

