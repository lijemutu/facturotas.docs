# Endpoints Timbrado

Esta sección detalla la operación del servicio de timbrado, permitiendo procesar y obtener el XML timbrado de un Comprobante Fiscal Digital (CFDI).

## timbrar

### Descripción de la Operación

Esta operación permite timbrar un CFDI en sus versiones 3.3 o 4.0 y retornar el XML timbrado completo del comprobante fiscal.

### Parámetros de Entrada (Input)

| Parámetro | Tipo de Dato | Descripción                                       |
| :-------- | :----------- | :------------------------------------------------ |
| `apikey`  | `string`     | Credencial de acceso al servicio (32 caracteres). |
| `xmlCFDI` | `string`     | Contenido del documento XML del CFDI v3.3 o v4.0. |

### Parámetros de Salida (Output) - RespuestaTimbrado

| Atributo  | Tipo de Dato | Descripción                             |
| :-------- | :----------- | :-------------------------------------- |
| `code`    | `string`     | Código de respuesta de la operación.    |
| `message` | `string`     | Mensaje detallado de la respuesta.      |
| `data`    | `string`     | XML del CFDI timbrado en caso de éxito. |

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
   /// Agregar los parámetros necesarios
   /// <summary>
   /// Genera un XML de ejemplo para un CFDI 4.0.
   /// En una aplicación real, este XML se construiría dinámicamente.
   /// </summary>
   /// <returns>Un string con el contenido del CFDI.</returns>
   private static string TimbrarCfdi() => """ 
        <?xml version="1.0" encoding="UTF-8"?>
        <cfdi:Comprobante
            xmlns:cfdi="http://www.sat.gob.mx/cfd/4"
            Version="4.0"
            Serie="F"
            Folio="123"
            Fecha="2025-07-02T12:00:00"
            SubTotal="100.00"
            Moneda="MXN"
            Total="116.00"
            TipoDeComprobante="I"
            Exportacion="01"
            MetodoPago="PUE"
            FormaPago="01"
            LugarExpedicion="64000">
            <cfdi:Emisor Rfc="ABC010101XYZ" Nombre="Empresa Ejemplo" RegimenFiscal="601"/>
            <cfdi:Receptor Rfc="XAXX010101000" Nombre="Publico en General" 
                          DomicilioFiscalReceptor="64000" RegimenFiscalReceptor="616" UsoCFDI="G03"/>
            <cfdi:Conceptos>
                <cfdi:Concepto ClaveProdServ="84111506" Cantidad="1" ClaveUnidad="ACT" 
                              Descripcion="Servicio de Ejemplo" ValorUnitario="100.00" Importe="100.00">
                    <cfdi:Impuestos>
                        <cfdi:Traslados>
                            <cfdi:Traslado Base="100.00" Impuesto="002" TipoFactor="Tasa" 
                                         TasaOCuota="0.160000" Importe="16.00"/>
                        </cfdi:Traslados>
                    </cfdi:Impuestos>
                </cfdi:Concepto>
            </cfdi:Conceptos>
            <cfdi:Impuestos TotalImpuestosTrasladados="16.00">
                <cfdi:Traslados>
                    <cfdi:Traslado Impuesto="002" TipoFactor="Tasa" TasaOCuota="0.160000" Importe="16.00"/>
                </cfdi:Traslados>
            </cfdi:Impuestos>
        </cfdi:Comprobante>
        """;

   /// <summary>
   /// Invoca al servicio web para timbrar un CFDI de forma asíncrona.
   /// </summary>
   /// <param name="apiKey">La credencial de acceso al servicio.</param>
   /// <param name="xmlCfdi">El contenido del CFDI a timbrar.</param>
   /// <returns>Un objeto RespuestaTimbrado con el resultado de la operación.</returns>
   public async Task<RespuestaTimbrado> TimbrarAsync(string apiKey)
    {
        using var client = new ServicioTimbradoWSPortTypeClient("ServicioTimbradoWSPort");
        
        try
        {            
            var response = await client.timbrarAsync(apiKey, TimbrarCfdi());
                        
            return new RespuestaTimbrado
            {
                Code = response.code,
                Message = response.message,
                Data = response.data
            };
        }
        catch (FaultException<RespuestaTimbrado> ex)
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
   public class RespuestaTimbrado
   {
      public string? Code { get; set; }
      public string? Message { get; set; }
      public string? Data { get; set; }
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
// --- Archivo: RespuestaTimbrado.java ---
package com.facturaloplus.cliente;

// POJO (Plain Old Java Object) para encapsular la respuesta.
public class RespuestaTimbrado {
    private String code;
    private String message;
    private String data;

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
    // Se recomienda un ExecutorService para manejar los hilos de las tareas asíncronas.
    private final ExecutorService executor = Executors.newFixedThreadPool(5);

    public String generarXmlCfdiEjemplo() {
        // Usar la fecha y hora actual en formato UTC, requerido por el SAT.
        String fechaActual = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd'T'HH:mm:ss"));

        // Uso de Text Blocks (Java 15+) para el XML.
        // Nota: Se agregó ObjetoImp="02" en el concepto, requerido en CFDI 4.0.
        return """
            <?xml version="1.0" encoding="UTF-8"?>
            <cfdi:Comprobante
                xmlns:cfdi="http://www.sat.gob.mx/cfd/4"
                Version="4.0"
                Serie="F"
                Folio="123"
                Fecha="%s"
                SubTotal="100.00"
                Moneda="MXN"
                Total="116.00"
                TipoDeComprobante="I"
                Exportacion="01"
                MetodoPago="PUE"
                FormaPago="01"
                LugarExpedicion="64000">
                <cfdi:Emisor Rfc="ABC010101XYZ" Nombre="Empresa Ejemplo" RegimenFiscal="601"/>
                <cfdi:Receptor Rfc="XAXX010101000" Nombre="Publico en General"
                                DomicilioFiscalReceptor="64000" RegimenFiscalReceptor="616" UsoCFDI="G03"/>
                <cfdi:Conceptos>
                    <cfdi:Concepto ClaveProdServ="84111506" Cantidad="1" ClaveUnidad="ACT" ObjetoImp="02"
                                   Descripcion="Servicio de Ejemplo" ValorUnitario="100.00" Importe="100.00">
                        <cfdi:Impuestos>
                            <cfdi:Traslados>
                                <cfdi:Traslado Base="100.00" Impuesto="002" TipoFactor="Tasa"
                                               TasaOCuota="0.160000" Importe="16.00"/>
                            </cfdi:Traslados>
                        </cfdi:Impuestos>
                    </cfdi:Concepto>
                </cfdi:Conceptos>
                <cfdi:Impuestos TotalImpuestosTrasladados="16.00">
                    <cfdi:Traslados>
                        <cfdi:Traslado Impuesto="002" TipoFactor="Tasa" TasaOCuota="0.160000" Importe="16.00"/>
                    </cfdi:Traslados>
                </cfdi:Impuestos>
            </cfdi:Comprobante>
            """.formatted(fechaActual);
    }

    public CompletableFuture<RespuestaTimbrado> timbrarAsync(String apiKey, String xmlCfdi) {
        return CompletableFuture.supplyAsync(() -> {
            try {
                // Las clases ServicioTimbradoWS y ServicioTimbradoWSPortType son generadas por wsimport.
                ServicioTimbradoWS service = new ServicioTimbradoWS();
                ServicioTimbradoWSPortType port = service.getServicioTimbradoWSPort();

                logger.info("Iniciando timbrado para CFDI.");
                Respuesta response = port.timbrar(apiKey, xmlCfdi); // 'Respuesta' es la clase generada por wsimport.

                RespuestaTimbrado resultado = new RespuestaTimbrado();
                resultado.setCode(response.getCode());
                resultado.setMessage(response.getMessage());
                resultado.setData(response.getData());
                return resultado;

            } catch (SOAPFaultException ex) {
                logger.log(Level.SEVERE, "Error SOAP (SOAPFaultException): " + ex.getFault().getFaultString(), ex);
                throw new RuntimeException("Error del servicio: " + ex.getFault().getFaultString(), ex);
            } catch (WebServiceException ex) {
                logger.log(Level.SEVERE, "Error de comunicación con el servicio de timbrado.", ex);
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

        // El API Key debe obtenerse de una fuente segura, no estar en el código.
        String miApiKey = "TU_API_KEY_AQUI";
        String xmlParaTimbrar = timbradoService.generarXmlCfdiEjemplo();

        System.out.println("--- Intentando timbrar el siguiente CFDI: ---");
        System.out.println(xmlParaTimbrar);
        System.out.println("------------------------------------------");

        CompletableFuture<RespuestaTimbrado> future = timbradoService.timbrarAsync(miApiKey, xmlParaTimbrar);

        future.whenComplete((resultado, ex) -> {
            if (ex != null) {
                System.err.println("\nOcurrió una excepción no controlada: " + ex.getMessage());
            } else {
                if ("200".equals(resultado.getCode())) {
                    System.out.println("\n¡Timbrado Exitoso!");
                    System.out.println("Mensaje: " + resultado.getMessage());
                    System.out.println("\n--- XML Timbrado ---");
                    System.out.println(resultado.getData());
                } else {
                    System.err.println("\nError durante el timbrado:");
                    System.err.println("Código: " + resultado.getCode());
                    System.err.println("Mensaje: " + resultado.getMessage());
                }
            }
            // Es importante cerrar el ExecutorService cuando la aplicación termina.
            timbradoService.shutdown();
        });
        
        // Mantiene el programa principal vivo mientras se completa la tarea asíncrona.
        // En una aplicación real (ej. un servidor de aplicaciones), esto no sería necesario.
        future.join(); 
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
# --- Archivo: timbrado_service.py ---
import asyncio
import logging
from datetime import datetime
from zeep import Client
from zeep.asyncio import AsyncClient
from zeep.exceptions import Fault, TransportError
from dataclasses import dataclass

# Configuración básica de logging
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

WSDL_URL_DESARROLLO = "https://dev.facturaloplus.com/ws/servicio.do?wsdl"

@dataclass
class RespuestaTimbrado:
    """Clase de datos para encapsular la respuesta del servicio."""
    code: str = None
    message: str = None
    data: str = None

class TimbradoService:
    """Clase que encapsula la lógica para interactuar con el servicio de timbrado."""

    def __init__(self, wsdl_url: str):
        self.wsdl_url = wsdl_url
        # El cliente asíncrono se crea al momento de usarlo
        self.async_client = AsyncClient(self.wsdl_url)

    def generar_xml_cfdi_ejemplo(self) -> str:
        """
        Genera un XML de ejemplo para un CFDI 4.0.
        En una aplicación real, este XML se construiría dinámicamente.
        """
        # Usar la fecha y hora actual en formato UTC, requerido por el SAT.
        fecha_actual = datetime.utcnow().strftime('%Y-%m-%dT%H:%M:%S')
        
        # Uso de f-strings para formatear el XML.
        # Nota: Se agregó ObjetoImp="02" en el concepto, requerido en CFDI 4.0.
        return f"""
<?xml version="1.0" encoding="UTF-8"?>
<cfdi:Comprobante
    xmlns:cfdi="http://www.sat.gob.mx/cfd/4"
    Version="4.0"
    Serie="F"
    Folio="123"
    Fecha="{fecha_actual}"
    SubTotal="100.00"
    Moneda="MXN"
    Total="116.00"
    TipoDeComprobante="I"
    Exportacion="01"
    MetodoPago="PUE"
    FormaPago="01"
    LugarExpedicion="64000">
    <cfdi:Emisor Rfc="ABC010101XYZ" Nombre="Empresa Ejemplo" RegimenFiscal="601"/>
    <cfdi:Receptor Rfc="XAXX010101000" Nombre="Publico en General"
                    DomicilioFiscalReceptor="64000" RegimenFiscalReceptor="616" UsoCFDI="G03"/>
    <cfdi:Conceptos>
        <cfdi:Concepto ClaveProdServ="84111506" Cantidad="1" ClaveUnidad="ACT" ObjetoImp="02"
                       Descripcion="Servicio de Ejemplo" ValorUnitario="100.00" Importe="100.00">
            <cfdi:Impuestos>
                <cfdi:Traslados>
                    <cfdi:Traslado Base="100.00" Impuesto="002" TipoFactor="Tasa"
                                   TasaOCuota="0.160000" Importe="16.00"/>
                </cfdi:Traslados>
            </cfdi:Impuestos>
        </cfdi:Concepto>
    </cfdi:Conceptos>
    <cfdi:Impuestos TotalImpuestosTrasladados="16.00">
        <cfdi:Traslados>
            <cfdi:Traslado Impuesto="002" TipoFactor="Tasa" TasaOCuota="0.160000" Importe="16.00"/>
        </cfdi:Traslados>
    </cfdi:Impuestos>
</cfdi:Comprobante>
"""

    async def timbrar_async(self, api_key: str, xml_cfdi: str) -> RespuestaTimbrado:
        """Invoca al servicio web para timbrar un CFDI de forma asíncrona."""
        try:
            logging.info("Iniciando timbrado para CFDI.")
            # El nombre del servicio 'ServicioTimbradoWSPort' y la operación 'timbrar'
            # son definidos por el WSDL.
            response = await self.async_client.service.timbrar(apikey=api_key, xmlCFDI=xml_cfdi)
            
            # Zeep convierte la respuesta en un objeto similar a un diccionario.
            return RespuestaTimbrado(
                code=response.code,
                message=response.message,
                data=response.data
            )
        except Fault as ex:
            logging.error(f"Error SOAP (Fault): Code={ex.code}, Message={ex.message}")
            raise ConnectionError(f"Error del servicio: {ex.message}") from ex
        except TransportError as ex:
            logging.error(f"Error de comunicación con el servicio de timbrado: {ex}")
            raise

# --- Archivo: main.py (Ejemplo de uso) ---
async def main():
    """Función principal para ejecutar el ejemplo."""
    service = TimbradoService(WSDL_URL_DESARROLLO)
    
    # El API Key debe obtenerse de una fuente segura, no estar en el código.
    mi_api_key = "TU_API_KEY_AQUI"
    xml_para_timbrar = service.generar_xml_cfdi_ejemplo()

    print("--- Intentando timbrar el siguiente CFDI: ---")
    print(xml_para_timbrar)
    print("------------------------------------------")

    try:
        resultado = await service.timbrar_async(mi_api_key, xml_para_timbrar)
        
        if resultado.code == "200":
            print("\033[92m\n¡Timbrado Exitoso!\033[0m") # Color verde
            print(f"Mensaje: {resultado.message}")
            print("\n--- XML Timbrado ---")
            print(resultado.data)
        else:
            print("\033[91m\nError durante el timbrado:\033[0m") # Color rojo
            print(f"Código: {resultado.code}")
            print(f"Mensaje: {resultado.message}")

    except Exception as ex:
        print(f"\033[91m\nOcurrió una excepción no controlada: {ex}\033[0m")

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
// --- Archivo: Timbrado.php ---

// Habilitar la visualización de errores para depuración
ini_set('display_errors', 1);
error_reporting(E_ALL);

/**
 * Clase DTO (Data Transfer Object) para encapsular la respuesta del servicio.
 */
class RespuestaTimbrado {
    public ?string $code = null;
    public ?string $message = null;
    public ?string $data = null;
}

/**
 * Clase que encapsula la lógica para interactuar con el servicio de timbrado.
 */
class TimbradoService {
    private string $wsdlUrl;

    public function __construct(string $wsdlUrl) {
        $this->wsdlUrl = $wsdlUrl;
    }

    /**
     * Genera un XML de ejemplo para un CFDI 4.0.
     */
    public function generarXmlCfdiEjemplo(): string {
        // Usar la fecha y hora actual en formato UTC, requerido por el SAT.
        $fechaActual = gmdate('Y-m-d\TH:i:s');

        // Nota: Se agregó ObjetoImp="02" en el concepto, requerido en CFDI 4.0.
        return <<<XML
<?xml version="1.0" encoding="UTF-8"?>
<cfdi:Comprobante
    xmlns:cfdi="http://www.sat.gob.mx/cfd/4"
    Version="4.0"
    Serie="F"
    Folio="123"
    Fecha="{$fechaActual}"
    SubTotal="100.00"
    Moneda="MXN"
    Total="116.00"
    TipoDeComprobante="I"
    Exportacion="01"
    MetodoPago="PUE"
    FormaPago="01"
    LugarExpedicion="64000">
    <cfdi:Emisor Rfc="ABC010101XYZ" Nombre="Empresa Ejemplo" RegimenFiscal="601"/>
    <cfdi:Receptor Rfc="XAXX010101000" Nombre="Publico en General"
                    DomicilioFiscalReceptor="64000" RegimenFiscalReceptor="616" UsoCFDI="G03"/>
    <cfdi:Conceptos>
        <cfdi:Concepto ClaveProdServ="84111506" Cantidad="1" ClaveUnidad="ACT" ObjetoImp="02"
                       Descripcion="Servicio de Ejemplo" ValorUnitario="100.00" Importe="100.00">
            <cfdi:Impuestos>
                <cfdi:Traslados>
                    <cfdi:Traslado Base="100.00" Impuesto="002" TipoFactor="Tasa"
                                   TasaOCuota="0.160000" Importe="16.00"/>
                </cfdi:Traslados>
            </cfdi:Impuestos>
        </cfdi:Concepto>
    </cfdi:Conceptos>
    <cfdi:Impuestos TotalImpuestosTrasladados="16.00">
        <cfdi:Traslados>
            <cfdi:Traslado Impuesto="002" TipoFactor="Tasa" TasaOCuota="0.160000" Importe="16.00"/>
        </cfdi:Traslados>
    </cfdi:Impuestos>
</cfdi:Comprobante>
XML;
    }

    /**
     * Invoca al servicio web para timbrar un CFDI.
     */
    public function timbrar(string $apiKey, string $xmlCfdi): RespuestaTimbrado {
        $respuesta = new RespuestaTimbrado();
        
        try {
            // Opciones para el cliente SOAP. 'trace' es útil para depurar.
            $options = [
                'trace' => 1,
                'exceptions' => true, // Lanza SoapFault en caso de error
                'cache_wsdl' => WSDL_CACHE_NONE
            ];

            // Crear el cliente SOAP
            $soapClient = new SoapClient($this->wsdlUrl, $options);
            
            // Los parámetros deben estar en un array asociativo
            $params = [
                'apikey'  => $apiKey,
                'xmlCFDI' => $xmlCfdi
            ];
            
            // Llamar a la operación 'timbrar' del servicio
            $response = $soapClient->timbrar($params);
            
            $respuesta->code = $response->return->code ?? null;
            $respuesta->message = $response->return->message ?? null;
            $respuesta->data = $response->return->data ?? null;

        } catch (SoapFault $e) {
            // Capturar errores específicos de SOAP
            error_log("Error SOAP (SoapFault): " . $e->getMessage());
            $respuesta->code = "FAULT";
            $respuesta->message = "Error en la comunicación SOAP: " . $e->getMessage();
        } catch (Exception $e) {
            // Capturar otros errores
            error_log("Error general: " . $e->getMessage());
            $respuesta->code = "EXCEPTION";
            $respuesta->message = "Ocurrió una excepción: " . $e->getMessage();
        }
        
        return $respuesta;
    }
}

// --- Archivo: index.php (Ejemplo de uso) ---

// Incluir la clase del servicio
// require_once 'Timbrado.php';

$wsdlDesarrollo = "https://dev.facturaloplus.com/ws/servicio.do?wsdl";
$timbradoService = new TimbradoService($wsdlDesarrollo);

// El API Key debe obtenerse de una fuente segura, como una variable de entorno.
$miApiKey = getenv('TIMBRADO_API_KEY') ?: "TU_API_KEY_AQUI";
$xmlParaTimbrar = $timbradoService->generarXmlCfdiEjemplo();

// Imprimir en formato de texto plano para una mejor visualización en consola o web
header('Content-Type: text/plain; charset=utf-8');

echo "--- Intentando timbrar el siguiente CFDI: ---\n";
echo $xmlParaTimbrar;
echo "\n------------------------------------------\n\n";

$resultado = $timbradoService->timbrar($miApiKey, $xmlParaTimbrar);

if ($resultado->code === '200') {
    echo "¡Timbrado Exitoso!\n";
    echo "Mensaje: {$resultado->message}\n";
    echo "\n--- XML Timbrado ---\n";
    echo $resultado->data;
} else {
    echo "Error durante el timbrado:\n";
    echo "Código: {$resultado->code}\n";
    echo "Mensaje: {$resultado->message}\n";
}
?>
```
  
  {{< /tab >}}

{{< /tabs >}}

