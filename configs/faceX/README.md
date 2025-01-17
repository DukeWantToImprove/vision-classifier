# Representation Learning Configuration instructions

* model
  * [Model Config](backbone_conf.yaml)
  * [Head Config](head_conf.yaml)  
    
```markdown
model:
  task: cbir
  image_size: &imgsz 224
  backbone:
    swintransformer: # tiny small base
      model_size: base
      image_size: *imgsz
      feat_dim: &featd 128
  head:
    arcface:
      feat_dim: *featd
      num_class: 58671
      margin_arc: 0.35
      margin_am: 0.0
      scale: 32
```

* data
```markdown
data:
  root: <your-data-path>
  nw: 64 # if not multi-nw, set to 0
  train:
    bs: 80 # per gpu
    base_aug: null
    class_aug: null
    augment: # refer to utils/augment.py
       - random_choice:
          transforms: 
            - random_color_jitter:
                brightness: 0.1
                contrast: 0.1
                saturation: 0.1
                hue: 0.1
            - random_cutout:
                n_holes: 3
                length: 12
                prob: 0.1
                color: [0, 255]
            - random_gaussianblur:
                kernel_size: 5
            - random_rotate:
                degrees: 10
            - random_adjustsharpness:
                p: 0.5
       - random_horizonflip: 
           p: 0.5
       - random_choice:
          transforms:
            - resize_and_padding: 
                size: *imgsz
                training: True
            - random_crop_and_resize:
                size: *imgsz
                scale: [0.7, 1]
          p: [0.9, 0.1]
       - to_tensor: no_params
       - normalize: 
            mean: [0.485, 0.456, 0.406]
            std: [0.229, 0.224, 0.225]
    aug_epoch: 24 
  val:
    bs: 128
    metrics:
      metrics: [mrr, recall, precision, auc, ndcg]
      cutoffs: [1, 3, 5] 
    augment:
        - resize_and_padding:
            size: *imgsz
            training: False
        - to_tensor: no_params
        - normalize: 
            mean: [0.485, 0.456, 0.406]
            std: [0.229, 0.224, 0.225]
```
* hyp
```markdown
hyp:
  epochs: 25
  lr0: 0.006 
  lrf_ratio: null # decay to lrf_ratio * lr0, if None, 0.1
  momentum: 0.937
  weight_decay: 0.0005
  warmup_momentum: 0.8
  warm_ep: 1
  loss:
    ce: True
  label_smooth: 0.0
  optimizer: 
    - sgd # sgd, adam or sam
    - True # Different layers in the model set different learning rates, in built/layer_optimizer
  scheduler: cosine_with_warm # linear or cosine
```