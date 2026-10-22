# GoMad

**Guía de Ocio: Madrid Activo y Deportivo.** Una web pensada para el móvil que, a partir de datos abiertos, te dice **dónde y cuándo** hacer deporte o salir a pasear en cualquiera de los 179 municipios de la Comunidad de Madrid, y **por qué ruta** llegar.

👉 **Pruébala en https://gomaduc3m.duckdns.org**: no hace falta registrarse.

> *«Corre en este parque a las 20:00: baja la temperatura, el aire es bueno y el 68 % del camino va por zona verde.»*

Proyecto presentado al **MAD Open Data Challenge 2026** (UC3M, Comunidad de Madrid y Oracle).

## Qué hace

- **Recomendador de actividad.** Eliges una actividad (caminar, correr, nadar, pádel, tenis, fútbol, baloncesto, gimnasio) o pulsas «Sorpréndeme», y la app te propone los mejores destinos cercanos, cada uno con una nota de 0 a 10 y la razón de esa nota.
- **Mejor momento.** Combina la previsión horaria con la calidad del aire para proponerte la mejor franja. Si llueve, hace calor o el aire es malo, te sugiere una instalación cubierta o una hora más adecuada.
- **Ruta saludable.** Compara la ruta directa con otra más verde y con menos tráfico. Indica distancia, tiempo, porcentaje de zona verde, fuentes, bancos y desfibriladores.
- **Personalización.** Puedes fijar el tiempo, la distancia, si prefieres interior o exterior y tu perfil (persona mayor, con carrito, sensible a la contaminación, con perro o con niños), y ajustar los pesos de cada factor. Los resultados se recalculan al momento.
- **Vista de municipio.** Indicadores de deporte, naturaleza, movilidad, salud, educación y conectividad de cada municipio, sin rankings.
- **Asistente de rutas.** Escribe algo como *«15 km andando, por un parque, con fuentes y bancos»* y lo convierte en parámetros que puedes editar. Te responde qué condiciones se cumplen y cuáles no.

El motor de reglas decide y la IA solo interpreta tu texto: ninguna recomendación, nota ni ruta la elige un modelo de lenguaje. Los umbrales de aire y de calor son los oficiales (ICA del MITECO y plan de calor del Ministerio de Sanidad).

> ⚠️ GoMad no da consejo médico.

## Privacidad

No hay registro ni cuentas. Tu ubicación solo se usa para responder a tu consulta y no se guarda; nunca se envía a los servicios de previsión ni al de IA.

## Arquitectura

```
app/
  ingest/    Ingesta y normalización de fuentes (Python): GeoParquet + DuckDB y grafo peatonal de OSM
  api/       API REST (FastAPI): motor de puntuación, rutas (igraph), asistente
  frontend/  SPA en React + Vite + TypeScript con MapLibre GL, servida con Caddy
  config/    Pesos, umbrales, actividades e indicadores en YAML (sin valores fijados en el código)
  deploy/    Docker Compose para el despliegue
```

## Ejecutarlo en local

Necesitas Docker (con al menos 6 GB de RAM para preparar los datos).

```bash
cd app
cp .env.example .env                       # claves opcionales (asistente con OCI Language)
cp deploy/.env.example deploy/.env         # DOMAIN=:80 en local

# 1. Descargar y preparar los datos (20–30 min la primera vez)
docker compose -f deploy/docker-compose.init.yml run --rm ingest

# 2. Levantar la app
docker compose -f deploy/docker-compose.yml up -d --build
curl -s localhost/api/health               # {"status":"ok"}
```

Después abre http://localhost. La previsión del tiempo (Open-Meteo) no necesita clave. Sin credenciales de OCI, el asistente funciona solo con su diccionario.

### Desarrollo

```bash
# API
cd app/api && python3 -m venv .venv && .venv/bin/pip install -e ".[dev]"
.venv/bin/uvicorn main:app --reload --port 8000
.venv/bin/pytest -m "not slow"

# Front (en desarrollo, /api se redirige a :8000)
cd app/frontend && npm ci && npm run dev
npm test
```

## Fuentes de datos

Todo se descarga automáticamente por API desde los portales oficiales:

| Fuente | Licencia |
|---|---|
| [Datos abiertos de la Comunidad de Madrid](https://datos.comunidad.madrid) (CKAN), [IDEM](https://idem.comunidad.madrid) y Nomecalles (Instituto de Estadística) | CC BY 4.0 |
| [Portal de datos abiertos del Ayuntamiento de Madrid](https://datos.madrid.es) (complemento para la capital) | CC BY 4.0 |
| [OpenStreetMap](https://www.openstreetmap.org/copyright), a través de Geofabrik | ODbL |
| Previsión meteorológica y aire previsto (CAMS): *Weather data by [Open-Meteo.com](https://open-meteo.com/)* | CC BY 4.0 |
| Modelo digital del terreno MDT05 del [IGN](https://www.ign.es) | CC BY 4.0 |

La app muestra la fuente y la fecha de cada dato.

## Licencia

Código bajo licencia [MIT](LICENSE). La fuente Figtree se distribuye bajo la licencia SIL OFL (`app/frontend/public/fonts/Figtree-OFL.txt`).
