---
copyright:
  years: 2018
lastupdated: "2018-03-13"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}

# Gestión del despliegue de IBM CIS para una fiabilidad óptima

Para lograr la fiabilidad óptima para su despliegue de IBM CIS, puede configurar una configuración DNS útil y puede configurar equilibradores de carga globales. Para una fiabilidad adicional, puede utilizar nuestras reglas de página para asegurarse de que el contenido web se entregue a sus clientes, incluso aunque el servidor de origen o la memoria caché tiene un problema. Este documento proporciona detalles sobre algunas mejores prácticas para hacer que el despliegue de IBM CIS sea óptimamente fiable.

Generalmente, nuestras mejores prácticas recomendadas son estas:

 * Configure el DNS para aprovechar los servidores proxy de IBM CIS y otras características
 * Utilice uno o varios equilibradores de carga globales para distribuir si tráfico al sitio de manera uniforme
 * Configure las reglas de página apropiadas para gestionar el almacenamiento en memoria caché y otras opciones

Cada uno de estos elementos proporciona determinada funcionalidad que puede utilizar para crear un despliegue de CIS más fiable.

Tenga en cuenta que la interfaz de CIS está organizada en secciones para *seguridad*, *fiabilidad* y *rendimiento*. El menú de navegación principal se muestra en la figura siguiente, con los equilibradores de carga globales y los elementos de menú de DNS revelados:

![DNS de navegación izquierda](images/cis-left-navigation.png)


