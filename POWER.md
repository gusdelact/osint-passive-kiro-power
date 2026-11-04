---
name: "osint-passive-recon"
displayName: "OSINT Passive Recon"
description: "Curso completo de OSINT pasivo (HTB Academy, 23 secciones) con herramientas MCP sin API keys: Company Information, Infrastructure y Leaks, cerrando con Origin -> Impact -> State bajo metodología OODA. Combina OpenOSINT (RDAP, WHOIS, DNS, dorks, GitHub, pastes, IP) y OSINT Toolbox (crt.sh, Wayback Machine, ExifTool). Incluye guías de steering por categoría, referencia de herramientas, reglas de evidencia y plantilla de informe."
keywords: ["osint", "reconocimiento-pasivo", "footprinting", "hack-the-box", "whois-rdap", "certificate-transparency", "wayback-machine"]
author: "Gustavo De la Cruz Tovar"
---

# OSINT Passive Recon

## Overview

Este power entrega el **conocimiento del módulo Open-Source Intelligence de HTB Academy (23
secciones)** y dos servidores MCP para practicarlo de forma **100 % pasiva** sobre un dominio u
organización autorizada:

- **OpenOSINT**: RDAP, WHOIS, DNS con alertas SPF/DMARC, información de IP/ASN, generación de
  Google dorks, búsqueda en GitHub y en pastes.
- **OSINT Toolbox**: certificados vía crt.sh (Certificate Transparency), snapshots de Wayback
  Machine, WHOIS/DNS de cross-check y metadatos de archivos locales con ExifTool.

Ninguna herramienta habilitada envía tráfico al objetivo: todas consultan a terceros (registros,
resolvers DNS, crt.sh, web.archive.org, ipinfo.io, GitHub) o trabajan en local.

OSINT **no busca entrar**: busca **entender sin tocar** cuánto puede saber cualquiera sobre la
organización, y dejarlo documentado con evidencia.

> **Regla de encuadre:** solo se investigan objetivos dentro del alcance autorizado (dominio de
> laboratorio como `inlanefreight.htb`/`example.com`, dominio propio o permiso por escrito). Este
> material es didáctico y no concede autorización por sí mismo. Si encuentras un bucket o servicio
> expuesto, **lo documentas y te detienes**.

Creado para el curso de **Hackeo Ético, Ingeniería Cibernética (ULSA)** con asistencia de Kiro. El
criterio de selección de los servidores está en `ia_ethicalhack/osint/Investigacion_MCP_Server_OSINT.md`
del repositorio del curso.

## Cómo está organizado el conocimiento (steering/)

Las 23 secciones están repartidas en guías de steering que se cargan **bajo demanda** (acción
`readSteering`): lee solo la que necesites según la fase.

| Guía | Secciones HTB | Contenido |
|---|---|---|
| `osint-metodologia-flujo.md` | 1, 2, 3 | Principios, frontera legal pasivo/activo, dos navegadores, flujo por fases con las herramientas MCP. **Léela primero.** |
| `osint-company-information.md` | 4–10 | Organization, Locations, Staff, Contact Information, Business Records, Services, Social Networks. |
| `osint-infraestructura-dominio.md` | 11, 12, 13 | Domain Information, Public Domain Records (registro, DNS, ASN, netblocks), Domain Structure (subdominios). |
| `osint-cloud-terceros.md` | 14, 16 | Cloud Storage (S3, Blob, GCS) y Third Parties, con la regla de detención. |
| `osint-emails-redes-tecnologias.md` | 15, 17, 18 | Email Addresses, Compounded Networks, Technologies in Use. |
| `osint-leaks.md` | 19–22 | Leaked Information, Archives (Wayback, Pastebin), Internal Leaks (metadatos), Breaches. |
| `osint-intelligence-reporte.md` | 23 | Origin → Impact → State, estructura de evidencia y plantilla de informe. |
| `osint-herramientas-mcp.md` | (transversal) | Argumentos de cada herramienta, qué está apagado y por qué, verificación sin LLM, errores comunes. |

