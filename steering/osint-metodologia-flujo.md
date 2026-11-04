# OSINT: metodología y flujo de trabajo

> Guía de steering del power **osint-passive-recon**, derivada del módulo *Open-Source
> Intelligence* de HTB Academy (secciones 1, 2 y 3). Es el punto de partida: explica los
> principios, la frontera legal pasivo/activo y el flujo por fases con las herramientas MCP del
> power. Las demás guías `osint-*.md` desarrollan cada categoría de información.
>
> **Encuadre:** solo se investigan objetivos autorizados (dominio de laboratorio, dominio propio o
> permiso por escrito). OSINT es **solo pasivo**: se consulta a terceros, nunca al objetivo.

## Qué es OSINT (sección 1)

OSINT es encontrar información **públicamente disponible** sobre una empresa o personas para
identificar eventos, dependencias internas y externas y conexiones. La palabra clave es
**Intelligence**: no basta con juntar datos sueltos, hay que **relacionarlos** hasta formar patrones
y perfiles del objetivo.

- Elegir herramientas es apenas **~10 %** del trabajo. Ninguna reemplaza el análisis manual: cada
  una depende de unas fuentes concretas y por eso está limitada.
- Definición operativa: información de fuentes públicas, obtenible por cualquiera sin
  restricciones, de forma legal y éticamente aceptable.

### La frontera legal (crítica)

OSINT usa fuentes públicas accesibles **sin registro ni autenticación** y **sin interactuar** con el
objetivo. Cualquier enumeración activa ya **no es OSINT** y puede violar el contrato o ser delito.

| Actividad | ¿OSINT? | En este power |
|---|---|---|
| Ver la web pública, "About Us", vacantes | Sí | La abre el humano en su navegador |
| Buscar empleados en redes sociales | Sí | `generate_dorks` arma la búsqueda; la abre el humano |
| Revisar logs de Certificate Transparency | Sí | `crtsh_certificate_search` |
| Archivos históricos (Wayback, Pastebin) | Sí | `wayback_snapshots`, `search_paste` |
| Terceros que **ya** recopilaron info del objetivo | Sí | RDAP/WHOIS, DNS público, `search_ip` |
| Metadatos de archivos **ya publicados** | Sí | `exiftool_metadata` sobre una copia local |
| Brute-forcing de subdominios o directorios | **No** | Fuera de alcance |
| Transferencia de zona (AXFR) | **No** | Fuera de alcance (`dnsrecon` apagado) |
| Terceros que **escanean por encargo** al objetivo | **No** | Fuera de alcance |
| Listar o abrir un bucket/servicio encontrado | **No** | Se documenta y se detiene |

> Ejemplo del curso: encontrar un bucket S3 abierto **adivinando nombres** ya es una violación de
> contrato aunque nunca se abra. Regla mental del caso ACME: no es "mirar sin tocar", es
> **"entender sin tocar"**.

## Metodología (sección 2)

OSINT se apoya en dos conceptos que dependen uno del otro:

- **Core Elements**, lo que buscamos: nombres y versiones de software, servidores, usuarios,
  hashes, URLs, contraseñas.
- **Information Resources**, de dónde lo sacamos: home page, archivos, redes sociales, buscadores,
  plataformas de desarrollo, fuentes de leaks.

| Company Information | Infrastructure | Leaks |
|---|---|---|
| Organization | Domain Information | Archives |
| Locations | Public Domain Records | Internal Leaks |
| Staff | Domain Structure | Breaches |
| Contact Information | Cloud Storage | |
| Business Records | Email Addresses | |
| Services | Third Parties | |
| Social Networks | Technologies in Use | |
| | Compounded Networks | |

### Principios

1. Trabajar en **ciclos**, de lo **general a lo detallado**. Cada dato nuevo abre otro ciclo (un
   subdominio nuevo de crt.sh se vuelve a pasar por DNS y Wayback).
2. **No hay orden fijo** de recursos. Es un flujo estructurado, no una receta paso a paso.
3. **Se adaptan los recursos a la metodología, no la metodología a los recursos.** Primero decides
   qué categoría investigas; después eliges la herramienta.

### Dos navegadores (trabajo del humano)

- **Research Browser**: solo para buscar. Historial limpio al empezar y exportado al final con
  *History Master* (timestamp, título, URL, query). Aquí se abren los dorks que genera el agente.
