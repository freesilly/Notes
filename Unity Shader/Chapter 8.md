**透明度测试：** 一种根据Alpha阈值决定片元是否被渲染的技术，通过测试的片元按不透明物体处理（写入深度），未通过的则被完全舍弃，从而实现硬边缘的镂空效果，而非平滑的半透明效果。

**透明度混合：** 透明度混合通过混合颜色来实现平滑的半透明效果，其技术核心是让深度缓冲变为“只读”（即关闭深度写入但保留深度测试），这虽然能被不透明物体正确遮挡，但也导致了必须严格控制透明物体自身的渲染顺序。

### 8.2 shader的渲染顺序

![[Pasted image 20250914211838.png]]

Unity 在内部使用一系列整数索引来表示每个渲染队列，且索引号越小表示越早被渲染。

### 8.4 透明度混合

我们会把源颜色的混合因 SrcFactor 设为 SrcAlpha, 而目标颜色的混合因子 DstFacto 设为 OneMinusSrcAlpha。经过混合后新的颜色是：
$DstColor_{new}=SrcAlpha*SrcColor+(1-SrcAlpha)*DstColor_{old}$


### 8.6 unity shader的混合命令

![[Pasted image 20250914220549.png]]


**正常透明度混合：** Blend SrcAlpha OneMinusScrAlpha

**柔和相加：** Blend OneMinusDstColor One

**正片叠底：** Blend DstColor Zero

**两倍相乘：** Blend DstColor SrcColor

**变暗：** 
BlendOp Min
Blend One One

**变亮：**
BlendOp Max
Blend One One

**滤色：** Blend OneMinusDstColor One

**线性减淡：** Blend One One

![[Pasted image 20250914221049.png]]


