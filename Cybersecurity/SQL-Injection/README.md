# SQL Injection Lab — von der Schwachstelle zum Fix

Ein kleines, bewusst verwundbares Login-System, das zeigt, **wie eine SQL-Injection
funktioniert** und **wie man sie mit einer einzigen Codeänderung stoppt**.
Gebaut als Lern- und Portfolio-Projekt.

> ⚠️ **Nur für die lokale Lab-Umgebung.** Die verwundbare App ist absichtlich
> unsicher. Niemals öffentlich erreichbar machen und niemals gegen fremde Systeme
> testen — Angriffe ausschließlich gegen dieses eigene, lokale Lab.

---

## Inhaltsverzeichnis

1. [Was ist das Projekt?](#1-was-ist-das-projekt)
2. [Wie Docker funktioniert](#2-wie-docker-funktioniert)
3. [Was steckt in der Zip?](#3-was-steckt-in-der-zip)
4. [Setup — in 3 Schritten starten](#4-setup--in-3-schritten-starten)
5. [Wie die Datenbank aufgesetzt wird](#5-wie-die-datenbank-aufgesetzt-wird)
6. [Wie die SQL-Injection funktioniert (die Essenz)](#6-wie-die-sql-injection-funktioniert-die-essenz)
7. [Angriffe zum Nachstellen](#7-angriffe-zum-nachstellen)
8. [Der Fix und warum er wirkt](#8-der-fix-und-warum-er-wirkt)
9. [Defense in Depth](#9-defense-in-depth)
10. [Screenshots](#10-screenshots)
11. [Einsatz von KI in diesem Projekt](#11-einsatz-von-ki-in-diesem-projekt)

---

## 1. Was ist das Projekt?

Zwei Versionen derselben Login-App laufen parallel:

| Version    | URL                     | Beschreibung                                |
|------------|-------------------------|---------------------------------------------|
| Verwundbar | http://localhost:5000   | Baut Eingaben direkt in den SQL-String ein  |
| Sicher     | http://localhost:5001   | Parameterisierte Query + bcrypt-Hashing     |

Man greift die verwundbare Version an, sieht den Effekt, und stellt denselben
Angriff gegen die sichere Version — wo er abprallt. Vorher/Nachher zum Anfassen.

**Stack:** Python · Flask (Web-Framework) · SQLite (Datenbank) · Docker

---

## 2. Wie Docker funktioniert

Docker packt eine Anwendung **samt allem, was sie zum Laufen braucht** (hier:
Python, Flask, die Datenbank), in ein abgeschottetes Paket — einen **Container**.
Dadurch läuft die App überall identisch und nicht mehr nur „auf meinem Rechner".

Drei Begriffe, die im Projekt vorkommen:

- **Image** — der Bauplan/die Vorlage. Wird aus dem `Dockerfile` gebaut.
- **Container** — eine laufende Instanz eines Images (das, was aktiv ist).
- **docker compose** — startet mehrere Container zusammen anhand einer
  Konfigurationsdatei (`docker-compose.yml`). Ein Befehl, alles läuft.

Zusatz-Nutzen für dieses Projekt: Der bewusst unsichere Code sitzt in seiner
eigenen Box, abgeschottet vom restlichen System — sicher zum Experimentieren.

In diesem Projekt startet `docker compose up` drei Container:
1. **seed** — baut einmalig die Datenbank und beendet sich dann
2. **vulnerable** — die verwundbare App (Port 5000)
3. **secure** — die sichere App (Port 5001)

---

## 3. Was steckt in der Zip?

```
sqli-lab/
├── docker-compose.yml        # startet alle Container zusammen
├── README.md                 # diese Datei
├── .gitignore
├── screenshots/              # hier kommen deine Fotos rein (siehe Abschnitt 10)
└── app/
    ├── Dockerfile            # Bauplan für das Image
    ├── requirements.txt      # Python-Pakete (flask, bcrypt)
    ├── seed.py               # erstellt die Demo-Datenbank
    ├── app_vulnerable.py     # die Schwachstelle
    ├── app_secure.py         # der Fix
    └── templates/
        └── login.html        # das Login-Formular (für beide Versionen)
```

---

## 4. Setup — in 3 Schritten starten

**Voraussetzung:** Docker + Docker Compose.
Auf **Omarchy** aktivierst du das über *Super + Alt + Space →
Install > Development > Docker*.

```bash
# 1. Zip entpacken und reingehen
unzip sqli-lab.zip && cd sqli-lab

# 2. Alles bauen und starten
sudo docker compose up --build
#    (bei neueren Omarchy-Versionen mit sudo)

# 3. Im Browser öffnen
#    Verwundbar: http://localhost:5000
#    Sicher:     http://localhost:5001
```

Beenden: im Terminal `Strg + C`, danach `sudo docker compose down`.

---

## 5. Wie die Datenbank aufgesetzt wird

Die Datenbank ist eine einzelne **SQLite**-Datei — kein separater Datenbankserver
nötig, SQLite ist in Python schon eingebaut.

Das Skript `seed.py` baut sie beim Start automatisch (Container **seed**):

1. Legt eine Tabelle **`users`** an (id, username, password).
2. Legt eine Tabelle **`secrets`** an (id, label, value) — diese dient dazu,
   den Datenabfluss per UNION-Angriff zu demonstrieren.
3. Füllt beide mit Beispieldaten (u. a. `admin` und ein simulierter API-Key).

Wichtiger Punkt zum Verständnis:
- In der **verwundbaren** DB (`lab.db`) stehen Passwörter im **Klartext** —
  bewusst simpel gehalten, damit der Fokus auf der Injection liegt.
- Die **sichere** App baut sich eine **eigene** DB (`lab_secure.db`), in der
  Passwörter mit **bcrypt gehasht** gespeichert werden (siehe `app_secure.py`,
  Funktion `build_secure_db`).

Beide DBs liegen in einem Docker-Volume namens `dbdata` und überstehen so einen
Neustart der Container.

---

## 6. Wie die SQL-Injection funktioniert (die Essenz)

Der Kern in **einem Satz**: Die verwundbare App vermischt **Code (SQL)** und
**Daten (Eingabe)** in einem einzigen Textstring — dadurch kann der Nutzer über
die Eingabe die Struktur der Abfrage verändern.

Die verwundbare Query (`app_vulnerable.py`):

```python
query = f"SELECT id, username FROM users WHERE username = '{user}' AND password = '{pw}'"
```

Gibt man als Username `admin'--` ein, entsteht:

```sql
SELECT id, username FROM users WHERE username = 'admin'-- ' AND password = '...'
```

- Das `'` **schließt** den Username-String direkt nach `admin`.
- Das `--` macht den **Rest der Zeile zu einem Kommentar** — die komplette
  Passwortprüfung wird also ignoriert.
- Ergebnis: Login als `admin`, ganz ohne Passwort.

Merksatz: **Ohne führendes `'` schreibst du nur *im* Text. Mit `'` schreibst du
*SQL*.** Genau dieser Ausbruch aus dem String ist der Kern der Schwachstelle.

---

## 7. Angriffe zum Nachstellen

Alle in der **verwundbaren** Version (Port 5000). Passwortfeld: beliebig.

**1. Auth-Bypass per Kommentar (Hauptbeispiel)**
Username: `admin'--`
→ Login als admin, Passwortprüfung wird wegkommentiert.

**2. Auth-Bypass ohne gültigen Namen (Variante)**
Username: `' OR '1'='1' -- `
→ Bedingung immer wahr, es matchen alle Nutzer.

**3. Datenabfluss per UNION (Variante)**
Username: `' UNION SELECT label, value FROM secrets -- `
→ Zieht Daten aus einer *anderen* Tabelle (`secrets`) ins Ergebnis.

Hinweise: hinter `--` ein **Leerzeichen** lassen, und ein **gerades** `'`
tippen (kein typografisches `’`). Die App zeigt die ausgeführte Query mit an —
ideal für den Screenshot.

---

## 8. Der Fix und warum er wirkt

In `app_secure.py`:

```python
query = "SELECT id, username, pw_hash FROM users WHERE username = ?"
row = con.execute(query, (user,)).fetchone()
```

Der Platzhalter `?` (**parameterisierte Query / Prepared Statement**) trennt
**Code von Daten**: Die Struktur der Abfrage steht schon *fest*, bevor die
Eingabe überhaupt dazukommt. Die Eingabe wird als **reiner Wert** übergeben.

Gibt man jetzt `admin'--` ein, sucht die Datenbank wörtlich nach einem Nutzer,
der buchstäblich `admin'--` heißt — den gibt es nicht. Das `'` bricht nichts
mehr aus, das `--` kommentiert nichts weg. Der Angriff prallt ab.

---

## 9. Defense in Depth

Parameterisierte Queries sind die Hauptlösung. Zusätzlich als Best Practice
(teils in diesem Lab umgesetzt):

- **Passwort-Hashing** mit bcrypt statt Klartext-Vergleich (in `app_secure.py`)
- **Least Privilege** — der DB-Nutzer sollte nur die nötigen Rechte haben
- **Input-Validierung** als zusätzliche Schicht (nicht als Ersatz!)
- **Logging/Monitoring**, um Angriffsversuche früh zu erkennen

**Impact eines echten Angriffs:** Anmeldung ohne gültige Zugangsdaten
(Account-Übernahme), Auslesen fremder Tabellen (Datenabfluss), je nach Rechten
sogar Verändern oder Löschen von Daten.

---

## 10. Screenshots

Lege deine Screenshots im Ordner `screenshots/` ab. Für das Portfolio empfehlen
sich zwei Bilder:

1. **Angriff erfolgreich** — Port 5000, Username `admin'--`, mit sichtbarer
   „Login OK als admin"-Meldung und der manipulierten Query darunter.
2. **Angriff prallt ab** — Port 5001, derselbe Username, „Login fehlgeschlagen".

Dieses Vorher/Nachher ist das Herzstück für einen LinkedIn-Post.

Einbinden im README (Beispiel):

```markdown
![Angriff erfolgreich](screenshots/attack.png)
![Angriff prallt ab](screenshots/secure.png)
```

---

## 11. Einsatz von KI in diesem Projekt

Transparenzhinweis: Das Grundgerüst dieses Labs (Code, Docker-Setup und diese
Dokumentation) wurde mit Unterstützung eines KI-Assistenten erstellt. Die
Angriffe wurden anschließend selbst in der lokalen Umgebung nachvollzogen und
verifiziert. KI diente hier als Werkzeug zum schnellen Aufsetzen und Erklären —
das Verständnis der Schwachstelle und des Fixes steht im Mittelpunkt.