---
name: "osint-passive-recon"
displayName: "OSINT Passive Recon"
description: "Curso completo de OSINT pasivo con herramientas MCP sin API keys: Company Information, Infrastructure y Leaks, cerrando con Origin -> Impact -> State bajo metodología OODA. Combina OpenOSINT (RDAP, WHOIS, DNS, dorks, GitHub, pastes, IP) y OSINT Toolbox (crt.sh, Wayback Machine, ExifTool). Incluye guías de steering por categoría, referencia de herramientas, reglas de evidencia y plantilla de informe."
keywords: ["osint", "reconocimiento-pasivo", "footprinting", "whois-rdap", "certificate-transparency", "wayback-machine"]
author: "Gustavo De la Cruz Tovar"
---

# OSINT Passive Recon

## Overview

Este power entrega un **cuerpo de conocimiento de OSINT pasivo** (metodología, categorías de
información y flujo de trabajo) y dos servidores MCP para practicarlo de forma **100 % pasiva** sobre
un dominio u organización autorizada:

- **OpenOSINT**: RDAP, WHOIS, DNS con alertas SPF/DMARC, información de IP/ASN, generación de
  Google dorks, búsqueda en GitHub y en pastes.
- **OSINT Toolbox**: certificados vía crt.sh (Certificate Transparency), snapshots de Wayback
  Machine, WHOIS/DNS de cross-check y metadatos de archivos locales con ExifTool.

Ninguna herramienta habilitada envía tráfico al objetivo: todas consultan a terceros (registros,
resolvers DNS, crt.sh, web.archive.org, ipinfo.io, GitHub) o trabajan en local.

OSINT **no busca entrar**: busca **entender sin tocar** cuánto puede saber cualquiera sobre la
organización, y dejarlo documentado con evidencia.

> **Regla de encuadre:** solo se investigan objetivos dentro del alcance autorizado (dominio de
> laboratorio como `example.com`, dominio propio o permiso por escrito). Este
> material es didáctico y no concede autorización por sí mismo. Si encuentras un bucket o servicio
> expuesto, **lo documentas y te detienes**.

Creado para el curso de **Hackeo Ético, Ingeniería Cibernética (ULSA)** con asistencia de Kiro.

