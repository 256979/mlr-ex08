# REPORT

## Best model

The model selected for deployment is **`moredata_aug_cos30`**. It uses the original pretrained MobileNetV3-Small architecture and adds last year's 700 training photographs, geometric and colour augmentation, a cosine learning-rate schedule, and 30 training epochs.

The final model was exported as `leaf_model_v2.onnx` (6.36 MB). It uses the same input format, class order, preprocessing and architecture as the original model, so it can be used in the course camera application without changing the page.

## Results table

The baseline is the original Task 2 pipeline: pretrained MobileNetV3-Small, Adam with a learning rate of 0.001, CrossEntropyLoss, batch size 32 and 15 epochs, without augmentation.

Each experiment changes one part of the baseline unless the variant explicitly combines changes. Important experiments were repeated with three random seeds (0, 1 and 2).

| Variant | Training images | Epochs | Our validation | Our camera-like | External validation | External camera-like |
|---|---:|---:|---:|---:|---:|---:|
| `moredata` | 944 | 15 | 0.849 | 0.694 | **0.903** | 0.815 |
| `big_no_best` | 944 | 30 | 0.812 | 0.706 | 0.900 | 0.873 |
| `best_combo` | 944 | 30 | 0.887 | 0.723 | 0.896 | **0.882** |
| `moredata_aug_cos30` | 944 | 30 | 0.849 | 0.717 | 0.886 | 0.866 |
| `moredata_jitter` | 944 | 15 | 0.790 | 0.691 | 0.886 | 0.814 |
| `jitter` | 244 | 15 | 0.753 | 0.696 | 0.633 | 0.623 |
| `aug` | 244 | 15 | 0.801 | 0.766 | 0.583 | 0.616 |
| `aug_cosine_ep30` | 244 | 30 | 0.849 | **0.803** | 0.611 | 0.633 |
| `baseline` | 244 | 15 | 0.753 | 0.673 | 0.533 | 0.511 |
| `geom` | 244 | 15 | 0.758 | 0.688 | 0.486 | 0.510 |
| `ep30` | 244 | 30 | 0.758 | 0.624 | 0.478 | 0.461 |
| `frozen` | 244 | 15 | 0.790 | 0.635 | 0.464 | 0.527 |
| `leafnet` | 244 | 15 | 0.688 | 0.402 | 0.389 | 0.267 |
| `leafnet_aug` | 244 | 15 | 0.688 | 0.583 | 0.408 | 0.407 |
| `leafnet_bn_aug` | 244 | 15 | 0.742 | 0.575 | 0.489 | 0.433 |
| `leafnet_aug_moredata` | 944 | 15 | 0.656 | 0.504 | 0.797 | 0.626 |

The results are averages across seeds where multiple runs were performed. The camera-like validation sets were generated once using fixed random transformations, so every model was evaluated on the same images.

## What each change brought

**Adding more training data — the biggest improvement.**

Adding 700 photographs from last year's training dataset increased external validation accuracy from 0.533 to 0.903. This was the largest improvement in the experiments. It also reduced variation between random seeds, making the training results more consistent.

**Colour jitter — a useful improvement.**

Changing brightness, contrast, saturation and hue increased external accuracy from 0.533 to 0.633 using the original 244 training images. It was the most useful individual augmentation on the smaller dataset.

**Geometric augmentation — improved camera robustness.**

Random cropping, flipping and rotation alone did not improve external accuracy. However, combining geometric and colour augmentation improved performance on camera-like images. With 30 epochs and a cosine learning-rate schedule, `aug_cosine_ep30` achieved 0.803 on our camera-like validation set, the highest result in that column.

**Combining more data and augmentation — the best practical result.**

The `moredata_aug_cos30` model achieved 0.866 on external camera-like images, compared with 0.815 for `moredata`. This suggests that additional training photographs improve general classification, while augmentation helps the model handle different camera angles, framing and lighting.

**Changing the network architecture — generally worse.**

The notebook's smaller CNN, called LeafNet, had fewer parameters than MobileNetV3-Small but achieved lower accuracy. Batch normalisation improved its results somewhat, but it still did not outperform the MobileNet baseline. This indicates that keeping the pretrained MobileNet architecture is the better choice.

## Is the improvement larger than the noise?

Yes. The baseline achieved an average external accuracy of 0.533, with results ranging from 0.408 to 0.658 across three seeds.

The `moredata` variant achieved 0.903 ± 0.004. Even its lowest result was substantially higher than the best baseline result.

The paired comparison on the same 120 external validation photographs also supports this improvement. The `moredata` model corrected 148 errors and introduced 15 new errors compared with the baseline

The improvement from adding more data is therefore much larger than normal variation between training runs.

The augmentation results on the smaller dataset are less conclusive. Some improvements are visible, particularly with colour jitter, but several results overlap with the baseline's variation. The camera-like results must also be interpreted carefully because the validation images were generated using transformations similar to those used during training.

## What did not help

Several experiments failed to produce meaningful improvements:

- **Geometric augmentation alone:** did not improve external accuracy.
- **Training for 30 epochs without additional data:** reduced external accuracy, suggesting overfitting.
- **Best-epoch selection:** improved accuracy on our validation photos but reduced external accuracy.
- **Freezing the pretrained backbone:** reduced accuracy, although training was faster.
- **AdamW and weight decay:** did not provide a measurable improvement.
- **Label smoothing and dropout:** did not solve the classification errors.
- **Cosine scheduling alone:** produced no convincing improvement.
- **Multi-view test-time augmentation:** gave only small improvements on some images and worse results on camera-like images, while requiring several additional inference passes.
- **Training a smaller CNN from scratch:** performed worse than fine-tuning pretrained MobileNetV3-Small.

Overall, the experiments show that the main limitation was insufficient and unrepresentative training data rather than the choice of optimiser or other hyperparameters.

## Which model would I put on the phone?

I would deploy `moredata_aug_cos30`, using seed 0 selected before comparing the test results.

Its performance on the exported ONNX model was:

| Validation set | Original model | New model |
|---|---:|---:|
| Our 62 validation photos | 0.774 | **0.871** |
| Last year's 120 validation photos | 0.642 | **0.858** |

On last year's photographs, the new model correctly classified 103 out of 120 images, compared with 77 out of 120 for the original model.

The largest improvement was in cherry classification: the original model correctly classified only 4 out of 30 cherry photographs, while the new model correctly classified 23 out of 30. The chestnut-to-maple confusion was also eliminated, with all 30 chestnut photographs classified correctly.

The remaining apple–cherry confusion is partly caused by differences and similarities in the photographs themselves. Some leaves labelled as apple and cherry look very similar, making this distinction difficult.

The new model was exported using the required ONNX export procedure from Task 2e. PyTorch and ONNX Runtime produced matching outputs, with a maximum logit difference of approximately 5.7 × 10⁻⁶.

The exported file is approximately 6.36 MB and has the same architecture and input requirements as the original model. Inference takes approximately 16 ms in the browser, so the improvement does not require additional model size or a slower camera application.

The model was successfully tested with the course camera application without requiring changes to the page.

**Final decision:** `moredata_aug_cos30` is the best practical choice because it combines the benefit of additional training data with improved robustness to real camera images while preserving the original model architecture, size and inference speed.