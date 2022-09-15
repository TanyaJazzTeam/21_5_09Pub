---
layout: layouts/blog-post.njk
title: Gewinnen von Sicherheit und Datenschutz durch Partitionieren des Caches
description: |2

  Die HTTP-Cache-Partitionierung von Chrome trägt zu mehr Sicherheit und Datenschutz bei.
authors:
  - Alterktmr
date: '2020-10-06'
updated: '2020-10-24'
---

Im Allgemeinen kann Caching die Leistung verbessern, indem Daten gespeichert werden, sodass zukünftige Anforderungen für dieselben Daten schneller bedient werden. Beispielsweise kann eine zwischengespeicherte Ressource aus dem Netzwerk einen Roundtrip zum Server vermeiden. Ein zwischengespeichertes Berechnungsergebnis kann die Zeit für dieselbe Berechnung auslassen.

In Chrome wird der Cache-Mechanismus auf verschiedene Weise verwendet, und der HTTP-Cache ist ein Beispiel.

## Wie der HTTP-Cache von Chrome derzeit funktioniert

Ab Version 85 speichert Chrome aus dem Netzwerk abgerufene Ressourcen unter Verwendung ihrer jeweiligen Ressourcen-URLs als Cache-Schlüssel. (Ein Cache-Schlüssel wird verwendet, um eine zwischengespeicherte Ressource zu identifizieren.)

Das folgende Beispiel veranschaulicht, wie ein einzelnes Bild zwischengespeichert und in drei verschiedenen Kontexten behandelt wird:

