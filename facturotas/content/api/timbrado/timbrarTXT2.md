---
title: Timbrar TXT + PDF
---

Esta operación extiende la funcionalidad de `timbrarTXT`, permitiendo no solo generar y timbrar un CFDI a partir de un layout de texto plano (TXT), sino también obtener una representación en formato PDF del comprobante, personalizada con una plantilla y un logotipo.

### Descripción de la Operación

Esta operación toma un layout TXT, lo convierte en un CFDI (versión 4.0), lo sella con los certificados proporcionados y lo timbra. Como resultado, devuelve una estructura JSON que contiene tanto el XML del CFDI timbrado como el contenido del PDF generado en formato Base64.

### Parámetros de Entrada (Input)

| Parámetro   | Tipo de Dato | Descripción                                                                                                |
| :---------- | :----------- | :--------------------------------------------------------------------------------------------------------- |
| `apikey`    | `string`     | Credencial de acceso al servicio ([Solicita aquí](../../../#Requisitos)).                                      |
| `txtB64`    | `string`     | Cadena en formato Base64 que contiene el layout TXT con la información del CFDI a generar.  [Conversor Base 64](../../../herramientas/convertidorBase64)              |
| `keyPEM`    | `string`     | Contenido del archivo de la llave privada (`.key`) en formato PEM.                                         |
| `cerPEM`    | `string`     | Contenido del archivo del certificado de llave pública (`.cer`) en formato PEM.                              |
| `plantilla` | `string`     | Identificador numérico de la plantilla a utilizar para la generación del PDF.                              |
| `logob64`   | `string`     | Imagen (logo) de la empresa en formato PNG y codificada en Base64. [Conversor Base 64](../../../herramientas/convertidorBase64) |

### Parámetros de Salida (Output) - RespuestaTimbrado

| Atributo  | Tipo de Dato | Descripción                                                                                                                                |
| :-------- | :----------- | :----------------------------------------------------------------------------------------------------------------------------------------- |
| `code`    | `string`     | Código de respuesta de la operación.                                                                                                       |
| `message` | `string`     | Mensaje detallado de la respuesta.                                                                                                         |
| `data`    | `string`     | **JSON string** que contiene el XML del CFDI timbrado y el contenido del PDF en Base64. Debe ser decodificado para acceder a sus valores. |

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

<!-- ### Estructura del JSON en el campo `data`

El campo `data` de la respuesta contiene un JSON con la siguiente estructura:

```json
{
  "xml": "PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0iVVRGLTgiPz48Y2ZkaT...",
  "pdf": "JVBERi0xLjQKJdPr6eEKMSAwIG9iago8PC9UeXBlL0NhdGFsb2cvT3V0bGlu..."
}
```

- `xml`: String con el contenido del CFDI timbrado.
- `pdf`: String en formato Base64 con el contenido del archivo PDF. -->

### Ejemplo de Código

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
  // Se requiere la librería System.Text.Json
  using System.Text.Json;
  using System.Text;

    /// <summary>
    /// Genera dinámicamente el layout del CFDI en formato TXT.
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


  // Clase para deserializar la respuesta en el campo 'data'
  public class RespuestaData
  {
      public string Xml { get; set; }
      public string Pdf { get; set; }
  }

  public async Task<RespuestaTimbrado> TimbrarTxt2Async(string apiKey, string txtLayout, string keyPem, string cerPem, string plantilla, string logoB64)
  {
      var txtB64 = Convert.ToBase64String(Encoding.UTF8.GetBytes(txtLayout));
      using var client = new ServicioTimbradoWSPortTypeClient("ServicioTimbradoWSPort");
      
      // Asumiendo que la operación se llama 'timbrarTXT2'
      var response = await client.timbrarTXT2Async(apiKey, txtB64, keyPem, cerPem, plantilla, logoB64);
      return new RespuestaTimbrado { Code = response.code, Message = response.message, Data = response.data };
  }

  // Ejemplo de uso
  public async Task EjemploUsoTimbrarTxt2Async()
  {
      string apiKey = "TU_API_KEY_AQUI";
      string txtLayout = GetTxtLayout(); // Generación dinámica
      string keyPem = File.ReadAllText("ruta/a/llave.key.pem");
      string cerPem = File.ReadAllText("ruta/a/certificado.cer.pem");
      string plantilla = "1"; // ID de la plantilla a usar
      string logoB64 = Convert.ToBase64String(File.ReadAllBytes("ruta/a/logo.png"));


      var resultado = await TimbrarTxt2Async(apiKey, txtLayout, keyPem, cerPem, plantilla, logoB64);

      if (resultado?.Code == "200" && !string.IsNullOrEmpty(resultado.Data))
      {
          Console.WriteLine("¡Timbrado con TXT y PDF Exitoso!");
          
          // Deserializar el JSON de la respuesta
          var respuestaData = JsonSerializer.Deserialize<RespuestaData>(resultado.Data);
          
          Console.WriteLine("--- XML Timbrado ---");
          Console.WriteLine(respuestaData.Xml);

          // Guardar el PDF
          var pdfBytes = Convert.FromBase64String(respuestaData.Pdf);
          File.WriteAllBytes("comprobante.pdf", pdfBytes);
          Console.WriteLine("
PDF guardado como comprobante.pdf");
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
  // Se recomienda usar una librería como Gson o Jackson para parsear el JSON de respuesta.
  // import com.google.gson.Gson;

    /**
     * Genera dinámicamente el layout del CFDI en formato TXT.
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

  // Clase para mapear el JSON en el campo 'data'
  class RespuestaData {
      String xml;
      String pdf;
  }

  public CompletableFuture<RespuestaTimbrado> timbrarTxt2Async(String apiKey, String txtLayout, String keyPem, String cerPem, String plantilla, String logoB64) {
      return CompletableFuture.supplyAsync(() -> {
          try {
              String txtB64 = Base64.getEncoder().encodeToString(txtLayout.getBytes(StandardCharsets.UTF_8));
              ServicioTimbradoWSPortType port = new ServicioTimbradoWS().getServicioTimbradoWSPort();
              
              // Asumiendo que la operación se llama 'timbrarTXT2'
              Respuesta response = port.timbrarTXT2(apiKey, txtB64, keyPem, cerPem, plantilla, logoB64);

              RespuestaTimbrado resultado = new RespuestaTimbrado();
              resultado.setCode(response.getCode());
              resultado.setMessage(response.getMessage());
              resultado.setData(response.getData());
              return resultado;
          } catch (Exception e) {
              throw new RuntimeException("Error en timbrarTXT2", e);
          }
      }, executor);
  }

  // Ejemplo de uso
  public static void main(String[] args) {
      // ... (inicialización de service, apiKey, etc.)
      String txtLayout = getTxtLayout(); // Generación dinámica
      String plantilla = "1";
      String logoB64 = Base64.getEncoder().encodeToString(Files.readAllBytes(Paths.get("ruta/a/logo.png")));


      service.timbrarTxt2Async(apiKey, txtLayout, keyPem, cerPem, plantilla, logoB64).whenComplete((resultado, ex) -> {
          if (ex == null && "200".equals(resultado.getCode())) {
              System.out.println("¡Timbrado con TXT y PDF Exitoso!");
              
              // Parsear el JSON de respuesta
              // Gson gson = new Gson();
              // RespuestaData data = gson.fromJson(resultado.getData(), RespuestaData.class);
              
              // System.out.println("XML: " + data.xml);
              // byte[] pdfBytes = Base64.getDecoder().decode(data.pdf);
              // Files.write(Paths.get("comprobante.pdf"), pdfBytes);
              System.out.println("PDF guardado correctamente.");

          } else { /* ... manejo de error ... */ }
          service.shutdown();
      }).join();
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
  # ... (Definición de RespuestaTimbrado y configuración de logging)
  import json

    def get_txt_layout() -> str:
        """
        Genera dinámicamente el layout del CFDI como un string de texto.
        """
        return "
".join([
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

  async def timbrar_txt2_async(self, api_key: str, txt_layout: str, key_pem: str, cer_pem: str, plantilla: str, logo_b64: str) -> RespuestaTimbrado:
      txt_b64 = base64.b64encode(txt_layout.encode('utf-8')).decode('utf-8')
      
      # Asumiendo que la operación se llama 'timbrarTXT2'
      response = await self.async_client.service.timbrarTXT2(
          apikey=api_key, txtB64=txt_b64, keyPEM=key_pem, cerPEM=cer_pem, plantilla=plantilla, logob64=logo_b64
      )
      return RespuestaTimbrado(code=response.code, message=response.message, data=response.data)

  # Ejemplo de uso
  async def main():
      # ... (inicialización de service, apiKey, etc.)
      layout = get_txt_layout() # Generación dinámica
      plantilla = "1"
      with open("ruta/a/logo.png", "rb") as image_file:
          logo_b64 = base64.b64encode(image_file.read()).decode('utf-8')


      resultado = await service.timbrar_txt2_async(api_key, layout, key_pem, cer_pem, plantilla, logo_b64)

      if resultado.code == "200" and resultado.data:
          print("¡Timbrado con TXT y PDF Exitoso!")
          
          # Decodificar el JSON de la respuesta
          respuesta_data = json.loads(resultado.data)
          xml_timbrado = respuesta_data.get('xml')
          pdf_b64 = respuesta_data.get('pdf')

          print("--- XML Timbrado ---")
          print(xml_timbrado)

          # Guardar el PDF
          with open("comprobante.pdf", "wb") as f:
              f.write(base64.b64decode(pdf_b64))
          print("
PDF guardado como comprobante.pdf")
      else:
          print(f"Error: {resultado.code} - {resultado.message}")

  ```
  {{< /tab >}}

  {{< tab >}}
  ### Herramienta SoapClient
  PHP tiene soporte nativo para SOAP a través de la extensión SOAP. Asegúrate de que la extensión php-soap esté habilitada en tu archivo php.ini.

  ### Implementación
  ```php
  <?php
    /**
     * Genera dinámicamente el layout del CFDI como un string TXT.
     */
    function getTxtLayout(): string {
        return implode("
", [
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

  public function timbrarTxt2(string $apiKey, string $txtLayout, string $keyPem, string $cerPem, string $plantilla, string $logoB64): RespuestaTimbrado {
      $respuesta = new RespuestaTimbrado();
      try {
          $txtB64 = base64_encode($txtLayout);
          $soapClient = new SoapClient($this->wsdlUrl, ['trace' => 1, 'exceptions' => true]);
          
          $params = [
              'apikey' => $apiKey, 'txtB64' => $txtB64, 
              'keyPEM' => $keyPem, 'cerPEM' => $cerPem, 
              'plantilla' => $plantilla, 'logob64' => $logoB64
          ];

          // Asumiendo que la operación se llama 'timbrarTXT2'
          $response = $soapClient->timbrarTXT2($params);
          
          $respuesta->code = $response->return->code ?? null;
          $respuesta->message = $response->return->message ?? null;
          $respuesta->data = $response->return->data ?? null;

      } catch (Exception $e) { /* ... manejo de error ... */ }
      return $respuesta;
  }

  // Ejemplo de uso
  // ... (inicialización de service, apiKey, etc.)
  $txtLayout = getTxtLayout(); // Generación dinámica
  $plantilla = "1";
  $logoB64 = base64_encode(file_get_contents("ruta/a/logo.png"));


  $resultado = $service->timbrarTxt2($apiKey, $txtLayout, $keyPem, $cerPem, $plantilla, $logoB64);

  if ($resultado->code === '200' && !empty($resultado->data)) {
      echo "¡Timbrado con TXT y PDF Exitoso!
";
      
      // Decodificar el JSON de respuesta
      $data = json_decode($resultado->data, true);
      $xml = $data['xml'];
      $pdfB64 = $data['pdf'];

      echo "--- XML Timbrado ---
";
      echo $xml;

      // Guardar el PDF
      file_put_contents("comprobante.pdf", base64_decode($pdfB64));
      echo "
PDF guardado como comprobante.pdf
";
  } else {
      echo "Error: {$resultado->code} - {$resultado->message}
";
  }
  ?>
  ```
  {{< /tab >}}

{{< /tabs >}}


#### Respuesta (Response)

La estructura de la respuesta SOAP es similar a las anteriores, pero el contenido de la etiqueta `data` es un JSON que encapsula el XML y el PDF.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<SOAP-ENV:Envelope ...>
   <SOAP-ENV:Body>
      <ns1:timbrarTXT2Response xmlns:ns1="urn:ws_api">
         <return xsi:type="tns:RespuestaTimbrado">
            <code xsi:type="xsd:string">200</code>
            <message xsi:type="xsd:string">OK</message>
            <data xsi:type="xsd:string"><![CDATA[{
  "xml": "PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0iVVRGLTgiPz48Y2ZkaT...",
  "pdf": "JVBERi0xLjQKJdPr6eEKMSAwIG9iago8PC9UeXBlL0NhdGFsb2cvT3V0bGlu..."
}]]></data>
         </return>
      </ns1:timbrarTXT2Response>
   </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

##### Códigos de respuesta
{{< codigos-timbrado >}}
