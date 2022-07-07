# Fourier Transform



# 频率变化

高频即频率变化快，低频即频率变化慢。低频意味着函数更加smooth，而

时域上的卷积可以对应于频域的乘积。

在GAMES101中，我们可以将卷积理解为滤波。而在这里，我们可以有更通用的：两个函数相乘后做积分，就可以视为一个卷积（滤波）操作。积分后的结果的频率是相乘函数中最低的频率。



**基函数**

对于某一个函数，它可以是一系列函数的线性组合，而这一系列函数即基函数。



Spherical Harmonics是一系列的二维基函数，它们定义在球面上。可以理解为球面上的某一个点就描述为一个方向[x,y]。它们很像一维情况下的傅里叶函数。在Rendering中会遇到很多球面上的函数，为了对其进行Fourier变换，展开成不同的频率进行表示。而Spherical Harmonics本身定义于球面上，也就很适用于直接分析球面上的一些性质。

对于每一个SH基函数，我们利用Legendre多项式进行描述。这是数学上的描述。

给定一个二维函数，如果用SH的线性组合对其进行展开，对于每一个SH的系数，有
$$
c_i=\int_{\Omega}f(\omega)B_i(\omega)d\omega
$$
求任意函数展开成SH的系数的过程，即”投影“。所有$B_i(\omega)$都是正交的，求其系数即求其在某个维度上的长度。这样的投影操作可以表现为点乘，而在点乘后我们还要做积分即Product Intergral。一般意义上的投影只是“点乘”，而在这里我们在投影后还要做积分，是因为

SH基函数从低到高，是可以把不同的频率表示出来的。我们如果需要用SH基函数对原函数进行拟合，我们用的Basis个数越多，越能引入更多高频的信息。

<img src="Spherical Harmonics.assets/image-20220511133408680.png" alt="image-20220511133408680" style="zoom:33%;" />

同时，SH也有很多很棒的性质：

- Orthonormal

- Simple Projection/Reconstruction

- Simple Rotation

  旋转任何一个SH Basis，它都可以由同阶SH的线性组合得到。对一个光照进行旋转，我们就可以立即得到另一个SH的新的线性组合。

- Simple Convolution

- Few Basis Functions: Low freqs



引入基函数，我们可以尝试解决，在环境光照下对Diffuse物体的Shading问题。

Diffuse BRDF是很光滑的函数，可以视为一个低通滤波器，
$$
E_{lm}=A_lL_{lm}
$$
其中$L_{lm}$是一个球面函数，Diffuse BRDF是一个光滑的球面函数。对其做逐点相乘并进行积分，即Product Intergral的操作。我们如果需要SH描述Diffuse BRDF，由于其低频特性，我们只需要三阶即可将其恢复得比较好。同样，对于光照项，我们也只需不超过三阶的SH就能对原球面函数拟合出较好的效果。

<img src="Spherical Harmonics.assets/image-20220511133349540.png" alt="image-20220511133349540" style="zoom:33%;" />

但此时仍旧没有引入阴影。我们目前只考虑了Shading。



PRT: Precomputed Radiance Transfer

Rendering under environment lighting
$$
L(o)=\int_{\Omega}L(i)V(i)\rho(i,o)max(0,n\cdot i)di
$$




环境光照下如何取得阴影？

假设场景中只有光照发生变化，其他地方都视作不变。则可将渲染方程拆为两部分：Lighting和Light Transport：

<img src="Spherical Harmonics.assets/image-20220511143827795.png" alt="image-20220511143827795" style="zoom:33%;" />
$$
L(o)=\int_{\Omega}L(i)V(i)\rho(i,o)max(0,n\cdot i)di\\
L(o)\approx \rho\sum l_i\int_{\Omega}B_i(i)V(i)max(0,n\cdot i)di
$$
其中$\int_{\Omega}B_i(i)V(i)max(0,n\cdot i)di$可以视为投影，只是一个数，可以通过预计算获得。在实际运算过程中，我们只需要求点乘：
$$
L(o)\approx \rho\sum l_iT_i
$$
但它只适用于静态场景，如果在运行时场景发生变化，预计算内容将不再适用。

在这里，预计算的基函数，首要选择即Sperical Harmonics。