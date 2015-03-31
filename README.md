#Picufy

Generate semi-random images for texts and decode such graphics back to their text representations.

<table><tbody><tr><td>
    <img src="https://raw.githubusercontent.com/futape/picufy/master/misc/hello-world.png" alt="Picufy image representation of &quot;Hello world&quot;" width="200" />
</td><td>
    <pre><code>&gt; my_picufy = new Picufy(&quot;Hello world&quot;)
[object Picufy]
&gt; my_picufy_decode = Picufy.decode(my_picufy.canvas)
[object Picufy]
&gt; my_picufy_decode.text
&quot;Hello world&quot;</code></pre>
</td></tr></tbody></table>



##Table of contents

1.  [API](#api)
2.  [System requirements](#system-requirements)
3.  [The Picufy sepcification](#the-picufy-sepcification)
4.  [A word about the master branch](#a-word-about-the-master-branch)
5.  [License](#license)
6.  [Support](#support)
7.  [Contributing](#contributing)
8.  [Author](#author)



##API

The library's functions are available via the `Picufy` class. That class is defined in the global namespace as `window.Picufy`.

###Instance properties

####text

`string text`

The text represented by the image.  
See [`Picufy()`](#picufy-1) for more information.

####greyscale

`bool greyscale`

Whether the image consists of shades of grey only.  
This happens when the first character of the encoded text is outside of the range of `a-zA-Z`.

####baseHue

`int baseHue`

The base hue value upon which the squares' colors' hue values are calculated.  
The `baseHue` is determined upon the lowercased, first character of the text. For "a" `baseHue` will be 0, for "b" it will be 60, for "c": 120, "d": 180, "e": 240, "f": 300, and for "g": 0 again, and so on for letters from "h" to "z".  
Characters outside of the range of `a-zA-Z` ([`greyscale`d](#greyscale)) will produce a `baseHue` of 0.

####matrix

`int[][] matrix`

A two-dimensional array containing the HSL color values for the squares in its second level and the *rows* of the image in its root level.  
The number of rows and the number of color values in a row is equal to the square root of the number all squares in the produced image. Therefore the overall number of items in this array is equal to the number of all squares.  
If the number of squares isn't a square number, the next higher one is used instead and *bit sequence terminators* are used to add the missing squares in randomly picked places.  
The bottom right square is always a *bit sequence terminator* and is called [*base square*](#square-types) since it always represents the base color, upon which the other squares' colors are calculated. The square to its left (if any) will never be a *bit sequence terminator*, too, and will therfore never have the same color.  
Each square [represents](#square-types) one bit or a *bit sequence terminator* marking the end of a [sequence of bits](#bit-sequences), making up one character.  
Whether a square marks up a 0 or a 1 bit, is controlled by its color's hue value. If it's lower than the [`baseHue`](#basehue), it's a 0-bit, if it's greater than `baseHue`, it's a 1-bit. Squares with a hue value equal to the `baseHue` mark a [*bit sequence terminator*](#square-types).  
For [`greyscale`d](#greyscale) images the lightness value is used instead of the hue value. If it's lower than 50, a 0-bit is assumed, if it's greater than 50, it's a 1-bit, and if the lightness value is equal to 50, the square marks a *bit sequnce termiator*. The hue value for such images is always 0.  
Also, for greyscaled images the saturation is always set to 0, while it's a value between 50 and 100 for colored images.  
For colored images, the hue value is always in the range of `baseHue - 30` to `baseHue + 30`, and the lightness value is a number between 25 and 75.  
Except for the direction (+ or -) of the difference between the `baseHue` and the hue value of a square for a colored image, or between the lightness of a square and 50 for greyscaled images, everything else is randomly picked inside of the specified ranges.

#####size

`int size`

The width and height of the created canvas. If this property's value can't be divided by the number of squares per row evenly, the value is increased, so that each square consists of *full pixels* only.  
For more information see [`Picufy()`](#picufy-1).

####canvas

`HTMLCanvasElement canvas`

The generated canvas representing the encoded text.  
A [`<canvas>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/canvas) element that has the sqares described by the [`matrix`](#matrix) property drawn to it.   The [`width`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/canvas#attr-width) and [`height`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/canvas#attr-height) attributes of the canvas are set to the value of the [`size`](#size) property.

###Instance functions

####Picufy()

`void Picufy( string text [, int size ] )`

The constructor of a `Picufy` object initializing the objects properties.

+   `text`: The text to encode as an image. Should not be empty.
+   `size`: The size of the generated canvas. If not specified, [`Picufy.config.canvasSize`](#config) is used instead.  
     Must be greater than 0, otherwise 1 is used instead.

####toString()

`string toString()`

This function is automatically called when a `Picufy` object is converted to a string (e.g. when concatenating it with a string).

Returns the value of the [`text`](#text) property.

###Static properties

####config

`object config`

An object containig the Picufy configuration options.  
The following options are available.

+   `int canvasSize = 200`: The default size of the created canvas used as default value for the Picufy constructor's second parameter.


###Static functions

####encode()

`Picufy encode(string text [, int size ])`

This function is identical to `new Picufy(text, size)`.  
For more information, see [`Picufy()`](#picufy-1).

####decode()

`Picufy decode( HTMLCanvasElement canvas )`

Decodes a canvas generated by Picufy and creates a new `Picufy` object from the information retrieved from the canvas.  
When decoding an image file (e.g. a `.png` file), a new [`HTMLCanvasElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLCanvasElement) object has to be created and that image must be drawn to it. When doing so, it's important, that the canvas has the exact same dimensions as the image and that the image is not scaled down or up when drawing it to the canvas.

+   `canvas`: The canvas to decode. The image must comply with the [Picufy specification](#the-picufy-sepcification) that is also follow by Picufy when *creating* a canvas.



##System requirements

To use Picufy, browsers must provide Canvas support.



##The Picufy sepcification

An image created by Picufy is made up of several squares distributed evenly to the horizontal and the vertical. The number of squares is always a square number, therfore the created image is a square, too. If the number of squares isn't a square number, as many squares as needed to reach the next higher square number are added. The added squares are [*bit sequence terminators*](#square-types).  
The bottom right square is the reference point called the [*base square*](#square-types).  
Each square represents one bit and is colored in a variation of the [*base hue* or the *base lightness*](#base-hue-and-base-lightness).

###Color or greyscale

This depends on the first character of the passed text.  
If it's in the range of `a-zA-Z`, the image will be colored. Otherwise, shades of grey are used instead.  
For colored images, the following starting characters have been assigned to specific hues.

+   **A**, **a**: `0`
+   **B**, **b**: `60`
+   **C**, **c**: `120`
+   **D**, **d**: `180`
+   **E**, **e**: `240`
+   **F**, **f**: `300`
+   **G**, **g**: `0`
+   **H**, **h**: `60`
+   **I**, **i**: `120`
+   **J**, **j**: `180`
+   **K**, **k**: `240`
+   **L**, **l**: `300`
+   **M**, **m**: `0`
+   **N**, **n**: `60`
+   **O**, **o**: `120`
+   **P**, **p**: `180`
+   **Q**, **q**: `240`
+   **R**, **r**: `300`
+   **S**, **s**: `0`
+   **T**, **t**: `60`
+   **U**, **u**: `120`
+   **V**, **v**: `180`
+   **W**, **w**: `240`
+   **X**, **x**: `300`
+   **Y**, **y**: `0`
+   **Z**, **z**: `60`

###Square types

+   **0-bit**: A square representing a 0-bit marked up by using a value lower than the [*base hue* or the *base lightness*](#base-hue-and-base-lightness) for the HSL color value's respective component.
+   **1-bit**: A square representing a 1-bit marked up by using a value greater than the [*base hue* or the *base lightness*](#base-hue-and-base-lightness) for the HSL color value's respective component.
+   **Bit sequence terminator**: Marks the end of a [*bit sequence*](#bit-sequences), describing one character. Such squares don't have any other effect and don't mark up a bit of a character. They are marked up by using the [*base hue* or the *base lightness*](#base-hue-and-base-lightness) for the HSL color value's respective component.
+   **Base square**: A *bit sequence terminator* square, defining the [*base hue* or the *base lightness*](#base-hue-and-base-lightness), used as a reference point to compare the other squares with. Also it is used to determine the squares' widths. The square is always placed in the bottom right corner of the created image. A neighboring square to its left must not have the same color as the *base square*.

###*Base hue* and *base lightness*

Defined by the *base square* and specifying the value, the other squares' colors are treated relatively to.  
If the image is greyscaled, the *base lightness* is used to retrieve the squares' meanings and types. If it's colored, the *base hue* is used instead.

###Bit sequences

To decode the image, you have to process square by square, from left to right, and from top to bottom.  
As said above, every square ([*0-bit* and *1-bit* squares](#square-types)) represents one bit. Multiple consecutive squares build a *bit sequence*. Those are ended by a [*bit sequence terminator*](#square-types) square. After ending a *bit sequence*, a new one is opened again immediately.  
The bits in a *bit sequence* describe a dual number. The corresponding decimal number is used as a character's code in the Unicode. Therfore, each *bit sequence* marks up one character. However, this does not mean, that the encoded text has exactly as much characters as *bit sequence terminator* squares are present in the image since those squares are also used as padding to increase the number of squares to match the next higher square number (if not already one; [see above](#the-picufy-sepcification)).  
Empty *bit sequences* are ignored.  
For information on the meaning and type of specific squares, see [*Square types*](#square-types).



##A word about the `master` branch

This repository has two main branches, the `develop` branch and the `master` branch.  
Branch management is done using [Vincent Driessen](http://nvie.com/posts/a-successful-git-branching-model/)'s branching model, meaning that all bleeding-edge features are available on the `develop` branch, while the `master` branch contains the stable releases only. Commits on the `master` branch introducing changes to the public API are tagged with a version number.

Versioning is done using [semantic versioning](http://semver.org/). This means that a version identifier consists of three parts, the first one being the *major* version number, the second one the *minor* version number and the third one speciying the *patch* number, separated by dots. Whenever a API-incompatible change is introduced, the major version is number increased. If the change is backwards-compatible to the public API, the minor version number is increased. A hotfix to the source increases the patch number.

A list of releases can be seen [here](https://github.com/futape/php-semver/releases). Please note, that releases with a major version number of 0 belong to the initial development phase and are not considered to be absolutely stable. However, every release since version 1.0.0 is considered to be stable.



##License

The Picufy source is published under the permissive [*New* BSD License](http://opensource.org/licenses/BSD-3-Clause).  
A [copy of that license](https://github.com/futape/picufy/blob/master/src/futape/semver/LICENSE) is located under `src`.

Any other content is, if not otherwise stated, published under a [Creative Commons Attribution 4.0 International License](http://creativecommons.org/licenses/by/4.0/).  
<a href="http://creativecommons.org/licenses/by/4.0/"><img alt="Creative Commons License" border="0" src="https://i.creativecommons.org/l/by/4.0/80x15.png" /></a>

The `test.html` file is released into the public domain and published under the [CC0 1.0 Universal License](http://creativecommons.org/publicdomain/zero/1.0/).  
<a href="http://creativecommons.org/publicdomain/zero/1.0/"><img src="http://i.creativecommons.org/p/zero/1.0/80x15.png" border="0" alt="CC0" /></a>



##Support

<a href="https://flattr.com/submit/auto?user_id=lucaskrause&amp;url=http%3A%2F%2Fpicufy.futape.de" target="_blank"><img src="http://button.flattr.com/flattr-badge-large.png" alt="Flattr this" title="Flattr this" border="0"></a>



##Contributing

Contributing to this project is currently not available.



##Author

<table><tbody><tr><td>
    <img src="http://www.gravatar.com/avatar/118bcae2fda8b302155ad47a2bfda556.png?s=100&amp;d=monsterid" />
</td><td>
    Lucas Krause (<a href="https://twitter.com/futape">@futape</a>)
</td></tr></tbody></table>

For a full list of contributors, click [here](https://github.com/futape/picufy/graphs/contributors).
