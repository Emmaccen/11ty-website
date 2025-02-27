---
pageTitle: Image
eleventyNavigation:
  key: Image
  order: -.2
  excerpt: A utility to resize and generate images.
communityLinksKey: image
---
{% tableofcontents %}

Low level utility to perform build-time image transformations for both vector and raster images. Output multiple sizes, save multiple formats, cache remote images locally. Uses the [sharp](https://sharp.pixelplumbing.com/) image processor.

* [`eleventy-img` on GitHub](https://github.com/11ty/eleventy-img)

You maintain full control of the HTML. Use with `<picture>` or `<img>` or CSS `background-image`, or others! Works great to add `width` and `height` to your images!

* Accepts: `jpeg`, `png`, `webp`, `gif`, `tiff`, `avif` {% addedin "Image 0.6.0" %}, and `svg`.
* Output multiple sizes, keeps original aspect ratio. Never upscales raster images larger than original size (exception for SVG input).
* Output multiple formats, supports: `jpeg`, `png`, `webp`, `avif` {% addedin "Image 0.6.0" %}, and `svg` (requires SVG input)
* Retrieve metadata about your new images (see [sample return object](#sample-return-object)).
  * Use this to add `width` and `height` attributes on `<img>` elements for [proper aspect ratio mapping](https://developer.mozilla.org/en-US/docs/Web/Media/images/aspect_ratio_mapping).
* Does not require or rely on file extensions (like `.png` or `.jpg`) in URLs or local files, which may be missing or inaccurate.
* Save remote images locally using [`eleventy-cache-assets`](/docs/plugins/cache/).
  * Use local images in your HTML to prevent broken image URLs.
  * Manage the [cache duration](/docs/plugins/fetch/#change-the-cache-duration).
* Fast: de-duplicates image requests and use both an in-memory and disk cache.


## Installation

* [`eleventy-img` on npm](https://www.npmjs.com/package/@11ty/eleventy-img)

```
npm install @11ty/eleventy-img
```

## Usage

This utility returns a Promise and works best in `async` friendly functions, filters, shortcodes. It _can_ also work in synchronous environments ([Synchronous Usage](#synchronous-shortcode)).

{% codetitle ".eleventy.js" %}

```js
const Image = require("@11ty/eleventy-img");

(async () => {
  let url = "https://images.unsplash.com/photo-1608178398319-48f814d0750c";
  let stats = await Image(url, {
    widths: [300]
  });

  console.log( stats );
})();
```

Three things happen here:

1. If the first argument is a full URL (not a local file path), we download [the remote image](https://unsplash.com/photos/uXchDIKs4qI) and cache it locally using the [Cache plugin](/docs/plugins/fetch/). This cached original is then used for the cache duration to avoid a bunch of network requests.
2. From that cached full-size original, images are created for each format and width, in this case: `./img/6dfd7ac6-300.webp` and `./img/6dfd7ac6-300.jpeg`.
3. The metadata object is populated and returned, describing those new images:

<div id="sample-return-object"></div>

```js
{
  webp: [
    {
      format: 'webp',
      width: 300,
      height: 300,
      filename: '6dfd7ac6-300.webp',
      outputPath: 'img/6dfd7ac6-300.webp',
      url: '/img/6dfd7ac6-300.webp',
      sourceType: 'image/webp',
      srcset: '/img/6dfd7ac6-300.webp 300w',
      size: 10184
    }
  ],
  jpeg: [
    {
      format: 'jpeg',
      width: 300,
      height: 300,
      filename: '6dfd7ac6-300.jpeg',
      outputPath: 'img/6dfd7ac6-300.jpeg',
      url: '/img/6dfd7ac6-300.jpeg',
      sourceType: 'image/jpeg',
      srcset: '/img/6dfd7ac6-300.jpeg 300w',
      size: 15616
    }
  ]
}
```

Here’s the output images, one webp and one jpeg:

{% callout "demo" %}

<img src="/img/plugins/image/6dfd7ac6-300.webp" alt="the webp output created by this Image plugin (it’s a nebula)" width="300" height="300">

<img src="/img/plugins/image/6dfd7ac6-300.jpeg" alt="the jpeg output created by this Image plugin (it’s a nebula)" width="300" height="300">

{% endcallout %}


### Output Widths

Controls how many output images will be created for each image format. Aspect ratio is preserved.

* `widths: ["auto"]` (default, keep original width) `"auto"` was added ({% addedin "Image 0.8.1" %}) as a clearer alias for the previously recommended `widths: [null]`.
* `widths: [200]` (output one 200px width)
* `widths: [200, "auto"]` (output 200px and original width)

### Output Formats

Use almost any combination of these:

* `formats: ["webp", "jpeg"]` (default)
* `formats: ["png"]`
* `formats: ["auto"]` (keep original format) `"auto"` was added ({% addedin "Image 0.8.1" %}) as a clearer alias for the previously recommended `formats: [null]` ({% addedin "Image 0.4.0" %}).
* `formats: ["svg"]` (requires SVG input) {% addedin "Image 0.4.0" %}
* `formats: ["avif"]` {% addedin "Image 0.6.0" %}

### Output Locations

#### URL Path

A path-prefix-esque directory for the `<img src>` attribute. e.g. `/img/` for `<img src="/img/MY_IMAGE.jpeg">`:

* `urlPath: "/img/"` (default)

#### Output Directory

Where to write the new images to disk. Project-relative path to the output image directory. Maybe you want to write these to your output directory directly (e.g. `./_site/img/`)?

* `outputDir: "./img/"` (default)

### Caching Remote Images Locally

{% addedin "Image 0.3.0" %} For any full URL first argument to this plugin, the full-size remote image will be downloaded and cached locally. See all [relevant `eleventy-fetch` options](/docs/plugins/fetch/#options).

```js
{
  cacheOptions: {
    // if a remote image URL, this is the amount of time before it fetches a fresh copy
    duration: "1d",

    // project-relative path to the cache directory
    directory: ".cache",

    removeUrlQueryParams: false,
  },
}
```

### Options for SVG

#### Skip raster formats for SVG

{% addedin "Image 0.4.0" %} If using SVG output (the input format is SVG and `svg` is added to your `formats` array), we will skip all of the raster formats even if they’re in `formats`. This may be useful in a CMS-driven workflow when the input could be vector or raster.

* `svgShortCircuit: false` (default)
* `svgShortCircuit: true`

#### Allow SVG to upscale

{% addedin "Image 0.4.0" %} While we do prevent raster images from upscaling (and filter upscaling `widths` from the output), you can optionally enable SVG input to upscale to larger sizes when converting to raster format.

* `svgAllowUpscale: true` (default)
* `svgAllowUpscale: false`

### Custom Filenames

{% addedin "Image 0.4.0" %} Don’t like those hash ids? Make your own!

```js
{
  // Define custom filenames for generated images
  filenameFormat: function (id, src, width, format, options) {
    // id: hash of the original image
    // src: original image path
    // width: current width in px
    // format: current file format
    // options: set of options passed to the Image call

    return `${id}-${width}.${format}`;
  }
}
```

<details>
<summary>Custom Filename Example: Use the original file slug</summary>

```js
const path = require("path");
const Image = require("@11ty/eleventy-img");

await Image("./test/bio-2017.jpg", {
  widths: [300],
  formats: ["auto"],
  filenameFormat: function (id, src, width, format, options) {
    const extension = path.extname(src);
    const name = path.basename(src, extension);

    return `${name}-${width}w.${format}`;
  }
});

// Writes: "test/img/bio-2017-300w.jpeg"
```

</details>

### Using a Hosted Image Service

#### Custom URLs

{% addedin "Image 0.8.3" %} Want to use a hosted image service instead? You can override the entire URL. Takes precedence over `filenameFormat` option. Useful with `statsSync` or `statsByDimensionsSync`.

When you use this, returned data will not include `filename` or `outputPath`.

```js
{
  urlFormat: function ({
    hash, // not included for `statsOnly` images
    src,
    width,
    format,
  }) {
    return `https://sample-image-service.11ty.dev/${encodeURIComponent(src)}/${width}/${format}/`;
  }
}
```

* [_Example on `11ty-website`_](https://github.com/11ty/11ty-website/blob/516faa397a98f8990f3d02eb41e1c99bedfab9cf/.eleventy.js#L105)

#### Stats Only

{% addedin "Image 1.1.0" %} Skips all image processing to return metadata. Doesn’t read files, doesn’t write files. Use this as an alternative to the separate `statsSync*` functions—this will use in-memory cache and de-duplicate requests.

* `statsOnly: false` (default)
* `statsOnly: true`

With remotely hosted images, you may also need to supply the dimensions when using `statsOnly`:

```js
{
  statsOnly: true,
  remoteImageMetadata: {
    width,
    height,
    format, // optional
  }
}
```


### Change the default Hash length

{% addedin "1.0.0" %} You can customize the length of the default filename format hash by using the `hashLength` property.

```js
const Image = require("@11ty/eleventy-img");

await Image("./test/bio-2017.jpg", {
  hashLength: 8 // careful, don’t make it _too_ short!
});
```

## Use this in your templates

### Asynchronous Shortcode

{% callout "info" %}The examples below use a <a href="/docs/languages/nunjucks/#asynchronous-shortcodes">Nunjucks</a> <code>async</code> shortcode (different from the traditional shortcode configuration method). The <a href="/docs/languages/javascript/#asynchronous-javascript-template-functions">JavaScript</a> and <a href="/docs/languages/liquid/#asynchronous-shortcodes">Liquid</a> template engines also work here and are asynchronous without additional changes. Note that <a href="https://mozilla.github.io/nunjucks/templating.html#macro">Nunjucks macros cannot use asynchronous shortcodes</a>. If you use macros, use Synchronous shortcodes described below.{% endcallout %}

<is-land on:visible import="/js/seven-minute-tabs.js">
<seven-minute-tabs>
  <div role="tablist" aria-label="Easy or DIY mode chooser">
    Choose one:
    <a href="#filter-easy" role="tab">We generate the HTML</a>
    <a href="#filter-diy-img" role="tab">Do it yourself: &lt;img&gt;</a>
    <a href="#filter-diy-picture" role="tab">Do it yourself: &lt;picture&gt;</a>
  </div>
  <div id="filter-easy" role="tabpanel">

{% addedin "Image 0.7.2" %}The `generateHTML` function is available in Eleventy Image v0.7.2 or higher.

{% codetitle ".eleventy.js" %}

```js
const Image = require("@11ty/eleventy-img");

async function imageShortcode(src, alt, sizes) {
  let metadata = await Image(src, {
    widths: [300, 600],
    formats: ["avif", "jpeg"]
  });

  let imageAttributes = {
    alt,
    sizes,
    loading: "lazy",
    decoding: "async",
  };

  // You bet we throw an error on missing alt in `imageAttributes` (alt="" works okay)
  return Image.generateHTML(metadata, imageAttributes);
}

module.exports = function(eleventyConfig) {
  eleventyConfig.addAsyncShortcode("image", imageShortcode);

  // Or add them individually
  eleventyConfig.addNunjucksAsyncShortcode("image", imageShortcode);
  eleventyConfig.addLiquidShortcode("image", imageShortcode);
  eleventyConfig.addJavaScriptFunction("image", imageShortcode);

};
```

{% callout "info", "md" %}You’re only allowed one `module.exports` in your configuration file! If one already exists, copy the content of the above into your existing `module.exports` function.{% endcallout %}

{% addedin "Image 0.7.3" %}You can use the `whitespaceMode` option to strip the whitespace from the output of the `<picture>` element (a must-have for use in markdown files).

```js
async function imageShortcode(src, alt, sizes) {
  // […]
  return Image.generateHTML(metadata, imageAttributes, {
    whitespaceMode: "inline"
  });
}

// Don’t copy and paste this code block!
// Some code from the above example was removed for brevity.
```

  </div>
  <div id="filter-diy-img" role="tabpanel">

{% codetitle ".eleventy.js" %}

```js
const Image = require("@11ty/eleventy-img");

async function imageShortcode(src, alt) {
  if(alt === undefined) {
    // You bet we throw an error on missing alt (alt="" works okay)
    throw new Error(`Missing \`alt\` on myImage from: ${src}`);
  }

  let metadata = await Image(src, {
    widths: [600],
    formats: ["jpeg"]
  });

  let data = metadata.jpeg[metadata.jpeg.length - 1];
  return `<img src="${data.url}" width="${data.width}" height="${data.height}" alt="${alt}" loading="lazy" decoding="async">`;
}

module.exports = function(eleventyConfig) {
  eleventyConfig.addAsyncShortcode("image", imageShortcode);

  // Or add them individually
  eleventyConfig.addNunjucksAsyncShortcode("image", imageShortcode);
  eleventyConfig.addLiquidShortcode("image", imageShortcode);
  eleventyConfig.addJavaScriptFunction("image", imageShortcode);
};
```

{% callout "info", "md" %}You’re only allowed one `module.exports` in your configuration file! If one already exists, copy the content of the above into your existing `module.exports` function.{% endcallout %}

  </div>
  <div id="filter-diy-picture" role="tabpanel">

{% codetitle ".eleventy.js" %}

```js
const Image = require("@11ty/eleventy-img");

async function imageShortcode(src, alt, sizes = "100vw") {
  if(alt === undefined) {
    // You bet we throw an error on missing alt (alt="" works okay)
    throw new Error(`Missing \`alt\` on responsiveimage from: ${src}`);
  }

  let metadata = await Image(src, {
    widths: [300, 600],
    formats: ['webp', 'jpeg']
  });

  let lowsrc = metadata.jpeg[0];
  let highsrc = metadata.jpeg[metadata.jpeg.length - 1];

  return `<picture>
    ${Object.values(metadata).map(imageFormat => {
      return `  <source type="${imageFormat[0].sourceType}" srcset="${imageFormat.map(entry => entry.srcset).join(", ")}" sizes="${sizes}">`;
    }).join("\n")}
      <img
        src="${lowsrc.url}"
        width="${highsrc.width}"
        height="${highsrc.height}"
        alt="${alt}"
        loading="lazy"
        decoding="async">
    </picture>`;
}

module.exports = function(eleventyConfig) {
  eleventyConfig.addAsyncShortcode("image", imageShortcode);

  // Or add them individually
  eleventyConfig.addNunjucksAsyncShortcode("image", imageShortcode);
  eleventyConfig.addLiquidShortcode("image", imageShortcode);
  eleventyConfig.addJavaScriptFunction("image", imageShortcode);
};
```

{% callout "info", "md" %}You’re only allowed one `module.exports` in your configuration file! If one already exists, copy the content of the above into your existing `module.exports` function.{% endcallout %}

  </div>
</seven-minute-tabs>
</is-land>

{% callout "info", "md" %}Read more about the [`loading`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/img#attr-loading) and [`decoding`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/img#attr-decoding) HTML attributes.{% endcallout %}

Now you can use it in your templates:

<is-land on:visible import="/js/seven-minute-tabs.js">
<seven-minute-tabs>
  {% renderFile "./src/_includes/syntax-chooser-tablist.11ty.js", {id: "shortcode"} %}
  <div id="shortcode-liquid" role="tabpanel">
    {% codetitle "Liquid", "Syntax" %}
{%- highlight "liquid" %}{% raw %}
{% image "./src/images/cat.jpg", "photo of my cat" %}
{% image "./src/images/cat.jpg", "photo of my cat", "(min-width: 30em) 50vw, 100vw" %}
{% endraw %}{% endhighlight %}
    <p>The comma between arguments is <strong>optional</strong> in Liquid templates.</p>
  </div>
  <div id="shortcode-njk" role="tabpanel">
    {% codetitle "Nunjucks", "Syntax" %}
{%- highlight "jinja2" %}{% raw %}
{% image "./src/images/cat.jpg", "photo of my cat" %}
{% image "./src/images/cat.jpg", "photo of my cat", "(min-width: 30em) 50vw, 100vw" %}
{% endraw %}{% endhighlight %}
    <p>The comma between arguments is <strong>required</strong> in Nunjucks templates.</p>
  </div>
  <div id="shortcode-js" role="tabpanel">
    {% codetitle "JavaScript", "Syntax" %}
{%- highlight "js" %}{% raw %}
module.exports = function() {
  let img1 = await this.image("./src/images/cat.jpg", "photo of my cat");
  let img2 = await this.image("./src/images/cat.jpg", "photo of my cat", "(min-width: 30em) 50vw, 100vw");

  return `${img1}
${img2}`;
};
{% endraw %}{% endhighlight %}
  </div>
  <div id="shortcode-hbs" role="tabpanel">

This `image` shortcode example [requires an async-friendly template language](#asynchronous-usage) and is not available in Handlebars.

  </div>
</seven-minute-tabs>
</is-land>

And you’ll have the appropriate HTML generated for you (based on your specified Image options).

### Synchronous Shortcode

Use `Image.statsSync` to get the metadata of a source even if the image generation is not finished yet:

{% codetitle ".eleventy.js" %}

```js
const Image = require("@11ty/eleventy-img");
function imageShortcode(src, cls, alt, sizes, widths) {
  let options = {
    widths: widths,
    formats: ['jpeg'],
  };

  // generate images, while this is async we don’t wait
  Image(src, options);

  let imageAttributes = {
    class: cls,
    alt,
    sizes,
    loading: "lazy",
    decoding: "async",
  };
  // get metadata even if the images are not fully generated yet
  let metadata = Image.statsSync(src, options);
  return Image.generateHTML(metadata, imageAttributes);
}

module.exports = function(eleventyConfig) {
  eleventyConfig.addNunjucksShortcode("myImage", imageShortcode);
}
```

### Process images as a Custom Template

Use Eleventy’s [Custom Template Language](/docs/languages/custom/) feature to process images. _This one is not yet available on the docs: do you want to contribute it?_

### Process images as Data Files

{% addedin "2.0.0-canary.10" %} _Nontraditional use case._ Eleventy’s [Custom Data File Formats](/docs/data-custom/) features an example of [processing Images as data files to feed EXIF data into the Data Cascade](/docs/data-custom/#feed-exif-image-data-into-the-data-cascade). You can use the same feature to add the metadata stats returned from the Image utility directly to the Data Cascade for use in your templates.

* Benefits:
  * Processing happens in the data cascade so this works in any template language.
  * Images stored in the [global data folder](/docs/data-global/) will be processed and available to all templates
* Drawbacks:
  * You can’t customize the Image options (e.g. `widths` or `formats`) from the template code. It is set globally in the config.
* Both a benefit and a drawback:
  * Beholden to Eleventy’s Data Cascade file name conventions when using with [template/directory data files](/docs/data-template-dir/).

<details>
  <summary><strong>Show the code</strong></summary>

{% codetitle ".eleventy.js" %}

```js
const Image = require("@11ty/eleventy-img");
const path = require("path");

module.exports = function(eleventyConfig) {
  eleventyConfig.addDataExtension("png,jpeg", {
    read: false, // Don’t read the input file, argument is now a file path
    parser: async imagePath => {
      let stats = await Image(imagePath, {
        widths: ["auto"],
        formats: ["avif", "webp", "jpeg"],
        outputDir: path.join(eleventyConfig.dir.output, "img", "built")
      });

      return {
        image: {
          stats
        }
      };
    },
  });

  // This works sync or async: images were processed ahead of time in the data cascade
  eleventyConfig.addShortcode("dataCascadeImage", (stats, alt, sizes) => {
    let imageAttributes = {
      alt,
      sizes,
      loading: "lazy",
      decoding: "async",
    };
    return Image.generateHTML(stats, imageAttributes);
  });
};
```

With a template `my-blog-post.md` and an image file `my-blog-post.jpeg`, you could use the above configuration code in your template like this:

{% codetitle "my-blog-post.md" %}

{% raw %}
```liquid
{% dataCascadeImage image.stats, "My alt text" %}
```
{% endraw %}

Note this also means that `folder/folder.jpeg` would be processed for all templates in `folder/*` and any images stored in your global `_data` would also be populated into the data cascade based on their folder structure.

</details>

<div class="youtube-related">
  {%- youtubeEmbed "oCTAZumAGNc", "Use images as data files (Weekly №11)", "244" -%}
</div>

## Advanced Usage

### Caching

#### In-Memory Cache

{% addedin "Image 0.7.0" %} To prevent duplicate work and improve build performance, repeated calls to the same source image (remote or local) with the same options will return a cached results object. If a request in-progress, the pending promise will be returned. This in-memory cache is maintained across builds in watch/serve mode. If you quit Eleventy, the in-memory cache will be lost.

Images will be regenerated (and the cache ignored) if:

* The source image file size changes (on local image files)
* The [cache asset](/docs/plugins/fetch/) duration expires (for remote images).

You can disable this behavior by using the `useCache` boolean option:

* `useCache: true` (default)
* `useCache: false` to bypass the cache and generate a new image every time.

##### Examples

<details>
<summary>Example of in-memory cache reuse (returns the same promise)</summary>

{% codetitle ".eleventy.js" %}

```js
const Image = require("@11ty/eleventy-img");

(async () => {
  let stats1 = Image("./test/bio-2017.jpg");
  let stats2 = Image("./test/bio-2017.jpg");

  console.assert(stats1 === stats2, "The same promise");
})();
```

</details>
<details>
<summary>Example of in-memory cache (returns a new promise with different options)</summary>

{% codetitle ".eleventy.js" %}

```js
const Image = require("@11ty/eleventy-img");

(async () => {
  let stats1 = Image("./test/bio-2017.jpg");
  let stats2 = Image("./test/bio-2017.jpg", { widths: [300] });

  console.assert(stats1 !== stats2, "A different promise");
})();
```

</details>

#### Disk Cache

{% addedin "Image 1.0.0" %} Eleventy will skip processing files that are unchanged and already exist in the output directory. This requires the built-in hashing algorithm and is not yet supported with custom filenames. More background at <a href="https://github.com/11ty/eleventy-img/issues/51">Issue #51</a>.

New tip: [**Re-use and persist the disk cache across Netlify builds**](https://github.com/11ty/demo-eleventy-img-netlify-cache)

### Dry-Run

{% addedin "Image 0.7.0" %} If you want to try it out and not write any files (useful for testing), use the `dryRun` option.

* `dryRun: false` (default)
* `dryRun: true`

### Change Global Plugin Concurrency

```js
const Image = require("@11ty/eleventy-img");
Image.concurrency = 4; // default is 10
```

### Advanced control of Sharp image processor

[Extra options to pass to the Sharp constructor](https://sharp.pixelplumbing.com/api-constructor#parameters) or the [Sharp image format converter for webp](https://sharp.pixelplumbing.com/api-output#webp), [png](https://sharp.pixelplumbing.com/api-output#png), [jpeg](https://sharp.pixelplumbing.com/api-output#jpeg), or [avif](https://sharp.pixelplumbing.com/api-output#avif).

* `sharpOptions: {}` {% addedin "Image 0.4.0" %}
* `sharpWebpOptions: {}` {% addedin "Image 0.4.2" %}
* `sharpPngOptions: {}` {% addedin "Image 0.4.2" %}
* `sharpJpegOptions: {}` {% addedin "Image 0.4.2" %}
* `sharpAvifOptions: {}` {% addedin "Image 0.6.0" %}

### Output animated GIF or WebP with Sharp

{% addedin "Image 1.1.0" %} To process and output animated `gif` or `webp` images, use the `animated` option for the Sharp constructor.

```js
const Image = require("@11ty/eleventy-img");
const options = {
  // Your other options ...
  formats: ['webp', 'gif']
  sharpOptions: {
    animated: true
  }
}
const metadata = await Image(src, options)
```
