# Windkanal

Ein virtueller Windkanal im Browser: Modell wählen, Wind einstellen, starten – dann zeigt die Seite Stromlinien, Geschwindigkeit und Druck um das Modell. Gerechnet wird mit der Lattice-Boltzmann-Methode, komplett auf der Grafikkarte (WebGPU). Keine Installation, kein Server, eine einzige Datei.

## Starten

Online: **https://fbw159.github.io/windkanal/** – oder `index.html` doppelt anklicken. Nötig ist ein Browser mit WebGPU, am besten aktuelles **Chrome oder Edge**. Unter Windows mit zwei Grafikkarten (z. B. Quadro und Intel) in den Windows-Grafikeinstellungen für den Browser „Hohe Leistung“ einstellen.

## Ablauf

1. **Modell** wählen: Pkw, Lkw, Tragflügel, Kugel, Haus oder Schornstein – oder eine eigene STL- oder OBJ-Datei laden (Knopf oder Datei ins Fenster ziehen).
2. **Wind** einstellen: Richtung am Drehteller (Pfeil ziehen), bei frei schwebenden Modellen auch die Neigung, dazu die Geschwindigkeit.
3. **Genauigkeit** wählen (sieben Stufen von 0,25 bis 16 Mio. Zellen). Der grüne Punkt markiert die Empfehlung für die eigene Grafikkarte: Beim Laden wird kurz gemessen, wie schnell sie rechnet, und die feinste Stufe empfohlen, bei der die Strömung in etwa 25 Sekunden eingelaufen ist.
4. **Windkanal starten.** Nach ein bis zwei Durchläufen der Luft durch den Kanal ist die Strömung eingelaufen (Anzeige oben rechts).
5. **Darstellung**: Farbe nach Geschwindigkeit oder Druck, Stromlinien (Anzahl, Start als Fläche oder Reihe in wählbarer Höhe), Modell einfärben (Druck auf der Oberfläche bzw. Geschwindigkeit dicht daneben), Schnittebene (waagerecht, längs, quer), zeitliche Mittelung für ein ruhigeres Bild, Strömung animieren (wandernde Lichtpunkte auf den Stromlinien).

Ansicht: linke Maustaste ziehen = drehen, rechte Maustaste oder Umschalttaste = verschieben, Mausrad = näher/weiter, Doppelklick = zurücksetzen. Leertaste = Start/Pause, R = neu starten, H = Hilfe. Modell und Wind bleiben beim Drehen fest im Raum. Die Knöpfe 3D, Seite, Oben und Vorn richten sich nach dem Wind: In „Seite“ und „Oben“ kommt er immer von links.

## Eigene Modelle

- STL (binär und ASCII) und OBJ.
- Nach dem Laden lassen sich die **größte Länge in Metern**, die **Hochachse der Datei** (z oder y), die **Ausrichtung** (in 90°-Schritten) und **„steht auf dem Boden“** einstellen, beim Boden zusätzlich, ob er wie bei einem Gebäude (fest, Wind mit Höhenprofil) oder wie bei einem Fahrzeug (glatt) wirken soll. Dateien mit Abmessungen über 50 werden zunächst als Millimeter gelesen. Liegt die Unterseite der Datei bei z = 0, ist „steht auf dem Boden“ schon eingeschaltet.
- Zum Ausprobieren liegt `Beispielmodell/Transporter.stl` bei (Kastenwagen, 5,4 m, in Millimetern).
- Das Modell sollte geschlossen sein. Offene Netze werden als dünne Wand gerechnet (Hinweis erscheint).
- Teile, die kleiner als eine Zelle sind, gehen verloren. Dann eine feinere Stufe wählen.

## Was gerechnet wird

- Lattice-Boltzmann D3Q19 mit rekursiv regularisiertem Stoß und Smagorinsky-Turbulenzmodell (Konstante 0,15).
- Der Kanal wird immer in Windrichtung aufgebaut: Bei einer neuen Windrichtung dreht sich das Rechengitter um das feste Modell. Länge etwa 1,2 Modellgrößen vor und 3 Modellgrößen hinter dem Modell, seitlich und oben je 1 Modellgröße.
- Ränder: vorn strömt der Wind ein, seitlich und oben ungestörter Wind, hinten verlässt die Luft den Kanal (mit dämpfender Randzone). Modelle auf dem Boden: bei Fahrzeugen ist der Boden glatt, ohne Reibung (ähnlich wie im Fahrzeug-Windkanal mit Laufband), bei Gebäuden ist der Boden fest und der Wind hat ein Höhenprofil (Potenzgesetz mit Exponent 1/7).
- Einheiten: Geschwindigkeit in m/s, Druck als Unterschied zum Druck der ungestörten Anströmung in Pa (Luft 20 °C, 1,204 kg/m³). Oben rechts steht die Reynoldszahl des echten Objekts und, falls kleiner, die, mit der das Gitter tatsächlich rechnet.
- Die Rechnung überwacht sich selbst: Wird sie instabil, startet sie automatisch mit mehr Dämpfung neu (höchstens dreimal, danach hält sie an und sagt es).
- Selbsttest beim Start: Die Grafikkarte rechnet ein kleines Gitter mit Quader, und das Ergebnis wird mit derselben Rechnung auf dem Prozessor verglichen. Dazu läuft im Leistungstest eine gleichförmige Strömung auf einem größeren Gitter, die gleichförmig bleiben muss. Fällt eine Prüfung durch, erscheint oben ein roter Hinweis; Einzelheiten stehen in der Hilfe („?“) unter „Diese Grafikkarte“.

## Grenzen

Die Seite ist für anschauliche Strömungsbilder und Tendenzen gedacht, nicht für genaue Zahlen. Das Gitter ist für die sehr hohen Reynoldszahlen echter Autos oder Häuser viel zu grob; das Turbulenzmodell sorgt dafür, dass die Rechnung trotzdem stabil bleibt. Bei hohen Geschwindigkeiten ändert sich das Bild deshalb kaum, die Werte in der Legende wachsen aber mit.

## Technik

- Eine HTML-Datei ohne externe Abhängigkeiten. Shader in WGSL, alle 19 Gitterrichtungen ausgeschrieben; die Gittermaße stehen im Uniform-Puffer, die Shader werden also nur einmal übersetzt.
- Speicherbedarf etwa 190 Byte je Zelle (Stufe 5 mit 4 Mio. Zellen: rund 0,8 GB Grafikspeicher). Reicht der Speicher nicht, wird automatisch eine Stufe gröber gerechnet.
