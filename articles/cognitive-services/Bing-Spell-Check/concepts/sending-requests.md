---
title: Envío de solicitudes a Bing Spell Check API
titleSuffix: Azure Cognitive Services
description: Conozca los modos de Bing Spell Check, la configuración y otra información relacionada con la API.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-spell-check
ms.topic: conceptual
ms.date: 06/27/2019
ms.author: aahi
ms.openlocfilehash: 893317b8f46415b1df540d67ebf28b65c5ba6d32
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2020
ms.locfileid: "68883449"
---
# <a name="sending-requests-to-the-bing-spell-check-api"></a>Envío de solicitudes a Bing Spell Check API

Para buscar en una cadena de texto existente errores ortográficos y gramaticales, enviaría una solicitud GET al punto de conexión siguiente:  

```
https://api.cognitive.microsoft.com/bing/v7.0/spellcheck
```  
  
La solicitud debe usar el protocolo HTTPS.

Se recomienda que todas las solicitudes procedan de un servidor. Al distribuir la clave como parte de una aplicación cliente, existe una mayor probabilidad de que un tercero malintencionado acceda a ella. Un servidor también proporciona un único punto de actualización para futuras versiones de la API.

La solicitud debe especificar el parámetro de consulta [text](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-spell-check-api-v5-reference#text), que contiene la cadena de texto para probar. Aunque es opcional, la solicitud debe especificar también el parámetro de consulta [mkt](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-spell-check-api-v5-reference#mkt), que identifica el mercado de donde desea que procedan los resultados. Para ver una lista de parámetros de consulta opcionales, como `mode`, consulte [Query Parameters](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-spell-check-api-v5-reference#query-parameters) (Parámetros de consulta). Todos los valores de parámetro de consulta deben estar codificados con una dirección URL.  
  
La solicitud debe especificar el encabezado [Ocp-Apim-Subscription-Key](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-spell-check-api-v5-reference#subscriptionkey). Aunque es opcional, se recomienda especificar los encabezados siguientes. Estos encabezados ayudan a Bing Spell Check API a obtener resultados más precisos:  
  
-   [User-Agent](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-spell-check-api-v5-reference#useragent)  
-   [X-MSEdge-ClientID](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-spell-check-api-v5-reference#clientid)  
-   [X-Search-ClientIP](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-spell-check-api-v5-reference#clientip)  
-   [X-Search-Location](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-spell-check-api-v5-reference#location)  

Para ver una lista de todos los encabezados de solicitud y respuesta, consulte [Encabezados](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-spell-check-api-v5-reference#headers).

Cuando se llama a Bing Spell Check API desde JavaScript, las características de seguridad integradas de su explorador podrían impedirle acceder a los valores de estos encabezados.

Para resolver este problema, puede realizar una solicitud de Bing Spell Check API a través de un proxy CORS. La respuesta de un proxy de este tipo tiene un encabezado `Access-Control-Expose-Headers` que agrega los encabezados de respuesta a listas blancas y hace que estén disponibles para JavaScript.

Es fácil instalar un proxy CORS para permitir que la [aplicación de tutorial](../tutorials/spellcheck.md) obtenga acceso a los encabezados de cliente opcionales. En primer lugar, si aún no lo tiene, [instale Node.js](https://nodejs.org/en/download/). A continuación, escriba el siguiente comando en un símbolo del sistema.

    npm install -g cors-proxy-server

Luego, cambie el punto de conexión de Bing Spell Check API del archivo HTML a:

    http://localhost:9090/https://api.cognitive.microsoft.com/bing/v7.0/spellcheck/

Por último, inicie el proxy CORS con el siguiente comando:

    cors-proxy-server

Deje abierta la ventana de comandos mientras usa la aplicación del tutorial, ya que si la cierra, se detendrá el proxy. En la sección expandible de encabezados HTTP que está situada bajo los resultados de búsqueda, ahora puede ver el encabezado `X-MSEdge-ClientID` (entre otras cosas) y comprobar que es el mismo para cada solicitud.

## <a name="example-api-request"></a>Solicitud de API de ejemplo

A continuación se muestra una solicitud que incluye todos los encabezados y parámetros de consulta sugeridos. Si es la primera vez que llama a cualquiera de las API de Bing, no incluya el encabezado de identificador de cliente. Solo debe incluir el identificador de cliente si se ha llamado previamente a una API de Bing y Bing ha devuelto un identificador de cliente para esa combinación de usuario y dispositivo. 
  
> ```  
> GET https://api.cognitive.microsoft.com/bing/v7.0/spellcheck?text=when+its+your+turn+turn,+john,+come+runing&mkt=en-us HTTP/1.1
> Ocp-Apim-Subscription-Key: 123456789ABCDE  
> X-MSEdge-ClientIP: 999.999.999.999  
> X-Search-Location: lat:47.60357;long:-122.3295;re:100  
> X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
> Host: api.cognitive.microsoft.com  
> ```  

A continuación se muestra la respuesta a la solicitud anterior. En el ejemplo también se muestran los encabezados de respuesta específicos de Bing.

[!INCLUDE [cognitive-services-bing-url-note](../../../../includes/cognitive-services-bing-url-note.md)]

```json
BingAPIs-TraceId: 76DD2C2549B94F9FB55B4BD6FEB6AC
X-MSEdge-ClientID: 1C3352B306E669780D58D607B96869
BingAPIs-Market: en-US

{  
    "_type" : "SpellCheck",  
    "flaggedTokens" : [{  
        "offset" : 5,  
        "token" : "its",  
        "type" : "UnknownToken",  
        "suggestions" : [{  
            "suggestion" : "it's",  
            "score" : 1  
        }]  
    },  
    {  
        "offset" : 25,  
        "token" : "john",  
        "type" : "UnknownToken",  
        "suggestions" : [{  
            "suggestion" : "John",  
            "score" : 1  
        }]  
    },  
    {  
        "offset" : 19,  
        "token" : "turn",  
        "type" : "RepeatedToken",  
        "suggestions" : [{  
            "suggestion" : "",  
            "score" : 1  
        }]  
    },  
    {  
        "offset" : 35,  
        "token" : "runing",  
        "type" : "UnknownToken",  
        "suggestions" : [{  
            "suggestion" : "running",  
            "score" : 1  
        }]  
    }]  
}  
```  

## <a name="next-steps"></a>Pasos siguientes

- [¿Qué es Bing Spell Check API?](../overview.md)
- [Referencia de Bing Spell Check API v7](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-spell-check-api-v7-reference)
