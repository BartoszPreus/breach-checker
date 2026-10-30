# Zadanie 5 — Docker Compose - łączymy wszystko

## 1. Healthcheck backendu
```
docker compose up -d --build
[+] Building 1.2s (23/23) FINISHED
 => [internal] load local bake definitions                                                                                                                                                          0.0s 
 => => reading from stdin 1.06kB                                                                                                                                                                    0.0s
 => [frontend internal] load build definition from Dockerfile                                                                                                                                       0.0s
 => => transferring dockerfile: 342B                                                                                                                                                                0.0s
 => [backend internal] load build definition from Dockerfile                                                                                                                                        0.0s 
 => => transferring dockerfile: 205B                                                                                                                                                                0.0s 
 => [backend internal] load metadata for docker.io/library/python:3.12-slim                                                                                                                         0.8s 
 => [frontend internal] load metadata for docker.io/library/nginx:alpine                                                                                                                            0.0s 
 => [frontend internal] load .dockerignore                                                                                                                                                          0.0s 
 => => transferring context: 66B                                                                                                                                                                    0.0s 
 => [frontend 1/5] FROM docker.io/library/nginx:alpine@sha256:df221db836e1754089190208cee7eeda94f233197056426eda74a43ab1abeac2                                                                      0.0s 
 => => resolve docker.io/library/nginx:alpine@sha256:df221db836e1754089190208cee7eeda94f233197056426eda74a43ab1abeac2                                                                               0.0s 
 => [frontend internal] load build context                                                                                                                                                          0.0s 
 => => transferring context: 120B                                                                                                                                                                   0.0s 
 => CACHED [frontend 2/5] RUN apk add --no-cache nodejs npm && npm install -g htmlhint                                                                                                              0.0s 
 => CACHED [frontend 3/5] COPY index.html style.css script.js /usr/share/nginx/html/                                                                                                                0.0s 
 => CACHED [frontend 4/5] COPY nginx.conf /etc/nginx/conf.d/default.conf                                                                                                                            0.0s 
 => CACHED [frontend 5/5] RUN htmlhint /usr/share/nginx/html/index.html                                                                                                                             0.0s 
 => [frontend] exporting to image                                                                                                                                                                   0.1s
 => => exporting layers                                                                                                                                                                             0.0s 
 => => exporting manifest sha256:615e8363500703aa63c1decb587c6ded09a1374e69bb512f4672e96281dad733                                                                                                   0.0s 
 => => exporting config sha256:de486cda498dcb175ba6edee81c6d81822b02b38b2d255d7f1200ea2325aeb3b                                                                                                     0.0s 
 => => exporting attestation manifest sha256:24b665118f9c03db93f4719234a42fef5b6f7d0fcf6ad125009432d2571c0b39                                                                                       0.0s 
 => => exporting manifest list sha256:e68d2539ac6de22658feb5e36ba33aea5ee1ce98ffd1bcf5a0c11fffcf6c840e                                                                                              0.0s 
 => => naming to docker.io/library/breach-checker-frontend:latest                                                                                                                                   0.0s 
 => => unpacking to docker.io/library/breach-checker-frontend:latest                                                                                                                                0.0s 
 => [frontend] resolving provenance for metadata file                                                                                                                                               0.0s 
 => [backend internal] load .dockerignore                                                                                                                                                           0.0s
 => => transferring context: 79B                                                                                                                                                                    0.0s
 => [backend 1/5] FROM docker.io/library/python:3.12-slim@sha256:aa943ce411423c7582288f18707ff66eddcbaeb510502d747a72aeb34f30d159                                                                   0.0s 
 => => resolve docker.io/library/python:3.12-slim@sha256:aa943ce411423c7582288f18707ff66eddcbaeb510502d747a72aeb34f30d159                                                                           0.0s 
 => [backend internal] load build context                                                                                                                                                           0.0s 
 => => transferring context: 63B                                                                                                                                                                    0.0s 
 => CACHED [backend 2/5] WORKDIR /app                                                                                                                                                               0.0s 
 => CACHED [backend 3/5] COPY requirements.txt .                                                                                                                                                    0.0s 
 => CACHED [backend 4/5] RUN pip install --no-cache-dir -r requirements.txt                                                                                                                         0.0s 
 => CACHED [backend 5/5] COPY app.py .                                                                                                                                                              0.0s 
 => [backend] exporting to image                                                                                                                                                                    0.1s 
 => => exporting layers                                                                                                                                                                             0.0s 
 => => exporting manifest sha256:82083893706bc7126e119df56b8d24c6034491024296c59f398295551c719bc4                                                                                                   0.0s 
 => => exporting config sha256:dde397730b289f6cf82259b912f1c2485900a5b91365cb11c8e49c4196ef7506                                                                                                     0.0s 
 => => exporting attestation manifest sha256:17b04fb37fef95d9ee7d03a0c75ad8490b5fb199779501e891e291e935f41c01                                                                                       0.0s 
 => => exporting manifest list sha256:077236f72f2d67a09f407af97ae24055147d0e38987e4be9ab2ecfa9677a03d0                                                                                              0.0s 
 => => naming to docker.io/library/breach-checker-backend:latest                                                                                                                                    0.0s 
 => => unpacking to docker.io/library/breach-checker-backend:latest                                                                                                                                 0.0s 
 => [backend] resolving provenance for metadata file                                                                                                                                                0.0s 
[+] up 6/6
 ✔ Image breach-checker-frontend       Built                                                                                                                                                         1.3s
 ✔ Image breach-checker-backend        Built                                                                                                                                                         1.3s
 ✔ Network breach-checker_default      Created                                                                                                                                                       0.0s
 ✔ Container breach-checker-db-1       Healthy                                                                                                                                                       5.8s
 ✔ Container breach-checker-backend-1  Healthy                                                                                                                                                      11.4s
 ✔ Container breach-checker-frontend-1 Started
```
```
docker compose ps
NAME                        IMAGE                     COMMAND                  SERVICE    CREATED          STATUS                    PORTS
breach-checker-backend-1    breach-checker-backend    "python app.py"          backend    16 seconds ago   Up 10 seconds (healthy)   5000/tcp
breach-checker-db-1         postgres:16-alpine        "docker-entrypoint.s…"   db         16 seconds ago   Up 15 seconds (healthy)   5432/tcp
breach-checker-frontend-1   breach-checker-frontend   "/docker-entrypoint.…"   frontend   16 seconds ago   Up 4 seconds              0.0.0.0:8080->80/tcp, [::]:8080->80/tcp
```


