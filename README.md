# Infraestructura de Microservicios

Este módulo contiene la infraestructura base para la arquitectura de microservicios.

---

## Componentes actuales

- Config Server (Spring Cloud Config Server)
- config-repo (configuración externa)

---

## Componentes planificados

- Eureka (Service Registry)
- API Gateway
- Circuit Breaker
- Seguridad
- Balanceador externo

---

## Arquitectura

```text
Microservicios → Config Server → config-repo
```

Evolución:

```text
Client → Gateway → Microservicios → Eureka → Config Server
```

---

## Red de infraestructura

Se utiliza una red Docker común CREADA en config-server:

```
ms-net
```

Esta red permite la comunicación entre:

- config-server
- registry-server (futuro)
- gateway (futuro)
- microservicios

---


## Estructura del módulo

```
infra/
  config-server/
  config-repo/
  docker-compose.yml
```

---

# Config Server

## Descripción

Servidor de configuración centralizada para todos los microservicios.

Permite:

- externalizar configuración
- separar código de configuración
- soportar múltiples entornos (`dev`, `prod`)
- facilitar despliegue en microservicios

---

## Configuración utilizada

Modo:

```
native
```

Ruta del repositorio:

```
/config-repo
```

Montado como volumen Docker.

---

## Levantar Config Server
DEV
```bash
mvn spring-boot:run
```
PROD
```bash
docker compose up
```
o
```bash
docker compose up -d config-server
```

---

## Acceso

### DEV

```
http://localhost:7071
```

### PROD

```
http://config-server:7072
```

---

## Prueba

```bash
curl http://localhost:7071/catalogo/dev
```

o:

```bash
curl http://localhost:7072/catalogo/prod
```

---

# config-repo

Contiene la configuración externa de los microservicios.

Ejemplo:

```
config-repo/
  catalogo-dev.yml
  catalogo-prod.yml
```

---

## Ejemplo de configuración

```yaml
spring:
  datasource:
    url: jdbc:mysql://${CATALOGO_DB_HOST}:${CATALOGO_DB_PORT}/${CATALOGO_DB_NAME}
    username: ${CATALOGO_DB_USERNAME}
    password: ${CATALOGO_DB_PASSWORD}
```

---

# Flujo de uso

1. Levantar infraestructura

```bash
docker compose up -d config-server
```

2. Levantar microservicio (ej: catalogo)

3. Microservicio obtiene configuración desde Config Server

---

# Problemas comunes

## 1. No conecta a config-server

Causa:
- red incorrecta

Solución:
- conectar microservicio a `ms-net`

---

## 2. Configuración no cargada

Causa:
- archivo no existe en config-repo

Solución:
- verificar nombre: `catalogo-dev.yml`, `catalogo-prod.yml`

---

## 3. Uso incorrecto de localhost

Dentro de Docker:

❌ `localhost`  
✔ `config-server`

---

# Estado de avance

- [x] Config Server
- [ ] Eureka
- [ ] API Gateway
- [ ] Circuit Breaker
- [ ] Seguridad
- [ ] Balanceador

---

# Siguiente paso

Implementar **Eureka (Service Registry)** para permitir:

- registro automático de microservicios
- descubrimiento dinámico
- integración con API Gateway

---

# Tag sugerido

```bash
git tag -a vs02-config-server -m "Infraestructura: Config Server operativo"
git push origin vs02-config-server
```