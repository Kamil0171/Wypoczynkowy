# Wypoczynkowy

Aplikacja webowa rozwijana we współpracy z firmą **Webtechnika** przez **5‑osobowy zespół** w ramach przedmiotu *Projekt zespołowy*. Portal udostępnia moduły ogłoszeń, wydarzeń i imprez oraz system opinii/ocen, z uwierzytelnianiem i panelem użytkownika. Projekt oparty o **Django 5** i **PostgreSQL**, przygotowany do uruchomienia lokalnie lub w kontenerach.

## Spis treści
- [Funkcje](#funkcje)
- [Stos technologiczny](#stos-technologiczny)
- [Szybki start (lokalnie)](#szybki-start-lokalnie)
- [Szybki start (Docker Compose)](#szybki-start-docker-compose)
- [Zmienne środowiskowe (.env)](#zmienne-środowiskowe-env)
- [URL-e / nawigacja](#url-e--nawigacja)

## Funkcje
- **Ogłoszenia** – lista, szczegóły i dodawanie własnych ogłoszeń; filtrowanie po kategorii, lokalizacji i cenie, paginacja.
- **Wydarzenia i imprezy** – przegląd i szczegóły z datami rozpoczęcia/zakończenia.
- **Opinie i oceny** – wystawianie 1–5★ do ogłoszeń (jedna ocena per użytkownik), średnie oceny i sortowanie.
- **Uwierzytelnianie** – rejestracja, logowanie, wylogowanie (Django auth).
- **Panel użytkownika** – podgląd własnych wpisów oraz najlepiej ocenianych treści.
- **Kontakt** – formularz z ochroną **reCAPTCHA** i wysyłką e‑mail (SMTP).
- **Zarządzanie multimediami** – obsługa obrazów (Pillow), katalogi `media/` i `static/`.

## Stos technologiczny
- **Backend:** Python, **Django 5.x**, WSGI (**Gunicorn** w prod)
- **Baza danych:** **PostgreSQL**
- **Front:** szablony Django + statyczne pliki (CSS/JS)
- **Biblioteki:** `django-recaptcha`, `Pillow`, `requests`, `python-dotenv`
- **Konteneryzacja:** Docker / Docker Compose (przykładowe pliki poniżej)

> Wymagania: Python 3.11+, pip, (opcjonalnie) Docker 24+ i Docker Compose.

## Szybki start (lokalnie)

1. Sklonuj repozytorium i przejdź do katalogu projektu:
   ```bash
   git clone <URL-repo>
   cd Wypoczynkowy
   ```

2. Utwórz i aktywuj wirtualne środowisko, zainstaluj zależności:
   ```bash
   python -m venv .venv
   # Windows
   .venv\Scripts\activate
   # macOS/Linux
   # source .venv/bin/activate

   pip install -r requirements.txt
   ```

3. Skonfiguruj plik `.env` (patrz niżej) i uruchom bazę PostgreSQL lokalnie albo wskaż istniejącą instancję.

4. Zastosuj migracje i uruchom serwer deweloperski.
   > Repozytorium nie zawiera pliku `manage.py`. Użyj poniższych poleceń z modułem ustawień:
   ```bash
   # Windows (PowerShell)
   $env:DJANGO_SETTINGS_MODULE="Wypoczynkowy.settings"
   python -m django migrate
   python -m django runserver

   # macOS/Linux (bash/zsh)
   export DJANGO_SETTINGS_MODULE="Wypoczynkowy.settings"
   python -m django migrate
   python -m django runserver
   ```

5. (Opcjonalnie) utwórz konto administratora:
   ```bash
   python -m django createsuperuser
   ```

Aplikacja będzie dostępna pod adresem `http://127.0.0.1:8000/`.

## Szybki start (Docker Compose)

Jeśli chcesz uruchomić projekt w kontenerach, dodaj do repo dwa pliki i uruchom Compose.

**`Dockerfile`**
```dockerfile
# Dockerfile
FROM python:3.11-slim
ENV PYTHONDONTWRITEBYTECODE=1 PYTHONUNBUFFERED=1
WORKDIR /app
COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
# Kolekcja statyków (opcjonalnie w prod)
# RUN DJANGO_SETTINGS_MODULE=Wypoczynkowy.settings python -m django collectstatic --noinput
CMD ["python", "-m", "django", "runserver", "0.0.0.0:8000"]
```

**`docker-compose.yml`**
```yaml
version: "3.9"
services:
  web:
    build: .
    ports:
      - "8000:8000"
    env_file: .env
    depends_on:
      - db
  db:
    image: postgres:16
    environment:
      POSTGRES_DB: ${DATABASE_NAME:-wypoczynkowy_db}
      POSTGRES_USER: ${DATABASE_USERNAME:-postgres}
      POSTGRES_PASSWORD: ${DATABASE_PASSWORD:-postgres123}
    volumes:
      - pgdata:/var/lib/postgresql/data
    ports:
      - "5433:5432"
volumes:
  pgdata: {}
```

Uruchom:
```bash
docker compose up --build
```

## Zmienne środowiskowe (.env)

Poniżej **przykład**. Nie commituj haseł do repozytorium.

```dotenv
# Django
DJANGO_SECRET_KEY=zmien_to
# Uwaga: w kodzie DEBUG jest rzutowane bool(os.environ.get("DEBUG", 0)).
# Pozostaw puste lub usuń, aby był False; ustaw dowolny niepusty string, aby był True.
DEBUG=1
DJANGO_ALLOWED_HOSTS=127.0.0.1,0.0.0.0,localhost

# Baza (PostgreSQL)
DATABASE_NAME=wypoczynkowy_db
DATABASE_USERNAME=postgres
DATABASE_PASSWORD=postgres123
DATABASE_HOST=localhost   # lub 'db' przy Docker Compose
DATABASE_PORT=5432

# E-mail (SMTP) — np. Mailtrap
EMAIL_HOST=sandbox.smtp.mailtrap.io
EMAIL_PORT=587
EMAIL_HOST_USER=twoj_login
EMAIL_HOST_PASSWORD=twoje_haslo
CONTACT_EMAIL=kontakt@example.com
EMAIL_USE_TLS=True

# reCAPTCHA
RECAPTCHA_PUBLIC_KEY=twoj_public_key
RECAPTCHA_PRIVATE_KEY=twoj_private_key

# Media/static (opcjonalnie w prod)
# STATIC_ROOT=/app/staticfiles
# MEDIA_ROOT=/app/media
```

## URL-e / nawigacja

Zdefiniowane w `Wypoczynkowy/urls.py` oraz `Aplikacja_web/urls.py`:
- `/` – strona główna
- `/login/`, `/register/`, `/logout/`, `/dashboard/`
- `/ogloszenia/` – lista ogłoszeń
- `/ogloszenie/<id>/` – szczegóły ogłoszenia
- `/dodaj/` – dodawanie ogłoszenia
- `/wydarzenia/`, `/wydarzenie/<id>/`
- `/imprezy/`, `/impreza/<id>/`
- `/archiwum-ogloszen/`
- `/kontakt/`, `/formularz/`, `/success/`
- `/oserwisie/`, `/cennik/`, `/infoipomoc/`

---

### Notatka końcowa

Repozytorium stanowi solidną bazę do dalszej rozbudowy (np. płatne wyróżnienia ogłoszeń, kalendarz i subskrypcje wydarzeń, REST API pod SPA). Konfiguracja celowo pozostaje prosta, aby ułatwić szybkie uruchomienie lokalne i pracę w zespole.
