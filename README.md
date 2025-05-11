# Demons - HackMyVM (Easy)

![Demons.png](Demons.png)

## Übersicht

*   **VM:** Demons
*   **Plattform:** [HackMyVM](https://hackmyvm.eu/machines/machine.php?vm=Demons)
*   **Schwierigkeit:** Easy
*   **Autor der VM:** DarkSpirit
*   **Datum des Writeups:** 01. November 2022
*   **Original-Writeup:** https://alientec1908.github.io/Demons_HackMyVM_Easy/
*   **Autor:** Ben C.

## Kurzbeschreibung

Das Ziel dieser "Easy"-Challenge war es, Root-Zugriff auf der Maschine "Demons" zu erlangen. Der Angriff begann mit der Enumeration eines FTP-Servers mit anonymem Zugriff, von dem Dateien heruntergeladen wurden. Parallel wurde ein Webserver untersucht. Ein in den heruntergeladenen Dateien gefundener SSH-Private-Key ermöglichte den initialen Zugriff als Benutzer `aim`. Als `aim` wurde die User-Flag und eine Bilddatei (`key8_8.jpg`) gefunden, die Hinweise auf die Passwortstruktur für den Benutzer `agares` enthielt. Mittels `crunch` und einem Python-Skript zum Filtern wurde eine gezielte Wortliste erstellt und das Passwort für `agares` (`d3monf4m`) mit einem `su`-Brute-Force-Skript geknackt. Nach dem Wechsel zu `agares` wurde eine `sudo`-Regel entdeckt, die es erlaubte, `byebug` (einen Ruby-Debugger) als Root auszuführen. Durch das Debuggen eines einfachen Ruby-Skripts, das eine Bash-Shell startete, wurden Root-Rechte erlangt und die Root-Flag gelesen.

## Disclaimer / Wichtiger Hinweis

Die in diesem Writeup beschriebenen Techniken und Werkzeuge dienen ausschließlich zu Bildungszwecken im Rahmen von legalen Capture-The-Flag (CTF)-Wettbewerben und Penetrationstests auf Systemen, für die eine ausdrückliche Genehmigung vorliegt. Die Anwendung dieser Methoden auf Systeme ohne Erlaubnis ist illegal. Der Autor übernimmt keine Verantwortung für missbräuchliche Verwendung der hier geteilten Informationen. Handeln Sie stets ethisch und verantwortungsbewusst.

## Verwendete Tools

*   `arp-scan`
*   `nmap`
*   `ftp`
*   `wget`
*   `gobuster`
*   `stegsnow`
*   `stegseek`
*   `hydra`
*   `ssh`
*   `find`
*   `python3` (für http.server und Skripte)
*   `crunch`
*   `su`
*   `sudo` (auf Zielsystem)
*   `byebug` (Ruby-Debugger, als Exploit-Vektor)
*   Standard Linux-Befehle (`vi`, `cat`, `ls`, `echo`, `id`, `cd`)

## Lösungsweg (Zusammenfassung)

Der Angriff auf die Maschine "Demons" gliederte sich in folgende Phasen:

1.  **Reconnaissance & FTP Enumeration:**
    *   IP-Findung mittels `arp-scan` (Ziel: `192.168.2.100`).
    *   `nmap`-Scan identifizierte FTP (21/tcp, vsftpd 3.0.3, anonymer Login erlaubt), SSH (22/tcp) und Apache (80/tcp).
    *   Dateien wurden vom anonymen FTP-Server heruntergeladen. Darunter befand sich (vermutlich) ein SSH-Private-Key.

2.  **Web Enumeration & Initial Access (SSH als aim):**
    *   `gobuster` auf Port 80 fand u.a. das Verzeichnis `/hell/`.
    *   Steganographie-Versuche auf ein im Web gefundenes Bild (`Bael.jpg`) waren erfolglos.
    *   Ein Hydra-Brute-Force-Versuch auf SSH für den Benutzer `astaroth` scheiterte, da Passwort-Authentifizierung deaktiviert war.
    *   Der zuvor vom FTP-Server heruntergeladene SSH-Private-Key gehörte zum Benutzer `aim`. Ein SSH-Login als `aim` mit diesem Schlüssel war erfolgreich.

3.  **Privilege Escalation (von `aim` zu `agares`):**
    *   Als `aim` wurde die User-Flag (`DemonsDontCry`) gelesen.
    *   Eine Datei `key8_8.jpg` wurde im Home-Verzeichnis von `aim` gefunden und auf die Angreifer-Maschine übertragen. Diese Datei enthielt (vermutlich) Hinweise auf Zeichen (`3,4,o,d,f,n,m`), die im Passwort für den Benutzer `agares` vorkommen.
    *   Mit `crunch` wurde eine Passwortliste generiert, die dann mit einem Python-Skript gefiltert wurde, um nur Passwörter mit den spezifischen Zeichen zu behalten.
    *   Ein Python-Skript (`sucrack.py`, ausgeführt auf dem Zielsystem oder als Vorlage) wurde verwendet, um `su agares` mit der gefilterten Passwortliste zu bruteforcen. Das Passwort `d3monf4m` wurde gefunden.
    *   Ein Wechsel zum Benutzer `agares` mit `su agares` und dem Passwort `d3monf4m` war erfolgreich.

4.  **Privilege Escalation (von `agares` zu `root` via Sudo/Byebug):**
    *   Als `agares` wurde mittels `sudo -l` entdeckt, dass `/bin/byebug` (ein Ruby-Debugger) als jeder Benutzer (`ALL:ALL`) ohne Passwort ausgeführt werden darf.
    *   Ein einfaches Ruby-Skript (`script.rb`) mit dem Inhalt `system('/bin/bash')` wurde erstellt.
    *   Der Befehl `sudo /bin/byebug script.rb` wurde ausgeführt. Innerhalb von `byebug` wurde `continue` eingegeben.
    *   Dies führte das Ruby-Skript als `root` aus und startete eine Root-Shell. Die Root-Flag wurde aus `/root/root.txt` gelesen.

## Wichtige Schwachstellen und Konzepte

*   **Anonymer FTP-Zugriff mit sensiblen Dateien:** Ein privater SSH-Schlüssel wurde über anonymes FTP preisgegeben.
*   **Information Disclosure:** Hinweise auf Passwortstrukturen in einer Bilddatei (`key8_8.jpg`).
*   **Schwache Passwörter / Gezieltes Password Cracking:** Das Passwort für `agares` (`d3monf4m`) konnte durch gezielte Wortlistenerstellung und Brute-Force erraten werden.
*   **Unsichere `sudo`-Regel (Byebug):** Die Erlaubnis, einen Debugger (`byebug`) als Root auszuführen, ermöglichte eine einfache Privilegieneskalation durch Code-Ausführung innerhalb des Debuggers.
*   **Steganographie (Versuch):** Obwohl nicht erfolgreich, wurde es als möglicher Vektor untersucht.

## Flags

*   **User Flag (`/home/aim/user.txt`):** `DemonsDontCry`
*   **Root Flag (`/root/root.txt`):** `inTheDark_BeAlone_OrLAst!`

## Tags

`HackMyVM`, `Demons`, `Easy`, `FTP`, `Anonymous Access`, `SSH Private Key`, `Web`, `Password Cracking`, `crunch`, `su Brute-Force`, `sudo`, `byebug`, `Ruby`, `Privilege Escalation`, `Linux`
