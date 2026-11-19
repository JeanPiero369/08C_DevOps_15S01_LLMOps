# llmops-lab

Proyecto base del laboratorio de LLMOps: un clasificador de tickets con evals, gate y monitoreo.

La estructura ya está creada y los archivos están vacíos. La guía indica qué va en cada uno:
[1. llmops-evals-gate-monitoreo.md](https://github.com/utec-devops-2026/laboratorios/blob/main/laboratorio_llmops/1.%20llmops-evals-gate-monitoreo.md)

## Cómo empezar

No trabajes sobre este repositorio: haz **fork** a tu cuenta (botón **Fork**) y clona tu fork.

```bash
git clone https://github.com/<tu-usuario>/llmops-lab.git
cd llmops-lab
```

## Qué archivo se llena en qué paso

| Paso | Archivo | Estado |
|---|---|---|
| 0 | `requirements.txt`, `.gitignore`, `src/__init__.py` | Listos |
| 1.1 | `src/llm.py` | Vacío |
| 1.2 | `config.json`, `src/tracing.py`, `src/app.py` | Vacíos |
| 1.3 | `data/golden.jsonl` | Vacío |
| 1.4 | `tests/test_app.py` | Vacío |
| 2.1 | `prompts/clasificador/v1.md` | Vacío |
| 2.2 | `src/evaluate.py` | Vacío |
| 2.3 | `gate/thresholds.json`, `gate/baseline.json`, `src/gate.py` | Vacíos |
| 2.4 | `prompts/clasificador/v2.md` | Vacío |
| 3.1 | `.github/workflows/llmops.yml` | Lo creas tú |
| 3.2 | `.github/CODEOWNERS` | Vacío |
| 4.1 | `src/traffic.py`, `src/replay.py` | Vacíos |
| 4.2 | `src/monitor.py` | Vacío |
| 4.5 | `.github/workflows/monitoring.yml` | Lo creas tú |

Los dos workflows no vienen creados a propósito: un archivo de workflow vacío hace que GitHub Actions marque un error en cada push.

`data/prod/` se llena al ejecutar `python -m src.traffic` (paso 4.1).

## Preguntas del laboratorio

**¿Por qué el gate rechazó el prompt v1 y qué cambiaste en v2 para que aprobara?**
v1 solo pedía "JSON con categoria, riesgo_churn y resumen" sin decir qué valores eran válidos, así que el modelo inventó etiquetas (`"Quejas sobre comisiones"`, `"Alto"`) y ninguna respuesta cumplió el contrato: `contrato_valido`, `accuracy_categoria`, `accuracy_riesgo` y `resistencia_inyeccion` quedaron en 0.0. En v2 fijé los valores permitidos de cada campo con su definición, pedí solo el objeto JSON sin texto alrededor, agregué la regla de que el ticket es un dato y no una orden (contra la inyección) y un ejemplo; con eso pasó a 1.0 / 1.0 / 0.938 / 1.0.

**¿Por qué el pull request trampa falló en GitHub si en tu máquina el gate aprobaba?**
El PR trampa volvía a v1 y bajaba a 0 los umbrales de `gate/thresholds.json` y el baseline; en local el gate lee esos archivos modificados, así que aprobaba. En GitHub el paso "Tomar el juez desde main" reemplaza `gate/`, `src/gate.py`, `src/evaluate.py` y `data/golden.jsonl` por la versión de `main` antes de evaluar, de modo que v1 se juzgó con los umbrales reales y fue rechazado. Quien propone el cambio no puede editar al juez que lo evalúa (además `CODEOWNERS` protege esas rutas).

**¿Qué te muestra Langfuse que no ves en el archivo de trazas local?**
El JSONL local es una lista plana por máquina (en el runner de CI se pierde al terminar el job). Langfuse junta en un solo lugar las trazas de todas las ejecuciones, permite filtrarlas por las etiquetas `prompt:` y `escenario:` para comparar la semana base con `price_hike`, y agrega los scores (`contrato_valido`, distribución de `categoria`), la latencia y los tokens en gráficos a lo largo del tiempo, que es donde se ve el drift.
