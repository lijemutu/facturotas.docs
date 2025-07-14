---
title: Timbrar Sector Primario
---

Esta sección detalla la operación del servicio de timbrado especializado para el Sector Primario.

### Descripción de la Operación

Esta operación permite timbrar un Comprobante Fiscal Digital por Internet (CFDI) que incluye el **Complemento para el Sector Primario**. Está diseñada para contribuyentes que realizan actividades agrícolas, silvícolas, ganaderas o pesqueras.

El servicio procesa un CFDI (versión 4.0), lo valida, y lo timbra ante el SAT, devolviendo el XML completo con el Timbre Fiscal Digital (TFD) incorporado.

### Parámetros de Entrada (Input)

| Parámetro | Tipo de Dato | Descripción                                                                                                |
| :-------- | :----------- | :--------------------------------------------------------------------------------------------------------- |
| `apikey`  | `string`     | Credencial de acceso al servicio ([Solicita aquí](../../../#Requisitos)).                                      |
| `xmlCFDI` | `string`     | Contenido del documento XML del CFDI v4.0 que **debe incluir el Complemento para el Sector Primario**.      |

### Parámetros de Salida (Output) - RespuestaTimbrado

La respuesta de esta operación es idéntica a la de la operación `timbrar`.

| Atributo  | Tipo de Dato | Descripción                             |
| :-------- | :----------- | :-------------------------------------- |
| `code`    | `string`     | Código de respuesta de la operación.    |
| `message` | `string`     | Mensaje detallado de la respuesta.      |
| `data`    | `string`     | XML del CFDI timbrado en caso de éxito. |

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
   private static string GetXmlCfdiSectorPrimario() => """ 
        <?xml version="1.0" encoding="UTF-8"?>
        <cfdi:Comprobante
            xmlns:cfdi="http://www.sat.gob.mx/cfd/4"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:sp="http://www.sat.gob.mx/sectorprimario"
            xsi:schemaLocation="http://www.sat.gob.mx/cfd/4 http://www.sat.gob.mx/sitio_internet/cfd/4/cfdv40.xsd http://www.sat.gob.mx/sectorprimario http://www.sat.gob.mx/sitio_internet/cfd/sectorprimario/sectorprimario.xsd"
            Version="4.0" Serie="SP" Folio="100" Fecha="2025-07-05T12:00:00"
            SubTotal="5000.00" Moneda="MXN" Total="5000.00" TipoDeComprobante="I"
            Exportacion="01" MetodoPago="PUE" FormaPago="01"
            NoCertificado="30001000000300023708" Certificado="MIIC..." Sello="..."
            LugarExpedicion="60123">
            <cfdi:Emisor Rfc="ABC010101XYZ" Nombre="Productor Agrícola Ejemplo" RegimenFiscal="621"/>
            <cfdi:Receptor Rfc="XAXX010101000" Nombre="Comprador General" DomicilioFiscalReceptor="64000" RegimenFiscalReceptor="616" UsoCFDI="G01"/>
            <cfdi:Conceptos>
                <cfdi:Concepto ObjetoImp="01" ClaveProdServ="01010101" ClaveUnidad="KGM" Cantidad="100" Descripcion="Maíz Blanco" ValorUnitario="50.00" Importe="5000.00"/>
            </cfdi:Conceptos>
            <cfdi:Complemento>
                <sp:SectorPrimario Version="1.0" TipoOperacion="Venta"/>
            </cfdi:Complemento>
        </cfdi:Comprobante>
        """;

   public async Task<RespuestaTimbrado> TimbrarSPAsync(string apiKey, string xmlCfdi)
    {
        using var client = new ServicioTimbradoWSPortTypeClient("ServicioTimbradoWSPort");
        try
        {
            var response = await client.timbrarSPAsync(apiKey, xmlCfdi);
            return new RespuestaTimbrado
            {
                Code = response.code,
                Message = response.message,
                Data = response.data
            };
        }
        catch (Exception ex)
        {
           Console.WriteLine($"Error al timbrar Sector Primario: {ex.Message}");
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
   public async Task EjemploUsoTimbrarSPAsync()
   {
       string apiKey = "TU_API_KEY_AQUI";
       string xmlCfdi = GetXmlCfdiSectorPrimario();

       var resultado = await TimbrarSPAsync(apiKey, xmlCfdi);

       if (resultado?.Code == "200")
       {
           Console.WriteLine("¡Timbrado Sector Primario Exitoso!");
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

  public class TimbradoSPService {

    // ... (Definición de ExecutorService, Logger, etc.)

    public String generarXmlCfdiSectorPrimario() {
        return """
        <?xml version="1.0" encoding="UTF-8"?>
        <cfdi:Comprobante
            xmlns:cfdi="http://www.sat.gob.mx/cfd/4"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:sp="http://www.sat.gob.mx/sectorprimario"
            xsi:schemaLocation="http://www.sat.gob.mx/cfd/4 http://www.sat.gob.mx/sitio_internet/cfd/4/cfdv40.xsd http://www.sat.gob.mx/sectorprimario http://www.sat.gob.mx/sitio_internet/cfd/sectorprimario/sectorprimario.xsd"
            Version="4.0" Serie="SP" Folio="100" Fecha="2025-07-05T12:00:00"
            SubTotal="5000.00" Moneda="MXN" Total="5000.00" TipoDeComprobante="I"
            Exportacion="01" MetodoPago="PUE" FormaPago="01"
            NoCertificado="30001000000300023708" Certificado="MIIC..." Sello="..."
            LugarExpedicion="60123">
            <cfdi:Emisor Rfc="ABC010101XYZ" Nombre="Productor Agrícola Ejemplo" RegimenFiscal="621"/>
            <cfdi:Receptor Rfc="XAXX010101000" Nombre="Comprador General" DomicilioFiscalReceptor="64000" RegimenFiscalReceptor="616" UsoCFDI="G01"/>
            <cfdi:Conceptos>
                <cfdi:Concepto ObjetoImp="01" ClaveProdServ="01010101" ClaveUnidad="KGM" Cantidad="100" Descripcion="Maíz Blanco" ValorUnitario="50.00" Importe="5000.00"/>
            </cfdi:Conceptos>
            <cfdi:Complemento>
                <sp:SectorPrimario Version="1.0" TipoOperacion="Venta"/>
            </cfdi:Complemento>
        </cfdi:Comprobante>
        """;
    }

    public CompletableFuture<RespuestaTimbrado> timbrarSPAsync(String apiKey, String xmlCfdi) {
        return CompletableFuture.supplyAsync(() -> {
            try {
                ServicioTimbradoWS service = new ServicioTimbradoWS();
                ServicioTimbradoWSPortType port = service.getServicioTimbradoWSPort();
                
                Respuesta response = port.timbrarSP(apiKey, xmlCfdi);

                RespuestaTimbrado resultado = new RespuestaTimbrado();
                resultado.setCode(response.getCode());
                resultado.setMessage(response.getMessage());
                resultado.setData(response.getData());
                return resultado;

            } catch (Exception ex) {
                throw new RuntimeException("Error al timbrar Sector Primario", ex);
            }
        }, executor);
    }

    // Ejemplo de uso
    public static void main(String[] args) {
        TimbradoSPService service = new TimbradoSPService();
        String apiKey = "TU_API_KEY_AQUI";
        String xmlCfdi = service.generarXmlCfdiSectorPrimario();

        service.timbrarSPAsync(apiKey, xmlCfdi).whenComplete((resultado, ex) -> {
            if (ex != null) {
                System.err.println("Error: " + ex.getMessage());
            } else if ("200".equals(resultado.getCode())) {
                System.out.println("¡Timbrado Sector Primario Exitoso!");
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

  def generar_xml_cfdi_sector_primario() -> str:
      return """\
        <?xml version="1.0" encoding="UTF-8"?>
        <cfdi:Comprobante
            xmlns:cfdi="http://www.sat.gob.mx/cfd/4"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:sp="http://www.sat.gob.mx/sectorprimario"
            xsi:schemaLocation="http://www.sat.gob.mx/cfd/4 http://www.sat.gob.mx/sitio_internet/cfd/4/cfdv40.xsd http://www.sat.gob.mx/sectorprimario http://www.sat.gob.mx/sitio_internet/cfd/sectorprimario/sectorprimario.xsd"
            Version="4.0" Serie="SP" Folio="100" Fecha="2025-07-05T12:00:00"
            SubTotal="5000.00" Moneda="MXN" Total="5000.00" TipoDeComprobante="I"
            Exportacion="01" MetodoPago="PUE" FormaPago="01"
            NoCertificado="30001000000300023708" Certificado="MIIC..." Sello="..."
            LugarExpedicion="60123">
            <cfdi:Emisor Rfc="ABC010101XYZ" Nombre="Productor Agrícola Ejemplo" RegimenFiscal="621"/>
            <cfdi:Receptor Rfc="XAXX010101000" Nombre="Comprador General" DomicilioFiscalReceptor="64000" RegimenFiscal="616" UsoCFDI="G01"/>
            <cfdi:Conceptos>
                <cfdi:Concepto ObjetoImp="01" ClaveProdServ="01010101" ClaveUnidad="KGM" Cantidad="100" Descripcion="Maíz Blanco" ValorUnitario="50.00" Importe="5000.00"/>
            </cfdi:Conceptos>
            <cfdi:Complemento>
                <sp:SectorPrimario Version="1.0" TipoOperacion="Venta"/>
            </cfdi:Complemento>
        """

  class TimbradoService:
      def __init__(self, wsdl_url: str):
          self.wsdl_url = wsdl_url
          self.async_client = AsyncClient(self.wsdl_url)

      async def timbrar_sp_async(self, api_key: str, xml_cfdi: str) -> RespuestaTimbrado:
          response = await self.async_client.service.timbrarSP(
              apikey=api_key,
              xmlCFDI=xml_cfdi
          )
          return RespuestaTimbrado(
              code=response.code,
              message=response.message,
              data=response.data
          )

  async def main():
      service = TimbradoService("https://dev.facturaloplus.com/ws/servicio.do?wsdl")
      api_key = "TU_API_KEY_AQUI";
      xml_cfdi = generar_xml_cfdi_sector_primario();

      resultado = await service.timbrar_sp_async(api_key, xml_cfdi);

      if resultado.code == "200":
          print("¡Timbrado Sector Primario Exitoso!");
          print(resultado.data);
      else:
          print(f"Error: {resultado.code} - {resultado.message}");

  if __name__ == "__main__":
      asyncio.run(main());
  ```
  {{< /tab >}}

  {{< tab >}}
  ### Herramienta SoapClient
  PHP tiene soporte nativo para SOAP a través de la extensión SOAP. Asegúrate de que la extensión php-soap esté habilitada en tu archivo php.ini.

  ### Implementación 
  El siguiente código muestra una implementación orientada a objetos para consumir el servicio de timbrado.
  ```php
  <?php
  function generarXmlCfdiSectorPrimario(): string {
      return <<<'XML'
        <?xml version="1.0" encoding="UTF-8"?>
        <cfdi:Comprobante
            xmlns:cfdi="http://www.sat.gob.mx/cfd/4"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:sp="http://www.sat.gob.mx/sectorprimario"
            xsi:schemaLocation="http://www.sat.gob.mx/cfd/4 http://www.sat.gob.mx/sitio_internet/cfd/4/cfdv40.xsd http://www.sat.gob.mx/sectorprimario http://www.sat.gob.mx/sitio_internet/cfd/sectorprimario/sectorprimario.xsd"
            Version="4.0" Serie="SP" Folio="100" Fecha="2025-07-05T12:00:00"
            SubTotal="5000.00" Moneda="MXN" Total="5000.00" TipoDeComprobante="I"
            Exportacion="01" MetodoPago="PUE" FormaPago="01"
            NoCertificado="30001000000300023708" Certificado="MIIC..." Sello="..."
            LugarExpedicion="60123">
            <cfdi:Emisor Rfc="ABC010101XYZ" Nombre="Productor Agrícola Ejemplo" RegimenFiscal="621"/>
            <cfdi:Receptor Rfc="XAXX010101000" Nombre="Comprador General" DomicilioFiscalReceptor="64000" RegimenFiscalReceptor="616" UsoCFDI="G01"/>
            <cfdi:Conceptos>
                <cfdi:Concepto ObjetoImp="01" ClaveProdServ="01010101" ClaveUnidad="KGM" Cantidad="100" Descripcion="Maíz Blanco" ValorUnitario="50.00" Importe="5000.00"/>
            </cfdi:Conceptos>
            <cfdi:Complemento>
                <sp:SectorPrimario Version="1.0" TipoOperacion="Venta"/>
            </cfdi:Complemento>
        </cfdi:Comprobante>
XML;
  }

  class TimbradoService {
      private string $wsdlUrl;

      public function __construct(string $wsdlUrl) {
          $this->wsdlUrl = $wsdlUrl;
      }

      public function timbrarSP(string $apiKey, string $xmlCfdi): RespuestaTimbrado {
          $respuesta = new RespuestaTimbrado();
          try {
              $soapClient = new SoapClient($this->wsdlUrl, ['trace' => 1, 'exceptions' => true]);
              $params = [
                  'apikey'  => $apiKey,
                  'xmlCFDI' => $xmlCfdi
              ];

              $response = $soapClient->timbrarSP($params);
              
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
  $xmlCfdi = generarXmlCfdiSectorPrimario();

  $resultado = $service->timbrarSP($apiKey, $xmlCfdi);

  header('Content-Type: text/plain');
  if ($resultado->code === '200') {
      echo "¡Timbrado Sector Primario Exitoso!\n";
      echo $resultado.data;
  } else {
      echo "Error: {$resultado.code} - {$resultado.message}\n";
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
      <ns1:timbrarSPResponse xmlns:ns1="urn:ws_api">
         <return xsi:type="tns:RespuestaTimbrado">
            <code xsi:type="xsd:string">CÓDIGO</code>
            <message xsi:type="xsd:string">MENSAJE</message>
            <data xsi:type="xsd:string"><![CDATA[CFDI TIMBRADO]]></data>
         </return>
      </ns1:timbrarSPResponse>
   </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

##### Códigos de respuesta
{{< codigos-timbrado >}}

```