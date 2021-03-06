---
title: Maps, Slow and Fast
layout: lecture
tags:
  - deckgl
  - keplergl
  - geojson
  - geo
description: >-
  This lecture starts the process of exploring OpenGL-type concepts for rendering, why we might be interested in using them, and how we can apply them to visualizations.
---

## Software Frameworks

 * Getting Data
 * A whirlwind tour of frameworks!
   * Choices, choices choices
   * Deployment
 * Web-based frameworks
   * kepler.gl / vis.gl / deck.gl
   * ThreeJS (and pythreejs)
   * regl

---

## Getting Data

Last time we talked about rendering stuff in regl, with a focus on the specific
WebGL code.  Today we're going to talk a bit more broadly about getting data
into frameworks, and using those frameworks.

---

## Getting Data: Local Reduction

 * Load data from some remote source in its unprocessed form
 * This data is then reduced locally, internal to the web browser

In vega-lite we would define a loop kernel, for instance like this:

 ```json
"transform": [
    {"calculate": "2*datum.b", "as": "b2"},
    {"filter": "datum.b2 > 60"}
  ],
 ```

The individual data point is called `datum` in this case.

---

## Getting Data: Remote Reduction

Alternately, if the amount of data is prohibitively large to transfer,
particularly if the eventual reduction is extensive, there are occasionally
mechanisms that can be used to execute reductions on a server and transmit only
a small portion of the data.

Typically these are through encoding information in URLs and accessing via web
requests.  We will not cover these in detail today, but some examples can be
found in the Marvel API.

---

## Getting Data: Up from local

There's another possibility, which we will see "in action" today.  This is the
transmission of data from a local filesystem to the same local machine's web
browser memory.

Web-browsers are extremely security-conscious; the barrier between the local
machine and the virtual machine is (by design) very strong.  Accessing a local
file is possible, but requires deliberate action on the part of the user.

Some client-side apps -- and some server-side apps -- allow uploading of files.

---

## Internal Representation

Internally, we are concerned about essentially three different ways of
representing the data.  The first is through `Array` objects of javascript objects, as we have seen before:

```json
[ {'val1': 1.0, 'val2': 1.2}, {'val1': 2.3, 'val2': 1.0}, ...]
```

The second would be an object composed of columns of data:

```json
{'val1': [1.0, 2.3, ...], 'val2': [1.2, 1.0, ...] }
```

The third type, which we will not address, is through `TypedArray` objects in javascript.

---

## Example: Widgyts

The `widgyts` project is a hybrid of different methods for describing and transmitting data.

Data is supplied by a running Jupyter kernel, reduced locally, and uses a
combination of Javascript arrays and `TypedArray` objects.

---

## GeoJSON I - Intro

http://geojson.org/ and http://geojson.io/

```json
{
  "type": "Feature",
  "geometry": {
    "type": "Point",
    "coordinates": [125.6, 10.1]
  },
  "properties": {
    "name": "Dinagat Islands"
  }
}
```

---

## GeoJSON II - Primitive Types

GeoJSON defines several primitive types:

 * `Point` with associated `coordinates` (single set of two)
 * `LineString` with associated `coordinates` that are lists of two items each
 * `Polygon` with holes able to be cut out of it

Multi-part types are defined out of these.

---

## GeoJSON III - MultiPart Types

These are combined into multi-part types as follows:

 * `MultiPoint`
 * `MultiLineString`
 * `MultiPolygon`

---

## Drawing in a Browser

We will concern ourselves with two modes of drawing in a web browser:

 * [Canvas API](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API)
 * [WebGL](https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API)

(Note that we refer to the WebGL API as distinct, even though there is a WebGL
context for the Canvas.)

---

## Canvas API

```html
<canvas id="canvas"></canvas>
```

```javascript
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');

ctx.fillStyle = 'green';
ctx.fillRect(10, 10, 150, 100);
```

(jsfiddle.net to test this out!)

---

## WebGL API

The WebGL API is a slightly-modified form of the OpenGL API, designed for usage
in a web browser.  For our purposes we will think of things in terms of the
"vertex" and "fragment" shaders, and how these interact and are defined.

We will use a bit of WebGL today, but will largely not discuss this week.  It
is important to note that much of the data supplied to WebGL will be in the
form of `TypedArray` objects.  There are four mechanisms of supplying data to WebGL:

 * Uniforms - global variables for a given draw call
 * Textures - arrays of data (1D, 2D, and 3D)
 * Varyings - variables that are interpolated between vertex values
 * Attributes / Buffers - blobs of binary data that can become vertices, inform color, etc.

---

## Whirlwind Tour of Frameworks

We're going to take a look at several frameworks today, and dive the deepest
into a few developed by Uber.

---

## Setting Up

At first, we will access deployments that are fully online.  We will attempt,
later, to build isolated items.

Your homework for next week is to set up a development environment for these web
technologies.

---

## Web-Based Frameworks

Uber has released several components of a full-stack geoanalytics platform that
we will explore at length today.  These include components for transforming,
displaying, overlaying and brushing of data.  Typically, they are used for
spatial data.

There are several component layers made available in this suite, linked from [vis.gl](http://vis.gl/).

 * [deck.gl](http://deck.gl/) - reactive web framework for "large data analysis"
 * [luma.gl](http://luma.gl/) - general purpose GPU computing and high-performance rendering
 * [kepler.gl](http://kepler.gl/) - application for analytics and visualization

---

## ThreeJS

https://threejs.org/

ThreeJS is a high-level framework for constructing objects utilizing WebGL.
Typically these are 3D objects, but this is not always the case.

PyThreeJS is a convenient way to interact with ThreeJS, as it exposes a
"traited" interface to the object model.

---

## regl

https://regl.party/

regl is a reactive, functional WebGL dialect.

(We did regl last time and won't really do it again today.)

---

## PyDeck

Today we're going to use Deck, first in its form in Jupyter.

You will need to install `pydeck`.  We will not be using the jupyterlab extension today!

```
conda install pydeck
```

---

## PyDeck I: Hello, "World"

Let's see what pydeck gives us to start with.

```
import pydeck as pdk

r = pdk.Deck()
r.to_html()
```

What does this give you?

---

## PyDeck II: Add a Layer

PyDeck accepts "layers" which draw from one of several different classes.  The
first class is the "Core" layers, which are the lowest level.  A few of these:

 * `ArcLayer` -- "to" and "from"
 * `BitmapLayer` -- Draw some pictures
 * `ColumnLayer` -- render vertical cylinders
 * `GeoJson` -- directly render GeoJson files
 * `ScatterPlotLayer` -- Dots

In addition, there are aggregation layers which can be very useful:

 * `ContourLayer` -- render isobands or isolines based on values that have been binned onto the map
 * `HexagonLayer` -- hexagonal binning based on arrays of input

---

## PyDeck III: Getting data to a Layer


Typically, we will supply data via either a *URL*, a list of dictionaries, or a pandas dataframe.

We then create a layer and add it to our "deck."

Now, let's go to the interactive demo!

