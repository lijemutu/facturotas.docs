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

   public async Task<RespuestaTimbrado> TimbrarAsync()
    {
        using var client = new ServicioTimbradoWSPortTypeClient("ServicioTimbradoWSPort");
        
        try
        {            
            var response = await client.timbrarAsync("APIKEY", TimbrarCfdi());
                        
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

  ```
    
  
  {{< /tab >}}
  {{< tab >}}**Java**: YAML is a human-readable data serialization language.{{< /tab >}}
  {{< tab >}}**Python**: TOML aims to be a minimal configuration file format that's easy to read due to obvious semantics.{{< /tab >}}
  {{< tab >}}**PHP**: TOML aims to be a minimal configuration file format that's easy to read due to obvious semantics.{{< /tab >}}

{{< /tabs >}}

