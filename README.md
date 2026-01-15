[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/V0UhzOJZ)
# P4.1 — Segunda Parte (Evaluacion RA2) — Nginx en Docker

Documento de entrega unico (obligatorio):
[DESPLIEGUE.md](./DESPLIEGUE.md)

En DESPLIEGUE.md debes incluir:
- Parte 1: evidencias de que tu infraestructura inicial funciona (Nginx + SFTP + volumen + webs + HTTPS).
- Parte 2: respuestas y evidencias de los criterios RA2 (a–j) de este README.

---

## 0) Requisitos y reglas de entrega

### 0.1 Requisitos
- Docker instalado
- Docker Compose plugin disponible

Comprobacion:
```bash
docker --version
docker compose version
```

### 0.2 Estructura recomendada del repo

```
.
├── README.md
├── DESPLIEGUE.md
├── docker-compose.yml
├── default.conf
├── nginx-selfsigned.crt
├── nginx-selfsigned.key
├── webdata/
└── evidencias/
```

### 0.3 Normas para evidencias (obligatorio)
- Todas las capturas van en evidencias/.
- Todas las evidencias deben aparecer enlazadas en DESPLIEGUE.md, con:
  - que demuestra
  - comando ejecutado (si aplica)
  - captura (archivo de imagen)

---

# 1) Parte 1 — Evidencias minimas (deben estar en DESPLIEGUE.md)

Estas evidencias confirman que la primera parte (infraestructura base) esta operativa.

## Fase 1: Instalacion y configuracion

1) Servicio Nginx activo
- Evidencia requerida: captura de `docker compose ps` (desde fuera) o `service nginx status` (desde dentro del contenedor) mostrando el servicio activo. Nota: `systemctl` no suele funcionar dentro de Docker.

2) Configuracion cargada
- Evidencia requerida: captura listando el directorio de configuracion dentro del contenedor (ej: `ls -l /etc/nginx/conf.d/` o `sites-enabled` segun la imagen usada) donde se vea tu archivo `.conf`.

3) Resolucion de nombres
- Evidencia requerida: captura del navegador con la barra de direcciones mostrando `http://nombre_web` (no la IP) y la pagina cargada.

4) Contenido Web
- Evidencia requerida: la misma captura anterior, pero debe verse claramente el diseno de la web importada de Cloud Academy.

## Fase 2: Transferencia SFTP (Filezilla)

5) Conexion SFTP exitosa
- Evidencia requerida: captura de Filezilla mostrando en el panel de registro "Status: Connected to..." y en el panel derecho el listado de carpetas remoto. Nota: en Docker la ruta suele ser `/home/usuario/upload`, no `/var/www`.

6) Permisos de escritura
- Evidencia requerida: captura de Filezilla mostrando la transferencia completada o los archivos ya presentes en el servidor remoto.

## Fase 3: Infraestructura Docker

7) Contenedores activos
- Evidencia requerida: captura de `docker compose ps` donde se vean los dos servicios con estado Up y los puertos `0.0.0.0:8080->80/tcp` y `0.0.0.0:2222->22/tcp`.

8) Persistencia (Volumen compartido)
- Evidencia requerida: evidencia cruzada (Filezilla + navegador) demostrando que lo subido al SFTP se ve en la web (por ejemplo `localhost:8080`).

9) Despliegue multi-sitio
- Evidencia requerida: captura del navegador en `http://localhost:8080/reloj` mostrando el reloj funcionando.

## Fase 4: Seguridad HTTPS

10) Cifrado SSL
- Evidencia requerida: captura del navegador accediendo por `https://...` mostrando el candado (o alerta de certificado autofirmado) y el puerto configurado (ej. `8443`).

11) Redireccion forzada
- Evidencia requerida: captura de la pestaña Network (DevTools) mostrando `301 Moved Permanently` al intentar entrar por HTTP.

---

# 2) Parte 2 — Evaluacion RA2 (a–j)

