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

- STL (binär und ASCII) und OBJ. Der Leser kommt auch mit Dateien zurecht, bei denen die Dreieckszahl im Kopf nicht zur Dateigröße passt, die abgeschnitten sind oder einen Anhang haben, und mit Textdateien in Großbuchstaben oder mit Dezimalkomma; was er dabei übergangen hat, steht in der Meldung nach dem Laden.
- Nach dem Laden lassen sich die **größte Länge in Metern**, die **Hochachse der Datei** (z oder y), die **Ausrichtung** (in 90°-Schritten) und **„steht auf dem Boden“** einstellen, beim Boden zusätzlich, ob er wie bei einem Gebäude (fest, Wind mit Höhenprofil) oder wie bei einem Fahrzeug (glatt) wirken soll. Dateien mit Abmessungen über 50 werden zunächst als Millimeter gelesen.
- Zum Ausprobieren liegt `Beispielmodell/Transporter.stl` bei (Kastenwagen, 5,4 m, in Millimetern).
- Das Modell sollte geschlossen sein. Offene Netze werden als dünne Wand gerechnet (Hinweis erscheint); verdrehte Dreiecke und doppelte Flächen stören nicht. Hohlräume ohne Verbindung nach außen zählen als fest. Flächen ohne Dicke (nur eine Dreiecksschicht, wie aus Flächenmodellierern) werden als dünne Wand gerechnet.
- Ist das Modell nur wenige Zellen groß, kommt eine Meldung; Beiwerte sind dann nicht brauchbar. Ein Modell ohne Stirnfläche (ebene Platte längs zum Wind) bekommt die Grundfläche als Bezug.

## Was gerechnet wird

