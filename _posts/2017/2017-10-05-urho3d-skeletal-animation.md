---
layout: post
title: "Urho3D骨骼动画系统"
description: "Urho3D骨骼动画系统"
category: CG
tags: [CG,c++,游戏引擎,Urho3D]
---

&nbsp; &nbsp; &nbsp; &nbsp;之前一篇博文自己实现了一个骨骼动画系统，其实还是比较复杂的，其实从某个角度讲，建立在Scene Graph以下的渲染/动画系统，都没有啥实用价值。这年头，连UI系统(比如安卓/IOS/Qt Quick)都建立在Scene Graph以上。

<!-- more -->

* Table of Contents
{:toc}

# Urho3D中的骨骼动画控制方式

&nbsp; &nbsp; &nbsp; &nbsp;Urho3D中提供了两者控制骨骼动画的方式，一种是完全手工控制，使用AnimationState对象，在E_UPDATE事件回调中手工计算下一帧的状态。另一种是交给引擎来托管，使用AnimationController组件，由引擎自动完成动画循环，开发者可以直接使用AnimationController提供的Play/Stop等方法操作动画片段，不用关心每帧的绘制方法，非常方便。

# 手工方式，使用AnimationState

&nbsp; &nbsp; &nbsp; &nbsp;首先创建一个名为Jack的Node

```c++
jackNode_ = scene_->CreateChild("Jack");
jackNode_->SetPosition(Vector3(-5.0f, 0.0f, 20.0f));
AnimatedModel* modelObject = jackNode_->CreateComponent<AnimatedModel>();
>GetResource<Material>("Materials/Jack.xml"));
modelObject->SetModel(cache->GetResource<Model>("Models/Kachujin/Kachujin.mdl"));
modelObject->SetMaterial(cache->GetResource<Material>("Models/Kachujin/Materials/Kachujin.xml"));
modelObject->SetCastShadows(true);
```

&nbsp; &nbsp; &nbsp; &nbsp;为Jack节点添加动画片段，并设置动画播放相关的属性

```c++
Animation* walkAnimation = cache->GetResource<Animation>("Models/Kachujin/Kachujin_Walk.ani");

AnimationState* state = modelObject->AddAnimationState(walkAnimation);
// The state would fail to create (return null) if the animation was not found
if (state)
{
	// Enable full blending weight and looping
   state->SetWeight(1.0f);
   state->SetLooped(true);
   state->SetTime(Random(walkAnimation->GetLength()));
}
```

&nbsp; &nbsp; &nbsp; &nbsp;要让动画播放，需要在E_UPDATE事件绑定的回调函数中，添加如下逻辑

```c++
// Get the model's first (only) animation state and advance its time. Note the convenience accessor to other components
// in the same scene node
AnimatedModel* model = jackNode_->GetComponent<AnimatedModel>(true);
if (model->GetNumAnimationStates())
{
	AnimationState* state = model->GetAnimationStates()[0];
	state->AddTime(timeStep);
}
```

> 注意，如果要暂停动画，可以state->AddTime(0);

&nbsp; &nbsp; &nbsp; &nbsp;完整代码可以参考[这里](https://github.com/THISISAGOODNAME/urho3DSamples/blob/master/samples/11-navigation/main.cpp)

# 自动方式，使用AnimationController

&nbsp; &nbsp; &nbsp; &nbsp;在创建jackNode后，添加AnimationController组件

```c++
jackNode_->CreateComponent<AnimationController>();
```

&nbsp; &nbsp; &nbsp; &nbsp;之后，可以直接播放特定名称的动画片段

```c++
static const char* WALKING_ANI = "Models/Kachujin/Kachujin_Walk.ani";

AnimationController* animCtrl = jackNode_->GetComponent<AnimationController>();

if (animCtrl->IsPlaying(WALKING_ANI)) {
                
} else {
	animCtrl->Play(WALKING_ANI, 0, true, 0.1f);
}


// Stop
// animCtrl->Stop(WALKING_ANI, 0.5f);                
```

> AnimationController会自动处理动画加载，帧更新问题，非常方便

&nbsp; &nbsp; &nbsp; &nbsp;完整代码可以参考[这里](https://github.com/THISISAGOODNAME/urho3DSamples/blob/master/samples/14-crowdNavigation/main.cpp)

# 应用场景

&nbsp; &nbsp; &nbsp; &nbsp;手工控制虽然比较麻烦，但是响应性更好一些，比较推荐处理玩家操作角色。自动控制动画则用来绘制NPC的逻辑。