A partir de aqui, TODO lo que hagas debe quedar documentado en DESPLIEGUE.md con:
- explicacion breve
- fragmento de configuracion si aplica
- evidencias con comandos y capturas

---

## a) Parametros de administracion mas importantes del servidor web

### Tarea
1. Localiza en el contenedor los fragmentos relevantes de /etc/nginx/nginx.conf donde aparezcan:
- worker_processes
- worker_connections
- access_log
- error_log
- keepalive_timeout
- include
- gzip (aunque este comentado)

2. Para cada directiva:
- que controla
- un ejemplo realista de configuracion incorrecta y su efecto
- como lo comprobarías (comando + evidencia)

3. Aplica un cambio seguro y medible:
- ajusta keepalive_timeout (ej. 65 → 30) sin editar directamente el nginx.conf del contenedor
- demuestra nginx -t OK + recarga OK

### Evidencias obligatorias
- evidencias/a-01-grep-nginxconf.png
  Que demuestra: aparecen las directivas solicitadas en nginx.conf.
  Comando antes de capturar:
```bash
docker compose exec web sh -c "grep -nE 'worker_processes|worker_connections|access_log|error_log|gzip|include|keepalive_timeout' /etc/nginx/nginx.conf"
```

- evidencias/a-02-nginx-t.png
  Que demuestra: validacion correcta tras el cambio.
  Comando antes de capturar:
```bash
docker compose exec web nginx -t
```

- evidencias/a-03-reload.png
  Que demuestra: recarga correcta tras el cambio.
  Comando antes de capturar:
```bash
docker compose exec web nginx -s reload
```

---

## b) Ampliacion de funcionalidad mediante activacion y configuracion (elige 1 opcion) + modulo adicional investigado

Elige una de las dos opciones: B1 (Gzip) o B2 (Cabeceras).
Ademas, obligatorio: investigar y explicar un modulo real de Nginx consultando fuentes en Internet.

### B1) Opcion Gzip (activacion y prueba)

#### Tarea
1. Crea gzip.conf con:
- gzip on;
- gzip_types (minimo: html, css, js, json, text/plain)
- gzip_comp_level 5;
- gzip_vary on;

2. Monta gzip.conf en /etc/nginx/conf.d/gzip.conf mediante volumen.
3. Valida y recarga Nginx.
4. Demuestra con curl que una respuesta se entrega comprimida.

#### Evidencias obligatorias (B1)
- evidencias/b1-01-gzipconf.png
  Que demuestra: contenido de gzip.conf (configuracion aplicada).

- evidencias/b1-02-compose-volume-gzip.png
  Que demuestra: montaje de gzip.conf en docker-compose.yml (volumen).

- evidencias/b1-03-nginx-t.png
  Que demuestra: config valida.
  Comando:
```bash
docker compose exec web nginx -t
```

- evidencias/b1-04-curl-gzip.png
  Que demuestra: Content-Encoding: gzip en respuesta.
  Comando:
```bash
curl -I -H "Accept-Encoding: gzip" http://localhost:8080/
curl -I -k -H "Accept-Encoding: gzip" https://localhost:8443/
```

Si no aparece compresion, crea un recurso grande y prueba contra el:
```bash
python3 - <<'PY'
from pathlib import Path
Path("webdata/largo.txt").write_text(("Linea de prueba para gzip.\n"*2000))
print("Creado webdata/largo.txt")
PY
curl -I -H "Accept-Encoding: gzip" http://localhost:8080/largo.txt
```

### B2) Opcion Cabeceras de seguridad (activacion y prueba)

#### Tarea
1. Anade en el server que sirve contenido (preferiblemente el de HTTPS) las cabeceras:
- X-Content-Type-Options: nosniff
- X-Frame-Options: DENY
- Content-Security-Policy basica

2. Valida y recarga.
3. Demuestra con curl que aparecen las cabeceras en HTTPS.
   (En HTTP puede no aparecer si la respuesta es un 301 generado por otro server block.)

#### Evidencias obligatorias (B2)
- evidencias/b2-01-defaultconf-headers.png
  Que demuestra: configuracion de cabeceras en default.conf. Y que significa cada una.

