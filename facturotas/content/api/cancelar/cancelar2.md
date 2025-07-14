---
title: Cancelar CFDI
---

Esta sección detalla la operación para solicitar la cancelación de un CFDI utilizando los Certificados de Sello Digital (CSD) del emisor.

### Descripción de la Operación

Esta operación permite realizar la solicitud de cancelación de un CFDI (versión 3.3 o 4.0) directamente ante el SAT. A diferencia de otros métodos, esta operación requiere el envío de los archivos del Certificado de Sello Digital (.key y .cer) y su contraseña, además de la información del comprobante a cancelar.

Se debe especificar uno de los motivos de cancelación definidos por el SAT. Opcionalmente, si el motivo es "Comprobante emitido con errores con relación" (clave 01), se debe proporcionar el folio fiscal (UUID) del CFDI que sustituye al cancelado.

### Parámetros de Entrada (Input)

| Parámetro         | Tipo de Dato | Descripción                                                                                                                               |
| :---------------- | :----------- | :---------------------------------------------------------------------------------------------------------------------------------------- |
| `apikey`          | `string`     | Credencial de acceso al servicio ([Solicita aquí](../../../#Requisitos)).                                                                    |
| `keyCSD`          | `string`     | Contenido del archivo de la llave privada (`.key`) del emisor, codificado en Base64. [Conversor Base 64](../../../herramientas/convertidorBase64) |
| `cerCSD`          | `string`     | Contenido del archivo del certificado de llave pública (`.cer`) del emisor, codificado en Base64. [Conversor Base 64](../../../herramientas/convertidorBase64) |
| `passCSD`         | `string`     | Contraseña de la llave privada (CSD) del emisor.                                                                                          |
| `uuid`            | `string`     | Folio Fiscal (UUID) del CFDI que se desea cancelar.                                                                                       |
| `rfcEmisor`       | `string`     | RFC del contribuyente emisor del CFDI.                                                                                                    |
| `rfcReceptor`     | `string`     | RFC del contribuyente receptor del CFDI.                                                                                                  |
| `total`           | `double`     | Monto total exacto del CFDI a cancelar.                                                                                                   |
| `motivo`          | `string`     | Clave del motivo de la cancelación. Valores válidos: `01`, `02`, `03`, `04`.                                                              |
| `folioSustitucion`| `string`     | (Opcional) Folio Fiscal (UUID) del CFDI que sustituye al comprobante cancelado. En otros casos, enviar vacío. |

### Parámetros de Salida (Output) - RespuestaCancelar

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

   public class CancelarRequest
   {
        public string Apikey { get; set; }
        public string KeyCSD { get; set; }
        public string CerCSD { get; set; }
        public string PassCSD { get; set; }
        public string Uuid { get; set; }
        public string RfcEmisor { get; set; }
        public string RfcReceptor { get; set; }
        public double Total { get; set; }
        public string Motivo { get; set; }
        public string FolioSustitucion { get; set; }
   }

   public async Task<RespuestaCancelar> CancelarCfdiAsync(CancelarRequest request)
    {
        using var client = new ServicioTimbradoWSPortTypeClient("ServicioTimbradoWSPort");
        try
        {
            var response = await client.cancelar2Async(
                request.Apikey, request.KeyCSD, request.CerCSD, request.PassCSD, 
                request.Uuid, request.RfcEmisor, request.RfcReceptor, 
                request.Total, request.Motivo, request.FolioSustitucion
            );
            return new RespuestaCancelar { Code = response.code, Message = response.message, Data = response.data, Status = response.status };
        }
        catch (Exception ex)
        {
           Console.WriteLine($"Error al cancelar CFDI: {ex.Message}");
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
   public async Task EjemploUsoCancelarAsync()
   {
       var request = new CancelarRequest
       {
           Apikey = "TU_API_KEY_AQUI",
           KeyCSD = Convert.ToBase64String(File.ReadAllBytes("ruta/al/csd.key")),
           CerCSD = Convert.ToBase64String(File.ReadAllBytes("ruta/al/csd.cer")),
           PassCSD = "tu_contraseña",
           Uuid = "5FD4E09E-52F4-4A0E-8E4D-39E6B8E47168",
           RfcEmisor = "ABC010101XYZ",
           RfcReceptor = "XAXX010101000",
           Total = 116.00,
           Motivo = "02", // 02: Comprobante emitido con errores sin relación.
           FolioSustitucion = ""
       };

       var resultado = await CancelarCfdiAsync(request);

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

  public class CancelarService {

    // ... (Definición de ExecutorService, Logger, etc.)

    public CompletableFuture<RespuestaCancelar> cancelarAsync(String apiKey, String keyCSD, String cerCSD, String passCSD, String uuid, String rfcEmisor, String rfcReceptor, double total, String motivo, String folioSustitucion) {
        return CompletableFuture.supplyAsync(() -> {
            try {
                ServicioTimbradoWS service = new ServicioTimbradoWS();
                ServicioTimbradoWSPortType port = service.getServicioTimbradoWSPort();
                
                Respuesta response = port.cancelar2(apiKey, keyCSD, cerCSD, passCSD, uuid, rfcEmisor, rfcReceptor, total, motivo, folioSustitucion);

                RespuestaCancelar resultado = new RespuestaCancelar();
                resultado.setCode(response.getCode());
                resultado.setMessage(response.getMessage());
                resultado.setData(response.getData());
                resultado.setStatus(response.getStatus());
                return resultado;

            } catch (Exception ex) {
                throw new RuntimeException("Error al cancelar CFDI", ex);
            }
        }, executor);
    }

    // Ejemplo de uso
    public static void main(String[] args) {
        CancelarService service = new CancelarService();
        String apiKey = "TU_API_KEY_AQUI";
        try {
            String keyCSD = Base64.getEncoder().encodeToString(Files.readAllBytes(Paths.get("ruta/al/csd.key")));
            String cerCSD = Base64.getEncoder().encodeToString(Files.readAllBytes(Paths.get("ruta/al/csd.cer")));
            String passCSD = "tu_contraseña";
            String uuid = "5FD4E09E-52F4-4A0E-8E4D-39E6B8E47168";
            String rfcEmisor = "ABC010101XYZ";
            String rfcReceptor = "XAXX010101000";
            double total = 116.00;
            String motivo = "02";
            String folioSustitucion = "";

            service.cancelarAsync(apiKey, keyCSD, cerCSD, passCSD, uuid, rfcEmisor, rfcReceptor, total, motivo, folioSustitucion)
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

  class CancelarService:
      def __init__(self, wsdl_url: str):
          self.wsdl_url = wsdl_url
          self.async_client = AsyncClient(self.wsdl_url)

      async def cancelar_async(self, **kwargs) -> dict:
          response = await self.async_client.service.cancelar2(**kwargs)
          return response

  async def main():
      service = CancelarService("https://dev.facturaloplus.com/ws/servicio.do?wsdl")
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
          "uuid": "5FD4E09E-52F4-4A0E-8E4D-39E6B8E47168",
          "rfcEmisor": "ABC010101XYZ",
          "rfcReceptor": "XAXX010101000",
          "total": 116.00,
          "motivo": "02",
          "folioSustitucion": ""
      }

      resultado = await service.cancelar_async(**params)

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
  class CancelarService {
      private string $wsdlUrl;

      public function __construct(string $wsdlUrl) {
          $this->wsdlUrl = $wsdlUrl;
      }

      public function cancelar(array $params): ?object {
          try {
              $soapClient = new SoapClient($this->wsdlUrl, ['trace' => 1, 'exceptions' => true]);
              $response = $soapClient->cancelar2($params);
              return $response->return ?? null;
          } catch (Exception $e) {
              echo "Error: " . $e->getMessage();
              return null;
          }
      }
  }

  // Ejemplo de uso
  $service = new CancelarService("https://dev.facturaloplus.com/ws/servicio.do?wsdl");
  $apiKey = "TU_API_KEY_AQUI";
  
  $key_path = "ruta/al/csd.key";
  $cer_path = "ruta/al/csd.cer";

  $params = [
      'apikey'          => $apiKey,
      'keyCSD'          => base64_encode(file_get_contents($key_path)),
      'cerCSD'          => base64_encode(file_get_contents($cer_path)),
      'passCSD'         => "tu_contraseña",
      'uuid'            => "5FD4E09E-52F4-4A0E-8E4D-39E6B8E47168",
      'rfcEmisor'       => "ABC010101XYZ",
      'rfcReceptor'     => "XAXX010101000",
      'total'           => 116.00,
      'motivo'          => "02",
      'folioSustitucion'=> ""
  ];

  $resultado = $service->cancelar($params);

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
      <ns1:cancelar2Response xmlns:ns1="urn:ws_api">
         <return xsi:type="tns:RespuestaCancelar">
            <code xsi:type="xsd:string">200</code>
            <message xsi:type="xsd:string">Solicitud de cancelación recibida</message>
            <data xsi:type="xsd:string"><![CDATA[<Acuse ... />]]></data>
            <status xsi:type="xsd:string">success</status>
         </return>
      </ns1:cancelar2Response>
   </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

##### Códigos de respuesta
{{< codigos-cancelacion >}}
