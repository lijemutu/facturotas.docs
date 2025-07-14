# timbrarConSello

Esta sección detalla la operación del servicio de timbrado para un CFDI que aún no ha sido sellado.

### Descripción de la Operación

Esta operación está diseñada para tomar un Comprobante Fiscal Digital por Internet (CFDI) en formato XML al que le falta el atributo `sello`. El servicio utiliza la llave privada (`.key`) proporcionada para generar el sello digital, lo integra en el XML, y posteriormente realiza el timbrado ante el SAT. Como resultado, devuelve el XML completo, timbrado y sellado.

Es ideal para escenarios donde el sistema que genera el CFDI no tiene la capacidad de crear el sello digital, delegando esta responsabilidad al servicio de timbrado.

### Parámetros de Entrada (Input)

| Parámetro | Tipo de Dato | Descripción                                                                                                |
| :-------- | :----------- | :--------------------------------------------------------------------------------------------------------- |
| `apikey`  | `string`     | Credencial de acceso al servicio ([Solicita aquí](../../../#Requisitos)).                                      |
| `xmlCFDI` | `string`     | Contenido del documento XML del CFDI (v3.3 o v4.0) **sin el atributo `sello`**. |
| `keypem`  | `string`     | Contenido del archivo de la llave privada (`.key`) en formato PEM, utilizado para generar el sello del CFDI. |

### Parámetros de Salida (Output) - RespuestaTimbrado

La respuesta de esta operación es idéntica a la de la operación `timbrar`.

| Atributo  | Tipo de Dato | Descripción                             |
| :-------- | :----------- | :-------------------------------------- |
| `code`    | `string`     | Código de respuesta de la operación.    |
| `message` | `string`     | Mensaje detallado de la respuesta.      |
| `data`    | `string`     | XML del CFDI sellado y timbrado en caso de éxito. |

### Ejemplo de Código

A continuación se presenta un ejemplo de cómo construir la solicitud y procesar la respuesta.

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

   /// <summary>
   /// Genera un XML de ejemplo para un CFDI 4.0 sin el atributo 'sello'.
   /// </summary>
   private static string GetXmlCfdiSinSello() => """ 
        <?xml version=\"1.0\" encoding=\"UTF-8\"?>
        <cfdi:Comprobante
            xmlns:cfdi=\"http://www.sat.gob.mx/cfd/4\"
            xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\"
            xsi:schemaLocation=\"http://www.sat.gob.mx/cfd/4 cfdv40.xsd\" 
            Version=\"4.0\"
            Serie=\"F\"
            Folio=\"123\"
            Fecha=\"2025-07-05T12:00:00\"
            SubTotal=\"100.00\"
            Moneda=\"MXN\"
            Total=\"116.00\"
            TipoDeComprobante=\"I\"
            Exportacion=\"01\"
            MetodoPago=\"PUE\"
            FormaPago=\"01\"
            NoCertificado=\"30001000000300023708\"
            Certificado=\"MIIC...\"
            Sello=\"\"
            LugarExpedicion=\"64000\">
            <cfdi:Emisor Rfc=\"ABC010101XYZ\" Nombre=\"Empresa Ejemplo\" RegimenFiscal=\"601\"/>
            <cfdi:Receptor Rfc=\"XAXX010101000\" Nombre=\"Publico en General\" DomicilioFiscalReceptor=\"64000\" RegimenFiscalReceptor=\"616\" UsoCFDI=\"G03\"/>
            <cfdi:Conceptos>
                <cfdi:Concepto ObjetoImp=\"02\" ClaveProdServ=\"01010101\" ClaveUnidad=\"ACT\" Cantidad=\"1\" Descripcion=\"Servicio\" ValorUnitario=\"100.00\" Importe=\"100.00\">
                    <cfdi:Impuestos>
                        <cfdi:Traslados>
                            <cfdi:Traslado Base=\"100.00\" Impuesto=\"002\" TipoFactor=\"Tasa\" TasaOCuota=\"0.160000\" Importe=\"16.00\"/>
                        </cfdi:Traslados>
                    </cfdi:Impuestos>
                </cfdi:Concepto>
            </cfdi:Conceptos>
            <cfdi:Impuestos TotalImpuestosTrasladados=\"16.00\">
               <cfdi:Traslados>
                     <cfdi:Traslado Base=\"100.00\" Impuesto=\"002\" TipoFactor=\"Tasa\" TasaOCuota=\"0.160000\" Importe=\"16.00\"/>
               </cfdi:Traslados>
            </cfdi:Impuestos>
        </cfdi:Comprobante>
        """;

   public async Task<RespuestaTimbrado> TimbrarConSelloAsync(string apiKey, string xmlCfdi, string keyPem)
    {
        using var client = new ServicioTimbradoWSPortTypeClient("ServicioTimbradoWSPort");
        try
        {
            var response = await client.timbrarConSelloAsync(apiKey, xmlCfdi, keyPem);
            return new RespuestaTimbrado
            {
                Code = response.code,
                Message = response.message,
                Data = response.data
            };
        }
        catch (Exception ex)
        {
           // Manejo de errores
           Console.WriteLine($"Error al timbrar con sello: {ex.Message}");
           throw;
        }
    }

    public class RespuestaTimbrado
   {
      public string? Code { get; set; }
      public string? Message { get; set; }
      public string? Data { get; set; }
   }

   // Ejemplo de uso
   public async Task EjemploUsoTimbrarConSelloAsync()
   {
       string apiKey = "TU_API_KEY_AQUI";
       string xmlCfdi = GetXmlCfdiSinSello();
       string keyPem = await File.ReadAllTextAsync("ruta/a/tu/llave.key.pem");

       var resultado = await TimbrarConSelloAsync(apiKey, xmlCfdi, keyPem);

       if (resultado?.Code == "200")
       {
           Console.WriteLine("¡Timbrado con Sello Exitoso!");
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
  import java.util.concurrent.CompletableFuture;

  public class TimbradoConSelloService {

    // ... (Definición de ExecutorService, Logger, etc.)

    public String generarXmlCfdiSinSello() {
        return """
        <?xml version=\"1.0\" encoding=\"UTF-8\"?>
        <cfdi:Comprobante
            xmlns:cfdi=\"http://www.sat.gob.mx/cfd/4\"
            xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\"
            xsi:schemaLocation=\"http://www.sat.gob.mx/cfd/4 cfdv40.xsd\" 
            Version=\"4.0\" Serie=\"F\" Folio=\"123\" Fecha=\"2025-07-05T12:00:00\"
            SubTotal=\"100.00\" Moneda=\"MXN\" Total=\"116.00\" TipoDeComprobante=\"I\"
            Exportacion=\"01\" MetodoPago=\"PUE\" FormaPago=\"01\"
            NoCertificado=\"30001000000300023708\" Certificado=\"MIIC...\" Sello=\"\"
            LugarExpedicion=\"64000\">
            <cfdi:Emisor Rfc=\"ABC010101XYZ\" Nombre=\"Empresa Ejemplo\" RegimenFiscal=\"601\"/>
            <cfdi:Receptor Rfc=\"XAXX010101000\" Nombre=\"Publico en General\" DomicilioFiscalReceptor=\"64000\" RegimenFiscalReceptor=\"616\" UsoCFDI=\"G03\"/>
            <cfdi:Conceptos>
                <cfdi:Concepto ObjetoImp=\"02\" ClaveProdServ=\"01010101\" ClaveUnidad=\"ACT\" Cantidad=\"1\" Descripcion=\"Servicio\" ValorUnitario=\"100.00\" Importe=\"100.00\">
                    <cfdi:Impuestos>
                        <cfdi:Traslados>
                            <cfdi:Traslado Base=\"100.00\" Impuesto=\"002\" TipoFactor=\"Tasa\" TasaOCuota=\"0.160000\" Importe=\"16.00\"/>
                        </cfdi:Traslados>
                    </cfdi:Impuestos>
                </cfdi:Concepto>
            </cfdi:Conceptos>
            <cfdi:Impuestos TotalImpuestosTrasladados=\"16.00\">
               <cfdi:Traslados>
                     <cfdi:Traslado Base=\"100.00\" Impuesto=\"002\" TipoFactor=\"Tasa\" TasaOCuota=\"0.160000\" Importe=\"16.00\"/>
               </cfdi:Traslados>
            </cfdi:Impuestos>
        </cfdi:Comprobante>
        """;
    }

    public CompletableFuture<RespuestaTimbrado> timbrarConSelloAsync(String apiKey, String xmlCfdi, String keyPem) {
        return CompletableFuture.supplyAsync(() -> {
            try {
                ServicioTimbradoWS service = new ServicioTimbradoWS();
                ServicioTimbradoWSPortType port = service.getServicioTimbradoWSPort();
                
                Respuesta response = port.timbrarConSello(apiKey, xmlCfdi, keyPem);

                RespuestaTimbrado resultado = new RespuestaTimbrado();
                resultado.setCode(response.getCode());
                resultado.setMessage(response.getMessage());
                resultado.setData(response.getData());
                return resultado;

            } catch (Exception ex) {
                throw new RuntimeException("Error al timbrar con sello", ex);
            }
        }, executor);
    }

    // Ejemplo de uso
    public static void main(String[] args) {
        TimbradoConSelloService service = new TimbradoConSelloService();
        String apiKey = "TU_API_KEY_AQUI";
        String xmlCfdi = service.generarXmlCfdiSinSello();
        try {
            String keyPem = new String(Files.readAllBytes(Paths.get("ruta/a/tu/llave.key.pem")));

            service.timbrarConSelloAsync(apiKey, xmlCfdi, keyPem).whenComplete((resultado, ex) -> {
                if (ex != null) {
                    System.err.println("Error: " + ex.getMessage());
                } else if ("200".equals(resultado.getCode())) {
                    System.out.println("¡Timbrado con Sello Exitoso!");
                    System.out.println(resultado.getData());
                } else {
                    System.err.println("Error: " + resultado.getCode() + " - " + resultado.getMessage());
                }
                service.shutdown();
            }).join();
        } catch (IOException e) {
            System.err.println("No se pudo leer el archivo de la llave PEM.");
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
  from zeep.asyncio import AsyncClient

  def generar_xml_cfdi_sin_sello() -> str:
      return """\
        <?xml version=\"1.0\" encoding=\"UTF-8\"?>
        <cfdi:Comprobante
            xmlns:cfdi=\"http://www.sat.gob.mx/cfd/4\"
            xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\"
            xsi:schemaLocation=\"http://www.sat.gob.mx/cfd/4 cfdv40.xsd\" 
            Version=\"4.0\" Serie=\"F\" Folio=\"123\" Fecha=\"2025-07-05T12:00:00\"
            SubTotal=\"100.00\" Moneda=\"MXN\" Total=\"116.00\" TipoDeComprobante=\"I\"
            Exportacion=\"01\" MetodoPago=\"PUE\" FormaPago=\"01\"
            NoCertificado=\"30001000000300023708\" Certificado=\"MIIC...\" Sello=\"\"
            LugarExpedicion=\"64000\">
            <cfdi:Emisor Rfc=\"ABC010101XYZ\" Nombre=\"Empresa Ejemplo\" RegimenFiscal=\"601\"/>
            <cfdi:Receptor Rfc=\"XAXX010101000\" Nombre=\"Publico en General\" DomicilioFiscalReceptor=\"64000\" RegimenFiscalReceptor=\"616\" UsoCFDI=\"G03\"/>
            <cfdi:Conceptos>
                <cfdi:Concepto ObjetoImp=\"02\" ClaveProdServ=\"01010101\" ClaveUnidad=\"ACT\" Cantidad=\"1\" Descripcion=\"Servicio\" ValorUnitario=\"100.00\" Importe=\"100.00\">
                    <cfdi:Impuestos>
                        <cfdi:Traslados>
                            <cfdi:Traslado Base=\"100.00\" Impuesto=\"002\" TipoFactor=\"Tasa\" TasaOCuota=\"0.160000\" Importe=\"16.00\"/>
                        </cfdi:Traslados>
                    </cfdi:Impuestos>
                </cfdi:Concepto>
            </cfdi:Conceptos>
            <cfdi:Impuestos TotalImpuestosTrasladados=\"16.00\">
               <cfdi:Traslados>
                     <cfdi:Traslado Base=\"100.00\" Impuesto=\"002\" TipoFactor=\"Tasa\" TasaOCuota=\"0.160000\" Importe=\"16.00\"/>
               </cfdi:Traslados>
            </cfdi:Impuestos>
        </cfdi:Comprobante>      
      """

  class TimbradoService:
      def __init__(self, wsdl_url: str):
          self.wsdl_url = wsdl_url
          self.async_client = AsyncClient(self.wsdl_url)

      async def timbrar_con_sello_async(self, api_key: str, xml_cfdi: str, key_pem: str) -> RespuestaTimbrado:
          response = await self.async_client.service.timbrarConSello(
              apikey=api_key,
              xmlCFDI=xml_cfdi,
              keypem=key_pem
          )
          return RespuestaTimbrado(
              code=response.code,
              message=response.message,
              data=response.data
          )

  async def main():
      service = TimbradoService("https://dev.facturaloplus.com/ws/servicio.do?wsdl")
      api_key = "TU_API_KEY_AQUI";
      xml_cfdi = generar_xml_cfdi_sin_sello();
      with open("ruta/a/tu/llave.key.pem", "r") as f:
          key_pem = f.read()

      resultado = await service.timbrar_con_sello_async(api_key, xml_cfdi, key_pem);

      if resultado.code == "200":
          print("¡Timbrado con Sello Exitoso!")
          print(resultado.data)
      else:
          print(f"Error: {resultado.code} - {resultado.message}")

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
  function generarXmlCfdiSinSello(): string {
      return <<<'XML'
        <?xml version=\"1.0\" encoding=\"UTF-8\"?>
        <cfdi:Comprobante
            xmlns:cfdi=\"http://www.sat.gob.mx/cfd/4\"
            xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\"
            xsi:schemaLocation=\"http://www.sat.gob.mx/cfd/4 cfdv40.xsd\" 
            Version=\"4.0\" Serie=\"F\" Folio=\"123\" Fecha=\"2025-07-05T12:00:00\"
            SubTotal=\"100.00\" Moneda=\"MXN\" Total=\"116.00\" TipoDeComprobante=\"I\"
            Exportacion=\"01\" MetodoPago=\"PUE\" FormaPago=\"01\"
            NoCertificado=\"30001000000300023708\" Certificado=\"MIIC...\" Sello=\"\"
            LugarExpedicion=\"64000\">
            <cfdi:Emisor Rfc=\"ABC010101XYZ\" Nombre=\"Empresa Ejemplo\" RegimenFiscal=\"601\"/>
            <cfdi:Receptor Rfc=\"XAXX010101000\" Nombre=\"Publico en General\" DomicilioFiscalReceptor=\"64000\" RegimenFiscalReceptor=\"616\" UsoCFDI=\"G03\"/>
            <cfdi:Conceptos>
                <cfdi:Concepto ObjetoImp=\"02\" ClaveProdServ=\"01010101\" ClaveUnidad=\"ACT\" Cantidad=\"1\" Descripcion=\"Servicio\" ValorUnitario=\"100.00\" Importe=\"100.00\">
                    <cfdi:Impuestos>
                        <cfdi:Traslados>
                            <cfdi:Traslado Base=\"100.00\" Impuesto=\"002\" TipoFactor=\"Tasa\" TasaOCuota=\"0.160000\" Importe=\"16.00\"/>
                        </cfdi:Traslados>
                    </cfdi:Impuestos>
                </cfdi:Concepto>
            </cfdi:Conceptos>
            <cfdi:Impuestos TotalImpuestosTrasladados=\"16.00\">
               <cfdi:Traslados>
                     <cfdi:Traslado Base=\"100.00\" Impuesto=\"002\" TipoFactor=\"Tasa\" TasaOCuota=\"0.160000\" Importe=\"16.00\"/>
               </cfdi:Traslados>
            </cfdi:Impuestos>
        </cfdi:Comprobante>
XML;
  }

  class TimbradoService {
      private string $wsdlUrl;

      public function __construct(string $wsdlUrl) {
          $this->wsdlUrl = $wsdlUrl;
      }

      public function timbrarConSello(string $apiKey, string $xmlCfdi, string $keyPem): RespuestaTimbrado {
          $respuesta = new RespuestaTimbrado();
          try {
              $soapClient = new SoapClient($this->wsdlUrl, ['trace' => 1, 'exceptions' => true]);
              $params = [
                  'apikey'  => $apiKey,
                  'xmlCFDI' => $xmlCfdi,
                  'keypem'  => $keyPem
              ];

              $response = $soapClient->timbrarConSello($params);
              
              $respuesta->code = $response->return->code ?? null;
              $respuesta->message = $response->return->message ?? null;
              $respuesta->data = $response->return->data ?? null;

          } catch (Exception $e) {
              $respuesta->code = "CLIENT_ERROR";
              $respuesta->message = $e->getMessage();
          }
          return $respuesta;
      }
  }

  // Ejemplo de uso
  $service = new TimbradoService("https://dev.facturaloplus.com/ws/servicio.do?wsdl");
  $apiKey = "TU_API_KEY_AQUI";
  $xmlCfdi = generarXmlCfdiSinSello();
  $keyPem = file_get_contents("ruta/a/tu/llave.key.pem");

  $resultado = $service->timbrarConSello($apiKey, $xmlCfdi, $keyPem);

  header('Content-Type: text/plain');
  if ($resultado->code === '200') {
      echo "¡Timbrado con Sello Exitoso!\n";
      echo $resultado->data;
  } else {
      echo "Error: {$resultado->code} - {$resultado->message}\n";
  }
  ?>
  ```
  {{< /tab >}}

{{< /tabs >}}


#### Respuesta (Response)

La estructura de la respuesta SOAP es la misma que la de la operación `timbrar`.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<SOAP-ENV:Envelope ...>
   <SOAP-ENV:Body>
      <ns1:timbrarConSelloResponse xmlns:ns1="urn:ws_api">
         <return xsi:type="tns:RespuestaTimbrado">
            <code xsi:type="xsd:string">CÓDIGO</code>
            <message xsi:type="xsd:string">MENSAJE</message>
            <data xsi:type="xsd:string"><![CDATA[CFDI SELLADO Y TIMBRADO]]></data>
         </return>
      </ns1:timbrarConSelloResponse>
   </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

##### Códigos de respuesta
{{< codigos-timbrado >}}
