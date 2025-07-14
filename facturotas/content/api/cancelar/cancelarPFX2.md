---
title: Cancelar Archivo PFX
---

Esta sección detalla la operación para solicitar la cancelación de un CFDI utilizando un archivo PFX que contiene el Certificado de Sello Digital (CSD) del emisor.

### Descripción de la Operación

Esta operación permite realizar la solicitud de cancelación de un CFDI (versión 3.3 o 4.0) directamente ante el SAT. A diferencia de otros métodos, esta operación simplifica el envío de credenciales al requerir un único archivo `.pfx` que contiene tanto el certificado como la llave privada, protegido por una contraseña.

Se debe especificar uno de los motivos de cancelación definidos por el SAT. Opcionalmente, se debe proporcionar el folio fiscal (UUID) del CFDI que sustituye al cancelado.

### Parámetros de Entrada (Input)

| Parámetro         | Tipo de Dato | Descripción                                                                                                                               |
| :---------------- | :----------- | :---------------------------------------------------------------------------------------------------------------------------------------- |
| `apikey`          | `string`     | Credencial de acceso al servicio ([Solicita aquí](../../../#Requisitos)).                                                                    |
| `filePFX`         | `string`     | Contenido del archivo PFX (`.pfx`), codificado en Base64. [Conversor Base 64](../../../herramientas/convertidorBase64) |
| `passPFX`         | `string`     | Contraseña del archivo PFX.                                                                                                               |
| `uuid`            | `string`     | Folio Fiscal (UUID) del CFDI que se desea cancelar.                                                                                       |
| `rfcEmisor`       | `string`     | RFC del contribuyente emisor del CFDI.                                                                                                    |
| `rfcReceptor`     | `string`     | RFC del contribuyente receptor del CFDI.                                                                                                  |
| `total`           | `double`     | Monto total exacto del CFDI a cancelar.                                                                                                   |
| `motivo`          | `string`     | Clave del motivo de la cancelación. Valores válidos: `01`, `02`, `03`, `04`.                                                              |
| `folioSustitucion`| `string`     | (Opcional) Folio Fiscal (UUID) del CFDI que sustituye al comprobante cancelado. En otros casos, enviar vacío. |

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

   public class CancelarPFXRequest
   {
        public string Apikey { get; set; }
        public string FilePFX { get; set; }
        public string PassPFX { get; set; }
        public string Uuid { get; set; }
        public string RfcEmisor { get; set; }
        public string RfcReceptor { get; set; }
        public double Total { get; set; }
        public string Motivo { get; set; }
        public string FolioSustitucion { get; set; }
   }

   public async Task<RespuestaCancelar> CancelarCfdiPfxAsync(CancelarPFXRequest request)
    {
        using var client = new ServicioTimbradoWSPortTypeClient("ServicioTimbradoWSPort");
        try
        {
            var response = await client.cancelarPFX2Async(
                request.Apikey, request.FilePFX, request.PassPFX, 
                request.Uuid, request.RfcEmisor, request.RfcReceptor, 
                request.Total, request.Motivo, request.FolioSustitucion
            );
            return new RespuestaCancelar { Code = response.code, Message = response.message, Data = response.data, Status = response.status };
        }
        catch (Exception ex)
        {
           Console.WriteLine($"Error al cancelar CFDI con PFX: {ex.Message}");
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
   public async Task EjemploUsoCancelarPfxAsync()
   {
       var request = new CancelarPFXRequest
       {
           Apikey = "TU_API_KEY_AQUI",
           FilePFX = Convert.ToBase64String(File.ReadAllBytes("ruta/al/certificado.pfx")),
           PassPFX = "tu_contraseña_pfx",
           Uuid = "5FD4E09E-52F4-4A0E-8E4D-39E6B8E47168",
           RfcEmisor = "ABC010101XYZ",
           RfcReceptor = "XAXX010101000",
           Total = 116.00,
           Motivo = "01",
           FolioSustitucion = "8E4D39E6-B8E4-4A0E-52F4-5FD4E09E7168"
       };

       var resultado = await CancelarCfdiPfxAsync(request);

       if (resultado?.Status == "success")
       {
           Console.WriteLine("¡Cancelación Exitosa!");
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

  public class CancelarPfxService {

    // ... (Definición de ExecutorService, Logger, etc.)

    public CompletableFuture<RespuestaCancelar> cancelarPfxAsync(String apiKey, String filePFX, String passPFX, String uuid, String rfcEmisor, String rfcReceptor, double total, String motivo, String folioSustitucion) {
        return CompletableFuture.supplyAsync(() -> {
            try {
                ServicioTimbradoWS service = new ServicioTimbradoWS();
                ServicioTimbradoWSPortType port = service.getServicioTimbradoWSPort();
                
                Respuesta response = port.cancelarPFX2(apiKey, filePFX, passPFX, uuid, rfcEmisor, rfcReceptor, total, motivo, folioSustitucion);

                RespuestaCancelar resultado = new RespuestaCancelar();
                resultado.setCode(response.getCode());
                resultado.setMessage(response.getMessage());
                resultado.setData(response.getData());
                resultado.setStatus(response.getStatus());
                return resultado;

            } catch (Exception ex) {
                throw new RuntimeException("Error al cancelar CFDI con PFX", ex);
            }
        }, executor);
    }

    // Ejemplo de uso
    public static void main(String[] args) {
        CancelarPfxService service = new CancelarPfxService();
        String apiKey = "TU_API_KEY_AQUI";
        try {
            String filePFX = Base64.getEncoder().encodeToString(Files.readAllBytes(Paths.get("ruta/al/certificado.pfx")));
            String passPFX = "tu_contraseña_pfx";
            String uuid = "5FD4E09E-52F4-4A0E-8E4D-39E6B8E47168";
            String rfcEmisor = "ABC010101XYZ";
            String rfcReceptor = "XAXX010101000";
            double total = 116.00;
            String motivo = "01";
            String folioSustitucion = "8E4D39E6-B8E4-4A0E-52F4-5FD4E09E7168";

            service.cancelarPfxAsync(apiKey, filePFX, passPFX, uuid, rfcEmisor, rfcReceptor, total, motivo, folioSustitucion)
                .whenComplete((resultado, ex) -> {
                    if (ex != null) {
                        System.err.println("Error: " + ex.getMessage());
                    } else if ("success".equals(resultado.getStatus())) {
                        System.out.println("¡Cancelación Exitosa!");
                        System.out.println("Acuse: " + resultado.getData());
                    } else {
                        System.err.println("Error: " + resultado.getCode() + " - " + resultado.getMessage());
                    }
                    service.shutdown();
                }).join();
        } catch (IOException e) {
            System.err.println("No se pudo leer el archivo PFX.");
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

  class CancelarPfxService:
      def __init__(self, wsdl_url: str):
          self.wsdl_url = wsdl_url
          self.async_client = AsyncClient(self.wsdl_url)

      async def cancelar_pfx_async(self, **kwargs) -> dict:
          response = await self.async_client.service.cancelarPFX2(**kwargs)
          return response

  async def main():
      service = CancelarPfxService("https://dev.facturaloplus.com/ws/servicio.do?wsdl")
      api_key = "TU_API_KEY_AQUI"
      
      with open("ruta/al/certificado.pfx", "rb") as pfx_file:
          pfx_b64 = base64.b64encode(pfx_file.read()).decode('utf-8')

      params = {
          "apikey": api_key,
          "filePFX": pfx_b64,
          "passPFX": "tu_contraseña_pfx",
          "uuid": "5FD4E09E-52F4-4A0E-8E4D-39E6B8E47168",
          "rfcEmisor": "ABC010101XYZ",
          "rfcReceptor": "XAXX010101000",
          "total": 116.00,
          "motivo": "01",
          "folioSustitucion": "8E4D39E6-B8E4-4A0E-52F4-5FD4E09E7168"
      }

      resultado = await service.cancelar_pfx_async(**params)

      if resultado and resultado['status'] == 'success':
          print("¡Cancelación Exitosa!")
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
  class CancelarPfxService {
      private string $wsdlUrl;

      public function __construct(string $wsdlUrl) {
          $this->wsdlUrl = $wsdlUrl;
      }

      public function cancelarPfx(array $params): ?object {
          try {
              $soapClient = new SoapClient($this->wsdlUrl, ['trace' => 1, 'exceptions' => true]);
              $response = $soapClient->cancelarPFX2($params);
              return $response->return ?? null;
          } catch (Exception $e) {
              echo "Error: " . $e->getMessage();
              return null;
          }
      }
  }

  // Ejemplo de uso
  $service = new CancelarPfxService("https://dev.facturaloplus.com/ws/servicio.do?wsdl");
  $apiKey = "TU_API_KEY_AQUI";
  
  $pfx_path = "ruta/al/certificado.pfx";

  $params = [
      'apikey'          => $apiKey,
      'filePFX'         => base64_encode(file_get_contents($pfx_path)),
      'passPFX'         => "tu_contraseña_pfx",
      'uuid'            => "5FD4E09E-52F4-4A0E-8E4D-39E6B8E47168",
      'rfcEmisor'       => "ABC010101XYZ",
      'rfcReceptor'     => "XAXX010101000",
      'total'           => 116.00,
      'motivo'          => "01",
      'folioSustitucion'=> "8E4D39E6-B8E4-4A0E-52F4-5FD4E09E7168"
  ];

  $resultado = $service->cancelarPfx($params);

  header('Content-Type: text/plain');
  if ($resultado && $resultado->status === 'success') {
      echo "¡Cancelación Exitosa!\n";
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
      <ns1:cancelarPFX2Response xmlns:ns1="urn:ws_api">
         <return xsi:type="tns:RespuestaCancelar">
            <code xsi:type="xsd:string">200</code>
            <message xsi:type="xsd:string">Solicitud de cancelación recibida</message>
            <data xsi:type="xsd:string"><![CDATA[<Acuse ... />]]></data>
            <status xsi:type="xsd:string">success</status>
         </return>
      </ns1:cancelarPFX2Response>
   </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

##### Códigos de respuesta
{{< codigos-cancelacion >}}
