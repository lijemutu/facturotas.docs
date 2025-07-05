# timbrarTFD

### Descripción de la Operación

Esta operación permite timbrar un Comprobante Fiscal Digital por Internet (CFDI) en sus versiones 3.3 o 4.0 y, a diferencia de la operación `timbrar`, retorna únicamente el Timbre Fiscal Digital (TFD) como un XML independiente. Esto es útil cuando se necesita solo el TFD para procesos de validación o almacenamiento por separado.

### Parámetros de Entrada (Input)

| Parámetro | Tipo de Dato | Descripción                                                            |
| :-------- | :----------- | :--------------------------------------------------------------------- |
| `apikey`  | `string`     | Credencial de acceso al servicio ([Solicita aquí](../../../#Requisitos)). |
| `xmlCFDI` | `string`     | Contenido del documento XML del CFDI v3.3 o v4.0 a timbrar.            |

### Parámetros de Salida (Output) - RespuestaTimbradoTFD

| Atributo  | Tipo de Dato | Descripción                                     |
| :-------- | :----------- | :---------------------------------------------- |
| `code`    | `string`     | Código de respuesta de la operación.            |
| `message` | `string`     | Mensaje detallado de la respuesta.              |
| `data`    | `string`     | XML del Timbre Fiscal Digital (TFD) en caso de éxito. |

### Ejemplo de Código

A continuación se presenta un ejemplo de cómo construir la solicitud y procesar la respuesta utilizando el servicio Web.

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
   /// <summary>
   /// Genera un XML de ejemplo para un CFDI 4.0.
   /// En una aplicación real, este XML se construiría dinámicamente.
   /// </summary>
   /// <returns>Un string con el contenido del CFDI.</returns>
   private static string GenerarCfdiParaTimbrar() => """ 
        <?xml version="1.0" encoding="UTF-8"?>
        <cfdi:Comprobante
            xmlns:cfdi="http://www.sat.gob.mx/cfd/4"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://www.sat.gob.mx/cfd/4 cfdv40.xsd" 
            Version="4.0"
            Serie="F"
            Folio="123"
            Fecha="2025-07-02T12:00:00"
            SubTotal="100.00"
            TipoCambio="1.0"
            Serie="A"
            Moneda="MXN"
            Descuento="0.00"
            Total="116.00"
            TipoDeComprobante="I"
            Exportacion="01"
            MetodoPago="PUE"
            CondicionesDePago="CONDICIONES"
            FormaPago="01"
            Confirmacion="A1234"
            NoCertificado="30001000000300023708"
            Certificado=""
            Sello=""
            LugarExpedicion="64000">
            <cfdi:InformacionGlobal Meses="18" Año="2025" Periodicidad="05" />
            <cfdi:CfdiRelacionados TipoRelacion="09">
               <cfdi:CfdiRelacionado UUID="ED1752FE-E865-4FF2-BFE1-0F552E770DC9" />
            </cfdi:CfdiRelacionados>
            <cfdi:Emisor FacAtrAdquirente="0123456789" Rfc="ABC010101XYZ" Nombre="Empresa Ejemplo" RegimenFiscal="601"/>
            <cfdi:Receptor Rfc="XAXX010101000" Nombre="Publico en General" 
                          DomicilioFiscalReceptor="64000" RegimenFiscalReceptor="616" 
                          ResidenciaFiscal="MEX"
                          UsoCFDI="G03"/>
            <cfdi:Conceptos>
                <cfdi:Concepto ObjetoImp="01" ClaveProdServ="01010101" ClaveUnidad="C81" 
                      NoIdentificacion="00001" Cantidad="1.5" 
                        Unidad="TONELADA" Descripcion="ACERO" ValorUnitario="1500000" Importe="2250000">
                    <cfdi:Impuestos>
                        <cfdi:Traslados>
                            <cfdi:Traslado Base="100.00" Impuesto="002" TipoFactor="Tasa" 
                                         TasaOCuota="0.160000" Importe="16.00"/>
                        </cfdi:Traslados>
                        <cfdi:Retenciones>
                           <cfdi:Retencion Base="2250000" Impuesto="001" TipoFactor="Tasa" TasaOCuota="0.300000" Importe="247500"/>
                        </cfdi:Retenciones>
                    </cfdi:Impuestos>
                    <cfdi:CuentaPredial Numero="51888"/>
                </cfdi:Concepto>
            </cfdi:Conceptos>
            <cfdi:Impuestos TotalImpuestosRetenidos="247500"      
                            TotalImpuestosTrasladados="360000">
                <cfdi:Retenciones>
                  <cfdi:Retencion Impuesto="001" Importe="247000"/>
                  <cfdi:Retencion Impuesto="003" Importe="500"/>
               </cfdi:Retenciones>
               <cfdi:Traslados>
                     <cfdi:Traslado Base="1.00" Impuesto="002" TipoFactor="Tasa" TasaOCuota="1.600000" Importe="360000"/>
               </cfdi:Traslados>
            </cfdi:Impuestos>
            <cfdi:Complemento></cfdi:Complemento>
        </cfdi:Comprobante>
        """;

   /// <summary>
   /// Invoca al servicio web para obtener solo el TFD de un CFDI de forma asíncrona.
   /// </summary>
   /// <param name="apiKey">La credencial de acceso al servicio.</param>
   /// <param name="xmlCfdi">El contenido del CFDI a timbrar.</param>
   /// <returns>Un objeto RespuestaTimbradoTFD con el resultado de la operación.</returns>
   public async Task<RespuestaTimbradoTFD> TimbrarTFDAsync(string apiKey, string xmlCfdi)
    {
        using var client = new ServicioTimbradoWSPortTypeClient("ServicioTimbradoWSPort");
        
        try
        {            
            // La operación se llama 'timbrarTFD' en el servicio web
            var response = await client.timbrarTFDAsync(apiKey, xmlCfdi);
                        
            return new RespuestaTimbradoTFD
            {
                Code = response.code,
                Message = response.message,
                Data = response.data // Contiene solo el XML del TFD
            };
        }
        catch (FaultException<RespuestaTimbradoTFD> ex)
        {
            _logger.LogError(ex, "Error SOAP: {Code} - {Message}", ex.Detail.code, ex.Detail.message);
            throw;
        }
        catch (CommunicationException ex)
        {
            _logger.LogError(ex, "Error de comunicación con el servicio");
            throw;
        }
        catch (TimeoutException ex)
        {
            _logger.LogError(ex, "Timeout en la comunicación");
            throw;
        }
    }

   /// <summary>
   /// Define la estructura del objeto de respuesta para mayor claridad.
   /// </summary>
   public class RespuestaTimbradoTFD
   {
      public string? Code { get; set; }
      public string? Message { get; set; }
      public string? Data { get; set; }
   }

   // Ejemplo de uso de TimbrarTFDAsync
   public async Task EjemploUsoTimbrarTFDAsync()
   {
       string apiKey = "TU_API_KEY_AQUI";
       string xmlCfdi = GenerarCfdiParaTimbrar();

       try
       {
           var resultado = await TimbrarTFDAsync(apiKey, xmlCfdi);
           if (resultado.Code == "200")
           {
               Console.WriteLine("¡Timbrado para TFD Exitoso!");
               Console.WriteLine("Mensaje: " + resultado.Message);
               Console.WriteLine("--- XML del Timbre Fiscal Digital (TFD) ---");
               Console.WriteLine(resultado.Data);
           }
           else
           {
               Console.WriteLine("Error durante el timbrado TFD:");
               Console.WriteLine("Código: " + resultado.Code);
               Console.WriteLine("Mensaje: " + resultado.Message);
           }
       }
       catch (Exception ex)
       {
           Console.WriteLine("Ocurrió una excepción: " + ex.Message);
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
// --- Archivo: RespuestaTimbradoTFD.java ---
package com.facturaloplus.cliente;

// POJO para encapsular la respuesta de la operación timbrarTFD.
public class RespuestaTimbradoTFD {
    private String code;
    private String message;
    private String data; // Contendrá el XML del TFD

    // Getters y Setters
    public String getCode() { return code; }
    public void setCode(String code) { this.code = code; }
    public String getMessage() { return message; }
    public void setMessage(String message) { this.message = message; }
    public String getData() { return data; }
    public void setData(String data) { this.data = data; }
}

// --- Archivo: TimbradoService.java ---
package com.facturaloplus.cliente;

import javax.xml.ws.Service;
import javax.xml.ws.WebServiceException;
import javax.xml.ws.soap.SOAPFaultException;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.logging.Level;
import java.util.logging.Logger;

public class TimbradoService {

    private static final Logger logger = Logger.getLogger(TimbradoService.class.getName());
    private final ExecutorService executor = Executors.newFixedThreadPool(5);

    public String generarXmlCfdiEjemplo() {
        String fechaActual = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd'T'HH:mm:ss"));
        return """
            <?xml version="1.0" encoding="UTF-8"?>
        <cfdi:Comprobante
            xmlns:cfdi="http://www.sat.gob.mx/cfd/4"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://www.sat.gob.mx/cfd/4 cfdv40.xsd" 
            Version="4.0"
            Serie="F"
            Folio="123"
            Fecha="%s"
            SubTotal="100.00"
            TipoCambio="1.0"
            Serie="A"
            Moneda="MXN"
            Descuento="0.00"
            Total="116.00"
            TipoDeComprobante="I"
            Exportacion="01"
            MetodoPago="PUE"
            CondicionesDePago="CONDICIONES"
            FormaPago="01"
            Confirmacion="A1234"
            NoCertificado="30001000000300023708"
            Certificado=""
            Sello=""
            LugarExpedicion="64000">
            <cfdi:InformacionGlobal Meses="18" Año="2025" Periodicidad="05" />
            <cfdi:CfdiRelacionados TipoRelacion="09">
               <cfdi:CfdiRelacionado UUID="ED1752FE-E865-4FF2-BFE1-0F552E770DC9" />
            </cfdi:CfdiRelacionados>
            <cfdi:Emisor FacAtrAdquirente="0123456789" Rfc="ABC010101XYZ" Nombre="Empresa Ejemplo" RegimenFiscal="601"/>
            <cfdi:Receptor Rfc="XAXX010101000" Nombre="Publico en General" 
                          DomicilioFiscalReceptor="64000" RegimenFiscalReceptor="616" 
                          ResidenciaFiscal="MEX"
                          UsoCFDI="G03"/>
            <cfdi:Conceptos>
                <cfdi:Concepto ObjetoImp="01" ClaveProdServ="01010101" ClaveUnidad="C81" 
                      NoIdentificacion="00001" Cantidad="1.5" 
                        Unidad="TONELADA" Descripcion="ACERO" ValorUnitario="1500000" Importe="2250000">
                    <cfdi:Impuestos>
                        <cfdi:Traslados>
                            <cfdi:Traslado Base="100.00" Impuesto="002" TipoFactor="Tasa" 
                                         TasaOCuota="0.160000" Importe="16.00"/>
                        </cfdi:Traslados>
                        <cfdi:Retenciones>
                           <cfdi:Retencion Base="2250000" Impuesto="001" TipoFactor="Tasa" TasaOCuota="0.300000" Importe="247500"/>
                        </cfdi:Retenciones>
                    </cfdi:Impuestos>
                    <cfdi:CuentaPredial Numero="51888"/>
                </cfdi:Concepto>
            </cfdi:Conceptos>
            <cfdi:Impuestos TotalImpuestosRetenidos="247500"      
                            TotalImpuestosTrasladados="360000">
                <cfdi:Retenciones>
                  <cfdi:Retencion Impuesto="001" Importe="247000"/>
                  <cfdi:Retencion Impuesto="003" Importe="500"/>
               </cfdi:Retenciones>
               <cfdi:Traslados>
                     <cfdi:Traslado Base="1.00" Impuesto="002" TipoFactor="Tasa" TasaOCuota="1.600000" Importe="360000"/>
               </cfdi:Traslados>
            </cfdi:Impuestos>
            <cfdi:Complemento></cfdi:Complemento>
        </cfdi:Comprobante>
            """.formatted(fechaActual);
    }

    public CompletableFuture<RespuestaTimbradoTFD> timbrarTFDAsync(String apiKey, String xmlCfdi) {
        return CompletableFuture.supplyAsync(() -> {
            try {
                ServicioTimbradoWS service = new ServicioTimbradoWS();
                ServicioTimbradoWSPortType port = service.getServicioTimbradoWSPort();

                logger.info("Iniciando timbrado para obtener TFD.");
                // La clase 'Respuesta' es generada por wsimport
                Respuesta response = port.timbrarTFD(apiKey, xmlCfdi); 

                RespuestaTimbradoTFD resultado = new RespuestaTimbradoTFD();
                resultado.setCode(response.getCode());
                resultado.setMessage(response.getMessage());
                resultado.setData(response.getData());
                return resultado;

            } catch (SOAPFaultException ex) {
                logger.log(Level.SEVERE, "Error SOAP: " + ex.getFault().getFaultString(), ex);
                throw new RuntimeException("Error del servicio: " + ex.getFault().getFaultString(), ex);
            } catch (WebServiceException ex) {
                logger.log(Level.SEVERE, "Error de comunicación con el servicio.", ex);
                throw new RuntimeException("Error de comunicación.", ex);
            }
        }, executor);
    }
    
    public void shutdown() {
        executor.shutdown();
    }
}

// --- Archivo: Main.java (Ejemplo de uso) ---
package com.facturaloplus.cliente;

import java.util.concurrent.CompletableFuture;

public class Main {
    public static void main(String[] args) {
        TimbradoService timbradoService = new TimbradoService();
        String miApiKey = "TU_API_KEY_AQUI";
        String xmlParaTimbrar = timbradoService.generarXmlCfdiEjemplo();

        CompletableFuture<RespuestaTimbradoTFD> future = timbradoService.timbrarTFDAsync(miApiKey, xmlParaTimbrar);

        future.whenComplete((resultado, ex) -> {
            if (ex != null) {
                System.err.println("\nOcurrió una excepción: " + ex.getMessage());
            } else {
                if ("200".equals(resultado.getCode())) {
                    System.out.println("\n¡Timbrado para TFD Exitoso!");
                    System.out.println("Mensaje: " + resultado.getMessage());
                    System.out.println("\n--- XML del Timbre Fiscal Digital (TFD) ---");
                    System.out.println(resultado.getData());
                } else {
                    System.err.println("\nError durante el timbrado TFD:");
                    System.err.println("Código: " + resultado.getCode());
                    System.err.println("Mensaje: " + resultado.getMessage());
                }
            }
            timbradoService.shutdown();
        });
        
        future.join(); 
    }
}

  {{< /tab >}}
  {{< tab >}}
  
### Herramienta Zeep
Instala la librería `zeep` usando pip:
```****
pip install zeep
```

### Implementación
```python
# --- Archivo: timbrado_service.py ---
import asyncio
import logging
from datetime import datetime
from zeep import Client
from zeep.asyncio import AsyncClient
from zeep.exceptions import Fault, TransportError
from dataclasses import dataclass

logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

WSDL_URL_DESARROLLO = "https://dev.facturaloplus.com/ws/servicio.do?wsdl"

@dataclass
class RespuestaTimbradoTFD:
    """Clase de datos para encapsular la respuesta del servicio timbrarTFD."""
    code: str = None
    message: str = None
    data: str = None # Contendrá el XML del TFD

class TimbradoService:
    def __init__(self, wsdl_url: str):
        self.wsdl_url = wsdl_url
        self.async_client = AsyncClient(self.wsdl_url)

    def generar_xml_cfdi_ejemplo(self) -> str:
        fecha_actual = datetime.utcnow().strftime('%Y-%m-%dT%H:%M:%S')
        return f"""
<?xml version="1.0" encoding="UTF-8"?>
        <cfdi:Comprobante
            xmlns:cfdi="http://www.sat.gob.mx/cfd/4"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://www.sat.gob.mx/cfd/4 cfdv40.xsd" 
            Version="4.0"
            Serie="F"
            Folio="123"
            Fecha="{fecha_actual}"
            SubTotal="100.00"
            TipoCambio="1.0"
            Serie="A"
            Moneda="MXN"
            Descuento="0.00"
            Total="116.00"
            TipoDeComprobante="I"
            Exportacion="01"
            MetodoPago="PUE"
            CondicionesDePago="CONDICIONES"
            FormaPago="01"
            Confirmacion="A1234"
            NoCertificado="30001000000300023708"
            Certificado=""
            Sello=""
            LugarExpedicion="64000">
            <cfdi:InformacionGlobal Meses="18" Año="2025" Periodicidad="05" />
            <cfdi:CfdiRelacionados TipoRelacion="09">
               <cfdi:CfdiRelacionado UUID="ED1752FE-E865-4FF2-BFE1-0F552E770DC9" />
            </cfdi:CfdiRelacionados>
            <cfdi:Emisor FacAtrAdquirente="0123456789" Rfc="ABC010101XYZ" Nombre="Empresa Ejemplo" RegimenFiscal="601"/>
            <cfdi:Receptor Rfc="XAXX010101000" Nombre="Publico en General" 
                          DomicilioFiscalReceptor="64000" RegimenFiscalReceptor="616" 
                          ResidenciaFiscal="MEX"
                          UsoCFDI="G03"/>
            <cfdi:Conceptos>
                <cfdi:Concepto ObjetoImp="01" ClaveProdServ="01010101" ClaveUnidad="C81" 
                      NoIdentificacion="00001" Cantidad="1.5" 
                        Unidad="TONELADA" Descripcion="ACERO" ValorUnitario="1500000" Importe="2250000">
                    <cfdi:Impuestos>
                        <cfdi:Traslados>
                            <cfdi:Traslado Base="100.00" Impuesto="002" TipoFactor="Tasa" 
                                         TasaOCuota="0.160000" Importe="16.00"/>
                        </cfdi:Traslados>
                        <cfdi:Retenciones>
                           <cfdi:Retencion Base="2250000" Impuesto="001" TipoFactor="Tasa" TasaOCuota="0.300000" Importe="247500"/>
                        </cfdi:Retenciones>
                    </cfdi:Impuestos>
                    <cfdi:CuentaPredial Numero="51888"/>
                </cfdi:Concepto>
            </cfdi:Conceptos>
            <cfdi:Impuestos TotalImpuestosRetenidos="247500"      
                            TotalImpuestosTrasladados="360000">
                <cfdi:Retenciones>
                  <cfdi:Retencion Impuesto="001" Importe="247000"/>
                  <cfdi:Retencion Impuesto="003" Importe="500"/>
               </cfdi:Retenciones>
               <cfdi:Traslados>
                     <cfdi:Traslado Base="1.00" Impuesto="002" TipoFactor="Tasa" TasaOCuota="1.600000" Importe="360000"/>
               </cfdi:Traslados>
            </cfdi:Impuestos>
            <cfdi:Complemento></cfdi:Complemento>
        </cfdi:Comprobante>
"""

    async def timbrar_tfd_async(self, api_key: str, xml_cfdi: str) -> RespuestaTimbradoTFD:
        """Invoca al servicio web para obtener solo el TFD de un CFDI."""
        try:
            logging.info("Iniciando timbrado para obtener TFD.")
            response = await self.async_client.service.timbrarTFD(apikey=api_key, xmlCFDI=xml_cfdi)
            
            return RespuestaTimbradoTFD(
                code=response.code,
                message=response.message,
                data=response.data
            )
        except Fault as ex:
            logging.error(f"Error SOAP (Fault): {ex.message}")
            raise ConnectionError(f"Error del servicio: {ex.message}") from ex
        except TransportError as ex:
            logging.error(f"Error de comunicación: {ex}")
            raise

# --- Archivo: main.py (Ejemplo de uso) ---
async def main():
    service = TimbradoService(WSDL_URL_DESARROLLO)
    mi_api_key = "TU_API_KEY_AQUI"
    xml_para_timbrar = service.generar_xml_cfdi_ejemplo()

    try:
        resultado = await service.timbrar_tfd_async(mi_api_key, xml_para_timbrar)
        
        if resultado.code == "200":
            print("\033[92m\n¡Timbrado para TFD Exitoso!\033[0m")
            print(f"Mensaje: {resultado.message}")
            print("\n--- XML del Timbre Fiscal Digital (TFD) ---")
            print(resultado.data)
        else:
            print(f"\033[91m\nError durante el timbrado TFD:\033[0m")
            print(f"Código: {resultado.code}")
            print(f"Mensaje: {resultado.message}")

    except Exception as ex:
        print(f"\033[91m\nOcurrió una excepción: {ex}\033[0m")

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
// --- Archivo: TimbradoTFD.php ---

ini_set('display_errors', 1);
error_reporting(E_ALL);

class RespuestaTimbradoTFD {
    public ?string $code = null;
    public ?string $message = null;
    public ?string $data = null; // Contendrá el XML del TFD
}

class TimbradoService {
    private string $wsdlUrl;

    public function __construct(string $wsdlUrl) {
        $this->wsdlUrl = $wsdlUrl;
    }

    public function generarXmlCfdiEjemplo(): string {
        $fechaActual = gmdate('Y-m-d\TH:i:s');
        return <<<XML
<?xml version="1.0" encoding="UTF-8"?>
        <cfdi:Comprobante
            xmlns:cfdi="http://www.sat.gob.mx/cfd/4"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://www.sat.gob.mx/cfd/4 cfdv40.xsd" 
            Version="4.0"
            Serie="F"
            Folio="123"
            Fecha="{$fechaActual}"
            SubTotal="100.00"
            TipoCambio="1.0"
            Serie="A"
            Moneda="MXN"
            Descuento="0.00"
            Total="116.00"
            TipoDeComprobante="I"
            Exportacion="01"
            MetodoPago="PUE"
            CondicionesDePago="CONDICIONES"
            FormaPago="01"
            Confirmacion="A1234"
            NoCertificado="30001000000300023708"
            Certificado=""
            Sello=""
            LugarExpedicion="64000">
            <cfdi:InformacionGlobal Meses="18" Año="2025" Periodicidad="05" />
            <cfdi:CfdiRelacionados TipoRelacion="09">
               <cfdi:CfdiRelacionado UUID="ED1752FE-E865-4FF2-BFE1-0F552E770DC9" />
            </cfdi:CfdiRelacionados>
            <cfdi:Emisor FacAtrAdquirente="0123456789" Rfc="ABC010101XYZ" Nombre="Empresa Ejemplo" RegimenFiscal="601"/>
            <cfdi:Receptor Rfc="XAXX010101000" Nombre="Publico en General" 
                          DomicilioFiscalReceptor="64000" RegimenFiscalReceptor="616" 
                          ResidenciaFiscal="MEX"
                          UsoCFDI="G03"/>
            <cfdi:Conceptos>
                <cfdi:Concepto ObjetoImp="01" ClaveProdServ="01010101" ClaveUnidad="C81" 
                      NoIdentificacion="00001" Cantidad="1.5" 
                        Unidad="TONELADA" Descripcion="ACERO" ValorUnitario="1500000" Importe="2250000">
                    <cfdi:Impuestos>
                        <cfdi:Traslados>
                            <cfdi:Traslado Base="100.00" Impuesto="002" TipoFactor="Tasa" 
                                         TasaOCuota="0.160000" Importe="16.00"/>
                        </cfdi:Traslados>
                        <cfdi:Retenciones>
                           <cfdi:Retencion Base="2250000" Impuesto="001" TipoFactor="Tasa" TasaOCuota="0.300000" Importe="247500"/>
                        </cfdi:Retenciones>
                    </cfdi:Impuestos>
                    <cfdi:CuentaPredial Numero="51888"/>
                </cfdi:Concepto>
            </cfdi:Conceptos>
            <cfdi:Impuestos TotalImpuestosRetenidos="247500"      
                            TotalImpuestosTrasladados="360000">
                <cfdi:Retenciones>
                  <cfdi:Retencion Impuesto="001" Importe="247000"/>
                  <cfdi:Retencion Impuesto="003" Importe="500"/>
               </cfdi:Retenciones>
               <cfdi:Traslados>
                     <cfdi:Traslado Base="1.00" Impuesto="002" TipoFactor="Tasa" TasaOCuota="1.600000" Importe="360000"/>
               </cfdi:Traslados>
            </cfdi:Impuestos>
            <cfdi:Complemento></cfdi:Complemento>
        </cfdi:Comprobante>
XML;
    }

    public function timbrarTFD(string $apiKey, string $xmlCfdi): RespuestaTimbradoTFD {
        $respuesta = new RespuestaTimbradoTFD();
        
        try {
            $options = [
                'trace' => 1,
                'exceptions' => true,
                'cache_wsdl' => WSDL_CACHE_NONE
            ];

            $soapClient = new SoapClient($this->wsdlUrl, $options);
            
            $params = [
                'apikey'  => $apiKey,
                'xmlCFDI' => $xmlCfdi
            ];
            
            // Llamar a la operación 'timbrarTFD'
            $response = $soapClient->timbrarTFD($params);
            
            $respuesta->code = $response->return->code ?? null;
            $respuesta->message = $response->return->message ?? null;
            $respuesta->data = $response->return->data ?? null;

        } catch (SoapFault $e) {
            error_log("Error SOAP: " . $e->getMessage());
            $respuesta->code = "FAULT";
            $respuesta->message = "Error en la comunicación SOAP: " . $e->getMessage();
        }
        
        return $respuesta;
    }
}

// --- Archivo: index.php (Ejemplo de uso) ---
// require_once 'TimbradoTFD.php';

$wsdlDesarrollo = "https://dev.facturaloplus.com/ws/servicio.do?wsdl";
$timbradoService = new TimbradoService($wsdlDesarrollo);

$miApiKey = getenv('TIMBRADO_API_KEY') ?: "TU_API_KEY_AQUI";
$xmlParaTimbrar = $timbradoService->generarXmlCfdiEjemplo();

header('Content-Type: text/plain; charset=utf-8');

$resultado = $timbradoService->timbrarTFD($miApiKey, $xmlParaTimbrar);

if ($resultado->code === '200') {
    echo "¡Timbrado para TFD Exitoso!\n";
    echo "Mensaje: {$resultado->message}\n";
    echo "\n--- XML del Timbre Fiscal Digital (TFD) ---\n";
    echo $resultado->data;
} else {
    echo "Error durante el timbrado TFD:\n";
    echo "Código: {$resultado->code}\n";
    echo "Mensaje: {$resultado->message}\n";
}
?>
```
  
  {{< /tab >}}

{{< /tabs >}}

#### Respuesta (Response)

El campo `data` contendrá únicamente el XML del Timbre Fiscal Digital (TFD).

```xml
<?xml version="1.0" encoding="UTF-8"?>
<SOAP-ENV:Envelope SOAP-ENV:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/"
xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/"
xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-
instance" xmlns:SOAP-ENC="http://schemas.xmlsoap.org/soap/encoding/"
xmlns:tns="urn:ws_api">
   <SOAP-ENV:Body>
      <ns1:timbrarTFDResponse xmlns:ns1="urn:ws_api">
         <return xsi:type="tns:RespuestaTimbrado">
            <code xsi:type="xsd:string">CÓDIGO</code>
            <message xsi:type="xsd:string">MENSAJE</message>
            <data xsi:type="xsd:string">
              <![CDATA[CFDI TIMBRADO]]>
            </data>
         </return>
      </ns1:timbrarTFDResponse>
   </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

##### Códigos de respuesta
{{< codigos-timbrado >}}
