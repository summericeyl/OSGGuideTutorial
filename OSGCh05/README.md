### 05.01 渲染状态示例
* osg::PolygonMode
* osg::AnimationPathCallback: 派生自 osg::NodeCallback
    - 可以创建一个旋转动画(构造函数设置旋转的中心, 轴和旋转速度)
* osg::ClipNode
    - createClipBox
* osg::MatrixTransform 设置动画(使用更新回调)
    - setUpdateCallback

### 05.02 二维纹理映射
* 创建纹理对象
* osg::Texture2D
    - setDataVariance
    - setImage

### 05.03 多重纹理映射
* 演示多重纹理, 第二个纹理使用了纹理坐标球面生成的方式, 并和第一重纹理进行混合
* osg::TexGen: 设置纹理坐标生成
    - setMode --- 设置生成模式
* osg::TexEnv: 映射模式
    - setMode --- 设置模式为混合

### 05.04 Mipmap 纹理映射示例
* 创建一个带有 Mipmap 的图像数据, 而后将其作为纹理
* osg::Image::MipmapDataType: 创建一个 MipmapDataType 列表, 用来存放各层图片数据的偏移地址
* osg::Image::setMipmapLevels 可以放置上面的变量, 而后设置 Mipmap 即可

### 05.05 TextureRectangle 纹理映射
* 纹理坐标不需要被单元化, 纹理的大小不需要是2的幂次方, 不支持 REPEAT 环绕模式, 过滤不支持 Mipmap, 不支持纹理边框
* osg::TexMat 设置纹理矩阵, 在纹理属性里面设置其对象

### 05.06 自动生成纹理坐标
* osg::TexGen
    - setMode
    - setPlane
* osg::StateSet::setTextureMode 设置 GL_TEXTURE_GEN_S, GL_TEXTURE_GEN_T

### 05.07 计算纹理坐标
* 通过一个纹理坐标生成器(继承自 osg::NodeVisitor 访问器)遍历模型的所有顶点及法线, 然后根据顶点,法线及一定的比例来确定纹理坐标
* osg::Geometry
	- getVertexArray()
	- getNormalArray()

### 05.08 立方体纹理 (细读)
* 通过派生 Transform, 可以预先移动子节点, 这样实现相机始终位于天空盒内的效果
* 创建一个节点回调, 可以在 Cull 回调中计算出新的纹理矩阵
* osg::TextureCubeMap
    - setImage --- 设置六个面的图像
* osg::StateSet::setRenderBinDetails --- 设置渲染顺序
* osg::Depth --- 设置深度属性
    - setFunction
    - setRange
* osg::ClearNode
* osg::Transform 可以预先计算矩阵

### 05.09 渲染到纹理 (细读)
* osg::AnimationPathCallback 根据构造函数创建动画路径回调
* osg::Geometry
    - setSupportsDisplayList(false): 可设置是否支持显示列表
* osg::Texture
	- setTextureSize: 创建纹理大小
* 学习如何使用 VBO
    - osg::Geometry::setVertexArray  
    - osg::Vec3Array::setVertexBufferObject  
    - osg::Geometry::setUseVertexBufferObjects --- 设置该几何体使用 VBO 
* 创建相机, 将其渲染到纹理(RTT)
    - osg::Camera
        - 设置清空背景颜色, 清空标识, 参考帧, 视口, 渲染目标实现
        - attach() --- 挂接到纹理或图像
		- setRenderTargetImplementation: 可设置渲染至 FBO
		- setPostDrawCallback: 设置后期绘制回调
* osg::Camera::DrawCallback
	- 相机的绘制回调

### 05.10 一维纹理
* osg::Texture1D: 一维纹理
* osg::TexGen: 生成纹理坐标

### 05.11 三维纹理
* 这里用四张图像组成一个三维纹理
* 使用自定义的 graphicscontext, 理由为何待研究

### 05.12 简单光源
* osg::Light 类派生自 osg::StateAttribute 类, 继承了对模式与属性参数信息的操作接口. 在 osg::light 类中通过 apply(State&state)函数将灯光的状态参数信息应用到OpenGL的状态机中
* osg::LightSource 类派生自 osg::Group 类, 将灯光作为一个节点可以加入到场景图中进行渲染.
    - void setReferenceFrame(ReferenceFrame rf) // 设置帧引用
        - 帧引用有两个枚举变量: RELATIVE_RF(相对帧引用), ABSOLUTE_RF(绝对帧引用)
    - 推荐在场景的顶部使用绝对的光源
* 法线的重缩放, 可以改善光照结果的明亮
    osg::StateSet* state = geode->getOrCreateStateSet();  
    state->setMode(GL_RESCALE_NORMAL, osg::StateAttribute::ON);  
* 自动单元化法线
    osg::StateSet* state = geode->getOrCreateStateSet();  
    state->setMode(GL_NORMALIZE, osg::StateAttribute::ON);  
* 允许根节点的光照以及两个光源
    osg::StateSet* state = root->getOrCreateStateSet();  
    state->setMode(GL_LIGHTING, osg::StateAttribute::ON);  
    state->setMode(GL_LIGHT0, osg::StateAttribute::ON);  
    state->setMode(GL_LIGHT1, osg::StateAttribute::ON);  
* 使用 osg::Light 设置光的属性, osg::LightSource 添加该对象

### 05.13 聚光灯
* 创建一张图像, 自中心逐渐衰减
    - osg::Image::data() 得到数据
* 创建动画路径: osg::AnimationPath
    - 设置循环模式 setLoopMode
    - 插入控制点(时间, 位置, 旋转) insert
* osg::MatrixTransform 可以设置矩阵变换, 例如
```
positioned->setMatrix(osg::Matrix::translate(-bs.center())*
	osg::Matrix::scale(size, size, size)*
	osg::Matrix::rotate(osg::inDegrees(180.0f), 0.0f, 0.0f, 2.0f));
```
* osg::MatrixTransform 可以设置更新回调为 osg::AnimationPathCallback 以实现路径动画
* 使用网格高度域创建高低不平的平面 osg::HeightField 
* osg::TexGenNode 用于创建一个节点, 可以自动生成纹理坐标
    - osg::TexGen 设置纹理坐标生成方法
        - setMode 和 setPlanesFromMatrix
* osg::Texture2D 可以设置边框颜色, wrap 
* osg::StateSet 可以用 setTextureMode 设置某个纹理单元的模式, 如纹理每个方向的生成是否打开
* 本例设置的是点光源, 所以没有聚光灯的效果, 要实现聚光灯的效果, 需要通过创建第二个纹理的方式实现, 在目标处自动生成纹理坐标

### 05.14 材质
* osg::Material 派生自 osg::StateAttribute 类, 其封装了 OpenGL 的 glMaterial() 和 glColorMaterial() 函数
* 设置节点的材质属性, 首先创建一个 osg::Material, 而后设置颜色和其他参数, 在关联到场景图形的 StateSet 中
```
osg::StateSet* state = node->getOrCreateStateSet();
osg::ref_ptr<osg::Material> mat = new osg::Material;
state->setAttribute(mat.get());
```
* 几个枚举类型: Face, ColorMode
* 要颜色跟踪材质需要调用 setColorMode()(内部应该是 glColorMaterial 函数), 不需要客户使用 setAttributeAndeModes() 来控制 GL_COLOR_MATERIAL
* 代码设置 osg::Material 属性
* 程序看起来是用了默认光照
