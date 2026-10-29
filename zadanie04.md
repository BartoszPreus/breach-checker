# Zadanie 4 — sieci, porty i dane

## 1. Endpoint GET /health

Odpowiedź z przeglądarki:
```
{"status":"ok"}
```
```
Odpowiedź z terminala PowerShell:
curl.exe -i http://localhost:5000/health
HTTP/1.1 200 OK
Server: Werkzeug/3.1.9 Python/3.12.15
Date: Thu, 29 Oct 2026 20:11:44 GMT
Content-Type: application/json
Content-Length: 16
Access-Control-Allow-Origin: http://localhost:8080
Connection: close

{"status":"ok"}
```

```
docker logs backend
172.19.0.1 - - [29/Oct/2026 20:04:04] "GET /health HTTP/1.1" 200
```


## 2. Kontener diagnostyczny healthcheck-probe
```
docker run --rm --network breach-net healthcheck-probe
backend: OK
```
```
docker run --rm healthcheck-probe 
backend: UNREACHABLE
```
```
docker stop backend
backend
```
```
docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```
```
docker run --rm --network breach-net healthcheck-probe
backend: UNREACHABLE
```

Bez opcji --network breach-net sonda trafia do domyślnej sieci bridge, a backend działa w osobnej sieci breach-net. Domyślna sieć bridge nie zapewnia automatycznego tłumaczenia nazw kontenerów na adresy IP, dlatego sonda nie odnajduje nazwy backend, mimo że backend jest uruchomiony.

## 3. Trwałość danych:
```
docker exec backend sh -c "echo notatka > /tmp/notatka.txt"
docker restart backend
backend
docker exec backend cat /tmp/notatka.txt
notatka
```
```
docker rm -f backend
backend
docker run -d --name backend --network breach-net -p 5000:5000 -v ./data/leaked-sha1.txt:/app/data/leaked-sha1.txt:ro breach-checker-backend:v1
>>
47e3f0f13b4fcef1565562acbbed64b0b355ce548149bb2913efa33abc2ea005
docker exec backend cat /tmp/notatka.txt
cat: /tmp/notatka.txt: No such file or directory
```
```
docker run --rm -v breach-dane:/dane alpine sh -c "date >> /dane/historia.txt"
docker run --rm -v breach-dane:/dane alpine sh -c "date >> /dane/historia.txt; cat /dane/historia.txt"
Thu Oct 29 20:41:10 UTC 2026
Thu Oct 29 20:41:52 UTC 2026
```

Plik /tmp/notatka.txt przetrwał restart, ponieważ restart zachowuje warstwę zapisywalną kontenera, natomiast docker rm usuwa ją razem z kontenerem. Plik historia.txt przetrwał usunięcie obu kontenerów, ponieważ znajdował się w nazwanym wolumenie breach-dane, który istnieje niezależnie od nich. Lista wycieków również przetrwałaby docker rm, ponieważ znajduje się na dysku hosta i jest jedynie udostępniana kontenerowi przez bind mount.