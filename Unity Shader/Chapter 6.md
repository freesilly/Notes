### 6.2 标准光照模型

**自发光(emissive)：** 这个部分用千描述当给定一 个方向时， 一个表面本身会向该方向发射多少辐射量。需

**高光反射(specular)：** 这个部分用千描述当光线从光源照 射到模型表面时， 该表面会在完全镜面反射方向散射多少辐射量。

**漫反射(diffuse)：** 这个部分用于描述， 当光线从光源照射 到模型表面时， 该表面会向每个方向散射多少辐射量。

**环境光(ambient)：** 它用于描述其他所有的间接光照。

#### 6.2.1 环境光

 $C_{ambient}=g_{amboent}$

#### 6.2.2 自发光
$C_{emissive}=m_{emissive}$

#### 6.2.3 漫反射
$c_{diffuse}=(c_{light}*m_{diffuse})max(0,n*I)$

其中， 是n表面法线，$I$是指向光源的单位矢量， $m_{diffuse}$是材质的漫反射颜色，$c_{light}$是光源颜色。

#### 6.2.4 高光反射

$c_{spscular}=(c_{light}*m_{specular})max(0,v*r)^{m_{gloss}}$


### 6.3 半兰伯特模型

$c_{diffuse}=(c_{light}*m_{diffuse})(\alpha(n*I)+\beta)$

大多数情况下$\alpha$和$\beta$都为0.5


### 6.4 Blinn-Phong光照模型

binn-phong模型引入了一个新的矢量$\hat{h}$

$\hat{h}=\dfrac{\hat{v}+\hat{I}}{|\hat{v}+\hat{I}|}$

Blinn 的高光反射公式：

$c_{specular}=(c_{light}*m_{specular})max(0,\hat{n}*\hat{h})^{m_{gloss}}$


