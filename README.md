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

Otwórz http://127.0.0.1:8000/polls. Instrukcja nie dotyczy wdrożenia produkcyjnego.