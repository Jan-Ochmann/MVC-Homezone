# MVC-Homezone
A lightweight educational MVC framework in PHP (Smarty, Routing, Controllers, Models, PDO). Built to learn clean architecture and modern PHP project structure.

Ein **leichtgewichtiges PHP-MVC-Framework**, das ich entwickelt habe, um moderne Webarchitektur von Grund auf zu verstehen und in eigenen Projekten einzusetzen.  
Der Fokus liegt auf **klarem Routing**, **sauberer Ordnerstruktur**, **Smarty-Templating**, einer **PDO-Datenbankschicht** sowie **automatisierter Datenbankvisualisierung (Mermaid / DBVision)**.

---
<img width="1258" height="857" alt="image" src="https://github.com/user-attachments/assets/dfced039-d484-436c-8dd1-3429b70a12ac" />
<img width="1007" height="925" alt="image" src="https://github.com/user-attachments/assets/2713ffa3-5362-4176-a298-950c00eb6744" />


---


## 🚀 Features

- **Custom Router**  
Eigene Routing-Schicht mit Unterstützung für GET/POST, dynamischen URL-Parametern und automatischem Controller-Mapping.
Die Request-Verarbeitung ist vollständig entkoppelt von Controllern und Views.

- **Controller & Actions**  
Konsequente MVC-Trennung: Controller enthalten nur Logik, Views kümmern sich ausschließlich um die Darstellung,
Models um Datenzugriff und Validierung.

- **Smarty Templates** 
Leichte, strukturierte UI-Layer mit wiederverwendbaren Komponenten, Layouts und Partials. Saubere Trennung von HTML und PHP-Logik.

- **PDO Database Layer**  
Abstrakte Datenbankschicht mit vorbereiteten Statements, sicherer Query-Ausführung und optionaler Metadaten-Analyse (Tabellen, Primärschlüssel usw.).

- **DBVision Integration**  
Automatische Analyse der aktiven Datenbank über INFORMATION_SCHEMA und Erzeugung eines vollständig dynamischen Mermaid-ER-Diagramms direkt im Browser,
anhand der Primary & Foreign Keys. 

- **Modulare Struktur**  
Jede Komponente ist separat erweiterbar: zusätzliche Controller, Views, Services oder Tools (z. B. PDO-Playground, DBVision) lassen sich ohne Änderungen am Kern integrieren.

---

## 🏗️ Architekturüberblick

```
flowchart LR
    A[Entry Point index.php] --> B[Router]
    B --> C[Controller]
    C --> D[Model (PDO)]
    C --> E[Smarty View]
    D --> F[(Datenbank)]
```

```text

MVC-Homezone/
│
├── public/                           # Öffentlicher Einstiegspunkt
│   ├── css/
│   ├── js/
│   ├── images/                       # Hintergrundbilder für die Website
│   ├── templates_c/                  # Smarty Cache
│   ├── .htaccess
│   └── index.php                     # Startpunkt lädt Init.php
│
├── src/
│   ├── AbstractClass/
│   │   ├── Controller/
│   │   │   └── AContoller.php        # ← Snippet 1: Basis-Controller
│   │   └── Library/
│   │       ├── AControllerFactory.php# ← Snippet 2: Routing/Factory
│   │       └── ARequestHandler.php   # ← Snippet 2: Routing
│   │
│   ├── Controllers/
│   │   └── Pdoplay/
│   │       ├── Controller.php        # ← Snippet 3: PDO Playground
│   │       └── DBVisionController.php# ← Snippet 4: DBVision Logik
│   │
│   ├── Library/
│   │   ├── ControllerFactory.php     # Projekt-spezifische Factory
│   │   └── RequestHandler.php        # Projekt-spezifischer Router
│   │
│   ├── Services/
│   │   └── DbConnect/
│   │       └── DBVision.php          # ← Snippet 4: Mermaid Generator
│   │
│   ├── Views/
│   │   └── Pdoplay/
│   │       ├── pdoplay.tpl.html      # PDO Playground Template
│   │       ├── ViewPdoplay.php       # View Layer Select Abfrage
│   │       ├── dbvision.tpl.html     # ← Snippet 5: DBVision Template
│   │       └── DBVisionView.php      # ← Snippet 5: View Layer
│   │
│   └── Init.php                      # Bootstrap / Smarty Setup
│
├── template/                         # Layouts, Head, Foot, Navigation
│
├── vendor/                           # Composer Dependencies
│
├── composer.json
└── composer.lock

```

