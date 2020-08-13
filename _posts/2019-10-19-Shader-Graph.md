---
layout: post
author: 贺工
---
标题：Shader Graph制作扫描消失和重现的视觉特效  
来源：公众号  
作者：Unity官方平台  
链接：https://mp.weixin.qq.com/s?__biz=MzU5MjQ1NTEwOA==&mid=2247502368&idx=1&sn=5dc2447592f5b9ec14a2e1a863382af9&chksm=fe1dfe8bc96a779d57fc3881e3cb81ee298b219f6cbfe329f52efaef59242d0daa75cc6cf9dd  
标签：Unity ShaderGraph   
内容： </br>
Unity教程｜手把手教你实现“美女与野兽”特效（上）
原创： Bogumił Mazurek Unity官方平台 今天
本文由来自波兰的游戏开发者Bogumił Mazurek分享使用Visual Effect Graph和Shader Graph制作“美女与野兽”的视觉特效。


 不要跟我引述远古秘法，巫婆。在制定它的时候，我也在场。

―《纳尼亚传奇1：狮子、女巫和魔衣橱》



很多开发者对女巫化身为络新妇妖怪的特效非常感兴趣，本篇教程将详解实现类似效果的方法，手把手教你使用Unity最新工具Visual Effect Graph和Shader Graph实现“美女与野兽“的视觉特效。



下面视频是我打算重现的效果。




项目和资源下载

本教程提供完整的工程项目下载，为了方便刚开始学习Unity的初学者，我们会从创建项目开始介绍。



受篇幅限制，本教程将分为上下二篇文章：

上篇将介绍：项目设置、角色和动画设置、自定义着色器制作。

下篇将介绍：编写MonoBehaviour脚本用于控制效果、使用Visual Effects Graph制作VFX、为粒子加入光线。



温馨提示：

本教程有大量截图和动画，并提供相应的注释。由于微信端自动压缩图片的问题，会导致部分图片设置模糊。我们将提供原始清晰版图片下载。

本教程提供带有附带演示场景的完整项目下载。运行时，请点击空格键让美女化身为野兽。

请发送“美女与野兽” 到微信后台，获得资源下载地址。


第一步：项目设置

为了使用Visual Effect Graph，我们需要把项目设置为使用可编程渲染管线SRP。由于通用渲染管线URP的支持推出得较晚，本项目我们就用了高清晰渲染管线HDRP。



Unity团队提供了相关的Unity官方手册和相关文章。考虑到有些初学者的需要，我在这部分先介绍如何设置这类项目。

 

首先，使用Unity 2018.3或更高版本，创建一个新项目，选中High-Definition RP模板。




图 01-创建HDRP项目



然后，我们打开资源包管理器，下载最新版High Definition RP资源包。




图 02-更新High Definition RP


过程中还需要制作着色器，所以我们要使用到Shader Graph。





图 03-导入Shader Graph



最后，下载并安装最新版Visual Effect Graph。




图 04-导入Visual Effect Graph



拥有最新工具后，我们要准备测试内容的场景。为了节省时间，我使用模板的SampleScene场景，去掉一些不必要的对象。




图 05-关卡设置


第二步：角色和动画

在完成Unity项目设置后，下一步就是找到项目中要使用的角色模型。



我们可以在很多地方找到可直接使用的免费模型。Asset Store资源商店是不错的地方，本文我们将使用的是Mixamo网站，这里不仅有角色模型，还有在Unity里适用的动画。

 


图 06-MIXAMO

 

我们的原始效果是展示一个美丽的女巫化身为丑陋的蜘蛛妖怪过程，所以本文制作的效果会把美女转变为野兽。

 

首先，找到怪兽的模型。我选择的是“Maw J Laygo”，该模型在Mixamo模型库中最具辨识度。为了让他看起来没那么可怕，我选择给他使用“Silly Dancing”动画。





图 07-选择角色和动画





图 08- 下载角色模型和动画

 

