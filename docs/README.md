# Startup Hunter System

El sistema transforma una tesis de inversión en una lista priorizada de startups con un diagnóstico basado en fuentes y criterios versionados.

## Problema y solución

La investigación inicial suele repartir criterios y fuentes entre documentos y herramientas distintas.

El pipeline reúne la búsqueda, el análisis por empresa y la evaluación en un flujo reproducible.

Cada conclusión se contrasta con marcos de evaluación versionados y conserva las evidencias disponibles.

## Proceso

1. La tesis se convierte en consultas para varias fuentes públicas.
2. Los resultados se normalizan, deduplican y filtran.
3. Cada startup pasa por una fase de investigación y tres análisis paralelos.
4. Un sintetizador reúne los análisis y los contrasta con los marcos cargados como contexto.
5. El agente de informe ordena los resultados y ejecuta las evaluaciones de calidad.

## Diseño técnico

La aplicación usa Python y Google ADK con Vertex AI para coordinar agentes y modelos.

El estado tipado separa el descubrimiento, la investigación, el análisis, la síntesis y el informe.

Los marcos pequeños y estáticos se leen desde `knowledge/`; el sistema no necesita una base vectorial.

Los fallos de una startup quedan aislados para que no descarten el trabajo ya terminado para las demás.

Los límites de candidatos, tiempos de espera y cuotas acotan el coste de las fuentes y los modelos.

## Desarrollo

Se necesitan Python 3.11 a 3.13, uv y credenciales ADC de Google Cloud para las llamadas reales.

```powershell
uv sync --all-groups
uv run python -m pytest -q
uv run python -m google.adk.cli web agents
```

Las pruebas unitarias no requieren credenciales ni llamadas externas.

Las evaluaciones en vivo usan Vertex AI y los proveedores de búsqueda configurados.

## Despliegue y límites

El script `scripts/deploy.ps1` despliega Cloud Run con acceso autenticado y guarda la clave de Firecrawl en Secret Manager.

El despliegue es una demo de usuario único y no ofrece aislamiento de sesiones entre clientes.

No se debe conceder acceso de invocación a usuarios que no deban compartir el estado de la aplicación.

Las fuentes y la búsqueda varían con el tiempo, por lo que los resultados en vivo no son deterministas.