---

## ⚙️ Installation & Setup

Um das Projekt lokal (z. B. in XAMPP, Laragon oder einer Linux-VM) auszuführen:

```bash
git clone https://github.com/Jan-Ochmann/MVC-Homezone.git
cd MVC-Homezone
composer install
```

1. .env anlegen
```
DB_HOST=localhost
DB_NAME=demo
DB_USER=root
DB_PASS=password
```
2. Webserver konfigurieren

Für die Ausführung wird ein Webserver benötigt (Apache oder nginx), der folgende Anforderungen erfüllt:
- Document-Root auf /public setzen
- mod_rewrite (Apache) oder passendes Routing in nginx aktivieren

3. Anwendung starten

http://localhost/

---

🎯 Motivation & Lernziele

MVC-Homezone ist ein bewusst schlank gehaltenes Lernframework, das folgende Ziele unterstützt:

Verständnis für den Aufbau moderner MVC-Architekturen

Trennung von Routing, Controller-Logik und Views

Saubere Datenbankoperationen mit PDO und Prepared Statements

Nutzung von Smarty als Template-Engine

Automatisierte Visualisierung von Datenbanken (Mermaid / DBVision)

Aufbau von Dokumentation & Architekturverständnis

Vorbereitung auf professionelle Webprojekte

Das Projekt richtet sich nicht an Produktivbetrieb, sondern dient als pädagogische Mini-Architektur, um Systemverständnis zu vertiefen.

---

## Code Snippets mit Erklärung

## 1) Base Controller (AContoller)
Der abstrakte Basis-Controller kapselt den gemeinsamen Ablauf: indexAction() ruft automatisch die passende View auf. 
Die View-Klasse wird konventionsbasiert aus dem Controller-Namen zusammengesetzt (Namespace → Views\<Name>\View<Name>).

```
<?php

namespace MVC-Homezone\AbstractClass\Controller;

use MVC-Homezone\Interfaces\Controller\IController;
use MVC-Homezone\Library\RequestHandler;

abstract class AContoller implements IController
{
    public function indexAction(): void
    {
        $this->getView();
    }

    final public function __construct(protected RequestHandler $requestHandler) {}

    /**
     * Konventionsbasierte View-Ermittlung:
     * Namespace → Views\<ControllerName>\View<ControllerName>
     */

    protected function getView(): void
    {
        $controllerName = $this->requestHandler->controllerName;

        $parts     = explode('\\', __NAMESPACE__);
        $className = $parts[0] . '\\Views\\' . $controllerName . '\\View' . $controllerName;
        $method    = 'get' . ucfirst($controllerName);

        $className::$method();
    }
}

```

## 2) Request Pipeline ARequestHandler & AControllerFactory

Die Request-Verarbeitung ist in zwei abstrakte Klassen aufgeteilt:
ARequestHandler zerschneidet die URI und extrahiert Controller/Action, AControllerFactory baut
daraus den vollqualifizierten Klassennamen und prüft, ob Controller + Action existieren.
Konkrete Implementierungen hängen vom Projekt ab, die Architektur bleibt gleich.

ARequestHandler
```
<?php
// src/AbstractClass/Library/ARequestHandler.php

namespace MVC-Homezone\AbstractClass\Library;

use MVC-Homezone\Interfaces\Library\IRequestHandler;

abstract class ARequestHandler implements IRequestHandler
{
    /**
     * Zerlegt die aktuelle URI in Teile (z.B. /Pdoplay/index).
     */
    abstract protected function cutUriParts(): array;

    /**
     * Validiert und verarbeitet die gefundenen URI-Teile.
     */
    abstract protected function checkIfUriPartsExists($matches): void;
}

```

AControllerFactory
```
<?php
// src/AbstractClass/Library/AControllerFactory.php

namespace mvc_eight\AbstractClass\Library;

use mvc_eight\Interfaces\Library\IControllerFactory;

abstract class AControllerFactory implements IControllerFactory
{
    protected string $controllerName;
    protected object $controllerObject;

    abstract protected function buildControllerNameWithNamespace(): void;
    abstract protected function loadController(): void;
    abstract protected function buildControllerPath(): void;
    abstract protected function checkIfControllerExists(): void;
    abstract protected function checkIfActionExists(): void;
}

```
## 3) PDO Playground - SELECT Only + Inline-Edit

