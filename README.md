# Windkanal

Ein virtueller Windkanal im Browser: Modell wählen, Wind einstellen, starten – dann zeigt die Seite Stromlinien, Geschwindigkeit und Druck um das Modell, dazu Luftwiderstand und Auftrieb (c<sub>W</sub>, c<sub>A</sub>). Gerechnet wird mit der Lattice-Boltzmann-Methode, komplett auf der Grafikkarte (WebGPU). Keine Installation, kein Server, eine einzige Datei.

## Starten

Online: **https://fbw159.github.io/windkanal/** – oder `index.html` doppelt anklicken. Nötig ist ein Browser mit WebGPU, am besten aktuelles **Chrome oder Edge**. Unter Windows mit zwei Grafikkarten (z. B. Quadro und Intel) in den Windows-Grafikeinstellungen für den Browser „Hohe Leistung“ einstellen.

## Ablauf

1. **Modell** wählen: Pkw, Lkw, Tragflügel, Kugel, Haus oder Schornstein – oder eine eigene STL- oder OBJ-Datei laden (Knopf oder Datei ins Fenster ziehen).
2. **Wind** einstellen: Richtung am Drehteller (Pfeil ziehen), bei frei schwebenden Modellen auch die Neigung, dazu die Geschwindigkeit.
3. **Rechnung** wählen: **instationär** (zeitgenau, Wirbel lösen sich ab und ziehen mit) oder **stationär** (zeitlich gemittelte Strömung, rechnet bis c<sub>W</sub> und c<sub>A</sub> sich nicht mehr ändern).
4. **Genauigkeit** wählen (Stufen bis 24 Mio. Zellen, je nach Grafikspeicher). Der grüne Punkt markiert die Empfehlung für die eigene Grafikkarte.
5. **Windkanal starten.** Nach zwei Durchläufen der Luft durch den Kanal ist die Strömung eingelaufen; danach werden c<sub>W</sub> und c<sub>A</sub> gemittelt (Anzeige oben rechts, mit Verlauf).
6. **Darstellung**: Farbe nach Geschwindigkeit oder Druck, Stromlinien, Modell einfärben, Schnittebene, zeitliche Mittelung, Strömung animieren.

Ansicht: linke Maustaste ziehen = drehen, rechte Maustaste oder Umschalttaste = verschieben, Mausrad = näher/weiter, Doppelklick = zurücksetzen. Leertaste = Start/Pause, R = neu starten, H = Hilfe.

## Eigene Modelle

- STL (binär und ASCII) und OBJ.
- Nach dem Laden lassen sich die **größte Länge in Metern**, die **Hochachse der Datei** (z oder y), die **Ausrichtung** (in 90°-Schritten) und **„steht auf dem Boden“** einstellen, beim Boden zusätzlich, ob er wie bei einem Gebäude (fest, Wind mit Höhenprofil) oder wie bei einem Fahrzeug (glatt) wirken soll. Dateien mit Abmessungen über 50 werden zunächst als Millimeter gelesen.
- Zum Ausprobieren liegt `Beispielmodell/Transporter.stl` bei (Kastenwagen, 5,4 m, in Millimetern).
- Das Modell sollte geschlossen sein. Offene Netze werden als dünne Wand gerechnet (Hinweis erscheint).

## Was gerechnet wird

- **Lattice-Boltzmann** (D3Q19) mit der echten Zähigkeit der Luft, Stoß rekursiv regularisiert. Gitterwind 0,1 Zellen je Schritt.
- **Instationär:** Grobstruktursimulation mit dem WALE-Modell für die Wirbel, die kleiner als eine Zelle sind.
- **Stationär:** k-ω-SST-Turbulenzmodell (RANS) mit Wandfunktionen.
- **Wand:** liegt an ihrer echten Stelle zwischen den Zellmitten (Interpolation nach Bouzidi). Teile, die dünner als eine Zelle sind (Hinterkanten, abgeschnittene Kanten), zählen trotzdem als Wand. Die Grenzschicht direkt an der Wand ist viel dünner als eine Zelle: ein Wandgesetz (Spalding) setzt die Reibung an, eine zusätzliche Zähigkeit in den Zellen an der Wand (Mischungsweg nach Prandtl, Dämpfung nach van Driest) bringt den Schwung der Strömung bis an die Wand. Ein Ausgleich verhindert falsche Druckbuckel an den Treppenstufen gekrümmter Wände.
- **Laminar oder turbulent:** Die Reynoldszahl des Modells (Geschwindigkeit mal Bezugslänge durch Zähigkeit, oben rechts angezeigt) entscheidet, ob die Grenzschicht am Modell laminar (unter 2·10⁵) oder turbulent (über 5·10⁵) gerechnet wird, dazwischen gleitend. Laminar löst die Strömung an Rundungen früher ab. Wo genau der Umschlag liegt, rechnet die Seite nicht aus.
- **Ränder:** vorn strömt der Wind ein (Schallwellen können hinaus), seitlich, oben und hinten kann die Luft frei ein- und ausströmen (mit dämpfender Randzone). Kanal vor dem Modell 1,5, hinter dem Modell 3,5, seitlich und oben je 1,5 Modellgrößen (Modellgröße = dritte Wurzel aus dem Volumen des umgebenden Quaders). Fahrzeuge: glatter Boden ohne Reibung (wie Fahrzeug-Windkanal mit Laufband); unter den Rädern wird der schmale Keil zwischen Boden und Reifen dem Modell zugeschlagen (bis 4 Zellen von der Aufstandsfläche). Gebäude: rauer Boden (Rauigkeitslänge 5 cm), Wind mit logarithmischem Höhenprofil, die angegebene Geschwindigkeit gilt in Gebäudehöhe.
- **Kräfte:** Impulsaustausch an jeder Wandverbindung plus Wandreibung; c<sub>W</sub> und c<sub>A</sub> bezogen auf Staudruck und Stirnfläche (Tragflügel: Flügelfläche), gemittelt über die Zeit nach dem Einlaufen.
- **Selbsttest beim Start:** Die Grafikkarte rechnet kleine Gitter (Kugel frei, Quader auf glattem und rauem Boden, je instationär und stationär), das Ergebnis wird mit derselben Rechnung auf dem Prozessor (doppelte Genauigkeit) verglichen. Fällt der Test durch, erscheint oben ein roter Hinweis.