## 2. Trwałość bazy

```docker compose exec db psql -U breach -d breachdb -c "INSERT INTO leaked_hashes VALUES ('bc45408273d7e1914b741c3407b0b559764f7876');"
INSERT 0 1
```
```
'{"hash":"bc45408273d7e1914b741c3407b0b559764f7876"}' | curl.exe -X POST http://localhost:8080/api/check -H "Content-Type: application/json" -d '@-'
{"found":true}
```
```
docker compose down
[+] down 4/4
 ✔ Container breach-checker-frontend-1 Removed                                                                                                                                                       0.3s
 ✔ Container breach-checker-backend-1  Removed                                                                                                                                                       3.2s
 ✔ Container breach-checker-db-1       Removed                                                                                                                                                       0.2s
 ✔ Network breach-checker_default      Removed                                                                                                                                                       0.2s
```
```
docker compose up -d
[+] up 4/4
 ✔ Network breach-checker_default      Created                                                                                                                                                       0.0s
 ✔ Container breach-checker-db-1       Healthy                                                                                                                                                       5.8s
 ✔ Container breach-checker-backend-1  Healthy                                                                                                                                                      11.4s
 ✔ Container breach-checker-frontend-1 Started                                                                                                                                                      11.5s
```
```
'{"hash":"bc45408273d7e1914b741c3407b0b559764f7876"}' | curl.exe -X POST http://localhost:8080/api/check -H "Content-Type: application/json" -d '@-'
{"found":true}
```
```
docker compose down -v
[+] down 5/5
 ✔ Container breach-checker-frontend-1 Removed                                                                                                                                                       0.3s
 ✔ Container breach-checker-backend-1  Removed                                                                                                                                                       3.2s
 ✔ Container breach-checker-db-1       Removed                                                                                                                                                       0.2s
 ✔ Volume breach-checker_pgdata        Removed                                                                                                                                                       0.0s
 ✔ Network breach-checker_default      Removed                                                                                                                                                       0.2s
```
```
docker compose up -d
[+] up 5/5
 ✔ Network breach-checker_default      Created                                                                                                                                                       0.0s
 ✔ Volume breach-checker_pgdata        Created                                                                                                                                                       0.0s
 ✔ Container breach-checker-db-1       Healthy                                                                                                                                                       5.8s
 ✔ Container breach-checker-backend-1  Healthy                                                                                                                                                      11.4s
 ✔ Container breach-checker-frontend-1 Started                                                                                                                                                      11.5s
```
```
'{"hash":"bc45408273d7e1914b741c3407b0b559764f7876"}' | curl.exe -X POST http://localhost:8080/api/check -H "Content-Type: application/json" -d '@-'
{"found":false}
```

