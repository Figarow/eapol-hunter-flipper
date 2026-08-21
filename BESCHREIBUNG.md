# EAPOL Hunter — Flipper Zero App

## Beschreibung

**EAPOL Hunter** durchsucht direkt auf dem Flipper Zero die von der WiFi-Marauder-Erweiterung
aufgezeichneten Paket-Captures (`.pcap`/`.pcapng`) und erkennt darin automatisch WPA-Handshakes
(EAPOL 4-Way-Handshake) und PMKID-Frames. Gefundene Treffer werden in einen eigenen Unterordner
kopiert und als Hashcat-kompatible Metadatendatei aufbereitet — alles ohne PC, direkt am Gerät.

## Funktionen

| Menüpunkt | Funktion |
|---|---|
| **1. Ordner analysieren** | Durchsucht `/ext/apps_data/marauder/pcaps` rekursiv nach Capture-Dateien, prüft jede per Byte-Pattern-Erkennung auf EAPOL/PMKID-Signaturen, kopiert Treffer in `EAPOL_gefiltert/` und schreibt eine Übersichtsdatei (`alle_handshakes.22000`) |
| **2. Unbrauchbare löschen** | Entfernt alle Capture-Dateien, die beim letzten Scan keinen Handshake enthielten — räumt die SD-Karte auf |
| **3. Ergebnis anzeigen** | Zeigt die Statistik des letzten Scans (geprüfte Dateien, Treffer, gelöschte Dateien) |
| **4. Export für Hashcat (PC)** | Hinweisbildschirm: die exportierte Datei ist bereit für die Weiterverarbeitung mit Hashcat auf dem PC |
| **5. Über die App** | Erklärung, warum das Passwort-Cracking selbst nicht auf dem Flipper läuft |
| **6. Beenden** | App schließen |

## Technischer Hintergrund

Die Erkennung läuft komplett lokal auf dem Flipper per einfachem Byte-Pattern-Matching:
- **EAPOL-Erkennung**: sucht den EtherType `0x88 0x8E` gefolgt vom EAPOL-Key-Frame-Typ (0x03)
- **PMKID-Erkennung**: sucht die RSN-Key-Data-Signatur (Tag `0xDD`, OUI `00:0F:AC`, Type `4`)

Das eigentliche **Passwort-Cracking findet bewusst NICHT auf dem Flipper statt** — der
Cortex-M4-Prozessor (64 MHz, keine Krypto-Beschleunigung) würde für die PBKDF2-HMAC-SHA1-Berechnung
(4096 Runden pro Passwortversuch) nur etwa 1–5 Hashes/Sekunde schaffen, gegenüber ca. 20.000/Sekunde
auf einer normalen PC-CPU. Ein rockyou.txt-Wörterbuchlauf (14,3 Mio. Einträge), der am PC rund
8 Minuten dauert, würde auf dem Flipper Wochen bis Monate benötigen. Die App exportiert daher nur
die aufbereiteten Metadaten — das eigentliche Cracking übernimmt Hashcat auf dem PC.

## Design

- Eigener Splash-Screen im Grey-Alien-Stil (glatte ovale Kopfform, große schräge Mandelaugen)
- Klar strukturiertes Textmenü mit Fortschrittsbalken während der Analyse
- Statusanzeigen für Dateizahl, Treffer und aktuell verarbeitete Datei

## Installationsort

`/ext/apps/GPIO/eapol_hunter.fap`

## Getestet mit

24 reale WiFi-Marauder-Captures — 5 korrekt erkannte Treffer (inkl. zweier unabhängig am PC
per `hcxpcapngtool` verifizierter PMKID-Captures), keine Fehlklassifizierung bei reinen
Beacon-/Raw-Captures ohne Handshake.
