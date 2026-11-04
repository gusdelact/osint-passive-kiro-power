# OSINT: Intelligence, evidencia e informe

> Steering del power **osint-passive-recon**, HTB *Open-Source Intelligence*, sección 23
> (Intelligence), más las reglas de evidencia y la plantilla de informe del power.

## Sección 23 — Origin → Impact → State

La fase final es donde está el valor de OSINT: **descubrir y enlazar las conexiones** entre los
hallazgos.

```text
Origin  ──▶  Impact  ──▶  State
```

- **Origin:** una actividad pasada hecha para cambiar un estado futuro (registrar un dominio,
  migrar el correo, lanzar un sitio en WordPress). Cada actividad tiene un propósito.
- **Impact:** propósito + acción. Ni la meta sola ni una acción al azar producen un impacto
  dirigido.
- **State:** el resultado observable hoy. Alcanzarlo deja **side-effects**: efectos no buscados.

En seguridad, los side-effects (misconfiguraciones involuntarias al buscar una funcionalidad) son
lo que busca el pentester. El trabajo es reconstruir los pasos entre impact y state.

**Ejemplo con salidas del power:**

| Origin (fuente, fecha) | Impact | State observable hoy |
|---|---|---|
| Dominio registrado en 2014 (`search_rdap`) | Sitio montado en WordPress (rutas `wp-content/` en Wayback 2016) | Rutas de WordPress siguen en snapshots de 2025 → posible CMS vigente (hipótesis) |
| Migración de correo a Microsoft 365 (TXT `MS=`, `search_dns`) | Correo y archivos en un tercero | MX en `outlook.com`; DMARC `p=none` → correo suplantable (indicador) |
| Certificado emitido para `vpn.<dominio>` (crt.sh, 2022) | Acceso remoto publicado | `vpn.<dominio>` sigue resolviendo (`dns_lookup`) → superficie expuesta (sin verificar servicio) |

## Regla de persistencia de evidencia (obligatoria)

La **evidencia cruda se guarda** en el workspace del proyecto, nunca solo en `/tmp`. Estructura
recomendada por investigación:

```text
evidencia/osint/<objetivo>-<AAAA-MM-DD>/
├── 00-alcance.md            # objetivo, propósito, autorización, técnicas permitidas
├── herramientas/            # una salida cruda por llamada MCP
│   ├── 01-search_rdap.json
│   ├── 02-search_dns.json
│   └── ...
├── archivos/                # documentos públicos descargados + SHA256SUMS
├── navegador/               # history.csv (History Master) y páginas SingleFile
└── informe.md
```

Cada archivo de `herramientas/` registra: herramienta, argumentos, fecha y hora (UTC), y la salida
**sin editar**. Si una herramienta falla, se guarda también el error.

## Reglas cognitivas para el informe

1. **Separa** hechos observados, inferencias, hipótesis y pendientes.
2. **Toda afirmación lleva procedencia:** herramienta + argumentos (o URL + captura) + fecha.
3. **Nada sin salida real.** Si una herramienta devolvió vacío o error, se escribe así. No se
   rellena con conocimiento previo del modelo.
4. **Un indicador no es una vulnerabilidad.** SPF débil, un CMS antiguo o un bucket listado en un
   indexador son exposición a validar en una fase autorizada.
5. **Fecha de cada dato:** un snapshot de 2019 describe 2019, no hoy.

## Plantilla de informe

```markdown
# Informe OSINT pasivo — <objetivo>

## 1. Alcance y autorización
Objetivo, propósito, quién autoriza, técnicas permitidas y prohibidas, fechas.

## 2. Resumen ejecutivo
3–5 frases sin tecnicismos: qué puede saber cualquiera sobre la organización.

## 3. Hallazgos
| # | Categoría HTB | Hallazgo | Tipo (hecho/inferencia/hipótesis) | Fuente (herramienta + args) | Fecha | Impacto |
|---|---|---|---|---|---|---|

## 4. Infraestructura
Registro (RDAP/WHOIS), DNS (raíz + www), SPF/DMARC, IP → ASN → proveedor, subdominios (crt.sh +
Wayback) con estado vivo/histórico/tercero.

## 5. Company Information y Leaks
Dorks generados (no ejecutados por el agente), resultados que el humano aportó, pastes, metadatos.

## 6. Intelligence: Origin → Impact → State
Tabla que conecta los hallazgos.

## 7. Pendientes y fallos de herramientas
Lo que no se pudo obtener y por qué (p. ej. crt.sh HTTP 502, TLD sin RDAP, DKIM fuera de alcance).

## 8. Decisiones de no hacer
Acciones que se descartaron por estar fuera de alcance (AXFR, abrir un bucket, verificar un correo).

## 9. Recomendaciones
Por cada indicador: riesgo y mitigación (p. ej. DMARC `p=reject`, retirar metadatos de PDFs).

## 10. Herramientas y fuentes
Servidores MCP y versiones, herramientas usadas, fuentes de terceros consultadas.
```

## Preguntas didácticas (úsalas al revisar el informe)

- ¿Qué afirmación **observaste** y cuál solo **inferiste**?
- ¿Qué salida concreta respalda cada hallazgo?
- ¿Qué evidencia adicional confirmaría o refutaría la hipótesis, y requiere una fase activa?
- ¿Alguna consulta tocó al objetivo o a un tercero que escanea por encargo?
- ¿Cómo explicarías el riesgo a la dirección sin tecnicismos?