- evidencias/b2-02-nginx-t.png
  Que demuestra: config valida.
  Comando:
```bash
docker compose exec web nginx -t
```

- evidencias/b2-03-curl-https-headers.png
  Que demuestra: cabeceras presentes en respuesta HTTPS.
  Comando:
```bash
curl -I -k https://localhost:8443/
```

### Obligatorio en b) — Modulo adicional investigado (con fuente)

#### Tarea
Busca en Internet un modulo real de Nginx y explica en DESPLIEGUE.md:
- nombre del modulo
- para que sirve
- como se instala/carga “de forma general” (paquete / load_module / compilacion)
- enlace(s) a la fuente consultada

#### Evidencia obligatoria
- Seccion en DESPLIEGUE.md llamada:
  Modulo investigado: <NOMBRE>
  con explicacion y enlace(s) a fuente.

---

## c) Creacion y configuracion de sitios virtuales / multi-sitio por path + otros tipos

### Tarea
1. Evidencia del multi-sitio por path ya hecho:
- web principal en /
- web secundaria en /reloj

2. Explica (6–10 lineas):
- diferencia entre multi-sitio por path y por nombre (server_name)
- indica al menos 2 tipos adicionales de multi-sitio que conozcas (por ejemplo: por nombre, por puerto, por IP, subdominios).

3. Localiza y muestra tu configuracion activa dentro del contenedor:
- fichero default.conf montado en /etc/nginx/conf.d/default.conf
- senala directivas clave: root, location, try_files y como se soporta /reloj

### Evidencias obligatorias
- evidencias/c-01-root.png
  Que demuestra: web principal en /.

- evidencias/c-02-reloj.png
  Que demuestra: web secundaria en /reloj.

- evidencias/c-03-defaultconf-inside.png
  Que demuestra: contenido de default.conf activo dentro del contenedor.
  Comando:
```bash
docker compose exec web sh -c "sed -n '1,220p' /etc/nginx/conf.d/default.conf"
```

---

## d) Autenticacion y control de acceso

### Tarea: proteger /admin
1. Crea webdata/admin/index.html con contenido simple.
2. Configura auth_basic para /admin/ con .htpasswd.
3. Prueba:
- sin credenciales → 401
- con credenciales → 200

### Evidencias obligatorias
- evidencias/d-01-admin-html.png
  Que demuestra: existe el contenido webdata/admin/index.html (captura del fichero o del listado).

- evidencias/d-02-defaultconf-auth.png
  Que demuestra: location /admin/ con auth_basic configurado.

- evidencias/d-03-curl-401.png
  Que demuestra: acceso sin credenciales devuelve 401.
  Comando:
```bash
curl -I -k https://localhost:8443/admin/
```

- evidencias/d-04-curl-200.png
  Que demuestra: acceso con credenciales devuelve 200.
  Comando:
```bash
curl -I -k -u admin:Admin1234\! https://localhost:8443/admin/
```

---

## e) Obtencion e instalacion de certificados digitales

### Tarea
1. Explica en DESPLIEGUE.md:
- que es .crt y .key
- por que -nodes se usa en laboratorio

2. Evidencia de:
- ficheros generados
- montaje en compose
- uso en default.conf

### Evidencias obligatorias
- evidencias/e-01-ls-certs.png
  Que demuestra: existen los certificados en el host.
  Comando:
```bash
ls -l nginx-selfsigned.crt nginx-selfsigned.key
```

- evidencias/e-02-compose-certs.png
  Que demuestra: montaje de cert y key en docker-compose.yml.

- evidencias/e-03-defaultconf-ssl.png
  Que demuestra: ssl_certificate y ssl_certificate_key con rutas correctas.

---

## f) Asegurar comunicaciones cliente-servidor

### Tarea
1. Evidencia de HTTPS operativo.
2. Evidencia de redireccion HTTP→HTTPS con 301.
3. Explica por que se usan dos server blocks (80 redirige, 443 sirve).

