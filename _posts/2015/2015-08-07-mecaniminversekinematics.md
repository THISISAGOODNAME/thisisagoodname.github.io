---
layout: post
title: "Mecanim研究之Inverse Kinematics"
description: "Mecanim研究之Crowd Simulation"
category: Unity3D
tags: [Unity3D]
---

* Table of Contents
{:toc}

&#160; &#160; &#160; &#160;Inverse Kinematics，简称IK，即反向动力学。它是一种与前向动力学相对应的运动关系。一般来说骨骼动画都是传统的从父结点骨骼到子结点骨骼的带动方式。但有些情况下，比如上楼梯，就需要IK了。

<!-- more -->

&#160; &#160; &#160; &#160;本场景的目标：在Game窗口勾选“激活IK”选项之后，在Unity编辑器中移动Effector对象的位置时，角色的身体会跟随移动

##创建场景

&#160; &#160; &#160; &#160;在Project视图的Assets/_Scenes目录下复制场景MyCrowd，重命名为MyIK

&#160; &#160; &#160; &#160;移除原场景的Dude和Teddy对象以及Environment上绑定的脚本。本节不需要角色移动，移除Player上的Character Controller组件和AnimatorMove.cs脚本

&#160; &#160; &#160; &#160;调整Camera的位置使角色位于Game窗口的中央。然后设置脚本CameraMover中的便利distanceAway为-4，distanceUp为2.5

运行游戏，Camera自动调整到角色正对面

&#160; &#160; &#160; &#160;选中Player对象，创建一个空的子对象，重命名为Effectors，并重置其Transform组件

&#160; &#160; &#160; &#160;创建一个Sphere对象，重命名为Body Effector，设定其为Effectors的子对象，重置其Transform组件，设置scale为(0.1,0.1,0.1)调整Effectors的方位，使之位于角色正前方

&#160; &#160; &#160; &#160;讲Body Effector对象复制5份，各自的命名如下图：

