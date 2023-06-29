---
title: torchvision.transform和Albumentation和imgaug三个数据增强库对batch操作的支持
date: 2023-06-07 09:30:00 +0800
categories: [CV]
tags: [data-augment]   # TAG names should always be lowercase
toc: true
comments: true
math: true
mermaid: true  # 图表生成工具
---

_



# torchvision.transform和Albumentation和imgaug三个数据增强库对batch操作的支持

这三个库（`torchvision.transforms`, `Albumentation`, 和 `imgaug`）均支持数据增强操作，但它们的要求和用法各有不同。下面介绍这三个库的输入图像形状要求和批量处理的能力，并给出相应的示例。

1. **torchvision.transforms**

`torchvision.transforms` 通常用于处理PIL图像或形状为 (C, H, W) 的张量。它不直接支持批量处理，但您可以在数据加载器（如 `torch.utils.data.DataLoader` ）中对每个样本应用相应的变换。

示例：

```python
from torchvision import transforms
from torchvision.datasets import MNIST
from torch.utils.data import DataLoader

transform = transforms.Compose([
    transforms.RandomHorizontalFlip(),
    transforms.RandomRotation(10),
    transforms.ToTensor()
])

train_dataset = MNIST(root='./data', train=True, transform=transform, download=True)
train_loader = DataLoader(dataset=train_dataset, batch_size=32, shuffle=True)

for images, labels in train_loader:
    # 对于每个 batch，images 的形状为 (B, C, H, W)
    pass
```

2. **Albumentation**

`Albumentation` 库接受 NumPy 数组作为输入，图像的形状应为 (H, W, C)。它也不直接支持批量处理，但您可以在数据加载器中对每个样本应用相应的变换。

示例：

```python
import cv2
import albumentations as A
from albumentations.pytorch import ToTensorV2

transform = A.Compose([
    A.HorizontalFlip(p=0.5),
    A.Rotate(limit=10),
    ToTensorV2()
])

class MyDataset(torch.utils.data.Dataset):
    def __init__(self, images, labels, transform=None):
        self.images = images
        self.labels = labels
        self.transform = transform

    def __len__(self):
        return len(self.images)

    def __getitem__(self, idx):
        image = self.images[idx]
        label = self.labels[idx]

        if self.transform:
            image = self.transform(image=image)["image"]

        return image, label

# 用您自己的图片和标签实例化数据集
train_dataset = MyDataset(images, labels, transform=transform)
train_loader = DataLoader(dataset=train_dataset, batch_size=32, shuffle=True)
```

3. **imgaug**

`imgaug` 库同样接受形状为 (H, W, C) 的 NumPy 数组。与前两个库不同，`imgaug` 支持批量处理，可以对整个批次的图像应用变换。

示例：

```python
import cv2
import imgaug as ia
import imgaug.augmenters as iaa

seq = iaa.Sequential([
    iaa.Fliplr(0.5),
    iaa.Affine(rotate=(-10, 10)),
    iaa.ToFloat32(),
])

class MyDataset(torch.utils.data.Dataset):
    def __init__(self, images, labels, transform=None):
        self.images = images
        self.labels = labels
        self.transform = transform

    def __len__(self):
        return len(self.images)

    def __getitem__(self, idx):
        image = self.images[idx]
        label = self.labels[idx]

        if self.transform:
            image = self.transform.augment_image(image)

        return image, label

# 用您自己的图片和标签实例化数据集
train_dataset = MyDataset(images, labels, transform=seq)
train_loader = DataLoader(dataset=train_dataset, batch_size=32, shuffle=True)
```

综上所述，这三个库都可以处理三维的 RGB 图像。不过，`torchvision.transforms` 和 `Albumentation` 默认不支持批量处理，需要在数据加载器中分别对每个样本应用变换。而 `imgaug` 支持批量处理，可以直接对整个批次的图像应用变换。