### Evidencias obligatorias
- evidencias/f-01-https.png
  Que demuestra: navegacion por https://localhost:8443.

- evidencias/f-02-301-network.png
  Que demuestra: 301 en DevTools → Network al entrar por HTTP.

---

## g) Documentacion (configuracion, administracion segura y recomendaciones)

### Tarea
Completa DESPLIEGUE.md con:
- arquitectura (servicios, puertos, volumenes)
- configuracion Nginx (ubicacion de default.conf, server blocks, root, /reloj)
- seguridad (certificados, HTTPS, redirect, opcion b elegida)
- autenticacion /admin (si procede)
- logs y analisis (criterio j)
- evidencias Parte 1 + Parte 2 con enlaces a capturas

### Evidencia obligatoria
- DESPLIEGUE.md enlazado al inicio de este README (ya esta enlazado arriba).

---

## h) Ajustes necesarios para implantacion de aplicaciones en Nginx

### Tarea
1. Explica que implica desplegar una segunda app en /reloj (rutas relativas/absolutas).
2. Describe un problema tipico de permisos al subir por SFTP y tu solucion.
3. Evidencia de / y /reloj.

### Evidencias obligatorias
- evidencias/h-01-root.png
  Que demuestra: / funciona.

- evidencias/h-02-reloj.png
  Que demuestra: /reloj funciona.

---

## i) Virtualizacion en despliegue (contenedores)

### Tarea
1. Explica diferencia operativa entre:
- instalacion nativa en SO
- contenedor efimero + configuracion por volumenes

2. Evidencia de contenedores en marcha.

### Evidencias obligatorias
- evidencias/i-01-compose-ps.png
  Que demuestra: servicios activos con puertos y estado Up.
  Comando:
```bash
docker compose ps
```

---

## j) Logs: monitorizacion, consolidacion y analisis en tiempo real

### Tarea
1. Genera trafico y errores 404:
```bash
for i in $(seq 1 20); do curl -s -o /dev/null http://localhost:8080/; done
for i in $(seq 1 10); do curl -s -o /dev/null http://localhost:8080/no-existe-$i; done
```

2. Monitoriza en tiempo real:
```bash
docker compose logs -f web
```

3. Extrae metricas basicas (top URLs, codigos, 404) desde el contenedor:
```bash
docker compose exec web sh -c "awk '{print \$7}' /var/log/nginx/access.log | sort | uniq -c | sort -nr | head"
docker compose exec web sh -c "awk '{print \$9}' /var/log/nginx/access.log | sort | uniq -c | sort -nr | head"
docker compose exec web sh -c "awk '\$9==404 {print \$7}' /var/log/nginx/access.log | sort | uniq -c | sort -nr | head"
```

### Evidencias obligatorias
- evidencias/j-01-logs-follow.png
  Que demuestra: monitorizacion en tiempo real de logs.
  Comando antes de capturar:
```bash
docker compose logs -f web
```

- evidencias/j-02-metricas.png
  Que demuestra: extraccion de metricas (URLs / codigos / 404).
  Comandos antes de capturar: (captura tras ejecutar los 3 comandos anteriores)

---

# 3) Checklist final (debe aparecer en DESPLIEGUE.md)

En DESPLIEGUE.md incluye un checklist con:
- Parte 1: items 1 a 11 ✅/⬜
- Parte 2: RA2 a–j ✅/⬜
  Incluye enlaces a las evidencias correspondientes.


# 4) Entrega

**OBLIGATORIO:** la estructura del repo debe ser similar a la recomendada en 0.2, por tanto tienes que tener todos los archivos que has utilizado (docker-compose.yml, default.conf, certificados, webdata/, evidencias/, DESPLIEGUE.md, README.md).

- Sube todo el repo a tu GitHub.
- Asegurate de que DESPLIEGUE.md incluye todo lo solicitado.
- Comprueba que las evidencias estan en evidencias/ y enlazadas en DESPLIEGUE.md.
- Comparte el enlace del repo a tu instructor o en la plataforma correspondiente