PDO Playground Controller
Der Pdoplay\Controller implementiert einen sicheren SQL-Playground:
Nur SELECT-Abfragen sind erlaubt, gleichzeitig werden bei SELECT * FROM <tabelle> Primärschlüssel erkannt
und darüber Inline-Insert/Update/Delete per vorbereiteten PDO-Statements ermöglicht. 
Alle Daten für das Template werden zentral in der Session abgelegt.

Controller (gekürzte Version)
```
<?php

namespace MVC-Homezone\Controllers\Pdoplay;

use MVC-Homezone\AbstractClass\Controller\AContoller;
use MVC-Homezone\Services\DbQuery\PdoQuery;
use MVC-Homezone\Views\Pdoplay\ViewPdoplay;
use PDO;

class Controller extends AContoller
{
    protected function getView(): void
    {
        $defaultDb  = 'Marvel_Lexikon'; // 
        $defaultSql = "SELECT * FROM character_name LIMIT 50";

        // 1. Aktive DB bestimmen (POST → Session → Default)
        if (!empty($_POST['db_name'])) {
            $dbName = $_POST['db_name'];
        } elseif (!empty($_SESSION['pdoplay']['db_name'])) {
            $dbName = $_SESSION['pdoplay']['db_name'];
        } else {
            $dbName = $defaultDb;
        }

        // 2. SQL bestimmen (POST → Session → Default)
        $sql = $_POST['sql'] ?? ($_SESSION['pdoplay']['query'] ?? $defaultSql);

        $rows = $columns = [];
        $error = $info = null;
        $baseTable = $pkColumn = null;

        // Basistabelle aus "SELECT * FROM <tabelle>" auslesen
        if (preg_match('/^\s*select\s+\*\s+from\s+`?([A-Za-z0-9_]+)`?/i', $sql, $m)) {
            $baseTable = $m[1];

            try {
                $metaPk = new PdoQuery();
                $metaPk->setUseDbName($dbName);
                $pkColumn = $metaPk->getPrimaryKeyColumn($baseTable);
            } catch (\Throwable $e) {
                $pkColumn = null;
            }
        }

        // 2b. Insert / Update / Delete nur über vorbereitete Statements
        // (aus Platzgründen hier nur angedeutet)
        if ($baseTable && $pkColumn && !empty($_POST)) {
            $dbEdit = new PdoQuery();
            $dbEdit->setUseDbName($dbName);

            // Delete, Update, Insert werden über PK gesteuert
            // und laufen ausschließlich über parametrisiertes SQL.
            // ...
        }

        // 3. SELECT nur erlauben, wenn Abfrage wirklich mit SELECT beginnt
        if (array_key_exists('sql', $_POST)) {
            try {
                if (!preg_match('/^\s*select/i', $sql)) {
                    throw new \RuntimeException('Nur SELECT-Abfragen sind erlaubt.');
                }

                $db = new PdoQuery();
                $db->setUseDbName($dbName);
                $db->setQuery($sql);

                $rows = $db->getFetchAll(PDO::FETCH_ASSOC);
                if (!empty($rows)) {
                    $columns = array_keys($rows[0]);
                }

                $info = 'Abfrage erfolgreich ausgeführt.';
            } catch (\Throwable $e) {
                $error = $e->getMessage();
            }
        }

        // 4. Tabellenliste + verfügbaren Datenbanken laden
        try {
            $meta = new PdoQuery();
            $meta->setUseDbName($dbName);
            $tables = $meta->getShowTables();
        } catch (\Throwable $e) {
            $tables = [];
        }

        try {
            $dbListQuery = new PdoQuery();
            $dbList = $dbListQuery->getShowDatabases();
        } catch (\Throwable $e) {
            $dbList = [$defaultDb];
        }

        // 5. Message/Fehler für Template aufbereiten
        $message = $error ?: $info;
        $isError = $error !== null;

        // 6. Session-Namespace 'pdoplay' füllen
        $_SESSION['pdoplay'] = [
            'db_name'    => $dbName,
            'db_list'    => $dbList,
            'query'      => $sql,
            'columns'    => $columns,
            'rows'       => $rows,
            'tables'     => $tables,
            'base_table' => $baseTable,
            'pk_column'  => $pkColumn,
            'message'    => $message,
            'is_error'   => $isError,
        ];

        // 7. View rendern
        ViewPdoplay::getPdoplay();
    }

    // Thin Controller für DBVision
    public function dbvisionAction(): void
    {
        $controller = new DBVisionController();
        $controller->show();
    }
}

```

