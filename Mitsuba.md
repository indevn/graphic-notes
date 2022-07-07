# Mitsuba

包含四个基本支持库：

- libcore 实现基本功能

- librender 包含加载和表示场景（包括其中光源、材料等）的所需抽象

- libhw 实现跨平台显示库、OpenGL wrapper、交互式预览的支持

- libbidir 用来实现双向渲染算法(BDPT, MLT, etc.)



两种设计自定义积分器的通用方法：

1. 基于采样的积分器：能够生成沿指定光线的入射辐射率估计值

   优势在于内置的并行化和网络渲染，并且可以嵌套在其他积分器当中。

2. 通用积分器：允许更通用的实现

 

## 基本实现



``` c++
#include <mitsuba/render/scene.h> // 框架位于名为mitsuba的namespace当中
MTS_NAMESPACE_BEGIN
class MyIntegrator : public SamplingIntegrator {
public:
    MTS_DECLARE_CLASS()
};
MTS_IMPLEMENT_CLASS_S(MyIntegrator, false, SamplingIntegrator) // _S: 可序列化的类
MTS_EXPORT_PLUGIN(MyIntegrator, "A contrived integrator");
MTS_NAMESPACE_END
```