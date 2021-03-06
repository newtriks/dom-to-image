# DOM to Image

## What is it
**dom-to-image** is a library which can turn arbitrary DOM node into a vector (SVG) or raster (PNG) image, written in JavaScript. It's based on [domvas by Paul Bakaus](https://github.com/pbakaus/domvas) and has been completely rewritten, with some bugs fixed and some new features (like web font and image support) added.

## Usage
All the top level functions accept DOM node and rendering options, and return promises, which are fulfilled with corresponding data URLs.  
Get a PNG image base64-encoded data URL and display right away:
```javascript
var node = document.getElementById('my-node');

domtoimage.toPng(node)
    .then(function (dataUrl) {
        var img = new Image();
        img.src = dataUrl;
        document.appendChild(img);
    })
    .catch(function (error) {
        console.error('oops, something went wrong!', error);
    });
```
Get a PNG image blob and download it (using [FileSaver](https://github.com/eligrey/FileSaver.js/), for example):
```javascript
domtoimage.toBlob(document.getElementById('my-node'))
    .then(function (blob) {
        window.saveAs(blob, 'my-node.png');
    });
```
Get an SVG data URL, but filter out all the `<i>` elements:
```javascript
function filter (node) {
    return (node.tagName !== 'i');
}

domtoimage.toSvg(document.getElementById('my-node'), {filter: filter})
    .then(function (dataUrl) {
        /* do something */
    });
```
All the functions under `impl` are not public API and are exposed only for unit testing.

## Browsers
It's tested on latest Chrome and Firefox (44 and 40 respectively at the time of writing), with Chrome performing  significantly better on big DOM trees, possibly due to it's more performant SVG support, and the fact that it supports `CSSStyleDeclaration.cssText` property.

## Dependencies

### Source
Only standard lib is currently used, but make sure your browser supports:  
* [Promise](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Promise)  
* SVG `<foreignObject>` tag

### Tests
Most importantly, tests depend on:
* [js-imagediff](https://github.com/HumbleSoftware/js-imagediff), to compare rendered and control images
* [ocrad.js](https://github.com/antimatter15/ocrad.js), for the parts when you can't compare images (due to the browser
rendering differences) and just have to test whether the text is rendered

## How it works
There might some day exist (or maybe already exists?) a simple and standard way of exporting parts of the HTML to image (and then this script can only serve as an evidence of all the hoops I had to jump through in order to get such obvious thing done) but I haven't found one so far.  

This library uses a feature of SVG that allows having arbitrary HTML content inside of the `<foreignObject>` tag. So, in order to render that DOM node for you, following steps are taken:  

1. Clone the original DOM node recursively
2. Compute the style for the node and each sub-node and copy it to corresponding clone
  * and don't forget to recreate pseudo-elements, as they are not cloned in any way, of course
3. Embed web fonts
  * find all the `@font-face` declarations that might represent web fonts
  * parse file URLs, download corresponding files
  * base64-encode and inline content as `data:` URLs
  * concatenate all the processed CSS rules and put them into one `<style>` element, then attach it to the clone
4. Embed images
  * embed image URLs in `<img>` elements
  * inline images used in `background` CSS property, in a fashion similar to fonts
5. Serialize the cloned node to XML
6. Wrap XML into the `<foreignObject>` tag, then into the SVG, then make it a data URL
7. Optionally, to get PNG content, create an Image element with the SVG as a source, and render it on an off-screen canvas, that you have also created, then read the content from the canvas
9. Done!  

## Things to watch out for  
* if the DOM node you want to render includes a `<canvas>` element with something drawn on it, it should be handled fine, unless the canvas is [tainted](https://developer.mozilla.org/en-US/docs/Web/HTML/CORS_enabled_image) - in this case rendering will rather not succeed.  
* at the time of writing, Firefox has a problem with some external stylesheets (see #13). In such case, the error will be caught and logged.  

## Authors
Anatolii Saienko, Paul Bakaus (original idea)

## License
MIT
