---
name: image-transform-studio
description: Convert, resize, and compress an uploaded image from a natural-language request such as "make it jpg under 300 KB" or "resize to max 1080".
metadata:
  homepage: https://github.com/anxkhn/gemma-skills/tree/main/image-transform-studio
---

# Image Transform Studio

## Purpose

Use this skill when the user wants to prepare an image file by describing the
operation in natural language. Typical requests include:

- Convert an image to JPG, JPEG, PNG, or WebP.
- Resize an image to a maximum width, maximum height, maximum longest edge, exact
  dimensions, or percentage scale.
- Compress an image to a target size such as under 300 KB.
- Combine conversion, resizing, metadata stripping, and compression in one job.

This skill does not receive the image directly from the chat. It creates an
interactive webview with the requested settings already filled in. The user then
uploads the source image in the webview, runs the transform, previews the output,
and saves the generated file.

## Execution Model

Call the `run_js` tool with the following exact parameters:

- script name: index.html
- data: A JSON string following the schema in the next section.

The JavaScript runner returns a webview. Show that webview to the user. Keep the
chat response short because the webview is the primary output.

## Data Schema To Pass To `run_js`

Pass a JSON string with these fields:

```json
{
  "originalRequest": "Convert this image to JPG under 300 KB and max 1080 px",
  "outputFormat": "jpeg",
  "targetSizeKB": 300,
  "quality": 85,
  "resize": {
    "mode": "fit-within",
    "maxWidth": 1080,
    "maxHeight": 1080,
    "width": null,
    "height": null,
    "percentage": null,
    "onlyShrink": true
  },
  "stripMetadata": true,
  "autoOrient": true,
  "backgroundColor": "#ffffff",
  "filenameHint": "converted-image"
}
```

### Field Reference

`originalRequest`

- Type: String.
- Required.
- Put the user's exact image task in this field, cleaned only for whitespace.
- This is displayed in the webview and helps the user confirm that the settings
  match their intent.

`outputFormat`

- Type: String.
- Required.
- Allowed values:
  - `original`: Keep the uploaded file's detected web-safe format when possible.
  - `jpeg`: Output JPEG/JPG.
  - `png`: Output PNG.
  - `webp`: Output WebP.
- Use `jpeg` for user wording such as JPG, JPEG, photo format, smaller file for
  sharing, or when the user only says "under N KB" without naming a format.
- Use `png` for user wording such as PNG, transparent background, lossless, or
  keep transparency.
- Use `webp` only when the user explicitly asks for WebP or asks for a modern
  web image format.
- Use `original` only when the user asks only to resize and does not request
  conversion or size compression.

`targetSizeKB`

- Type: Number or null.
- Unit: Kilobytes.
- Use a number when the user says phrases like "under 300 KB", "less than 1 MB",
  "max 500kb", "compress to 2 MB", or "make it below 250 kilobytes".
- Convert MB to KB by multiplying by 1024. Example: 2 MB becomes 2048.
- If the user does not ask for a file-size target, use `null`.
- If the user asks for an unrealistically tiny value, still pass the requested
  value. The webview will try quality reduction and downscaling, then warn the
  user if the exact target could not be reached.

`quality`

- Type: Number or null.
- Range: 1 to 100.
- Meaning: Encoding quality preference for JPEG and WebP.
- Use a specific value if the user names quality, such as "quality 70".
- Use `85` when the user asks for a target file size and does not specify
  quality.
- Use `92` when outputting JPEG/WebP without a target file size and the user
  does not specify quality.
- Use `null` for PNG unless the user explicitly mentions quality.
- The webview may lower quality automatically when `targetSizeKB` is set.

`resize`

- Type: Object.
- Required.
- If the user does not ask for resizing, pass:

```json
{
  "mode": "none",
  "maxWidth": null,
  "maxHeight": null,
  "width": null,
  "height": null,
  "percentage": null,
  "onlyShrink": true
}
```

Allowed `resize.mode` values:

- `none`: Do not resize unless the file-size target cannot be reached without
  downscaling.
- `fit-within`: Preserve aspect ratio and fit inside a maximum box. Use this for
  wording like "max 1080", "longest side 1080", "resize to max 1920 px", "within
  800 by 800", "not larger than 1200".
- `width`: Preserve aspect ratio and set the width. Use this for "width 1200",
  "make it 1200 px wide".
