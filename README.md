# Infraestructura de Microservicios

Este modulo contiene la infraestructura base de la arquitectura de microservicios.

---

## Componentes actuales

- config-repo (configuracion externa)
- Config Server (Spring Cloud Config Server)
- Registry Server (Eureka)
- API Gateway

---

## Componentes planificados

- Seguridad
- Gestion del trafico en Gateway
- Observabilidad y trazabilidad
- Integracion con frontend

---

## Arquitectura actual

```text
Client -> Gateway -> Microservicios -> Registry Server -> Config Server -> config-repo
```

Evolucion objetivo:

```text
Client -> Gateway + atributos de calidad -> Microservicios -> Registry Server -> Config Server
```

---

## Puertos utilizados

| Servicio | Puerto |
|---|---:|
| Config Server DEV | 7071 |
| Config Server PROD | 7072 |
| Registry Server DEV | 7081 |
| Registry Server PROD | 7082 |
| Gateway DEV | 7091 |
| Gateway PROD | 7092 |

---

## Red de infraestructura

Se utiliza una red Docker comun:

```text
ms-net
```

Esta red permite la comunicacion entre:

- config-server
- registry-server
- gateway
- microservicios

---

## Estructura del modulo

```text
infra/
  config-server/
  registry-server/
  gateway/
  config-repo/
  docker-compose.yml
```

---

## Config Server

Servidor de configuracion centralizada para los microservicios.

Permite:

- externalizar configuracion
- separar codigo de configuracion
- soportar multiples entornos (`dev`, `prod`)
- facilitar despliegue de microservicios

Modo utilizado:

```text
native
```

Ruta del repositorio montado:

```text
/config-repo
```

### Levantar Config Server

DEV:

```bash
cd infra/config-server
mvn spring-boot:run
```

PROD:

```bash
cd infra
docker compose up -d config-server
```

### Pruebas

DEV:

```bash
curl http://localhost:7071/catalogo/dev
```

PROD:

```bash
curl http://localhost:7072/catalogo/prod
```

---

## Registry Server

Servidor de registro y descubrimiento de servicios.

Permite:

- registro automatico de microservicios
- descubrimiento dinamico
- integracion con API Gateway mediante `lb://`

### Levantar Registry Server

DEV:

```bash
cd infra/registry-server
mvn spring-boot:run
```

PROD:

```bash
cd infra
docker compose up -d registry-server
```

### Acceso a Eureka

```text
DEV  -> http://localhost:7081
PROD -> http://localhost:7082
```

---

## config-repo

Contiene la configuracion externa de infraestructura y microservicios.

Archivos actuales:

```text
config-repo/
  catalogo-dev.yml
  catalogo-prod.yml
  gateway-dev.yml
  gateway-prod.yml
  producto-dev.yml
  producto-prod.yml
  registry-server-dev.yml
  registry-server-prod.yml
```

Ejemplo base:

```yaml
eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
```

Ejemplo actual de resiliencia externa para `producto`:

```yaml
resilience4j:
  circuitbreaker:
    instances:
      catalogo:
        slidingWindowSize: 5
        minimumNumberOfCalls: 3
        failureRateThreshold: 50
        waitDurationInOpenState: 5s
        permittedNumberOfCallsInHalfOpenState: 2
        automaticTransitionFromOpenToHalfOpenEnabled: true
```

---

## Flujo de uso

1. Levantar infraestructura base.

```bash
docker compose up -d
```

2. Verificar endpoints.

```text
http://localhost:7072/catalogo/prod
http://localhost:7082
http://localhost:7092/api/v1/catalogo/instancia
http://localhost:7092/api/v1/producto/instancia
```

3. Levantar microservicios.
4. Verificar registro en Eureka.
5. Probar enrutamiento por Gateway con `lb://catalogo`.
6. Probar enrutamiento por Gateway con `lb://producto`.

---

## Resiliencia actual

La infraestructura ya soporta configuracion externa para Circuit Breaker desde `config-repo`.

Estado actual:

- `producto` consume configuracion Resilience4j desde `producto-dev.yml` y `producto-prod.yml`
- el circuito configurado se llama `catalogo`
- protege la llamada remota de `producto` hacia `catalogo`
- `catalogo` no requirio cambios para esta fase

Separacion de responsabilidades:

- `infra` centraliza la configuracion externa
- `producto` implementa el uso del Circuit Breaker
- `catalogo` mantiene su API sin cambios

---

## Problemas comunes

### 1. Microservicio no conecta a config-server

Causa:

- red incorrecta

Solucion:

- conectar el servicio a `ms-net`

### 2. Microservicio no aparece en Eureka

Causa:

- `defaultZone` incorrecto
- `registry-server` no disponible

Solucion:

- en DEV usar `http://localhost:7081/eureka`
- en Docker usar `http://registry-server:7081/eureka`

### 3. Configuracion no cargada

Causa:

- archivo no existe en `config-repo`

Solucion:

- verificar nombres por entorno (`*-dev.yml`, `*-prod.yml`)

### 4. Uso incorrecto de localhost en Docker

Dentro de Docker:

- Incorrecto: `localhost`
- Correcto: `config-server`, `registry-server`

---

## Estado de avance

- [x] Config Server
- [x] Registry Server (Eureka)
- [x] API Gateway
- [x] Enrutamiento `lb://catalogo` y `lb://producto`
- [x] Feign
- [x] Circuit Breaker
- [ ] Seguridad
- [ ] Gestion del trafico (filtros, politicas y control de peticiones)
- [ ] Observabilidad y trazabilidad
- [ ] Integracion con frontend

---

## Siguiente paso

Continuar con los atributos de calidad sobre la base actual:

- integrar seguridad con autenticacion y autorizacion
- aplicar gestion del trafico en Gateway
- fortalecer observabilidad y trazabilidad entre servicios
- habilitar integracion con frontend

---

## Tag sugerido

```bash
git tag -a vs05-circuitbreaker-config -m "Infraestructura: configuracion externa y documentacion de Circuit Breaker para producto"
git push origin vs05-circuitbreaker-config
```
