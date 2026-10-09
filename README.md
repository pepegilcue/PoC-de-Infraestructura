# PoC de Infraestructura

**Qué es:** una página web sencilla que funciona dentro de una "cajita" aislada
(un contenedor Docker). Dentro de la cajita hay un programa llamado **Nginx**,
cuyo trabajo es enseñar la web a quien la pida.

Módulo 0614 Despliegue de Aplicaciones Web · 2º DAW · Campus Cámara Comercio Sevilla

## Qué hay en el proyecto

```
nginx-static-poc/
├── conf/default.conf      → las reglas de Nginx (cómo debe comportarse)
├── html/
│   ├── css/style.css      → el diseño (colores, letras)
│   └── index.html         → la página web
├── .dockerignore          → lista de archivos que NO se meten en la cajita
├── docker-compose.yml     → la "receta" para arrancar todo con un solo comando
└── Dockerfile             → instrucciones para fabricar la cajita con la web ya dentro
```

## Qué necesito para usarlo

Windows 11, Docker Desktop, VS Code y Git.

## Cómo llega una visita a la web

```
Navegador ──► Mi ordenador, puerto 8080 ──► Cajita (Docker), puerto 80 (Nginx)
                                              ├─ Web: carpeta html (solo se puede leer)
                                              ├─ Reglas: default.conf (solo se puede leer)
                                              └─ Registros (logs) ► se ven con docker compose logs
```

Explicado: cuando escribes `localhost:8080` en el navegador, tu ordenador manda
la petición a la cajita, donde Nginx responde con la web. Los números 8080 y 80
son como "puertas": la 8080 está en tu ordenador y la 80 está dentro de la cajita.

## Comandos para manejar la cajita

```bash
cd nginx-static-poc                  # ir a la carpeta del proyecto (siempre primero)
docker compose up -d                 # arrancar la cajita (en segundo plano)
docker compose ps                    # ver si está funcionando
curl -I http://localhost:8080        # pedir solo los "datos de cabecera" de la web
docker compose logs -f web           # ver el diario de lo que pasa, en directo
docker compose exec web sh           # entrar dentro de la cajita a mirar
docker compose down                  # apagar y borrar la cajita
```

Otra forma: fabricar la cajita con la web ya metida dentro (sin depender
de mis carpetas):

```bash
docker build -t nginx-static-poc:1.0 .
docker run -d -p 8080:80 --name nginx-static-poc nginx-static-poc:1.0
```

## Medidas de seguridad (en default.conf)

| Medida | Para qué sirve, en simple |
|---|---|
| `server_tokens off` | Nginx deja de decir qué versión es. Si un atacante sabe la versión, sabe qué fallos conocidos probar. |
| `X-Frame-Options: SAMEORIGIN` | Nadie puede meter mi web dentro de otra web para engañar al usuario. |
| `X-Content-Type-Options: nosniff` | El navegador no "adivina" qué es cada archivo, sino que hace caso a lo que dice el servidor. Evita que un archivo se ejecute como programa por error. |
| `Referrer-Policy` | Cuando el usuario sale de mi web a otra, no se le cuenta a esa otra de dónde viene con todo detalle. |
| `Content-Security-Policy: default-src 'self'` | La web solo puede cargar cosas que sean suyas, no de sitios externos. Frena código malicioso metido desde fuera. |
| `always` | Estas protecciones también se ponen cuando hay un error (403, 404), no solo cuando todo va bien. |
| `expires 5m` | El navegador guarda el CSS 5 minutos para cargar más rápido, pero no demasiado tiempo. |
| logs a `/dev/stdout` y `/dev/stderr` | El "diario" de Nginx
