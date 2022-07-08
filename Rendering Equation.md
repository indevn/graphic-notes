[TOC]



# Rendering Equation

## 辐射度量学

Difference among three def:

- Irradiance: power per projected unit area

- Intensity: power per solid angle 

- Radiance: Irradiance per solid angle

  Radiance: Intensity per projected unit area

Difference between Irradiance and Radiance:

- Irradiance: total power received by area $dA$

- Radiance: power received by area $dA$ from “direction” $d\omega$



## 双向反射分布函数：对不同材质的描述

双向反射分布函数（BRDF）描述光线和物体材质表面的交互作用，它是对不同材质的描述。它是一个四维函数，在$4\pi$球面度上定义，在表面上的每一个点都有定义。它的四个维度分别是:

- $\Psi$ - 入射方向 $(x_1,y_1)$
- $\Theta$ - 出射方向 $(x_2,y_2)$

BRDF定义为一个比值，是点x处在出射方向$\Psi$上反射的相对辐射亮度与通过不同立体角入射的相对辐照度之比，可表示为

$$
f_r(x,\Psi \to \Theta)=\frac{dL(x\to \Theta)}{dE(x\gets\Theta)}=\frac{dL(x\to \Theta)}{L(x\gets\Theta)cos(N_x,\Psi)d\omega_{\Psi}}
$$
其中，$N_x$是法线向量，$cos(N_x,\Psi)$是法线向量和入射方向向量的余弦，也即入射方向向量在表面上（假设表面是平面）的投影。

### BRDF的性质

1. 互反律
   $$
   f_r(x,\Psi \to \Theta)=f_r(x,\Theta \to \Psi)
   $$
   亦可直接将BRDF表示为
   $$
   f_r(x,\Psi \leftrightarrow  \Theta)
   $$
   
2. 入射与出射亮度的关系

   在不透明的非发射表面点周围的半球上有一些辐照度分布，总反射亮度可表示为
   $$
   dL(x\to\Theta)=f_r(x,\Psi\to\Theta)dE(x\gets\Psi)\\
   L(x\to\Theta)=\int_{\Omega x}f_r(x,\Psi\to\Theta)dE(x\gets\Psi)\\
   L(x\to\Theta)=f_r(x,\Psi\to\Theta)L(x\gets\Theta)cos(N_x,\Psi)d\omega_{\Psi}
   $$

3. 能量守恒

   对于半球上任何入射辐射亮度$L(x\gets \Psi)$的分布，每单位表面积的总入射功率是半球的总辐照度：
   $$
   E=\int_{\Omega x}L(x\gets\Psi)cos(N_x,\Psi)d\omega_\Psi
   $$
   同时，由能量守恒定律，*在某点于某方向*的出射辐射亮度，等于该点*在某点于某方向*上的自发光和*在某点于某方向*上的反射光的辐射亮度，即：
   $$
   L(x\to \Theta) = L_e(x\to\Theta)+L_r(x\to\Theta)
   $$
   可展开为渲染方程：
   $$
   L_o(x,\vec\omega_o)=L_e(x,\omega_o)+\int_{H^2}f_r(x,\vec\omega_o,\vec\omega_i)L_i(x,\vec\omega_i)cos\theta_id\vec\omega_i\\
   $$
   *其中$f_r$是散射函数，即 BRDF方程。余弦项是入射光和法线之间的夹角。*



对于经验模型的BRDF，我们希望它是一个良好的、合理的BRDF，也即需要它满足能量守恒和互反律。

### BRDF示例：漫反射、镜面反射与折射

1. 漫反射
   $$
   f_r(x,\Psi\leftrightarrow\Theta)=\frac{\rho_d}{\pi}
   $$
   其中$\rho_d$反射率是反射能量与入射能量之比。对于基于物理的材质，$\rho _d\in(0,1]$。

2. 镜面

   - 只考虑折射

     按反射定律找到出射方向：假设入射方向$\Psi$，表面法线$N$，反射方向为$R=2(N\cdot\Psi)N-\Psi$

   - 考虑散射

     **方向：**Snell's law

     按Snell's law计算镜面折射方向，即根据$\eta_1sin\theta_1=\eta_2sin\theta_2$，则有透射光线
     $$
     T=-\frac{\eta_1}{\eta_2}\Psi+N(\frac{\eta_1}{\eta_2}cos\theta_1-\sqrt{1-(\frac{\eta_1}{\eta_2})^2(1-cos^2\theta_1)}\\
     =-\frac{\eta_1}{\eta_2}\Psi+N(\frac{\eta_1}{\eta_2}cos\theta_1-\sqrt{(1-(\frac{\eta_1}{\eta_2})^2(1-N\cdot\Psi)^2}
     $$
     同时须考虑到内全反射（考虑临界角）。
   
     **能量：**Fresnel Equations

     上述等式考虑到了反射和折射方向（角度），菲涅尔方程则考虑到了能量。

   - 透明表面的互反律

     使用BSDF描述透明面时，需要注意透明表面可能不具备互反律。




## 渲染方程：描述光能的整体平衡分布

渲染方程用于描述场景中光能的平衡分布，对于任意表面点和方向，渲染方程给出最后的出射辐射亮度$L(x\to\Theta)$。

### 基本形式

**半球形公式**是最常用的渲染方程形式。

按照BRDF定义，
$$
f_r(x,\Psi \to \Theta)=\frac{dL_r(x\to \Theta)}{dE(x\gets\Theta)}
$$
其中，
$$
L_r(x\to\Theta)=\int_{\Omega_x}f_r(x,\Psi\to\Theta)L(x\gets\Psi)cos(N_x,\Psi)d\omega_{\Psi}
$$
则得到第二种弗雷德霍姆方程形式的渲染方程：
$$
L(x\to\Theta)=L_e(x\to\Theta)+\int_{\Omega_x}f_r(x,\Psi\to\Theta)L(x\gets\Psi)cos(N_x,\Psi)d\omega_{\Psi}
$$



**区域公式**则是将定义域定义在物体表面区域。我们假设光线从点$x$投射到点$y$，我们可以将半球形定义域投射到表面区域得到公式：
$$
L(x\to\Theta)=L_e(x\to\Theta)+\int_{\Omega_x}f_r(x,\Psi\to\Theta)L(x\gets\Psi)V(x,y)G(x,y)dA_y
$$
其中可视性函数V(x,y)指定了两点间可见性，几何项G则封装了涉及相对几何的内容：
$$
G(x,y)=\frac{cos(N_x,\Psi)cos(N_y,-\Psi)}{r_{xy}^2}
$$


### 直接光照和间接光照

渲染方程告诉我们，最后的出射光照由反射光照和物体自发光emission组成。而反射光照$L_r$，则由直接光照和间接光照构成。

直接光照是来自外界物体的emission，后者从表面沿着连线方向到当前的可见点，可用区域公式为：
$$
L_{d}=\int_A f_r(x,\vec{xy}\to\Theta)L_e(y\to\vec{yx})V(x,y)G(x,y)dA_y
$$
间接光照则更多使用半球形公式，因为它不是单纯由某个特定区域反射而来，而是光线经过多次弹射Bounce得来，我们只考虑半球：
$$
L_{id}=\int_{\Omega_x} f_r(x,\Psi\to\Theta)L_i(x\gets\Psi)cos(N_x,\Psi)d\omega_\Psi
$$
同时，我们可以知道，间接照明是点$x:r(x,\Psi)$处从半球上所有可见点反射的辐射亮度：
$$
L_i(x\gets\Psi)=L_r(r(x,\Psi)\to-\Psi)
$$

