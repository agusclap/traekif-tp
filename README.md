# TP Traefik - Reverse Proxy / API Gateway + Load Balancing

## Integrantes
- Maximo Agustin Rodeyro
- Bautista Macedo Rodriguez
- Bruno Garibaldi

## Descripción
La arquitectura base de este trabajo práctico implementa una arquitectura simple basada en microservicios utilizando Traefik como reverse proxy y API Gateway.

Se utilizan:
- Traefik como punto único de entrada
- Servicio API (`whoami`) con 3 instancias
- Servicio static (`nginx`)
- Ruteo por path
- Middleware StripPrefix
- Load balancing real entre instancias

## Observaciones
- Solo Traefik expone puertos al exterior
- Los servicios se encuentran en la misma red interna de Docker
- El balanceo de carga se realiza automáticamente entre instancias
- Se utiliza la imagen traefik/whoami para simplificar la implementación del servicio API

## Arquitectura
- Traefik expone:
  - puerto 80 (HTTP)
  - puerto 8080 (dashboard)
- Los servicios no exponen puertos al host
- Todo el tráfico pasa por Traefik

## Requisitos
- Docker
- Docker Compose v2

## Instalación y puesta en marcha arquitectura base.
Deberá clonar este repositorio y ejecutar (para version 2 de compose):
```bash
cd traekif-tp
```

Aclaración: Notará que el archivo dynamic.yml esta vacio, esto es porque para esta implementación no será utilizada (revisar sección - Configuración dinámica (File Provider))
En static-content podrá ver un index.html que es el que se servirá con NGINX.

## Levantar el entorno
```bash
docker compose up -d
```

## Verificar contenedores
```bash
docker compose ps
```

## Verificar contenedores formateados
```bash
docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Status}}\t{{.Ports}}\t{{.Image}}"
```


## Dashboard de Traefik
Abrir en navegador:
```bash
http://localhost:8080/dashboard/
```

## Probar servicio API
```bash
curl -i http://localhost/api/whoami
```

## Ver logs y access logs
```bash
docker logs -f traefik 
```

## Probar servicio static
```bash
curl -i http://localhost/static/
```

## Probar balanceo de carga
```bash
for i in {1..10}; do curl -s http://localhost/api/whoami | grep "Hostname"; done
```
Se espera observar diferentes hostnames en cada request, lo que demuestra el balanceo de carga entre múltiples instancias.

## Middleware utilizado
Se implementa el middleware StripPrefix:

- /api/whoami → /whoami
- /static/ → /

Esto permite que los servicios internos reciban rutas correctas sin el prefijo.

## Configuración dinámica (File Provider)

Además de la configuración mediante labels (Docker provider), se utiliza un archivo `dynamic.yml` como configuración dinámica adicional.
Para ir probando las distintas implementaciones y ejemplos:

```bash
cd "7-Problemas-Soluciones-Extras"
```

Allí se encuentran 3 carpetas. Dentro de cada una de ellas se hayan el docker-compose.yml correspondiente para la implementación y el dynamic.yml.
Para probar su funcionamiento simplemente debe copiar el contenido de esos archivos en los archivos principales de /traefik-tp/docker-compose.yml y /traefik-tp/docker-compose.yml corriendo nuevamente el compose.

### 7.1 Autodescubrimiento y File Provider: configuración dinámica sin reinicio

Prueba a file.
```bash
curl http://localhost/file
```
### 7.2 HealthChecks

Ver contenedores activos:

```bash
docker ps
```

Pausar un contenedor: 
```bash
docker pause <ID_DEL_CONTENEDOR>
```
¿Qué pasa técnicamente? Docker utiliza una característica del kernel de Linux llamada cgroups freezer. Básicamente, le dice al procesador: "No le des ni un solo ciclo de reloj a este proceso".

Memoria RAM: El contenedor sigue ocupando memoria RAM. Todo el estado de la aplicación (variables, conexiones abiertas, datos temporales) permanece intacto en la memoria.

Despausar un contenedor: 
```bash
docker unpause <ID_DEL_CONTENEDOR>
```

Comprobar funcionamiento:
```bash
for i in {1..10}; do curl -s http://localhost/api/whoami | grep "Hostname"; done
```

### 7.3 Enrutamiento a distintos host con balanceo de carga por prioridad

En la sección 7.3 encontrará una carpeta llamada "balanceador de carga con pesos". Allí se encontrará el dynamic.yml configurado para que funcione el balanceo con wrr por pesos. 
El otro archivo dynamic.yml corresponde a la primer implementación sin wrr con pesos con ruteo a distintos host.

Prueba a externo:
```bash
curl http://localhost/externo/get
```

Prueba balanceo de carga con pesos:
```bash
for i in {1..10}; do curl -s http://localhost/externo/get. | grep "Hostname"; done
```
De cada 4 peticiones, 3 deberían responder con el JSON de httpbin y 1 debería dar bad gateway (porque pusimos una url random). Eso justamente demuestra que el tráfico se está repartiendo entre dos hosts distintos.



## Evidencias
Se incluyen en la carpeta `evidencias/`:

- Estado de contenedores (docker compose ps)
- Puertos expuestos (docker ps)
- Prueba de API
- Prueba de static
- Prueba de balanceo de carga
- Acceso al dashboard
- Prueba de ruta externa configurada con File Provider



