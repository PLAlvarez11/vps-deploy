### Configuración del Entorno 
Esta sección describe cómo se configuran las variables de entorno, 
el servidor Caddy para producción y la orquestación del servicio mediante docker-compose.vps.yml.

 ## Archivo .env (Entorno Local)
El archivo .env define las variables necesarias para ejecutar el servicio en modo local.
Debe colocarse en la raíz del proyecto.

 ## Función de cada variable
| Variable   | Descripción                        |
| ---------- | ---------------------------------- |
| APP_ENV    | Entorno actual (local/dev/prod)    |
| APP_DEBUG  | Modo debug                         |
| APP_PORT   | Puerto donde corre el servicio PHP |
| DW_HOST    | Host del DW (PostgreSQL)           |
| DW_DB      | Base de datos analítica            |
| DW_USER    | Usuario con permisos de lectura    |
| REDIS_HOST | Host de Redis                      |

 ## Archivo .env.vps (Entorno de Producción en VPS)
Este archivo define la configuración final utilizada para despliegue en servidores productivos, 
usualmente protegidos por Caddy y HTTPS.

 ## Diferencias con .env local
-APP_DEBUG está desactivado.
-Las credenciales deben ser definitivas y seguras.
-Usa hostnames productivos: postgres_dw, redis_cache.
-Se agrega PUBLIC_URL para Caddy o rutas de proxy.

 ## Archivo Caddyfile
-Este archivo configura el servidor web Caddy, encargado de:
-TLS automático con Let’s Encrypt.
-Redirección HTTPS.
-Reverse Proxy hacia el servicio PHP interno.
-Seguridad adicional (CORS, HSTS).

 ## Explicación de los bloques
| Bloque          | Función                                             |
| --------------- | --------------------------------------------------- |
| `reverse_proxy` | Envía tráfico HTTPS al contenedor del microservicio |
| `tls`           | Certificado Let’s Encrypt                           |
| `encode gzip`   | Compresión de respuestas                            |
| `header`        | Aumenta la seguridad del proxy                      |

 ## Archivo docker-compose.vps.yml
Este archivo levanta el servicio en un VPS con contenedores interconectados.
Usa .env.vps como archivo de configuración.

 ## Flujo de Despliegue en VPS
1. Copiar archivo de configuración
cp .env.vps.example .env.vps
2. Crear red compartida
docker network create analytics-net || true
3. Levantar servicios
docker compose -f docker-compose.vps.yml --env-file .env.vps up -d --build
4. Verificar estado
docker ps
docker logs -f report-service
docker logs -f caddy-server