Dlaczego skrót przetrwał down, a nie przetrwał down -v?
Komenda docker compose down zatrzymuje i usuwa kontenery, ale domyślnie zachowuje wolumeny Docker, w których przechowywane są fizyczne pliki bazy danych. Dopiero użycie flagi -v (--volumes) wymusza usunięcie tych wolumenów, co bezpowrotnie kasuje całą zawartość bazy.

Dlaczego po down -v w bazie znów są 89 skróty z próbki, choć cała baza została skasowana?
Po usunięciu starego wolumenu i ponownym uruchomieniu (up -d) kontener bazy danych tworzy się od nowa, a skrypty inicjalizacyjne projektu automatycznie odtwarzają strukturę tabel i ładują do nich domyślny zestaw początkowy (próbkę 89 skrótów).


## 3. Skalowanie
```
docker compose ps
NAME                        IMAGE                     COMMAND                  SERVICE    CREATED          STATUS                    PORTS
breach-checker-backend-1    breach-checker-backend    "python app.py"          backend    4 minutes ago    Up 4 minutes (healthy)    5000/tcp
breach-checker-backend-2    breach-checker-backend    "python app.py"          backend    18 seconds ago   Up 16 seconds (healthy)   5000/tcp
breach-checker-backend-3    breach-checker-backend    "python app.py"          backend    18 seconds ago   Up 16 seconds (healthy)   5000/tcp
breach-checker-db-1         postgres:16-alpine        "docker-entrypoint.s…"   db         4 minutes ago    Up 4 minutes (healthy)    5432/tcp
breach-checker-frontend-1   breach-checker-frontend   "/docker-entrypoint.…"   frontend   4 minutes ago    Up 4 minutes              0.0.0.0:8080->80/tcp, [::]:8080->80/tcp
```
```
docker compose exec frontend nslookup backend
Server:         127.0.0.11
Address:        127.0.0.11:53

Non-authoritative answer:

Non-authoritative answer:
Name:   backend
Address: 172.19.0.5
Name:   backend
Address: 172.19.0.6
Name:   backend
Address: 172.19.0.3
```
```
docker compose logs backend
...
backend-1  | check: hash=bc454...
backend-1  | 172.19.0.4 - - [30/Oct/2026 12:36:16] "POST /check HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-1  | 172.19.0.4 - - [30/Oct/2026 12:36:16] "POST /check HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-1  | 172.19.0.4 - - [30/Oct/2026 12:36:16] "POST /check HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-1  | 172.19.0.4 - - [30/Oct/2026 12:36:17] "POST /check HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-1  | 172.19.0.4 - - [30/Oct/2026 12:36:17] "POST /check HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-1  | 172.19.0.4 - - [30/Oct/2026 12:36:17] "POST /check HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-1  | 172.19.0.4 - - [30/Oct/2026 12:36:17] "POST /check HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-1  | 172.19.0.4 - - [30/Oct/2026 12:36:17] "POST /check HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-1  | 172.19.0.4 - - [30/Oct/2026 12:36:17] "POST /check HTTP/1.1" 200 -
...
```
```
docker compose restart frontend
[+] restart 0/1
 - Container breach-checker-frontend-1 Restarting                                                                                                                                                    0.4s
```
```
docker compose logs backend    
...
backend-3  | check: hash=bc454...
backend-3  | 172.19.0.4 - - [30/Oct/2026 12:37:30] "POST /check HTTP/1.1" 200 -
...
backend-1  | check: hash=bc454...
backend-1  | 172.19.0.4 - - [30/Oct/2026 12:31:23] "POST /check HTTP/1.1" 200 -
...
backend-2  | check: hash=bc454...
backend-2  | 172.19.0.4 - - [30/Oct/2026 12:37:30] "POST /check HTTP/1.1" 200 -
...
backend-1  | check: hash=bc454...
backend-1  | 172.19.0.4 - - [30/Oct/2026 12:36:16] "POST /check HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-1  | 172.19.0.4 - - [30/Oct/2026 12:36:16] "POST /check HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-1  | 172.19.0.4 - - [30/Oct/2026 12:36:16] "POST /check HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-1  | 172.19.0.4 - - [30/Oct/2026 12:36:17] "POST /check HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-1  | 172.19.0.4 - - [30/Oct/2026 12:36:17] "POST /check HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-1  | 172.19.0.4 - - [30/Oct/2026 12:36:17] "POST /check HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-1  | 172.19.0.4 - - [30/Oct/2026 12:36:17] "POST /check HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-1  | 172.19.0.4 - - [30/Oct/2026 12:36:17] "POST /check HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-1  | 172.19.0.4 - - [30/Oct/2026 12:36:17] "POST /check HTTP/1.1" 200 -
...
backend-1  | check: hash=bc454...
backend-1  | 172.19.0.4 - - [30/Oct/2026 12:37:30] "POST /check HTTP/1.1" 200 -
backend-1  | 127.0.0.1 - - [30/Oct/2026 12:37:30] "GET /health HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-1  | 172.19.0.4 - - [30/Oct/2026 12:37:30] "POST /check HTTP/1.1" 200 -
...
```

Dlaczego przed restartem frontendu ruch trafiał do jednej repliki?
Ruch przed restartem frontendu trafiał do jednej repliki, ponieważ Nginx rozwiązuje nazwę DNS backend na adresy IP jednorazowo podczas swojego startu. Ponieważ w tamtym momencie kontener repliki był tylko jeden, Nginx zapamiętał pojedynczy adres IP i kierował do niego cały ruch, ignorując późniejsze pojawienie się nowych replik (dopiero polecenie docker compose restart frontend zmusiło Nginx do ponownego odpytania DNS i odkrycia wszystkich trzech adresów IP). 

Dlaczego skalowanie byłoby niemożliwe, gdyby backend nadal miał ports: "5000:5000"?
Skalowanie byłoby niemożliwe, gdyby backend nadal miał zdefiniowany port hosta (ports: "5000:5000"), ponieważ doszłoby do konfliktu portów – system operacyjny hosta pozwoliłby na zmapowanie portu 5000 tylko dla pierwszej uruchomionej repliki, a kolejne próby startu kontenerów zakończyłyby się błędem zajętości portu.