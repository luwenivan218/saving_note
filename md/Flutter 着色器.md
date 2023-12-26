> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7311150820578033727)

着色器
===

在计算机图形学中，着色器是一种计算机程序，用于在 3D 场景渲染过程中计算适当的光、暗和颜色级别，这一过程称为着色。

着色器已经发展到现在可以在计算机图形特效和视频后处理以及图形处理单元上的通用计算中执行各种专门功能。

当您编写并运行 Dart 代码时，编译器会将您的代码转换为机器代码。它本质上是将人类可读的语言翻译成您的硬件可以理解的语言。在大多数情况下，该硬件就是您的 CPU。所以在编写着色器代码的时候我们不直接使用 Dart。那么我们将在 Flutter 中使用的语言是 [GLSL——](https://link.juejin.cn?target=https%3A%2F%2Fwww.khronos.org%2Fopengl%2Fwiki%2FCore_Language_(GLSL) "https://www.khronos.org/opengl/wiki/Core_Language_(GLSL)")一种来自 OpenGL 库的着色语言。

创建
==

在项目目录中创建一个`.frag`结尾的着色器文件：

```
#version 460 core

out vec4 fragColor;

void main() {
   fragColor = vec4(1, 0, 0, 1);
}


```

#version 460 core

*   告诉编译器我们将使用哪个版本的 GLSL。除非您需要特定版本的某些特定功能，否则请始终保持此行相同。 

out vec4 fragColor;

*   接下来，我们需要声明一个输出变量。它是一个变量，在着色器的末尾，我们必须存储将在屏幕上显示的输出颜色。在变量声明之前我们通过使用`out`关键字来做到这一点。您可能还注意到我声明的变量是某种奇怪的`vec4`类型。
*   在 GLSL 中，我们编写的代码级别比 Dart 低得多。因此，我们实际上并没有一堆花哨的类来表示颜色等复杂的东西。相反，我们直接处理该颜色组成的数据。但我们并不完全依靠自己——我们有载体。 
*   向量是一种数据结构，基本上将一堆数字组合成一个实体，然后通过公开一些有用的 getter 和运算符使您的生活变得更轻松，从而使算法的实现变得更加愉快。因此，为了表示一种颜色，我们需要四个数字——红色、绿色、蓝色和 alpha 通道分量——因此`vec4`.

使用
--

1、我们可以使用 ShaderBuilder 组件来加载加载着色器

```
ShaderBuilder(
   assetKey: 'xxx.frag',
   (BuildContext context, FragmentShader shader, _) => CustomPaint(
     size: MediaQuery.of(context).size,
       painter: MyShaderPainter(shader),
      ),
  );


```

2、 我们可以使用 FutureBuilder 来加载着色器

```
 FutureBuilder<ui.FragmentShader>(
     //从着色器文件夹加载着色器。
     future: _loadShader(widget.weatherCondition.shader),
     builder: (context, snapshot) {
       .....
    }
 }

     
///使用指定的[shaderName]从资源加载并返回[ui.FragmentShader]。
///此方法从具有给定[shaderName]的assets文件夹加载着色器程序
///并返回相应的[ui.FragmentShader]。
///参数：
///-[shaderName]要加载的着色器文件的名称。
Future<ui.FragmentShader> _loadShader(String shaderName) async {
  //从资源文件夹加载着色器程序。
  ui.FragmentProgram program =
      await ui.FragmentProgram.fromAsset('shaders/$shaderName');
 //返回片段着色器。
  return program.fragmentShader();
}


```

例子
==

1、雨滴效果

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a4e17b351cf8412cb6f58c1686314772~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=295&h=640&s=4578725&e=gif&f=67&b=07182b)