我为美女角色使用了“Pirate by P. Konstantinov”模型，并为她选择了“Snake Hip Hop Dance”舞蹈动画。

 



图 09-下载美女的角色模型和动画

 

我们将角色导入Unity并放置在场景中，接着进行基础配置。



在项目窗口选择模型，在检视窗口打开Materials标签页。勾选Import Materials，单击Extract Texture按钮，使用Extract Materials按钮对材质进行同样的处理。




图 10-材质设置

 

打开Animation标签页，勾选Import Animation。




图 11-动画设置

 

对于美女角色，我把Model标签页下的Scale Factor设为2，单击Apply按钮保存改动。

 


图 12-模型设置

 

现在给角色添加动画。为此，我们首先创建Animation Controller。




 图 13-创建Animation Controller



下一步，在编辑器打开Animation Controller，把动画拖到角色上，创建默认动画。




图 14-在Animation Controller设置默认动画

 

最后一步是为场景中的角色设置Animation Controller。




图 15-为角色设置Animation Controller

 

对两个角色都执行了以上操作后，我们可以看到角色开始动起来了。




图 16-动画设置效果


第三步：自定义着色器

现在为模型添加一些颜色。通常，我会使用标准着色器，然而我想要实现的效果不只需要自定义粒子系统，还需要自定义着色器，所以还需要一个着色器能够覆盖角色的部分位置。

 

首先使用HDRP/Lit Graph着色器模板，命名为Transformation。




图 17-使用模板创建着色器

 



图 18-空白着色器

 

现在，我们创建常见着色器中的参数，例如：BaseColorMap、NormalMap、NormalScale、SpecularMap、AmbientOcclusionMap和EmissionColor。



下图展示了如何连接我们创建的每个参数。




图 19-基础着色器设置

 

只有怪兽的模型包含发光纹理，我决定也为美女模型制作自定义发光纹理，从而实现更好的外观。



下图纹理的白点对应衣服上的扣环，我们可以让这些位置发出红光。




图 20－美女模型的发光纹理

 

下图展示了两个角色的材质设置。

 


图 21-角色的材质设置

 



图 22-材质设置效果

 

现在，我们处理着色器的遮罩部分。在着色器视图中，点击主节点右上角的齿轮图标，打开着色器设置，勾选Alpha Clipping。

 

创建ClippingLevel参数，该参数会影响每个像素的可见度。在世界空间中，位置值大于ClippingLevel值的像素都会被隐藏。





图 23-Alpha Clipping设置

 



图 24-ClippingLevel参数

 



图 25-AlphaClipTreshold节点设置

 



图 26-ClippingLevel的效果

 

如果我们对边缘加入不规则变化，分解效果会看起来更绚丽。为此，我们使用PerlinNoise纹理。




图 27-PerlinNoise纹理

 

为了使用该纹理，我们创建两个参数：一个是NoiseMap，作为纹理槽。另一个是NoiseSpread，用于存储噪声效果的大小信息。我们需要调整连接到主节点AlphaClippTreshold输入的节点。




图 28-NoiseMap和NoiseSpread节点设置
 



图 29-NoiseMap和NoiseSpread的效果
 

通过使用类似的方法，我们给边缘添加发光效果。使用的参数包括：GlowColor、GlowSpread和GlowPower ，这些参数来控制发光效果。请注意，我们需要混合边缘的发光颜色和之前添加的发光纹理。




图 30-发光部分的节点设置

 



图 31-Glow设置效果

 


图 32-Glow设置的效果动画

 
下图是改动后的材质设置。




图 33-材质设置


小结

在本篇教程中，我们学习了Unity项目设置、角色和动画设置以及自定义着色器制作。在下一篇中，我们将正式学习制作特效，敬请期待。



下载Unity Connect APP，请点击此处。 观看更多Unity官方精彩视频，请关注“Unity官方”B站账户。



你可以访问Unity答疑专区留下你的问题，Unity社区和官方团队帮你解答：

Connect.unity.com/g/discussion