- **Lattice-Boltzmann** (D3Q19) mit der echten Zähigkeit der Luft, Stoß rekursiv regularisiert. Gitterwind 0,1 Zellen je Schritt.
- **Instationär:** Grobstruktursimulation mit dem WALE-Modell für die Wirbel, die kleiner als eine Zelle sind.
- **Stationär:** k-ω-SST-Turbulenzmodell (RANS) mit Wandfunktionen; Anströmwerte nach Spalart & Rumsey (k = 10⁻⁶ U², ω = 5 U/L), von Umgebungstermen bis zum Modell gehalten.
- **Wand:** liegt an ihrer echten Stelle zwischen den Zellmitten (Interpolation nach Bouzidi). Teile, die dünner als eine Zelle sind (Hinterkanten, abgeschnittene Kanten), zählen trotzdem als Wand. Die Grenzschicht direkt an der Wand ist viel dünner als eine Zelle: ein Wandgesetz (Spalding, am rauen Boden logarithmisch) liefert die Wandschubspannung, die als wirksame Zähigkeit der Wandzelle angesetzt wird (wie die Wandfunktionen der Finite-Volumen-Programme); eine zusätzliche Zähigkeit in den Zellen an der Wand (Mischungsweg nach Prandtl, Dämpfung nach van Driest) bringt den Schwung der Strömung bis an die Wand. Ein Ausgleich verhindert falsche Druckbuckel an den Treppenstufen gekrümmter Wände. Spalte, die schmaler als eine Zelle sind (etwa zwischen Fahrerhaus und Koffer oder zwischen Reifen und Radkasten), gelten als ruhende Luft: Dort kann nichts hindurchströmen. Ohne diese Regel schwang der Impuls in solchen Spalten Schritt für Schritt hin und her und störte Wandgesetz, Turbulenzmodell und Kräfte. Liegt eine Wand fast genau in der Mitte zwischen zwei Zellmitten, wird sie um höchstens 0,04 Zellen verschoben: Genau in der Mitte wird die Wandregel zum reinen Rückprall, und der Stoß beim Start klingt dann erst nach Hunderten von Schritten ab. An scharfen Kanten gelten auch die Diagonalverbindungen, an denen die Kante knapp vorbeiläuft, als Wand; sonst hinge der Widerstand kantiger Körper davon ab, wo ihre Flächen im Gitter liegen (Quader: 0,66 oder 0,97).
- **Wind über rauem Boden (Gebäude):** logarithmisches Profil am Einlass; in den untersten vier Zellen gilt die Wirbelviskosität der Bodengrenzschicht κ·u*·(z + z₀) als Untergrenze, sonst verlor der Wind dort auf dem Weg zum Gebäude seinen Schwung (im leeren Kanal kehrte die Bodenzelle um). Die unterste Zelle bleibt trotzdem langsamer als das Profil; das Profil gilt verlässlich ab etwa zwei Zellen Höhe.
- **Rechengebiet:** vor dem Modell 1,5, dahinter 3,5 Modelllängen (Kubikwurzel des Volumens), seitlich und oben 1,5; bei flachen Modellen (Platte, Wand) nach der Stirnfläche, Nachlauf mindestens drei Stirnbreiten. Das Modell versperrt höchstens 7 % des Querschnitts, sonst wächst der Kanal: Bei mehr staute sich die Luft vor einer Platte, bis der Einlass umkehrte.
- **Laminar oder turbulent:** Die Reynoldszahl des Modells (Geschwindigkeit mal Bezugslänge durch Zähigkeit, oben rechts angezeigt) entscheidet, ob die Grenzschicht am Modell laminar (unter 2·10⁵) oder turbulent (über 5·10⁵) gerechnet wird, dazwischen gleitend. Laminar löst die Strömung an Rundungen früher ab. Wo genau der Umschlag liegt, rechnet die Seite nicht aus.
- **Ränder:** vorn strömt der Wind ein (Schallwellen können hinaus; heben Versperrung oder Bodenreibung den Druck im Kanal, strömt etwas weniger als die eingestellte Geschwindigkeit ein, bei Fahrzeugen unter 1 %, bei Gebäuden einige % – die Beiwerte sind auf die tatsächliche Einströmgeschwindigkeit bezogen), seitlich, oben und hinten kann die Luft frei ein- und ausströmen (mit dämpfender Randzone). Kanal vor dem Modell 1,5, hinter dem Modell 3,5, seitlich und oben je 1,5 Modellgrößen (Modellgröße = dritte Wurzel aus dem Volumen des umgebenden Quaders). Fahrzeuge: glatter Boden ohne Reibung (wie Fahrzeug-Windkanal mit Laufband); unter den Rädern wird der schmale Keil zwischen Boden und Reifen dem Modell zugeschlagen (bis 4 Zellen von der Aufstandsfläche). Gebäude: rauer Boden (Rauigkeitslänge 5 cm), Wind mit logarithmischem Höhenprofil, die angegebene Geschwindigkeit gilt in Gebäudehöhe; stationär wirkt am oberen Rand die Schubspannung des Profils, damit es auf dem Weg zum Gebäude nicht abflacht.
- **Kräfte:** Impulsaustausch an jeder Wandverbindung plus Wandreibung; c<sub>W</sub> und c<sub>A</sub> bezogen auf Staudruck und Stirnfläche (Tragflügel: Flügelfläche), gemittelt über die Zeit nach dem Einlaufen.
- **Darstellung des Drucks:** Die Farbskala reicht von −q bis +q (q = Staudruck) und ist in der Mitte gestreckt (Farbe ~ |c<sub>p</sub>|^0,7), damit auch der schwache Unterdruck hinter einem Fahrzeug (rund ein Zehntel des Staudrucks) und die Druckspuren der Wirbel zu sehen sind, nicht nur der Staupunkt. Der gezeigte Druck ist über etwa 50 Rechenschritte geglättet (Schall und Zellenrauschen verschwinden, die Wirbel bleiben).
- **Selbsttest beim Start:** Die Grafikkarte rechnet kleine Gitter (Kugel frei, Quader auf glattem und rauem Boden, je instationär und stationär), das Ergebnis wird mit derselben Rechnung auf dem Prozessor (doppelte Genauigkeit) verglichen. Fällt der Test durch, erscheint oben ein roter Hinweis.

## Genauigkeit

Geprüft an Körpern mit bekannten Messwerten (Quadro RTX 4000, Stufe mit 8 Mio. Zellen, instationär, Mittel über zwei Durchläufe nach eineinhalb Durchläufen Einlaufzeit; Stand Fassung 5):