- **Resource Browser**: guarda lo encontrado con *SingleFile* (un `.html` autocontenido por
  página), que sirve de evidencia y de búsqueda offline.

```bash
csvtojson < history.csv | jq .                     # historial exportado -> JSON legible
cat *.html | html2text | grep "Nombre Apellido"     # buscar en las páginas guardadas
```

## Investigación de negocio: preguntas guía (sección 3)

- **Company Information:** ¿cuántos empleados?, ¿qué servicios ofrece?, ¿posición en el mercado?,
  ¿situación financiera?, ¿ubicaciones?, ¿seguridad física?, ¿reputación?
- **Staff:** ¿qué puestos y departamentos existen?, ¿de qué es responsable cada uno?, ¿qué
  dependencias tienen?
- **Infrastructure:** ¿cómo está montada la IT?, ¿quiénes son los admins?, ¿qué tecnologías usan?,
  ¿cuántos dominios?, ¿ASN y netblocks?, ¿certificados?, ¿terceros?, ¿servidores públicos?, ¿emails?

## Flujo recomendado por fases (OODA en cada paso)

Cada llamada a herramienta es un **Act**: antes decide la consulta que resuelve tu duda con menos
exposición; después vuelve a **Observe**, lee la salida cruda y separa hechos de inferencias.

| Fase | Pregunta | Herramienta MCP | Guía |
|---|---|---|---|
| 0. Alcance | ¿Qué objetivo está autorizado y para qué? | (ninguna; se declara por escrito) | este documento |
| 1. Registro | ¿Quién registró el dominio, cuándo, con quién? | `search_rdap` → `search_whois` / `whois_lookup` | `osint-infraestructura-dominio.md` |
| 2. DNS | ¿Qué registros publica? ¿SPF/DMARC? | `search_dns` + `dns_lookup` (cross-check) | `osint-infraestructura-dominio.md` |
| 3. Netblock / ASN | ¿Dónde está alojado? | `search_ip`, `whois_lookup(query=<ip>)` | `osint-infraestructura-dominio.md` |
| 4. Subdominios | ¿Qué nombres aparecen en certificados? | `crtsh_certificate_search` | `osint-infraestructura-dominio.md` |
| 5. Huella histórica | ¿Qué existió y ya no se ve? | `wayback_snapshots` | `osint-leaks.md` |
| 6. Empresa y personas | ¿Qué publican la empresa y su personal? | `generate_dorks`, `search_github` | `osint-company-information.md` |
| 7. Cloud y terceros | ¿Qué proveedores aparecen en DNS/TXT? | (inferencia sobre fases 2–5) | `osint-cloud-terceros.md` |
| 8. Leaks | ¿Hay volcados o documentos con metadatos? | `search_paste`, `exiftool_metadata` | `osint-leaks.md` |
| 9. Intelligence | ¿Cómo se conectan los hallazgos? | (análisis) | `osint-intelligence-reporte.md` |

Los subdominios nuevos de la fase 4 o 5 vuelven a la fase 2 (nuevo ciclo). Cada salida se guarda en
`evidencia/osint/` del workspace (ver `osint-intelligence-reporte.md`).

## Mapa de guías de steering de este power

| Guía | Secciones HTB | Cuándo usarla |
|---|---|---|
| `osint-metodologia-flujo.md` | 1, 2, 3 | Punto de partida: principios, frontera legal y flujo. |
| `osint-company-information.md` | 4–10 | Organización, ubicaciones, personal, contacto, negocio, servicios, redes. |
| `osint-infraestructura-dominio.md` | 11, 12, 13 | Registro del dominio, DNS, ASN/netblocks, subdominios. |
| `osint-cloud-terceros.md` | 14, 16 | Almacenamiento cloud y proveedores externos. |
| `osint-emails-redes-tecnologias.md` | 15, 17, 18 | Formato de email, redes mixtas, stack tecnológico. |
| `osint-leaks.md` | 19, 20, 21, 22 | Archivos históricos, metadatos, breaches. |
| `osint-intelligence-reporte.md` | 23 | Origin → Impact → State, evidencia e informe. |
| `osint-herramientas-mcp.md` | (transversal) | Referencia de herramientas, guardrails y errores comunes. |
