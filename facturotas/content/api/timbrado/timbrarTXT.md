---
title: Timbrar TXT
---

Esta sección detalla la operación del servicio de timbrado a partir de un layout en formato de texto plano (TXT), permitiendo generar, sellar y obtener el XML timbrado de un Comprobante Fiscal Digital (CFDI) de forma simplificada.

### Descripción de la Operación

Esta operación facilita la creación de un CFDI (versión 4.0) a partir de un archivo de texto plano (`.txt`). El servicio interpreta el layout, construye la estructura del XML, lo sella con los certificados proporcionados y devuelve el XML timbrado completo, junto con su folio fiscal (UUID) y el sello digital del SAT.

### Parámetros de Entrada (Input)

| Parámetro | Tipo de Dato | Descripción                                                                                                |
| :-------- | :----------- | :--------------------------------------------------------------------------------------------------------- |
| `apikey`  | `string`     | Credencial de acceso al servicio ([Solicita aquí](../../../#Requisitos)).                                      |
| `txtB64` | `string`     | Cadena en formato Base64 que contiene el layout TXT con la información del CFDI a generar (ver estructura abajo). [Conversor Base 64](../../../herramientas/convertidorBase64) |
| `keyPEM`  | `string`     | Contenido del archivo de la llave privada (`.key`) en formato PEM.                                         |
| `cerPEM`  | `string`     | Contenido del archivo del certificado de llave pública (`.cer`) en formato PEM.                              |

### Parámetros de Salida (Output) - RespuestaTimbrado

La respuesta de esta operación es idéntica a la de la operación `timbrar`.

| Atributo  | Tipo de Dato | Descripción                             |
| :-------- | :----------- | :-------------------------------------- |
| `code`    | `string`     | Código de respuesta de la operación.    |
| `message` | `string`     | Mensaje detallado de la respuesta.      |
| `data`    | `string`     | XML del CFDI timbrado en caso de éxito. |

### Estructura del TXT de Entrada

El TXT enviado en el parámetro `txtB64` debe seguir la siguiente estructura de "pipe" (`|`). Cada línea representa una sección del CFDI y los campos se separan por `|`. Los valores vacíos se indican con un `|` consecutivo. [Conversor Base 64](../../../herramientas/convertidorBase64)

```txt
COMPROBANTE|4.0|Serie|Folio|Fecha|FormaPago|NoCertificado|CondicionesDePago|SubTotal|Descuento|Moneda|TipoCambio|Total|TipoDeComprobante|Exportacion|MetodoPago|LugarExpedicion|Confirmacion|
INFORMACIONGLOBAL|Periodicidad|Meses|Año|
CFDIRELACIONADOS|numeroCfdiRelacionados|TipoRelacion|
CFDIRELACIONADO|UUID-1|UUID-2|...|UUID-N|
EMISOR|Rfc|Nombre|RegimenFiscal|FacAtrAdquirente|
RECEPTOR|Rfc|Nombre|DomicilioFiscalReceptor|ResidenciaFiscal|NumRegIdTrib|RegimenFiscalReceptor|UsoCFDI|
CONCEPTOS|numeroConceptos|
	CONCEPTO|ClaveProdServ|NoIdentificacion|Cantidad|ClaveUnidad|Unidad|Descripcion|ValorUnitario|Importe|Descuento|ObjetoImp|
		IMPUESTOSCONCEPTO|numeroTraslados|numeroRetenciones|
			TRASLADOCONCEPTO|Base|Impuesto|TipoFactor|TasaOCuota|Importe|
			RETENCIONCONCEPTO|Base|Impuesto|TipoFactor|TasaOCuota|Importe|
		ACUENTATERCEROS|RfcACuentaTerceros|NombreACuentaTerceros|RegimenFiscalACuentaTerceros|DomicilioFiscalACuentaTerceros|
		INFORMACIONADUANERA|numeroNumerosPedimento|
			NUMEROSPEDIMENTO|pedimento-1|pedimento-2|...|pedimento-N|
		CUENTAPREDIAL|numeroNumeros|
			NUMEROS|numero-1|numero-2|...|numero-N|
IMPUESTOS|numeroTraslados|numeroRetenciones|TotalImpuestosTrasladados|TotalImpuestosRetenidos|
	TRASLADO|Base|Impuesto|TipoFactor|TasaOCuota|Importe|
	RETENCION|Impuesto|Importe|
CORREO|correo1|correo2|...|correoN|
CAMPOS_EXTRA|numeroCamposExtra|
	CAMPO_EXTRA|llave|valor|descripcion|
```

**Consulta el archivo completo aquí:**  
[`TXT`](../layout_cfdi40.txt)

{{< callout type="warning" >}}
  El NoCertificado se puede encontrar en la FIEL como NO. Serie
{{< /callout >}}

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
   using System.Text;
   using System.Threading.Tasks;
   
   // ... (Asegúrate de tener la referencia al cliente SOAP generado por svcutil)

    /// <summary>
    /// Genera dinámicamente el layout del CFDI en formato TXT.
    /// En una aplicación real, los valores se obtendrían de una base de datos o de la entrada del usuario.
    /// </summary>
    /// <returns>Un string con el TXT del CFDI.</returns>
    private static string GetTxtLayout()
    {
        var sb = new StringBuilder();
        sb.AppendLine("COMPROBANTE|4.0|F|123|2025-07-04T12:00:00|01|||100.00|0.00|MXN|1|116.00|I|01|PUE|64000|");
        sb.AppendLine("EMISOR|ABC010101XYZ|Empresa Ejemplo SA de CV|601|");
        sb.AppendLine("RECEPTOR|XAXX010101000|Publico en General|64000|||616|G03");
        sb.AppendLine("CONCEPTOS|1");
        sb.AppendLine("	CONCEPTO|01010101||1.0|ACT|Actividad|Producto de prueba|100.00|100.00||02");
        sb.AppendLine("		IMPUESTOSCONCEPTO|1|0");
        sb.AppendLine("			TRASLADOCONCEPTO|100.00|002|Tasa|0.160000|16.00");
        sb.AppendLine("IMPUESTOS|1|0|16.00|");
        sb.AppendLine("	TRASLADO|100.00|002|Tasa|0.160000|16.00");
        return sb.ToString();
    }

   /// <summary>
   /// Prepara y codifica los datos para la operación timbrarTXT.
   /// </summary>
   public async Task<RespuestaTimbrado> TimbrarTxtAsync(string apiKey, string txtLayout, string keyPemPath, string cerPemPath)
   {
       // 1. Codificar el TXT a Base64
       var txtBytes = Encoding.UTF8.GetBytes(txtLayout);
       var txtB64 = Convert.ToBase64String(txtBytes);

       // 2. Leer el contenido de los archivos PEM
       var keyPem = await File.ReadAllTextAsync(keyPemPath);
       var cerPem = await File.ReadAllTextAsync(cerPemPath);

       // 3. Invocar al servicio web
       using var client = new ServicioTimbradoWSPortTypeClient("ServicioTimbradoWSPort");
       try
       {
           // Asumiendo que la operación en el WSDL se llama 'timbrarTXT'
           var response = await client.timbrarTXTAsync(apiKey, txtB64, keyPem, cerPem);
           
           return new RespuestaTimbrado
           {
               Code = response.code,
               Message = response.message,
               Data = response.data
           };
       }
       catch (Exception ex)
       {
           // Manejo de errores (FaultException, etc.)
           Console.WriteLine($"Error al timbrar con TXT: {ex.Message}");
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
   public async Task EjemploUsoTimbrarTxtAsync()
   {
       string apiKey = "TU_API_KEY_AQUI";
       string txtLayout = GetTxtLayout(); // Carga el layout dinámicamente
       string keyPemFile = "ruta/a/tu/llave.key.pem";
       string cerPemFile = "ruta/a/tu/certificado.cer.pem";

       var resultado = await TimbrarTxtAsync(apiKey, txtLayout, keyPemFile, cerPemFile);

       if (resultado?.Code == "200")
       {
           Console.WriteLine("¡Timbrado con TXT Exitoso!");
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
  import java.nio.charset.StandardCharsets;
  import java.nio.file.Files;
  import java.nio.file.Paths;
  import java.util.Base64;
  import java.util.concurrent.CompletableFuture;

  // ... (Asegúrate de tener las clases generadas por wsimport)

  public class TimbradoTxtService {

      // ... (Definición de ExecutorService y Logger)

    /**
     * Genera dinámicamente el layout del CFDI en formato TXT.
     * En una aplicación real, los valores se obtendrían de una base de datos o de la entrada del usuario.
     */
    public static String getTxtLayout() {
        return String.join(System.lineSeparator(),
            "COMPROBANTE|4.0|F|123|2025-07-04T12:00:00|01|||100.00|0.00|MXN|1|116.00|I|01|PUE|64000|",
            "EMISOR|ABC010101XYZ|Empresa Ejemplo SA de CV|601|",
            "RECEPTOR|XAXX010101000|Publico en General|64000|||616|G03",
            "CONCEPTOS|1",
            "	CONCEPTO|01010101||1.0|ACT|Actividad|Producto de prueba|100.00|100.00||02",
            "		IMPUESTOSCONCEPTO|1|0",
            "			TRASLADOCONCEPTO|100.00|002|Tasa|0.160000|16.00",
            "IMPUESTOS|1|0|16.00|",
            "	TRASLADO|100.00|002|Tasa|0.160000|16.00"
        );
    }

      public CompletableFuture<RespuestaTimbrado> timbrarTxtAsync(String apiKey, String txtLayout, String keyPemPath, String cerPemPath) {
          return CompletableFuture.supplyAsync(() -> {
              try {
                  // 1. Codificar TXT a Base64
                  String txtB64 = Base64.getEncoder().encodeToString(txtLayout.getBytes(StandardCharsets.UTF_8));

                  // 2. Leer archivos PEM
                  String keyPem = new String(Files.readAllBytes(Paths.get(keyPemPath)));
                  String cerPem = new String(Files.readAllBytes(Paths.get(cerPemPath)));

                  // 3. Llamar al servicio
                  ServicioTimbradoWS service = new ServicioTimbradoWS();
                  ServicioTimbradoWSPortType port = service.getServicioTimbradoWSPort();
                  
                  // Asumiendo que la operación se llama 'timbrarTXT'
                  Respuesta response = port.timbrarTXT(apiKey, txtB64, keyPem, cerPem);

                  RespuestaTimbrado resultado = new RespuestaTimbrado();
                  resultado.setCode(response.getCode());
                  resultado.setMessage(response.getMessage());
                  resultado.setData(response.getData());
                  return resultado;

              } catch (Exception ex) {
                  // Manejo de errores
                  throw new RuntimeException("Error al timbrar con TXT", ex);
              }
          }, executor);
      }
  
  // Ejemplo de uso
  public static void main(String[] args) {
      TimbradoTxtService service = new TimbradoTxtService();
      String apiKey = "TU_API_KEY_AQUI";
      String keyPem = "ruta/a/tu/llave.key.pem";
      String cerPem = "ruta/a/tu/certificado.cer.pem";
      String txtLayout = getTxtLayout(); // Carga el layout dinámicamente

      service.timbrarTxtAsync(apiKey, txtLayout, keyPem, cerPem)
          .whenComplete((resultado, ex) -> {
              if (ex != null) {
                  System.err.println("Error: " + ex.getMessage());
              } else if ("200".equals(resultado.getCode())) {
                  System.out.println("¡Timbrado con TXT Exitoso!");
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
  ```****
  pip install zeep
  ```

  ### Implementación
  ```python
  import asyncio
  import base64
  from zeep.asyncio import AsyncClient

  # ... (Definición de RespuestaTimbrado y configuración de logging)

  def get_txt_layout() -> str:
      """
      Genera dinámicamente el layout del CFDI como un string de texto.
      En una aplicación real, los valores se obtendrían de una base de datos o de la entrada del usuario.
      """
      return "\n".join([
          "COMPROBANTE|4.0|F|123|2025-07-04T12:00:00|01|||100.00|0.00|MXN|1|116.00|I|01|PUE|64000|",
          "EMISOR|ABC010101XYZ|Empresa Ejemplo SA de CV|601|",
          "RECEPTOR|XAXX010101000|Publico en General|64000|||616|G03",
          "CONCEPTOS|1",
          "	CONCEPTO|01010101||1.0|ACT|Actividad|Producto de prueba|100.00|100.00||02",
          "		IMPUESTOSCONCEPTO|1|0",
          "			TRASLADOCONCEPTO|100.00|002|Tasa|0.160000|16.00",
          "IMPUESTOS|1|0|16.00|",
          "	TRASLADO|100.00|002|Tasa|0.160000|16.00"
      ])

  class TimbradoTxtService:
      def __init__(self, wsdl_url: str):
          self.wsdl_url = wsdl_url
          self.async_client = AsyncClient(self.wsdl_url)

      async def timbrar_txt_async(self, api_key: str, txt_layout: str, key_pem_path: str, cer_pem_path: str) -> RespuestaTimbrado:
          try:
              # 1. Convertir TXT string a Base64
              txt_b64 = base64.b64encode(txt_layout.encode('utf-8')).decode('utf-8')

              # 2. Leer archivos PEM
              with open(key_pem_path, 'r') as f:
                  key_pem = f.read()
              with open(cer_pem_path, 'r') as f:
                  cer_pem = f.read()

              # 3. Llamar al servicio
              # Asumiendo que la operación se llama 'timbrarTXT'
              response = await self.async_client.service.timbrarTXT(
                  apikey=api_key,
                  txtB64=txt_b64,
                  keyPEM=key_pem,
                  cerPEM=cer_pem
              )
              return RespuestaTimbrado(
                  code=response.code,
                  message=response.message,
                  data=response.data
              )
          except Exception as e:
              logging.error(f"Error al timbrar con TXT: {e}")
              raise

  # Ejemplo de uso
  async def main():
      service = TimbradoTxtService("https://dev.facturaloplus.com/ws/servicio.do?wsdl")
      api_key = "TU_API_KEY_AQUI";
      key_pem = "ruta/a/tu/llave.key.pem";
      cer_pem = "ruta/a/tu/certificado.cer.pem";
      
      layout = get_txt_layout(); // Carga el layout dinámicamente

      resultado = await service.timbrar_txt_async(api_key, layout, key_pem, cer_pem);

      if resultado.code == "200":
          print("¡Timbrado con TXT Exitoso!");
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
  ```php
  <?php
  // ... (Definición de la clase RespuestaTimbrado)

    /**
     * Genera dinámicamente el layout del CFDI como un string TXT.
     * En una aplicación real, los valores se obtendrían de una base de datos.
     */
    function getTxtLayout(): string {
        return implode("\n", [
            "COMPROBANTE|4.0|F|123|2025-07-04T12:00:00|01|||100.00|0.00|MXN|1|116.00|I|01|PUE|64000|",
            "EMISOR|ABC010101XYZ|Empresa Ejemplo SA de CV|601|",
            "RECEPTOR|XAXX010101000|Publico en General|64000|||616|G03",
            "CONCEPTOS|1",
            "	CONCEPTO|01010101||1.0|ACT|Actividad|Producto de prueba|100.00|100.00||02",
            "		IMPUESTOSCONCEPTO|1|0",
            "			TRASLADOCONCEPTO|100.00|002|Tasa|0.160000|16.00",
            "IMPUESTOS|1|0|16.00|",
            "	TRASLADO|100.00|002|Tasa|0.160000|16.00"
        ]);
    }

  class TimbradoTxtService {
      private string $wsdlUrl;

      public function __construct(string $wsdlUrl) {
          $this->wsdlUrl = $wsdlUrl;
      }

      public function timbrarTxt(string $apiKey, string $txtLayout, string $keyPemPath, string $cerPemPath): RespuestaTimbrado {
          $respuesta = new RespuestaTimbrado();
          try {
              // 1. Codificar TXT a Base64
              $txtB64 = base64_encode($txtLayout);

              // 2. Leer archivos PEM
              $keyPem = file_get_contents($keyPemPath);
              $cerPem = file_get_contents($cerPemPath);

              if ($keyPem === false || $cerPem === false) {
                  throw new Exception("No se pudieron leer los archivos .pem");
              }

              // 3. Llamar al servicio
              $soapClient = new SoapClient($this->wsdlUrl, ['trace' => 1, 'exceptions' => true]);
              
              $params = [
                  'apikey' => $apiKey,
                  'txtB64' => $txtB64,
                  'keyPEM' => $keyPem,
                  'cerPEM' => $cerPem
              ];

              // Asumiendo que la operación se llama 'timbrarTXT'
              $response = $soapClient->timbrarTXT($params);
              
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
  $service = new TimbradoTxtService("https://dev.facturaloplus.com/ws/servicio.do?wsdl");
  $apiKey = "TU_API_KEY_AQUI";
  $keyPem = "ruta/a/tu/llave.key.pem";
  $cerPem = "ruta/a/tu/certificado.cer.pem";
  $txtLayout = getTxtLayout(); // Carga el layout dinámicamente

  $resultado = $service->timbrarTxt($apiKey, $txtLayout, $keyPem, $cerPem);

  header('Content-Type: text/plain');
  if ($resultado->code === '200') {
      echo "¡Timbrado con TXT Exitoso!\n";
      echo $resultado->data;
  } else {
      echo "Error: {$resultado->code} - {$resultado->message}\n";
  }
  ?>
  ```
  {{< /tab >}}

{{< /tabs >}}



#### Respuesta (Response)

La estructura de la respuesta SOAP es la misma que la de la operación `timbrar`. El contenido de la etiqueta `data` será el XML del CFDI timbrado.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<SOAP-ENV:Envelope ...>
   <SOAP-ENV:Body>
      <ns1:timbrarTXTResponse xmlns:ns1="urn:ws_api">
         <return xsi:type="tns:RespuestaTimbrado">
            <code xsi:type="xsd:string">CÓDIGO</code>
            <message xsi:type="xsd:string">MENSAJE</message>
            <data xsi:type="xsd:string"><![CDATA[CFDI TIMBRADO]]></data>
         </return>
      </ns1:timbrarTXTResponse>
   </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

##### Códigos de respuesta
{{< codigos-timbrado >}}
