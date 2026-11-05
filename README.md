# Pipeline de contenido financiero con IA

Entrega final del curso de automatización con IA: **Ecosistema de Automatización IA Autónomo para Negocios**.

Sistema que convierte una idea cargada en Notion en un post educativo de finanzas personales para LinkedIn, Instagram, X o Facebook. El texto lo redacta Claude con una base de conocimiento propia (RAG), pasa por aprobación humana en Slack y queda registrado para medir el sistema.

## Stack

| Categoría | Herramienta | Rol |
| --- | --- | --- |
| Orquestador | n8n | Flujo principal (43 nodos) |
| Base de datos | Notion | Centro de comando, base de conocimiento, directrices y registro de ejecuciones |
| Procesamiento IA | Claude Sonnet 5.5 (Anthropic) | AI Agent con herramienta de consulta a la base de conocimiento |
| Canal de salida | Slack | Aprobación humana (HITL) y logs por hilo |

## Cómo funciona

1. **Disparo:** el Notion Trigger detecta una idea nueva en Centro de comando; el filtro `¿Idea nueva?` deja pasar solo las que están en estado `Pendiente`.
2. **Preparación:** el flujo lee las directrices de la red elegida (tono, largo, imagen, tipo de aviso legal) y calcula el rango de caracteres disponible.
3. **Redacción con IA:** el AI Agent consulta la base de conocimiento por etiqueta y devuelve un JSON de 6 campos, validado por un Structured Output Parser.
4. **Validación:** el aviso legal se agrega por código, fuera de la IA, y se controla el largo contra el rango y el tope de la red.
5. **Aprobación humana:** el borrador llega a Slack con sus fuentes y el flujo espera Aprobar o Rechazar (máximo 24 h).
6. **Publicación:** un Switch enruta a la rama de cada red (publicación simulada) y el resultado se registra en Notion.

Cualquier fallo sigue una de las 7 rutas de error (E1 a E7): aviso en el hilo de Slack, fila en el Registro de ejecuciones y estado `Error` en la idea.

![Diagrama de arquitectura](Diagrama_Arquitectura.png)

## Contenido del repositorio

| Archivo | Descripción |
| --- | --- |
| `Entrega_Final.json` | Flujo de n8n exportado, listo para importar |
| `Diagrama_Arquitectura.pdf` / `.png` | Diagrama de arquitectura del sistema |
| `Logica_del_Flujo_JSON.docx` | JSON del flujo como texto, con índice de nodos, prompt del agente y código de los nodos Code |
| `Tema_Finanzas_Personales_DEFINITIVA.csv` | Base de conocimiento (RAG) para importar en Notion |
| `Directrices_Artículos_ACTUALIZADO.csv` | Directrices por red social para importar en Notion |
| `Matriz_de_Costos.xlsx` | Matriz de costos, comparativa de modelos y ROI |

> Ajustá esta tabla si cambiás nombres o agregás carpetas (por ejemplo, `capturas/`).

## Cómo importarlo

1. En n8n: **Workflows → Import from File** y elegí `Entrega_Final.json`.
2. Creá tres credenciales (el JSON solo las referencia por nombre, no incluye claves):
   - **Anthropic account**: API key de Anthropic.
   - **Notion account**: OAuth2, con acceso a las cuatro bases.
   - **Slack account**: OAuth2, con acceso a los canales del flujo.
3. En Notion, creá las cuatro bases (los CSV sirven para la base de conocimiento y las directrices) y volvé a seleccionarlas en los nodos de Notion:

| Base | Campos principales |
| --- | --- |
| Centro de comando (Ideas) | Idea semilla, Etiquetas, Red Social, URL de imagen, Estado, Borrador generado, Aprobado, Última ejecución |
| Tema Finanzas Personales | Registro, ID, Etiqueta, Nivel, Detalle, Ejemplo práctico, Advertencias, Fuente sugerida, Verificar actualidad |
| Directrices Artículos | Red Social, Concepto, Regla, Largo min, Largo max, Tope total |
| Registro de ejecuciones | Ejecución, Resultado, Detalle, Fecha, Post (relación con Centro de comando) |

4. En Slack, creá los canales `#aprobar-entrega-final`, `#log-errores` y `#log-rechazados`, y seleccionalos en los nodos de Slack.
5. Cargá una idea con Estado `Pendiente` y activá el flujo.

## Controles principales

- **Sin bucles:** el disparador solo detecta páginas nuevas, `¿Idea nueva?` filtra por `Pendiente` y el agente tiene un tope de 5 iteraciones.
- **Costos acotados:** Max Tokens en 4.000 y 3 reintentos ante fallas de la API. Costo medido: unos USD 0,046 por posteo.
- **Contenido controlado:** la IA solo afirma datos de la base de conocimiento y declara lo que no está respaldado; el aviso legal lo agrega el flujo.
- **Minimización de datos:** el agente solo lee la base de conocimiento; no recibe IDs, URLs de Notion ni datos de personas.
- **Nada se publica sin aprobación humana.**

## Resultados del período de pruebas

13 ejecuciones (1 y 2 de noviembre de 2026): 5 publicadas, 1 rechazada, 1 vencida y 6 fallas inyectadas a propósito para validar las rutas de error.

| KPI | Valor |
| --- | --- |
| Volumen de salida | 5 posteos |
| Tasa de aprobación humana | 83,3 % (5 de 6) |
| Tasa de error (fallas forzadas) | 46,2 % (6 de 13) |
| Errores interceptados y registrados | 100 % (6 de 6) |
| Tiempo de procesamiento automático | ~40 s, sin contar la espera de la aprobación |

## Enlaces

- [Video demo](https://drive.google.com/file/d/1ynEIyQv8EJgBYK3NuO3MmZsAavVi8ESy/view?usp=sharing)
- [Dashboard de control (Notion)](https://app.notion.com/p/403b3a58a9048054b1fffc1868f081d6?v=40fb3a58a90480b8841c000cfc601c5d)
- [Carpeta completa de la entrega (Google Drive)](https://drive.google.com/drive/folders/188BuAdBBToAoRt20oGG96WLCMRxcOMyw?usp=sharing)

## Autor

Jony · Entrega final del curso de automatización con IA, 2026.
