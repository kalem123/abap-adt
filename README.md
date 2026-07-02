# abap-adt: Your Gateway to ABAP Development Tools (ADT)

This project provides a server that allows you to interact with SAP ABAP systems using a standardized tool protocol. Think of it as a bridge that lets tools like [Cline](https://marketplace.visualstudio.com/items?itemName=saoudrizwan.claude-dev) (a VS Code extension) or Claude Desktop talk to your ABAP system and retrieve information like source code, table structures, and more. It's like having a remote control for your ABAP development environment!

## Schnellstart: Erste Inbetriebnahme auf einem neuen Rechner

Diese Kurzanleitung ist für den Fall gedacht, dass du diesen Ordner (`abap-adt`) frisch per Copy-Paste auf einen neuen Rechner kopiert hast und ihn zum ersten Mal in Betrieb nehmen willst.

### Schritt 1: Node.js installieren (falls noch nicht vorhanden)

Dieses Projekt braucht **Node.js**, um überhaupt laufen zu können – ohne geht nichts.

1.  Prüfen, ob Node.js schon installiert ist: Terminal/PowerShell öffnen und eingeben:
    ```bash
    node -v
    npm -v
    ```
    Kommen zwei Versionsnummern zurück (z.B. `v20.11.0` und `10.2.4`), ist Node.js schon da – weiter mit Schritt 2.

2.  Falls eine Fehlermeldung kommt ("node wird nicht erkannt" o.ä.): Node.js ist noch nicht installiert.
    *   Gehe auf [nodejs.org](https://nodejs.org/en/download/)
    *   Lade die **LTS-Version** herunter (das ist die empfohlene, stabile Version – NICHT die "Current"-Version)
    *   Installer ausführen, dabei einfach überall "Weiter"/"Next" klicken (die Standardeinstellungen reichen, npm wird automatisch mitinstalliert)
    *   Nach der Installation: Terminal/PowerShell **neu öffnen** (wichtig, sonst wird die PATH-Änderung nicht erkannt) und `node -v` / `npm -v` nochmal testen

### Schritt 2: Terminal im `abap-adt`-Ordner öffnen

Im Windows-Explorer in den Ordner `abap-adt` navigieren, dann z.B. Umschalt+Rechtsklick auf eine freie Fläche → "PowerShell-Fenster hier öffnen" (oder "Terminal hier öffnen", je nach Windows-Version).

### Schritt 3: Abhängigkeiten installieren

```bash
npm install
```
Das lädt alle benötigten Bibliotheken herunter (kann ein paar Minuten dauern). Nur beim allerersten Einrichten nötig bzw. wenn sich `package.json` mal ändert.

### Schritt 4: Projekt bauen

```bash
npm run build
```
Das übersetzt den Quellcode (TypeScript) in ausführbaren JavaScript-Code (Ordner `dist/`). Das musst du nach jeder Code-Änderung erneut ausführen, damit die Änderung auch wirkt.

### Schritt 5: `.env`-Datei mit den SAP-Zugangsdaten anlegen

**Ganz wichtig – ohne diesen Schritt startet der Server nicht:**

1.  Datei `.env.example` im Ordner kopieren und die Kopie in `.env` umbenennen (ohne Dateiendung, nur `.env`)
2.  `.env` mit einem Texteditor öffnen (Notepad, VS Code, …) und die Platzhalter durch die echten Zugangsdaten des Ziel-SAP-Systems ersetzen:
    ```
    SAP_URL=https://euer-sap-system.com:8000
    SAP_USERNAME=euer_benutzername
    SAP_PASSWORD=euer_passwort
    SAP_CLIENT=100
    ```
3.  Speichern. Diese Datei **niemals** weitergeben oder in Git einchecken – sie enthält ein Klartext-Passwort.

### Schritt 6: Kurzer Funktionstest

```bash
npm run start
```
Wenn im Terminal Meldungen erscheinen, dass der Server läuft (kein rotes Fehler-Stacktrace), funktioniert das Grundsetup. Mit `Strg+C` wieder beenden – im Normalbetrieb wird der Server nicht so manuell gestartet, sondern automatisch von Cline/Claude Desktop verbunden (siehe Abschnitt 4 weiter unten).

**Kurz zusammengefasst, die vier Befehle, die bei jedem frischen Setup nötig sind:**
```bash
npm install
npm run build
# .env-Datei manuell anlegen/befüllen (siehe Schritt 5)
npm run start   # nur zum Testen, siehe oben
```

---

## Neues Setup bei einem anderen Kunden (neuer Rechner + neues SAP-System)

Für ein komplett neues Setup bei einem anderen Kunden brauchst du zwei getrennte Teile: Client-Seite (Rechner) und SAP-Seite (System). Nutze dafür das "saubere" Repo `abap-adt` (ohne MCP-Branding), nicht `mcp-abap-adt`.

### 1. Client-Seite (neuer Rechner)
- Node.js installieren (falls nicht vorhanden)
- `git clone https://github.com/kalem123/abap-adt.git`
- `npm install` und `npm run build`
- `.env` aus `.env.example` neu anlegen mit den Zugangsdaten des neuen Kunden-Systems (URL, User, Client, Passwort/Auth, `TLS_REJECT_UNAUTHORIZED=0` falls Zertifikat selbstsigniert)
- MCP-Server in Claude Code registrieren (`.mcp.json` bzw. `claude mcp add`, zeigt auf den lokalen `dist`-Ordner)

### 2. SAP-Seite (im Zielsystem, einmalig)
- Klasse `ZCL_TABLECONTENT` in SE24/ADT anlegen und aktivieren (Quelltext liegt als `ZCL_TABLECONTENT.txt` im Repo)
- SICF-Knoten `/z_util/z_tablecontent` anlegen (zwei Ebenen: `z_util` als Container, `z_tablecontent` mit Handler-Klasse `ZCL_TABLECONTENT`), Service aktivieren
- Kurzer Test direkt im Browser: `https://<host>:<port>/z_util/z_tablecontent/T000?maxRows=5` sollte JSON liefern

### 3. Verifizieren
- Claude Code neu starten (frischer Node-Prozess für den MCP-Server)
- Testabfrage einer Standardtabelle (`T000`) über `GetTableContents`

Wichtig: Berechtigungen des SAP-Users beim Kunden müssen `S_TABU_DIS` für die relevante Berechtigungsgruppe(n) enthalten, sonst blockt der `AUTHORITY-CHECK` in der Klasse.

---

This guide covers:

1.  **Prerequisites:** What you need before you start.
2.  **Setup:**  Getting everything up and running.
3.  **Running the Server:**  Starting the server in different modes.
4.  **Integrating with Cline:** Connecting this server to the Cline VS Code extension.
5.  **Troubleshooting:**  Common problems and solutions.
6.  **Available Tools:**  A list of the commands you can use.

## 1. Prerequisites

Before you begin, you'll need a few things:

*   **An SAP ABAP System:**  This server connects to an existing ABAP system.  You'll need:
    *   The system's URL (e.g., `https://my-sap-system.com:8000`)
    *   A valid username and password for that system.
    *   The SAP client number (e.g., `100`).
    *   Ensure that your SAP system allows connections via ADT (ABAP Development Tools). This usually involves making sure the necessary services are activated in transaction `SICF`.  Your basis administrator can help with this. Specifically, you will need the following services to be active:
        * `/sap/bc/adt`
    *   For the `GetTableContents` Tool, you will need the implementation of a custom service `/z_util/z_tablecontent`. You can follow this guide [here](https://community.sap.com/t5/application-development-blog-posts/how-to-use-rfc-read-table-from-javascript-via-webservice/ba-p/13172358)

*   **Node.js and npm:** Node.js is a JavaScript runtime that lets you run JavaScript code outside of a web browser.  npm (Node Package Manager) is included with Node.js and is used to install packages (libraries of code).
    *   [Download Node.js](https://nodejs.org/en/download/).  **Choose the LTS (Long Term Support) version.**  This is the most stable version. Follow the installation instructions for your operating system.  Make sure to include npm in the installation (it's usually included by default).
    *   **Verify Installation:** After installing Node.js, open a new terminal (command prompt on Windows, Terminal on macOS/Linux) and type:
        ```bash
        node -v
        npm -v
        ```
        You should see version numbers for both Node.js and npm.  If you see an error, Node.js might not be installed correctly, or it might not be in your system's PATH.  (See Troubleshooting below).

*   **VS Code:** Recommended for editing and for running the Cline extension. [Download VS Code](https://code.visualstudio.com/).

## 2. Setup

This folder already contains the project code. To get it running:

1.  **Install Dependencies:**  This downloads all the necessary libraries the project needs.  In the terminal, inside the root directory (this folder), run:
    ```bash
    npm install
    ```
    This might take a few minutes.

2.  **Build the Project:** This compiles the code into an executable format.
    ```bash
    npm run build
    ```

3.  **Create a `.env` file:** This file stores sensitive information like your SAP credentials.  It's *very* important to keep this file secure. A template is provided in `.env.example`.
    1.  Copy `.env.example` to a new file named `.env` (no extension).
    2.  Open the `.env` file in a text editor (like Notepad, VS Code, etc.).
    3.  Fill in the placeholders with your actual SAP system information:
        Important: If your password contains a "#" character, make sure to enclose your password in quotes!
        ```
        SAP_URL=https://your-sap-system.com:8000  # Your SAP system URL
        SAP_USERNAME=your_username              # Your SAP username
        SAP_PASSWORD=your_password              # Your SAP password
        SAP_CLIENT=100                         # Your SAP client
        ```
        **Important:**  Never share your `.env` file with anyone, and never commit it to a Git repository!

## 3. Running the Server

To be fair, you usually don't "run" this server on its own. It is supposed to be integrated into a tool client like Cline or Claude Desktop. But you *can* manually run the server in two main ways:

*   **Standalone Mode:**  This runs the server directly, and it will output messages to the terminal. The server will start and wait for client connections, so potentially rendering it useless except to see if it starts.
*   **Development/Debug Mode:** This runs the server with the Inspector tool. You can open the URL that it outputs in your browser and start playing around.

### 3.1 Standalone Mode

To run the server in standalone mode, use the following command in the terminal (from the root directory):

```bash
npm run start
```

You should see messages in the terminal indicating that the server is running.  It will listen for connections from clients.  The server will keep running until you stop it (usually with Ctrl+C).

### 3.2 Development/Debug Mode (with Inspector)

This mode is useful for debugging.

1.  **Start the server in debug mode:**
    ```bash
    npm run dev
    ```
    This will start the server and output a message with a local URL for the Inspector.
    This is the URL you'll use to open the inspector in your browser.

## 4. Integrating with Cline

Cline is a VS Code extension that connects to tool servers like this one to provide language support. Here's how to connect this ABAP server to Cline:

1.  **Install Cline:** If you haven't already, install the "Cline" extension in VS Code.

2.  **Open Cline Settings:**
    *   Open the VS Code settings (File -> Preferences -> Settings, or Ctrl+,).
    *   Search for "Cline MCP Settings" (this is Cline's own internal naming for its server-connection settings).
    *   Click "Edit in settings.json". This will open the `cline_mcp_settings.json` file (fixed file name, defined by the Cline extension itself).  The full path is usually something like: `C:\Users\username\AppData\Roaming\Code\User\globalStorage\saoudrizwan.claude-dev\settings\cline_mcp_settings.json` (replace `username` with your Windows username).

3.  **Add the Server Configuration:**  You'll need to add an entry to the `servers` array in the `cline_mcp_settings.json` file.  Here's an example:

    ```json
    {
      "mcpServers":
        {
          "abap-adt": {
            "command": "node",
            "args": [
              "C:/PATH_TO/abap-adt/dist/index.js"
            ],
            "disabled": true,
            "autoApprove": []
          }
        // ... other server configurations ...
        }
    }
    ```

    (Note: `mcpServers` is a fixed key name required by Cline's own configuration format — it can't be renamed.)

4.  **Test the Connection:**
    *   Cline should automatically connect to the server.  You will see the server appear in the connected-servers panel (in the Cline extension, you'll find different buttons on the top.)
    *   Ask Cline to get the source code of a program and it should try to use the corresponding tools

## 5. Troubleshooting

*   **`node -v` or `npm -v` gives an error:**
    *   Make sure Node.js is installed correctly.  Try reinstalling it.
    *   Ensure that the Node.js installation directory is in your system's PATH environment variable.  On Windows, you can edit environment variables through the System Properties (search for "environment variables" in the Start Menu).
*   **`npm install` fails:**
    *   Make sure you have an internet connection.
    *   Try deleting the `node_modules` folder and running `npm install` again.
    *   If you're behind a proxy, you might need to configure npm to use the proxy.  Search online for "npm proxy settings".
*   **Cline doesn't connect to the server:**
    *   Double-check the settings in `cline_mcp_settings.json`.  It *must* be the correct, absolute path to the `dist/index.js` file, and use double backslashes or forward slashes consistently on Windows.
    *   Make sure the server is running (use `npm run start` to check).
    *   Restart VS Code.
    *   Alternatively:
    *   Navigate to the root folder of this project in your Explorer, Shift+Right-Click and select "Open Powershell here". (Or open a Powershell and navigate to the folder using `cd C:/PATH_TO/abap-adt/`)
    *   Run "npm install"
    *   Run "npm run build"
    *   Run "npx @modelcontextprotocol/inspector node dist/index.js"
    *   Open your browser at the URL it outputs. Click "connect" on the left side.
    *   Click "Tools" on the top, then click "List Tools"
    *   Click GetProgram and enter "SAPMV45A" or any other Report name as Program Name on the right
    *   Test and see what the output is
*   **SAP connection errors:**
    *   Verify your SAP credentials in the `.env` file.
    *   Ensure that the SAP system is running and accessible from your network.
    *   Make sure that your SAP user has the necessary authorizations to access the ADT services.
    *   Check that the required ADT services are activated in transaction `SICF`.
    *   If you're using self-signed certificates or there is an issue with your SAP system's http config, make sure to set TLS_REJECT_UNAUTHORIZED as described above!

## 6. Available Tools

This server provides the following tools, which can be used through Cline (or any other compatible client):

| Tool Name           | Description                                       | Input Parameters                                                   | Example Usage (in Cline)                                   |
| ------------------- | ------------------------------------------------- | ------------------------------------------------------------------ | ---------------------------------------------------------- |
| `GetProgram`        | Retrieve ABAP program source code.                | `program_name` (string): Name of the ABAP program.                 | `@tool GetProgram program_name=ZMY_PROGRAM`                |
| `GetClass`          | Retrieve ABAP class source code.                  | `class_name` (string): Name of the ABAP class.                     | `@tool GetClass class_name=ZCL_MY_CLASS`                   |
| `GetFunctionGroup`  | Retrieve ABAP Function Group source code.         | `function_group` (string): Name of the function group              | `@tool GetFunctionGroup function_group=ZMY_FUNCTION_GROUP` |
| `GetFunction`       | Retrieve ABAP Function Module source code.        | `function_name` (string), `function_group` (string)                | `@tool GetFunction function_name=ZMY_FUNCTION function_group=ZFG`|
| `GetStructure`      | Retrieve ABAP Structure.                          | `structure_name` (string): Name of the DDIC Structure.             | `@tool GetStructure structure_name=ZMY_STRUCT`             |
| `GetTable`          | Retrieve ABAP table structure.                    | `table_name` (string): Name of the ABAP DB table.                  | `@tool GetTable table_name=ZMY_TABLE`                      |
| `GetTableContents`  | Retrieve contents of an ABAP table.               | `table_name` (string), `max_rows` (number, optional, default 100)  | `@tool GetTableContents table_name=ZMY_TABLE max_rows=50`  |
| `GetPackage`        | Retrieve ABAP package details.                    | `package_name` (string): Name of the ABAP package.                 | `@tool GetPackage package_name=ZMY_PACKAGE`                |
| `GetTypeInfo`       | Retrieve ABAP type information.                   | `type_name` (string): Name of the ABAP type.                       | `@tool GetTypeInfo type_name=ZMY_TYPE`                     |
| `GetInclude`        | Retrieve ABAP include source code                 | `include_name` (string): name of the ABAP include`                 | `@tool GetInclude include_name=ZMY_INCLUDE`                |
| `SearchObject`      | Search for ABAP objects using quick search.       | `query` (string), `maxResults` (number, optional, default 100)     | `@tool SearchObject query=ZMY* maxResults=20`              |
| `GetInterface`      | Retrieve ABAP interface source code.              | `interface_name` (string): Name of the ABAP interface.             | `@tool GetInterface interface_name=ZIF_MY_INTERFACE`       |
| `GetTransaction`    | Retrieve ABAP transaction details.                | `transaction_name` (string): Name of the ABAP transaction.         | `@tool GetTransaction transaction_name=ZMY_TRANSACTION`    |
