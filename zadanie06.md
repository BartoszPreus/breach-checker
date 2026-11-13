# Zadanie 1 — Własne testy

## 1. z .lower()

```
docker compose exec backend pytest -v
=============================== test session starts ===============================
platform linux -- Python 3.12.15, pytest-8.4.2, pluggy-1.7.0 -- /usr/local/bin/python3.12
cachedir: .pytest_cache
rootdir: /app
collected 7 items                                                                 

tests/test_check.py::test_leaked_hash_is_found PASSED                       [ 14%] 
tests/test_check.py::test_unknown_hash_is_not_found PASSED                  [ 28%] 
tests/test_check.py::test_missing_hash_field PASSED                         [ 42%] 
tests/test_check.py::test_plaintext_password_is_rejected PASSED             [ 57%] 
tests/test_check.py::test_uppercase_leaked_hash_is_found PASSED             [ 71%]
tests/test_check.py::test_39_character_hash_is_rejected PASSED              [ 85%] 
tests/test_check.py::test_non_hex_hash_is_rejected PASSED                   [100%] 

================================ 7 passed in 0.16s ================================
```
```
## 2. bez .lower()

docker compose exec backend pytest -v
=============================== test session starts ===============================
platform linux -- Python 3.12.15, pytest-8.4.2, pluggy-1.7.0 -- /usr/local/bin/python3.12
cachedir: .pytest_cache
rootdir: /app
collected 7 items                                                                 

tests/test_check.py::test_leaked_hash_is_found PASSED                       [ 14%]
tests/test_check.py::test_unknown_hash_is_not_found PASSED                  [ 28%]
tests/test_check.py::test_missing_hash_field PASSED                         [ 42%] 
tests/test_check.py::test_plaintext_password_is_rejected PASSED             [ 57%]
tests/test_check.py::test_uppercase_leaked_hash_is_found FAILED             [ 71%]
tests/test_check.py::test_39_character_hash_is_rejected PASSED              [ 85%] 
tests/test_check.py::test_non_hex_hash_is_rejected PASSED                   [100%] 

==================================== FAILURES =====================================
_______________________ test_uppercase_leaked_hash_is_found _______________________

client = <FlaskClient <Flask 'app'>>

    def test_uppercase_leaked_hash_is_found(client):
        response = client.post("/check", json={"hash": LEAKED.upper()})
>       assert response.status_code == 200
E       assert 400 == 200
E        +  where 400 = <WrapperTestResponse streamed [400 BAD REQUEST]>.status_code

tests/test_check.py:27: AssertionError
============================= short test summary info =============================
FAILED tests/test_check.py::test_uppercase_leaked_hash_is_found - assert 400 == 200
=========================== 1 failed, 6 passed in 0.33s ===========================

```
Dlaczego testy z zajęć nie wykryły usunięcia .lower()?
W testach z zajęć wszystkie poprawne skróty były już zapisane małymi literami, więc usunięcie `.lower()` nie wpływało na ich wyniki. Problem wykrył test `test_uppercase_leaked_hash_is_found`, ponieważ bez normalizacji walidacja odrzuciła skrót zapisany wielkimi literami, a API zwróciło `400` zamiast oczekiwanego `200`.


# Zadanie 2 — Awaria a blokada
```
'{"hash":"7c4a8d09ca3762af61e59520943dc26494f8941b"}' | curl.exe -s -o blad.html -w "HTTP %{http_code}\n" -X POST http://localhost:8080/api/check -H "Content-Type: application/json" -d '@-'
HTTP 500
```
```
docker compose ps -a
NAME                        IMAGE                     COMMAND                  SERVICE    CREATED          STATUS                     PORTS
breach-checker-backend-1    breach-checker-backend    "flask --app app run…"   backend    34 minutes ago   Exited (1) 6 minutes ago
breach-checker-db-1         postgres:16-alpine        "docker-entrypoint.s…"   db         34 minutes ago   Exited (0) 7 minutes ago
breach-checker-frontend-1   breach-checker-frontend   "/docker-entrypoint.…"   frontend   34 minutes ago   Up 33 minutes              0.0.0.0:8080->80/tcp, [::]:8080->80/tcp
```
```
docker inspect --format '{{.State.Status}} {{.State.ExitCode}}' breach-checker-backend-1
exited 1
```
**Treść odpowiedzi zapisanej w `blad.html`**

