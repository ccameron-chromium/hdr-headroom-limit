# Maximum HDR Headroom CSS Property

## Introduction

This proposal introduces a new CSS property that can be used to control the
maximum brightness of high dynamic range content on the web.

## Background

### HDR Display

A display is considered high dynamic range (HDR) if it can display colors
brighter than SDR white (`#FFFFFF`). The HDR capability of a display is
parameterized by its HDR headroom, which defined as the ratio between the
maximum brightness that the display can currently produce to the brightness
of SDR white.

A display which is not HDR (has an HDR headroom of 1) is called standard
dynamic range (SDR).

## HDR Content

Content is considered high dynamic range (HDR) if it contains colors that
are intended to be rendered brighter than SDR white.

Existing HDR content includes:

* Videos that use the Perceptual Quantizer (PQ) and Hybrid Log-Gamma (HLG)
  transfer functions as described in
  [ITU-R BT.2100](https://www.itu.int/rec/R-REC-BT.2100).
* Images that use the PQ and HLG transfer functions (as described in
  [ISO/TS 22028-5](https://www.iso.org/standard/81863.html)).
* Images that include a
  [HDR Gain Map](https://helpx.adobe.com/si/camera-raw/using/gain-map.html).

Future HDR content could include HTML canvas and CSS colors.

## Tonemapping Content to a Display

When HDR content is rendered to a display, it is transformed by a tone mapping
operation to produce an image that fits within the display's HDR headroom.

A tone mapping operation depends on the content being displayed. Examples of
tone mapping algorithms may be found in:
* [ITU-R BT.2408 Annex 5](https://www.itu.int/pub/R-REP-BT.2408)
* [ITU-R BT.2446-1, Section 4.1](https://www.itu.int/pub/R-REP-BT.2446)
* [SMPTE ST 2094, Annex B](https://ieeexplore.ieee.org/document/9095450)
* [Hdr Gain Map](https://helpx.adobe.com/si/camera-raw/using/gain-map.html)

These tone mapping operations are parameterized a target HDR headroom.

## Limiting Tonemapped HDR headroom

By default, HDR content is tone mapped using output display's full HDR headroom
as the target HDR headroom parameter.

In this proposal, we introduce a CSS property that specifies a maximum HDR
headroom parameter value to use during tone mapping. This has the effect of
limiting how bright HDR content can appear.

## Use Cases

Several applications that show galleries of images and videos have requested
this feature. They wish for the contents of the gallery to have a nearly uniform
brightness.

Some applications have requested to enforce that all images be strictly SDR,
while some applications have requested that HDR images exceed SDR, but only
slightly.

Applications have also requested the ability to animate the increasing of this
maximum when hovering an image, and to animate the complete removal of this
limit when focusing on an image.

## Privacy and Security

The precise HDR headroom value of a display is a fingerprinting vector, and
should not be exposed.

## API changes

Add a new CSS property, `max-hdr-headroom`, which specifies the maximum HDR
headroom.

This property can be set to:
* A `number` which represents a specific fixed headroom. The value must be
  greater than 1.0.
* A `percentage`, which represents a percentage of the display's current HDR
  headroom.

This property has a default value of `100%` indicating that the full display HDR
headroom is to be used.

This property is animatable. Animating of this property has been requested, e.g,
when hovering over or selecting an image.

This property is inherited. It is useful to specify a headroom for a gallery's
container.

## Example

The following shows an example gallery:

```html
  <p style='max-hdr-headroom:1.5;'>
    <img src='TestImage0.avif'/>
    <img src='TestImage1.avif'/>
  </p>
```

The following shows the same gallery where the second image has removed its
limit.

```html
  <p style='max-hdr-headroom:1.5;'>
    <img src='TestImage0.avif'/>
    <img src='TestImage1.avif' style='max-hdr-headroom:100%;'/>
  </p>
```
