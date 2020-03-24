[< Back home](/images-responsiver/#documentation)

# Tutorial

## Default behavior without the plugin

Let's say you have this HTML file:

<script src="https://gist-it.appspot.com/github/nhoizey/images-responsiver/raw/master/examples/01_default/page.html"></script>

And this CSS file:

<script src="https://gist-it.appspot.com/github/nhoizey/images-responsiver/raw/master/examples/01_default/styles.css"></script>

We want the content to occupy 90% of the available space (but no more than `40em`, better for readability of multi-lines text), and the logo to use 50% of this content width, floated on the right.

The page probably looks exactly how you want, thanks to the clean HTML structure and the CSS rules.

But each image is available in only one single dimension (large probably), even if people with many different devices/browsers, with different viewport widths, would rather download only what's necessary.

## Enhanced behavior with `images-responsiver` and default configuration

Now let's try to run `images-responsiver` on the HTML to enhance it.

You can use this Node.js script:

<script src="https://gist-it.appspot.com/github/nhoizey/images-responsiver/raw/master/examples/01_default/run.js"></script>

Run it from the command line:

```bash
node run.js
```

You'll get the enhanced page in this new HTML file:

<script src="https://gist-it.appspot.com/github/nhoizey/images-responsiver/raw/master/examples/01_default/page-enhanced.html"></script>

_Note: a `pristine` value is added to the image's dataset with the original URL, in case you want to do anything else with it later (provide a "zoom" link for example)._

The situation is better, because users with small viewports (and reasonable screen densities) will download smaller images.

But there are a few issues:

- The `sizes` attributes with a `100vw` value tells the browser that the image will be rendered on the full width of the viewport, but that's not what we want:
  - We know the second image (`my-photo.jpg`) should occupy only the width of the content, which per CSS rules is `90vw` with a maximum of `40em`. This `40em` width for the content is reached when the viewport reaches `40 / 0.9 = 45em` (rounded). So we should be able to set a `sizes` attribute with the value `(max-width: 45em) 90vw, 40em` (or `(min-width: 45em) 40em, 90vw` but the result is the same).
  - For the logo (the first image), we need one fifth of the content width, so the `sizes` attribute value is simple math from the previous one: `(max-width: 45em) 18vw, 8em`.
- If the maximum width for the logo is `8em` on largest viewports, and most users have a browser with a default root font size of `16px`, these `8em` are computed to `128px`. Let's consider the user is on a high density display ("Retina" in Apple language), so double it, and that [the user might need to increase the font size for readability](https://nicolas-hoizey.com/articles/2018/06/15/users-do-change-font-size/), so double it a second time. We then need an image with a maximum useful width of `512px`. We see the HTML tells the browser that the maximum width for the image is `2560px` (the `w` descriptor is the "`w`idth in pixels"). Far beyond what we need! We should be able to set a maximum (and sometimes a minimum) for the sequence of image widths in the `srcset` attribute.

These issues exist because there is no default configuration that would be correct for all use cases. You have to tell the plugin what to do with the images:

## Enhanced behavior with some configuration

We need different `sizes` attribute values for different image use cases, but we don't want to repeat them for each images, and we want to provide content authors (even if that's us) with something as simple as possible so that they can focus on content.

Let's define **presets**, and configuration options attached to them.

First, we define a `default` preset that will be used for all images where the author doesn't set anything special.

```javascript
const options = {
  default: {
    sizes: '(max-width: 45em) 90vw, 40em',
  },
};
```

We set the default `sizes` attribute value, considering we will never have any image larger than the content. It overrides the plugin's default value of `100vw`.

We now need to add a specific preset for the logo, which has different needs:

```javascript
const options = {
  default: {
    sizes: '(max-width: 45em) 90vw, 40em',
  },
  logo: {
    minWidth: 58,
    maxWidth: 512,
    steps: 3,
    fallbackWidth: 128,
    sizes: '(max-width: 45em) 18vw, 8em',
  },
};
```

The logo takes one fifth of the content width, so on small `320px` viewports with normal screen density it needs `320px * 90% * 1/5 = 58px` (rounded), and on largest viewports on Retina screens, it needs `512px` as we computed earlier.

We also tell the plugin with `steps` that 3 different image widths should be enough, instead of the default 5.

Finally, `fallbackWidth` is the width of the image we put in the `src` attribute for compatibility with really old browsers. Most of these browsers are desktop ones, so don't a value too small.

We also need a way for content authors to specify what preset an image should use, if the default one is not enough. We will use a `data-responsiver` data attribute, with the name of the preset as the value.

So for example, we change the HTML for the logo from:

```html
<img src="my-logo.png" alt="My logo" class="logo" />
```

To:

```html
<img src="my-logo.png" alt="My logo" class="logo" data-responsiver="logo" />
```

We can now run this updated Node.js script:

<script src="https://gist-it.appspot.com/github/nhoizey/images-responsiver/raw/master/examples/02_simple/run.js"></script>

Here's the new enhanced HTML that we get:

<script src="https://gist-it.appspot.com/github/nhoizey/images-responsiver/raw/master/examples/02_simple/page-enhanced.html"></script>

