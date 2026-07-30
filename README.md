# PHPizza Emoji (Phemoji) 
A simple library that provides standard Unicode [emoji](https://en.wikipedia.org/wiki/Emoji) support across all platforms.

**Phemoji v17.0** adheres to the [Unicode 17.0 spec](https://unicode.org/versions/Unicode17.0.0/) and supports the [Emoji 17.0 spec](https://www.unicode.org/reports/tr51/tr51-21.html). _We do not support custom emoji, that is for PHPizza to support._

The Phemoji library offers support for all Unicode-defined emoji which are recommended for general interchange (RGI).

## Usage

### CDN Support

While Elon Musk paid for MaxCDN to host Phemoji, this is not Phemoji but a fork. ~~It may come with PHPizza soon, but it is not on a central CDN.~~It has now been added into PHPizza

### Download

If you want to download a specific version, it is possible to download the codebase at a specific commit using GitHub's interface.

## API

If you would not like to do the parsing yourself, I would recommend [PHPizza](https://github.com/johnnycharlesw/phpizza).

But anyways, following are all the methods exposed in the `phemoji` namespace in JavaScript. But, I'd recommend you use PHP on the server instead:
```php
    protected function loadPhemojiAssets()
    {
        $map = [];
        $files = glob($_SERVER['DOCUMENT_ROOT'] . '/assets/phemoji/assets/72x72/*.png');

        foreach ($files as $file) {
            $name = pathinfo($file, PATHINFO_FILENAME);
            $codepoints = explode('-', $name);

            $sequence = '';

            foreach ($codepoints as $codepoint) {
                $sequence .= mb_chr(hexdec($codepoint), 'UTF-8');
            }

            $map[$sequence] = '/assets/72x72/'. basename($file);
        }

        uksort($map, function ($a, $b) {
            return mb_strlen($b, 'UTF-8') <=> mb_strlen($a, 'UTF-8');
        });

        return $map;
    }

    protected function replacePhemoji($html)
    {
        $emojiMap = $this->loadPhemojiAssets();

        foreach ($emojiMap as $sequence => $asset) {
            $newline = "\n";
            $img = $newline . '<img class="phemoji" src="/assets/phemoji/' . $asset . '"/>'. $newline;

            $html = str_replace($sequence, $img, $html);
        }

        return $html;
    }
```

### phemoji.parse( ... ) V1

This is the main parsing utility and has 3 overloads per parsing type.

Although there are two kinds of parsing supported by this utility, we recommend you use [DOM parsing](https://github.com/johnnycharlesw/phemoji#dom-parsing), explained below. Each type of parsing accepts a callback to generate an image source or an options object with parsing info.

The second kind of parsing is string parsing, explained in the legacy documentation [here](https://github.com/johnnycharlesw/phemoji/blob/master/LEGACY.md#string-parsing). This is unrecommended because this method does not sanitize the string or otherwise prevent malicious code from being executed; such sanitization is out of scope.

#### DOM parsing

If the first argument to `phemoji.parse` is an `HTMLElement`, generated image tags will replace emoji that are **inside `#text` nodes only** without compromising surrounding nodes or listeners, and completely avoiding the usage of `innerHTML`.

If security is a major concern, this parsing can be considered the safest option but with a slight performance penalty due to DOM operations that are inevitably *costly*.

```js
var div = document.createElement('div');
div.textContent = 'I \u2764\uFE0F emoji!';
document.body.appendChild(div);

phemoji.parse(document.body);

var img = div.querySelector('img');

// note the div is preserved
img.parentNode === div; // true

img.src;        // https://twemoji.maxcdn.com/v/latest/72x72/2764.png
img.alt;        // \u2764\uFE0F
img.className;  // emoji
img.draggable;  // false

```

All other overloads described for `string` are available in exactly the same way for DOM parsing.

### Object as parameter

Here's the list of properties accepted by the optional object that can be passed to the `parse` function.

```js
  {
    callback: Function,   // default the common replacer
    attributes: Function, // default returns {}
    base: string,         // default PHPizza
    ext: string,          // default ".png"
    className: string,    // default "emoji"
    size: string|number,  // default "72x72"
    folder: string        // in case it's specified
                          // it replaces .size info, if any
  }
```

#### callback

The function to invoke in order to generate image `src`(s).

By default it is a function like the following one:

```js
function imageSourceGenerator(icon, options) {
  return ''.concat(
    options.base, // by default Twitter Inc. CDN
    options.size, // by default "72x72" string
    '/',
    icon,         // the found emoji as code point
    options.ext   // by default ".png"
  );
}
```

#### base

The default url is the same as `phemoji.base`, so if you modify the former, it will reflect as default for all parsed strings or nodes.

#### ext

The default image extension is the same as `phemoji.ext` which is `".png"`.

If you modify the former, it will reflect as default for all parsed strings or nodes.

#### className

The default `class` for each generated image is `emoji`. It is possible to specify a different one through this property.

##### size

The default asset size is the same as `phemoji.size` which is `"72x72"`.

If you modify the former, it will reflect as default for all parsed strings or nodes.

#### folder

In case you don't want to specify a size for the image. It is possible to choose a folder, as in the case of SVG emoji.

```js
phemoji.parse(genericNode, {
  folder: 'svg',
  ext: '.svg'
});
```

This will generate urls such `http://phpizza.localhost/node_modules/phemoji/svg/2764.svg` instead of using a specific size based image.

## Utilities

Basic utilities / helpers to convert code points to JavaScript surrogates and vice versa.

### phemoji.convert.fromCodePoint()

For a given HEX codepoint, returns UTF-16 surrogate pairs.

```js
phemoji.convert.fromCodePoint('1f1e8');
 // "\ud83c\udde8"
```

### phemoji.convert.toCodePoint()

For given UTF-16 surrogate pairs, returns the equivalent HEX codepoint.

```js
 phemoji.convert.toCodePoint('\ud83c\udde8\ud83c\uddf3');
 // "1f1e8-1f1f3"

 phemoji.convert.toCodePoint('\ud83c\udde8\ud83c\uddf3', '~');
 // "1f1e8~1f1f3"
```

## Tips

### Inline Styles

If you'd like to size the emoji according to the surrounding text, you can add the following CSS to your stylesheet:

```css
img.emoji {
   height: 1em;
   width: 1em;
   margin: 0 .05em 0 .1em;
   vertical-align: -0.1em;
}
```

This will make sure emoji derive their width and height from the `font-size` of the text they're shown with. It also adds just a little bit of space before and after each emoji, and pulls them upwards a little bit for better optical alignment.

### UTF-8 Character Set

To properly support emoji, the document character set must be set to UTF-8. This can be done by including the following meta tag in the document `<head>`

```html
<meta charset="utf-8">
```

### Exclude Characters (V1)

To exclude certain characters from being replaced by phemoji.js, call phemoji.parse() with a callback, returning false for the specific unicode icon. For example:

```js
phemoji.parse(document.body, {
    callback: function(icon, options, variant) {
        return ''.concat(options.base, options.size, '/', icon, options.ext);
    }
});
```

## Legacy API (V1)

If you're still using our V1 API, you can read our legacy documentation [here](https://github.com/johnnycharlesw/phemoji/tree/master/LEGACY.md).

## Contributing

The contributing documentation can be found [here](https://github.com/johnnycharlesw/phemoji/tree/master/CONTRIBUTING.md).

## Attribution Requirements

As an open source project, attribution is critical from a legal, practical and motivational perspective in our opinion. The graphics are licensed under the CC-BY 4.0 which has a pretty good guide on [best practices for attribution](https://wiki.creativecommons.org/Best_practices_for_attribution).

However, we consider the guide a bit onerous and as a project, will accept a mention in a project README or an 'About' section or footer on a website. In mobile applications, a common place would be in the Settings/About section (for example, see the mobile Twitter application Settings->About->Legal section). We would consider a mention in the HTML/JS source sufficient also.

## Community Projects

* [Phemoji Cheatsheet](https://twemoji-cheatsheet.vercel.app) by [@ShahriarKh](https://github.com/ShahriarKh): An easy-to-use cheatsheet for exploring, copying and downloading emojis!
* [Phemoji Amazing](https://github.com/SebastianAigner/twemoji-amazing) by [@SebastianAigner](https://github.com/SebastianAigner): Use Phemoji using CSS classes (like [Font Awesome](http://fortawesome.github.io/Font-Awesome/)).
* [Phemoji Ruby](https://github.com/jollygoodcode/twemoji) by [@JollyGoodCode](https://twitter.com/jollygoodcode): Use Phemoji in Ruby.
* [Phemoji Utils](https://github.com/gustavwilliam/twemoji-utils) by [@gustavwilliam](https://github.com/gustavwilliam): Utilities for finding and downloading Phemoji source files.
* [Phemoji for Pencil](https://github.com/nathanielw/Phemoji-for-Pencil) by [@Nathanielnw](https://twitter.com/nathanielnw): Use Phemoji in Pencil.
* [FrwPhemoji - Phemoji in dotnet](http://github.frenchw.net/FrwPhemoji/) by [@FrenchW](https://twitter.com/frenchw): Use Phemoji in any dotnet project (C#, asp.net ...).
* [Emojiawesome - Phemoji for Yellow](https://github.com/datenstrom/yellow-extensions/tree/master/source/emojiawesome) by [@datenstrom](https://github.com/datenstrom/): Use Phemoji on your website.
* [EmojiPanel for Twitter](https://github.com/danbovey/EmojiPanel) by [@danielbovey](https://twitter.com/danielbovey/status/749580050274582528): Insert Phemoji into your tweets on twitter.com.
* [Twitter Color Emoji font](https://github.com/eosrei/twemoji-color-font) by [@bderickson](https://twitter.com/bderickson): Use Phemoji as your system default font on Linux & OS X.
* [Emojica](https://github.com/xoudini/emojica) by [@xoudini](https://twitter.com/xoudini): An iOS framework allowing you to replace all standard emoji in strings with Phemoji.
* [gwt-twemoji](https://github.com/phpmonkeys-de/gwt-twemoji) by [@nbartels](https://github.com/nbartels): Use Phemoji in GWT
* [JavaFXEmojiTextFlow](https://github.com/pavlobu/emoji-text-flow-javafx) by [@pavlobu](https://github.com/pavlobu): A JavaFX library allowing you to replace all standard emoji in extended EmojiTextFlow with Phemoji.
* [Vue Phemoji Picker](https://github.com/kevinfaguiar/vue-twemoji-picker) by [@kevinfaguiar](https://github.com/kevinfaguiar): A fast plug-n-play Phemoji Picker (+textarea for Phemoji rendering) for Vue.
* [Unmaintained] [Phemoji Awesome](http://ellekasai.github.io/twemoji-awesome/) by [@ellekasai](https://twitter.com/ellekasai/): Use Phemoji using CSS classes (like [Font Awesome](http://fortawesome.github.io/Font-Awesome/)).
* [EmojiOnRoku](https://github.com/KasperGam/EmojiOnRoku) by [@KasperGam](https://github.com/KasperGam): Use Phemoji on Roku!
* [LaTeX Phemoji](https://gitlab.com/rossel.jost/latex-twemojis) by [@rossel.jost](https://gitlab.com/rossel.jost): Use Phemoji in LaTeX.
* [PHP Phemoji](https://github.com/Astrotomic/php-twemoji) by [@Astrotomic](https://github.com/Astrotomic): Use twemoji within your PHP website project's by replacing standard Emoji with twemoji urls.

## Committers and Contributors

* Justine De Caires (Twitter)
* Jason Sofonia (Twitter)
* Bryan Haggerty (ex-Twitter)
* Nathan Downs (ex-Twitter)
* Tom Wuttke (ex-Twitter)
* Andrea Giammarchi (ex-Twitter)
* Joen Asmussen (WordPress)
* Marcus Kazmierczak (WordPress)
* John Woods (PHPizza)

The goal of this project is to provide emoji for everyone and not cause political problems in the process. We definitely welcome improvements and fixes, but we may not merge every pull request suggested by the community due to the simple nature of the project.

The rules for contributing are available in the `CONTRIBUTING.md` file.

Thank you to all of our [contributors](https://github.com/johnnycharlesw/phemoji/graphs/contributors).

## License

Copyright 2019 Twitter, Inc and other contributors

Code licensed under the MIT License: <http://opensource.org/licenses/MIT>

Graphics licensed under CC-BY 4.0: <https://creativecommons.org/licenses/by/4.0/>
