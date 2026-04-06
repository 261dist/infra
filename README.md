# Infraestructura de Microservicios

Este modulo contiene la infraestructura base de la arquitectura de microservicios.

---

## Componentes actuales

- Config Server (Spring Cloud Config Server)
- Registry Server (Eureka)
- config-repo (configuracion externa)

---

## Componentes planificados

- API Gateway
- Circuit Breaker
- Seguridad
- Balanceador externo

---

## Arquitectura (estado actual)

```text
Microservicios -> Registry Server -> Config Server -> config-repo
```

Evolucion objetivo:

```text
Client -> Gateway -> Microservicios -> Registry Server -> Config Server
```

---

## Red de infraestructura

Se utiliza una red Docker comun:

```text
ms-net
```

Esta red permite la comunicacion entre:

- config-server
- registry-server
- gateway (futuro)
- microservicios

---

## Estructura del modulo

```text
infra/
  config-server/
  registry-server/
  config-repo/
  docker-compose.yml
```

---

# Config Server

## Descripcion

Servidor de configuracion centralizada para los microservicios.

Permite:

- externalizar configuracion
- separar codigo de configuracion
- soportar multiples entornos (`dev`, `prod`)
- facilitar despliegue de microservicios

---

## Configuracion utilizada

Modo:

```text
native
```

Ruta del repositorio montado:

```text
/config-repo
```

---

## Levantar Config Server

DEV (desde `infra/config-server`):

```bash
mvn spring-boot:run
```

PROD (desde `infra`):

```bash
docker compose up -d config-server
```

---

## Pruebas de Config Server

DEV:

```bash
curl http://localhost:7071/catalogo/dev
```

PROD:

```bash
curl http://localhost:7072/catalogo/prod
```

---

# Registry Server (Eureka)

## Descripcion

Servidor de registro y descubrimiento de servicios.

Permite:

- registro automatico de microservicios
- descubrimiento dinamico
- integracion posterior con API Gateway (`lb://`)

---

## Levantar Registry Server

DEV (desde `infra/registry-server`):

```bash
mvn spring-boot:run
```

PROD (desde `infra`):

```bash
docker compose up -d registry-server
```

---

## Acceso a Eureka

DEV:

```text
http://localhost:8761
```

PROD (host):

```text
http://localhost:8762
```

---

# config-repo

Contiene la configuracion externa de infraestructura y microservicios.

Archivos actuales:

```text
config-repo/
  catalogo-dev.yml
  catalogo-prod.yml
  registry-server-dev.yml
  registry-server-prod.yml
```

Ejemplo:

```yaml
eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
```

---

# Flujo de uso

1. Levantar infraestructura base

```bash
docker compose up -d
```

2. Verificar endpoints

```text
http://localhost:7072/catalogo/prod
http://localhost:8762
```

3. Levantar microservicio (ejemplo: catalogo)

4. Verificar registro del microservicio en Eureka

---

# Problemas comunes

## 1. Microservicio no conecta a config-server

Causa:
- red incorrecta

Solucion:
- conectar el servicio a `ms-net`

---

## 2. Microservicio no aparece en Eureka

Causa:
- `defaultZone` incorrecto
- `registry-server` no disponible

Solucion:
- en DEV usar `http://localhost:8761/eureka`
- en Docker usar `http://registry-server:8761/eureka`

---

## 3. Configuracion no cargada

Causa:
- archivo no existe en `config-repo`

Solucion:
- verificar nombres por entorno (`*-dev.yml`, `*-prod.yml`)

---

## 4. Uso incorrecto de localhost en Docker

Dentro de Docker:

- Incorrecto: `localhost`
- Correcto: `config-server`, `registry-server`

---

# Estado de avance

- [x] Config Server
- [x] Registry Server (Eureka)
- [ ] API Gateway
- [ ] Circuit Breaker
- [ ] Seguridad
- [ ] Balanceador

---

# Siguiente paso

Implementar **API Gateway** para:

- enrutar trafico hacia servicios registrados
- usar descubrimiento dinamico (`lb://catalogo`)
- preparar balanceo entre multiples instancias

---

# Tag sugerido

```bash
git tag -a vs03-registry-server -m "Infraestructura: Registry Server operativo"
git push origin vs03-registry-server
```