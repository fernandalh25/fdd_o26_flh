# Bitácora — uv dentro de Docker

## Quién soy

- Usuario de GitHub: fernandalh25
- Usuario de Docker Hub: fersy25

## El paquete que agregaste

Paquete: Faker

Para qué lo usa tu fila: genera un nombre falso aleatorio

## Salida en tu máquina

La salida completa del reporte corrido con uv en tu máquina.

```text
                                     Mi ambiente                                      
┏━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ Qué              ┃ Valor                                                            ┃
┡━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┩
│ Python           │ 3.14.2                                                           │
│ Intérprete       │ /Users/fernandaleonhernandez/Desktop/acceso_rapido/7_sem/fdd/fdd │
│                  │ _o26_flh/estudiantes/fernandalh25/09_python/uv_docker/.venv/bin/ │
│                  │ python                                                           │
│ sys.prefix       │ /Users/fernandaleonhernandez/Desktop/acceso_rapido/7_sem/fdd/fdd │
│                  │ _o26_flh/estudiantes/fernandalh25/09_python/uv_docker/.venv      │
│ ¿En un ambiente? │ sí                                                               │
│ Sistema          │ Darwin arm64                                                     │
│ Nombre falso     │ James Kirby                                                      │
└──────────────────┴──────────────────────────────────────────────────────────────────┘
    Paquetes instalados     
┏━━━━━━━━━━━━━━━━┳━━━━━━━━━┓
┃ Paquete        ┃ Versión ┃
┡━━━━━━━━━━━━━━━━╇━━━━━━━━━┩
│ Faker          │ 40.43.0 │
│ Pygments       │ 2.21.0  │
│ markdown-it-py │ 4.2.0   │
│ mdurl          │ 0.1.2   │
│ rich           │ 15.0.0  │
└────────────────┴─────────┘
```

## Salida en el contenedor

La salida completa del reporte corrido desde tu imagen.

```text
  Mi ambiente                 
┏━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━┓
┃ Qué              ┃ Valor                 ┃
┡━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━┩
│ Python           │ 3.13.16               │
│ Intérprete       │ /app/.venv/bin/python │
│ sys.prefix       │ /app/.venv            │
│ ¿En un ambiente? │ sí                    │
│ Sistema          │ Linux aarch64         │
│ Nombre falso     │ Tonya Watkins         │
└──────────────────┴───────────────────────┘
    Paquetes instalados     
┏━━━━━━━━━━━━━━━━┳━━━━━━━━━┓
┃ Paquete        ┃ Versión ┃
┡━━━━━━━━━━━━━━━━╇━━━━━━━━━┩
│ Faker          │ 40.43.0 │
│ Pygments       │ 2.21.0  │
│ markdown-it-py │ 4.2.0   │
│ mdurl          │ 0.1.2   │
│ rich           │ 15.0.0  │
└────────────────┴─────────┘
```

## Qué cambió y qué no

Tres líneas, con los valores de arriba: qué salió igual en las dos, qué salió
distinto, y por qué.

Cambió la versión de python, el intérprete, sys.prefix porque una corrió en mi local de donde toma la versión de python 3.14.2, mientras que la del contenedor utiliza 3.13.16 indicado en el Dockerfile
También cambió el sistema y el Fake Name por la diferencia en la arquitectura (una de mi máquina y otra de la imagen que viene de linux) y el nombre porque se generó cada vez que se ejecuta

Salió igual: Que estoy en un ambiente y todos los paquetes con sus versiones correspondientes porque es justo lo que asegura un venv, que se trabaje sobre las mismas versiones exactas

## Tu imagen en Docker Hub

URL pública: https://hub.docker.com/r/fersy25/reporte

Digest: sha256:6f1d99ea2acf9f401ee8ee4599359862eec77f925e30ea1a701e14201d93ddf5 

Comando para correrla: docker run fersy25/reporte:latest

## Prueba de que se baja del registro

La salida completa, en este orden, de cerrar sesión en el registro, borrar
tu imagen local con la bandera de forzar, y correrla otra vez.

```text
Unable to find image 'fersy25/reporte:latest' locally
latest: Pulling from fersy25/reporte
dc0ee3016bdb: Download complete 
Digest: sha256:6f1d99ea2acf9f401ee8ee4599359862eec77f925e30ea1a701e14201d93ddf5
Status: Downloaded newer image for fersy25/reporte:latest
                Mi ambiente                 
┏━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━┓
┃ Qué              ┃ Valor                 ┃
┡━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━┩
│ Python           │ 3.13.16               │
│ Intérprete       │ /app/.venv/bin/python │
│ sys.prefix       │ /app/.venv            │
│ ¿En un ambiente? │ sí                    │
│ Sistema          │ Linux aarch64         │
│ Nombre falso     │ Alex Ramirez          │
└──────────────────┴───────────────────────┘
    Paquetes instalados     
┏━━━━━━━━━━━━━━━━┳━━━━━━━━━┓
┃ Paquete        ┃ Versión ┃
┡━━━━━━━━━━━━━━━━╇━━━━━━━━━┩
│ Faker          │ 40.43.0 │
│ Pygments       │ 2.21.0  │
│ markdown-it-py │ 4.2.0   │
│ mdurl          │ 0.1.2   │
│ rich           │ 15.0.0  │
└────────────────┴─────────┘

```