## 4) DBVision automatisches ER-Diagramm mit Mermaid
Der DBVisionController liest die aktive Datenbankverbindung aus der Session, baut eine PDO-Connection auf
und übergibt sie an den DBVision-Service. Dieser erzeugt dynamisch ein Mermaid-Diagramm aus INFORMATION_SCHEMA,
das in der UI als ER-Diagramm gerendert wird.

```
<?php

namespace MVC-Homezone\Controllers\Pdoplay;

use MVC-Homezone\Services\DbConnect\DBVision;
use MVC-Homezone\Views\Pdoplay\DBVisionView;
use PDO;

class DBVisionController
{
    public function show(): void
    {
        $dbName = $_SESSION['pdoplay']['db_name'] ?? null;

        if (!$dbName) {
            die('DB Vision: Kein Datenbankname in der Session gefunden.');
        }

        $host = 'localhost';
        $user = 'someuser';
        $pass = 'somepass';
        $dsn  = "mysql:host={$host};dbname={$dbName};charset=utf8mb4";

        $pdo = new PDO($dsn, $user, $pass, [
            PDO::ATTR_ERRMODE            => PDO::ERRMODE_EXCEPTION,
            PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
        ]);

        // Mermaid-ER-Diagramm für die aktive DB erzeugen
        $visionService = new DBVision();
        $mermaid       = $visionService->buildMermaidForDatabase($pdo, $dbName);

        DBVisionView::show($dbName, $mermaid);
    }
}

```

## 5) DBVision View + Template - Visual Layer
Die View-Schicht trennt Ausgabe und Logik: DBVisionView übergibt nur die Daten an Smarty,
das Template dbvision.tpl.html kümmert sich um die Darstellung – inklusive Cyberpunk-Look,
ausklappbarer Hinweisbox und eingebettetem Mermaid-Block.

```
<?php
// src/Views/Pdoplay/DBVisionView.php

declare(strict_types=1);

namespace MVC-Homezone\Views\Pdoplay;

final class DBVisionView
{
    public static function show(string $dbName, string $mermaid): void
    {
        $_SESSION['smarty']->assign('dbName', $dbName);
        $_SESSION['smarty']->assign('mermaid', $mermaid);

        $_SESSION['smarty']->display(__DIR__ . '/dbvision.tpl.html');
    }
}

```
dbvision.tpl.html (Ausschnitt)
```
{* template: dbvision.tpl.html (Ausschnitt) *}

<div class="container my-4 dbv-page-bg">
    <div class="card shadow-sm border-0 glass-card">
        <div class="card-body">

            <!-- Kopfbereich -->
            <div class="d-flex justify-content-between align-items-center mb-3 flex-wrap gap-2">
                <div>
                    <div class="dbv-header-kicker">PDO Playground – Visual Layer</div>
                    <div class="dbv-header-title">DB Vision</div>
                    <p class="dbv-header-sub">
                        Relationales Modell für Datenbank:
                        <strong>{$dbName|escape}</strong>
                    </p>
                </div>
                <div>
                    <a href="/Pdoplay" class="btn btn-sm btn-cyber-primary">
                        Zurück zum PDO Playground
                    </a>
                </div>
            </div>

            <!-- Hinweisbox mit Collapse -->
            <div class="dbv-hint">
                <button class="btn btn-sm"
                        style="background:none; border:none; color:#1af2ff; font-weight:600; padding:0;"
                        type="button"
                        data-bs-toggle="collapse"
                        data-bs-target="#dbvisionHintBox">
                    Hinweis zur Darstellung ▾
                </button>

                <div class="collapse mt-2" id="dbvisionHintBox">
                    <p class="dbv-hint-text">
                        Das Diagramm wird automatisch aus <code>INFORMATION_SCHEMA</code> erzeugt.
                        Angezeigt werden Tabellen, <strong>PK</strong> und
                        <strong>FK-Beziehungen</strong> (Pfeile von FK&nbsp;→&nbsp;PK).
                    </p>
                </div>
            </div>

            <!-- Mermaid-Block -->
            <div class="dbv-mermaid-card">
                <pre class="mermaid">{$mermaid nofilter}</pre>
            </div>
        </div>
    </div>
</div>

```









