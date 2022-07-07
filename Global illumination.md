# 全局光照

全局光照：直接光照+间接光照

间接光照的计算以渲染方程为核心。在渲染方程中，如果$L_i$直接来自于光源就是直接光照，如果来自于其它物体就是间接光照。其它物体的$L_i$的计算是递归的，截止条件是与达到最大弹射次数、或光源相交、或能量消失。

对于渲染方程本身，它具有递归性、全局性并且包含积分，它<u>没有解析解</u>。

在解决渲染问题的过程中会遇到很多困难，主要是由于场景的复杂性(visibility)、材质的复杂性(BRDF)、光线传输计算的困难。



常用的渲染方程解法（渲染方法/光线追踪方法）主要分为两种：蒙特卡罗方法和有限元方法。

蒙特卡罗方法又可细分几种方法：

- 路径追踪 Path Tracing
- 双向路径追踪 Bidirectional Path Tracing

- MLT, Metropolis Light Transport
- 光子映射 Photon Mapping

有限元可细分为两种方法：

- 辐射度 Radiosity
- 基于点的全局光照 Point based global illumination



不同方法的衡量标准：

1. 速度
2. 空间存储
3. 算法易用性
4. 是否无偏
5. 是否擅长当前效果

一条路径将有多个bounce，将会面临不同材质效果的组合。



## 蒙特卡洛系方法

光线追踪在2000年以前，其基本方法已经提出。如今的方法都是当前的变种。

**Whitted Ray Tracing (1980)**

- 路径中只有纯镜面Specular
- 路径产生过程没有采样，是确定性的，不会有噪声
- 材质只有菲涅尔项，用不到微表面模型，没有roughness

**Distributed Ray Tracing (Cook 1984)**

- 发射多条光线产生模糊效果Glossy
- 发射光线数量随着bounce增加指数级增加（性能差）

**Monte Carlo Path Tracing (1986)**

- 每次bounce只反射一次（但会发射多条Path，即有多个Primary并进行后续反射）
- 通过采样产生一些路径，将相机与光源相连
- 采样次数与弹射次数一致

**Bidirectional Path Tracing (1993)**

- Path Tracing噪声过大，双向路径追踪是由light端和camera端都发射光线产生两端子路径，将所有可能的方式用多重重要性采样连接起来。

**Metropolis Light Transport (1997)**

- 场景基本都是由间接光照所照亮（这个path很困难orz只有一个主要光源且在很复杂的路径上才能找到这个光源），若使用BDPT则会产生很多的噪声。
- MLT只要找到一个困难路径，就能得到较好的效果。



在2000年以后的方法基本都是原先方法的发展：

**Monte Carlo based methods**

- PSSMLT(2002)
- ERPT(2005)
- MEMLT(2012)
- HSLT(2014)
- Improved HSLT(2015)
- Specular Manifold Sampling(2020)

**Path guiding**

- 对于有很多glossy的场景，Path Tracing的效果往往不是很好。这主要由于其Sampling方法不是很好。Path Guiding主要用于困难路径和困难场景下。
- Pipeline
  1. 在不知道入射光分布的情况下渲染一张图，渲染完成后更新（学习）分布（大概知道从哪个方向过来的是重要的）
  2. 迭代过程
  3. 最后发现结果分布已经与完美分布十分相近
- 渲染过程中进行学习，在渲染时获得分布，将其指导渲染，并对其学习和更新分布的内容。
- 如何表示入射光分布？如何对入射光分布和BRDF进行采样？
- GMM，random sampling（2014）：随机采样很快但采样质量不高
- GMM，product sampling（2016）：将卷积操作转化为Gauss上的相乘操作，比较简单地获取相乘后的分布
- SD-Tree，MIS sampling（2017）
- Stree+vMF，product sampling（2019）
- Neural importance sampling（2019）：单次采样代价高

**Next event estimation**

**Regulization**

**Gradient domain rendering**