- `height`: Preserve aspect ratio and set the height. Use this for "height 800",
  "make it 800 px tall".
- `exact`: Resize to exact width and height. This can change aspect ratio. Use
  it only when the user explicitly asks for exact dimensions like "make it
  exactly 1024x512" or "force 800 by 600".
- `percentage`: Resize by percentage. Use this for "50%", "half size", "scale to
  25 percent", or "double size".

`resize.maxWidth` and `resize.maxHeight`

- Type: Number or null.
- Used with `fit-within`.
- For "max 1080", "longest side 1080", or "resize to max 1080", set both
  `maxWidth` and `maxHeight` to `1080`.
- For "max width 1200", set `maxWidth` to `1200` and leave `maxHeight` null.
- For "max height 900", set `maxHeight` to `900` and leave `maxWidth` null.
- For "within 1600x900", set `maxWidth` to `1600` and `maxHeight` to `900`.

`resize.width` and `resize.height`

- Type: Number or null.
- Used with `width`, `height`, and `exact`.
- For `width`, set only `width`.
- For `height`, set only `height`.
- For `exact`, set both `width` and `height`.

`resize.percentage`

- Type: Number or null.
- Used with `percentage`.
- Examples:
  - "half size" means `50`.
  - "double size" means `200`.
  - "scale to 25%" means `25`.

`resize.onlyShrink`

- Type: Boolean.
- Default: `true`.
- Use `true` for max-size requests because users usually do not want small
  images enlarged.
- Use `false` only when the user explicitly asks to enlarge, upscale, double, or
  force exact dimensions.

`stripMetadata`

- Type: Boolean.
- Default: `true`.
- Use `true` unless the user explicitly asks to keep metadata, EXIF, GPS data,
  or color/profile metadata.
- Metadata stripping usually helps reduce file size and protects privacy.

`autoOrient`

- Type: Boolean.
- Default: `true`.
- Use `true` unless the user explicitly asks not to rotate or not to correct
  orientation.
- This applies camera orientation before resizing or conversion.

`backgroundColor`

- Type: String.
- Default: `#ffffff`.
- Used when converting transparent images to JPEG, because JPEG does not support
  alpha transparency.
- If the user says "white background", use `#ffffff`.
- If the user says "black background", use `#000000`.
- If the user provides a color, use a CSS hex color when possible.

`filenameHint`

- Type: String.
- Optional.
- Use a short kebab-case description such as `converted-image`,
  `compressed-photo`, `resized-image`, or `profile-picture`.
- Do not include a file extension. The webview adds the correct extension.

## Natural-Language Interpretation Rules

Follow these rules before calling `run_js`:

1. Identify the requested output format.
2. Identify any target file size and convert it to KB.
3. Identify any resize instruction.
4. Choose conservative defaults for unspecified settings.
5. Pass the normalized JSON payload. Do not ask the user for an image in chat;
   the webview handles upload.

### Format Rules

- "jpg", "jpeg", "convert to photo", "small shareable image" -> `jpeg`.
- "png", "transparent", "lossless" -> `png`.
- "webp" -> `webp`.
- "compress under N KB" with no format -> `jpeg`.
- "resize to max N" with no conversion/compression -> `original`.

### Size Rules

- "under 300 KB", "below 300kb", "max 300 kilobytes" -> `targetSizeKB: 300`.
- "under 1 MB" -> `targetSizeKB: 1024`.
- "less than 2.5 MB" -> `targetSizeKB: 2560`.
- "small enough for upload limit 500 KB" -> `targetSizeKB: 500`.

### Resize Rules

- "max 1080", "max 1080 px", "longest side 1080", "resize to max 1080" ->
  `fit-within`, `maxWidth: 1080`, `maxHeight: 1080`, `onlyShrink: true`.
- "max width 1200" -> `fit-within`, `maxWidth: 1200`, `maxHeight: null`.
- "max height 900" -> `fit-within`, `maxWidth: null`, `maxHeight: 900`.
- "within 1600x900" -> `fit-within`, `maxWidth: 1600`, `maxHeight: 900`.
- "1200 wide" or "width 1200" -> `width`, `width: 1200`.
- "800 tall" or "height 800" -> `height`, `height: 800`.
- "exactly 1024x512", "force 1024 by 512" -> `exact`, `width: 1024`,
  `height: 512`, `onlyShrink: false`.