Cada guía incluye una tabla **técnica HTB → herramienta del power**, que separa lo que cubre el MCP,
lo que hace el humano en su navegador y lo que queda fuera de alcance (activo o con llave).

## Principios de OSINT (secciones 1–2)

- **Intelligence, no recolección.** El valor está en relacionar los datos, no en acumularlos. Elegir
  herramientas es ~10 % del trabajo.
- **Ciclos de lo general a lo detallado.** Cada hallazgo abre otro ciclo (un subdominio nuevo vuelve
  a pasar por DNS y Wayback).
- **Se adaptan los recursos a la metodología**, no al revés.
- **Solo pasivo.** Brute-forcing de subdominios o directorios, AXFR, escaneos, abrir buckets o probar
  credenciales **no son OSINT** y están fuera de alcance.

| Company Information | Infrastructure | Leaks |
|---|---|---|
| Organization, Locations, Staff, Contact Information, Business Records, Services, Social Networks | Domain Information, Public Domain Records, Domain Structure, Cloud Storage, Email Addresses, Third Parties, Compounded Networks, Technologies in Use | Archives, Internal Leaks, Breaches |

Cierre: **Origin → Impact → State** (sección 23), que conecta cuándo se originó cada hallazgo, qué
impacto tiene y qué estado se observa hoy.

## Metodología cognitiva: OSINT con OODA

| Etapa | Pregunta | Qué haces |
|---|---|---|
| **Observe** | ¿Qué sé realmente? | Lee la salida cruda de la herramienta. Separa **hechos** (MX apunta a `outlook.com`) de **inferencias** (usan Microsoft 365). |
| **Orient** | ¿Qué significa? | Relaciona el dato con la categoría HTB y con otros hallazgos. Formula hipótesis. |
| **Decide** | ¿Qué conviene y está autorizado? | Elige la consulta pasiva que reduzca la incertidumbre. Confirma que no toca al objetivo. |
| **Act** | ¿Qué pasó? | Ejecuta, guarda la salida en `evidencia/osint/` y vuelve a Observe. |

### Reglas cognitivas (críticas para el alumno)

1. **Un indicador no es una vulnerabilidad.** DMARC `p=none` o un WordPress antiguo en Wayback son
   exposición a validar en una fase autorizada.
2. **Registra por separado** hechos, inferencias, hipótesis y pendientes.
3. **Toda observación lleva procedencia** (herramienta + argumentos, o URL) y **fecha**. Un snapshot
   de 2019 describe 2019.
4. **Nada sin salida real.** Si una herramienta falla o devuelve vacío, se reporta así; no se rellena
   con conocimiento del modelo.
5. **Las herramientas son limitadas**: crt.sh falla, RDAP no cubre todos los TLD, ipinfo tiene cuota.
   Contrasta con una segunda fuente (`whois_lookup`, `dns_lookup`).
6. **La información de una fase alimenta a otra.** Encadenar es el corazón del módulo.

## Onboarding

### Prerequisitos

- **`uv`** (trae `uvx`) para ejecutar ambos servidores sin clonar repositorios ni usar Docker:
  ```bash
  curl -LsSf https://astral.sh/uv/install.sh | sh
  ```
- Opcional: **ExifTool** para `exiftool_metadata` (`brew install exiftool` o `sudo apt install exiftool`).
- Opcional: **Node.js 22.19+** solo para depurar con el MCP Inspector.

### Instalación

Es un **Guided MCP Power**: al instalarlo, Kiro añade los dos servidores de `mcp.json`. No requiere
API keys, tokens ni cuentas.

Calentar la caché antes del primer uso (OpenOSINT trae ~20 dependencias):

```bash
uvx --python 3.12 --from openosint==2.36.1 openosint-mcp --help
uvx osint-toolbox-mcp@1.1.0 --check
```

