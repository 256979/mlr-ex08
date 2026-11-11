# Leaf classifier on your phone — MLR exercise 8

**Open this page:** <https://256979.github.io/mlr-ex08/>

Point a phone camera at a leaf and the page names the species with a probability for
every class, updating live. It runs the whole network **inside the browser of the
phone** — nothing to install, no account, no server doing the thinking. Open the link
(or scan the QR code at the bottom of the page) and press **Start camera**; the page
asks for camera permission once.

Four classes: `apple`, `cherry`, `chestnut`, `maple`.

## What is where

| File | What it is |
| --- | --- |
| `index.html` | the whole app: camera, preprocessing, inference, results, QR code — plain JavaScript, no build step, no framework |
| `leaf_model.onnx` | the trained network: MobileNetV3-Small, pretrained on ImageNet, last layer replaced by 4 outputs and fine-tuned on our group's leaf photographs (exported in the notebook, exercise 8 task 2e) |
| `model_info.json` | how the model must be fed: `classes`, `input_size`, `mean`, `std`. The page reads it at start-up, so the preprocessing and the class names come from the same file that travels with the weights |
| `check_image.jpg`, `check_expected.json` | one validation photo and the probabilities the notebook computes for it — used to prove the page and the notebook agree |

## The preprocessing, exactly

The page and the notebook must agree pixel for pixel, so the chain is written down:

1. take the **centre square** of the frame (side = `min(width, height)`);
2. resize it to **96 × 96**;
3. divide the RGB pixels by **255**;
4. standardise each channel with `mean = [0.485, 0.456, 0.406]`, `std = [0.229, 0.224, 0.225]`;
5. feed the tensor as **CHW** (three planes: red, green, blue), shape `1 × 3 × 96 × 96`;
6. the network returns raw scores (*logits*, the ONNX graph has no softmax node), so the page applies **softmax** itself.

The preview is `object-fit: cover` and square, so what you see in the frame really is
the square that gets classified.

## Does the page give the notebook's numbers?

`check_image.jpg` uploaded to the live page, against `check_expected.json` from the
notebook (measured in Chromium; ONNX Runtime Web 1.23.2, single WASM thread):

| class | page | notebook |
| --- | --- | --- |
| apple | 0.9833539 | 0.9833528 |
| cherry | 0.0166264 | 0.0166277 |
| chestnut | 0.0000043 | 0.0000043 |
| maple | 0.0000154 | 0.0000154 |

Largest difference `1.3e-06` — the same probabilities to three decimals. The remaining
difference is the browser's canvas resampling and float32 rounding in WebAssembly, not a
different preprocessing.

## Does anything leave the phone?

No. The photo is read by the camera into a `<video>` element and sampled on a
`<canvas>`; the pixels go to ONNX Runtime Web in the same tab. The page makes exactly
two kinds of requests:

* its own files from GitHub Pages (`index.html`, `leaf_model.onnx`, `model_info.json`),
* **`https://cdn.jsdelivr.net/npm/onnxruntime-web@…/dist/`** — the ONNX Runtime Web
  JavaScript and its WASM binary, from a CDN, downloaded once and then cached.

Nothing is uploaded: no analytics, no telemetry, no image POST. You can confirm it in
the browser's Network panel — the only external origin is `cdn.jsdelivr.net`. The QR
code is drawn by an encoder written into `index.html` itself, so even that does not ask
a QR service for a picture.

## Trying another model or runtime

Small query parameters, mostly for checking improvements against the shipped model:

* `?model=leaf_model_v2.onnx&info=model_info_v2.json` — another exported network and its description,
* `?ort=1.31.1` — another ONNX Runtime Web build,
* `?interval=400` — milliseconds between inferences (default 200).

## Notes

* GitHub Pages sends no COOP/COEP headers, so `SharedArrayBuffer` (and therefore the
  multi-threaded WASM build) is not available; the page pins a single WASM thread
  instead of letting the runtime probe for one.
* Safari on iPhone is the reason the camera stream uses `playsinline` and why the photo
  upload is there as a fallback: a still image exercises the same preprocessing and
  inference path.
