# Uruchomienie

**Wymagania:** Docker Desktop z uruchomionym Docker Engine i Compose. Do pracy lokalnej: Python 3.11.

Utwórz `.env` w katalogu projektu (wpisz własne hasło i sekret):

```
SECRET_KEY=zmien-na-losowy-sekret
ALLOWED_HOSTS=localhost,127.0.0.1
DB_NAME=wuda
DB_USER=wuda
DB_PASSWORD=zmien-haslo
DB_PORT=5432
DB_HOST=db
```

`DEBUG` jest opcjonalne (domyślnie `False`).

## Uruchomienie lokalne w Dockerze

```bash
docker compose up --build -d
```

Migracje wykonują się automatycznie przy starcie. Otwórz http://localhost/polls. Aby dodać nowe pytania
poprzez http://localhost/admin/, należy najpierw dodać superużytkownika:

```bash
docker compose exec web python manage.py createsuperuser
```

## Wdrożenie z obrazu GHCR

Obraz `ghcr.io/kamilfiacek/wuda` jest publikowany automatycznie po udanym teście zmian na gałęzi `master`.
Zespół wdrażający może utworzyć własny plik Compose na serwerze. Poniżej przykład pliku `docker-compose.yml`
zdolnego do uruchomienia aplikacji w zastosowaniu produkcyjnym:

```yaml
name: wuda

services:
  db:
    image: postgres:18
    restart: unless-stopped
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $${POSTGRES_USER} -d $${POSTGRES_DB}"]
      interval: 5s
      timeout: 5s
      retries: 10

  migrate:
    image: ghcr.io/kamilfiacek/wuda:latest
    env_file: .env
    command: python manage.py migrate --noinput
    restart: "no"
    depends_on:
      db:
        condition: service_healthy

  web:
    image: ghcr.io/kamilfiacek/wuda:latest
    restart: unless-stopped
    env_file: .env
    ports:
      - "80:8000"
    depends_on:
      migrate:
        condition: service_completed_successfully

volumes:
  postgres_data:
```

Utwórz obok pliku `docker-compose.yml` plik `.env` z wymaganymi zmiennymi środowiskowymi (`SECRET_KEY`,
`ALLOWED_HOSTS`, `DB_NAME`, `DB_USER`, `DB_PASSWORD`, `DB_PORT=5432`, `DB_HOST=db`).
Ustaw `ALLOWED_HOSTS` na domenę serwera i użyj silnego, prywatnego `SECRET_KEY`.

Uruchom wdrożenie:

```bash
docker compose pull
docker compose up -d
docker compose exec web python manage.py createsuperuser  # wykonaj tylko raz, aby utworzyć konto administratora
```

Tag `latest` wskazuje najnowszy obraz z tej gałęzi. Dla powtarzalnych wdrożeń można zamiast niego
użyć tagu `sha-...` wygenerowanego dla konkretnego commita. Wolumen `postgres_data` przechowuje bazę danych niezależnie od aktualizacji kontenera.

## Rozwijanie aplikacji lokalnie

Zmień w `.env` `DB_HOST` na `127.0.0.1` oraz dodaj `DEBUG=True`, a następnie uruchom samą bazę danych:

```bash
docker compose up -d db
```

Utwórz środowisko wirtualne:

```bash
py -3.11 -m venv .venv
```

Aktywuj je:

```bash
.\.venv\Scripts\Activate.ps1
```

Zainstaluj zależności:

```bash
python -m pip install -r requirements.txt
```

Wykonaj migracje:

```bash
python manage.py migrate
```

Uruchom serwer deweloperski:

```bash
python manage.py runserver
```

Otwórz http://127.0.0.1:8000/polls.