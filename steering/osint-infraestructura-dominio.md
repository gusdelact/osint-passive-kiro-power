# OSINT: dominio, registros públicos y estructura

> Steering del power **osint-passive-recon**, HTB *Open-Source Intelligence*, secciones 11–13
> (Domain Information, Public Domain Records, Domain Structure). Es la parte del módulo mejor
> cubierta por las herramientas MCP del power.

## Sección 11 — Domain Information

De lo general a lo detallado, en tres categorías: **registros públicos**, **terceros** y
**dominios** (SLD + TLD). El código público (GitHub, GitLab) puede filtrar IPs, hostnames,
configuraciones o credenciales: `search_github(query="<dominio>")`.

> **Terceros** (hosting, CDN, cloud): aparecen en los registros, pero probar sus sistemas requiere
> permiso explícito del proveedor. Ver `osint-cloud-terceros.md`.

## Sección 12 — Public Domain Records

Cuatro componentes a obtener: **1) netblocks/CIDR, 2) ASN, 3) servidores DNS, 4) servidores de
correo**.

| Comando HTB | Equivalente MCP en este power | Qué responde |
|---|---|---|
| `whois <dominio>` / ICANN lookup | `search_rdap(domain=...)`; si el TLD no tiene RDAP, `search_whois(domain=...)` o `whois_lookup(query=...)` | Registrador, fechas, NS, Registry Domain ID |
| `host www.<dominio>` | `search_dns(domain="www.<dominio>")` o `dns_lookup(name="www.<dominio>", types=["A","AAAA"])` | IP del servidor web |
| `whois <ip>` | `whois_lookup(query="<ip>")` | Rango/CIDR, organización, OriginAS |
| ASN y proveedor | `search_ip(ip="<ip>")` (ipinfo.io) | ASN, organización, hostname, ubicación |
| `whois AS<n>` | `whois_lookup(query="AS<n>")` | Titular del sistema autónomo |
| `dig any <dominio>` | `search_dns(domain=...)` + `dns_lookup(name=..., types=["A","AAAA","MX","NS","TXT","SOA","CAA"])` | Registros publicados y alertas SPF/DMARC |
| PTR (DNS inverso) | `dns_lookup(name="<ip>")` | Nombre inverso de la IP |
| MXToolbox (open relay) | **Fuera de alcance**: probar un relay es contacto activo | — |

### Cómo leer los registros DNS

| Registro | Qué revela |
|---|---|
| **A / AAAA** | IP del (sub)dominio. Pasarla por `search_ip` y `whois_lookup` para el netblock. |
| **MX** | Servidores de correo. Si es Google o Microsoft, el correo está en un tercero. |
| **NS** | Proveedor DNS/hosting. |
| **TXT** | Claves de verificación de terceros (Atlassian, Google, MS=, Mailgun...) y SPF/DMARC. **Muy valioso.** |
| **SOA** | Contacto administrativo (el primer `.` del rname es `@`) y, a veces, otros dominios relacionados. |
| **CAA** | Qué autoridades de certificación están autorizadas. |

**Pistas de infraestructura en TXT:** Atlassian (Confluence/Jira), Google (Workspace, posibles
GDrive compartidos), Microsoft `MS=` (Office 365, OneDrive, Azure storage), Mailgun (APIs de
correo), LogMeIn (acceso remoto). En SPF aparecen a veces **IPs internas** (`ip4:10.x.x.x`):
anótalas.

**SPF / DMARC.** `search_dns` señala SPF ausente o permisivo (`+all`, `?all`) y DMARC ausente o con
`p=none`. Ambos son **indicadores de exposición** (suplantación de correo), no vulnerabilidades
confirmadas. DKIM no se revisa: sondear selectores es activo.

## Sección 13 — Domain Structure

Técnicas de descubrimiento de subdominios. **Solo las cuatro primeras son pasivas**:

| # | Técnica | ¿Pasiva? | En este power |
|---|---|---|---|
| 1 | Certificate Transparency | ✅ | `crtsh_certificate_search(domain=..., include_expired=true)` |
| 2 | Forward DNS datasets (Rapid7 Sonar) | ✅ | Fuera del power (dataset de >30 GB) |
| 3 | Web search | ✅ | `generate_dorks` + dork propuesto `site:*.<dominio> -site:www.<dominio>` |
| 4 | Disclosure / leak | ✅ | `wayback_snapshots(url=..., match="domain")`, `search_paste` |
| 5 | Subdomain brute-forcing | ❌ | **Prohibido** |
| 6 | Zone transfer (AXFR) | ❌ | **Prohibido** (`dnsrecon` apagado) |

### Ciclo de subdominios

```text
crtsh_certificate_search(domain)  ─┐
wayback_snapshots(match="domain") ─┼─▶ lista de nombres únicos
                                   │
                                   ▼
        para cada nombre: search_dns / dns_lookup ─▶ IP
                                   │
                                   ▼
        solo IPs del propio objetivo: search_ip + whois_lookup ─▶ ASN / netblock / proveedor
```

- **Filtrar terceros:** un subdominio que resuelve a un CDN o SaaS no es infraestructura propia.
- **vHosts:** dos o más subdominios en la misma IP sugieren hosting virtual.
- **Nombres que ya no resuelven** (solo en Wayback o en certificados viejos) son huella histórica:
  documéntalos como tal, no como activos vivos.
- Shodan, VirusTotal, Censys, SecurityTrails y C99 aparecen en el módulo, pero en este power están
  apagados o fuera de alcance porque piden llave.

## Documentar

Tabla de infraestructura: nombre → IP → ASN/organización → proveedor → fuente (herramienta +
argumentos) → estado (vivo / histórico / tercero). Guardar cada salida cruda en
`evidencia/osint/` (ver `osint-intelligence-reporte.md`).

## Preguntas de laboratorio (HTB, sin respuestas)

- ¿Cuántos nameservers tiene el dominio?, FQDN del servidor de correo, Registry Domain ID,
  registrador y contenido del TXT (sección 12).
- Versión de Apache, proveedor de hosting, ASN, SO del servidor web y número de recursos JS
  (sección 13). Algunas requieren fuentes con llave (Shodan) o la web en vivo: indica cuáles no
  puedes responder con las herramientas del power.