```text
psycopg2.OperationalError: could not translate host name "db" to address: Name or service not known
```

**Dlaczego taka odpowiedź na serwerze produkcyjnym byłaby problemem bezpieczeństwa?**
Klient otrzymał stronę debuggera Werkzeug ze śladem stosu, ścieżką `/app/app.py` i fragmentami kodu. Komunikat wskazywał, że po zatrzymaniu usługi `db` backend nie mógł rozpoznać jej nazwy w sieci Dockera. Na produkcji taka odpowiedź ujawniałaby użytkownikowi szczegóły kodu i wewnętrznej konfiguracji aplikacji.


| Co porównuję | Awaria po zatrzymaniu bazy | Blokada tabeli z kroku 4 |
|---|---|---|
| Status backendu | Po restarcie `Exited (1)` | `Up (healthy)` w trakcie oczekiwania na blokadę |
| CPU backendu | Po zakończeniu procesu brak odczytu dla backendu w `docker stats` | Bliskie 0%, ponieważ proces czekał na bazę |
| Ostatnie linie logów | `psycopg2.OperationalError: could not translate host name "db" to address: Name or service not known` | W logach było widać, że żądanie dotarło do backendu. Wpis z wynikiem `POST /check` pojawił się dopiero po zwolnieniu blokady. |
| Co dostał klient | `HTTP 500` i stronę błędu Flaska zapisaną w `blad.html` | Żądanie czekało. Przy utrzymaniu blokady do limitu czasu nginx klient dostał `504 Gateway Time-out` |
| Po jakim czasie | Nie zmierzono | Około 60 s do `504`, jeśli blokada trwała do limitu nginx |

Plik `blad.html` zawierał stronę debuggera Flaska ze śladem stosu i wyjątkiem `psycopg2.OperationalError`. Szczegóły wskazywały, że backend nie mógł rozpoznać nazwy hosta bazy. Na produkcji taka odpowiedź ujawniałaby użytkownikom szczegóły kodu i wewnętrznej konfiguracji aplikacji, które mogłyby pomóc w przygotowaniu ataku.


# Zadanie 3 — Frontend w trybie deweloperskim


**Przed poprawką:**
```
curl.exe http://localhost:8080/nginx.conf
server {
    listen 80;

    # Pliki strony (index.html, style.css, script.js)
    location / {
        root /usr/share/nginx/html;
        index index.html;
    }

    # Wszystko pod /api/ nginx przekazuje do usługi backend w sieci Compose        
    location /api/ {
        proxy_pass http://backend:5000/;
    }
}
```

## Po poprawce:
```
curl.exe -I http://localhost:8080/nginx.conf
HTTP/1.1 404 Not Found
Server: nginx/1.31.6
Date: Thu, 12 Nov 2026 23:37:00 GMT
Content-Type: text/html
Content-Length: 153
Connection: keep-alive
```

**Dlaczego obraz produkcyjny frontendu nie ma tego problemu?**
W obrazie produkcyjnym `frontend/Dockerfile` kopiuje do katalogu serwowanego przez nginx tylko `index.html`, `style.css` i `script.js`, a konfigurację nginx umieszcza osobno. 

**Dlaczego montowanie całego katalogu projektu do katalogu serwowanego przez serwer WWW jest ryzykowne?**
Zamontowanie całego katalogu pod katalogiem WWW może udostępnić przez HTTP pliki takie jak `nginx.conf` i `Dockerfile`, ponieważ opcja `:ro` blokuje zapis, ale nadal pozwala nginx je odczytać i udostępnić.