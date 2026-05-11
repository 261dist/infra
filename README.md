# Infraestructura de Microservicios

Este modulo contiene la infraestructura base de la plataforma:

- Config Server
- Registry Server/Eureka
- Gateway
- `config-repo`
- red compartida `ms-net`

Kafka, observabilidad y microservicios viven en modulos separados, pero se integran con esta infraestructura.

## Componentes

| Componente | Rol |
|---|---|
| `config-server` | Sirve configuracion centralizada desde `infra/config-repo` |
| `registry-server` | Eureka para registro y descubrimiento |
| `gateway` | Punto unico de entrada HTTP y validacion JWT en el borde |
| `config-repo` | Configuracion por servicio y perfil |
| `ms-net` | Red Docker compartida de produccion |

## Puertos

| Servicio | DEV | PROD |
|---|---:|---:|
| Config Server | 7071 | 7072 |
| Registry Server | 7081 | 7082 |
| Gateway | 7091 | 7092 |

En Docker prod, los servicios se comunican por nombre interno:

```text
config-server:7071
registry-server:7081
gateway:7091
```

## Red Compartida

`infra/docker-compose.yml` crea la red:

```text
ms-net
```

La consumen como red externa:

- `auth`
- `catalogo`
- `producto`
- `orden-ms`
- `pago-ms`
- `kafka`
- `observability`

`infra` debe levantarse primero en prod para crear `ms-net`.

## Arquitectura

```text
cliente -> gateway -> microservicios
                    -> Eureka
                    -> Config Server -> config-repo

microservicios -> kafka
observability -> metricas/logs de infra y services
```

## config-repo

Contiene la configuracion externa por servicio y perfil.

Archivos principales:

```text
auth-dev.yml
auth-prod.yml
catalogo-dev.yml
catalogo-prod.yml
producto-dev.yml
producto-prod.yml
orden-ms-dev.yml
orden-ms-prod.yml
pago-ms-dev.yml
pago-ms-prod.yml
gateway-dev.yml
gateway-prod.yml
registry-server-dev.yml
registry-server-prod.yml
```

Los microservicios importan Config Server desde su `application.yml`:

```yaml
spring:
  config:
    import: "optional:configserver:${CONFIG_SERVER_URL:http://localhost:7071}"
```

## Servicios Integrados

| Servicio | Config centralizada | Eureka | Gateway | Observability |
|---|---|---|---|---|
| `auth` | si | si | si | si |
| `catalogo` | si | si | si | si |
| `producto` | si | si | si | si |
| `orden-ms` | si | si | si | si |
| `pago-ms` | si | si | si | si |

## Rutas Gateway

Rutas principales:

| Ruta | Servicio |
|---|---|
| `/auth/**` | `auth` |
| `/api/v1/categorias/**` | `catalogo` |
| `/api/v1/productos/**` | `producto` |
| `/ordenes/**` | `orden-ms` |
| `/pagos/**` | `pago-ms` |

En dev tambien se exponen rutas Swagger para los servicios que lo tienen habilitado.

## Seguridad

La seguridad principal esta centralizada en Gateway:

- `/auth/**` es publico.
- Actuator basico del Gateway queda publico para health/info/prometheus.
- Swagger dev queda publico.
- El resto de rutas requiere JWT.

Esto significa que `/ordenes/**` y `/pagos/**` quedan protegidos cuando se accede por Gateway.

Nota: los puertos directos de los microservicios se mantienen para laboratorio y pruebas. Para una produccion mas estricta, se pueden dejar sin publicar y consumirlos solo por `ms-net`.

## Levantar DEV

En dev normalmente se ejecutan los componentes Java con Maven.

Config Server:

```powershell
cd C:\ms1\ProyectosMS2026\infra\config-server
mvn spring-boot:run
```

Registry Server:

```powershell
cd C:\ms1\ProyectosMS2026\infra\registry-server
.\mvnw.cmd spring-boot:run
```

Gateway:

```powershell
cd C:\ms1\ProyectosMS2026\infra\gateway
.\mvnw.cmd spring-boot:run
```

Validaciones:

```text
http://localhost:7071/catalogo/dev
http://localhost:7071/orden-ms/dev
http://localhost:7071/pago-ms/dev
http://localhost:7081
http://localhost:7091/actuator/health
```

## Levantar PROD

Primero infra:

```powershell
cd C:\ms1\ProyectosMS2026\infra
docker compose up -d
```

Luego Kafka:

```powershell
cd C:\ms1\ProyectosMS2026\kafka
docker compose up -d
```

Luego microservicios, por ejemplo:

```powershell
cd C:\ms1\ProyectosMS2026\services\auth
docker compose up -d

cd C:\ms1\ProyectosMS2026\services\catalogo
docker compose up -d

cd C:\ms1\ProyectosMS2026\services\producto
docker compose up -d

cd C:\ms1\ProyectosMS2026\services\orden-ms
docker compose up -d

cd C:\ms1\ProyectosMS2026\services\pago-ms
docker compose up -d
```

Finalmente observability:

```powershell
cd C:\ms1\ProyectosMS2026\observability
docker compose up -d
```

Validaciones:

```text
http://localhost:7072/orden-ms/prod
http://localhost:7072/pago-ms/prod
http://localhost:7082
http://localhost:7092/actuator/health
```

## Observabilidad

`infra` no levanta Prometheus, Loki ni Grafana. Eso vive en `observability`.

La plataforma expone:

- Actuator en Gateway.
- Actuator/Prometheus en microservicios.
- Logs de Gateway en `infra/gateway/logs`.

`observability` consume metricas y logs desde:

- `gateway`
- `catalogo`
- `producto`
- `orden-ms`
- `pago-ms`
- `kafka-exporter`

## Problemas Comunes

### Config no carga

Revisar:

- que Config Server este arriba
- que `CONFIG_SERVER_URL` apunte a `http://config-server:7071` en Docker
- que exista el archivo `{spring.application.name}-{profile}.yml` en `config-repo`

### Servicio no aparece en Eureka

Revisar:

- dev: `http://localhost:7081/eureka`
- prod: `http://registry-server:7081/eureka`
- que el servicio tenga dependencia Eureka Client

### Gateway no enruta

Revisar:

- que el servicio aparezca en Eureka
- que la ruta exista en `gateway-dev.yml` o `gateway-prod.yml`
- que el JWT sea valido si la ruta no es publica

### Error usando localhost dentro de Docker

Dentro de Docker no usar `localhost` para otros contenedores.

Usar:

- `config-server`
- `registry-server`
- `kafka`
- nombre del servicio en `ms-net`

## Estado Actual

- [x] Config Server
- [x] Registry Server/Eureka
- [x] Gateway con `lb://`
- [x] Config centralizada para `auth`, `catalogo`, `producto`, `orden-ms`, `pago-ms`
- [x] Red compartida `ms-net`
- [x] Seguridad JWT en Gateway
- [x] Integracion Kafka para `orden-ms` y `pago-ms`
- [x] Configuracion Eureka dev con `localhost` estable
- [x] Rutas Gateway para `/api/v1/ordenes/**` y `/api/v1/pagos/**`
- [x] Actuator/Prometheus en servicios integrados
- [x] Logs centralizados consumibles por observability
- [ ] Politicas avanzadas de trafico en Gateway
- [ ] Seguridad por microservicio como defensa en profundidad
- [ ] Integracion con frontend

---

## Tag sugerido

```bash
git tag -a vs09-kafka -m "eda con vs09-kafka"
git push origin vs09-kafka
```
