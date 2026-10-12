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
docker compose exec web python manage.py migrate
```

Otwórz http://localhost.

## Rozwijanie aplikacji lokalnie

Zmień w `.env` `DB_HOST` na `localhost` oraz dodaj `DEBUG=True`, a następnie poprzez konsolę uruchom samą bazę danych i serwer deweloperski Django:

```bash
docker compose up -d db
py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
python manage.py migrate
python manage.py runserver
```

Otwórz http://127.0.0.1:8000. Instrukcja nie dotyczy wdrożenia produkcyjnego.