---
layout: post
title: "Mecanim研究之Crowd Simulation"
description: "Mecanim研究之Crowd Simulation"
category: Unity3D
tags: [Unity3D]
---

* Table of Contents
{:toc}

&#160; &#160; &#160; &#160;在Crowd Simulation场景中将实现如下目标：

1. 游戏开始时不断产生Dude人形模型并不断随机运动
2. 若玩家按下Fire1键则不断随机产生Teddy熊模型，直到设定的总人数

<!-- more -->

##1.创建场景

&#160; &#160; &#160; &#160;本次场景与Animator Controller相似，可以将MyAnimatorController场景复制一份，并重命名为MyCrowd

&#160; &#160; &#160; &#160;为了让场景看起来更空旷，以便能看清人群，可以移除Environment对象的三个子对象ramp_001、ramp_002和ramp_003，并移动场景中集装箱的位置，使场景中间空出来。在Hierarchy视图中移除Crates和Hurdles

&#160; &#160; &#160; &#160;添加Dude和Teddy对象

1. 从Project视图Assets/Mecanim Example/Characters/DudeLow目录下找到Dude_Low，将其拖动到Hierarchy视图中，重命名为Dude。
2. 从Project视图Assets/Mecanim Example/Characters/Teddy目录下找到模型Teddy，将其拖动到Hierarchy视图中，再调整Teddy的方位

此时，场景中共有三个角色，分别是Player、Dude和Teddy

##2.创建动画控制器

&#160; &#160; &#160; &#160;在Project视图的Assets/_Animators目录下将MyAnimatorAC复制一份，重命名为MyCrowdAC，将其拖动到Dude和Teddy上。勾选Dude和Teddy上Animator组件的Apply Root Motion选项

&#160; &#160; &#160; &#160;双击MyCrowdAC，在Animator编辑器中修改动画参数如下：

1. float型变量Speed
2. float型变量Direction
3. bool型变量Jump
4. bool型变量Dive

&#160; &#160; &#160; &#160;在Base Layer层上保持Idle、Run和Jump三个状态及他们之间的过渡关系不变；再仿照前面添加状态Jump的步骤添加一个新的状态Dive，实现俯冲

&#160; &#160; &#160; &#160;创建状态Run和Dive之间的Transitions：

1. 由状态Run过渡到Dive的条件为Dive等于true，取消勾选Has Exit Time选项
2. 由状态Dive过渡到Run的条件为默认选项

之后，动画转移状态如下：

![Base Layer状态机](http://img17.poco.cn/mypoco/myphoto/20150806/21/17800049220150806213906021.png)

&#160; &#160; &#160; &#160;勾选Base Layer的IK Pass选项，移除Arm Layer层；移除Environment对象上的脚本MyAnimatorUI.cs

##3.生成人群

&#160; &#160; &#160; &#160;在Project视图的Assets/_Scripts目录下新建一个子目录Crowd Simulation，新建一个脚本PlayerGenerator，并将其拖动到Environment上

&#160; &#160; &#160; &#160;在脚本如下：

<pre><code>
using UnityEngine;
using System.Collections;

public class PlayerGenerator : MonoBehaviour {

	public GameObject dude;
	public GameObject teddy;
	public int showCount = 0;
	public int maxPlayerCount = 50;

	static int count = 0;
	static float lastTime = 0;

	private float timeSpan = 0.1f;

	// Use this for initialization
	void Start () {
		lastTime = Time.time;
	}
	
	// Update is called once per frame
	void Update () {
		if (count < maxPlayerCount) {
			bool fired = Input.GetButton("Fire1");
			if ((Time.time - lastTime) > timeSpan) {
				if (dude != null && !fired) {
					Instantiate(dude, Vector3.zero, Quaternion.identity);
				}
				if (teddy != null && fired) {
					Instantiate(teddy, Vector3.zero, Quaternion.identity);
				}
				lastTime = Time.time;
				count++;
				showCount = count;
			}
		}
	}
}
</code></pre>

&#160; &#160; &#160; &#160;为PlayerGenerator脚本中的Dude和Teddy赋值，吧Dude和Teddy拖到上边

&#160; &#160; &#160; &#160;运行游戏，不断有克隆对象生成，但重叠在一起

##4.控制人群随机运动

&#160; &#160; &#160; &#160;在Assets/_Scripts/Crowd Simulation下新建脚本CrowdMovement.cs，分别拖动到Dude和Teddy上

&#160; &#160; &#160; &#160;CrowdMovement.cs脚本如下

<pre><code>
using UnityEngine;
using System.Collections;

public class CrowdMovement : MonoBehaviour {

	public float AvatarRange = 25;

	private Animator animator;
	private float SpeedDampTime = .25f;
	private float DirectionDampTime = .25f;
	private Vector3 TargetPosotion = Vector3.zero;

	// Use this for initialization
	void Start () {
		animator = GetComponent&lt;Animator> ();
	}
	
	// Update is called once per frame
	void Update () {
		if (animator == null)
			return;
		int r = Random.Range (0, 50);
		animator.SetBool ("Jump", r == 20);
		animator.SetBool ("Dive", r == 30);

		if (Vector3.Distance (TargetPosotion, animator.rootPosition) > 5) {
			animator.SetFloat ("Speed", 1, SpeedDampTime, Time.deltaTime);
			Vector3 currentDir = animator.rootRotation * Vector3.forward;
			Vector3 wantedDir = (TargetPosotion - animator.rootPosition).normalized;

			if (Vector3.Dot (currentDir, wantedDir) > 0) {
				animator.SetFloat ("Direction", Vector3.Cross (currentDir, wantedDir).y, DirectionDampTime, Time.deltaTime);
			} else {
				animator.SetFloat ("Direction", Vector3.Cross (currentDir, wantedDir).y > 0 ? 1 : -1, DirectionDampTime, Time.deltaTime);
			}
		} else {
			animator.SetFloat ("Speed", 0, SpeedDampTime, Time.deltaTime);
			if (animator.GetFloat ("Speed") &lt; 0.01f) {
				TargetPosotion = new Vector3(
					Random.Range (-AvatarRange, AvatarRange),
					0,
					Random.Range (-AvatarRange, AvatarRange));
			}
		}
	}
}
</code></pre>

&#160; &#160; &#160; &#160;运行游戏，可以看到不停生成直到50个Dude或者Teddy(按Fire1键)，并随机运动

##5.实际运行效果

<script type="text/javascript" src="http://cdn.bootcss.com/jquery/2.1.4/jquery.min.js"></script>
<script type="text/javascript">
$(document).ready(function(){
	$("iframe").toggle();
  $(".btn1").click(function(){
  $("iframe").toggle();
  });
});
</script>
<iframe style="overflow: hidden;" src="http://acidg.pub/MyMecanim/MyCrowd/MyCrowd.html" width="630px" height="420px" scrolling="no"  frameborder="0"></iframe>
<button class="btn1">运行UnityWebPlayer版本</button>

&#160;运行[WebGL版本](http://acidg.pub/MyMecanim/MyCrowd2/index.html)
