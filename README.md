# MediMun

Sitio web de **MediMun**, tienda de productos de salud: ortopedia, terapia de frío y calor, cuidado en casa y recuperación.
Hoy funciona como e-commerce; a futuro se busca abrir un canal de distribución para farmacias, clínicas, empresas y escuelas.

Proyecto académico para la materia **Applied Algorithmic Thinking** — Universidad de la Libertad.

## Estado

Mockup en `index.html` (HTML, CSS y JavaScript sin dependencias ni proceso de build), con el logo en `img/logo.png` y las fotos de producto en `img/productos/`.

- Catálogo de 20 productos con foto, descripción, precio y existencias, filtrable por categoría
- Carrito con cantidades y total
- Formulario de datos del cliente para confirmar el pedido (demo: no cobra, no envía ni guarda datos)
- **Cotizador para negocios:** calcula el precio de mayoreo por volumen (10% desde 10 piezas, 15% desde 25) y envía la cotización por correo con la API de FormSubmit. Ver [PROMPT_LOG.md](PROMPT_LOG.md).
- Diseño adaptable a celular

## Ver localmente

Abre `index.html` en el navegador.

## Deployment

Es un sitio estático, así que Vercel lo publica sin configuración:

1. En [vercel.com](https://vercel.com) → **Add New… → Project**
2. Importa este repositorio de GitHub
3. Framework Preset: **Other** (sin build command ni output directory)
4. **Deploy**

Cada push a `main` se publica automáticamente.
