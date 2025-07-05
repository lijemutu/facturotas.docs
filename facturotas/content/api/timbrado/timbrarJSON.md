# timbrarJSON

Esta sección detalla la operación del servicio de timbrado a partir de un layout JSON, permitiendo generar, sellar y obtener el XML timbrado de un Comprobante Fiscal Digital (CFDI) sin necesidad de construir el XML manualmente.

### Descripción de la Operación

Esta operación facilita la creación de un CFDI (versiones 3.3 o 4.0) a partir de un archivo JSON. El servicio se encarga de construir la estructura del XML, sellarlo con los certificados proporcionados y devolver el XML timbrado completo, junto con su folio fiscal (UUID) y el sello digital del SAT.

### Parámetros de Entrada (Input)

| Parámetro | Tipo de Dato | Descripción                                                                                                |
| :-------- | :----------- | :--------------------------------------------------------------------------------------------------------- |
| `apikey`  | `string`     | Credencial de acceso al servicio ([Solicita aquí](../../../#Requisitos)).                                      |
| `jsonB64` | `string`     | Cadena en formato Base64 que contiene el layout JSON con la información del CFDI a generar (ver estructura abajo). |
| `keyPEM`  | `string`     | Contenido del archivo de la llave privada (`.key`) en formato PEM.                                         |
| `cerPEM`  | `string`     | Contenido del archivo del certificado de llave pública (`.cer`) en formato PEM.                              |

### Parámetros de Salida (Output) - RespuestaTimbrado

La respuesta de esta operación es idéntica a la de la operación `timbrar`.

| Atributo  | Tipo de Dato | Descripción                             |
| :-------- | :----------- | :-------------------------------------- |
| `code`    | `string`     | Código de respuesta de la operación.    |
| `message` | `string`     | Mensaje detallado de la respuesta.      |
| `data`    | `string`     | XML del CFDI timbrado en caso de éxito. |

### Estructura del JSON de Entrada

El JSON enviado en el parámetro `jsonB64` debe seguir la siguiente estructura. Los valores vacíos deben enviarse como `""`.

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
   using System.Text.Json;
   using System.Threading.Tasks;
   
   // ... (Asegúrate de tener la referencia al cliente SOAP generado por svcutil)

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

   /// <summary>
   /// Prepara y codifica los datos para la operación timbrarJSON.
   /// </summary>
   public async Task<RespuestaTimbrado> TimbrarJsonAsync(string apiKey, string jsonLayout, string keyPemPath, string cerPemPath)
   {
       // 1. Codificar el JSON a Base64
       var jsonBytes = Encoding.UTF8.GetBytes(jsonLayout);
       var jsonB64 = Convert.ToBase64String(jsonBytes);

       // 2. Leer el contenido de los archivos PEM
       var keyPem = await File.ReadAllTextAsync(keyPemPath);
       var cerPem = await File.ReadAllTextAsync(cerPemPath);

       // 3. Invocar al servicio web
       using var client = new ServicioTimbradoWSPortTypeClient("ServicioTimbradoWSPort");
       try
       {
           // Asumiendo que la operación en el WSDL se llama 'timbrarJSON'
           var response = await client.timbrarJSONAsync(apiKey, jsonB64, keyPem, cerPem);
           
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
           Console.WriteLine($"Error al timbrar con JSON: {ex.Message}");
           throw;
       }
   }

   // Ejemplo de uso
   public async Task EjemploUsoTimbrarJsonAsync()
   {
       string apiKey = "TU_API_KEY_AQUI";
       string jsonLayout = GetJsonLayout(); // Carga el layout dinámicamente
       string keyPemFile = "ruta/a/tu/llave.key.pem";
       string cerPemFile = "ruta/a/tu/certificado.cer.pem";

       var resultado = await TimbrarJsonAsync(apiKey, jsonLayout, keyPemFile, cerPemFile);

       if (resultado?.Code == "200")
       {
           Console.WriteLine("¡Timbrado con JSON Exitoso!");
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

  public class TimbradoJsonService {

      // ... (Definición de ExecutorService y Logger)

    /**
     * Genera dinámicamente el layout del CFDI en formato JSON.
     * En una aplicación real, los valores se obtendrían de una base de datos o de la entrada del usuario.
     * Se recomienda usar una librería como Gson o Jackson para construir el JSON de forma más robusta.
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

      public CompletableFuture<RespuestaTimbrado> timbrarJsonAsync(String apiKey, String jsonLayout, String keyPemPath, String cerPemPath) {
          return CompletableFuture.supplyAsync(() -> {
              try {
                  // 1. Codificar JSON a Base64
                  String jsonB64 = Base64.getEncoder().encodeToString(jsonLayout.getBytes(StandardCharsets.UTF_8));

                  // 2. Leer archivos PEM
                  String keyPem = new String(Files.readAllBytes(Paths.get(keyPemPath)));
                  String cerPem = new String(Files.readAllBytes(Paths.get(cerPemPath)));

                  // 3. Llamar al servicio
                  ServicioTimbradoWS service = new ServicioTimbradoWS();
                  ServicioTimbradoWSPortType port = service.getServicioTimbradoWSPort();
                  
                  // Asumiendo que la operación se llama 'timbrarJSON'
                  Respuesta response = port.timbrarJSON(apiKey, jsonB64, keyPem, cerPem);

                  RespuestaTimbrado resultado = new RespuestaTimbrado();
                  resultado.setCode(response.getCode());
                  resultado.setMessage(response.getMessage());
                  resultado.setData(response.getData());
                  return resultado;

              } catch (Exception ex) {
                  // Manejo de errores
                  throw new RuntimeException("Error al timbrar con JSON", ex);
              }
          }, executor);
      }
  
  // Ejemplo de uso
  public static void main(String[] args) {
      TimbradoJsonService service = new TimbradoJsonService();
      String apiKey = "TU_API_KEY_AQUI";
      String keyPem = "ruta/a/tu/llave.key.pem";
      String cerPem = "ruta/a/tu/certificado.cer.pem";
      String jsonLayout = getJsonLayout(); // Carga el layout dinámicamente

      service.timbrarJsonAsync(apiKey, jsonLayout, keyPem, cerPem)
          .whenComplete((resultado, ex) -> {
              if (ex != null) {
                  System.err.println("Error: " + ex.getMessage());
              } else if ("200".equals(resultado.getCode())) {
                  System.out.println("¡Timbrado con JSON Exitoso!");
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
  import json
  from zeep.asyncio import AsyncClient

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

  class TimbradoJsonService:
      def __init__(self, wsdl_url: str):
          self.wsdl_url = wsdl_url
          self.async_client = AsyncClient(self.wsdl_url)

      async def timbrar_json_async(self, api_key: str, json_layout: dict, key_pem_path: str, cer_pem_path: str) -> RespuestaTimbrado:
          try:
              # 1. Convertir dict a JSON string y luego a Base64
              json_str = json.dumps(json_layout)
              json_b64 = base64.b64encode(json_str.encode('utf-8')).decode('utf-8')

              # 2. Leer archivos PEM
              with open(key_pem_path, 'r') as f:
                  key_pem = f.read()
              with open(cer_pem_path, 'r') as f:
                  cer_pem = f.read()

              # 3. Llamar al servicio
              # Asumiendo que la operación se llama 'timbrarJSON'
              response = await self.async_client.service.timbrarJSON(
                  apikey=api_key,
                  jsonB64=json_b64,
                  keyPEM=key_pem,
                  cerPEM=cer_pem
              )
              return RespuestaTimbrado(
                  code=response.code,
                  message=response.message,
                  data=response.data
              )
          except Exception as e:
              logging.error(f"Error al timbrar con JSON: {e}")
              raise

  # Ejemplo de uso
  async def main():
      service = TimbradoJsonService("https://dev.facturaloplus.com/ws/servicio.do?wsdl")
      api_key = "TU_API_KEY_AQUI";
      key_pem = "ruta/a/tu/llave.key.pem";
      cer_pem = "ruta/a/tu/certificado.cer.pem";
      
      layout = get_json_layout(); // Carga el layout dinámicamente

      resultado = await service.timbrar_json_async(api_key, layout, key_pem, cer_pem);

      if resultado.code == "200":
          print("¡Timbrado con JSON Exitoso!");
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

  class TimbradoJsonService {
      private string $wsdlUrl;

      public function __construct(string $wsdlUrl) {
          $this->wsdlUrl = $wsdlUrl;
      }

      public function timbrarJson(string $apiKey, string $jsonLayout, string $keyPemPath, string $cerPemPath): RespuestaTimbrado {
          $respuesta = new RespuestaTimbrado();
          try {
              // 1. Codificar JSON a Base64
              $jsonB64 = base64_encode($jsonLayout);

              // 2. Leer archivos PEM
              $keyPem = file_get_contents($keyPemPath);
              $cerPem = file_get_contents($cerPemPath);

              if ($keyPem === false || $cerPem === false) {
                  throw new Exception("No se pudieron leer los archivos .pem");
              }

              // 3. Llamar al servicio
              $soapClient = new SoapClient($this->wsdlUrl, ['trace' => 1, 'exceptions' => true]);
              
              $params = [
                  'apikey'  => $apiKey,
                  'jsonB64' => $jsonB64,
                  'keyPEM'  => $keyPem,
                  'cerPEM'  => $cerPem
              ];

              // Asumiendo que la operación se llama 'timbrarJSON'
              $response = $soapClient->timbrarJSON($params);
              
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
  $service = new TimbradoJsonService("https://dev.facturaloplus.com/ws/servicio.do?wsdl");
  $apiKey = "TU_API_KEY_AQUI";
  $keyPem = "ruta/a/tu/llave.key.pem";
  $cerPem = "ruta/a/tu/certificado.cer.pem";
  $jsonLayout = getJsonLayout(); // Carga el layout dinámicamente

  $resultado = $service->timbrarJson($apiKey, $jsonLayout, $keyPem, $cerPem);

  header('Content-Type: text/plain');
  if ($resultado->code === '200') {
      echo "¡Timbrado con JSON Exitoso!\n";
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
      <ns1:timbrarJSONResponse xmlns:ns1="urn:ws_api">
         <return xsi:type="tns:RespuestaTimbrado">
            <code xsi:type="xsd:string">CÓDIGO</code>
            <message xsi:type="xsd:string">MENSAJE</message>
            <data xsi:type="xsd:string"><![CDATA[CFDI TIMBRADO]]></data>
         </return>
      </ns1:timbrarJSONResponse>
   </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

##### Códigos de respuesta
{{< codigos-timbrado >}}
