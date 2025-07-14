---
title: Timbrar JSON + PDF
weight: 1
---

Esta operación extiende la funcionalidad de `timbrarJSON`, permitiendo no solo generar y timbrar un CFDI a partir de un layout JSON, sino también obtener una representación en formato PDF del comprobante utilizando una plantilla predefinida.

### Descripción de la Operación

Esta operación toma un layout JSON, lo convierte en un CFDI (versión 3.3 o 4.0), lo sella con los certificados proporcionados y lo timbra. Como resultado, devuelve una estructura JSON que contiene tanto el XML del CFDI timbrado como el contenido del PDF generado en formato Base64.

### Parámetros de Entrada (Input)

| Parámetro   | Tipo de Dato | Descripción                                                                                                |
| :---------- | :----------- | :--------------------------------------------------------------------------------------------------------- |
| `apikey`    | `string`     | Credencial de acceso al servicio ([Solicita aquí](../../../#Requisitos)).                                      |
| `jsonB64`   | `string`     | Cadena en formato Base64 que contiene el layout JSON con la información del CFDI a generar.  [Conversor Base 64](../../../herramientas/convertidorBase64)              |
| `keyPEM`    | `string`     | Contenido del archivo de la llave privada (`.key`) en formato PEM.                                         |
| `cerPEM`    | `string`     | Contenido del archivo del certificado de llave pública (`.cer`) en formato PEM.                              |
| `plantilla` | `string`     | Identificador numérico de la plantilla a utilizar para la generación del PDF. [Galería de plantillas](../../../herramientas/plantillas)                              |

### Parámetros de Salida (Output) - RespuestaTimbrado

| Atributo  | Tipo de Dato | Descripción                                                                                                                                |
| :-------- | :----------- | :----------------------------------------------------------------------------------------------------------------------------------------- |
| `code`    | `string`     | Código de respuesta de la operación.                                                                                                       |
| `message` | `string`     | Mensaje detallado de la respuesta.                                                                                                         |
| `data`    | `string`     | **JSON string** que contiene el XML del CFDI timbrado y el contenido del PDF en Base64. Debe ser decodificado para acceder a sus valores. |

### Estructura del JSON de Entrada

El JSON enviado en el parámetro `jsonB64` debe seguir la siguiente estructura. Los valores vacíos deben enviarse como `""`. [Conversor Base 64](../../../herramientas/convertidorBase64)

```json
{
  "Comprobante": {
    "Version": "4.0",
    "Serie": "",
    "Folio": "",
    "Fecha": "2025-07-04T12:00:00",
    "FormaPago": "01",
    "NoCertificado": "",
    "CondicionesDePago": "",
    "SubTotal": "100.00",
    "Descuento": "0.00",
    "Moneda": "MXN",
    "TipoCambio": "1",
    "Total": "116.00",
    "TipoDeComprobante": "I",
    "Exportacion": "01",
    "MetodoPago": "PUE",
    "LugarExpedicion": "64000",
    "Confirmacion": "",
    "Emisor": {
      "Rfc": "ABC010101XYZ",
      "Nombre": "Empresa Ejemplo SA de CV",
      "RegimenFiscal": "601"
    },
    "Receptor": {
      "Rfc": "XAXX010101000",
      "Nombre": "Publico en General",
      "DomicilioFiscalReceptor": "64000",
      "RegimenFiscalReceptor": "616",
      "UsoCFDI": "G03"
    },
    "Conceptos": [
      {
        "ClaveProdServ": "01010101",
        "Cantidad": "1.0",
        "ClaveUnidad": "ACT",
        "Descripcion": "Producto de prueba",
        "ValorUnitario": "100.00",
        "Importe": "100.00",
        "ObjetoImp": "02",
        "Impuestos": {
          "Traslados": [
            {
              "Base": "100.00",
              "Impuesto": "002",
              "TipoFactor": "Tasa",
              "TasaOCuota": "0.160000",
              "Importe": "16.00"
            }
          ]
        }
      }
    ],
    "Impuestos": {
      "TotalImpuestosTrasladados": "16.00",
      "Traslados": [
        {
          "Base": "100.00",
          "Impuesto": "002",
          "TipoFactor": "Tasa",
          "TasaOCuota": "0.160000",
          "Importe": "16.00"
        }
      ]
    }
  }
}
```

<!-- #### Estructura del JSON en el campo `data` TODO por hacer pruebas

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

    /// <summary>
    /// Genera dinámicamente el layout del CFDI en formato JSON.
    /// En una aplicación real, los valores se obtendrían de una base de datos o de la entrada del usuario.
    /// </summary>
    /// <returns>Un string con el JSON del CFDI.</returns>
    private static string GetJsonLayout()
    {
        // Usar un objeto anónimo para construir la estructura y luego serializarla.
        var cfdiLayout = new
        {
            Comprobante = new
            {
                Version = "4.0",
                Serie = "F",
                Folio = "123",
                Fecha = "2025-07-04T12:00:00",
                FormaPago = "01",
                SubTotal = "100.00",
                Moneda = "MXN",
                Total = "116.00",
                TipoDeComprobante = "I",
                Exportacion = "01",
                MetodoPago = "PUE",
                LugarExpedicion = "64000",
                Emisor = new
                {
                    Rfc = "ABC010101XYZ",
                    Nombre = "Empresa Ejemplo SA de CV",
                    RegimenFiscal = "601"
                },
                Receptor = new
                {
                    Rfc = "XAXX010101000",
                    Nombre = "Publico en General",
                    DomicilioFiscalReceptor = "64000",
                    RegimenFiscalReceptor = "616",
                    UsoCFDI = "G03"
                },
                Conceptos = new[]
                {
                    new
                    {
                        ClaveProdServ = "01010101",
                        Cantidad = "1.0",
                        ClaveUnidad = "ACT",
                        Descripcion = "Producto de prueba",
                        ValorUnitario = "100.00",
                        Importe = "100.00",
                        ObjetoImp = "02",
                        Impuestos = new
                        {
                            Traslados = new[]
                            {
                                new
                                {
                                    Base = "100.00",
                                    Impuesto = "002",
                                    TipoFactor = "Tasa",
                                    TasaOCuota = "0.160000",
                                    Importe = "16.00"
                                }
                            }
                        }
                    }
                },
                Impuestos = new
                {
                    TotalImpuestosTrasladados = "16.00",
                    Traslados = new[]
                    {
                        new
                        {
                            Base = "100.00",
                            Impuesto = "002",
                            TipoFactor = "Tasa",
                            TasaOCuota = "0.160000",
                            Importe = "16.00"
                        }
                    }
                }
            }
        };
        return JsonSerializer.Serialize(cfdiLayout);
    }


  // Clase para deserializar la respuesta en el campo 'data'
  public class RespuestaData
  {
      public string Xml { get; set; }
      public string Pdf { get; set; }
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

  public async Task<RespuestaTimbrado> TimbrarJson2Async(string apiKey, string jsonLayout, string keyPem, string cerPem, string plantilla)
  {
      var jsonB64 = Convert.ToBase64String(Encoding.UTF8.GetBytes(jsonLayout));
      using var client = new ServicioTimbradoWSPortTypeClient("ServicioTimbradoWSPort");
      
      // Asumiendo que la operación se llama 'timbrarJSON2'
      var response = await client.timbrarJSON2Async(apiKey, jsonB64, keyPem, cerPem, plantilla);
      return new RespuestaTimbrado { Code = response.code, Message = response.message, Data = response.data };
  }

  // Ejemplo de uso
  public async Task EjemploUsoTimbrarJson2Async()
  {
      string apiKey = "TU_API_KEY_AQUI";
      string jsonLayout = GetJsonLayout(); // Generación dinámica
      string keyPem = File.ReadAllText("ruta/a/llave.key.pem");
      string cerPem = File.ReadAllText("ruta/a/certificado.cer.pem");
      string plantilla = "1"; // ID de la plantilla a usar

      var resultado = await TimbrarJson2Async(apiKey, jsonLayout, keyPem, cerPem, plantilla);

      if (resultado?.Code == "200" && !string.IsNullOrEmpty(resultado.Data))
      {
          Console.WriteLine("¡Timbrado con JSON y PDF Exitoso!");
          
          // Deserializar el JSON de la respuesta
          var respuestaData = JsonSerializer.Deserialize<RespuestaData>(resultado.Data);
          
          Console.WriteLine("--- XML Timbrado ---");
          Console.WriteLine(respuestaData.Xml);

          // Guardar el PDF
          var pdfBytes = Convert.FromBase64String(respuestaData.Pdf);
          File.WriteAllBytes("comprobante.pdf", pdfBytes);
          Console.WriteLine("\nPDF guardado como comprobante.pdf");
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
     * Genera dinámicamente el layout del CFDI en formato JSON.
     */
    public static String getJsonLayout() {
        // Usamos un Text Block de Java 15+ para legibilidad.
        return """
        {
          "Comprobante": {
            "Version": "4.0",
            "Serie": "F",
            "Folio": "123",
            "Fecha": "2025-07-04T12:00:00",
            "FormaPago": "01",
            "SubTotal": "100.00",
            "Moneda": "MXN",
            "Total": "116.00",
            "TipoDeComprobante": "I",
            "Exportacion": "01",
            "MetodoPago": "PUE",
            "LugarExpedicion": "64000",
            "Emisor": {
              "Rfc": "ABC010101XYZ",
              "Nombre": "Empresa Ejemplo SA de CV",
              "RegimenFiscal": "601"
            },
            "Receptor": {
              "Rfc": "XAXX010101000",
              "Nombre": "Publico en General",
              "DomicilioFiscalReceptor": "64000",
              "RegimenFiscalReceptor": "616",
              "UsoCFDI": "G03"
            },
            "Conceptos": [
              {
                "ClaveProdServ": "01010101",
                "Cantidad": "1.0",
                "ClaveUnidad": "ACT",
                "Descripcion": "Producto de prueba",
                "ValorUnitario": "100.00",
                "Importe": "100.00",
                "ObjetoImp": "02",
                "Impuestos": {
                  "Traslados": [
                    {
                      "Base": "100.00",
                      "Impuesto": "002",
                      "TipoFactor": "Tasa",
                      "TasaOCuota": "0.160000",
                      "Importe": "16.00"
                    }
                  ]
                }
              }
            ],
            "Impuestos": {
              "TotalImpuestosTrasladados": "16.00",
              "Traslados": [
                {
                  "Base": "100.00",
                  "Impuesto": "002",
                  "TipoFactor": "Tasa",
                  "TasaOCuota": "0.160000",
                  "Importe": "16.00"
                }
              ]
            }
          }
        }
        """;
    }

  // Clase para mapear el JSON en el campo 'data'
  class RespuestaData {
      String xml;
      String pdf;
  }

  public CompletableFuture<RespuestaTimbrado> timbrarJson2Async(String apiKey, String jsonLayout, String keyPem, String cerPem, String plantilla) {
      return CompletableFuture.supplyAsync(() -> {
          try {
              String jsonB64 = Base64.getEncoder().encodeToString(jsonLayout.getBytes(StandardCharsets.UTF_8));
              ServicioTimbradoWSPortType port = new ServicioTimbradoWS().getServicioTimbradoWSPort();
              
              // Asumiendo que la operación se llama 'timbrarJSON2'
              Respuesta response = port.timbrarJSON2(apiKey, jsonB64, keyPem, cerPem, plantilla);

              RespuestaTimbrado resultado = new RespuestaTimbrado();
              resultado.setCode(response.getCode());
              resultado.setMessage(response.getMessage());
              resultado.setData(response.getData());
              return resultado;
          } catch (Exception e) {
              throw new RuntimeException("Error en timbrarJSON2", e);
          }
      }, executor);
  }

  // Ejemplo de uso
  public static void main(String[] args) {
      // ... (inicialización de service, apiKey, etc.)
      String jsonLayout = getJsonLayout(); // Generación dinámica
      String plantilla = "1";

      service.timbrarJson2Async(apiKey, jsonLayout, keyPem, cerPem, plantilla).whenComplete((resultado, ex) -> {
          if (ex == null && "200".equals(resultado.getCode())) {
              System.out.println("¡Timbrado con JSON y PDF Exitoso!");
              
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

def get_json_layout() -> dict:
      """
      Genera dinámicamente el layout del CFDI como un diccionario de Python.
      En una aplicación real, los valores se obtendrían de una base de datos o de la entrada del usuario.
      """
      return {
          "Comprobante": {
              "Version": "4.0",
              "Serie": "F",
              "Folio": "123",
              "Fecha": "2025-07-04T12:00:00",
              "FormaPago": "01",
              "SubTotal": "100.00",
              "Moneda": "MXN",
              "Total": "116.00",
              "TipoDeComprobante": "I",
              "Exportacion": "01",
              "MetodoPago": "PUE",
              "LugarExpedicion": "64000",
              "Emisor": {
                  "Rfc": "ABC010101XYZ",
                  "Nombre": "Empresa Ejemplo SA de CV",
                  "RegimenFiscal": "601"
              },
              "Receptor": {
                  "Rfc": "XAXX010101000",
                  "Nombre": "Publico en General",
                  "DomicilioFiscalReceptor": "64000",
                  "RegimenFiscalReceptor": "616",
                  "UsoCFDI": "G03"
              },
              "Conceptos": [
                  {
                      "ClaveProdServ": "01010101",
                      "Cantidad": "1.0",
                      "ClaveUnidad": "ACT",
                      "Descripcion": "Producto de prueba",
                      "ValorUnitario": "100.00",
                      "Importe": "100.00",
                      "ObjetoImp": "02",
                      "Impuestos": {
                          "Traslados": [
                              {
                                  "Base": "100.00",
                                  "Impuesto": "002",
                                  "TipoFactor": "Tasa",
                                  "TasaOCuota": "0.160000",
                                  "Importe": "16.00"
                              }
                          ]
                      }
                  }
              ],
              "Impuestos": {
                  "TotalImpuestosTrasladados": "16.00",
                  "Traslados": [
                      {
                          "Base": "100.00",
                          "Impuesto": "002",
                          "TipoFactor": "Tasa",
                          "TasaOCuota": "0.160000",
                          "Importe": "16.00"
                      }
                  ]
              }
          }
      }

  async def timbrar_json2_async(self, api_key: str, json_layout: dict, key_pem: str, cer_pem: str, plantilla: str) -> RespuestaTimbrado:
      json_b64 = base64.b64encode(json.dumps(json_layout).encode('utf-8')).decode('utf-8')
      
      # Asumiendo que la operación se llama 'timbrarJSON2'
      response = await self.async_client.service.timbrarJSON2(
          apikey=api_key, jsonB64=json_b64, keyPEM=key_pem, cerPEM=cer_pem, plantilla=plantilla
      )
      return RespuestaTimbrado(code=response.code, message=response.message, data=response.data)

  # Ejemplo de uso
  async def main():
      # ... (inicialización de service, apiKey, etc.)
      layout = get_json_layout() // Generación dinámica
      plantilla = "1";

      resultado = await service.timbrar_json2_async(api_key, layout, key_pem, cer_pem, plantilla)

      if resultado.code == "200" and resultado.data:
          print("¡Timbrado con JSON y PDF Exitoso!")
          
          # Decodificar el JSON de la respuesta
          respuesta_data = json.loads(resultado.data)
          xml_timbrado = respuesta_data.get('xml')
          pdf_b64 = respuesta_data.get('pdf')

          print("--- XML Timbrado ---")
          print(xml_timbrado)

          # Guardar el PDF
          with open("comprobante.pdf", "wb") as f:
              f.write(base64.b64decode(pdf_b64))
          print("\nPDF guardado como comprobante.pdf")
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
     * Genera dinámicamente el layout del CFDI como un string JSON.
     * En una aplicación real, los valores se obtendrían de una base de datos 
     * o se construiría un array asociativo y se convertiría con json_encode.
     */
    function getJsonLayout(): string {
        return <<<JSON
        {
          "Comprobante": {
            "Version": "4.0",
            "Serie": "F",
            "Folio": "123",
            "Fecha": "2025-07-04T12:00:00",
            "FormaPago": "01",
            "SubTotal": "100.00",
            "Moneda": "MXN",
            "Total": "116.00",
            "TipoDeComprobante": "I",
            "Exportacion": "01",
            "MetodoPago": "PUE",
            "LugarExpedicion": "64000",
            "Emisor": {
              "Rfc": "ABC010101XYZ",
              "Nombre": "Empresa Ejemplo SA de CV",
              "RegimenFiscal": "601"
            },
            "Receptor": {
              "Rfc": "XAXX010101000",
              "Nombre": "Publico en General",
              "DomicilioFiscalReceptor": "64000",
              "RegimenFiscalReceptor": "616",
              "UsoCFDI": "G03"
            },
            "Conceptos": [
              {
                "ClaveProdServ": "01010101",
                "Cantidad": "1.0",
                "ClaveUnidad": "ACT",
                "Descripcion": "Producto de prueba",
                "ValorUnitario": "100.00",
                "Importe": "100.00",
                "ObjetoImp": "02",
                "Impuestos": {
                  "Traslados": [
                    {
                      "Base": "100.00",
                      "Impuesto": "002",
                      "TipoFactor": "Tasa",
                      "TasaOCuota": "0.160000",
                      "Importe": "16.00"
                    }
                  ]
                }
              }
            ],
            "Impuestos": {
              "TotalImpuestosTrasladados": "16.00",
              "Traslados": [
                {
                  "Base": "100.00",
                  "Impuesto": "002",
                  "TipoFactor": "Tasa",
                  "TasaOCuota": "0.160000",
                  "Importe": "16.00"
                }
              ]
            }
          }
        }
        JSON;
    }

  public function timbrarJson2(string $apiKey, string $jsonLayout, string $keyPem, string $cerPem, string $plantilla): RespuestaTimbrado {
      $respuesta = new RespuestaTimbrado();
      try {
          $jsonB64 = base64_encode($jsonLayout);
          $soapClient = new SoapClient($this->wsdlUrl, ['trace' => 1, 'exceptions' => true]);
          
          $params = [
              'apikey' => $apiKey, 'jsonB64' => $jsonB64, 
              'keyPEM' => $keyPem, 'cerPEM' => $cerPem, 'plantilla' => $plantilla
          ];

          // Asumiendo que la operación se llama 'timbrarJSON2'
          $response = $soapClient->timbrarJSON2($params);
          
          $respuesta->code = $response->return->code ?? null;
          $respuesta->message = $response->return->message ?? null;
          $respuesta->data = $response->return->data ?? null;

      } catch (Exception $e) { /* ... manejo de error ... */ }
      return $respuesta;
  }

  // Ejemplo de uso
  // ... (inicialización de service, apiKey, etc.)
  $jsonLayout = getJsonLayout(); // Generación dinámica
  $plantilla = "1";

  $resultado = $service->timbrarJson2($apiKey, $jsonLayout, $keyPem, $cerPem, $plantilla);

  if ($resultado->code === '200' && !empty($resultado->data)) {
      echo "¡Timbrado con JSON y PDF Exitoso!\n";
      
      // Decodificar el JSON de respuesta
      $data = json_decode($resultado->data, true);
      $xml = $data['xml'];
      $pdfB64 = $data['pdf'];

      echo "--- XML Timbrado ---\n";
      echo $xml;

      // Guardar el PDF
      file_put_contents("comprobante.pdf", base64_decode($pdfB64));
      echo "\nPDF guardado como comprobante.pdf\n";
  } else {
      echo "Error: {$resultado->code} - {$resultado->message}\n";
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
      <ns1:timbrarJSON2Response xmlns:ns1="urn:ws_api">
         <return xsi:type="tns:RespuestaTimbrado">
            <code xsi:type="xsd:string">200</code>
            <message xsi:type="xsd:string">OK</message>
            <data xsi:type="xsd:string"></data>
         </return>
      </ns1:timbrarJSON2Response>
   </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

##### Códigos de respuesta
{{< codigos-timbrado >}}
