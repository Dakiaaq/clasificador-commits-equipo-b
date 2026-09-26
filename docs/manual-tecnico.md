# Manual técnico - API Clasificador de commits

## 1. Descripción

El proyecto consiste en una API REST para clasificar mensajes de commits utilizando dos motores diferentes: un motor basado en reglas llamado `eco` y un motor de inteligencia artificial local mediante Ollama.

La API permite recibir un mensaje, determinar su categoría y guardar el resultado de la inferencia en una base de datos PostgreSQL.

Las categorías utilizadas son:

* `feat`
* `fix`
* `docs`
* `test`
* `chore`
* `refactor`

## 2. Arquitectura

La solución está organizada de la siguiente manera:

```text
Cliente
   |
   v
FastAPI
   |
   +------------------+
   |                  |
   v                  v
Motor ECO          Ollama
(reglas)        (qwen2.5:0.5b)
   |                  |
   +--------+---------+
            |
            v
       PostgreSQL
            |
            v
      Registro de
       inferencias
```

El cliente realiza una petición HTTP a la API desarrollada con FastAPI.

La API recibe el mensaje y determina qué motor utilizar. Si se selecciona `eco`, se utilizan reglas de expresiones regulares. Si se selecciona `ollama`, se envía el mensaje al modelo local `qwen2.5:0.5b`.

Después de realizar la clasificación, el resultado se almacena en PostgreSQL.

## 3. Tecnologías utilizadas

* Python
* FastAPI
* Uvicorn
* PostgreSQL 16
* Docker
* Ollama
* Modelo `qwen2.5:0.5b`
* psycopg2
* python-dotenv
* Requests
* Git y GitHub

## 4. Base de datos

La base de datos utilizada se llama `iadb`.

PostgreSQL se ejecuta mediante Docker en el contenedor:

```text
db-ia
```

La tabla utilizada para almacenar las inferencias es:

```text
inferencias
```

Esta tabla almacena información como:

* Identificador de la inferencia.
* Fecha.
* Motor utilizado.
* Modelo.
* Texto de entrada.
* Resultado de la clasificación.
* Latencia de la operación.

También se utiliza el usuario de aplicación `app_ia` para realizar las operaciones necesarias desde la API.

## 5. Variables de entorno

La configuración se realiza mediante un archivo `.env`.

Las variables utilizadas son:

```text
DB_HOST
DB_PORT
DB_NAME
DB_USER
DB_PASSWORD
DB_ADMIN_PASSWORD
OLLAMA_URL
MODELO_OLLAMA
MOTOR_POR_DEFECTO
```

El archivo `.env` no debe ser incluido en el repositorio porque contiene información de configuración y credenciales.

Para compartir la estructura de configuración se utiliza `.env.example`.

## 6. API

La aplicación principal se encuentra en:

```text
app/main.py
```

La API utiliza FastAPI y proporciona los siguientes endpoints:

### GET /health

Permite comprobar que el servicio y la conexión con PostgreSQL están disponibles.

Respuesta de prueba:

```json
{
  "estado": "ok",
  "base_datos": "ok"
}
```

### POST /clasificar

Recibe un mensaje de commit y permite seleccionar el motor de clasificación.

Ejemplo:

```json
{
  "texto": "agrega una nueva función de búsqueda",
  "motor": "ollama"
}
```

La respuesta contiene el motor utilizado, el modelo, el texto recibido, la categoría obtenida y la latencia.

### GET /inferencias

Permite consultar las inferencias almacenadas en PostgreSQL.

Se puede indicar un límite mediante el parámetro `limite`.

Ejemplo:

```text
GET /inferencias?limite=20
```

## 7. Motores de clasificación

### Motor ECO

El motor `eco` utiliza reglas y expresiones regulares para identificar palabras relacionadas con cada categoría.

Por ejemplo, palabras como:

```text
fix
corrige
error
bug
```

pueden producir la categoría:

```text
fix
```

Este motor funciona como una línea base y no necesita utilizar un modelo de lenguaje.

### Motor Ollama

El motor `ollama` utiliza un modelo de lenguaje ejecutado localmente.

El modelo utilizado en las pruebas fue:

```text
qwen2.5:0.5b
```

La API envía al modelo el mensaje de commit y solicita una única categoría.

## 8. Pruebas realizadas

Se realizaron pruebas utilizando Swagger mediante:

```text
http://localhost:8000/docs
```

### Prueba del motor ECO

Entrada:

```json
{
  "texto": "corrige el error de login",
  "motor": "eco"
}
```

Resultado:

```text
tipo: fix
```

La petición respondió con código HTTP `200`.

### Prueba del motor Ollama

Entrada:

```json
{
  "texto": "agrega una nueva función de búsqueda",
  "motor": "ollama"
}
```

Resultado:

```text
tipo: feat
```

Modelo:

```text
qwen2.5:0.5b
```

La petición respondió con código HTTP `200`.

La latencia registrada fue de aproximadamente:

```text
4952 ms
```

## 9. Persistencia de las inferencias

Después de realizar las pruebas se consultó el endpoint:

```text
GET /inferencias?limite=20
```

La API devolvió los registros almacenados en PostgreSQL.

Se verificaron dos inferencias:

```text
ID 1
Motor: eco
Modelo: reglas-v1
Salida: fix
```

y:

```text
ID 2
Motor: ollama
Modelo: qwen2.5:0.5b
Salida: feat
```

Esto permitió comprobar que las clasificaciones realizadas por la API fueron almacenadas correctamente en la base de datos.

## 10. Seguridad y buenas prácticas

El proyecto utiliza variables de entorno para evitar escribir directamente las credenciales de la base de datos dentro del código fuente.

Además, el archivo `.env` debe permanecer fuera del repositorio mediante `.gitignore`.

La conexión de la aplicación utiliza el usuario de base de datos destinado para la aplicación y no el usuario administrador de PostgreSQL.

El acceso a la base de datos se realiza mediante una conexión protegida por las credenciales configuradas en las variables de entorno.

## 11. Ejecución del proyecto

Para ejecutar la API se utiliza:

```bash
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

Después se puede acceder a la documentación interactiva de FastAPI mediante:

```text
http://localhost:8000/docs
```

Antes de utilizar el motor Ollama se debe comprobar que el modelo configurado esté instalado localmente.

El modelo utilizado actualmente es:

```text
qwen2.5:0.5b
```

## 12. Estado actual

La API se encuentra funcionando y se comprobaron:

* Conexión entre FastAPI y PostgreSQL.
* Endpoint `/health`.
* Clasificación mediante el motor `eco`.
* Clasificación mediante Ollama.
* Uso del modelo `qwen2.5:0.5b`.
* Registro de las inferencias.
* Consulta de las inferencias almacenadas.
* Ejecución mediante Docker para PostgreSQL.