## Genauigkeit

Geprüft an Körpern mit bekannten Messwerten (Quadro RTX 4000, Stufe mit 8 Mio. Zellen, instationär):

| Körper | Rechnung | Messung |
|---|---|---|
| Kugel, 60 m/s (Re 4·10⁶, turbulent) | c<sub>W</sub> 0,21 | 0,18–0,20 |
| Kugel, 1 m/s (Re 7·10⁴, laminar) | c<sub>W</sub> 0,40 (stationär 0,33) | 0,47 |
| Kugel, sehr zäh (Re 100, 30 Zellen je Durchmesser) | c<sub>W</sub> 1,16 | 1,09 |
| Würfel, frontal, 30 m/s | c<sub>W</sub> 1,20 (4 Mio. Zellen: 1,09; 16 Mio.: 1,25) | 1,05 |
| Ahmed-Körper 25°, 60 m/s (vereinfachtes Auto) | c<sub>W</sub> 0,48, c<sub>A</sub> 0,36 (stationär 0,48 / 0,34; 24 Mio. Zellen: 0,44 / 0,24) | 0,29–0,30 / 0,35 |
| Tragflügel NACA 2412, 8°, 30 m/s | c<sub>A</sub> 0,49, c<sub>W</sub> 0,13 (24 Mio. Zellen: 0,34 / 0,11) | ≈ 0,70 / ≈ 0,05 |
| Schornstein (Zylinder, 2 m, 10 m/s) | c<sub>W</sub> 0,41 (stationär 0,41) | 0,35–0,5 (endlicher Zylinder, überkritisch) |

Wo die Strömung abreißt und wie die Wirbel dahinter ziehen, gibt die Rechnung gut wieder. Bei stumpfen Körpern wie Kugel und Würfel liegt c<sub>W</sub> innerhalb von etwa 20 % der Messung. Bei **gerundeten Vorderkanten** fällt c<sub>W</sub> zu hoch aus, weil die Rundung nur wenige Zellen umfasst (Ahmed-Körper: Radius 10 Zellen). **Dünne Flügelprofile** sind die größte Schwäche: Die Vorderkante ist kleiner als eine Zelle und die Grenzschicht viel dünner als eine Zelle. Der Auftrieb fällt ein Drittel bis die Hälfte zu klein aus, der Widerstand deutlich zu groß, und feinere Stufen helfen nicht, weil das wandnahe Modell (Mischungsweg über 2 bis 4 Zellen) an der Zellgröße hängt. Auch sonst ändern sich die Werte von Stufe zu Stufe um einige Prozent, der Auftrieb mitunter mehr (Ahmed-Körper). Der Umschlag laminar–turbulent ist grob nach der Reynoldszahl geschaltet. Der Kanal ist endlich groß, das Modell versperrt einige Prozent des Querschnitts; am Auslass wird ein Teil der Schallwellen zurückgeworfen.

Bei kantigen Körpern wie Auto, Lkw und Haus ändert sich die Strömung mit der Geschwindigkeit nur wenig (physikalisch richtig), Kräfte und Druck wachsen aber mit dem Quadrat der Geschwindigkeit. Bei gerundeten Körpern (Kugel, Schornstein) ändert sich das Bild deutlich, wenn die Grenzschicht von laminar zu turbulent wechselt.

## Technik

- Eine HTML-Datei ohne externe Abhängigkeiten. Shader in WGSL, alle 19 Gitterrichtungen ausgeschrieben; die Gittermaße stehen im Uniform-Puffer, die Shader werden also nur einmal übersetzt.
- Rechenleistung auf einer Quadro RTX 4000: etwa 1,3 Mrd. Zellen je Sekunde; 8 Mio. Zellen brauchen für sechs Durchläufe gut zwei Minuten.
- Prüfschnittstelle für automatische Tests: `WK.run({...})` (siehe Quelltext, Abschnitt „Prüfschnittstelle“).
