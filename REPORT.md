REPORT
Best model
The model selected for deployment is moredata_aug_cos30, which adds last year's training photos, geometric augmentation, colour jitter, a cosine learning-rate schedule, and 30 training epochs.

What helped
Adding more training data produced the largest improvement, increasing external accuracy from 0.533 to 0.903. Colour jitter also improved performance on the original dataset. Data augmentation improved robustness on camera-like images. 

What did not help
Geometry augmentation alone, longer training (30 epochs), best-epoch selection, frozen features, AdamW, weight decay, label smoothing, dropout, and test-time augmentation did not produce meaningful improvements.
Is the improvement larger than noise?
Yes. The baseline achieved 0.533 external accuracy, while the moredata variants achieved about 0.90 with very small variation across seeds. The improvement is substantially larger than the observed run-to-run variability.

Why this model?
Compared with the original model, the chosen model improved validation accuracy from 0.774 to 0.871 on our photos and from 0.642 to 0.858 on last year's photos. It also greatly reduced the dominant cherry–apple confusion. 

Camera-app test
The exported ONNX model was successfully loaded by the course camera application without modifying the page. Model size, input format, and inference speed remained unchanged. 