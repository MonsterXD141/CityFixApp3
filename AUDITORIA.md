## Información General

- **Proyecto auditado:** CityFixApp3
- **Repositorio:** https://github.com/MonsterXD141/CityFixApp3
- **Auditor:** Nicolas Martinez
- **Fecha:** 2026-06-02
- **Resultado:** ✅ TODAS LAS PRUEBAS PASARON

---

## Resultado de las Pruebas

---

## Arquitectura de Infraestructura

### Servicio definido en `docker-compose.yml`

El proyecto define un único servicio llamado `app` con las siguientes
características:

- **Imagen:** Construida localmente desde el `Dockerfile` del proyecto (`build: .`)
- **Imagen base:** `node:20-alpine` — versión ligera de Node.js 20 sobre Alpine Linux,
  lo que reduce el tamaño de la imagen considerablemente
- **Comando:** `npm test` — el contenedor tiene un único propósito: ejecutar las pruebas

### Dockerfile (proceso de build)

El build sigue 5 capas bien definidas:

| Capa | Instrucción | Propósito |
|---|---|---|
| 1 | `FROM node:20-alpine` | Imagen base ligera |
| 2 | `WORKDIR /app` | Define el directorio de trabajo |
| 3 | `COPY package.json .` | Copia solo el manifiesto primero |
| 4 | `RUN npm install` | Instala dependencias en una capa separada |
| 5 | `COPY . .` | Copia el resto del código fuente |

### Volúmenes

| Volumen | Propósito |
|---|---|
| `.:/app` | Monta el código fuente local dentro del contenedor |
| `/app/node_modules` | Volumen anónimo que protege los `node_modules` instalados en el contenedor, evitando que sean sobreescritos por el montaje del host |

### Red y DNS

Se configuraron servidores DNS externos explícitos:
- `8.8.8.8` (Google DNS primario)
- `8.8.4.4` (Google DNS secundario)

Esto garantiza que el contenedor pueda resolver los dominios de Supabase
correctamente sin depender del DNS del host, lo cual es crítico para las
pruebas E2E que hacen llamadas reales a la API.

---

## Pruebas E2E ejecutadas

Las pruebas verificaron tres aspectos de la conexión a Supabase:

1. **La conexión devuelve código exitoso y un array** — valida que la API
   responde con HTTP 200 y el body es un arreglo
2. **El array contiene datos reales (longitud mayor a cero)** — confirma que
   hay registros reales en la base de datos
3. **El primer reporte tiene las propiedades correctas** — verifica la
   estructura del objeto retornado

---

## Por qué la arquitectura es estable

1. **Separación del `COPY package.json` del `COPY . .`** en el Dockerfile
   aprovecha el sistema de caché de capas de Docker. Si el código cambia
   pero no las dependencias, Docker reutiliza la capa de `npm install` sin
   reinstalar todo, haciendo los builds mucho más rápidos.

2. **El volumen `/app/node_modules`** evita el error clásico donde el
   `node_modules` del host (compilado para Windows) sobreescribe el del
   contenedor (compilado para Linux), causando fallos de módulos nativos.

3. **El DNS explícito con Google (8.8.8.8 / 8.8.4.4)** garantiza resolución
   de nombres confiable dentro del contenedor, independiente de la
   configuración de red del host, lo cual es esencial para conexiones
   externas como Supabase.

4. **La imagen `node:20-alpine`** reduce la superficie de ataque y el
   tamaño de la imagen, siguiendo buenas prácticas de contenedores en
   producción.