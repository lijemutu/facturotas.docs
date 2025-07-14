 
# consultarCreditosDisponibles

Esta sección detalla la operación para consultar la cantidad de créditos (timbres) disponibles en la cuenta asociada a la API Key.

### Descripción de la Operación

Esta operación permite conocer el saldo actual de timbres para poder realizar operaciones de timbrado y cancelación. Es una consulta sencilla que solo requiere la credencial de acceso para obtener el número de créditos restantes.

### Parámetros de Entrada (Input)

| Parámetro | Tipo de Dato | Descripción                                                              |
| :-------- | :----------- | :----------------------------------------------------------------------- |
| `apikey`  | `string`     | Credencial de acceso al servicio ([Solicita aquí](../../../#Requisitos)).   |

### Parámetros de Salida (Output) - RespuestaCreditos

| Atributo  | Tipo de Dato | Descripción                                                                 |
| :-------- | :----------- | :-------------------------------------------------------------------------- |
| `code`    | `string`     | Código de respuesta que indica el resultado de la operación.                |
| `message` | `string`     | Mensaje descriptivo sobre el resultado de la operación.                     |
| `data`    | `int`        | Número de créditos (timbres) disponibles en la cuenta.                      |

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

   public class ConsultarCreditosRequest
   {
        public string Apikey { get; set; }
   }

   public async Task<RespuestaCreditos> ConsultarCreditosAsync(ConsultarCreditosRequest request)
    {
        using var client = new ServicioTimbradoWSPortTypeClient("ServicioTimbradoWSPort");
        try
        {
            var response = await client.consultarCreditosDisponiblesAsync(request.Apikey);
            return response;
        }
        catch (Exception ex)
        {
           Console.WriteLine($"Error al consultar los créditos: {ex.Message}");
           throw;
        }
    }

    public class RespuestaCreditos
   {
      public string? Code { get; set; }
      public string? Message { get; set; }
      public string? Data { get; set; }
   }

   // Ejemplo de uso
   public async Task EjemploUsoConsultarCreditosAsync()
   {
       var request = new ConsultarCreditosRequest
       {
           Apikey = "TU_API_KEY_AQUI"
       };

       var resultado = await ConsultarCreditosAsync(request);

       if (resultado != null && resultado.Code == "200")
       {
           Console.WriteLine("¡Consulta Exitosa!");
           Console.WriteLine($"Mensaje: {resultado.Message}");
           Console.WriteLine($"Créditos Disponibles: {resultado.Data}");
       }
       else
       {
           Console.WriteLine($"Error: {resultado?.Message}");
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

  public class ConsultarCreditosService {

    // ... (Definición de ExecutorService, Logger, etc.)

    public CompletableFuture<RespuestaCreditos> consultarCreditosAsync(String apiKey) {
        return CompletableFuture.supplyAsync(() -> {
            try {
                ServicioTimbradoWS service = new ServicioTimbradoWS();
                ServicioTimbradoWSPortType port = service.getServicioTimbradoWSPort();
                return port.consultarCreditosDisponibles(apiKey);
            } catch (Exception ex) {
                throw new RuntimeException("Error al consultar los créditos", ex);
            }
        }, executor);
    }

    // Ejemplo de uso
    public static void main(String[] args) {
        ConsultarCreditosService service = new ConsultarCreditosService();
        String apiKey = "TU_API_KEY_AQUI";

        service.consultarCreditosAsync(apiKey)
            .whenComplete((resultado, ex) -> {
                if (ex != null) {
                    System.err.println("Error: " + ex.getMessage());
                } else if ("200".equals(resultado.getCode())) {
                    System.out.println("¡Consulta Exitosa!");
                    System.out.println("Mensaje: " + resultado.getMessage());
                    System.out.println("Créditos Disponibles: " + resultado.getData());
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
  from zeep.asyncio import AsyncClient;

  class ConsultarCreditosService:
      def __init__(self, wsdl_url: str):
          self.wsdl_url = wsdl_url
          self.async_client = AsyncClient(self.wsdl_url)

      async def consultar_creditos_async(self, **kwargs) -> dict:
          response = await self.async_client.service.consultarCreditosDisponibles(**kwargs)
          return response

  async def main():
      service = ConsultarCreditosService("https://dev.facturaloplus.com/ws/servicio.do?wsdl")
      
      params = {
          "apikey": "TU_API_KEY_AQUI"
      }

      resultado = await service.consultar_creditos_async(**params)

      if resultado and resultado['code'] == '200':
          print("¡Consulta Exitosa!")
          print(f"Mensaje: {resultado['message']}")
          print(f"Créditos Disponibles: {resultado['data']}")
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
  class ConsultarCreditosService {
      private string $wsdlUrl;

      public function __construct(string $wsdlUrl) {
          $this->wsdlUrl = $wsdlUrl;
      }

      public function consultarCreditos(array $params): ?object {
          try {
              $soapClient = new SoapClient($this->wsdlUrl, ['trace' => 1, 'exceptions' => true]);
              $response = $soapClient->consultarCreditosDisponibles($params);
              return $response->return ?? null;
          } catch (Exception $e) {
              echo "Error: " . $e->getMessage();
              return null;
          }
      }
  }

  // Ejemplo de uso
  $service = new ConsultarCreditosService("https://dev.facturaloplus.com/ws/servicio.do?wsdl");
  
  $params = [
      'apikey'    => "TU_API_KEY_AQUI"
  ];

  $resultado = $service->consultarCreditos($params);

  header('Content-Type: text/plain');
  if ($resultado && $resultado->code == '200') {
      echo "¡Consulta Exitosa!\n";
      echo "Mensaje: {$resultado->message}\n";
      echo "Créditos Disponibles: {$resultado->data}\n";
  } else {
      echo "Error al realizar la consulta: " . ($resultado->message ?? 'Error desconocido');
  }
  ?>
  ```
  {{< /tab >}}

{{< /tabs >}}

#### Respuesta (Response)

La respuesta SOAP contiene el número de créditos disponibles.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<SOAP-ENV:Envelope SOAP-ENV:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/"
xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/"
xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns:SOAP-ENC="http://schemas.xmlsoap.org/soap/encoding/"
xmlns:tns="urn:ws_api">
    <SOAP-ENV:Body>
        <ns1:consultarCreditosDisponiblesResponse xmlns:ns1="urn:ws_api">
            <return xsi:type="tns:RespuestaCreditos">
                <code xsi:type="xsd:string">200</code>
                <message xsi:type="xsd:string">OK</code>
                <data xsi:type="xsd:int">1000</data>
            </return>
        </ns1:consultarCreditosDisponiblesResponse>
    </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

##### Códigos de respuesta

Los códigos de respuesta de la operación (éxito o error en la llamada) se pueden consultar en la sección general de códigos. El número de créditos se encuentra en el campo `data` de la respuesta.

```