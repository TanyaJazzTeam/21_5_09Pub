---
"$title": AMP for Ads-Spezifikation
order: '3'
formats:
  - Anzeigen
teaser:
  text: |2-

    _Wenn Sie Änderungen am Standard vorschlagen möchten, kommentieren Sie bitte die
    [Absicht
toc: wahr
---

<!--
This file is imported from https://github.com/ampproject/amphtml/blob/master/extensions/amp-a4a/amp-a4a-format.md.
Please do not change this file.
If you have found a bug or an issue please
have a look and request a pull request there.
-->

<!---
Copyright 2016 The AMP HTML Authors. All Rights Reserved.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS-IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->

*Wenn Sie Änderungen am Standard vorschlagen möchten, kommentieren Sie bitte die [Absicht zur Implementierung](https://github.com/ampproject/amphtml/issues/4264)* .

## AMPHTML-Anzeigenformatregeln<a name="amphtml-ad-format-rules"></a>

<table>
<thead><tr>
  <th>Regel</th>
  <th>Begründung</th>
</tr></thead>
<tbody>
<tr>
<td>Muss <code>&lt;html ⚡4ads&gt;</code> oder <code>&lt;html amp4ads&gt;</code> als umschließende Tags verwenden.</td>
<td>Ermöglicht Validierern, ein Creative-Dokument entweder als allgemeines AMP-Dokument oder als eingeschränktes AMPHTML-Anzeigendokument zu identifizieren und entsprechend zu versenden.</td>
</tr>
<tr>
<td>Muss <code>&lt;script async src="https://cdn.ampproject.org/amp4ads-v0.js"&gt;&lt;/script&gt;</code> als Laufzeitskript anstelle von <code>https://cdn.ampproject.org/v0.js</code> enthalten.</td>
<td>Ermöglicht angepasstes Laufzeitverhalten für AMPHTML-Anzeigen, die in Cross-Origin-iFrames geliefert werden.</td>
</tr>
<tr>
<td>Darf kein <code>&lt;link rel="canonical"&gt;</code> Tag enthalten.</td>
<td>Anzeigen-Creatives haben keine „nicht-AMP-kanonische Version“ und werden nicht unabhängig suchindiziert, sodass eine Selbstreferenzierung nutzlos wäre.</td>
</tr>
<tr>
<td>Kann optionale Meta-Tags im HTML-Kopf als Kennungen im Format <code>&lt;meta name="amp4ads-id" content="vendor=${vendor},type=${type},id=${id}"&gt;</code> . Diese Meta-Tags müssen vor dem <code>amp4ads-v0.js</code> platziert werden. Der Wert von <code>vendor</code> und <code>id</code> sind Zeichenfolgen, die nur [0-9a-zA-Z_-] enthalten. Der Wert von <code>type</code> ist entweder <code>creative-id</code> oder <code>impression-id</code> .</td>
<td>Diese benutzerdefinierten Kennungen können verwendet werden, um die Impression oder das Motiv zu identifizieren. Sie können beim Berichten und Debuggen hilfreich sein.<br><br><p> Beispiel:</p>
<pre>
&lt;meta name="amp4ads-id"
content="vendor=adsense,type=creative-id,id=1283474"&gt;
&lt;meta name="amp4ads-id"
content="vendor=adsense,type=impression-id,id=xIsjdf921S"&gt;</pre>
</td>
</tr>
<tr>
<td>
<code>&lt;amp-analytics&gt;</code> Sichtbarkeits-Tracking darf nur auf den vollständigen Anzeigenselektor abzielen, über <code>"visibilitySpec": { "selector": "amp-ad" }</code> wie in <a href="https://github.com/ampproject/amphtml/issues/4018">Issue #4018</a> und <a href="https://github.com/ampproject/amphtml/pull/4368">PR #4368</a> definiert. Insbesondere darf es nicht auf Selektoren für Elemente innerhalb des Werbemittels abzielen.</td>
<td>In einigen Fällen entscheiden sich AMPHTML-Anzeigen möglicherweise dafür, eine Anzeige in einem Iframe zu rendern. In diesen Fällen kann die Analyse der Hostseite ohnehin nur auf den gesamten Iframe abzielen und hat keinen Zugriff auf feinkörnigere Selektoren.<br><br><p> Beispiel:</p>
<pre>
&lt;amp-analytics id="nestedAnalytics"&gt;
&lt;script type="application/json"&gt;
{
"Anfragen": {
"Sichtbarkeit": "https://example.com/nestedAmpAnalytics"
},
"löst aus": {
"Sichtbarkeitsspezifikation": {
"selector": "amp-ad",
"visiblePercentageMin": 50,
"continuousTimeMin": 1000
}
}
}
&lt;/script&gt;
&lt;/amp-analytics&gt;
</pre>
<p>Diese Konfiguration sendet eine Anfrage an die URL <code>https://example.com/nestedAmpAnalytics</code> , wenn 50 % der umschließenden Anzeige 1 Sekunde lang kontinuierlich auf dem Bildschirm sichtbar waren.</p>
</td>
</tr>
</tbody>
</table>

### Boilerplate<a name="boilerplate"></a>

AMPHTML-Anzeigen-Creatives erfordern eine andere und wesentlich einfachere Boilerplate-Stillinie als [allgemeine AMP-Dokumente](https://github.com/ampproject/amphtml/blob/master/spec/amp-boilerplate.md) :

[sourcecode:html]
<style amp4ads-boilerplate>
  body {
    visibility: hidden;
  }
</style>
[/sourcecode]

*Begründung:* Der `amp-boilerplate` Stil verbirgt Body-Inhalte, bis die AMP-Laufzeit bereit ist und sie wieder einblenden kann. Wenn Javascript deaktiviert ist oder die AMP-Laufzeit nicht geladen werden kann, sorgt die Standard-Boilerplate dafür, dass der Inhalt letztendlich trotzdem angezeigt wird. In AMPHTML-Anzeigen werden jedoch keine AMPHTML-Anzeigen geschaltet, wenn Javascript vollständig deaktiviert ist, und es wird niemals eine Anzeige gezeigt, sodass der Abschnitt `<noscript>` nicht erforderlich ist. In Ermangelung der AMP-Laufzeit sind die meisten Mechanismen, auf die sich AMPHTML-Anzeigen verlassen (z. B. Analysen zur Sichtbarkeitsverfolgung oder `amp-img` für die Anzeige von Inhalten), nicht verfügbar, daher ist es besser, keine Anzeige anzuzeigen als eine fehlerhafte.

Schließlich verwendet die AMPHTML-Anzeigen-Boilerplate `amp-a4a-boilerplate` anstelle von `amp-boilerplate` , damit Validierer sie leicht identifizieren und genauere Fehlermeldungen erstellen können, um Entwicklern zu helfen.

Beachten Sie, dass die gleichen Regeln für Mutationen am Boilerplate-Text gelten wie im [allgemeinen AMP-Boilerplate](https://github.com/ampproject/amphtml/blob/master/spec/amp-boilerplate.md) .

### CSS<a name="css"></a>

<table>
<thead><tr>
  <th>Regel</th>
  <th>Begründung</th>
</tr></thead>
<tbody>
  <tr>
    <td>
<code>position:fixed</code> und <code>position:sticky</code> sind in Creative-CSS verboten.</td>
    <td>
<code>position:fixed</code> bricht aus Schatten-DOM aus, von dem AMPHTML-Anzeigen abhängen. Außerdem dürfen Anzeigen in AMP bereits keine feste Position verwenden.</td>
  </tr>
  <tr>
    <td>
<code>touch-action</code> ist verboten.</td>
    <td>Eine Anzeige, die <code>touch-action</code> manipulieren kann, kann die Fähigkeit des Benutzers beeinträchtigen, durch das Hostdokument zu scrollen.</td>
  </tr>
  <tr>
    <td>Creative-CSS ist auf 20.000 Byte begrenzt.</td>
    <td>Große CSS-Blöcke blähen das Creative auf, erhöhen die Netzwerklatenz und verschlechtern die Seitenleistung.</td>
  </tr>
  <tr>
    <td>Übergang und Animation unterliegen zusätzlichen Einschränkungen.</td>
    <td>AMP muss in der Lage sein, alle zu einer Anzeige gehörenden Animationen zu steuern, damit sie gestoppt werden können, wenn die Anzeige nicht auf dem Bildschirm zu sehen ist oder die Systemressourcen sehr niedrig sind.</td>
  </tr>
  <tr>
    <td>Herstellerspezifische Präfixe werden zu Validierungszwecken als Aliase für dasselbe Symbol ohne das Präfix betrachtet. Das bedeutet, wenn ein Symbol <code>foo</code> durch CSS-Validierungsregeln verboten ist, dann wird auch das Symbol <code>-vendor-foo</code> verboten.</td>
    <td>Einige Eigenschaften mit Anbieterpräfix bieten eine gleichwertige Funktionalität wie Eigenschaften, die ansonsten gemäß diesen Regeln verboten oder eingeschränkt sind.<br><br><p> Beispiel: <code>-webkit-transition</code> und <code>-moz-transition</code> <code>transition</code> beide als Aliase für transit angesehen. Sie sind nur in Kontexten zulässig, in denen ein bloßer <code>transition</code> zulässig wäre (siehe Abschnitt <a href="#selectors">Selektoren</a> weiter unten).</p>
</td>
  </tr>
</tbody>
</table>

#### CSS-Animationen und Übergänge<a name="css-animations-and-transitions"></a>

##### Selektoren<a name="selectors"></a>

Die `transition` und `animation` sind nur für Selektoren zulässig, die:

- Nur `transition` , `animation` , `transform` , `visibility` oder `opacity` enthalten.

    *Begründung:* Dadurch kann die AMP-Laufzeit diese Klasse aus dem Kontext entfernen, um Animationen zu deaktivieren, wenn dies für die Seitenleistung erforderlich ist.

**Gut**

[sourcecode:css]
.box {
  transform: rotate(180deg);
  transition: transform 2s;
}
[/sourcecode]

**Schlecht**

Eigenschaft in CSS-Klasse nicht zulässig.

[sourcecode:css]
.box {
  color: red; // non-animation property not allowed in animation selector
  transform: rotate(180deg);
  transition: transform 2s;
}
[/sourcecode]

##### Übergangsfähige und animierbare Eigenschaften<a name="transitionable-and-animatable-properties"></a>

Die einzigen Eigenschaften, die geändert werden können, sind Opazität und Transformation. ( [Begründung](http://www.html5rocks.com/en/tutorials/speed/high-performance-animations/) )

**Gut**

[sourcecode:css]
transition: transform 2s;
[/sourcecode]

**Schlecht**

[sourcecode:css]
transition: background-color 2s;
[/sourcecode]

**Gut**

[sourcecode:css]
@keyframes turn {
  from {
    transform: rotate(180deg);
  }

  to {
    transform: rotate(90deg);
  }
}
[/sourcecode]

**Schlecht**

[sourcecode:css]
@keyframes slidein {
  from {
    margin-left: 100%;
    width: 300%;
  }

  to {
    margin-left: 0%;
    width: 100%;
  }
}
[/sourcecode]

### Zulässige AMP-Erweiterungen und -Integrierte<a name="allowed-amp-extensions-and-builtins"></a>

Die folgenden *zulässigen* AMP-Erweiterungsmodule und integrierten AMP-Tags in einem AMPHTML-Anzeigen-Creative. Erweiterungen oder eingebaute Tags, die nicht ausdrücklich aufgeführt sind, sind verboten.

- [Amp-Akkordeon](https://amp.dev/documentation/components/amp-accordion)
- [amp-ad-exit](https://amp.dev/documentation/components/amp-ad-exit)
- [amp-analytik](https://amp.dev/documentation/components/amp-analytics)
- [amp-anim](https://amp.dev/documentation/components/amp-anim)
- [Amp-Animation](https://amp.dev/documentation/components/amp-animation)
- [Verstärker-Audio](https://amp.dev/documentation/components/amp-audio)
- [amp-bind](https://amp.dev/documentation/components/amp-bind)
- [Amp-Karussell](https://amp.dev/documentation/components/amp-carousel)
- [amp-fit-text](https://amp.dev/documentation/components/amp-fit-text)
- [amp-Schriftart](https://amp.dev/documentation/components/amp-font)
- [Amp-Form](https://amp.dev/documentation/components/amp-form)
- [amp-img](https://amp.dev/documentation/components/amp-img)
- [Verstärker-Layout](https://amp.dev/documentation/components/amp-layout)
- [Amp-Leuchtkasten](https://amp.dev/documentation/components/amp-lightbox)
- amp-mraid, auf experimenteller Basis. Wenn Sie erwägen, dies zu verwenden, öffnen Sie bitte ein Problem bei [wg-ads](https://github.com/ampproject/wg-ads/issues/new) .
- [Amp-Schnurrbart](https://amp.dev/documentation/components/amp-mustache)
- [Amp-Pixel](https://amp.dev/documentation/components/amp-pixel)
- [Verstärker-Position-Beobachter](https://amp.dev/documentation/components/amp-position-observer)
- [Verstärker-Wahlschalter](https://amp.dev/documentation/components/amp-selector)
- [amp-Social-Share](https://amp.dev/documentation/components/amp-social-share)
- [amp-video](https://amp.dev/documentation/components/amp-video)

Die meisten Auslassungen dienen entweder der Leistung oder der einfacheren Analyse von AMPHTML-Anzeigen.

*Beispiel:* `<amp-ad>` wird in dieser Liste weggelassen. Dies ist ausdrücklich nicht zulässig, da das Zulassen einer `<amp-ad>` innerhalb einer `<amp-ad>` möglicherweise zu unbegrenzten Wasserfällen beim Laden von Anzeigen führen könnte, was die Leistungsziele von AMPHTML-Anzeigen nicht erfüllt.

*Beispiel:* `<amp-iframe>` wird in dieser Liste weggelassen. Es ist nicht zulässig, da Anzeigen damit beliebiges Javascript ausführen und beliebige Inhalte laden könnten. Anzeigen, die solche Funktionen nutzen möchten, sollten von ihrem [a4aRegistry-](https://github.com/ampproject/amphtml/blob/master/ads/_a4a-config.js#L40) Eintrag „ `false` “ zurückgeben und den vorhandenen „3p-Iframe“-Anzeigerendering-Mechanismus verwenden.

*Beispiel:* `<amp-facebook>` , `<amp-instagram>` , `<amp-twitter>` und `<amp-youtube>` werden alle aus dem gleichen Grund weggelassen wie `<amp-iframe>` : Sie alle erstellen Iframes und können potenziell unbegrenzte Ressourcen verbrauchen in ihnen.

*Beispiel:* `<amp-ad-network-*-impl>` werden in dieser Liste weggelassen. Das `<amp-ad>` -Tag übernimmt die Delegierung an diese Implementierungs-Tags; Creatives sollten nicht versuchen, sie direkt einzuschließen.

*Beispiel:* `<amp-lightbox>` ist noch nicht enthalten, da sogar einige AMPHTML-Anzeigen-Creatives in einem Iframe gerendert werden können und es derzeit keinen Mechanismus gibt, mit dem eine Anzeige über einen Iframe hinaus erweitert werden kann. Dies kann in Zukunft unterstützt werden, wenn der Wunsch danach nachgewiesen wird.

### HTML-Tags<a name="html-tags"></a>

Die folgenden Tags sind in einem AMPHTML-Anzeigen-Creative *zulässig* . Nicht ausdrücklich erlaubte Tags sind verboten. Diese Liste ist eine Teilmenge der allgemeinen [AMP-Tag-Zulassungsliste](https://github.com/ampproject/amphtml/blob/master/extensions/amp-a4a/../../spec/amp-tag-addendum.md) . Wie diese Liste ist sie gemäß der HTML5-Spezifikation in Abschnitt 4 [Die Elemente von HTML](http://www.w3.org/TR/html5/single-page.html#html-elements) geordnet.

Die meisten Auslassungen sind entweder aus Leistungsgründen oder weil die Tags nicht dem HTML5-Standard entsprechen. Beispielsweise wird `<noscript>` weggelassen, weil AMPHTML-Anzeigen von der Aktivierung von JavaScript abhängig sind, sodass ein `<noscript>` -Block niemals ausgeführt wird und daher nur das Creative aufbläht und Bandbreite und Latenz kostet. Ebenso `<acronym>` , `<big>` , et al. sind verboten, da sie nicht HTML5-kompatibel sind.

#### 4.1 Das Wurzelelement<a name="41-the-root-element"></a>

4.1.1 `<html>`

- Muss die Typen `<html ⚡4ads>` oder `<html amp4ads>`

#### 4.2 Dokumentmetadaten<a name="42-document-metadata"></a>

4.2.1 `<head>`

4.2.2 `<title>`

4.2.4 `<link>`

- `<link rel=...>` -Tags sind nicht erlaubt, mit Ausnahme von `<link rel=stylesheet>` .

- **Hinweis:** Im Gegensatz zu allgemeinem AMP sind `<link rel="canonical">` -Tags verboten.

    4.2.5 `<style>` 4.2.6 `<meta>`

#### 4.3 Abschnitte<a name="43-sections"></a>

4.3.1 `<body>` 4.3.2 `<article>` 4.3.3 `<section>` 4.3.4 `<nav>` 4.3.5 `<aside>` 4.3.6 `<h1>` , `<h2>` , `<h3>` , `<h4>` , `<h5>` und `<h6>` 4.3.7 `<header>` 4.3.8 `<footer>` 4.3.9 `<address>`

#### 4.4 Gruppieren von Inhalten<a name="44-grouping-content"></a>

4.4.1 `<p>` 4.4.2 `<hr>` 4.4.3 `<pre>` 4.4.4 `<blockquote>` 4.4.5 `<ol>` 4.4.6 `<ul>` 4.4.7 `<li>` 4.4.8 `<dl>` 4.4. 9 `<dt>` 4.4.10 `<dd>` 4.4.11 `<figure>` 4.4.12 `<figcaption>` 4.4.13 `<div>` 4.4.14 `<main>`

#### 4.5 Semantik auf Textebene<a name="45-text-level-semantics"></a>

4.5.1 `<a>` 4.5.2 `<em>` 4.5.3 `<strong>` 4.5.4 `<small>` 4.5.5 `<s>` 4.5.6 `<cite>` 4.5.7 `<q>` 4.5.8 `<dfn>` 4.5. 9 `<abbr>` 4.5.10 `<data>` 4.5.11 `<time>` 4.5.12 `<code>` 4.5.13 `<var>` 4.5.14 `<samp>` 4.5.15 `<kbd >` 4.5.16 `<sub>` und `<sup>` 4.5.17 `<i>` 4.5.18 `<b>` 4.5.19 `<u>` 4.5.20 `<mark>` 4.5.21 `<ruby>` 4.5.22 `<rb>` 4.5.23 `<rt>` 4.5.24 `<rtc>` 4.5. 25 `<rp>` 4.5.26 `<bdi>` 4.5.27 `<bdo>` 4.5.28 `<span>` 4.5.29 `<br>` 4.5.30 `<wbr>`

#### 4.6 Bearbeitungen<a name="46-edits"></a>

4.6.1 `<ins>` 4.6.2 `<del>`

#### 4.7 Eingebettete Inhalte<a name="47-embedded-content"></a>

- Eingebettete Inhalte werden nur über AMP-Tags wie `<amp-img>` oder `<amp-video>` unterstützt.

#### 4.7.4 `<source>`<a name="474-source"></a>

<!---
#### 4.7.18 SVG <a name="4718-svg"></a>

SVG tags are not in the HTML5 namespace. They are listed below without section ids.

`<svg>`
`<g>`
`<path>`
`<glyph>`
`<glyphref>`
`<marker>`
`<view>`
`<circle>`
`<line>`
`<polygon>`
`<polyline>`
`<rect>`
`<text>`
`<textpath>`
`<tref>`
`<tspan>`
`<clippath>`
`<filter>`
`<lineargradient>`
`<radialgradient>`
`<mask>`
`<pattern>`
`<vkern>`
`<hkern>`
`<defs>`
`<use>`
`<symbol>`
`<desc>`
`<title>`
--->

#### 4.9 Tabellendaten<a name="49-tabular-data"></a>

4.9.1 `<table>` 4.9.2 `<caption>` 4.9.3 `<colgroup>` 4.9.4 `<col>` 4.9.5 `<tbody>` 4.9.6 `<thead>` 4.9.7 `<tfoot>` 4.9.8 `<tr>` 4.9. 9 `<td>` 4.9.10 `<th>`

#### 4.10 Formulare<a name="410-forms"></a>

4.10.8 `<button>`

#### 4.11 Skripterstellung<a name="411-scripting"></a>

- Wie bei einem allgemeinen AMP-Dokument muss das `<head>` -Tag des Creatives ein `<script async src="https://cdn.ampproject.org/amp4ads-v0.js"></script>` -Tag enthalten.
- Im Gegensatz zu allgemeinem AMP ist `<noscript>` verboten.
    - *Begründung:* Da für AMPHTML-Anzeigen Javascript aktiviert sein muss, um überhaupt zu funktionieren, dienen `<noscript>` in AMPHTML-Anzeigen keinem Zweck und kosten nur Netzwerkbandbreite.
- Im Gegensatz zu allgemeinem AMP ist `<script type="application/ld+json">` verboten.
    - *Begründung:* JSON LD wird für strukturiertes Daten-Markup auf Hostseiten verwendet, aber Werbeanzeigen sind keine eigenständigen Dokumente und enthalten keine strukturierten Daten. JSON LD-Blöcke in ihnen würden nur Netzwerkbandbreite kosten.
- Alle anderen Skriptregeln und -ausschlüsse werden vom allgemeinen AMP übernommen.
