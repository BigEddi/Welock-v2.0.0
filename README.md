# WeLock für Home Assistant

[![HACS Custom](https://img.shields.io/badge/HACS-Custom-41BDF5.svg)](https://hacs.xyz/docs/faq/custom_repositories)
[![Home Assistant](https://img.shields.io/badge/Home%20Assistant-2025.3%2B-03A9F4.svg)](https://www.home-assistant.io/)
![Version](https://img.shields.io/badge/Version-2.0.0-green.svg)

Diese Integration bindet **WeLock-Smartlocks**, die **WIFIBOX3** und **WeLock-Türkontakte** in Home Assistant ein. Du kannst damit die Tür aus Home Assistant entriegeln, den Batteriestand überwachen und siehst in Echtzeit, wer die Tür wann und wie geöffnet hat.

Die Verbindung läuft über die offizielle WeLock-Cloud (OAuth-Anmeldung). Ereignisse kommen per Push (MQTT) sofort an, es wird also nicht ständig abgefragt.

> Dies ist eine überarbeitete Version (v2) der Integration [hzjchina/hass-welock](https://github.com/hzjchina/hass-welock). Sie behebt mehrere Fehler, ergänzt neue Funktionen und kann die alte Version direkt ersetzen.

---

## Inhalt

- [Funktionen](#funktionen)
- [Unterstützte Geräte](#unterstützte-geräte)
- [Installation](#installation)
- [Einrichtung](#einrichtung)
- [Entitäten](#entitäten)
- [Optionen](#optionen)
- [Beispiel-Automationen](#beispiel-automationen)
- [Umstieg von Version 1.x](#umstieg-von-version-1x)
- [Fehlersuche](#fehlersuche)
- [Änderungen gegenüber v1.0.1](#änderungen-gegenüber-v101)
- [Entwicklung](#entwicklung)
- [Danksagung](#danksagung)

---

## Funktionen

- 🔓 **Tür entriegeln** direkt aus Home Assistant, per Sprachassistent oder Automation
- ⚡ **Echtzeit-Ereignisse** per Push: Öffnungen per Karte, Fingerabdruck, PIN, App, Fernbedienung …
- 👤 **Wer und wie?** Methode, Benutzer, Benutzer-ID und Uhrzeit jeder Öffnung
- 🎯 **Event-Entität** als sauberer Trigger für Automationen
- 🔋 **Batteriestand** von Schloss und Türkontakt, automatisch alle 30 Minuten aktualisiert
- 🚪 **Türkontakt** (offen/geschlossen)
- 🔘 **WIFIBOX3-Fernsteuerung** als Taste
- 🔁 **Robuste Verbindung** mit automatischem Wiederverbinden und erneuter Anmeldung bei abgelaufenem Zugang
- 💾 **Letzte Öffnung bleibt nach Neustart erhalten**
- 🧩 Neue Geräte werden automatisch erkannt, entfernte Geräte aufgeräumt
- 🩺 Diagnose-Download mit geschwärzten Zugangsdaten
- 🇩🇪 Vollständig deutsch und englisch übersetzt

## Unterstützte Geräte

| Gerät | Voraussetzung |
|---|---|
| WeLock-Schlösser (z. B. TOUCH, PCB, PBL, TouchEBL41 …) | über eine WIFIBOX mit der Cloud verbunden |
| WIFIBOX3 | in der WeLock-App eingerichtet |
| WeLock-Türkontakt | in der WeLock-App eingerichtet |

Schloss und WIFIBOX müssen in der **WeLock-App** bereits gekoppelt sein und dort funktionieren.

## Installation

### Variante A: HACS (empfohlen)

1. In Home Assistant **HACS** öffnen.
2. Oben rechts auf **⋮ → Benutzerdefinierte Repositories** klicken.
3. Die URL dieses Repositorys eintragen und als Kategorie **Integration** wählen.
4. Nach **WeLock** suchen und **Herunterladen** klicken.
5. Home Assistant **neu starten**.

### Variante B: Manuell

1. Den Ordner `custom_components/welock` aus diesem Repository herunterladen.
2. Ihn nach `/config/custom_components/welock` kopieren. Eine vorhandene alte Version wird dabei ersetzt.
3. Home Assistant **neu starten**.

## Einrichtung

> Wenn du bereits die alte Version nutzt, entfällt dieser Schritt. Siehe [Umstieg von Version 1.x](#umstieg-von-version-1x).

1. **Zugangsdaten in der WeLock-App holen:** App öffnen, im linken Menü auf das **Home-Assistant-Symbol** tippen und **Client-ID** sowie **Client-Secret** notieren.
2. In Home Assistant **Einstellungen → Geräte & Dienste → Integration hinzufügen** öffnen und **WeLock** auswählen.
3. Client-ID und Client-Secret eingeben.
4. Im sich öffnenden WeLock-Fenster mit deinem WeLock-Konto anmelden und den Zugriff bestätigen.
5. Fertig: Alle Geräte deines Kontos erscheinen automatisch.

## Entitäten

| Gerät | Entität | Typ | Beschreibung |
|---|---|---|---|
| Schloss | *(Gerätename)* | `lock` | Entriegeln. Das Schloss verriegelt automatisch und wird danach wieder als „verriegelt“ angezeigt. |
| Schloss | Batterie | `sensor` | Batteriestand in % |
| Schloss, WIFIBOX3 | Letzte Öffnung | `sensor` | Methode (Karte, Fingerabdruck, PIN-Code, App, …). Attribute: `user`, `user_id`, `time`, `method_id` |
| Schloss, WIFIBOX3 | Letzte Aktivität | `sensor` | Zeitpunkt der letzten Öffnung |
| Schloss, WIFIBOX3 | Aktivität | `event` | Wird bei jeder Öffnung ausgelöst (`unlock` oder `relay_action`) |
| WIFIBOX3 | Fernsteuerung auslösen | `button` | Löst die Fernsteuerung bzw. das Relais der WIFIBOX3 aus |
| Türkontakt | Tür | `binary_sensor` | Offen / geschlossen |
| Türkontakt | Batterie | `sensor` | Batteriestand in % |

**Mögliche Werte von „Letzte Öffnung“:**
`fingerprint` (Fingerabdruck), `passcode` (PIN-Code), `temporary_passcode` (temporärer Code), `card` (Karte), `nfc`, `face` (Gesichtserkennung), `app`, `bt_remote` (Bluetooth-Fernbedienung), `remote_433` (433-MHz-Fernbedienung), `wifibox` (WIFIBOX / Home Assistant), `vacation` (Urlaubsmodus), `local` (manuell von innen), `relay` (Relais), `other` (sonstiges).

In der Oberfläche werden die Werte übersetzt angezeigt. In Automationen verwendest du die englischen Werte aus der Liste.

## Optionen

Unter **Einstellungen → Geräte & Dienste → WeLock → Konfigurieren**:

| Option | Standard | Beschreibung |
|---|---|---|
| Automatische Verriegelung nach | 8 s | So lange zeigt Home Assistant das Schloss nach dem Öffnen als „entriegelt“ an. Passe den Wert an die Einstellung deines Schlosses an. |

## Beispiel-Automationen

**Benachrichtigung, wenn jemand die Tür öffnet**

```yaml
alias: WeLock – Tür geöffnet
triggers:
  - trigger: state
    entity_id: event.wohnungstur_aktivitat
    not_from: unavailable
actions:
  - action: notify.notify
    data:
      message: >
        Tür geöffnet per {{ trigger.to_state.attributes.method }}
        von {{ trigger.to_state.attributes.user or 'unbekannt' }}
```

**Licht im Flur einschalten, wenn per Karte geöffnet wird**

```yaml
alias: WeLock – Flurlicht bei Kartenöffnung
triggers:
  - trigger: state
    entity_id: event.wohnungstur_aktivitat
    not_from: unavailable
conditions:
  - condition: template
    value_template: "{{ trigger.to_state.attributes.method == 'card' }}"
actions:
  - action: light.turn_on
    target:
      entity_id: light.flur
```

**Warnung bei schwacher Batterie**

```yaml
alias: WeLock – Batterie schwach
triggers:
  - trigger: numeric_state
    entity_id: sensor.wohnungstur_batterie
    below: 20
actions:
  - action: notify.notify
    data:
      message: "WeLock-Batterie nur noch {{ states('sensor.wohnungstur_batterie') }} %"
```

> Die Entitäts-IDs (`event.wohnungstur_aktivitat` usw.) hängen vom Namen deines Geräts ab. Passe sie entsprechend an.

## Umstieg von Version 1.x

Version 2 verwendet dieselbe Domain (`welock`) und dieselben internen IDs wie die alte Version. Daher gilt:

- ✅ Geräte, Entitäts-IDs, Bereiche und Automationen bleiben erhalten.
- ✅ Die Anmeldung bleibt bestehen, eine neue Einrichtung ist nicht nötig.
- ✅ Das externe Python-Paket `welock-iot` wird nicht mehr benötigt.

**Vorgehen:** alten Ordner `custom_components/welock` durch den neuen ersetzen (oder in HACS aktualisieren) und Home Assistant neu starten.

**Kleine Unterschiede:**

- Der Sensor „Record“ heißt jetzt **„Letzte Öffnung“**. Er zeigt statt eines langen Textes (`unlocked by Card(10) at …`) die Methode an (`card`). Benutzer und Uhrzeit stehen in den Attributen.
- Die Schloss-Entität trägt jetzt den Gerätenamen statt „Operation“. Die Entitäts-ID ändert sich dadurch nicht.
- Neu hinzu kommen „Letzte Aktivität“ und „Aktivität“.

## Fehlersuche

**Beim Entriegeln erscheint „Entriegeln fehlgeschlagen: …“**
Die WeLock-Cloud hat den Befehl abgelehnt, meist weil die WIFIBOX offline ist oder das Schloss außer Reichweite. Der Text hinter dem Doppelpunkt stammt direkt von WeLock.

**„Verriegeln wird nicht unterstützt“**
WeLock-Schlösser verriegeln selbstständig. Die Cloud bietet keinen Befehl zum Verriegeln an.

**Ereignisse kommen nicht an**
Prüfe, ob die Öffnung in der WeLock-App angezeigt wird. Bricht die Push-Verbindung ab, verbindet sich die Integration automatisch neu (Warnung im Log: „WeLock push connection lost …“).

**„Erneut anmelden“ wird angezeigt**
Der Zugang ist abgelaufen oder wurde widerrufen. Auf die Meldung klicken und erneut bei WeLock anmelden.

**Debug-Protokoll aktivieren**

```yaml
logger:
  logs:
    custom_components.welock: debug
```

Alternativ unter **Einstellungen → Geräte & Dienste → WeLock → ⋮ → Debug-Protokollierung aktivieren**.

**Diagnose für einen Fehlerbericht**
**Einstellungen → Geräte & Dienste → WeLock → ⋮ → Diagnose herunterladen.** Zugangsdaten, Benutzernamen und Bluetooth-Namen werden dabei automatisch geschwärzt.

## Änderungen gegenüber v1.0.1

### Behobene Fehler

- Alle Geräte teilten sich intern einen einzigen Status-Speicher. Werte eines Geräts konnten dadurch bei anderen Geräten auftauchen.
- Fehlgeschlagenes Entriegeln wurde als Erfolg angezeigt.
- Die Push-Verbindung konnte bei unerwarteten Fehlern unbemerkt abbrechen, ohne sich neu zu verbinden.
- Abgelaufene Anmeldungen (HTTP 401) führten nicht zur erneuten Anmeldung.
- War die Cloud beim Start nicht erreichbar, blieb alles „nicht verfügbar“. Jetzt versucht Home Assistant es automatisch erneut.
- Die Bibliothek erzwang DEBUG-Logging und installierte einen eigenen Log-Handler.
- Veraltete API-Nutzung (`device_registry.devices`) entfernt, die ab Home Assistant 2027.9 nicht mehr funktioniert.
- „Record“ war nach jedem Neustart „unbekannt“; der Batteriestand wurde nur beim Neustart aktualisiert.

### Neu

- Event-Entität „Aktivität“
- Sensor „Letzte Aktivität“ (Zeitstempel)
- „Letzte Öffnung“ als übersetzter Auswahl-Sensor mit Attributen
- Schloss zeigt „entriegelt“ auch bei Öffnung direkt am Schloss
- Einstellbare Wiederverriegelungszeit
- Regelmäßige Aktualisierung (30 min), automatische Geräteerkennung
- Diagnose-Download
- Deutsche Übersetzung
- Automatisierte Tests

## Entwicklung

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements_test.txt
pytest
```

Die Tests laufen gegen eine echte Home-Assistant-Testinstanz mit simulierter WeLock-Cloud.

## Danksagung

Basiert auf der ursprünglichen Integration von [@hzjchina](https://github.com/hzjchina/hass-welock).
Diese Integration ist ein Community-Projekt und steht in keiner Verbindung zu WeLock.