### Configuración incluida en `mcp.json`

- **`openosint`**: 13 herramientas ocultas con `disabledTools` (las que piden llave, un extra o lanzan
  un agente propio). Modo pasivo por defecto; **nunca** añadir `--allow-active`.
- **`osint-toolbox`**: las CLIs externas (Sherlock, Maigret, Holehe, GHunt, theHarvester, subfinder,
  dnstwist, dnsrecon, PhoneInfoga, SpiderFoot, Blackbird) apuntan a `/nonexistent/disabled` para que
  el servidor no las detecte aunque estén instaladas (p. ej. en Kali). `osint_toolbox_status` oculta.
- **Sin `autoApprove`**: cada llamada se aprueba a mano para mantener trazabilidad.

## Herramientas disponibles (resumen)

| Servidor | Herramienta | Uso principal | Guía |
|---|---|---|---|
| `openosint` | `search_rdap`, `search_whois` | Registro del dominio | `osint-infraestructura-dominio.md` |
| `openosint` | `search_dns` | Registros DNS + alertas SPF/DMARC | `osint-infraestructura-dominio.md` |
| `openosint` | `search_ip` | ASN, organización y proveedor de una IP | `osint-infraestructura-dominio.md` |
| `openosint` | `generate_dorks` | URLs de Google dorks (local; las abre el humano) | `osint-company-information.md` |
| `openosint` | `search_github` | Repos, perfiles y correos de commits | `osint-company-information.md` |
| `openosint` | `search_paste` | Volcados de Pastebin (psbdmp.ws) | `osint-leaks.md` |
| `osint-toolbox` | `crtsh_certificate_search` | Subdominios y correos en certificados | `osint-infraestructura-dominio.md` |
| `osint-toolbox` | `wayback_snapshots` | Huella histórica, rutas, tecnologías retiradas | `osint-leaks.md` |
| `osint-toolbox` | `whois_lookup`, `dns_lookup` | Cross-check; WHOIS de IP/ASN, PTR, CAA | `osint-infraestructura-dominio.md` |
| `osint-toolbox` | `exiftool_metadata` | Metadatos de documentos públicos descargados | `osint-leaks.md` |

Argumentos, límites y lo que está apagado: `osint-herramientas-mcp.md`.

## Flujo de trabajo típico

1. **Alcance:** declarar objetivo, propósito y autorización en `evidencia/osint/<objetivo>-<fecha>/00-alcance.md`.
2. **Registro:** `search_rdap(domain=...)`; si el TLD no tiene RDAP, `search_whois` o `whois_lookup`.
3. **DNS:** `search_dns` sobre la raíz y `www`; cross-check con `dns_lookup`. Revisar SPF/DMARC y TXT.
4. **IP → ASN → proveedor:** `search_ip(ip=...)` y `whois_lookup(query=<ip>)`.
5. **Subdominios:** `crtsh_certificate_search(domain=...)`; si da HTTP 502, reintentar y, si persiste,
   dejarlo como pendiente. Cada subdominio nuevo vuelve al paso 3.
6. **Huella histórica:** `wayback_snapshots(url=..., match="domain", limit=50)` y `robots.txt`.
7. **Empresa y personas:** `generate_dorks(target=...)` (el humano abre las URLs) y `search_github`.
8. **Leaks:** `search_paste`; `exiftool_metadata` sobre documentos públicos que el humano descargó.
9. **Intelligence e informe:** Origin → Impact → State con la plantilla de `osint-intelligence-reporte.md`.

**Ejemplo de prompt** (adaptar `TU-DOMINIO` al objetivo autorizado):