- 采样出一条offset path获得Gradients，其中offset path与base path尽可能相近，从而Gradients则可以告诉我们哪些地方是平滑的、哪些地方是非平滑的。
- Gradient的模型应用于不同的渲染方法：GMLT/GPT/GBPT
- 不同的渲染方法的主要差别在于不同的offset path构建方法。



Monte Carlo系方法的效果很好，但它的收敛很慢。



### 路径追踪

> 每次bounce只反射一次（但会发射多条Path，即有多个Primary并进行后续反射）
>
> 通过采样产生一些路径，将相机与光源相连
>
> 采样次数与弹射次数一致

蒙特卡罗方法求解直接光照：

1. PDF采样得到$\omega_i$及该次采样的PDF$p(\omega_i)$

2. 多次采样后估计积分值$L(x,\omega_o)=\frac{1}{N}\sum\frac{g(\omega_i,\omega_o)}{p(\omega_i|\omega_o)}$

求解间接光照：将有多个Bounce

1. 路径$\overline{x}=(x_0,x_1,...,x_n) （其中x_0:相机,x_n:光源）$

   路径的贡献是$f(\overline{x})$，采样产生该路径的PDF是$p(\overline{x})$

2. 路径得到的辐射亮度是$\frac{f(\overline{x})}{p(\overline{x})}$

3. 多条路径来估算像素的辐射亮度：
   $$
   L(x,\omega_o)=\frac{1}{N}\sum\frac{f(\overline{x})}{p(\overline{x})}
   $$



我们会有两种不同的Tracing：

- Forward Tracing: 从光源到相机
- Backward Tracing: 从相机到光源（路径追踪即Backward Tracing）



**使用简单的Path Tracer构建路径**

路径$\overline{x}$中只有BRDF采样
$$
f(\overline{x})=\rho(\vec{x_1x_0},\vec{x_1x_2})(\vec{x_1x_2}\cdot n_1)
\rho(\vec{x_2x_1},\vec{x_2x_3})(\vec{x_2x_3}\cdot n_2)L_i(x_3,\vec{x_3x_2})\\
p(\overline{x})=PDF_\rho(\vec{x_1x_2})PDF_\rho(\vec{x_2x_3})
$$
Camera Ray与物体相交于$x_1$点后：

1. 在$x_1$进行BRDF采样得到出射方向，继续跟踪
2. 相交于$x_2$点后进行BRDF采样得到出射方向，继续跟踪
3. 于光源交于$x_3$点





**使用带NEE(next event estimation)的Path Tracer构建路径**

NEE要求对于每个bounce，都会连一下光源（对光源进行采样然后MIS组合）

Camera Ray与物体相交于$x_1$点后：

1. 进行光源采样获得$x_1^\prime$，连接$x_1x_1^\prime$，计算路径贡献与权重，更新radiance值

   其中权重：
   $$
   \frac{PDF_L(\vec{x_1x_1^\prime})}{PDF_L(\vec{x_1x_1^\prime})+PDF_\rho(\vec{x_1x_1^\prime})}
   $$

2. 在$x_1$进行BRDF采样得到出射方向，发射光线相交于$x_2$

3. 对$x_2$进行步骤1

4. 在$x_2$进行BRDF采样得到出射方向，与光源交于$x_3$，计算路径贡献及权重
   $$
   \frac{PDF_\rho(\vec{x_2x_3})}{PDF_L(\vec{x_2x_3})+PDF_\rho(\vec{x_2x_3})}
   $$

### BDPT

### Metropolis Light Transport

> 场景基本都是由间接光照所照亮（这个path很困难orz只有一个主要光源且在很复杂的路径上才能找到这个光源），若使用BDPT则会产生很多的噪声。
>
> MLT只要找到一个**困难路径**，就能得到较好的效果。

这是Markov Chain Monte Carlo(MCMC)的应用。马尔可夫链的思想在于，当前有一个样本，可以在其周围生成一个新的样本用来估计函数的值。