## Configuración de DNS
 
 Para empezar a configurar la configuración de DNS, seleccione **DNS** desde el menú de navegación, como se mostró anteriormente.
 
 Para obtener información detallada sobre la configuración y la gestión de DNS para la fiabilidad, [consulte este documento](dns.html#setting-up-your-domain-name-system-dns-for-ibm-cis).


## Configuración de equilibradores de carga globales


Para empezar a configurar los equilibradores de carga globales, seleccione **Equilibradores de carga globales** desde el menú de navegación.

Para obtener información detallada sobre la configuración y la gestión de los equilibradores de carga globales, [consulte este documento](glb.html#global-load-balancer-glb-concepts).

## Uso de reglas de página para aumentar la fiabilidad

Estos son algunos de los valores de Regla de página recomendados para dar a su sitio la máxima fiabilidad:

 * Siempre en línea
 * Control de memoria caché de origen
 * URL de reenvío

 ## Siempre en línea

Puede utilizar el valor de Regla de página **Siempre en línea** para guardar una versión limitada del sitio en línea si el servidor cae.

Con **Siempre en línea**, el servidor cae, IBM CIS servirá páginas desde nuestra memoria caché, para que los visitantes puedan seguir viendo algunas de las páginas que están intentando visitar. Los visitantes verán un mensaje en la parte superior de la página que les indica que están en modalidad de navegación fuera de línea. Siempre en línea devuelve un estado 503 de HTTP; sin embargo, 503 también lo utilizan muchas otras aplicaciones web. Cuando el servidor vuelva a estar en línea, IBM CIS llevará a los usuarios de nuevo a la navegación normal sin interrupciones.

Si IBM CIS no tiene la página solicitada en su memoria caché, el visitante verá una página de errores que le permitirá saber que la página del sitio web que está solicitando está fuera de línea.

### Cómo configurar Siempre en línea

Para habilitar **Siempre en línea**, siga estos pasos:

 * Utilice el menú de navegación para seleccionar las reglas de página en Rendimiento.
 * Cree una Regla de página con el patrón URL del dominio.
 * Añada el valor **Siempre en línea** con el conmutador activo.
 * Seleccione Suministro de recursos.

 ### Limitaciones de Siempre en línea

 * **Siempre en línea** almacena en la memoria caché los primeros 10 enlaces desde HTML raíz, luego solo los primeros enlaces de cada una de esas páginas, y finalmente los primeros enlaces de cada una de esas páginas posteriores. Esto significa que solo se verán algunas páginas del sitio cuando el servidor de origen caiga.

 * Los sitios recientemente añadidos no tendrán una gran memoria caché de su sitio disponible, lo que significa que **Siempre en línea** no puede funcionar si solo ha añadido el sitio hace unos días.

 * CIS no podrá mostrar contenido privado ni manejar el envío de formularios (POSTs) si el servidor está caído. A los visitantes se les mostrará un error en las páginas de pago o en los elementos que necesiten iniciar sesión para verse.

 * Para desencadenar **Siempre en línea**, el servidor web debe devolver un Código de error HTTP estándar de tiempo de espera 502 o 504. Siempre en línea también funciona cuando se encuentran problemas al ponerse en contacto con su origen (Errores 521 y 523), tiempos de espera (522 y 524), errores de SSL (525 y 526) o un error desconocido (520). **Siempre en línea** no se desencadenará para otros códigos de respuesta HTTP, como 404, 500, 503, errores de conexión de base de datos, errores de servidor interno o respuestas vacías del servidor.

 * **Siempre en línea** no funcionará si está habilitada una regla de página "Almacenar todo en la memoria caché" con el "TTL de caducidad de la memoria caché de extremo" inferior a la frecuencia de almacenamiento en la memoria caché (Clientes gratuitos: 7 días, Clientes Pro: 3 días, y Clientes Business y Enterprise: 1 día), porque el "TTL de caducidad de la memoria caché de extremo" hace que la memoria caché **Siempre en línea** se depure en el intervalo correspondiente.

## Control de memoria caché de origen

Puede utilizar el valor de Regla de página **Control de memoria caché de origen** para determinar qué contenido se almacena en la memoria caché desde su origen y con cuánta frecuencia se actualiza el contenido, lo que tiene un efecto en la fiabilidad y en el rendimiento. De forma predeterminada, si no se cambian los valores y no se envían cabeceras que impidan el almacenamiento en memoria caché desde su servidor de origen, IBM CIS almacenará en la memoria caché todo el contenido estático con determinadas extensiones. Estos tipos de contenido incluyen imágenes, CSS y JavaScript. Este almacenamiento en memoria caché es principalmente por motivos de rendimiento.

Para configurar **Control de memoria caché de origen**, utilizará reglas de página para activar cabeceras específicas que den el comportamiento deseado con respecto a cada recurso de contenido. Para comprender cómo utilizar **Control de memoria caché de origen**, se necesitan algunas explicaciones más generales de las reglas de página y un comportamiento de almacenamiento en memoria caché general para CIS para proporcionar contexto, lo que verá en las siguientes secciones. Existen tres métodos que puede utilizar para controlar el almacenamiento en memoria caché en general, y **Control de memoria caché de origen** es el segundo.

El establecimiento del **Control de memoria caché de origen** invoca las reglas de almacenamiento en memoria caché que buscan adherirse más a las mejores prácticas de Internet y RFC, principalmente con respecto a la revalidación. Por ejemplo, el comportamiento predeterminado de CIS con `max-age=0` es no almacenarse en la memoria caché en absoluto, mientras que establecer **Control de memoria caché de origen** almacena en la memoria caché, pero siempre revalida.

### Cómo configurar Control de memoria caché de origen

 * Utilice el menú de navegación para seleccionar las reglas de página en Rendimiento.
 * Cree una Regla de página con el patrón de URL que hace referencia al dominio.
 * Añada el valor **Control de memoria caché de origen** con el conmutador activo.
 * Seleccione Suministro de recursos.

### Prioridad de Regla de página

Tendrán prioridad dos Reglas de página específicas para el almacenamiento en memoria caché general:

 * Si una Regla de página tiene establecido **Nivel de memoria caché** en `Omitir`, los recursos que coincidan con dicha Regla de página no se almacenarán en la memoria caché. IBM CIS todavía actúa como un proxy, y el resto de nuestras características de rendimiento permanecerán activas. Sin embargo, el contenido no se servirá desde nuestra memoria caché, sino que se captará desde su servidor de origen directamente.

 * Si una Regla de página tiene el **Nivel de memoria caché** establecido en `Almacenarlo todo en la memoria caché`, los recursos que coincidan con la Regla de página se almacenarán en la memoria caché. **Utilizar este valor de Regla de página es la única forma de indicarnos que se almacenen recursos en la memoria caché más allá de lo que consideramos estático, incluido HTML.**

Si no hay establecida ninguna Regla de página, se utilizará la modalidad de memoria caché `Estándar`, que se basa en la extensión del recurso. Almacenaremos en la memoria caché solo recursos estáticos (como se mencionó anteriormente).

### Cabeceras cache-control de origen

La segunda forma de alterar lo que IBM CIS almacenará en la memoria caché es mediante las cabeceras de almacenamiento en la memoria caché enviadas desde el origen. CIS respetará estos valores, pero puede alterarlos temporalmente especificando un valor de Regla de página **TTL límite de almacenamiento en memoria caché**. Estas son algunas de las cabeceras que consideramos al decidir qué recursos almacenar en la memoria caché desde su origen:

 * Si la cabecera **Cache-Control** se establece en `private`, `no-store`, `no-cache` o `max-age=0`, o si hay una cookie en la respuesta, IBM CIS no almacenará en la memoria caché el recurso. Tenga en cuenta que el material confidencial no se debería almacenar en la memoria caché, por lo que podría pensar en utilizar una de estas cabeceras en tal caso.

 * Si la cabecera **Cache-Control** se establece en `public` y `max-age` es mayor que 0, o si las cabeceras `Expires` se establecen en cualquier momento del futuro, almacenaremos el recurso en la memoria caché.

**Nota:** Según las reglas de RFC, `Cache-Control: max-age` supera las cabeceras `Expires`. Si vemos los dos y no están de acuerdo, `max-age` ganará.

### Uso de la cabecera 's-maxage'

La tercera forma de controlar el comportamiento del almacenamiento en memoria caché y el comportamiento del almacenamiento en memoria caché del navegador juntos es utilizando la cabecera Cache-Control `s-maxage`.

Normalmente respetamos la directiva `max-age`:

`Cache-Control: max-age=1000`

Pero si se desea especificar un tiempo de espera de caché que es distinto del navegador, podemos utilizar `s-maxage`. A continuación hay un ejemplo que indica a IBM CIS que almacene en la memoria caché el objeto durante 200 segundos y al navegador que almacene en la memoria caché el objeto durante 60 segundos.

`Cache-Control: s-maxage=200, max-age=60`

Básicamente, `s-maxage` está pensado para ser seguido SOLO por los proxies inversos (por lo que el navegador debería ignorarlo), mientras que, por otro lado, damos prioridad a (IBM CIS) a `s-maxage` si está presente. Respetaremos el valor que sea mayor: el valor de memoria caché del navegador o la cabecera `max-age`.

### Resumen de las cabeceras cache control y de Reglas de página para su fiabilidad

Para resumir, aquí hay algunas áreas principales a considerar para la fiabilidad con respecto al almacenamiento en memoria caché:

 * Compruebe las cabeceras de almacenamiento en la memoria caché de origen para asegurarse de que no haya cabeceras que se alteren temporalmente para los recursos guardados en la memoria caché (`Cache-Control` y `Expires`).

 * CIS siempre almacena en la memoria caché contenido estático de forma predeterminada, con el TTL siguiente que depende del código de retorno:

```
200 301    120m;
302 303    20m;
403        5m; for reliability
404        5m;
any        0s;
```

 * Para almacenar en caché más, cree una Regla de página con el **Nivel de memoria caché** establecido en "Almacenarlo todo en la memoria caché" en el URL deseado (si el servidor web devuelve un 404 cuando solicite este URL, almacenaremos en la memoria caché este resultado solo durante 5 m).

 * Para evitar el almacenamiento en memoria caché en un URL, cree una Regla de página con **Nivel de memoria caché** establecido en "Omitir".


 ## URL de reenvío

Para asegurarse de que el contenido esté siempre disponible (HA), puede crear una Regla de página con el valor **URL de reenvío** que se utilizará en caso de que su sitio no esté disponible.

 **Nota:** Al habilitar un **URL de reenvío**, todos los demás valores se inhabilitan porque está enviando todo el tráfico a otro URL.

### Cómo configurar un URL de reenvío

 * Utilice el menú de navegación para seleccionar las reglas de página en Rendimiento.
 * Cree una Regla de página con el patrón de URL que hace referencia al dominio.
 * Añada el valor **URL de reenvío**.
 * Seleccione el tipo de reenvío y especifique el URL de destino.
 * Seleccione Suministro de recursos.

### Ejemplos de reenvío:

Supongamos que desea facilitar que cualquiera llegue a un URL como:

    *www.example.com/+

    *example.com/+

Este patrón coincidirá:

    http://example.com/+
    http://www.example.com/+
    https://www.example.com/+
    https://blog.example.com/+
    https://www.blog.example.com/+
    Etc...

No coincidirá:

    http://www.example.com/blog/+  [directorio adicional antes del +]
    http://www.example.com+  [sin barra inclinada final]


Una vez que haya creado el patrón que coincide con lo que desea, añada el valor **URL de reenvío** y seleccione el tipo de reenvío y especifique el URL de destino. Por ejemplo:

    https://plus.google.com/yourid

Seleccione Suministro de recursos. En pocos segundos, cualquier solicitud que coincida con el patrón se reenviará al nuevo URL con la redirección especificada.

### Opciones avanzadas de reenvío:

Si utiliza una redirección básica, como por ejemplo el reenvío del dominio raíz a www.yourdomain.com, no perderá nada más en el URL. Por ejemplo, podría configurar el patrón:

    example.com

Y reenviarlo a:

    http://www.example.com

Pero, si alguien ha especificado:

    example.com/some-particular-page.html

Se redirigirán a:

    www.example.com

en lugar de

    www.example.com/some-particular-page.html

La solución es utilizar variables. Cada comodín corresponde a una variable a la que se puede hacer referencia en la dirección de reenvío. Las variables están representadas por un `$` seguido por un número. Para hacer referencia al primer comodín, utilizará `$1`, para hacer referencia al segundo comodín, utilizará `$2`, y así sucesivamente. Para arreglar el reenvío desde la raíz hasta `www` en el ejemplo anterior, podría utilizar el mismo patrón:

    example.com/*

Entonces, configuraría el URL siguiente al que reenviar el tráfico:

    http://www.example.com/$1

En este caso, si alguien fue a:

    example.com/some-particular-page.html

Se redirigirá a:

    http://www.example.com/some-particular-page.html