> **Base didáctica:** este power está construido sobre el módulo **"OSINT: Corporate Recon"** de
> **Hack The Box Academy** ([academy.hackthebox.com/course/preview/osint-corporate-recon](https://academy.hackthebox.com/course/preview/osint-corporate-recon/osint-methodology)).
> Las categorías (Company Information, Infrastructure, Leaks), la metodología de ciclos de lo
> general a lo detallado y el cierre Origin → Impact → State provienen de ese módulo; aquí se
> adaptan a un flujo reproducible con herramientas MCP sin API keys, compuertas de aprendizaje
> anti copy-paste y entrega de evidencia versionable para uso académico.

## Cómo está organizado el conocimiento (steering/)

El conocimiento está repartido en guías de steering que se cargan **bajo demanda** (acción
`readSteering`): lee solo la que necesites según la fase.

| Guía | Categoría | Contenido |
|---|---|---|
| `osint-metodologia-flujo.md` | Metodología | Principios, frontera legal pasivo/activo, dos navegadores, flujo por fases con las herramientas MCP. **Léela primero.** |
| `osint-company-information.md` | Company Information | Organization, Locations, Staff, Contact Information, Business Records, Services, Social Networks. |
| `osint-infraestructura-dominio.md` | Infrastructure (dominio) | Domain Information, Public Domain Records (registro, DNS, ASN, netblocks), Domain Structure (subdominios). |
| `osint-cloud-terceros.md` | Infrastructure (cloud) | Cloud Storage (S3, Blob, GCS) y Third Parties, con la regla de detención. |
| `osint-emails-redes-tecnologias.md` | Infrastructure (email/tech) | Email Addresses, Compounded Networks, Technologies in Use. |
| `osint-leaks.md` | Leaks | Tipos de leak, Archives (Wayback, Pastebin), Internal Leaks (metadatos), Breaches. |
| `osint-intelligence-reporte.md` | Intelligence | Origin → Impact → State, estructura de evidencia y plantilla de informe. |
| `osint-herramientas-mcp.md` | (transversal) | Argumentos de cada herramienta, qué está apagado y por qué, verificación sin LLM, errores comunes. |
| `osint-compuertas-aprendizaje.md` | (didáctica) | **Compuertas anti copy-paste:** el agente para en cada fase y exige el razonamiento del alumno (Observe/Orient/Decide) antes de ejecutar. Cárgala al inicio en modo laboratorio. |

Cada guía incluye una tabla **técnica → herramienta del power**, que separa lo que cubre el MCP,
lo que hace el humano en su navegador y lo que queda fuera de alcance (activo o con llave).

## Modo laboratorio: compuertas de aprendizaje (anti *script kiddie*)

Para uso en clase, el power incluye dos piezas que **frenan el flujo y piden al alumno su
entendimiento y su decisión** antes de cada consulta, de modo que entienda el proceso en vez de
copiar y pegar:

1. **Steering `osint-compuertas-aprendizaje.md`** (cumplido por el modelo): convierte al agente en
   un tutor socrático que se detiene en cada fase, exige que el alumno exprese Observe/Orient/Decide
   con sus palabras, rechaza respuestas vacías o "solo hazlo", no se salta las compuertas aunque el
   alumno lo pida (solo el instructor), y registra cada decisión en
   `evidencia/osint/<objetivo>-<fecha>/bitacora-decisiones.md`. Cárgala al inicio de la sesión:

   > Activa el Power osint-passive-recon y carga el steering osint-compuertas-aprendizaje.md; trátalo como obligatorio durante este laboratorio.

2. **Hook `hooks/osint-gate.json`** (cumplido por el IDE): un `PreToolUse` que fuerza una
   confirmación manual antes de cada herramienta MCP, con un recordatorio del razonamiento exigido.
   Instálalo copiándolo a la carpeta de hooks del workspace del alumno:

   ```bash
   mkdir -p .kiro/hooks && cp powers/osint-passive-recon/hooks/osint-gate.json .kiro/hooks/
   ```

   Verás el hook en Kiro (sección *Agent Hooks*). Para una demo sin interrupciones, bórralo de
   `.kiro/hooks/` o desactívalo desde esa sección.

**Refuerzo recomendado:** pide a los alumnos trabajar en **modo Supervisado** (aprobación por cada
acción) y entrega la bitácora de decisiones como parte calificable, más una breve defensa oral del
informe. Las compuertas evitan el copy-paste; la evaluación del razonamiento lo confirma.

## Principios de OSINT

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

Cierre: **Origin → Impact → State**, que conecta cuándo se originó cada hallazgo, qué
impacto tiene y qué estado se observa hoy.

## Metodología cognitiva: OSINT con OODA

**OODA** es un ciclo de toma de decisiones formulado por el coronel **John Boyd** de la Fuerza Aérea
de EE. UU. a partir de su experiencia como piloto de combate. Boyd observó que, en un duelo aéreo,
no ganaba necesariamente el piloto con mejor avión, sino el que **completaba más rápido y con mejor
información** el ciclo **Observe → Orient → Decide → Act**: mirar lo que pasa, interpretarlo con su
experiencia y contexto, decidir una maniobra y ejecutarla, para luego volver a observar el resultado.

El ciclo saltó de la aviación al negocio, al litigio y a la ciberseguridad porque describe cualquier
situación en la que **se decide bajo incertidumbre con información parcial y cambiante**, que es
exactamente el escenario de OSINT: nunca ves "toda" la superficie de una organización, solo señales
indirectas (un registro DNS, un certificado, un snapshot de Wayback), y cada nueva señal cambia lo
que conviene preguntar a continuación.

Aplicado a OSINT pasivo, OODA fuerza tres disciplinas que evitan el "copy-paste de herramientas":

1. **Pensar antes de ejecutar.** Nunca se lanza una consulta "a ver qué sale"; primero se nombra qué
   se quiere reducir de incertidumbre.
2. **Separar hechos de interpretaciones.** La salida cruda es el hecho; "usan Microsoft 365" es
   interpretación. El informe los registra en columnas distintas.
3. **Cerrar el ciclo con evidencia.** Cada "Act" produce un archivo guardado en
   `evidencia/osint/`, no un recuerdo en la conversación.

| Etapa | Pregunta | Qué haces en OSINT |
|---|---|---|
| **Observe** | ¿Qué sé realmente? | Lee la salida cruda de la herramienta. Separa **hechos** (MX apunta a `outlook.com`) de **inferencias** (usan Microsoft 365). |
| **Orient** | ¿Qué significa? | Relaciona el dato con su categoría OSINT (Company / Infrastructure / Leaks) y con hallazgos previos. Formula hipótesis explícitas. |
| **Decide** | ¿Qué conviene y está autorizado? | Elige la **próxima consulta pasiva** que reduzca la incertidumbre más grande. Confirma que no toca al objetivo. |
| **Act** | ¿Qué pasó? | Ejecuta la herramienta MCP, guarda la salida cruda con fecha y argumentos en `evidencia/osint/`, y vuelve a Observe con el resultado. |

Cada ciclo es corto (una pregunta, una herramienta, una salida) y encadena al siguiente: un
subdominio nuevo descubierto en "Act" vuelve a entrar como "Observe" de otro ciclo sobre DNS y
Wayback. Las compuertas de aprendizaje de este power (steering + hook) existen justamente para que
el alumno no se salte Orient y Decide.

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
6. **La información de una fase alimenta a otra.** Encadenar es el corazón del método.

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

- **Hack The Box Academy — "OSINT: Corporate Recon"** (base conceptual del power, incluye las
  categorías Company Information / Infrastructure / Leaks y el cierre Origin → Impact → State):
  <https://academy.hackthebox.com/course/preview/osint-corporate-recon/osint-methodology>.
- Metodología **OODA** (Observe–Orient–Decide–Act) aplicada al reconocimiento pasivo.
- Las guías de steering de este power son autocontenidas: metodología, categorías y flujo están en
  `steering/osint-*.md`.
- Paquetes de los servidores MCP: `openosint` y `osint-toolbox-mcp` (PyPI).

---

**Paquetes:** `openosint==2.36.1` (PyPI), `osint-toolbox-mcp@1.1.0` (PyPI)
**Servidores MCP:** `openosint`, `osint-toolbox`
