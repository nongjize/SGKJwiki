---
layout: post
author: 贺工
---
标题：【贺靖轩】将shadertoy中的Improved matrix rain翻译为Unity中可用的Shader   
作者：贺靖轩  
标签：Unity Shader Matrix   
内容： </br>
https://blog.csdn.net/newtonsm/article/details/103821007   
Improved matrix rain的链接



因最近有需要要做一个类似骇客帝国数字矩阵的效果，正好发现shadertoy网上有大佬做了一个很取巧的着色器，靠纯shader代码就实现了类似的效果，所以把大佬的代码作为shader练习翻译了一下，在unity环境运行良好。着色器代码原文详见文末，现使用unity 2018.4.4f1环境开始着手翻译：

1.创建一个新的Unlit shader
命名为

Shader "Unlit/ImprovedMatrixRain"
2.删除多余的属性和代码
删除fog相关，以及类似下方的代码

_MainTex ("Texture", 2D) = "white" {}
 
            sampler2D _MainTex;
            float4 _MainTex_ST;
3.修改深度和剔除
在SubShader头部添加：

Cull Off ZWrite Off ZTest Always
4.修改顶点着色器
修改o.uv为屏幕UV

v2f vert (appdata v)
            {
                v2f o;
                o.vertex = UnityObjectToClipPos(v.vertex);
                //o.uv = TRANSFORM_TEX(v.uv, _MainTex);
				o.uv = ComputeScreenPos(o.vertex);
                return o;
            }
5.修改片元着色器
float4 frag (v2f i) : SV_Target
            {
				float2 uv = i.uv.xy;
				
				float2 st = float2(uv.x*_ScreenParams.x / _ScreenParams.y, uv.y);
				float3 color = matrix_func(st)*float3(0.2, 1.0, 0.2);
                return float4(color,1);
            }
其中vec2 uv = fragCoord/ iResolution.xy;

所以将

vec2 st = fragCoord/min(iResolution.x, iResolution.y);
修改为

float2 uv = i.uv.xy;
float2 st = float2(uv.x*_ScreenParams.x / _ScreenParams.y, uv.y);
6.修改主入口渲染程序matrix_func(float2 st)
注意保留关键字matrix。

Unity Shader _Time 的单位

名称	类型	说明
_Time	float4	t 是自该场景加载开始所经过的时间，4个分量分别是 (t/20, t, t*2, t*3)
_SinTime	float4	t 是时间的正弦值，4个分量分别是 (t/8, t/4, t/2, t)
_CosTime	float4	t 是时间的余弦值，4个分量分别是 (t/8, t/4, t/2, t)
unity_DeltaTime	float4	dt 是时间增量，4个分量的值分别是(dt, 1/dt, smoothDt, 1/smoothDt)
所以：

float iTime = _Time.y;

然后就是vec改为float。//精度float>half>fixed

取小数部分的函数fract改为frac。

因为参数不一致，将简写

vec3(rchar(ipos,fpos)*bright*glow);
修改为

float3(1,1,1)*rchar(ipos, fpos)*bright*glow;
7.补全其它函数
该替换替换，注意函数顺序即可，解释性语言对先后循序一般都有要求。

8.结束，enjoy！
源码下载链接

 

shadertoy的着色器代码原文：

float random(in float x){
    return fract(sin(x)*43758.5453);
}
 
float random(in vec2 st){
    return fract(sin(dot(st.xy,vec2(12.9898,78.233)))*43758.5453);
}
 
float rchar(in vec2 ipos,in vec2 fpos){
    // num of pixels in individual characters
    float grid=5.;
    // size of black borders to overlay on characters
    vec2 margin=vec2(.2,.05);
    float seed=23.;
    // mask for character borders
    vec2 borders=step(margin,fpos)*step(margin,1.-fpos);
    
    // randomly choose each character pixel to be on or off
    float chardata=random(ipos*seed+floor(fpos*grid));
    return step(.5,chardata)*borders.x*borders.y;
}
 
// st ranges from 0.0-1.0
vec3 matrix(in vec2 st){
    float rows=50.;
    
    // ipos: which character we're displaying
    vec2 ipos=floor(st*rows);
    
    // pick a random brightness
    vec2 ipos2=ipos+vec2(.0,floor((iTime+200.)*10.*random(ipos.x+1.)));
    // mix square and sawtooth waves
    float bright=-(abs((sin(ipos2.y/10.+2.))))*(ipos2.y/10.-floor(ipos2.y/10.))*.7+.2;
    
    // fpos: the position within the character
    vec2 fpos=fract(st*rows);
    vec2 center=(.5-fpos);
    float glow=(1.-dot(center,center)*3.)*2.;
    // glow = 1.0;
    
    return vec3(rchar(ipos,fpos)*bright*glow);
}
 
void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    vec2 st = fragCoord/min(iResolution.x, iResolution.y);
  
    vec3 color=matrix(st)*vec3(.2,1.,.2);
    fragColor=vec4(color,1.);
}
