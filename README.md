# Lab 3 Estructura de un backend

Código base del curso para leer, ejecutar y modificar a mano, sin IA.
Hardware de referencia: NodeMCU ESP8266, DHT11 y LED RGB de cátodo común.
Este proyecto es una propuesta de material base para acompañar la guía.

## Backend en Windows
Desde backend, en una terminal Command Prompt de VS Code:
```
py -m venv .venv
.venv\Scripts\python.exe -m pip install -r requirements.txt
.venv\Scripts\python.exe -m uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```
Swagger: http://127.0.0.1:8000/docs
En macOS/Linux: python3 -m venv .venv y usar .venv/bin/python.
Los cinco endpoints base están en app/routes.py. La lista guarda como máximo
100 lecturas en memoria y se pierde al reiniciar. Ejecutar un solo proceso.
POST /lecturas retorna HTTP 201 con {"lectura": {...}, "led": "apagado"}.
PUT /led acepta rojo, verde, azul o apagado. No hay autenticación:
utilizar únicamente en la red de laboratorio autorizada.

## Firmware
Abrir firmware con PlatformIO IDE en VS Code. Copiar include/config.example.h
a include/config.h y completar WiFi, URL con IPv4 del PC e identificador.
Build, Upload y Monitor a 115200 baudios. El ESP envía cada 5 segundos y aplica
el color devuelto por POST /lecturas; PUT /led se refleja en el próximo envío.
Conexiones y pruebas: ver guía. Las dependencias se descargan la primera vez.
No publicar config.h, contraseñas ni tokens.

## Reto
Implementar GET /lecturas/promedio?n=3 a mano; no se incluye la solución.
Devolver cantidad, temperatura_promedio y humedad_promedio (dos decimales).
Validar n de 1 a 100; sin lecturas responder 404; si hay menos de n, usar todas.

## Antes de la sesión
El docente debe probar el firmware en el hardware real y la red del aula,
precargar las herramientas y validar las dependencias en los equipos.
La guía destina 105 minutos a trabajar con el entorno ya preparado.


## Nombre
Doménica Fajardo

## Objetivo
●	Identificar la entrada de la aplicación, las rutas, los modelos, la lógica y el almacenamiento.
●	Ejecutar la API localmente y probar sus cinco endpoints desde Swagger.
●	Conectar un ESP32 simulado en Wokwi con el backend y comprobar el cambio del LED.
●	Agregar un endpoint y publicar su trabajo en su repositorio de GitHub.

## Ejecución de la API
Para ejecutar la API localmente en Windows, abre la terminal dentro de la carpeta `backend` y ejecuta:
1. Instalación de dependencias: `.venv\Scripts\python.exe -m pip install -r requirements.txt`
2. Inicio del servidor: `.venv\Scripts\python.exe -m uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload`
## Preguntas

Pregunta 1: ¿Qué función recibe el JSON y cuál lo guarda? La función post_lectura, localizada en el archivo routes.py, es la encargada de capturar el JSON de la solicitud entrante. Posteriormente, esta invoca a guardar_lectura (ubicada en services.py), que es la responsable de almacenar dicha información.   
Pregunta 2: ¿Qué archivo cambiarías para añadir una URL? Se debe editar el archivo routes.py, puesto que es allí donde se vinculan las rutas y los métodos HTTP con sus funciones correspondientes.   
Pregunta 3: ¿Qué ocurre con las lecturas y el color cuando se reinicia la API? ¿Por qué? Al reiniciar el servidor, se elimina el registro de las lecturas almacenadas y el LED vuelve a su condición inicial de "apagado". Esto sucede porque los datos residen en la memoria RAM del proceso (específicamente en las variables de store.py), por lo que su estado es temporal y se pierde al detener o reiniciar la aplicación
Pregunta 4: ¿Por qué una petición con JSON bien escrito puede devolver 422? Indica qué archivo define esa validación. Un error 422 se produce cuando el JSON tiene una estructura válida, pero sus valores violan las restricciones de negocio establecidas (por ejemplo, enviar un valor de humedad del 150% cuando el límite máximo permitido es menor). Estas reglas y límites se configuran en el archivo de esquemas del backend.   
Pregunta 5: Explica el recorrido completo de una lectura: ESP32 simulado, túnel de VS Code, ruta, modelo, servicio, almacenamiento y respuesta. ¿En qué momento cambia el LED? El flujo inicia cuando el ESP32 en Wokwi recolecta los datos del sensor y los transmite en formato JSON a través de una petición POST. Esta solicitud viaja por el túnel de VS Code para alcanzar el puerto 8000 de la computadora local. Al ingresar al backend, el archivo routes.py intercepta la petición, y FastAPI utiliza schemas.py para verificar que los datos cumplan con los formatos y rangos correctos. Tras una validación exitosa, services.py toma los datos y los guarda en la memoria volátil configurada en store.py. Finalmente, se envía una respuesta HTTP 201 al cliente que incluye los datos guardados y el color actual. El cambio físico en el LED ocurre al cierre de este ciclo, cuando el ESP32 lee la respuesta del servidor y enciende los pines correspondientes según el color indicado.

## Resultados de las pruebas
| Operación | Método | Ruta | Código Esperado | Código Obtenido |
| :--- | :--- | :--- | :--- | :--- |
| Revisión de salud | GET | `/salud` | 200 | 200 |
| Lecturas iniciales | GET | `/lecturas` | 200 | 200 |
| Estado inicial LED | GET | `/led` | 200 | 200 |
| Cambiar LED a verde | PUT | `/led` | 200 | 200 |
| Enviar lectura válida | POST | `/lecturas` | 201 | 201 |
| Verificar nueva lectura | GET | `/lecturas` | 200 | 200 |
| Verificar LED verde | GET | `/led` | 200 | 200 |
| Error: Humedad en 150 | POST | `/lecturas` | 422 | 422 |
| Error: Color rosa | PUT | `/led` | 422 | 422 |

## Enlace
https://wokwi.com/projects/479002620115663873