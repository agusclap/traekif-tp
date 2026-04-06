# TP Traefik - Reverse Proxy / API Gateway + Load Balancing

## Integrantes
- Maximo Agustin Rodeyro
- Bautista Macedo Rodriguez
- Bruno Garibaldi

## Descripción
Este trabajo práctico implementa una arquitectura simple basada en microservicios utilizando Traefik como reverse proxy y API Gateway.

Se utilizan:
- Traefik como punto único de entrada
- Servicio API (`whoami`) con 3 instancias
- Servicio static (`nginx`)
- Ruteo por path
- Middleware StripPrefix
- Load balancing real entre instancias

## Arquitectura
- Traefik expone:
  - puerto 80 (HTTP)
  - puerto 8080 (dashboard)
- Los servicios no exponen puertos al host
- Todo el tráfico pasa por Traefik

## Requisitos
- Docker
- Docker Compose

## Levantar el entorno
```bash
docker compose up -d
```

## Verificar contenedores
```bash
docker compose ps
```

## Probar servicio API
```bash
curl -i http://soagmr.mooo.com/api/whoami
```

## Probar servicio static
```bash
curl -i soagmr.mooo.com/static/
```

## Probar balanceo de carga
```bash
for i in {1..10}; do curl -s soagmr.mooo.com/api/whoami | grep "Hostname"; done
```
Se espera observar diferentes hostnames en cada request, lo que demuestra el balanceo de carga entre múltiples instancias.

## Dashboard de Traefik
Abrir en navegador:
```bash
http://soagmr.mooo.com:8080/dashboard/
```
## Middleware utilizado
Se implementa el middleware StripPrefix:

- /api/whoami → /whoami
- /static/ → /

Esto permite que los servicios internos reciban rutas correctas sin el prefijo.

## Evidencias
Se incluyen en la carpeta evidencias/:

- Estado de contenedores (docker compose ps)
- Puertos expuestos (docker ps)
- Prueba de API
- Prueba de static
- Prueba de balanceo de carga
- Acceso al dashboard

## Observaciones
- Solo Traefik expone puertos al exterior
- Los servicios se encuentran en una red interna de Docker
- El balanceo de carga se realiza automáticamente entre instancias
- Se utiliza la imagen traefik/whoami para simplificar la implementación del servicio API

