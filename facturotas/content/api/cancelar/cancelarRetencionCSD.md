---
title: Cancelar Retenciones
---

Esta sección detalla la operación para solicitar la cancelación de un CFDI de retenciones e información de pagos, utilizando los Certificados de Sello Digital (CSD) del emisor.

### Descripción de la Operación

Esta operación permite realizar la solicitud de cancelación de un CFDI de retenciones directamente ante el SAT. Para ello, se requiere el envío de los archivos del Certificado de Sello Digital (`.key` y `.cer`) y su contraseña, junto con el folio fiscal (UUID) del comprobante a cancelar y el RFC del emisor.

A diferencia de la cancelación de CFDI de ingresos, la cancelación de retenciones no requiere motivo, RFC receptor ni total.

### Parámetros de Entrada (Input)

| Parámetro | Tipo de Dato | Descripción                                                                                                                               |
| :-------- | :----------- | :---------------------------------------------------------------------------------------------------------------------------------------- |
| `apikey`    | `string`     | Credencial de acceso al servicio ([Solicita aquí](../../../#Requisitos)).                                                                    |
| `keyCSD`    | `string`     | Contenido del archivo de la llave privada (`.key`) del emisor, codificado en Base64. [Conversor Base 64](../../../herramientas/convertidorBase64) |
| `cerCSD`    | `string`     | Contenido del archivo del certificado de llave pública (`.cer`) del emisor, codificado en Base64. [Conversor Base 64](../../../herramientas/convertidorBase64) |
| `passCSD`   | `string`     | Contraseña de la llave privada (CSD) del emisor.                                                                                          |
| `rfcEmisor` | `string`     | RFC del contribuyente emisor del CFDI de retenciones.                                                                                     |
| `uuid`      | `string`     | Folio Fiscal (UUID) del CFDI de retenciones que se desea cancelar.                                                                        |

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
  using System.IO;
  using System.Threading.Tasks;

   public class CancelarRetencionRequest
   {
        public string Apikey { get; set; }
        public string KeyCSD { get; set; }
        public string CerCSD { get; set; }
        public string PassCSD { get; set; }
        public string RfcEmisor { get; set; }
        public string Uuid { get; set; }
   }

   public async Task<RespuestaCancelar> CancelarRetencionAsync(CancelarRetencionRequest request)
    {
        using var client = new ServicioTimbradoWSPortTypeClient("ServicioTimbradoWSPort");
        try
        {            var response = await client.cancelarRetencionCSDAsync(
                request.Apikey, request.KeyCSD, request.CerCSD, request.PassCSD, 
                request.RfcEmisor, request.Uuid
            );
            return new RespuestaCancelar { Code = response.code, Message = response.message, Data = response.data, Status = response.status };
        }
        catch (Exception ex)
        {
           Console.WriteLine($"Error al cancelar CFDI de retención: {ex.Message}");
           throw;
        }
    }

   // Ejemplo de uso
   public async Task EjemploUsoCancelarRetencionAsync()
   {
       var request = new CancelarRetencionRequest
       {
           Apikey = "TU_API_KEY_AQUI",
           KeyCSD = Convert.ToBase64String(File.ReadAllBytes("ruta/al/csd.key")),
           CerCSD = Convert.ToBase64String(File.ReadAllBytes("ruta/al/csd.cer")),
           PassCSD = "tu_contraseña",
           RfcEmisor = "ABC010101XYZ",
           Uuid = "A2B4E08E-12F3-4B0C-8A4D-78E6B8E47112"
       };

       var resultado = await CancelarRetencionAsync(request);

       if (resultado?.Status == "success")
       {
           Console.WriteLine("¡Cancelación de Retención Exitosa!");
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
  import java.nio.file.Files;
  import java.nio.file.Paths;
  import java.util.Base64;
  import java.util.concurrent.CompletableFuture;

  public class CancelarRetencionService {

    // ... (Definición de ExecutorService, Logger, etc.)

    public CompletableFuture<RespuestaCancelar> cancelarRetencionAsync(String apiKey, String keyCSD, String cerCSD, String passCSD, String rfcEmisor, String uuid) {
        return CompletableFuture.supplyAsync(() -> {
            try {
                ServicioTimbradoWS service = new ServicioTimbradoWS();
                ServicioTimbradoWSPortType port = service.getServicioTimbradoWSPort();
                
                Respuesta response = port.cancelarRetencionCSD(apiKey, keyCSD, cerCSD, passCSD, rfcEmisor, uuid);

                RespuestaCancelar resultado = new RespuestaCancelar();
                resultado.setCode(response.getCode());
                resultado.setMessage(response.getMessage());
                resultado.setData(response.getData());
                resultado.setStatus(response.getStatus());
                return resultado;

            } catch (Exception ex) {
                throw new RuntimeException("Error al cancelar CFDI de retención", ex);
            }
        }, executor);
    }

    // Ejemplo de uso
    public static void main(String[] args) {
        CancelarRetencionService service = new CancelarRetencionService();
        String apiKey = "TU_API_KEY_AQUI";
        try {
            String keyCSD = Base64.getEncoder().encodeToString(Files.readAllBytes(Paths.get("ruta/al/csd.key")));
            String cerCSD = Base64.getEncoder().encodeToString(Files.readAllBytes(Paths.get("ruta/al/csd.cer")));
            String passCSD = "tu_contraseña";
            String rfcEmisor = "ABC010101XYZ";
            String uuid = "A2B4E08E-12F3-4B0C-8A4D-78E6B8E47112";

            service.cancelarRetencionAsync(apiKey, keyCSD, cerCSD, passCSD, rfcEmisor, uuid)
                .whenComplete((resultado, ex) -> {
                    if (ex != null) {
                        System.err.println("Error: " + ex.getMessage());
                    } else if ("success".equals(resultado.getStatus())) {
                        System.out.println("¡Cancelación de Retención Exitosa!");
                        System.out.println("Acuse: " + resultado.getData());
                    } else {
                        System.err.println("Error: " + resultado.getCode() + " - " + resultado.getMessage());
                    }
                    service.shutdown();
                }).join();
        } catch (IOException e) {
            System.err.println("No se pudieron leer los archivos CSD.");
        }
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
  import base64
  from zeep.asyncio import AsyncClient

  class CancelarRetencionService:
      def __init__(self, wsdl_url: str):
          self.wsdl_url = wsdl_url
          self.async_client = AsyncClient(self.wsdl_url)

      async def cancelar_retencion_async(self, **kwargs) -> dict:
          response = await self.async_client.service.cancelarRetencionCSD(**kwargs)
          return response

  async def main():
      service = CancelarRetencionService("https://dev.facturaloplus.com/ws/servicio.do?wsdl")
      api_key = "TU_API_KEY_AQUI"
      
      with open("ruta/al/csd.key", "rb") as key_file:
          key_csd_b64 = base64.b64encode(key_file.read()).decode('utf-8')
      with open("ruta/al/csd.cer", "rb") as cer_file:
          cer_csd_b64 = base64.b64encode(cer_file.read()).decode('utf-8')

      params = {
          "apikey": api_key,
          "keyCSD": key_csd_b64,
          "cerCSD": cer_csd_b64,
          "passCSD": "tu_contraseña",
          "rfcEmisor": "ABC010101XYZ",
          "uuid": "A2B4E08E-12F3-4B0C-8A4D-78E6B8E47112"
      }

      resultado = await service.cancelar_retencion_async(**params)

      if resultado and resultado['status'] == 'success':
          print("¡Cancelación de Retención Exitosa!")
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
  class CancelarRetencionService {
      private string $wsdlUrl;

      public function __construct(string $wsdlUrl) {
          $this->wsdlUrl = $wsdlUrl;
      }

      public function cancelarRetencion(array $params): ?object {
          try {
              $soapClient = new SoapClient($this->wsdlUrl, ['trace' => 1, 'exceptions' => true]);
              $response = $soapClient->cancelarRetencionCSD($params);
              return $response->return ?? null;
          } catch (Exception $e) {
              echo "Error: " . $e->getMessage();
              return null;
          }
      }
  }

  // Ejemplo de uso
  $service = new CancelarRetencionService("https://dev.facturaloplus.com/ws/servicio.do?wsdl");
  $apiKey = "TU_API_KEY_AQUI";
  
  $key_path = "ruta/al/csd.key";
  $cer_path = "ruta/al/csd.cer";

  $params = [
      'apikey'      => $apiKey,
      'keyCSD'      => base64_encode(file_get_contents($key_path)),
      'cerCSD'      => base64_encode(file_get_contents($cer_path)),
      'passCSD'     => "tu_contraseña",
      'rfcEmisor'   => "ABC010101XYZ",
      'uuid'        => "A2B4E08E-12F3-4B0C-8A4D-78E6B8E47112"
  ];

  $resultado = $service->cancelarRetencion($params);

  header('Content-Type: text/plain');
  if ($resultado && $resultado->status === 'success') {
      echo "¡Cancelación de Retención Exitosa!\n";
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
      <ns1:cancelarRetencionCSDResponse xmlns:ns1="urn:ws_api">
         <return xsi:type="tns:RespuestaCancelar">
            <code xsi:type="xsd:string">200</code>
            <message xsi:type="xsd:string">Solicitud de cancelación recibida</message>
            <data xsi:type="xsd:string"><![CDATA[<Acuse ... />]]></data>
            <status xsi:type="xsd:string">success</status>
         </return>
      </ns1:cancelarRetencionCSDResponse>
   </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

##### Códigos de respuesta
{{< codigos-cancelacion-retenciones >}}