![Effectors命名](http://img17.poco.cn/mypoco/myphoto/20150807/10/17800049220150807100834096.png)

##IK功能实现

&#160; &#160; &#160; &#160;在Assets/_Scripts目录下新建一个目录IK，在IK目录下新建脚本MyIk.cs，并拖动到Player上

&#160; &#160; &#160; &#160;在MyIK中加入以下代码：

<pre><code>
using UnityEngine;
using System.Collections;

public class MyIK : MonoBehaviour {

	public Transform bodyObj = null;
	public Transform leftFootObj = null;
	public Transform rightFootObj = null;
	public Transform leftHandObj = null;
	public Transform rightHandObj = null;
	public Transform lookAtObj = null;

	private Animator avatar;
	private bool ikActive = false;

	// Use this for initialization
	void Start () {
		avatar = GetComponent&lt;Animator> ();
	}
	
	// Update is called once per frame
	void Update () {
		if (!ikActive) {
			if (bodyObj != null) {
				bodyObj.position = avatar.bodyPosition;
				bodyObj.rotation = avatar.bodyRotation;
			}

			if (leftFootObj != null) {
				leftFootObj.position = avatar.GetIKPosition (AvatarIKGoal.LeftFoot);
				leftFootObj.rotation = avatar.GetIKRotation (AvatarIKGoal.LeftFoot);
			}

			
			if(rightFootObj != null) {
				rightFootObj.position = avatar.GetIKPosition(AvatarIKGoal.RightFoot);
				rightFootObj.rotation  = avatar.GetIKRotation(AvatarIKGoal.RightFoot);
			}				
			
			if(leftHandObj != null) {
				leftHandObj.position = avatar.GetIKPosition(AvatarIKGoal.LeftHand);
				leftHandObj.rotation  = avatar.GetIKRotation(AvatarIKGoal.LeftHand);
			}				
			
			if(rightHandObj != null) {
				rightHandObj.position = avatar.GetIKPosition(AvatarIKGoal.RightHand);
				rightHandObj.rotation  = avatar.GetIKRotation(AvatarIKGoal.RightHand);
			}				

			if (lookAtObj != null) {
				lookAtObj.position = avatar.bodyPosition + avatar.bodyRotation * new Vector3 (0, 0.5f, 1);
			}
		}
	}

	void OnAnimatorIK (int layerIndex) {
		if (avatar == null)
			return;

		if (ikActive) {
			avatar.SetIKPositionWeight (AvatarIKGoal.LeftFoot, 1.0f);
			avatar.SetIKPositionWeight (AvatarIKGoal.LeftFoot, 1.0f);
			
			avatar.SetIKPositionWeight(AvatarIKGoal.RightFoot, 1.0f);
			avatar.SetIKRotationWeight(AvatarIKGoal.RightFoot, 1.0f);
			
			avatar.SetIKPositionWeight(AvatarIKGoal.LeftHand, 1.0f);
			avatar.SetIKRotationWeight(AvatarIKGoal.LeftHand, 1.0f);
			
			avatar.SetIKPositionWeight(AvatarIKGoal.RightHand, 1.0f);
			avatar.SetIKRotationWeight(AvatarIKGoal.RightHand, 1.0f);

			avatar.SetLookAtWeight (1.0f, 0.3f, 0.6f, 1.0f, 0.5f);

			if (bodyObj != null) {
				avatar.bodyPosition = bodyObj.position;
				avatar.bodyRotation = bodyObj.rotation;
			}

			if (leftFootObj != null) {
				avatar.SetIKPosition (AvatarIKGoal.LeftFoot, leftFootObj.position);
				avatar.SetIKRotation (AvatarIKGoal.LeftFoot, leftFootObj.rotation);
			}
			
			if(rightFootObj != null) {
				avatar.SetIKPosition(AvatarIKGoal.RightFoot,rightFootObj.position);
				avatar.SetIKRotation(AvatarIKGoal.RightFoot,rightFootObj.rotation);
			}				
			
			if(leftHandObj != null) {
				avatar.SetIKPosition(AvatarIKGoal.LeftHand,leftHandObj.position);
				avatar.SetIKRotation(AvatarIKGoal.LeftHand,leftHandObj.rotation);
			}				
			
			if(rightHandObj != null) {
				avatar.SetIKPosition(AvatarIKGoal.RightHand,rightHandObj.position);
				avatar.SetIKRotation(AvatarIKGoal.RightHand,rightHandObj.rotation);
			}				

			if (lookAtObj != null) {
				avatar.SetLookAtPosition (lookAtObj.position);
			}
		} else {
			avatar.SetIKPositionWeight (AvatarIKGoal.LeftFoot, 0);
			avatar.SetIKRotationWeight (AvatarIKGoal.LeftFoot, 0);
			
			avatar.SetIKPositionWeight(AvatarIKGoal.RightFoot,0);
			avatar.SetIKRotationWeight(AvatarIKGoal.RightFoot,0);
			
			avatar.SetIKPositionWeight(AvatarIKGoal.LeftHand,0);
			avatar.SetIKRotationWeight(AvatarIKGoal.LeftHand,0);
			
			avatar.SetIKPositionWeight(AvatarIKGoal.RightHand,0);
			avatar.SetIKRotationWeight(AvatarIKGoal.RightHand,0);

			avatar.SetLookAtWeight (0.0f);
		}
	}

	void OnGUI () {
		GUILayout.Label("激活IK然后在场景中移动Effector对象观察效果");
		ikActive = GUILayout.Toggle(ikActive, "激活IK");
	}
}
</code></pre>

&#160; &#160; &#160; &#160;在Inspector窗口为变量赋值，比如Body Effector赋值给Body Obj

* 设置角色位置(如左脚)时，调用Animator类成员函数GetIKPosition，并指定IK目标为左脚，即AvatarGoal.LeftFoot，这样GetIKPosition就会返回IK计算出的目标点(左脚)的位置。与此类似，GetIKPosition返回IK目标点的旋转量，类型是Quaternion对象
* 当玩家激活IK后，各部分IK权重为1.0，各部分运动到目标位置；禁用IK后，各部分IK权重为0，各部分回到初始位置

&#160; &#160; &#160; &#160;运行游戏，在Game窗口中勾选“激活IK”后，在Hierarchy中选择Player对象，移动各个小球的位置，可以看到Game窗口中肢体部位也跟着移动

##视频演示

<iframe src="http://www.tudou.com/programs/view/html5embed.action?type=0&code=6Uwe-WFSczs&lcode=&resourceId=32216118_06_05_99" allowtransparency="true" allowfullscreen="true" allowfullscreenInteractive="true" scrolling="no" border="0" frameborder="0" style="width:480px;height:400px;"></iframe>