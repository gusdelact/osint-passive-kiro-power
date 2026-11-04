# OSINT: Company Information

> Steering del power **osint-passive-recon**, HTB *Open-Source Intelligence*, secciones 4–10
> (Organization, Locations, Staff, Contact Information, Business Records, Services, Social Networks).

Esta categoría depende sobre todo de **navegación manual**: casi todas las fuentes (web de la
empresa, LinkedIn, Crunchbase, reseñas) se leen en el Research Browser. El agente ayuda a **armar
búsquedas** con `generate_dorks` y a buscar en GitHub con `search_github`, pero no abre páginas ni
ejecuta dorks.

> **Personas reales:** en el laboratorio solo se perfilan identidades sintéticas del caso (o
> `octocat` en GitHub). Perfilar personas reales sin autorización está fuera de alcance aunque la
> información sea pública.

## Qué hace `generate_dorks` y qué no

`openosint::generate_dorks(target=...)` no envía nada a la red: devuelve **12 plantillas fijas**
de Google con su URL: el término solo, `site:linkedin.com`, `site:x.com OR site:twitter.com`,
`site:facebook.com`, `site:instagram.com`, `site:github.com`, `filetype:pdf`, `inurl:profile`,
`resume OR cv`, `leaked OR breach OR dump`, `intitle:` y una exclusión de LinkedIn/Facebook.

Los dorks específicos del módulo (`site:docs.google.com`, `intext:"@dominio"`,
`site:pastebin.com`, `site:amazonaws.com`) **no** los genera la herramienta. El agente los escribe
como texto, marcados como "propuestos, no ejecutados", para que el humano los copie en su navegador.

## Sección 4 — Organization

Entender **procesos y dependencias** del negocio permite inferir cómo está montada la
infraestructura y en qué paso del proceso hay más exposición. Es un proceso repetible, no un
método estático.

## Sección 5 — Locations

Más relevante en Red Team físico. Fuentes: home page (países, ciudades, direcciones, mapas), redes
sociales y mapas (Google Maps/Earth, latitude.to para coordenadas).

- De las fotos de edificios se infieren ventanas, accesos secundarios, cámaras, antenas y
  mecanismos de puertas.
- Saber en qué países opera la empresa sugiere distribuciones de teclado (US/UK/DE), útil más
  adelante para listas de contraseñas. En OSINT solo se **anota**.

## Sección 6 — Staff

- **About Us** y biografías de dirección: contactos clave y temas para ingeniería social.
- **Ofertas de empleo**: revelan tecnologías actuales y futuras (lenguajes, bases de datos, nube).
- **LinkedIn**: número aproximado de empleados (cada uno es un vector de phishing) y filtrado por
  palabras clave para aislar IT y seguridad.
- Otras fuentes: Xing, Indeed, Glassdoor, TrustPilot.
- **Metadatos** de documentos publicados: ver `osint-leaks.md` (sección 21).

| Técnica HTB | En este power |
|---|---|
| Perfiles de empleados en LinkedIn | `generate_dorks(target="<empresa>")` → plantilla `site:linkedin.com`; la abre el humano |
| Repos y correos de commits del personal | `search_github(query="<empresa u org>")` (60 peticiones/hora sin token) |
| Vacantes con stack tecnológico | Lectura manual de la web y portales de empleo |

## Sección 7 — Contact Information

Teléfonos y correos alimentan escenarios de vishing y phishing. Las empresas grandes usan varios
dominios: el correo de contacto puede estar en otro.

| Recurso HTB | En este power |
|---|---|
| Hunter.io, RocketReach | Fuera del power (cuenta/llave); manual si el docente lo permite |
| Emailrep.io | Fuera del power; el estado de SPF/DMARC sí sale de `search_dns` |
| Wayback Machine (correos antiguos) | `wayback_snapshots(url="<dominio>/contact", match="prefix")` |
| Correos en certificados | `crtsh_certificate_search` también devuelve emails de los certificados |
| Buscadores | `generate_dorks` + dorks propuestos con `intext:"@<dominio>"` |

## Sección 8 — Business Records

Situación financiera y **cultura** de la empresa. Un ambiente laboral malo se traduce en errores.
Fuentes: Crunchbase, Yahoo! Finance (si cotiza), reseñas en Indeed/Glassdoor, y documentos con
`filetype:pdf` / `filetype:docx`. Prestar atención al **feedback negativo**, que la empresa no
publica.

## Sección 9 — Services

Descripciones de servicios, apps web/móviles y videos. Analizar a los **competidores** ayuda a
inferir funciones o tecnologías no publicadas. Las etiquetas **"Powered by"** delatan partners y
tecnologías (un proveedor cloud lleva a `osint-cloud-terceros.md`). Las fechas en foros técnicos
permiten estimar la antigüedad del software.

## Sección 10 — Social Networks (como elemento de información)

Las redes se tratan como **información de la empresa** (el análisis profundo es SOCMINT, otro
tema). Fuentes: blogs, newsletters, imágenes, videos, wikis, foros.

- Una foto de oficina puede mostrar SO, mensajería, modelo de laptop, aplicaciones abiertas o
  **credenciales/badges** visibles.
- Las wikis llevan a otras fuentes (GitHub, wikis internas mal publicadas).

## Qué documentar

Por cada hallazgo: fuente (URL o herramienta + argumentos), fecha y hora, captura o salida, y si
es **hecho observado** o **inferencia**. Ejemplo: "La vacante X pide experiencia en PostgreSQL"
(hecho, URL + SingleFile) → "probablemente usan PostgreSQL en producción" (inferencia).

## Preguntas de laboratorio (HTB, sin respuestas)

- Coordenadas de las sedes y número total de ubicaciones (sección 5).
- Nombre del COO y número de puestos abiertos (sección 6).
- Correo de un empleado concreto y de soporte enterprise (sección 7).
- EBIT de un trimestre concreto (sección 8).
- Nombre de la aplicación API que usa la empresa (sección 9).
- Número de redes sociales enlazadas en la web (sección 10).
