# Bitácora de prompts — Assignment 2: Cotizador para negocios (MediMun)

**Proyecto:** MediMun — tienda en línea de productos de salud y ortopedia, con canal de distribución para negocios.
**Deploy:** https://proyecto-aat-ul17.vercel.app
**Repositorio:** https://github.com/yehudamemun/Proyecto-AAT-

## Función elegida

De mis 3 candidatas (Pre-Class Work de la Sesión 19) elegí la **2. Cotizador para negocios**:

> Una farmacia, clínica o escuela indica qué productos y cuántos necesita, y la app calcula automáticamente el precio de mayoreo según el volumen y redacta una cotización lista para enviar.

La elegí porque es la que sirve al objetivo de MediMun de pasar de e-commerce a distribución: los negocios compran por volumen y necesitan un precio de mayoreo antes de pedir.

---

## Paso 1 — Especificación (3 campos)

> Nota honesta: en mi sección la Sesión 19 no tuvo Breakout (solo Poll y Pre-Class Work), así que esta especificación **no fue auditada en vivo por un compañero**. La escribí con Claude Code a partir de mi candidata 2.

**1. Qué hace**
Cuando un negocio elige productos del catálogo y escribe cuántas piezas quiere de cada uno, la app:
1. calcula para cada producto el precio de mayoreo según la cantidad de ese producto: de 1 a 9 piezas, precio normal; de 10 a 24, −10%; de 25 a 50, −15%;
2. muestra al momento una tabla con producto, piezas, precio por pieza, descuento y subtotal, más el total y cuánto se ahorra;
3. no deja cotizar más piezas de las que hay en existencia (50 por producto) y lo explica con un mensaje;
4. al presionar **"Enviar cotización"**, genera un folio (`MM-AAAAMMDD-####`) con vigencia de 15 días y envía la cotización completa por correo a MediMun (leacooperml@gmail.com) con la API de FormSubmit (`POST https://formsubmit.co/ajax/<correo>`).

**2. Qué datos / inputs necesita**
| Dato | De dónde viene | Tipo |
|---|---|---|
| Nombre del negocio | Lo escribe el usuario en el cotizador | texto (obligatorio) |
| Tipo de negocio | Lo elige el usuario | opción: Farmacia, Consultorio / Clínica, Empresa, Escuela, Otro |
| Correo de contacto | Lo escribe el usuario | texto con formato de correo (obligatorio) |
| Producto | Lo elige el usuario de una lista que sale del catálogo que **ya existe** en la app (arreglo `products`) | opción de lista cerrada (20 productos) |
| Precio de lista | Ya existe en el catálogo (`products[i].price`) | número (MXN) |
| Existencias | Ya existe en el catálogo (`products[i].stock` = 50) | número entero |
| Cantidad | La escribe el usuario por producto | número entero de 1 a 50 |
| Reglas de descuento por volumen | **Dato nuevo** que agregué al código (`VOLUME_DISCOUNTS`) | lista de {mínimo de piezas, % descuento} |
| Correo de MediMun | **Dato nuevo** en el código (`CONTACT_EMAIL`) | texto |

**3. Cómo se sabrá que funciona**
- **Caso de prueba principal:** negocio "Farmacia de Prueba", tipo Farmacia, pide **30 Tobilleras de compresión elástica** ($129 c/u) y **5 Torniquetes con broche** ($79 c/u). Resultado esperado:
  - Tobillera: 30 piezas → −15% → $109.65 c/u → subtotal **$3,289.50 MXN**
  - Torniquete: 5 piezas → sin descuento → $79.00 c/u → subtotal **$395.00 MXN**
  - **Total $3,684.50 MXN**, ahorro $580.50 MXN
  - Al enviar, la API de FormSubmit responde `"success": "true"` y la cotización llega a leacooperml@gmail.com con el folio y esas mismas cifras.
- **Casos límite:** 9 piezas → 0%; 10 → 10%; 24 → 10%; 25 → 15%; pedir 51 piezas de un producto → no se envía y aparece "Solo tenemos 50 piezas…".
- **Sin datos:** si falta el nombre o el correo, no se envía nada y se muestra qué falta.

---

## Paso 2 — Construcción: decisiones de aceptar / rechazar / revisar

Herramienta: Claude Code (Claude Opus 5.5), trabajando directamente sobre mi repositorio.

