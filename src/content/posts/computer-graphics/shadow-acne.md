---
title: 图形学：Shadow Acne
published: 2025-12-26
description: ''
image: './images/sphere_simple_emph.jpg'
tags: [计算机图形学, 路径追踪, 阴影失真, Shadow Acne]
category: 'Graphics'
draft: false 
lang: ''
---

最近结识了一位对图形学颇感兴趣的 Friend，出于对图形学的好奇，我也开始入门图形学：参考书籍 [Ray Tracing in One Weekend Series](https://raytracing.github.io/) 使用 Rust 实现简易的光线追踪渲染器。

## 问题重述

在 First Week 的收尾工作时，笔者创造出了 Shadow Acne 的 Bug：在给以巨大球体为基础的地面进行渲染后，得到的地面出现了类似摩尔纹的环状图案，如封面所示。不过原本的环形纹路并不是很明显，封面中的纹路是笔者在后来理解了问题产生的原理之后，对其进行了放大处理。


> [!TIP]
> 原先对 Shadow Acne 的认识不是很深刻，可以说只是对其有个粗浅的概念理解。不过通过在实践中解决 Bug，让笔者对 Shadow Acne 的原理有了更加深刻的理解。


## 问题分析

:::note[为什么在 First Week 的收尾工作的时候才发现这个问题？]

在之前的场景渲染中，为了渲染结果的美观，笔者使用的是带有颜色的地面材质。而这种材质不得不说不细看还真看不出来，Shadow Acne 的 Bug 在低对比度的颜色中难以察觉。

:::

![color sphere](./images/sphere_simple_color.jpg)

该项目中 Shadow Acne 的出现源于程序的计算精度设置。由于球体较大，每位移一处，变化的坐标值较小，要求的存储精度较高。而笔者在实现路径追踪时，使用的是32位浮点数类型来存储坐标值，因为这是对内存和精度的权衡。虽然大部分场景已经能够满足，但在巨大球体的地面上，32位浮点数的精度已经不足以满足需求。

在精度不足的情况下，如果我们还将求交时限定区间的下界 `t_min` 设置为 0 或者太接近于 0 的数值，就会导致程序在采样反射光线时，直接判定反射光线与球体相交，渲染采样就会额外增加一层递归，衰减系数变为原来的一倍，从而使颜色变深。

下面笔者给大家进行分析，以下是光线求交的主要代码片段：

```rust title="renderer.rs"
pub fn trace_ray(r: &Ray, s: &Scene, depth: u32, rec: &mut HitRecord) -> Color {
    // existing code...
    if s.get_closest_intersect(r, 1e-5, f32::INFINITY, rec) {
        let mut attenuation = Color::default();
        let mut scatter = Ray::default();
        if rec.material.as_ref().unwrap()
            .scatter(r, rec, &mut attenuation, &mut scatter)
        {
            return attenuation * Self::trace_ray(&scatter, s, depth - 1, rec);
        }
        return Color::black();
    }
    // existing code...
}
```

在光线求交的范围限定下界 `t_min` 处，我们使用了 `1e-5` 作为最小值，也就是说，只有当光线与物体的交点距离光线起点大于 `1e-5` 时，才会被判定为相交。

当程序第一次采样反射光线时，程序正常求交，进入这一行代码进行反射光线的采样：

```rust title="renderer.rs >> fn trace_ray()"
return attenuation * Self::trace_ray(&scatter, s, depth - 1, rec);
```

当程序第二次采样反射光线时，由于反射光线的起点非常接近第一次采样的交点（因为球体很大，坐标值变化很小），导致 `get_closest_intersect` 函数判定反射光线与球体相交，多进行了一次光线采样。

最后，我们得到的光线结果就是 `attenuation * attenuation * Color`，而我们原本预期的结果应该是 `attenuation * Color`，也就是只衰减一次。既然多了一次衰减，颜色自然就变深了。

## 解决方案

通过上面的分析，我们可以得出两种解决方案：

1. 提高存储坐标值的精度，使用 `f64` 类型来存储坐标值。不过这种方法会增加内存开销，增加渲染时间。
2. 在求交时，将 `t_min` 从 `1e-5` 改为一个较大的数值，例如 `0.001`，以避免反射光线与球体的误判交点。

笔者选择了第二种方法进行解决。代码经过调整后，封面相同内容的渲染结果如下图所示：

![resolved sphere](./images/sphere_simple_resolution.jpg)
