# Dynamic Range Limit CSS Property

## Introduction

This proposal introduces a new CSS property that can be used to control the
maximum brightness of high dynamic range content on the web.

## Background

### HDR Display

A display is considered high dynamic range (HDR) if it can display colors
brighter than white (`#FFFFFF`). The HDR capability of a display is
parameterized by its HDR headroom, which defined as the ratio between the
maximum brightness that the display can currently produce to the brightness
of white.

A display which is not HDR (has an HDR headroom of 1) is called standard
dynamic range (SDR).

### HDR Content

Content is considered high dynamic range (HDR) if it contains colors that
are intended to be rendered brighter than white.

Existing HDR content includes:

* Videos that use the Perceptual Quantizer (PQ) and Hybrid Log-Gamma (HLG)
  transfer functions as described in
  [ITU-R BT.2100](https://www.itu.int/rec/R-REC-BT.2100).
* Images that use the PQ and HLG transfer functions (as described in
  [ISO/TS 22028-5](https://www.iso.org/standard/81863.html)).
* Images that include a
  [HDR Gain Map](https://helpx.adobe.com/si/camera-raw/using/gain-map.html).

Future HDR content could include HTML canvas and CSS colors.

### Tonemapping Content to a Display

When HDR content is rendered to a display, it is transformed by a tone mapping
operation to produce an image that fits within the display's HDR headroom.

A tone mapping operation depends on the content being displayed. Examples of
tone mapping algorithms may be found in:
* [ITU-R BT.2408 Annex 5](https://www.itu.int/pub/R-REP-BT.2408)
* [ITU-R BT.2446-1, Section 4.1](https://www.itu.int/pub/R-REP-BT.2446)
* [SMPTE ST 2094, Annex B](https://ieeexplore.ieee.org/document/9095450)
* [HDR Gain Map](https://helpx.adobe.com/si/camera-raw/using/gain-map.html)

These tone mapping operations are directly parameterized a target HDR headroom,
or can be configured to be parameterized by a target HDR headroom.

## Use Cases

Several applications that show galleries of images and videos have requested
an ability to reduce the HDR headroom in their gallery view, so that all
contents of the gallery have a nearly uniform brightness.

Some applications have requested to enforce that all images be strictly SDR,
while some applications have requested that HDR images exceed SDR, but only
slightly.

Applications have requested the ability to remove this HDR headroom limit when
hovering the image, or when an image is selected for viewing. When the
HDR headroom limit for an element is changed, applications have requested that
the transition be smoothly animated, to avoid a jarring "pop" in brightness.

## Goal

The goal of this proposal is to provide a mechansim through which an application
may limit the maximum brightness of all HDR content (including images, videos,
and in the future HTML canvases and CSS colors).

## Non-goals

A non-goal of this proposal is to provide a mechanism through which a maximum
brightness of HDR content can be enforced on child elements of a given element.

## Privacy and Security

The precise HDR headroom value of a display is a fingerprinting vector, and
should not be exposed.

## API Proposal

Add a new CSS data type, `named-dynamic-range-limit`, which specifies a qualitative
maximum HDR headroom its values are:

* `standard`, which indicates that the content should render as SDR
  (equivalently, with an HDR headroom of 1).
* `high`, which indicates that the full HDR headroom available to the display
  should be used.
* `constrained-high`, which indicates that the content should render with a an HDR
  headroom lower than the maximum, but slightly higher than SDR. The precise
  underlying HDR headroom value that this corresponds is decided by the user
  agent, and may depend on the capabilities of the display and ambient viewing
  conditions.

Add a few CSS functional, `mix-dynamic-range-limit()` which allows for
interpolation between two `named-dynamic-range-limit` values, by a specified
percent. This is required for smooth animation between `named-dynamic-range-limit`
values.

Add a new CSS property, `dynamic-range-limit`, which specifies the maximum HDR
headroom to be used for rendering. This can be set to a `named-dynamic-range-limit`
or the `mix-dynamic-range-limit()` functional.

This property has a default value of `high` indicating that the full display HDR
headroom is to be used.

This property is animatable. Animating of this property has been requested, e.g,
when hovering over or selecting an image.

This property is inherited. It is useful to specify a headroom for a gallery's
container.

### Related APIs

The
[`dynamic-range` CSS media query](https://www.w3.org/TR/mediaqueries-5/#dynamic-range),
can indicate values of `standard` and `high`.

The macOS `UIImageView` API has a similar [`preferredImageDynamicRange`](https://developer.apple.com/documentation/uikit/uiimageview/4173133-preferredimagedynamicrange) property which can take values of `UIImageDynamicRangeStandard`, `UIImageDynamicRangeHigh`, and `UIImageDynamicRangeConstrainedHigh`.

## API Alternatives and Extensions

There are several other APIs that have been considered.

### Specifying an absolute HDR headroom value

Allow setting `dynamic-range-limit` to a number representing an absolute HDR
headroom value.

A deficiency of this model is that there is no way to specify an equivalent
of `high`. Doing so would require knowing the output device's HDR headroom,
which is a fingerprinting vector.

This model also does not address the requests from this feature. There have
been no requests for explicit control over the precise HDR headroom value,
there has been concern expressed about having to tune parameter to achieve
a `constrained-high` effect, and there has been concern about a lack of uniformity
of different applications arriving at different tunings to achieve the
`constrained-high` effect.

### Specifying an relative HDR headroom value

Allow setting `dynamic-range-limit` to a percent representing a percentage of
the output display's HDR headroom.

While this model does allow specifying an equialent of `standard` and `high`,
it suffers similar issues to the absolute model with respect to achieving
a `constrained-high` effect.

### Discussion

The suggested API does not exclude the possibility of adding the above
absolute and relative values for `dynamic-range-limit` if they prove to be
useful additions.

As mentioned in the Privacy and Security section, the precise HDR headroom
value of a display cannot be exposed. There is a risk of exposing that value
if the named, absolute, and relative HDR headroom values can be correlated.
To avoid this, all interpolation between HDR headroom values must return the
`mix-dynamic-range-limit()` functional.

## Example

The following shows an example gallery where content can be brighter than SDR,
but will not use the full HDR range.

```html
  <p style='dynamic-range-limit:constrained-high;'>
    <img src='TestImage0.avif'/>
    <img src='TestImage1.avif'/>
    <video src='TestVideo.vp9'/>
  </p>
```

The following shows the same gallery where the second image has removed its
limit.

```html
  <p style='dynamic-range-limit:constrained-high;'>
    <img src='TestImage0.avif'/>
    <img src='TestImage1.avif' style='dynamic-range-limit:high;'/>
    <video src='TestVideo.vp9'/>
  </p>
```