| # | Qué propuso o hizo la IA | Campo de la especificación en juego | Decisión | Por qué |
|---|---|---|---|---|
| 1 | La primera "función" que la IA construyó fue un formulario de mayoristas que enviaba nombre, tipo y correo a mi email. Funcionaba, pero **no era ninguna de mis 3 candidatas**: no calculaba precios ni cotizaba. La propia IA lo señaló al revisar la rúbrica. | **Qué hace** | **Revisar** | Pedí construir mi candidata 2 (cotizador). El formulario se reemplazó por el cotizador y solo se reutilizó el envío por correo. |
| 2 | Para enviar los datos al correo, la IA propuso tres caminos: Formspree (requiere crear cuenta), un botón que abre el correo del cliente (`mailto:`), o FormSubmit (sin cuenta). | **Qué datos necesita** (a dónde van los datos) | **Aceptar FormSubmit** | Lleva los datos directo a mi correo sin depender de que el cliente tenga correo configurado. La IA avisó que los datos pasan por un servicio externo y que la primera vez hay que activar el formulario desde el correo; lo acepté sabiéndolo. |
| 3 | Yo no tenía definidos los descuentos de mayoreo. La IA propuso 10% desde 10 piezas y 15% desde 25 piezas, **por producto**. | **Qué hace** (regla de cálculo) y **Qué datos necesita** (`VOLUME_DISCOUNTS`) | **Aceptar como provisional** | Le pedí que usara los datos más acertados posibles. La regla queda en una sola constante para cambiarla cuando confirme mis márgenes con mi proveedor. |
| 4 | La IA agregó por su cuenta que no se pueda cotizar más de las existencias (50 por producto). No estaba en mi candidata original. | **Cómo se sabrá que funciona** (caso límite de 51 piezas) | **Aceptar** | Ya había pedido "50 piezas restantes" por producto; cotizar más de lo que hay crearía promesas que no puedo surtir. |
| 5 | Antes, al armar el catálogo, la IA puso fotos genéricas de un banco de imágenes (Pexels) porque las fotos de mis Excel eran miniaturas. | **Qué datos necesita** (la fuente de las fotos de producto que usa el cotizador y el catálogo) | **Rechazar** | Las fotos no mostraban mi modelo exacto de producto y podían confundir al cliente. Las reemplacé por fotos de mi propio producto, 3 por artículo. |
| 6 | Para 3 productos sin dato de Mercado Libre (codera, paños de baño en seco, bota walker corta) la IA **estimó** el precio y lo marcó como estimado. | **Qué datos necesita** (precio de lista del catálogo, que es la base del cálculo) | **Revisar (pendiente)** | Acepto los precios para la entrega, pero quedan marcados para confirmar, porque cualquier cotización de esos productos depende de ese número. |

Prompts clave que le di a la IA (textuales):
- "quiero que me cambies todos los productos a estos que te voy a dar … pon en cada 1 50 pcs restantes de $ … ponlo un poco bajo de lo que esta en ml"
- "para esto quiero que llegue a leacooperml@gmail.com"
- "busca mas fotos en internet … quiero fotos pros" → después: "esta ponla en tobillera de compresion elastica" (reemplazo por fotos propias)
- "NO HAY BREAKOUT SOLO POLL Y PREWORK YA CONTESTA CON ESTO Y SI NO TIENES LOS DATOS HAZLOS LOS MAS ACERTADOS POSIBLES"

Prueba previa en mi computadora (sin enviar nada: se interceptó el envío a FormSubmit). Así se confirmó el cálculo antes de publicar:
```
Producto	Pzs	Precio c/u	Desc.	Subtotal
Tobillera de compresión elástica	30	$109.65 MXN	-15%	$3,289.50 MXN
Torniquete con broche	5	$79.00 MXN	—	$395.00 MXN
Total (ahorras $580.50 MXN)	$3,684.50 MXN
```
Límites: 9 → 0%, 10 → 10%, 24 → 10%, 25 → 15%, 50 → 15%. 51 piezas → "Solo tenemos 50 piezas de "Tobillera de compresión elástica" (pediste 51)." Sin nombre ni correo → "Escribe el nombre del negocio." / "Escribe un correo de contacto válido." y no se envió nada.

---

## Paso 3 — Validación en el deploy

> PENDIENTE: se completa al correr el caso de prueba principal en https://proyecto-aat-ul17.vercel.app con el envío real a FormSubmit.

- **Deploy responde:** _pendiente_
- **Input exacto:** _pendiente_
- **Output real (copiado tal cual):** _pendiente_
- **Veredicto:** _pendiente_

---

## Uso de IA

- **Herramienta:** Claude Code (modelo Claude Opus 5.5) dentro de la app de escritorio de Claude.
- **Para qué la usé:** escribir y modificar el código de `index.html` (catálogo, carrito, galería de fotos y el cotizador), recortar las fotos de mis collages, sacar precios de referencia de mis propios Excel de análisis de Mercado Libre, redactar esta bitácora y correr las pruebas en el navegador.
- **Qué decidí o verifiqué yo:** elegí la función (candidata 2), los productos, el correo al que llegan las cotizaciones, que los precios quedaran un poco abajo de Mercado Libre, las 50 piezas por producto, rechacé las fotos genéricas y aporté las fotos de mis productos, y acepté como provisional la regla de descuentos por volumen.
