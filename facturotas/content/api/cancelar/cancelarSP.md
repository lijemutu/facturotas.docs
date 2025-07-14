---
title: Cancelar Sector Primario
---

Esta sección detalla la operación para solicitar la cancelación de un CFDI emitido por un contribuyente del Sector Primario (SP).

### Descripción de la Operación

Esta operación permite cancelar un CFDI de Sector Primario de forma simplificada. A diferencia de otros métodos de cancelación, este no requiere el envío de los Certificados de Sello Digital (CSD) del emisor. La autenticación se realiza a través del `apikey`.

Este método es exclusivo para los CFDI que fueron expedidos para el Sector Primario.

### Parámetros de Entrada (Input)

| Parámetro | Tipo de Dato | Descripción                                                              |
| :-------- | :----------- | :----------------------------------------------------------------------- |
| `apikey`    | `string`     | Credencial de acceso al servicio ([Solicita aquí](../../../#Requisitos)).   |
| `rfcEmisor` | `string`     | RFC del contribuyente emisor del CFDI (del Sector Primario).             |
| `uuid`      | `string`     | Folio Fiscal (UUID) del CFDI que se desea cancelar.                      |

### Parámetros de Salida (Output) - RespuestaCancelar

La respuesta de esta operación es idéntica a la de la operación `cancelar2`.

| Atributo  | Tipo de Dato | Descripción                                                                                             |
| :-------- | :----------- | :------------------------------------------------------------------------------------------------------ | 
| `code`    | `string`     | Código de respuesta de la operación.                                                                    |
| `message` | `string`     | Mensaje detallado de la respuesta.                                                                      |
| `data`    | `string`     | Acuse de cancelación en formato XML, devuelto por el SAT.                                               |
| `status`  | `string`     | Indica el estado de la solicitud. Puede ser `success` (éxito) o `error` (fallido).                      |

### Ejemplo de Código

#### Solicitud (Request)

{{< tabs items="C#,Java,Python,PHP" >}}

  {{< tab >}}
  ### Herramienta svcutil
  Descarga e instala la herramienta [svcutil](https://learn.microsoft.com/en-us/dotnet/core/additional-tools/dotnet-svcutil-guide?tabs=dotnetsvcutil2x)
  
  Ejecuta el comando siguiente (**DESARROLLO**)

  ```
  svcutil.exe https://dev.facturaloplus.com/ws/servicio.do?wsdl /out:ServicioTimbradoClient.cs /config:app.config
  ```
  Esto genera dos archivos: ServicioTimbradoClient.cs y la configuración en app.config

  ### Implementación

  ```csharp
  using System;
  using System.Threading.Tasks;

   public class CancelarSPRequest
   {
        public string Apikey { get; set; }
        public string RfcEmisor { get; set; }
        public string Uuid { get; set; }
   }

   public async Task<RespuestaCancelar> CancelarSPAsync(CancelarSPRequest request)
    {
        using var client = new ServicioTimbradoWSPortTypeClient("ServicioTimbradoWSPort");
        try
        {
            var response = await client.cancelarSPAsync(request.Apikey, request.RfcEmisor, request.Uuid);
            return new RespuestaCancelar { Code = response.code, Message = response.message, Data = response.data, Status = response.status };
        }
        catch (Exception ex)
        {
           Console.WriteLine($"Error al cancelar CFDI de Sector Primario: {ex.Message}");
           throw;
        }
    }

    public class RespuestaCancelar
   {
      public string? Code { get; set; }
      public string? Message { get; set; }
      public string? Data { get; set; }
      public string? Status { get; set; }
   }

   // Ejemplo de uso
   public async Task EjemploUsoCancelarSPAsync()
   {
       var request = new CancelarSPRequest
       {
           Apikey = "TU_API_KEY_AQUI",
           RfcEmisor = "ABC010101XYZ",
           Uuid = "C3D4E08E-12F3-4B0C-8A4D-78E6B8E47113"
       };

       var resultado = await CancelarSPAsync(request);

       if (resultado?.Status == "success")
       {
           Console.WriteLine("¡Cancelación de CFDI SP Exitosa!");
           Console.WriteLine($"Mensaje: {resultado.Message}");
           Console.WriteLine("Acuse:");
           Console.WriteLine(resultado.Data);
       }
       else
       {
           Console.WriteLine($"Error: {resultado?.Code} - {resultado?.Message}");
       }
   }
  ```
  {{< /tab >}}

  {{< tab >}}
  ### Herramienta wsimport
  Java incluye la herramienta wsimport en el JDK para generar las clases cliente a partir de un WSDL.

  Ejecuta el siguiente comando en tu terminal para el ambiente de DESARROLLO:

  ```
  wsimport -keep -p com.facturaloplus.cliente https://dev.facturaloplus.com/ws/servicio.do?wsdl
  ```

  -keep: Conserva los archivos fuente .java generados.

  -p: Especifica el paquete (package) donde se guardarán las clases.

  ### Implementación

  ```java
  import java.util.concurrent.CompletableFuture;

  public class CancelarSPService {

    // ... (Definición de ExecutorService, Logger, etc.)

    public CompletableFuture<RespuestaCancelar> cancelarSPAsync(String apiKey, String rfcEmisor, String uuid) {
        return CompletableFuture.supplyAsync(() -> {
            try {
                ServicioTimbradoWS service = new ServicioTimbradoWS();
                ServicioTimbradoWSPortType port = service.getServicioTimbradoWSPort();
                
                Respuesta response = port.cancelarSP(apiKey, rfcEmisor, uuid);

                RespuestaCancelar resultado = new RespuestaCancelar();
                resultado.setCode(response.getCode());
                resultado.setMessage(response.getMessage());
                resultado.setData(response.getData());
                resultado.setStatus(response.getStatus());
                return resultado;

            } catch (Exception ex) {
                throw new RuntimeException("Error al cancelar CFDI de Sector Primario", ex);
            }
        }, executor);
    }

    // Ejemplo de uso
    public static void main(String[] args) {
        CancelarSPService service = new CancelarSPService();
        String apiKey = "TU_API_KEY_AQUI";
        String rfcEmisor = "ABC010101XYZ";
        String uuid = "C3D4E08E-12F3-4B0C-8A4D-78E6B8E47113";

        service.cancelarSPAsync(apiKey, rfcEmisor, uuid)
            .whenComplete((resultado, ex) -> {
                if (ex != null) {
                    System.err.println("Error: " + ex.getMessage());
                } else if ("success".equals(resultado.getStatus())) {
                    System.out.println("¡Cancelación de CFDI SP Exitosa!");
                    System.out.println("Acuse: " + resultado.getData());
                } else {
                    System.err.println("Error: " + resultado.getCode() + " - " + resultado.getMessage());
                }
                service.shutdown();
            }).join();
    }
  }
  ```
  {{< /tab >}}

  {{< tab >}}
  ### Herramienta Zeep
  Para interactuar con servicios SOAP en Python, la librería zeep es una excelente opción. Proporciona una interfaz limpia y moderna.

  Instala la librería usando pip:
  ```
  pip install zeep
  ```

  ### Implementación
  El siguiente código muestra una implementación robusta utilizando zeep y asyncio para realizar llamadas asíncronas al servicio web.
  ```python
  import asyncio
  from zeep.asyncio import AsyncClient

  class CancelarSPService:
      def __init__(self, wsdl_url: str):
          self.wsdl_url = wsdl_url
          self.async_client = AsyncClient(self.wsdl_url)

      async def cancelar_sp_async(self, **kwargs) -> dict:
          response = await self.async_client.service.cancelarSP(**kwargs)
          return response

  async def main():
      service = CancelarSPService("https://dev.facturaloplus.com/ws/servicio.do?wsdl")
      
      params = {
          "apikey": "TU_API_KEY_AQUI",
          "rfcEmisor": "ABC010101XYZ",
          "uuid": "C3D4E08E-12F3-4B0C-8A4D-78E6B8E47113"
      }

      resultado = await service.cancelar_sp_async(**params)

      if resultado and resultado['status'] == 'success':
          print("¡Cancelación de CFDI SP Exitosa!")
          print(f"Mensaje: {resultado['message']}")
          print(f"Acuse: {resultado['data']}")
      else:
          print(f"Error: {resultado['code']} - {resultado['message']}")

  if __name__ == "__main__":
      asyncio.run(main())
  ```
  {{< /tab >}}

  {{< tab >}}
  ### Herramienta SoapClient
  PHP tiene soporte nativo para SOAP a través de la extensión SOAP. Asegúrate de que la extensión php-soap esté habilitada en tu archivo php.ini.

  ### Implementación 
  El siguiente código muestra una implementación orientada a objetos para consumir el servicio de timbrado.
  ```php
  <?php
  class CancelarSPService {
      private string $wsdlUrl;

      public function __construct(string $wsdlUrl) {
          $this->wsdlUrl = $wsdlUrl;
      }

      public function cancelarSP(array $params): ?object {
          try {
              $soapClient = new SoapClient($this->wsdlUrl, ['trace' => 1, 'exceptions' => true]);
              $response = $soapClient->cancelarSP($params);
              return $response->return ?? null;
          } catch (Exception $e) {
              echo "Error: " . $e->getMessage();
              return null;
          }
      }
  }

  // Ejemplo de uso
  $service = new CancelarSPService("https://dev.facturaloplus.com/ws/servicio.do?wsdl");
  
  $params = [
      'apikey'      => "TU_API_KEY_AQUI",
      'rfcEmisor'   => "ABC010101XYZ",
      'uuid'        => "C3D4E08E-12F3-4B0C-8A4D-78E6B8E47113"
  ];

  $resultado = $service->cancelarSP($params);

  header('Content-Type: text/plain');
  if ($resultado && $resultado->status === 'success') {
      echo "¡Cancelación de CFDI SP Exitosa!\n";
      echo "Mensaje: {$resultado->message}\n";
      echo "Acuse: {$resultado->data}\n";
  } else {
      echo "Error: {$resultado->code} - {$resultado->message}\n";
  }
  ?>
  ```
  {{< /tab >}}

{{< /tabs >}}

#### Respuesta (Response)

El campo `data` contendrá el acuse de cancelación del SAT.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<SOAP-ENV:Envelope ...>
   <SOAP-ENV:Body>
      <ns1:cancelarSPResponse xmlns:ns1="urn:ws_api">
         <return xsi:type="tns:RespuestaCancelar">
            <code xsi:type="xsd:string">200</code>
            <message xsi:type="xsd:string">Solicitud de cancelación recibida</message>
            <data xsi:type="xsd:string"><![CDATA[<Acuse ... />]]></data>
            <status xsi:type="xsd:string">success</status>
         </return>
      </ns1:cancelarSPResponse>
   </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

##### Códigos de respuesta
{{< codigos-cancelacion >}}