<figure class="float-left">
  {% Img src="image/T4FyVKpzu4WKF1kBNvXepbi08t52/zqkRCKG9jR3uBtcEwPgV.png", alt="Cache Key: https://x.example/doge.png", width="570", height="433" %}
  <figcaption>
    <b>Cache Key</b>: { <code>https://x.example/doge.png</code> }
  </figcaption>
</figure>

Ein Benutzer besucht eine Seite ( `https://a.example` ), die ein Bild anfordert ( `https://x.example/doge.png` ). Das Bild wird vom Netzwerk angefordert und mit `https://x.example/doge.png` als Schlüssel zwischengespeichert.

<figure class="float-left">
  {% Img src="image/T4FyVKpzu4WKF1kBNvXepbi08t52/sXZTOs9iABokE7VsOoXT.png", alt="Cache Key: https://x.example/doge.png", width="570", height="433" %}
  <figcaption>
    <b>Cache Key</b>: { <code>https://x.example/doge.png</code> }
  </figcaption>
</figure>

Derselbe Benutzer besucht eine andere Seite ( `https://b.example` ), die dasselbe Bild anfordert ( `https://x.example/doge.png` ).
 Der Browser überprüft seinen HTTP-Cache, um festzustellen, ob er diese Ressource bereits zwischengespeichert hat, wobei er die Bild-URL als Schlüssel verwendet. Der Browser findet eine Übereinstimmung in seinem Cache und verwendet daher die zwischengespeicherte Version der Ressource.

<figure class="float-left">
  {% Img src="image/T4FyVKpzu4WKF1kBNvXepbi08t52/c8jEGuxXemlwMbezevOc.png", alt="Cache Key: https://x.example/doge.png", width="570", height="433" %}
  <figcaption>
    <b>Cache Key</b>: { <code>https://x.example/doge.png</code> }
  </figcaption>
</figure>

Es spielt keine Rolle, ob das Bild aus einem Iframe geladen wird. Wenn der Benutzer eine andere Website ( `https://c.example` ) mit einem iframe ( `https://d.example` ) besucht und der iframe dasselbe Bild anfordert ( `https://x.example/doge.png` ), wird der Browser kann das Bild trotzdem aus seinem Cache laden, da der Cache-Schlüssel auf allen Seiten gleich ist.

Dieser Mechanismus funktioniert aus Performance-Sicht seit langem gut. Die Zeit, die eine Website benötigt, um auf HTTP-Anforderungen zu antworten, kann jedoch zeigen, dass der Browser in der Vergangenheit auf dieselbe Ressource zugegriffen hat, was den Browser für Sicherheits- und Datenschutzangriffe wie die folgenden anfällig macht:

- **Erkennen, ob ein Benutzer eine bestimmte Website besucht hat** : Ein Angreifer kann den Browserverlauf eines Benutzers erkennen, indem er überprüft, ob der Cache eine Ressource enthält, die für eine bestimmte Website oder Kohorte von Websites spezifisch sein könnte.
- **[Cross-Site Search Attack](https://portswigger.net/daily-swig/new-xs-leak-techniques-reveal-fresh-ways-to-expose-user-information)** : Ein Angreifer kann erkennen, ob sich eine beliebige Zeichenfolge in den Suchergebnissen des Benutzers befindet, indem er überprüft, ob sich ein „Keine Suchergebnisse“-Bild, das von einer bestimmten Website verwendet wird, im Cache des Browsers befindet.
- **Cross-Site-Tracking** : Der Cache kann verwendet werden, um Cookie-ähnliche Kennungen als Cross-Site-Tracking-Mechanismus zu speichern.

Um diese Risiken zu mindern, partitioniert Chrome seinen HTTP-Cache ab Chrome 86.

## Wie wirkt sich die Cache-Partitionierung auf den HTTP-Cache von Chrome aus?

Bei der Cache-Partitionierung werden zwischengespeicherte Ressourcen zusätzlich zur Ressourcen-URL mit einem neuen "Netzwerkisolationsschlüssel" verschlüsselt. Der Netzwerkisolationsschlüssel besteht aus der Website der obersten Ebene und der Website des aktuellen Frames.

{% Aside %} Die "Site" wird anhand von " [scheme://eTLD+1](https://web.dev/same-site-same-origin/) " erkannt, wenn also Anfragen von verschiedenen Seiten kommen, aber sie dasselbe Schema und dieselbe effektive Top-Level-Domain +1 haben, verwenden sie dieselbe Cache-Partition . Um mehr darüber zu erfahren, lesen [Sie Erläuterungen zu „gleicher Website“ und „gleichem Ursprung“](https://web.dev/same-site-same-origin/) . {% endAside %}

Sehen Sie sich noch einmal das vorherige Beispiel an, um zu sehen, wie die Cache-Partitionierung in verschiedenen Kontexten funktioniert:

<figure class="float-left">
  {% Img src="image/T4FyVKpzu4WKF1kBNvXepbi08t52/zqkRCKG9jR3uBtcEwPgV.png", alt="Cache Key { https://a.example, https://a.example, https://x.example/doge.png}", width="570", height="433" %}
  <figcaption>
    <b>Cache Key</b>: { <code>https://a.example</code>, <code>https://a.example</code>, <code>https://x.example/doge.png</code> }
  </figcaption>
</figure>

Ein Benutzer besucht eine Seite ( `https://a.example` ), die ein Bild anfordert ( `https://x.example/doge.png` ). In diesem Fall wird das Bild vom Netzwerk angefordert und mithilfe eines Tupels zwischengespeichert, das aus `https://a.example` (der Website der obersten Ebene), `https://a.example` (der Website des aktuellen Frames) und `https://x.example/doge.png` besteht `https://x.example/doge.png` (die Ressourcen-URL) als Schlüssel. (Beachten Sie, dass, wenn die Ressourcenanforderung vom Frame der obersten Ebene stammt, die Website der obersten Ebene und die Website des aktuellen Frames im Netzwerkisolationsschlüssel identisch sind.)

<figure class="float-left">
  {% Img src="image/T4FyVKpzu4WKF1kBNvXepbi08t52/sXZTOs9iABokE7VsOoXT.png", alt="Cache Key { https://a.example, https://a.example, https://x.example/doge.png}", width="570", height="433" %}
  <figcaption>
    <b>Cache Key</b>: { <code>https://b.example</code>, <code>https://b.example</code>, <code>https://x.example/doge.png</code> }
  </figcaption>
</figure>

Derselbe Benutzer besucht eine andere Seite ( `https://b.example` ), die dasselbe Bild anfordert ( `https://x.example/doge.png` ). Obwohl das gleiche Bild im vorherigen Beispiel geladen wurde, wird es kein Cache-Treffer sein, da der Schlüssel nicht übereinstimmt.

Das Bild wird vom Netzwerk angefordert und mithilfe eines Tupels zwischengespeichert, das aus `https://b.example` , `https://b.example` und `https://x.example/doge.png` als Schlüssel besteht.

<figure class="float-left">
  {% Img src="image/T4FyVKpzu4WKF1kBNvXepbi08t52/kr9TtbDQPQfR86rNSrJX.png", alt="Cache Key { https://a.example, https://a.example, https://x.example/doge.png}", width="570", height="433" %}
  <figcaption>
    <b>Cache Key</b>: { <code>https://a.example</code>, <code>https://a.example</code>, <code>https://x.example/doge.png</code> }
  </figcaption>
</figure>

Jetzt kommt der Benutzer zurück zu `https://a.example` , aber dieses Mal ist das Bild ( `https://x.example/doge.png` ) in einen Iframe eingebettet. In diesem Fall ist der Schlüssel ein Tupel, das `https://a.example` , `https://a.example` und `https://x.example/doge.png` enthält, und es kommt zu einem Cache-Treffer. (Beachten Sie, dass, wenn die Website der obersten Ebene und der Iframe dieselbe Website sind, die mit dem Frame der obersten Ebene zwischengespeicherte Ressource verwendet werden kann.

<div class="clearfix"></div>

<figure class="float-left">
  {% Img src="image/T4FyVKpzu4WKF1kBNvXepbi08t52/BIJfNKd7YfXuXR3xdafb.png", alt="Cache Key { https://a.example, https://a.example, https://x.example/doge.png}", width="570", height="433" %}
  <figcaption>
    <b>Cache Key</b>: { <code>https://a.example</code>, <code>https://c.example</code>, <code>https://x.example/doge.png</code> }
  </figcaption>
</figure>

Der Benutzer befindet sich wieder unter `https://a.example` , aber dieses Mal wird das Bild in einem Iframe von `https://c.example` .

In diesem Fall wird das Bild aus dem Netzwerk heruntergeladen, da im Cache keine Ressource vorhanden ist, die mit dem Schlüssel übereinstimmt, der aus `https://a.example` , `https://c.example` und `https://x.example/doge.png` .

<figure class="float-left">
  {% Img src="image/T4FyVKpzu4WKF1kBNvXepbi08t52/Gg99hTwbcxc3DdUtgdnM.png", alt="Cache Key { https://a.example, https://a.example, https://x.example/doge.png}", width="570", height="433" %}
  <figcaption>
    <b>Cache Key</b>: { <code>https://a.example</code>, <code>https://c.example</code>, <code>https://x.example/doge.png</code> }
  </figcaption>
</figure>

Was ist, wenn die Domain eine Subdomain oder eine Portnummer enthält? Der Benutzer besucht `https://subdomain.a.example` , das einen Iframe ( `https://c.example:8080` ) einbettet, der das Bild anfordert.

Da der Schlüssel basierend auf „scheme://eTLD+1“ erstellt wird, werden Subdomains und Portnummern ignoriert. Daher tritt ein Cache-Hit auf.

<figure class="float-left">
  {% Img src="image/T4FyVKpzu4WKF1kBNvXepbi08t52/47mGHE7I9qlFpPER12CL.png", alt="Cache Key { https://a.example, https://a.example, https://x.example/doge.png}", width="570", height="433" %}
  <figcaption>
    <b>Cache Key</b>: { <code>https://a.example</code>, <code>https://c.example</code>, <code>https://x.example/doge.png</code> }
  </figcaption>
</figure>

Was ist, wenn der Iframe mehrfach verschachtelt ist? Der Benutzer besucht `https://a.example` , das einen iframe ( `https://b.example` ) einbettet, der wiederum einen weiteren iframe ( `https://c.example` ) einbettet, der schließlich das Bild anfordert.

Da der Schlüssel aus dem oberen Frame ( `https://a.example` ) und dem unmittelbaren Frame, der die Ressource lädt ( `https://c.example` ), entnommen wird, tritt ein Cache-Treffer auf.

## Häufig gestellte Fragen

### Ist es in meinem Chrome bereits aktiviert? Wie kann ich das überprüfen?

Die Funktion wird bis Ende 2020 eingeführt. So überprüfen Sie, ob Ihre Chrome-Instanz sie bereits unterstützt:

1. Öffnen Sie `chrome://net-export/` und drücken **Sie Start Logging to Disk** .
2. Geben Sie an, wo die Protokolldatei auf Ihrem Computer gespeichert werden soll.
3. Surfen Sie eine Minute lang mit Chrome im Internet.
4. Gehen Sie zurück zu `chrome://net-export/` und drücken **Sie Stop Logging** .
5. Gehen Sie zu `https://netlog-viewer.appspot.com/#import` .
6. Drücken **Sie Datei auswählen** und übergeben Sie die gespeicherte Protokolldatei.

Sie sehen die Ausgabe der Protokolldatei.

Suchen Sie auf derselben Seite `SplitCacheByNetworkIsolationKey` . Wenn darauf `Experiment_[****]` folgt, ist die HTTP-Cache-Partitionierung auf Ihrem Chrome aktiviert. Wenn darauf `Control_[****]` oder `Default_[****]` folgt, ist es nicht aktiviert.

### Wie kann ich die HTTP-Cache-Partitionierung auf meinem Chrome testen?

Um die HTTP-Cache-Partitionierung auf Ihrem Chrome zu testen, müssen Sie Chrome mit einem Befehlszeilen-Flag starten: `--enable-features=SplitCacheByNetworkIsolationKey` . Befolgen Sie die Anweisungen unter [Chromium mit Flags ausführen](https://www.chromium.org/developers/how-tos/run-chromium-with-flags) , um zu erfahren, wie Sie Chrome mit einem Befehlszeilen-Flag auf Ihrer Plattform starten.

### Muss ich als Webentwickler auf diese Änderung reagieren?

Dies ist keine bahnbrechende Änderung, kann jedoch Leistungsüberlegungen für einige Webdienste mit sich bringen.

Zum Beispiel können diejenigen, die große Mengen an Ressourcen mit hoher Cache-Fähigkeit auf vielen Websites bereitstellen (z. B. Schriftarten und beliebte Skripte), einen Anstieg ihres Datenverkehrs feststellen. Auch diejenigen, die solche Dienste nutzen, können sich zunehmend auf sie verlassen.

(Es gibt einen Vorschlag, gemeinsam genutzte Bibliotheken unter Wahrung der Privatsphäre zu ermöglichen, genannt [Web Shared Libraries](https://docs.google.com/document/d/1lQykm9HgzkPlaKXwpQ9vNc3m2Eq2hF4TY-Vup5wg4qg/edit#) ( [Präsentationsvideo](https://www.youtube.com/watch?v=cBY3ZcHifXw) ), aber er wird noch geprüft.)

### Welche Auswirkungen hat diese Verhaltensänderung?

Die Gesamt-Cache-Miss-Rate steigt um etwa 3,6 %, Änderungen am FCP (First Contentful Paint) sind bescheiden (~0,3 %) und der Gesamtanteil der aus dem Netzwerk geladenen Bytes steigt um etwa 4 %. Weitere Informationen zu den Auswirkungen auf die Leistung finden Sie in [der Erläuterung zur HTTP-Cache-Partitionierung](https://github.com/shivanigithub/http-cache-partitioning#impact-on-metrics) .

### Ist das standardisiert? Verhalten sich andere Browser anders?

"HTTP-Cache-Partitionen" ist [in der Abrufspezifikation standardisiert,](https://fetch.spec.whatwg.org/#http-cache-partitions) obwohl sich Browser unterschiedlich verhalten:

- **Chrome** : Verwendet Schema der obersten Ebene: //eTLD+1 und Rahmenschema: //eTLD+1
- **Safari** : Verwendet [eTLD+1 der obersten Ebene](https://webkit.org/blog/8613/intelligent-tracking-prevention-2-1/)
- **Firefox** : [Planung der Implementierung](https://bugzilla.mozilla.org/show_bug.cgi?id=1536058) mit Top-Level-Schema://eTLD+1 und Erwägung eines zweiten Schlüssels wie Chrome

### Wie wird das Holen von Arbeitnehmern behandelt?

Engagierte Mitarbeiter verwenden denselben Schlüssel wie ihr aktueller Frame. Service Worker und Shared Worker sind komplizierter, da sie von mehreren Top-Level-Standorten gemeinsam genutzt werden können. Die Lösung für sie wird derzeit diskutiert.

## Ressourcen

- [Speicherisolationsprojekt](https://docs.google.com/document/d/1V8sFDCEYTXZmwKa_qWUfTVNAuBcPsu6FC0PhqMD6KKQ/edit#heading=h.oixrt0wpp8h5)
- [Explainer – Partitionieren Sie den HTTP-Cache](https://github.com/shivanigithub/http-cache-partitioning)
