# 👽 EAPOL Hunter — Flipper Zero App

Analysiert WiFi-Marauder-Captures **direkt auf dem Flipper Zero** auf WPA-Handshakes (EAPOL
4-Way-Handshake) und PMKID-Frames, filtert Treffer in einen eigenen Ordner und exportiert eine
Hashcat-kompatible Metadatendatei — ganz ohne PC, direkt am Gerät.

<p align="center">
  <img src="images/splash_128x64.png" width="256" alt="EAPOL Hunter Splash Screen" />
</p>

> ⚠️ **Nur für eigene Netzwerke und Geräte verwenden!**
> Diese App darf **ausschließlich** zur Überprüfung der Sicherheit **eigener** WLAN-Netzwerke
> bzw. mit **ausdrücklicher Erlaubnis** des Netzwerkbetreibers eingesetzt werden. Das Erfassen
> oder Auswerten von Handshakes fremder Netzwerke ohne Zustimmung ist in den meisten
> Rechtsordnungen strafbar. Nutzung auf eigene Verantwortung.

---

## ✨ Funktionen

| Menüpunkt | Funktion |
|---|---|
| **1. Ordner analysieren** | Durchsucht `/ext/apps_data/marauder/pcaps` rekursiv nach Capture-Dateien, prüft jede per Byte-Pattern-Erkennung auf EAPOL/PMKID-Signaturen, kopiert Treffer in `EAPOL_gefiltert/` und schreibt eine Übersichtsdatei (`alle_handshakes.22000`) |
| **2. Unbrauchbare löschen** | Entfernt alle Capture-Dateien, die beim letzten Scan keinen Handshake enthielten — räumt die SD-Karte auf |
| **3. Ergebnis anzeigen** | Zeigt die Statistik des letzten Scans (geprüfte Dateien, Treffer, gelöschte Dateien) |
| **4. Export für Hashcat (PC)** | Hinweisbildschirm: die exportierte Datei ist bereit für die Weiterverarbeitung mit Hashcat auf dem PC |
| **5. Über die App** | Erklärung, warum das Passwort-Cracking selbst nicht auf dem Flipper läuft |
| **6. Beenden** | App schließen |

## 🔬 Technischer Hintergrund

Die Erkennung läuft komplett lokal auf dem Flipper per einfachem Byte-Pattern-Matching — kein
voller 802.11-Parser nötig:

- **EAPOL-Erkennung:** sucht den EtherType `0x88 0x8E`, gefolgt vom EAPOL-Key-Frame-Typ (`0x03`)
- **PMKID-Erkennung:** sucht die RSN-Key-Data-Signatur (Tag `0xDD`, OUI `00:0F:AC`, Type `4`)

### Warum kein Cracking auf dem Flipper?

Das eigentliche **Passwort-Cracking findet bewusst NICHT auf dem Flipper statt.** Der
Cortex-M4-Prozessor (64 MHz, keine Krypto-Beschleunigung) würde für die PBKDF2-HMAC-SHA1-Berechnung
(4096 Runden pro Passwortversuch) nur etwa **1–5 Hashes/Sekunde** schaffen — gegenüber rund
**20.000 Hashes/Sekunde** auf einer normalen PC-CPU. Ein rockyou.txt-Wörterbuchlauf
(14,3 Mio. Einträge), der am PC rund 8 Minuten dauert, würde auf dem Flipper Wochen bis Monate
benötigen. Die App exportiert daher nur die aufbereiteten Metadaten — das eigentliche Cracking
übernimmt [Hashcat](https://hashcat.net/hashcat/) auf dem PC (Hash-Modus `22000`).

## 📦 Installation

### Fertiges `.fap` herunterladen (empfohlen)

1. Lade die aktuellste `eapol_hunter.fap` aus dem [Releases-Bereich](../../releases) herunter
2. Kopiere die Datei per qFlipper/USB-Massenspeicher-Modus nach `/ext/apps/GPIO/` auf deiner
   Flipper-SD-Karte
3. Starte die App über **Apps → GPIO → EAPOL Hunter**

### Selbst bauen (ufbt)

```bash
git clone https://github.com/Figarow/eapol-hunter-flipper.git
cd eapol-hunter-flipper
pip install ufbt      # falls noch nicht installiert
ufbt                   # baut die .fap
ufbt launch            # baut, installiert und startet direkt auf dem angeschlossenen Flipper
```

## ⚖️ Lizenz

MIT-Lizenz, siehe [LICENSE](LICENSE) — mit dem oben genannten **Nutzungshinweis**: nur für
eigene Netzwerke/Geräte oder mit ausdrücklicher Erlaubnis des Netzwerkbetreibers.

## 🙏 Danksagung

Erstellt für die [WiFi-Marauder](https://github.com/justcallmekoko/ESP32Marauder)-Erweiterung
auf [Flipper Zero](https://flipperzero.one/) / [Momentum Firmware](https://momentum-fw.dev/).
Passwort-Cracking-Workflow am PC nutzt [hcxtools](https://github.com/ZerBea/hcxtools) und
[Hashcat](https://hashcat.net/hashcat/).
