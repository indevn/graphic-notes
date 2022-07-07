# 基于物理的布料模拟 Physics-Based Cloth Simulation

[TOC]

## 质点弹簧系统

A Mass-Spring System的积分可以分为基于显式积分和基于隐式积分。



一个理想的弹簧满足胡克定律Hooke’s law：弹簧总是尝试去恢复到其原长。

对于一维情况：

<img src="Cloth Simulation.assets/image-20220422150603481.png" alt="image-20220422150603481" style="zoom:25%;" />
$$
E(x)=\frac{1}{2}k(x-L)^2\\
f(x)=-\frac{dE}{dx}=-k(x-L)
$$

对于二维情况：

<img src="Cloth Simulation.assets/image-20220422150629891.png" alt="image-20220422150629891" style="zoom:25%;" />
$$
E(x)=\frac{1}{2}k(||x_i-x_j||-L)^2\\
f_i(x)=-\nabla_iE=-k(||x_i-x_j||-L)\frac{x_i-x_j}{||x_i-x_j||}\\
f_j(x)=-\nabla_jE=-k(||x_j-x_i||-L)\frac{x_j-x_i}{||x_j-x_i||}
$$
弹簧的







This plugin implements the Irawan & Marschner BRDF, a realistic model for rendering woven materials. This spatially-varying reflectance model uses an explicit description of the underlying weave pattern to create fine-scale texture and realistic reflections across a wide range of different weave types. To use the model, you must provide a special weave pattern file—for an example of what these look like, see the examples scenes available on the Mitsuba website. A detailed explanation of the model is beyond the scope of this manual. For reference, it is described in detail in the PhD thesis of Piti Irawan (“The Appearance of Woven Cloth” [17]). The code in Mitsuba a modified port of a previous Java implementation by Piti, which has been extended with a simple domain-specific weave pattern description language.

老师我对于那个布料的问题还是有点疑惑。

前段时间有点感冒+期中考试，今天试了下mitsuba，看到mitsuba已经自带了一个对布料材质的论文复现，我这里也照着文档实现了一个场景。
