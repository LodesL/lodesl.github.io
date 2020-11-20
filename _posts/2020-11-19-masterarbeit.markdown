---
layout: post
title:  "Meine Masterarbeit: In-Situ Überwachung von Composite-Herstellungsprozessen mit tiefen neuronalen Netzen auf der Basis von 3D-Repräsentationen"
date:   2020-10-30 16:43:33 +0100
categories: jekyll update
---

![3D-Bauteil](/assets/part_full.png)

# Motivation

Meine Masterarbeit baut auf der bisherigen Forschung im Bereich der automatisierten Qualitätssicherung von Carbonteilen am ISSE der Uni Augsburg auf. Das Forschungsziel wird in [dieser Publikation](https://www.researchgate.net/publication/326423156_Transfer_Learning_for_Optimization_of_Carbon_Fiber_Reinforced_Polymer_Production) genauer erläutert. Die bisherigen Forschungsergebnisse sind in [diesem Paper](https://www.researchgate.net/publication/341908239_FlowFrontNet_Improving_Carbon_Composite_Manufacturing_with_CNNs) veröffentlicht worden.

Die Vision ist es, eine Vorhersage über die Qualität eines Carbonteils schon während der Produktion zu ermöglichen. Durch das Einbringen von Sensoren direkt in das Werkzeug können während der Herstellung große Datenmengen gesammelt werden, die dann wiederum mit tiefen neuronalen Netzen weiterverarbeitet werden. 

Bisher wurde allerdings lediglich ein flaches Bauteil betrachtet. Um die Forschungsergebnisse näher an die Realität zu bringen, soll ich daher in meiner Masterarbeit ein Bauteil mit dreidimensionalen Elementen betrachten. Das von mir verwendete Bauteil ist im Screenshot am Anfang zu sehen.

Eine weitere Motivation der Arbeit ist die Verwendung von 3D-Repräsentationen zum Lernen. Da bisher nur Sensorwerte isoliert als Vektor oder Matrix den neuronalen Netzen als Input gegeben wurden, soll ich in meiner Masterarbeit direkt auf einer Graphrepräsentation lernen. Ermöglicht wird das durch Graph Neural Networks, mit denen zuletzt große Fortschritte im Bereich von Physiksimulationen erzielt werden konnten. 

# Datenerzeugung

Die am ISSE betrachteten Bauteile werden mit dem [RTM-Verfahren (Resin Transfer Molding)](https://en.wikipedia.org/wiki/Transfer_molding) hergestellt. Bei diesem Prozess wird ein flüssiges Harz in vorgeformte Fasern mit großem Druck eingespritzt. Durch die Vermischung mit den Fasern und das anschließende Aushärten können auf diese Weise komplexe CFK-Bauteile hergestellt werden. Die Vorteile dieses Verfahrens sind u.a. eine hohe Automatisierbarkeit und Rentabilität. Der RTM-Prozess ist allerdings relativ Fehleranfällig, wodurch der Aufwand für die Qualitätssicherung für die Großserienfertigung zu hoch ist. Deshalb soll auch die Qualitätssicherung automatisiert werden. Die bisherige Forschung hat sich hauptsächlich auf die Detektion von sog. Dryspots konzentriert, also Stellen im Textil, die nicht vom Harz erreicht wurden. Solche Stellen führen im ausgehärteten Bauteil Instabilitäten. 

Da die Menge an Echtdaten sehr gering ist, werden zum Lernen simulierte Daten verwendet. Zur Simulation wird [PAM-RTM von ESI Systems](https://www.esi-group.com/products/composites) verwendet. Um Dryspots gezielt zu erzeugen, wurde für das Bauteil aus der [verlinkten](https://www.researchgate.net/publication/341908239_FlowFrontNet_Improving_Carbon_Composite_Manufacturing_with_CNNs) Publikation  der Faservolumengehalt an zufällig gewählten Stellen erhöht, sodass das Harz die Fasern an diesen Stellen schlechter durchtränken kann. Je nach Größe der Veränderung bleiben diese Stellen auch nach Ende der Harzinjektion trocken.

Diese Methode habe ich auf das neue, dreidimensionale Bauteil übertragen. Hiermit konnte der Herstellungsprozess 10 000 Mal simuliert werden und bildet so eine gute Datengrundlage

# Ansatz
Da das Bauteil in der Simulation als Triangle Mesh repräsentiert ist, war die Verwendung dieser 3D-Repräsentation naheliegend. Das Mesh ist in folgendem Detail-Screenshot zu sehen: 

![Mesh](/assets/mesh_details2.png)

Die Sensorwerte werden auf den Knoten platziert, die nächste Nachbarn zu den jeweiligen Sensoren sind. Da in der Simulation relativ viele Sensoren platziert sind (1 cm Abstand zwischen den Sensoren; 4290 Sensoren insgesamt), können Experimente mit verschiedenen Sensorabständen durchgeführt werden der

Mein Modell besteht aus gestapelten Graph Convolutional Layers, die die Information durch das Mesh propagieren und dabei Features extrahieren. 
Graph Covolutional Layers sind wie grundsätzlich folgt definiert: 

![GCN](/assets/gcn.png)

wobei A die Adjazenzmatrix des Meshes und W ein lernbare Gewichtsmatrix ist. In meiner Arbeit habe ich die [Erweiterung von Kipf et al.](https://tkipf.github.io/graph-convolutional-networks/) verwendet. 

Das Lernziel ist die Vorhersage des Harzfüllstandes an jedem Knoten im Mesh (auch Fließfront genannt). Die so gewonnene Bildrepräsentation kann dann mit Convolutional Neural Netzworks auf Dryspots analysiert werden. 

# Vorläufige Ergebnisse 

Analog zum bereits veröffentlichten Paper konnte ich feststellen, dass die Vorhersagequalität stark abhängig von der Anzahl der Sensoren ist. Mit den vollen 4290 platzierten Sensoren konnte die Fließfront gut rekonstruiert werden, wie die folgenden Screenshots zeigen.

Output: ![4290_out](/assets/4290_ff_out.png)  

Label:  ![4290_label](/assets/4290_ff.png)

Je weniger Sensoren und somit Informationen allerdings im Mesh verfügbar sind, desto schlechter wird auch die Vorhersage der Fließfront: 

Output: ![290_out](/assets/290_ff_out.png)

Label: ![290_label](/assets/290_ff_out.png)


Diese Bilder sind für die Analyse auf Dryspots natürlich nicht zu gebrauchen. Dennoch sind die Ergebnisse nicht durchweg negativ, da die Rekonstruktion mit vielen Sensoren qualitativ ähnlich zum bisherigen Ansatz sind, obwohl das verwendete neuronale Netz deutlich kleiner ist. Während das bisherige Netz mehrere Millionen Parameter zum Optimieren hat, müssen für mein Graph Convolutional Network nur ca. 10 000 Parameter optimiert werden.

# Fazit und Ausblick

Auch wenn die vorläufigen Ergebnisse der Arbeit weniger gut als erhofft sind, ist die Arbeit dennoch eine gute Basis für weitere Forschungsarbeiten. Neben den erzeugten neuen Daten wurde hier der erste Schritt mit Graph Neural Networks gemacht, die sich in der Literatur als sehr vielversprechend zeigen. Besonders die Netzarchitektur bietet noch viel Potential zur Optimierung. Ein Beispiel hierfür ist eine Encoder-Processor-Decododer Architektur wie von [Pfaff et al.](https://arxiv.org/abs/2010.03409). Eine besonderes Augenmerk sollte dabei auf dem Encoding-Schritt liegen, um mehr Informationen in den Graph zu bringen. Außerdem ist der Aspekt des Remeshings sehr interessant. Das verwendete Mesh ist für die Simulation sehr fein aufgelöst, was allerdings den Informationsfluss beim Lernen behindert. Eine neue Mesh-Struktur die sowohl für die Simulation als auch den Lernprozess positive Eigenschaften besitzt wäre daher erstrebenswert.