- "half size" -> `percentage`, `percentage: 50`.
- "double size" -> `percentage`, `percentage: 200`, `onlyShrink: false`.

## Examples

### "Convert it to JPEG"

```json
{
  "originalRequest": "Convert it to JPEG",
  "outputFormat": "jpeg",
  "targetSizeKB": null,
  "quality": 92,
  "resize": {
    "mode": "none",
    "maxWidth": null,
    "maxHeight": null,
    "width": null,
    "height": null,
    "percentage": null,
    "onlyShrink": true
  },
  "stripMetadata": true,
  "autoOrient": true,
  "backgroundColor": "#ffffff",
  "filenameHint": "converted-image"
}
```

### "Make this under 300 KB"

```json
{
  "originalRequest": "Make this under 300 KB",
  "outputFormat": "jpeg",
  "targetSizeKB": 300,
  "quality": 85,
  "resize": {
    "mode": "none",
    "maxWidth": null,
    "maxHeight": null,
    "width": null,
    "height": null,
    "percentage": null,
    "onlyShrink": true
  },
  "stripMetadata": true,
  "autoOrient": true,
  "backgroundColor": "#ffffff",
  "filenameHint": "compressed-image"
}
```

### "Resize to max 1080 and make it PNG"

```json
{
  "originalRequest": "Resize to max 1080 and make it PNG",
  "outputFormat": "png",
  "targetSizeKB": null,
  "quality": null,
  "resize": {
    "mode": "fit-within",
    "maxWidth": 1080,
    "maxHeight": 1080,
    "width": null,
    "height": null,
    "percentage": null,
    "onlyShrink": true
  },
  "stripMetadata": true,
  "autoOrient": true,
  "backgroundColor": "#ffffff",
  "filenameHint": "resized-image"
}
```

### "Make it a 1200 wide WebP under 500kb"

```json
{
  "originalRequest": "Make it a 1200 wide WebP under 500kb",
  "outputFormat": "webp",
  "targetSizeKB": 500,
  "quality": 85,
  "resize": {
    "mode": "width",
    "maxWidth": null,
    "maxHeight": null,
    "width": 1200,
    "height": null,
    "percentage": null,
    "onlyShrink": true
  },
  "stripMetadata": true,
  "autoOrient": true,
  "backgroundColor": "#ffffff",
  "filenameHint": "compressed-webp"
}
```

### "Make a profile picture exactly 512x512 jpg"

```json
{
  "originalRequest": "Make a profile picture exactly 512x512 jpg",
  "outputFormat": "jpeg",
  "targetSizeKB": null,
  "quality": 92,
  "resize": {
    "mode": "exact",
    "maxWidth": null,
    "maxHeight": null,
    "width": 512,
    "height": 512,
    "percentage": null,
    "onlyShrink": false
  },
  "stripMetadata": true,
  "autoOrient": true,
  "backgroundColor": "#ffffff",
  "filenameHint": "profile-picture"
}
```

## Webview Behavior

The webview:

- Loads `@imagemagick/magick-wasm` in the browser without Node.js.
- Imports the ES module from the CDN.
- Fetches `magick.wasm` as an `ArrayBuffer`.
- Initializes ImageMagick inside the webview.
- Prefills the controls from the payload you pass.
- Lets the user choose or drop an image.
- Applies orientation, metadata stripping, resizing, conversion, and compression.
- Uses iterative quality reduction for JPEG/WebP target-size requests.
- Uses progressive downscaling when the target size cannot be reached by quality
  alone.
- Warns the user if the requested target size cannot be fully reached.
- Provides a download button for the output file.

## Limitations To Keep In Mind

- The skill is intended for still images. Animated GIFs or animated WebP files
  may be processed as a static image or may fail depending on browser and
  ImageMagick support.
- PNG is lossless. A strict "PNG under N KB" request may require downscaling
  because quality reduction does not work like JPEG/WebP.
- JPEG does not support transparency. When converting transparent PNG/WebP to
  JPEG, the selected `backgroundColor` is used.
- Very large images can be memory intensive on mobile devices. If a transform
  fails, tell the user to try a smaller source image or a smaller max dimension.

## Response Guidance

After calling the tool, say only a short sentence such as:

"I opened the image transform panel with those settings prefilled. Upload the
image there and save the output."

Do not repeat the full JSON payload in the chat unless the user asks for it.