This is really much better, the browser will download images with much smaller dimensions (and weight), yet large enough for a great rendering.

## Making it more robust with image dimensions

But we might still have an issue:

Even if we set a maximum width lower than `2560px` (like `512px` for the logo), we should not be able to define a width that is larger than the actual width of the pristine image, the largest we have before any computing. If we do that, we lie to the browser, and it might render the image at the width we told him, instead of the actual one, resulting in bad rendered quality.

So we should be able to tell the plugin about the actual width of the pristine image.

Why invent a new parameter? We already have the `width` attribute in HTML, let's use it, `images-responsiver` can read it.

_Note: it's anyway always a good idea to have the `width` and `height` attributes defined in images, as [it will enhance the page rendering performance](https://www.youtube.com/watch?v=4-d_SoCHeWE)._

If the pristine image for the logo is `400px` wide, and the other pristine image is `1600px` wide, here's our new source HTML:

<script src="https://gist-it.appspot.com/github/nhoizey/images-responsiver/raw/master/examples/03_dimensions/page.html"></script>

If we run the exact same Node.js script on it:

<script src="https://gist-it.appspot.com/github/nhoizey/images-responsiver/raw/master/examples/03_dimensions/run.js"></script>

The result is further improved:

<script src="https://gist-it.appspot.com/github/nhoizey/images-responsiver/raw/master/examples/03_dimensions/page-enhanced.html"></script>

We should also update the CSS so that we don't try to render the image larger than it is. `width` can be replaced with `max-width`:

<script src="https://gist-it.appspot.com/github/nhoizey/images-responsiver/raw/master/examples/03_dimensions/styles.css"></script>

# Ok, but where and when are my-logo-58.png, my-logo-285.png, etc. generated?

`images-responsiver` doesn't transform images files, it "only" transforms HTML. That's already a lot, as you should have noticed.

You have to define how these multiple width images are generated:

- you can transform them yourself with an asynchronous batch script, but that might be difficult if you don't know the widths there will be in the HTML
- you can parse the enhanced HTML to list the images you need, either outside `images-responsiver` or thanks to the `runAfter` parameter, which is a hook function that runs at the end of the transformation
- you can use dynamic image rendering, computing the required image on the server when it is requested by the browser
  - either with your self hosted solution, with [a simple PHP script](https://css-tricks.com/snippets/php/server-side-image-resizer/) for example, or [thumbor](http://thumbor.org/), an "open-source smart on-demand image cropping, resizing and filters" solution
  - or with an image CDN like Cloudinary, Imgix, Akamai Image Manager, etc.

Some of these solutions might require specific URL for the images to compute, they might not be compatible with image names like `my-logo-58.png`.

## Defining your own URL format

That's why you can use the `resizedImageUrl` function in the options of the plugin. This is the default function:

```javascript
const defaultResizedImageUrl = (src, width) =>
  src.replace(/^(.*)(\.[^\.]+)$/, '$1-' + width + '$2');
```

It transforms `(my-logo.png, 58)` into `my-logo-58.png`.

You can define your own simple function to replace the default one.

For example, if the width has to be a `w` query parameter:

```javascript
const options = {
  resizedImageUrl: (src, width) => `${src}?w=${width}`,
};
```

It will transform `(my-logo.png, 58)` into `my-logo.png?w=58`.

## Using an image CDN

Relying on a third party service might make some fear of losing control, but resizing and optimizing images as much and as good as such services is really hard, and it requires computing power and storage space. Here, it requires "just" an URL.

### Using Cloudinary

For Cloudinary ([sign-up for free](https://nho.io/cloudinary-signup), it should be enough for most personal sites), like explained in [the exemple usage from nicolas-hoizey.com](./nicolashoizeycom.html), here is the `resizedImageUrl` function:

```javascript
(src, width) =>
  `https://res.cloudinary.com/nho/image/fetch/q_auto,f_auto,w_${width}/${src}`,
```

Here, `nho` is the cloud name, linked to my own Cloudinary account.

This URL will:

- resize the pristine image (`${src}`) to the desired width (`w_${width}`),
- chose the best compression level without sacrificing quality (`q_auto`),
- and chose the best encoding format depending of the browser capacity (`f_auto`), for example `WebP`, even if the pristine image is a `JPEG`.

### Using other image CDNs

Feel free to submit a [Pull Request](https://github.com/nhoizey/images-responsiver/pulls) to enhance documentation with other Image CDN examples.

# We can further enhance the image

_To be continued…_

## Adding classes

_To be continued…_

## Adding attributes

_To be continued…_

<!-- https://web.dev/native-lazy-loading/ -->

## Running hooks before and after transformation

_To be continued…_

## Targeting images to transform

`selector`

# Additional informations

- Each image can use multiple presets in the `data-responsiver` attribute, each value separated by a space like for classes.
- Settings from each preset surcharges the previous one(s), in the order they're declared.
- `images-responsiver` don't do anything to:
  - SVG images
  - bitmap images that don't have any `src` attribute
  - bitmap images that already have a `srcset` attribute