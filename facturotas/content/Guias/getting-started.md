---
title: "Guía de Inicio: Tu Primera Factura"
weight: 1
---

Esta guía te mostrará cómo crear y timbrar tu primera factura electrónica (CFDI) de ingresos utilizando la API de Facturotas. En solo unos pocos pasos, pasarás de tener los datos de una factura a obtener un XML timbrado por el SAT y una representación en PDF lista para enviar a tu cliente.

Para este ejemplo, asumiremos un caso de uso común en una fintech: emitir una factura por una comisión de servicio.

## El Proceso Completo

El flujo para emitir una factura es sencillo y se resume en los siguientes pasos:

1.  **Preparar tus datos**: Reúne tu API Key, tus certificados CSD y la información de la factura.
2.  **Construir el JSON**: Estructura los datos de la factura en el formato JSON requerido.
3.  **Llamar a la API**: Envía la petición al endpoint `timbrarJSON2` para generar el XML y el PDF.
4.  **Verificar el resultado**: Consulta el estado de tu factura para confirmar que fue timbrada correctamente.

---

## Paso 1: Requisitos Previos

Antes de hacer tu primera llamada a la API, asegúrate de tener lo siguiente:

### 1. API Key

Toda petición a la API debe ser autenticada con una `apikey`. Esta credencial única identifica tu cuenta.

