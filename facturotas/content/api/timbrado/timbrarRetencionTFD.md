# timbrarRetencionTFD

### Descripción de la Operación

Esta operación permite timbrar un Comprobante Fiscal de Retenciones e Información de Pagos y, a diferencia de la operación `timbrarRetencion`, retorna únicamente el Timbre Fiscal Digital (TFD) como un XML independiente. Esto es útil cuando se necesita solo el TFD para procesos de validación o almacenamiento por separado.

### Parámetros de Entrada (Input)

| Parámetro | Tipo de Dato | Descripción                                                            |
| :-------- | :----------- | :--------------------------------------------------------------------- |
| `apikey`  | `string`     | Credencial de acceso al servicio ([Solicita aquí](../../../#Requisitos)). |
| `xml`     | `string`     | Contenido del documento XML de la constancia de retenciones a timbrar. |

### Parámetros de Salida (Output) - RespuestaTimbradoTFD

| Atributo  | Tipo de Dato | Descripción                                     |
| :-------- | :----------- | :---------------------------------------------- |
| `code`    | `string`     | Código de respuesta de la operación.            |
| `message` | `string`     | Mensaje detallado de la respuesta.              |
| `data`    | `string`     | XML del Timbre Fiscal Digital (TFD) en caso de éxito. |

### Ejemplo de Código

A continuación se presenta un ejemplo de cómo construir la solicitud y procesar la respuesta.

#### Solicitud (Request)

{{< tabs items="C#,Java,Python,PHP" >}}

  {{< tab >}}
  ### Herramienta svcutil
  Ejecuta el comando para generar el cliente SOAP:
  ```
  svcutil.exe https://dev.facturaloplus.com/ws/servicio.do?wsdl /out:ServicioTimbradoClient.cs /config:app.config
  ```

  ### Implementación

  ```csharp
   private static string GetXmlRetencion() => """ 
        <?xml version="1.0" encoding="UTF-8"?>
        <retenciones:Retenciones 
            xmlns:retenciones="http://www.sat.gob.mx/esquemas/retencionpago/2" 
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
            xsi:schemaLocation="http://www.sat.gob.mx/esquemas/retencionpago/2 http://www.sat.gob.mx/esquemas/retencionpago/2/retencionpagov2.xsd" 
            Version="2.0" FolioInt="124" FechaExp="2025-07-05T11:00:00" LugarExpRetenc="64000" CveRetenc="14">
            <retenciones:Emisor RfcE="ABC010101XYZ" NomDenRazSocE="Empresa Emisora SA de CV" RegimenFiscalE="601"/>
            <retenciones:Receptor NacionalidadR="Nacional">
                <retenciones:Nacional RfcR="XAXX010101000" NomDenRazSocR="Receptor Nacional SA de CV" DomicilioFiscalR="67890"/>
            </retenciones:Receptor>
            <retenciones:Periodo MesIni="07" MesFin="07" Ejerc="2025"/>
            <retenciones:Totales montoTotOperacion="25000" montoTotGrav="25000" montoTotExent="0" montoTotRet="2500">
                <retenciones:ImpRetenidos BaseRet="25000" Impuesto="01" montoRet="2500" TipoPagoRet="Pago definitivo"/>
            </retenciones:Totales>
        </retenciones:Retenciones>
        """;

   public async Task<RespuestaTimbradoTFD> TimbrarRetencionTFDAsync(string apiKey, string xml)
    {
        using var client = new ServicioTimbradoWSPortTypeClient("ServicioTimbradoWSPort");
        try
        {
            var response = await client.timbrarRetencionTFDAsync(apiKey, xml);
            return new RespuestaTimbradoTFD
            {
                Code = response.code,
                Message = response.message,
                Data = response.data
            };
        }
        catch (Exception ex)
        {
           Console.WriteLine($"Error al timbrar TFD de retención: {ex.Message}");
           throw;
        }
    }

   // Ejemplo de uso
   public async Task EjemploUsoTimbrarRetencionTFDAsync()
   {
       string apiKey = "TU_API_KEY_AQUI";
       string xml = GetXmlRetencion();

       var resultado = await TimbrarRetencionTFDAsync(apiKey, xml);

       if (resultado?.Code == "200")
       {
           Console.WriteLine("¡Timbrado de TFD para Retención Exitoso!");
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
  Ejecuta el comando para generar las clases cliente:
  ```
  wsimport -keep -p com.facturaloplus.cliente https://dev.facturaloplus.com/ws/servicio.do?wsdl
  ```

  ### Implementación

  ```java
  import java.util.concurrent.CompletableFuture;

  public class TimbrarRetencionTFDService {

    // ... (Definición de ExecutorService, Logger, etc.)

    public String generarXmlRetencion() {
        return """
        <?xml version="1.0" encoding="UTF-8"?>
        <retenciones:Retenciones 
            xmlns:retenciones="http://www.sat.gob.mx/esquemas/retencionpago/2" 
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
            xsi:schemaLocation="http://www.sat.gob.mx/esquemas/retencionpago/2 http://www.sat.gob.mx/esquemas/retencionpago/2/retencionpagov2.xsd" 
            Version="2.0" FolioInt="124" FechaExp="2025-07-05T11:00:00" LugarExpRetenc="64000" CveRetenc="14">
            <retenciones:Emisor RfcE="ABC010101XYZ" NomDenRazSocE="Empresa Emisora SA de CV" RegimenFiscalE="601"/>
            <retenciones:Receptor NacionalidadR="Nacional">
                <retenciones:Nacional RfcR="XAXX010101000" NomDenRazSocR="Receptor Nacional SA de CV" DomicilioFiscalR="67890"/>
            </retenciones:Receptor>
            <retenciones:Periodo MesIni="07" MesFin="07" Ejerc="2025"/>
            <retenciones:Totales montoTotOperacion="25000" montoTotGrav="25000" montoTotExent="0" montoTotRet="2500">
                <retenciones:ImpRetenidos BaseRet="25000" Impuesto="01" montoRet="2500" TipoPagoRet="Pago definitivo"/>
            </retenciones:Totales>
        </retenciones:Retenciones>
        """;
    }

    public CompletableFuture<RespuestaTimbradoTFD> timbrarRetencionTFDAsync(String apiKey, String xml) {
        return CompletableFuture.supplyAsync(() -> {
            try {
                ServicioTimbradoWS service = new ServicioTimbradoWS();
                ServicioTimbradoWSPortType port = service.getServicioTimbradoWSPort();
                
                Respuesta response = port.timbrarRetencionTFD(apiKey, xml);

                RespuestaTimbradoTFD resultado = new RespuestaTimbradoTFD();
                resultado.setCode(response.getCode());
                resultado.setMessage(response.getMessage());
                resultado.setData(response.getData());
                return resultado;

            } catch (Exception ex) {
                throw new RuntimeException("Error al timbrar TFD de retención", ex);
            }
        }, executor);
    }

    // Ejemplo de uso
    public static void main(String[] args) {
        TimbrarRetencionTFDService service = new TimbrarRetencionTFDService();
        String apiKey = "TU_API_KEY_AQUI";
        String xml = service.generarXmlRetencion();

        service.timbrarRetencionTFDAsync(apiKey, xml).whenComplete((resultado, ex) -> {
            if (ex != null) {
                System.err.println("Error: " + ex.getMessage());
            } else if ("200".equals(resultado.getCode())) {
                System.out.println("¡Timbrado de TFD para Retención Exitoso!");
                System.out.println(resultado.getData());
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
  Instala la librería usando pip:
  ```
  pip install zeep
  ```

  ### Implementación
  ```python
  import asyncio
  from zeep.asyncio import AsyncClient

  def generar_xml_retencion() -> str:
      return """\
        <?xml version="1.0" encoding="UTF-8"?>
        <retenciones:Retenciones 
            xmlns:retenciones="http://www.sat.gob.mx/esquemas/retencionpago/2" 
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
            xsi:schemaLocation="http://www.sat.gob.mx/esquemas/retencionpago/2 http://www.sat.gob.mx/esquemas/retencionpago/2/retencionpagov2.xsd" 
            Version="2.0" FolioInt="124" FechaExp="2025-07-05T11:00:00" LugarExpRetenc="64000" CveRetenc="14">
            <retenciones:Emisor RfcE="ABC010101XYZ" NomDenRazSocE="Empresa Emisora SA de CV" RegimenFiscalE="601"/>
            <retenciones:Receptor NacionalidadR="Nacional">
                <retenciones:Nacional RfcR="XAXX010101000" NomDenRazSocR="Receptor Nacional SA de CV" DomicilioFiscalR="67890"/>
            </retenciones:Receptor>
            <retenciones:Periodo MesIni="07" MesFin="07" Ejerc="2025"/>
            <retenciones:Totales montoTotOperacion="25000" montoTotGrav="25000" montoTotExent="0" montoTotRet="2500">
                <retenciones:ImpRetenidos BaseRet="25000" Impuesto="01" montoRet="2500" TipoPagoRet="Pago definitivo"/>
            </retenciones:Totales>
        </retenciones:Retenciones>
      """

  class TimbradoService:
      def __init__(self, wsdl_url: str):
          self.wsdl_url = wsdl_url
          self.async_client = AsyncClient(self.wsdl_url)

      async def timbrar_retencion_tfd_async(self, api_key: str, xml: str) -> RespuestaTimbradoTFD:
          response = await self.async_client.service.timbrarRetencionTFD(
              apikey=api_key,
              xml=xml
          )
          return RespuestaTimbradoTFD(
              code=response.code,
              message=response.message,
              data=response.data
          )

  async def main():
      service = TimbradoService("https://dev.facturaloplus.com/ws/servicio.do?wsdl")
      api_key = "TU_API_KEY_AQUI"
      xml = generar_xml_retencion()

      resultado = await service.timbrar_retencion_tfd_async(api_key, xml);

      if resultado.code == "200":
          print("¡Timbrado de TFD para Retención Exitoso!")
          print(resultado.data)
      else:
          print(f"Error: {resultado.code} - {resultado.message}")

  if __name__ == "__main__":
      asyncio.run(main())
  ```
  {{< /tab >}}

  {{< tab >}}
  ### Herramienta SoapClient
  Asegúrate de que la extensión `php-soap` esté habilitada en tu `php.ini`.

  ### Implementación 
  ```php
  <?php
  function generarXmlRetencion(): string {
      return <<<'XML'
        <?xml version="1.0" encoding="UTF-8"?>
        <retenciones:Retenciones 
            xmlns:retenciones="http://www.sat.gob.mx/esquemas/retencionpago/2" 
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
            xsi:schemaLocation="http://www.sat.gob.mx/esquemas/retencionpago/2 http://www.sat.gob.mx/esquemas/retencionpago/2/retencionpagov2.xsd" 
            Version="2.0" FolioInt="124" FechaExp="2025-07-05T11:00:00" LugarExpRetenc="64000" CveRetenc="14">
            <retenciones:Emisor RfcE="ABC010101XYZ" NomDenRazSocE="Empresa Emisora SA de CV" RegimenFiscalE="601"/>
            <retenciones:Receptor NacionalidadR="Nacional">
                <retenciones:Nacional RfcR="XAXX010101000" NomDenRazSocR="Receptor Nacional SA de CV" DomicilioFiscalR="67890"/>
            </retenciones:Receptor>
            <retenciones:Periodo MesIni="07" MesFin="07" Ejerc="2025"/>
            <retenciones:Totales montoTotOperacion="25000" montoTotGrav="25000" montoTotExent="0" montoTotRet="2500">
                <retenciones:ImpRetenidos BaseRet="25000" Impuesto="01" montoRet="2500" TipoPagoRet="Pago definitivo"/>
            </retenciones:Totales>
        </retenciones:Retenciones>
XML;
  }

  class TimbradoService {
      private string $wsdlUrl;

      public function __construct(string $wsdlUrl) {
          $this->wsdlUrl = $wsdlUrl;
      }

      public function timbrarRetencionTFD(string $apiKey, string $xml): RespuestaTimbradoTFD {
          $respuesta = new RespuestaTimbradoTFD();
          try {
              $soapClient = new SoapClient($this->wsdlUrl, ['trace' => 1, 'exceptions' => true]);
              $params = [
                  'apikey'  => $apiKey,
                  'xml' => $xml
              ];

              $response = $soapClient->timbrarRetencionTFD($params);
              
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
  $xml = generarXmlRetencion();

  $resultado = $service->timbrarRetencionTFD($apiKey, $xml);

  header('Content-Type: text/plain');
  if ($resultado->code === '200') {
      echo "¡Timbrado de TFD para Retención Exitoso!\n";
      echo $resultado->data;
  } else {
      echo "Error: {$resultado->code} - {$resultado->message}\n";
  }
  ?>
  ```
  {{< /tab >}}

{{< /tabs >}}

#### Respuesta (Response)

El campo `data` contendrá únicamente el XML del Timbre Fiscal Digital (TFD).

```xml
<?xml version="1.0" encoding="UTF-8"?>
<SOAP-ENV:Envelope ...>
   <SOAP-ENV:Body>
      <ns1:timbrarRetencionTFDResponse xmlns:ns1="urn:ws_api">
         <return xsi:type="tns:RespuestaTimbradoTFD">
            <code xsi:type="xsd:string">CÓDIGO</code>
            <message xsi:type="xsd:string">MENSAJE</message>
            <data xsi:type="xsd:string"><![CDATA[<tfd:TimbreFiscalDigital ... />]]></data>
         </return>
      </ns1:timbrarRetencionTFDResponse>
   </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

##### Códigos de respuesta
{{< codigos-timbrado >}}

```