> Eres un analista OSINT. Objetivo autorizado: TU-DOMINIO. Usa solo técnicas pasivas: nada de
> transferencias de zona, escaneos ni accesos al objetivo. Si encuentras un bucket o servicio
> expuesto, documéntalo y detente. Usa solo las herramientas MCP de `openosint` y `osint-toolbox`.
> En cada hallazgo indica la herramienta y los argumentos, y no afirmes nada que no aparezca en una
> salida real. Si una herramienta falla, dilo en el informe. Guarda cada salida cruda en
> `evidencia/osint/`. Entrega la tabla de hallazgos, el cierre Origin → Impact → State y la lista de
> lo que decidiste no hacer.

## Evidencia y documentación

> **Regla de persistencia de evidencia (obligatoria):** la evidencia cruda **se guarda** en el
> workspace del proyecto, en `evidencia/osint/<objetivo>-<fecha>/`, nunca solo en `/tmp`. Cada salida
> de herramienta se guarda sin editar, con herramienta, argumentos y fecha UTC. Los documentos
> descargados van con su hash SHA-256.

El informe registra: alcance, hallazgos clasificados (hecho / inferencia / hipótesis), tabla de
infraestructura, dorks generados (no ejecutados por el agente), Origin → Impact → State, pendientes y
fallos de herramientas, decisiones de no hacer, recomendaciones y fuentes. Plantilla completa en
`osint-intelligence-reporte.md`.

## Errores comunes (resumen)

| Síntoma | Solución |
|---|---|
| `crt.sh answered HTTP 502` | Reintentar 1–2 veces; si persiste, pendiente en el informe. Nunca inventar datos. |
| `No RDAP server found for TLD` | Usar `search_whois` o `whois_lookup`. |
| El agente usa la búsqueda web del host | Pedir "solo herramientas MCP de openosint y osint-toolbox". |
| Primer arranque lento o timeout | Calentar la caché de `uvx` antes. |
| Kiro no encuentra `uvx` | Ruta absoluta de `which uvx` en `command`. |
| `search_dns` no reporta DKIM | Es activo; documentarlo como fuera de alcance. |

Detalle y más casos en `osint-herramientas-mcp.md`.

## Buenas prácticas

- **Nunca usar `--allow-active`** ni reactivar las CLIs del toolbox. Si un ejercicio necesita técnicas
  activas, es otra fase con otra autorización.
- **Aprobar cada llamada a mano** (sin `autoApprove`).
- **No ejecutar dorks de forma automatizada**: son URLs para que una persona las abra.
- **No instalar software** que el agente sugiera durante un ejercicio guiado.
- **No perfilar personas reales**: en el lab, solo identidades sintéticas del caso.
- **Ofuscar datos sensibles** (credenciales, datos personales) en el informe: se documenta su
  existencia, no su valor.
- **Tratar los hallazgos como indicadores de exposición**, no como vulnerabilidades confirmadas.

## Preguntas didácticas (úsalas en cada fase)

- ¿Qué afirmación **observaste** y cuál solo **inferiste**?
- ¿Qué salida de herramienta (o URL) respalda tu conclusión, y de qué fecha es?
- ¿Qué evidencia adicional confirmaría o refutaría la hipótesis? ¿Obtenerla sería pasivo?
- ¿Alguna consulta tocó al objetivo o a un tercero que escanea por encargo?
- ¿Cómo explicarías el riesgo al equipo técnico y a la dirección sin tecnicismos?

## MCP Config Placeholders

**No hay placeholders**: `mcp.json` usa paquetes públicos (`openosint`, `osint-toolbox-mcp`) vía `uvx`,
sin rutas ni credenciales del usuario.

## Referencias

- HTB Academy, módulo *Open-Source Intelligence* (23 secciones), PDFs en `osint/osint_HTB/`.
- Material local: `osint/OSINT_HTB_Resumen.md`, `osint/Lab_OSINT_ACME.md`,
  `ia_ethicalhack/osint/Investigacion_MCP_Server_OSINT.md`.

---

**Paquetes:** `openosint==2.36.1` (PyPI), `osint-toolbox-mcp@1.1.0` (PyPI)
**Servidores MCP:** `openosint`, `osint-toolbox`
