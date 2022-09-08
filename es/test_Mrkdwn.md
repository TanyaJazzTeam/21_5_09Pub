---
"$title": Especificación de AMP para anuncios
order: '3'
formats:
  - anuncios
teaser:
  text: |2-

    _Si desea proponer cambios al estándar, por favor comente en el
    [Intención
toc: verdadero
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

*Si desea proponer cambios al estándar, comente la [Intención de implementar](https://github.com/ampproject/amphtml/issues/4264)* .

## Reglas de formato de anuncios AMP HTML<a name="amphtml-ad-format-rules"></a> 11

<table>
<thead><tr>
  <th>Regla</th>
  <th>Razón fundamental</th>
</tr></thead>
<tbody>
<tr>
<td>Debe usar <code>&lt;html ⚡4ads&gt;</code> o <code>&lt;html amp4ads&gt;</code> como etiquetas adjuntas.</td>
<td>Permite a los validadores identificar un documento creativo como un documento AMP general o un documento publicitario AMP HTML restringido y enviarlo de manera adecuada.</td>
</tr>
<tr>
<td>Debe incluir <code>&lt;script async src="https://cdn.ampproject.org/amp4ads-v0.js"&gt;&lt;/script&gt;</code> como script de tiempo de ejecución en lugar de <code>https://cdn.ampproject.org/v0.js</code> .</td>
<td>Permite comportamientos de tiempo de ejecución personalizados para anuncios AMP HTML publicados en iframes de origen cruzado.</td>
</tr>
<tr>
<td>No debe incluir una etiqueta <code>&lt;link rel="canonical"&gt;</code> .</td>
<td>Las creatividades de anuncios no tienen una "versión canónica que no sea AMP" y no se indexarán de forma independiente, por lo que las referencias propias serían inútiles.</td>
</tr>
<tr>
<td>Puede incluir etiquetas meta opcionales en el encabezado HTML como identificadores, en el formato de <code>&lt;meta name="amp4ads-id" content="vendor=${vendor},type=${type},id=${id}"&gt;</code> . Esas metaetiquetas deben colocarse antes del <code>amp4ads-v0.js</code> . El valor del <code>vendor</code> y la <code>id</code> son cadenas que contienen solo [0-9a-zA-Z_-]. El valor de <code>type</code> es <code>creative-id</code> o <code>impression-id</code> .</td>
<td>Esos identificadores personalizados se pueden usar para identificar la impresión o la creatividad. Pueden ser útiles para informar y depurar.<br><br><p> Ejemplo:</p>
<pre>
&lt;meta nombre="amp4ads-id"
content="vendor=adsense,type=id-creative,id=1283474"&gt;
&lt;meta nombre="amp4ads-id"
content="vendor=adsense,type=impression-id,id=xIsjdf921S"&gt;</pre>
</td>
</tr>
<tr>
<td>El seguimiento de visibilidad de <code>&lt;amp-analytics&gt;</code> solo puede apuntar al selector de anuncios completos, a través de <code>"visibilitySpec": { "selector": "amp-ad" }</code> como se define en <a href="https://github.com/ampproject/amphtml/issues/4018">el problema n.° 4018</a> y <a href="https://github.com/ampproject/amphtml/pull/4368">PR n.° 4368</a> . En particular, no puede dirigirse a ningún selector de elementos dentro de la creatividad del anuncio.</td>
<td>En algunos casos, los anuncios AMP HTML pueden optar por presentar una creatividad de anuncio en un iframe. En esos casos, el análisis de la página de host solo puede orientar el iframe completo de todos modos y no tendrá acceso a selectores más detallados.<br><br><p> Ejemplo:</p>
<pre>
&lt;amp-analytics id="nestedAnalytics"&gt;
&lt;tipo de script="aplicación/json"&gt;
{
"peticiones": {
"visibilidad": "https://example.com/nestedAmpAnalytics"
},
"desencadenantes": {
"especificación de visibilidad": {
"selector": "amp-anuncio",
"porcentaje mínimo visible": 50,
"tiempo continuo mínimo": 1000
}
}
}
&lt;/script&gt;
&lt;/amp-análisis&gt;
</pre>
<p>Esta configuración envía una solicitud a la URL <code>https://example.com/nestedAmpAnalytics</code> cuando el 50 % del anuncio adjunto ha estado visible continuamente en la pantalla durante 1 segundo.</p>
</td>
</tr>
</tbody>
</table>

### repetitivo<a name="boilerplate"></a> 22

Las creatividades de anuncios AMP HTML requieren una línea de estilo repetitivo diferente y considerablemente más simple que [los documentos AMP generales](https://github.com/ampproject/amphtml/blob/master/spec/amp-boilerplate.md) :

[sourcecode:html]
<style amp4ads-boilerplate>
  body {
    visibility: hidden;
  }
</style>
[/sourcecode]

*Justificación:* el estilo `amp-boilerplate` oculta el contenido del cuerpo hasta que el tiempo de ejecución de AMP esté listo y pueda mostrarlo. Si Javascript está deshabilitado o el tiempo de ejecución de AMP no se carga, el modelo predeterminado garantiza que el contenido finalmente se muestre independientemente. Sin embargo, en los anuncios AMP HTML, si Javascript está completamente deshabilitado, los anuncios AMP HTML no se ejecutarán y nunca se mostrará ningún anuncio, por lo que no es necesaria la sección `<noscript>` . En ausencia del tiempo de ejecución de AMP, la mayoría de la maquinaria en la que se basan los anuncios AMP HTML (p. ej., análisis para el seguimiento de la visibilidad o `amp-img` para la visualización de contenido) no estará disponible, por lo que es mejor no mostrar ningún anuncio que uno que no funcione correctamente.

Por último, el texto estándar del anuncio AMP HTML utiliza `amp-a4a-boilerplate` lugar de `amp-boilerplate` para que los validadores puedan identificarlo fácilmente y producir mensajes de error más precisos para ayudar a los desarrolladores. 33

### CSS<a name="css"></a>

<table>
<thead><tr>
  <th>Regla</th>
  <th>Razón fundamental</th>
</tr></thead>
<tbody>
  <tr>
    <td>
<code>position:fixed</code> y <code>position:sticky</code> están prohibidos en el CSS creativo.</td>
    <td>
<code>position:fixed</code> sale del shadow DOM, del que dependen los anuncios AMP HTML. Además, los anuncios en AMP ya no pueden usar una posición fija.</td>
  </tr>
  <tr>
    <td>
<code>touch-action</code> está prohibida.</td>
    <td>Un anuncio que puede manipular <code>touch-action</code> puede interferir con la capacidad del usuario para desplazarse por el documento anfitrión.</td>
  </tr>
  <tr>
    <td>Creative CSS está limitado a 20.000 bytes.</td>
    <td>Los grandes bloques de CSS inflan la creatividad, aumentan la latencia de la red y degradan el rendimiento de la página.</td>
  </tr>
  <tr>
    <td>La transición y la animación están sujetas a restricciones adicionales.</td>
    <td>AMP debe poder controlar todas las animaciones que pertenecen a un anuncio, de modo que pueda detenerlas cuando el anuncio no está en la pantalla o los recursos del sistema son muy bajos.</td>
  </tr>
  <tr>
    <td>Los prefijos específicos del proveedor se consideran alias para el mismo símbolo sin el prefijo para fines de validación. Esto significa que si un símbolo <code>foo</code> está prohibido por las reglas de validación de CSS, entonces el símbolo <code>-vendor-foo</code> también estará prohibido.</td>
    <td>Algunas propiedades con prefijo de proveedor brindan una funcionalidad equivalente a las propiedades que, de otro modo, están prohibidas o restringidas según estas reglas.<br><br><p> Ejemplo: <code>-webkit-transition</code> y <code>-moz-transition</code> se consideran alias para <code>transition</code> . Solo se permitirán en contextos en los que se permitiría una <code>transition</code> simple (consulte la sección <a href="#selectors">Selectores</a> a continuación).</p>
</td>
  </tr>
</tbody>
</table>

#### Animaciones y transiciones CSS<a name="css-animations-and-transitions"></a>

##### Selectores<a name="selectors"></a>

Las propiedades de `transition` y `animation` solo se permiten en selectores que:

- Solo contienen propiedades de `transition` , `animation` , `transform` , `visibility` u `opacity` .

    *Justificación:* esto permite que el tiempo de ejecución de AMP elimine esta clase del contexto para desactivar las animaciones, cuando sea necesario para el rendimiento de la página.

**Bueno**

[sourcecode:css]
.box {
  transform: rotate(180deg);
  transition: transform 2s;
}
[/sourcecode]

**Malo**

Propiedad no permitida en la clase CSS.

[sourcecode:css]
.box {
  color: red; // non-animation property not allowed in animation selector
  transform: rotate(180deg);
  transition: transform 2s;
}
[/sourcecode]

##### Propiedades transitables y animables<a name="transitionable-and-animatable-properties"></a>

Las únicas propiedades que se pueden cambiar son la opacidad y la transformación. ( [Justificación](http://www.html5rocks.com/en/tutorials/speed/high-performance-animations/) )

**Bueno**

[sourcecode:css]
transition: transform 2s;
[/sourcecode]

**Malo**

[sourcecode:css]
transition: background-color 2s;
[/sourcecode]

**Bueno**

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

**Malo**

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

### Extensiones y elementos integrados de AMP permitidos<a name="allowed-amp-extensions-and-builtins"></a>

Los siguientes módulos de extensión de AMP y etiquetas integradas de AMP están *permitidos* en una creatividad de anuncio HTML de AMP. Las extensiones o etiquetas integradas que no se enumeran explícitamente están prohibidas.

- [acordeón](https://amp.dev/documentation/components/amp-accordion)
- [amp-ad-salir](https://amp.dev/documentation/components/amp-ad-exit)
- [amp-análisis](https://amp.dev/documentation/components/amp-analytics)
- [amp-anim](https://amp.dev/documentation/components/amp-anim)
- [amp-animación](https://amp.dev/documentation/components/amp-animation)
- [amplificador de audio](https://amp.dev/documentation/components/amp-audio)
- [enlace de amplificador](https://amp.dev/documentation/components/amp-bind)
- [carrusel de amplificadores](https://amp.dev/documentation/components/amp-carousel)
- [amp-fit-texto](https://amp.dev/documentation/components/amp-fit-text)
- [amp-fuente](https://amp.dev/documentation/components/amp-font)
- [forma de amplificador](https://amp.dev/documentation/components/amp-form)
- [amp-img](https://amp.dev/documentation/components/amp-img)
- [diseño de amplificador](https://amp.dev/documentation/components/amp-layout)
- [amplificador-caja de luz](https://amp.dev/documentation/components/amp-lightbox)
- amp-mraid, de forma experimental. Si está considerando usar esto, abra un problema en [wg-ads](https://github.com/ampproject/wg-ads/issues/new) .
- [amp-bigote](https://amp.dev/documentation/components/amp-mustache)
- [amp-píxel](https://amp.dev/documentation/components/amp-pixel)
- [amp-posición-observador](https://amp.dev/documentation/components/amp-position-observer)
- [selector de amplificador](https://amp.dev/documentation/components/amp-selector)
- [amp-social-share](https://amp.dev/documentation/components/amp-social-share)
- [amp-video](https://amp.dev/documentation/components/amp-video)

La mayoría de las omisiones se deben al rendimiento o para simplificar el análisis de los anuncios AMP HTML.

*Ejemplo:* `<amp-ad>` se omite de esta lista. No se permite explícitamente porque permitir un `<amp-ad>` dentro de un `<amp-ad>` podría conducir a cascadas ilimitadas de carga de anuncios, lo que no cumple con los objetivos de rendimiento de los anuncios AMP HTML.

*Ejemplo:* `<amp-iframe>` se omite de esta lista. No está permitido porque los anuncios podrían usarlo para ejecutar Javascript arbitrario y cargar contenido arbitrario. Los anuncios que deseen utilizar dichas capacidades deben devolver `false` desde su entrada [a4aRegistry](https://github.com/ampproject/amphtml/blob/master/ads/_a4a-config.js#L40) y utilizar el mecanismo de representación de anuncios '3p iframe' existente.

*Ejemplo:* `<amp-facebook>` , `<amp-instagram>` , `<amp-twitter>` y `<amp-youtube>` se omiten por el mismo motivo que `<amp-iframe>` : todos crean iframes y pueden consumir recursos ilimitados en ellos.

*Ejemplo:* `<amp-ad-network-*-impl>` se omiten de esta lista. La etiqueta `<amp-ad>` maneja la delegación a estas etiquetas de implementación; las creatividades no deben intentar incluirlos directamente.

*Ejemplo:* `<amp-lightbox>` aún no está incluido porque incluso algunas creatividades de anuncios AMP HTML pueden mostrarse en un iframe y actualmente no existe ningún mecanismo para que un anuncio se expanda más allá de un iframe. Se puede agregar apoyo para esto en el futuro, si se demuestra el deseo de hacerlo.

### Etiquetas HTML<a name="html-tags"></a>

Las siguientes son etiquetas *permitidas* en una creatividad de anuncios AMP HTML. Las etiquetas no permitidas explícitamente están prohibidas. Esta lista es un subconjunto de la lista de permitidos general [del apéndice de etiquetas AMP](https://github.com/ampproject/amphtml/blob/master/extensions/amp-a4a/../../spec/amp-tag-addendum.md) . Al igual que esa lista, está ordenada de acuerdo con las especificaciones de HTML5 en la sección 4 [Los elementos de HTML](http://www.w3.org/TR/html5/single-page.html#html-elements) .

La mayoría de las omisiones se deben al rendimiento o porque las etiquetas no son el estándar HTML5. Por ejemplo, `<noscript>` se omite porque los anuncios AMP HTML dependen de que JavaScript esté habilitado, por lo que un bloque `<noscript>` nunca se ejecutará y, por lo tanto, solo inflará la creatividad y costará ancho de banda y latencia. Del mismo modo, `<acronym>` , `<big>` , et al. están prohibidos porque no son compatibles con HTML5.

#### 4.1 El elemento raíz<a name="41-the-root-element"></a>

`<html>`

- Debe usar los tipos `<html ⚡4ads>` o `<html amp4ads>`

#### 4.2 Metadatos del documento<a name="42-document-metadata"></a>

4.2.1 `<head>`

4.2.2 `<title>`

4.2.4 `<link>`

- Las etiquetas `<link rel=...>` no están permitidas, excepto `<link rel=stylesheet>` .

- **Nota:** a diferencia de AMP general, las etiquetas `<link rel="canonical">` están prohibidas.

    4.2.5 `<style>` 4.2.6 `<meta>`

#### 4.3 Secciones<a name="43-sections"></a>

4.3.1 `<body>` 4.3.2 `<article>` 4.3.3 `<section>` 4.3.4 `<nav>` 4.3.5 `<aside>` 4.3.6 `<h1>` , `<h2>` , `<h3>` , `<h4>` , `<h5>` y `<h6>` 4.3.7 `<header>` 4.3.8 `<footer>` 4.3.9 `<address>`

#### 4.4 Agrupación de contenido<a name="44-grouping-content"></a>

4.4.1 `<p>` 4.4.2 `<hr>` 4.4.3 `<pre>` 4.4.4 `<blockquote>` 4.4.5 `<ol>` 4.4.6 `<ul>` 4.4.7 `<li>` 4.4.8 `<dl>` 4.4. 9 `<dt>` 4.4.10 `<dd>` 4.4.11 `<figure>` 4.4.12 `<figcaption>` 4.4.13 `<div>` 4.4.14 `<main>`

#### 4.5 Semántica a nivel de texto<a name="45-text-level-semantics"></a>

4.5.1 `<a>` 4.5.2 `<em>` 4.5.3 `<strong>` 4.5.4 `<small>` 4.5.5 `<s>` 4.5.6 `<cite>` 4.5.7 `<q>` 4.5.8 `<dfn>` 4.5. 9 `<abbr>` 4.5.10 `<data>` 4.5.11 `<time>` 4.5.12 `<code>` 4.5.13 `<var>` 4.5.14 `<samp>` 4.5.15 `<kbd >` 4.5.16 `<sub>` y `<sup>` 4.5.17 `<i>` 4.5.18 `<b>` 4.5.19 `<u>` 4.5.20 `<mark>` 4.5.21 `<ruby>` 4.5.22 `<rb>` 4.5.23 `<rt>` 4.5.24 `<rtc>` 4.5. 25 `<rp>` 4.5.26 `<bdi>` 4.5.27 `<bdo>` 4.5.28 `<span>` 4.5.29 `<br>` 4.5.30 `<wbr>`

#### 4.6 Ediciones<a name="46-edits"></a>

4.6.1 `<ins>` 4.6.2 `<del>`

#### 4.7 Contenido incrustado<a name="47-embedded-content"></a>

- El contenido incrustado solo se admite a través de etiquetas AMP, como `<amp-img>` o `<amp-video>` .

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

#### 4.9 Datos tabulares<a name="49-tabular-data"></a>

4.9.1 `<table>` 4.9.2 `<caption>` 4.9.3 `<colgroup>` 4.9.4 `<col>` 4.9.5 `<tbody>` 4.9.6 `<thead>` 4.9.7 `<tfoot>` 4.9.8 `<tr>` 4.9. 9 `<td>` 4.9.10 `<th>`

#### 4.10 Formularios<a name="410-forms"></a>

4.10.8 `<button>`

#### 4.11 Secuencias de comandos<a name="411-scripting"></a>

- Al igual que un documento AMP general, la etiqueta `<head>` de la creatividad debe contener una etiqueta `<script async src="https://cdn.ampproject.org/amp4ads-v0.js"></script>` .
- A diferencia de AMP general, `<noscript>` está prohibido.
    - *Justificación:* dado que los anuncios AMP HTML requieren que Javascript esté habilitado para funcionar, los bloques `<noscript>` no sirven para nada en los anuncios AMP HTML y solo cuestan ancho de banda de red. [123]
- A diferencia de AMP general, `<script type="application/ld+json">` está prohibido. [123]
    - *Justificación:* JSON LD se utiliza para el marcado de datos estructurados en páginas de host, pero las creatividades de anuncios no son documentos independientes y no contienen datos estructurados. Los bloques JSON LD en ellos solo costarían ancho de banda de red. [123]
- Todas las demás reglas y exclusiones de secuencias de comandos se transfieren de AMP general. [123]
