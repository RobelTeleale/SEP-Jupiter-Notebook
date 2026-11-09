# SEP Robotics × probly: Personenerkennung auf Bildern

Erster Baustein unseres SEP-Projekts: Ein **vortrainierter Objektdetektor**
(`facebook/detr-resnet-50`) erkennt Personen und andere Objekte auf Straßenbildern.
Das Notebook ist die **Einzelmodell-Baseline**. Später vergleichen wir es mit
Unsicherheitsmethoden aus [probly](https://github.com/pwhofman/probly), etwa Ensemble,
MC Dropout und MC DropConnect.

> Es wurde **nichts selbst trainiert**, und **probly ist noch nicht eingebaut**.
> Die Scores im Notebook sind Detektorkonfidenzen und keine probly-Unsicherheit.

---

## Inhalt des Repositorys

| Datei / Ordner | Was ist das? |
|---|---|
| `Personen_auf_Bildern.ipynb` | Das Notebook. Die Ausgaben sind gespeichert, man sieht die Ergebnisse also auch ohne Ausführen. |
| `bilder/` | Die 8 Testbilder (0–3 aus dem Internet, 4–7 KI-generiert). |
| `ergebnisse/` | Markierte Ergebnisbilder (`*_erkannt.png`) und `vergleich.json` mit allen Erkennungen bei Schwelle 0,5 und 0,9. |
| `Testbilder_Prompts.json` | Die Prompts, mit denen die KI-Bilder 4–7 erzeugt wurden. |
| `requirements.txt` | Benötigte Python-Pakete. |

---

## Nur anschauen (ohne Installation)

Das Notebook direkt hier auf GitHub anklicken. Code, Erklärungen und Ergebnisbilder
werden angezeigt. Alternativ kann man es in PyCharm oder VS Code öffnen.

## Selbst ausführen

**Voraussetzung:** Python 3.12. Getestet wurde es auf einem Mac (Apple Silicon) mit CPU.
Windows und Linux sollten auch funktionieren.

```bash
git clone <URL dieses Repositorys>
cd sep-robotics-probly-personenerkennung

python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt

jupyter lab Personen_auf_Bildern.ipynb
```

PyCharm geht auch: den Ordner öffnen, `.venv` als Interpreter wählen und das Notebook öffnen.

Beim **ersten Lauf** lädt das Notebook das Modell (ca. 170 MB) von Hugging Face
herunter. Ein Token ist nicht nötig. Danach liegt es im Cache.

---

## Bedienung

Die Zellen von oben nach unten mit **Shift+Enter** ausführen. `[*]` bedeutet: Die Zelle rechnet noch.

| Abschnitt | Was passiert | Was man einstellen kann |
|---|---|---|
| 1 | Pakete und Ordner prüfen | `DEVICE = "cpu"` (oder `"mps"` für die Apple-GPU, `"cuda"` für NVIDIA) |
| 2 | Modell laden, unterstützte Klassen anzeigen | `MODEL_ID`, z. B. `"facebook/detr-resnet-101"` |
| 3 | Bilder aus `bilder/` auflisten | Eigene JPG/PNG-Bilder in `bilder/` legen und die Zelle neu ausführen |
| 4 | Bild anzeigen | `BILD_NUMMER = 0` … `7` |
| 5 | Modell auf das Bild anwenden | – |
| 6 | Objekte filtern, markieren und zählen | `SCHWELLE` (z. B. 0.5 oder 0.9) und `KLASSEN` |
| 7 | Markiertes PNG und JSON in `ergebnisse/` speichern | – |

**Wichtig:**
- **Anderes Bild:** Abschnitte **4 → 5 → 6** neu ausführen.
- **Andere Schwelle oder andere Klassen:** Nur **Abschnitt 6** neu ausführen. Das Modell rechnet dabei nicht neu, es wird nur anders gefiltert.
- **`KLASSEN`-Beispiele:**
  - `None` zeigt alle Klassen.
  - `["person"]` zeigt nur Menschen.
  - `["person", "car", "bus", "truck", "bicycle", "motorcycle"]` zeigt Verkehrsteilnehmer.
  - Die Klassennamen sind Englisch, siehe die Liste in Abschnitt 2.
- **Fehler wie `NameError`:** **Kernel → Restart Kernel** und dann alles von oben ausführen.

---

## Bisherige Ergebnisse (Klasse `person`)

| Bild | Szene | Manuell gezählt | Modell @ 0,5 | Modell @ 0,9 |
|---|---|---|---|---|
| 0 | Bus und Personen | 4 | 4 | 4 |
| 1 | Wohnstraße mit Autos | 2 (+ unklares Motiv) | 3 | 2 |
| 2 | Nachtaufnahme | ca. 22–27 | 28 | 10 |
| 3 | Menschenmenge | ca. 85–105 | 29 | 13 |
| 4 | Distanz und Schaufensterpuppe (KI) | 4 | 11 | 5 |
| 5 | Nebel und Dämmerung (KI) | 3 | 3 | 2 |
| 6 | Regen und Gegenlicht (KI) | 3 | 3 | 3 |
| 7 | Verdeckung durch Lieferwagen (KI) | 3 | 4 | 3 |

Bei den Bildern 2 und 3 ist die manuelle Zählung eine Schätzung. Verglichen wurden
nur Anzahlen, nicht ob jedes Kästchen eine echte Person trifft.

**Kurzfazit:**
- Einfache Szenen klappen.
- Es gibt keine Schwelle, die überall passt: 0,5 erzeugt Fehlalarme, 0,9 übersieht Personen.
- Menschenmengen werden stark unterzählt.
- Der Score sagt nicht, ob man der Erkennung trauen kann. Genau hier soll probly ansetzen.

---

## Nächste Schritte

1. Kleines Klassifizierungs- und OOD-Experiment mit probly: Einzelmodell, Ensemble,
   MC Dropout und MC DropConnect vergleichen.
2. Eine saubere Testmenge mit manuell beschrifteten Personen-Kästchen anlegen.
3. Prüfen, ob hohe Unsicherheit tatsächlich mit Fehlern zusammenfällt.
4. Später Bilder aus dem CARLA-Simulator einbinden.

---

## Quellen und Hinweise

- Modell: [facebook/detr-resnet-50](https://huggingface.co/facebook/detr-resnet-50)
- Unsicherheitsbibliothek: [probly](https://github.com/pwhofman/probly)
- Bild 0: [Ultralytics-Beispielbild `bus.jpg`](https://github.com/ultralytics/ultralytics/blob/main/ultralytics/assets/bus.jpg)
- Bilder 1–3 stammen aus dem Internet. Die Lizenz ist nicht geklärt, deshalb ist das Repository **privat** und die Bilder sind nur für interne Uni-Zwecke gedacht.
- Bilder 4–7 sind synthetisch (KI-generiert). Die Prompts stehen in `Testbilder_Prompts.json`.