它可以在足够的时间下生成一系列以任意函数形状为PDF的样本。在周围产生更多的新样本，最后得以将最终的所有路径都找到。

<img src="Global illumination.assets/image-20220420110839621.png" alt="image-20220420110839621" style="zoom:33%;" />

Pros：

- 困难路径
- 水中的“反光”

Cons：

- 难以在理论上分析收敛速度

  不知道收敛速度就无法作为渲染动画的方法：同一帧结束后有的收敛完成有的尚未收敛最后图像会“发抖”

  

### 光子映射

Photon Mapping对于很多特效十分擅长如焦散caustics，多应用于电影中。

<img src="E:/Posts/Rendering.assets/image-20220418191609422.png" alt="image-20220418191609422" style="zoom: 25%;" />

Photon Mapping分为两步：

1. 从光源端发射光子，与场景（表面）进行交互，不断反射，直到在非Specular(Diffuse)的表面，将光子存储起来
2. 在渲染阶段Eye Pass，打出sub-paths，从eye出发打出光线，计算光线和场景的交点，根据交点周围的Density Estimation来估计光照量。

最后需要计算的也即最后的Local Density Estimation。这个估计建立在第二步的观察上，光子越集中的地方越亮、光子不集中的地方较暗。

对此我们需要做一个局部的密度估计：在任何一个着色点，找到离它最近的N个光子（可以通过加速结构快速找到），计算这些光子所占据的面积A，密度即$\frac{N}{A}$。

对称地，我们也可以取一定的面积计算其中有多少光子，但这样的方法是有偏且不一致的。

在这里，N取较小会伴随着噪声，N取较大会伴随着图像的模糊，后者是因为PM是一个有偏的方法，在Local Density Estimation中，
$$
\frac{dN}{dA}\neq \frac{\Delta N}{\Delta A}
$$
只有$dA$无限小才是正确的估计，同理，有足够多光子的情况下使用这个方法可以得到正确的结果。它不是无偏的，但它是一致Consistent的方法。

所谓有偏Biased，直观上是会让它模糊；而一致Consistent则是让它在有无限多采样点的时候不会模糊。

Photon Mapping方法是即时的、重用的，适合处理caustics和volume的渲染，常用于电影中。

但对于大多数Photon Mapping方法而言，它是有偏的；并且它只是擅长处理特定的效果。



如何解决光子的存储？

- Progressive Photon Mapping

### VCM

VCM是将双向路径追踪和光子映射相结合。BPT和PM都有其各自适应的场景，我们可以用MIS将其各自的优点结合起来。

同时，对于Volume而言，VCM也有很大的作用。

<img src="Global illumination.assets/image-20220420135612212.png" alt="image-20220420135612212" style="zoom: 25%;" />

### PBGI

PBGI：Point-based Global Illumination

将屏幕空间离散为很多点（含面积），将其组合成一定层次结构。

PBGI主要活跃在2008-2015，可适用于实时和离线场景。无噪声。

但它是个有偏的方法，参数过多，内存消耗过多。

### Many-lights

也被称为实时辐射度算法Instant Rasiosity。类似Photon Mapping，但不存成光子而存成虚拟点光源Virtual Point Light(VPL)。场景中分布了很多点光源。收集所有虚拟点光源的贡献。

IR相当于用直接光照方法就得到了间接光照的结果。

Pros

- 速度快且通常在漫反射场景有较好的效果

Cons

- 在窄的缝隙容易出问题

- 无法处理Glossy材质

综述：

<img src="E:/Posts/Rendering.assets/image-20220418201953369.png" alt="image-20220418201953369" style="zoom:25%;" />



### Radiance Cache



## 有限元系方法

### 辐射度方法

Radiosity

基于有限元的辐射度方法是一个很老的方法(1985-1990)，它将场景细分为很小的面片，每个面片可以发射和接受能量。我们递归迭代这个发射和接受的过程，直到面片的光能不再变化。

辐射度方法适用于模拟漫反射表面的间接光照和软阴影。