```
#include <flutter/runtime_effect.glsl>
uniform float iTime;
uniform vec3 iResolution;
uniform sampler2D iChannel0;
out vec4 fragColor;
// Licensed under "Do What The Fuck You Want To Public License"
// http://www.wtfpl.net/about/
vec2 hash22(vec2 p){
    vec2 p2 = fract(p * vec2(.1031,.1030));
    p2 += dot(p2, p2.yx+19.19);
    return fract((p2.x+p2.y)*p2);
}
#define round(x) floor( (x) + .5 )
float simplex2D(vec2 p ){
    const float K1 = (sqrt(3.)-1.)/2.;
    const float K2 = (3.-sqrt(3.))/6.;
    const float K3 = K2*2.;
    vec2 i = floor( p + dot(p,vec2(K1)) );
    vec2 a = p - i + dot(i,vec2(K2));
    vec2 o = 1.-clamp((a.yx-a)*1.e35,0.,1.);
    vec2 b = a - o + K2;
    vec2 c = a - 1.0 + K3;
    vec3 h = clamp( .5-vec3(dot(a,a), dot(b,b), dot(c,c) ), 0. ,1. );
    h*=h;
    h*=h;
    vec3 n = vec3( 
        dot(a,hash22(i   )-.5),
        dot(b,hash22(i+o )-.5),
        dot(c,hash22(i+1.)-.5)
    );
    return dot(n,h)*140.;
}

vec2 wetGlass(vec2 p) {

    p += simplex2D(p*0.1) * 3.; // distort drops
    
    float t = iTime;
    
    p *= vec2(.025, .025 * .25);
    
    p.y += t * .25; // make drops fall
    
    vec2 rp = round(p);
    vec2 dropPos = p - rp;
    vec2 noise = hash22(rp);
    
    dropPos.y *= 4.;
    
    t = t * noise.y + (noise.x*6.28);
    
    vec2 trailPos = vec2(dropPos.x, fract((dropPos.y-t)*2.) * .5 - .25 );
    
    dropPos.y += cos( t + cos(t) );  // make speed vary
   
    float trailMask = clamp(dropPos.y*2.5+.5,0.,1.); // hide trail in front of drop

    float dropSize  = dot(dropPos,dropPos);
    
    float trailSize = clamp(trailMask*dropPos.y-0.5,0.,1.) + 0.5;
    trailSize = dot(trailPos,trailPos) * trailSize * trailSize;
    
    float drop  = clamp(dropSize  * -60.+ 3.*noise.y, 0., 1.);
    float trail = clamp(trailSize * -60.+ .5*noise.y, 0., 1.);
    
    trail *= trailMask; // hide trail in front of drop
    
    return drop * dropPos + trailPos * trail;
}

void main()
{
    vec2 fragCoord = FlutterFragCoord();
	vec2 uv = fragCoord.xy / iResolution.xy;
    
    uv += wetGlass(fragCoord);
   
	fragColor = texture(iChannel0,uv);
}


```

2、雪片效果

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ab2d02fbf1ae4e7e9d3bc2634b6c87d8~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=295&h=640&s=3757730&e=gif&f=58&b=020007)

```
#include <flutter/runtime_effect.glsl>

uniform float iTime;
uniform vec3 iResolution;

out vec4 fragColor;

// Copyright (c) 2013 Andrew Baldwin (twitter: baldand, www: http://thndl.com)
// License = Attribution-NonCommercial-ShareAlike (http://creativecommons.org/licenses/by-nc-sa/3.0/deed.en_US)
// https://www.shadertoy.com/view/ldsGDn

// "Just snow"
// Simple (but not cheap) snow made from multiple parallax layers with randomly positioned 
// flakes and directions. Also includes a DoF effect. Pan around with mouse.

#define LAYERS 30
#define DEPTH .1
#define WIDTH .3
#define SPEED .8

void main()
{
  vec2 fragCoord = FlutterFragCoord();
  const mat3 p = mat3(13.323122,23.5112,21.71123,21.1212,28.7312,11.9312,21.8112,14.7212,61.3934);
  vec2 uv = vec2(0.5, 0.5) + vec2(1.,iResolution.y/iResolution.x)*fragCoord.xy / iResolution.xy;
  vec3 acc = vec3(0.0);
  float dof = 5.*sin(iTime*.1);
  for (int i=0;i<LAYERS;i++) {
    float fi = float(i);
    vec2 q = uv*(1.+fi*DEPTH);
    q += vec2(q.y*(WIDTH*mod(fi*7.238917,1.)-WIDTH*.5),SPEED*iTime/(1.+fi*DEPTH*.03));
    vec3 n = vec3(floor(q),31.189+fi);
    vec3 m = floor(n)*.00001 + fract(n);
    vec3 mp = (31415.9+m)/fract(p*m);
    vec3 r = fract(mp);

    
vec2 s = abs(mod(q,1.)-.5+.9*r.xy-.45);
    s += .01*abs(2.*fract(10.*q.yx)-1.);
    float d = .6*max(s.x-s.y,s.x+s.y)+max(s.x,s.y)-.01;
    float edge = .005+.05*min(.5*abs(fi-5.-dof),1.);
    acc += vec3(smoothstep(edge,-edge,d)*(r.x/(1.+.02*fi*DEPTH)));
  }
  fragColor = vec4(vec3(acc),1.0);
}


```