| Körper | Rechnung | Messung |
|---|---|---|
| Kugel, 60 m/s (Re 4·10⁶, turbulent) | c<sub>W</sub> 0,22 | 0,18–0,20 |
| Kugel, 1 m/s (Re 7·10⁴, laminar) | c<sub>W</sub> 0,32 | 0,47 |
| Würfel, frontal, 30 m/s | c<sub>W</sub> 1,28 (2 Mio. Zellen: 1,08) | 1,05 |
| Ahmed-Körper 25°, 60 m/s (vereinfachtes Auto) | c<sub>W</sub> 0,50, c<sub>A</sub> 0,34 (stationär 0,53 / 0,35) | 0,29–0,30 / 0,35 |
| Tragflügel NACA 2412, 8°, 30 m/s | c<sub>A</sub> 0,50, c<sub>W</sub> 0,13 | ≈ 0,70 / ≈ 0,05 |
| Schornstein (Zylinder, 2 m, 10 m/s) | c<sub>W</sub> 0,42 | 0,35–0,5 (endlicher Zylinder, überkritisch) |
| Platte 2 × 1,2 m quer zum Wind, 20 m/s (eigene STL-Datei) | c<sub>W</sub> 1,65 (2 Mio. Zellen; 4 Mio.: 1,75, stationär 1,31) | ≈ 1,17 |

Wo die Strömung abreißt und wie die Wirbel dahinter ziehen, gibt die Rechnung gut wieder. Kugel und Schornstein liegen nahe an der Messung, solange die Grenzschicht turbulent ist; die **laminare Kugel** (1 m/s) fällt zu niedrig aus. Bei **kantigen Körpern** (Würfel, Platte) liegt c<sub>W</sub> auf feinen Stufen 20–40 % über der Messung, auf groben Stufen näher dran: Die Scherschichten an den Kanten sind auch mit 40 Zellen je Kantenlänge nicht aufgelöst, der Unterdruck im Nachlauf fällt zu groß aus. Bei **gerundeten Vorderkanten** fällt c<sub>W</sub> zu hoch aus, weil die Rundung nur wenige Zellen umfasst (Ahmed-Körper: Radius 10 Zellen). **Dünne Flügelprofile** sind die größte Schwäche: Die Vorderkante ist kleiner als eine Zelle und die Grenzschicht viel dünner als eine Zelle. Der Auftrieb fällt ein Drittel zu klein aus, der Widerstand deutlich zu groß, und feinere Stufen helfen nicht, weil das wandnahe Modell (Mischungsweg über 2 bis 4 Zellen) an der Zellgröße hängt. Auch sonst ändern sich die Werte von Stufe zu Stufe um einige Prozent. Der Umschlag laminar–turbulent ist grob nach der Reynoldszahl geschaltet. Der Kanal ist endlich groß, das Modell versperrt bis zu 7 % des Querschnitts; am Auslass wird ein Teil der Schallwellen zurückgeworfen.

Bei kantigen Körpern wie Auto, Lkw und Haus ändert sich die Strömung mit der Geschwindigkeit nur wenig (physikalisch richtig), Kräfte und Druck wachsen aber mit dem Quadrat der Geschwindigkeit. Bei gerundeten Körpern (Kugel, Schornstein) ändert sich das Bild deutlich, wenn die Grenzschicht von laminar zu turbulent wechselt.

## Technik

- Eine HTML-Datei ohne externe Abhängigkeiten. Shader in WGSL, alle 19 Gitterrichtungen ausgeschrieben; die Gittermaße stehen im Uniform-Puffer, die Shader werden also nur einmal übersetzt.
- Rechenleistung auf einer Quadro RTX 4000: etwa 1,3 Mrd. Zellen je Sekunde; 8 Mio. Zellen brauchen für sechs Durchläufe gut zwei Minuten.
- Prüfschnittstelle für automatische Tests: `WK.run({...})` (siehe Quelltext, Abschnitt „Prüfschnittstelle“).