*   **[Ingresa a este enlace para solicitar tu Api Key](https://forms.gle/2eMfosbQEevqTvmE7)**

### 2. Certificados de Sello Digital (CSD)

Para firmar digitalmente tus facturas (un requisito del SAT), necesitas tu Certificado de Sello Digital. Este consiste en dos archivos:

*   `tu_llave.key`: Tu llave privada.
*   `tu_certificado.cer`: Tu llave pública.

La API espera que estos archivos estén en formato **PEM**. Si los tienes en su formato original (DER), puedes convertirlos fácilmente usando OpenSSL:

```bash
# Convertir la llave privada (.key) a PEM
openssl pkcs8 -inform DER -in tu_llave.key -out tu_llave.key.pem -passin pass:tu_contraseña

# Convertir el certificado (.cer) a PEM
openssl x509 -inform DER -in tu_certificado.cer -out tu_certificado.cer.pem
```

Una vez convertidos, deberás leer el contenido de estos archivos `.pem` como texto plano para enviarlo en la petición a la API.

---

## Paso 2: Construir el JSON de la Factura

El corazón de tu factura es un objeto JSON que contiene toda la información requerida por el SAT. La estructura sigue el estándar CFDI 4.0.

A continuación, se muestra un ejemplo de un JSON para una factura de ingresos simple. Puedes usar el [layout completo como referencia](../../../assets/cfdi40/layout_cfdi40.json) para ver todos los campos disponibles.

**Ejemplo de JSON para una factura de comisión:**

```json
{
  "Comprobante": {
    "Version": "4.0",
    "Serie": "F",
    "Folio": "1",
    "Fecha": "2025-07-11T10:00:00",
    "FormaPago": "03", // Transferencia electrónica de fondos
    "SubTotal": "150.00",
    "Moneda": "MXN",
    "Total": "174.00",
    "TipoDeComprobante": "I", // Ingreso
    "Exportacion": "01", // No aplica
    "MetodoPago": "PUE", // Pago en una sola exhibición
    "LugarExpedicion": "64000", // Código Postal del emisor
    "Emisor": {
      "Rfc": "FIN200101ABC",
      "Nombre": "Fintech Ejemplo SA de CV",
      "RegimenFiscal": "601" // General de Ley Personas Morales
    },
    "Receptor": {
      "Rfc": "GOME900101XYZ",
      "Nombre": "Mi Cliente Ejemplo",
      "DomicilioFiscalReceptor": "87000",
      "RegimenFiscalReceptor": "612", // Personas Físicas con Actividades Empresariales y Profesionales
      "UsoCFDI": "G03" // Gastos en general
    },
    "Conceptos": [
      {
        "ClaveProdServ": "84111506", // Servicios de facturación
        "Cantidad": "1.0",
        "ClaveUnidad": "E48", // Unidad de servicio
        "Descripcion": "Comisión por servicio de plataforma",
        "ValorUnitario": "150.00",
        "Importe": "150.00",
        "ObjetoImp": "02", // Sí es objeto de impuesto
        "Impuestos": {
          "Traslados": [
            {
              "Base": "150.00",
              "Impuesto": "002", // IVA
              "TipoFactor": "Tasa",
              "TasaOCuota": "0.160000",
              "Importe": "24.00"
            }
          ]
        }
      }
    ],
    "Impuestos": {
      "TotalImpuestosTrasladados": "24.00",
      "Traslados": [
        {
          "Base": "150.00",
          "Impuesto": "002",
          "TipoFactor": "Tasa",
          "TasaOCuota": "0.160000",
          "Importe": "24.00"
        }
      ]
    }
  }
}
```

---

## Paso 3: Timbrar la Factura y Generar el PDF

Con tu JSON listo, es hora de enviarlo a la API. Usaremos el endpoint `timbrarJSON2`, que es la forma más directa de obtener tanto el XML timbrado como un PDF.

Este endpoint requiere que el JSON sea enviado en formato **Base64**.

### Seleccionar una Plantilla de PDF

El parámetro `plantilla` te permite elegir el diseño de tu PDF. Para este ejemplo, usaremos la **Plantilla 1**, que es un diseño clásico y profesional. Puedes ver la [galería completa de plantillas aquí](../../herramientas/plantillas).

### Ejemplos de Llamada a la API

A continuación, te mostramos cómo realizar la llamada al endpoint `timbrarJSON2` en diferentes lenguajes de programación.

{{< tabs items="C#,Java,Python,PHP" >}}

{{< tab >}}

### Herramienta svcutil
Primero, genera el cliente del servicio WCF. Para un ambiente de **desarrollo**, ejecuta:
```
svcutil.exe https://dev.facturaloplus.com/ws/servicio.do?wsdl /out:ServicioTimbradoClient.cs /config:app.config
```

### Implementación
```csharp
// Asegúrate de tener la librería System.Text.Json
using System;
using System.IO;
using System.Text;
using System.Text.Json;
using System.Threading.Tasks;

// Clase para deserializar el JSON anidado en la respuesta
public class RespuestaData
{
    public string Xml { get; set; }
    public string Pdf { get; set; }
}

public class TimbradoEjemplo
{
    // Define el JSON de la factura como un string.
    private static string GetJsonLayout()
    {
        // Para un ejemplo real, este JSON se construiría dinámicamente.
        return "{\"Comprobante\":{\"Version\":\"4.0\",\"Serie\":\"F\",\"Folio\":\"1\",\"Fecha\":\"2025-07-11T10:00:00\",\"FormaPago\":\"03\",\"SubTotal\":\"150.00\",\"Moneda\":\"MXN\",\"Total\":\"174.00\",\"TipoDeComprobante\":\"I\",\"Exportacion\":\"01\",\"MetodoPago\":\"PUE\",\"LugarExpedicion\":\"64000\",\"Emisor\":{\"Rfc\":\"FIN200101ABC\",\"Nombre\":\"Fintech Ejemplo SA de CV\",\"RegimenFiscal\":\"601\"},\"Receptor\":{\"Rfc\":\"GOME900101XYZ\",\"Nombre\":\"Mi Cliente Ejemplo\",\"DomicilioFiscalReceptor\":\"87000\",\"RegimenFiscalReceptor\":\"612\",\"UsoCFDI\":\"G03\"},\"Conceptos\":[{\"ClaveProdServ\":\"84111506\",\"Cantidad\":\"1.0\",\"ClaveUnidad\":\"E48\",\"Descripcion\":\"Comisión por servicio de plataforma\",\"ValorUnitario\":\"150.00\",\"Importe\":\"150.00\",\"ObjetoImp\":\"02\",\"Impuestos\":{\"Traslados\":[{\"Base\":\"150.00\",\"Impuesto\":\"002\",\"TipoFactor\":\"Tasa\",\"TasaOCuota\":\"0.160000\",\"Importe\":\"24.00\"}]}},{\"Base\":\"150.00\",\"Impuesto\":\"002\",\"TipoFactor\":\"Tasa\",\"TasaOCuota\":\"0.160000\",\"Importe\":\"24.00\"}]}}} ";
    }

    public static async Task RealizarTimbradoAsync()
    {
        // 1. Configuración
        string apiKey = "TU_API_KEY_AQUI";
        string keyPem = File.ReadAllText("ruta/a/tu_llave.key.pem");
        string cerPem = File.ReadAllText("ruta/a/tu_certificado.cer.pem");
        string jsonLayout = GetJsonLayout();
        string plantilla = "1";

        // 2. Codificar JSON a Base64
        var jsonB64 = Convert.ToBase64String(Encoding.UTF8.GetBytes(jsonLayout));

        // 3. Realizar la llamada a la API
        using var client = new ServicioTimbradoWSPortTypeClient("ServicioTimbradoWSPort");
        var response = await client.timbrarJSON2Async(apiKey, jsonB64, keyPem, cerPem, plantilla);

        // 4. Procesar la respuesta
        if (response?.code == "200" && !string.IsNullOrEmpty(response.data))
        {
            Console.WriteLine("✅ ¡Factura timbrada con éxito!");
            var respuestaData = JsonSerializer.Deserialize<RespuestaData>(response.data);

            File.WriteAllText("factura_timbrada.xml", respuestaData.Xml);
            Console.WriteLine("📄 XML guardado como 'factura_timbrada.xml'");

            var pdfBytes = Convert.FromBase64String(respuestaData.Pdf);
            File.WriteAllBytes("factura.pdf", pdfBytes);
            Console.WriteLine("📄 PDF guardado como 'factura.pdf'");
        }
        else
        {
            Console.WriteLine($"❌ Error al timbrar: {response?.code} - {response?.message}");
        }
    }
}
```
{{< /tab >}}

{{< tab >}}
### Herramienta wsimport
Genera las clases cliente desde el WSDL. Para un ambiente de **desarrollo**, usa:
```
wsimport -keep -p com.facturotas.cliente https://dev.facturaloplus.com/ws/servicio.do?wsdl
```

### Implementación
```java
// Se recomienda usar una librería como Gson o Jackson para el JSON.
// import com.google.gson.Gson;
import java.nio.charset.StandardCharsets;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.Base64;

// Clase para mapear el JSON de respuesta
class RespuestaData {
    String xml;
    String pdf;
}

public class TimbradoEjemplo {

    public static String getJsonLayout() {
        // En una aplicación real, este JSON se construiría con un objeto y se serializaría.
        return "{\"Comprobante\":{\"Version\":\"4.0\",\"Serie\":\"F\",\"Folio\":\"1\",\"Fecha\":\"2025-07-11T10:00:00\",\"FormaPago\":\"03\",\"SubTotal\":\"150.00\",\"Moneda\":\"MXN\",\"Total\":\"174.00\",\"TipoDeComprobante\":\"I\",\"Exportacion\":\"01\",\"MetodoPago\":\"PUE\",\"LugarExpedicion\":\"64000\",\"Emisor\":{\"Rfc\":\"FIN200101ABC\",\"Nombre\":\"Fintech Ejemplo SA de CV\",\"RegimenFiscal\":\"601\"},\"Receptor\":{\"Rfc\":\"GOME900101XYZ\",\"Nombre\":\"Mi Cliente Ejemplo\",\"DomicilioFiscalReceptor\":\"87000\",\"RegimenFiscalReceptor\":\"612\",\"UsoCFDI\":\"G03\"},\"Conceptos\":[{\"ClaveProdServ\":\"84111506\",\"Cantidad\":\"1.0\",\"ClaveUnidad\":\"E48\",\"Descripcion\":\"Comisión por servicio de plataforma\",\"ValorUnitario\":\"150.00\",\"Importe\":\"150.00\",\"ObjetoImp\":\"02\",\"Impuestos\":{\"Traslados\":[{\"Base\":\"150.00\",\"Impuesto\":\"002\",\"TipoFactor\":\"Tasa\",\"TasaOCuota\":\"0.160000\",\"Importe\":\"24.00\"}]}},{\"Base\":\"150.00\",\"Impuesto\":\"002\",\"TipoFactor\":\"Tasa\",\"TasaOCuota\":\"0.160000\",\"Importe\":\"24.00\"}]}}} ";
    }

    public static void main(String[] args) throws Exception {
        // 1. Configuración
        String apiKey = "TU_API_KEY_AQUI";
        String keyPem = new String(Files.readAllBytes(Paths.get("ruta/a/tu_llave.key.pem")));
        String cerPem = new String(Files.readAllBytes(Paths.get("ruta/a/tu_certificado.cer.pem")));
        String jsonLayout = getJsonLayout();
        String plantilla = "1";

        // 2. Codificar JSON a Base64
        String jsonB64 = Base64.getEncoder().encodeToString(jsonLayout.getBytes(StandardCharsets.UTF_8));

        // 3. Realizar la llamada a la API
        ServicioTimbradoWSPortType port = new ServicioTimbradoWS().getServicioTimbradoWSPort();
        Respuesta response = port.timbrarJSON2(apiKey, jsonB64, keyPem, cerPem, plantilla);

        // 4. Procesar la respuesta
        if (response.getCode().equals("200") && response.getData() != null) {
            System.out.println("✅ ¡Factura timbrada con éxito!");
            // Gson gson = new Gson();
            // RespuestaData data = gson.fromJson(response.getData(), RespuestaData.class);
            
            // Files.write(Paths.get("factura_timbrada.xml"), data.xml.getBytes());
            // System.out.println("📄 XML guardado como 'factura_timbrada.xml'");
            
            // byte[] pdfBytes = Base64.getDecoder().decode(data.pdf);
            // Files.write(Paths.get("factura.pdf"), pdfBytes);
            // System.out.println("📄 PDF guardado como 'factura.pdf'");
        } else {
            System.out.println("❌ Error al timbrar: " + response.getCode() + " - " + response.getMessage());
        }
    }
}
```
{{< /tab >}}

{{< tab >}}
### Herramienta Zeep
Usa `pip` para instalar la librería `zeep`:
```
pip install zeep
```

### Implementación
```python
import zeep
import base64
import json

# 1. Define el JSON de la factura
def get_json_layout():
    return {
      "Comprobante": {
        "Version": "4.0", "Serie": "F", "Folio": "1", "Fecha": "2025-07-11T10:00:00",
        "FormaPago": "03", "SubTotal": "150.00", "Moneda": "MXN", "Total": "174.00",
        "TipoDeComprobante": "I", "Exportacion": "01", "MetodoPago": "PUE", "LugarExpedicion": "64000",
        "Emisor": {
          "Rfc": "FIN200101ABC", "Nombre": "Fintech Ejemplo SA de CV", "RegimenFiscal": "601"
        },
        "Receptor": {
          "Rfc": "GOME900101XYZ", "Nombre": "Mi Cliente Ejemplo", "DomicilioFiscalReceptor": "87000",
          "RegimenFiscalReceptor": "612", "UsoCFDI": "G03"
        },
        "Conceptos": [{
            "ClaveProdServ": "84111506", "Cantidad": "1.0", "ClaveUnidad": "E48",
            "Descripcion": "Comisión por servicio de plataforma", "ValorUnitario": "150.00", "Importe": "150.00",
            "ObjetoImp": "02",
            "Impuestos": { "Traslados": [{"Base": "150.00", "Impuesto": "002", "TipoFactor": "Tasa", "TasaOCuota": "0.160000", "Importe": "24.00"}] }
        }],
        "Impuestos": {
            "TotalImpuestosTrasladados": "24.00",
            "Traslados": [{"Base": "150.00", "Impuesto": "002", "TipoFactor": "Tasa", "TasaOCuota": "0.160000", "Importe": "24.00"}]
        }
      }
    }

def timbrar_factura():
    # 2. Configuración
    WSDL_URL = "https://dev.facturaloplus.com/ws/servicio.do?wsdl"
    API_KEY = "TU_API_KEY_AQUI"
    
    with open("ruta/a/tu_llave.key.pem", 'r') as f: key_pem = f.read()
    with open("ruta/a/tu_certificado.cer.pem", 'r') as f: cer_pem = f.read()

    # 3. Codificar JSON a Base64
    json_layout = get_json_layout()
    json_string = json.dumps(json_layout)
    json_b64 = base64.b64encode(json_string.encode('utf-8')).decode('utf-8')

    # 4. Realizar la llamada a la API
    try:
        client = zeep.Client(wsdl=WSDL_URL)
        response = client.service.timbrarJSON2(
            apikey=API_KEY, jsonB64=json_b64, keyPEM=key_pem, cerPEM=cer_pem, plantilla="1"
        )

        # 5. Procesar la respuesta
        if response.code == "200" and response.data:
            print("✅ ¡Factura timbrada con éxito!")
            data = json.loads(response.data)
            
            with open("factura_timbrada.xml", "w") as f: f.write(data.get('xml'))
            print("📄 XML guardado como 'factura_timbrada.xml'")

            with open("factura.pdf", "wb") as f: f.write(base64.b64decode(data.get('pdf')))
            print("📄 PDF guardado como 'factura.pdf'")
        else:
            print(f"❌ Error al timbrar: {response.code} - {response.message}")

    except Exception as e:
        print(f"Ocurrió un error: {e}")

if __name__ == "__main__":
    timbrar_factura()
```
{{< /tab >}}

{{< tab >}}
### Herramienta SoapClient
Asegúrate de que la extensión `php-soap` esté habilitada en tu `php.ini`.

### Implementación
```php
<?php
function getJsonLayout(): string {
    // En una aplicación real, esto sería un array de PHP convertido con json_encode.
    return '{"Comprobante":{"Version":"4.0","Serie":"F","Folio":"1","Fecha":"2025-07-11T10:00:00","FormaPago":"03","SubTotal":"150.00","Moneda":"MXN","Total":"174.00","TipoDeComprobante":"I","Exportacion":"01","MetodoPago":"PUE","LugarExpedicion":"64000","Emisor":{"Rfc":"FIN200101ABC","Nombre":"Fintech Ejemplo SA de CV","RegimenFiscal":"601"},"Receptor":{"Rfc":"GOME900101XYZ","Nombre":"Mi Cliente Ejemplo","DomicilioFiscalReceptor":"87000","RegimenFiscalReceptor":"612","UsoCFDI":"G03"},"Conceptos":[{"ClaveProdServ":"84111506","Cantidad":"1.0","ClaveUnidad":"E48","Descripcion":"Comisión por servicio de plataforma","ValorUnitario":"150.00","Importe":"150.00","ObjetoImp":"02","Impuestos":{"Traslados":[{"Base":"150.00","Impuesto":"002","TipoFactor":"Tasa","TasaOCuota":"0.160000","Importe":"24.00"}]}}],"Impuestos":{"TotalImpuestosTrasladados":"24.00","Traslados":[{"Base":"150.00","Impuesto":"002","TipoFactor":"Tasa","TasaOCuota":"0.160000","Importe":"24.00"}]}}}';
}

function timbrarFactura() {
    // 1. Configuración
    $wsdlUrl = 'https://dev.facturaloplus.com/ws/servicio.do?wsdl';
    $apiKey = 'TU_API_KEY_AQUI';
    $keyPem = file_get_contents('ruta/a/tu_llave.key.pem');
    $cerPem = file_get_contents('ruta/a/tu_certificado.cer.pem');
    $jsonLayout = getJsonLayout();
    $plantilla = '1';

    // 2. Codificar JSON a Base64
    $jsonB64 = base64_encode($jsonLayout);

    // 3. Realizar la llamada a la API
    try {
        $client = new SoapClient($wsdlUrl, ['trace' => 1, 'exceptions' => true]);
        $params = [
            'apikey'  => $apiKey,
            'jsonB64' => $jsonB64,
            'keyPEM'  => $keyPem,
            'cerPEM'  => $cerPem,
            'plantilla' => $plantilla
        ];
        $response = $client->timbrarJSON2($params)->return;

        // 4. Procesar la respuesta
        if ($response->code == '200' && !empty($response->data)) {
            echo "✅ ¡Factura timbrada con éxito!\n";
            $data = json_decode($response->data, true);
            
            file_put_contents('factura_timbrada.xml', $data['xml']);
            echo "📄 XML guardado como 'factura_timbrada.xml'\n";

            file_put_contents('factura.pdf', base64_decode($data['pdf']));
            echo "📄 PDF guardado como 'factura.pdf'\n";
        } else {
            echo "❌ Error al timbrar: {$response->code} - {$response->message}\n";
        }
    } catch (Exception $e) {
        echo "Ocurrió un error: " . $e->getMessage() . "\n";
    }
}

timbrarFactura();
?>
```
{{< /tab >}}

{{< /tabs >}}



Una respuesta exitosa (`code: "200"`) te devolverá un JSON en el campo `data` que contiene el XML oficial y el PDF en Base64.

---

## Paso 4: Verificar la Factura (Opcional pero recomendado)

Para estar 100% seguro de que tu factura fue registrada correctamente, puedes consultarla usando su Folio Fiscal Único (UUID). El UUID se encuentra dentro del XML que recibiste en el paso anterior.

**1. Extraer el UUID:** Busca en el archivo `factura_timbrada.xml` el nodo `tfd:TimbreFiscalDigital` y copia el valor del atributo `UUID`.

**2. Consultar el endpoint `consultarCFDI`:**

{{< tabs items="C#,Java,Python,PHP" >}}

{{< tab >}}

```csharp
// (Continuación del script anterior)
public static async Task VerificarFacturaAsync(string apiKey, string uuid)
{
    using var client = new ServicioTimbradoWSPortTypeClient("ServicioTimbradoWSPort");
    var response = await client.consultarCFDIAsync(apiKey, uuid);

    if (response?.code == "200")
    {
        Console.WriteLine($"✅ Verificación exitosa: El CFDI con UUID {uuid} está activo.");
        Console.WriteLine($"Estado SAT: {response.data}");
    }
    else
    {
        Console.WriteLine($"❌ Error en la consulta: {response?.code} - {response?.message}");
    }
}

// Para llamarlo:
// await VerificarFacturaAsync("TU_API_KEY_AQUI", "EL_UUID_EXTRAIDO_DEL_XML");
```

{{< /tab >}}

{{< tab >}}

```java
// (Continuación del script anterior)
public static void verificarFactura(String apiKey, String uuid) throws Exception {
    ServicioTimbradoWSPortType port = new ServicioTimbradoWS().getServicioTimbradoWSPort();
    Respuesta response = port.consultarCFDI(apiKey, uuid);

    if (response.getCode().equals("200")) {
        System.out.println("✅ Verificación exitosa: El CFDI con UUID " + uuid + " está activo.");
        System.out.println("Estado SAT: " + response.getData());
    } else {
        System.out.println("❌ Error en la consulta: " + response.getCode() + " - " + response.getMessage());
    }
}

// Para llamarlo:
// verificarFactura("TU_API_KEY_AQUI", "EL_UUID_EXTRAIDO_DEL_XML");
```

{{< /tab >}}

{{< tab >}}

```python
# (Continuación del script anterior)
def verificar_factura(api_key, uuid):
    WSDL_URL = "https://dev.facturaloplus.com/ws/servicio.do?wsdl"
    try:
        client = zeep.Client(wsdl=WSDL_URL)
        response = client.service.consultarCFDI(apikey=api_key, uuid=uuid)

        if response.code == "200":
            print(f"✅ Verificación exitosa: El CFDI con UUID {uuid} está activo.")
            print(f"Estado SAT: {response.data}")
        else:
            print(f"❌ Error en la consulta: {response.code} - {response.message}")

    except Exception as e:
        print(f"Ocurrió un error en la consulta: {e}")

# Para llamarlo:
# verificar_factura("TU_API_KEY_AQUI", "EL_UUID_EXTRAIDO_DEL_XML")
```

{{< /tab >}}

{{< tab >}}

```php
<?php
// (Continuación del script anterior)
function verificarFactura($apiKey, $uuid) {
    $wsdlUrl = 'https://dev.facturaloplus.com/ws/servicio.do?wsdl';
    try {
        $client = new SoapClient($wsdlUrl, ['trace' => 1, 'exceptions' => true]);
        $params = ['apikey' => $apiKey, 'uuid' => $uuid];
        $response = $client->consultarCFDI($params)->return;

        if ($response->code == '200') {
            echo "✅ Verificación exitosa: El CFDI con UUID {$uuid} está activo.\n";
            echo "Estado SAT: {$response->data}\n";
        } else {
            echo "❌ Error en la consulta: {$response->code} - {$response->message}\n";
        }
    } catch (Exception $e) {
        echo "Ocurrió un error en la consulta: " . $e->getMessage() . "\n";
    }
}

// Para llamarlo:
// verificarFactura('TU_API_KEY_AQUI', 'EL_UUID_EXTRAIDO_DEL_XML');
?>
```

{{< /tab >}}

{{< /tabs >}}


Si la respuesta es exitosa, ¡felicidades! Has completado el ciclo de emisión de una factura electrónica.

## Próximos Pasos

Ahora que has dominado el timbrado de una factura de ingresos, puedes explorar otras funcionalidades:

*   [**Cancelar un CFDI**](../../api/cancelar/): Aprende cómo cancelar una factura que ya emitiste.
*   **Crear otros tipos de comprobantes**: Explora cómo generar facturas de pago, notas de crédito o retenciones.
*   **Personalizar tus PDFs**: Revisa la [galería de plantillas](../../herramientas/plantillas) para encontrar el diseño que mejor se adapte a tu marca.
