---
layout: post
title: "Mecanim研究之Animator Controller"
description: "Mecanim研究之Animator Controller"
category: Unity3D
tags: [Unity3D]
---

* Table of Contents
{:toc}

&#160; &#160; &#160; &#160;学习Unity新版动画系统——Mecanim。依次实现运动状态机、人群模拟，多层IK绑定、武器绑定和导航网格巡逻功能，体验到Mecanim动画系统真正的强大之处。

<!-- more -->

##1.加载场景

###1.1.素材准备

&#160; &#160; &#160; &#160;在Assets Store上搜索“Mecanim Example”后，下载最新的官方资源包并导入。

&#160; &#160; &#160; &#160;如果导入后，感觉Project视图过于混乱，可以新建一个文件夹，命名为“Mecanim Example”，然后把其他文件全部拖到该文件夹下。

&#160; &#160; &#160; &#160;在Project下新建四个子目录，分别为：

1. _Scene:用于存放新建的场景文件
2. _Scripts:用于存放新建的脚本文件
3. _Animators:用于存放动画控制器
4. _AvatarMasks:用于存放AvatarMask

都建好之后，Project视图中的目录结构如下：

![导入结果](http://img17.poco.cn/mypoco/myphoto/20150806/14/17800049220150806145915039.png)

###1.2.创建场景

&#160; &#160; &#160; &#160;新建一个场景，重命名为MyAnimatorController，将其保持在目录Assets/_Scenes下。

&#160; &#160; &#160; &#160;添加环境对象。从Project视图的Assets/Mecanim Example/Prefabs目录下找到预制体tutorialArena_01_static，并拖放到场景中，重命名为Environment。重置其Transform组件，勾选Static选项，并为此对象生成Lightmaps。

&#160; &#160; &#160; &#160;添加角色。如果使用自带模型，把Assets/Mecanim Example/Characters/U/\_Character目录下的模型U\_Character\_REF拖放到场景中，并重命名为Player。但Mecanim是一个十分强大的动画系统，可以实现模型动画跨模型复用，可以自己随便找一个人物模型，放到Assets中，只要是Humanoid类型的模型，就可以表现出强大能力(我在过去的博文中介绍过)。我就选择了Unity的官方角色，Unity娘(UnityChan)。重置其Transform组件，然后设置其Tag为Player。调整Player到合适的位置，是Player站立在地面上，并不与环境中其他物体碰撞。

&#160; &#160; &#160; &#160;在Player对象上添加一个Character Controller组件(在Component->Physics菜单下)，设置其Center的值为(0,1.01,0)。其他属性保持默认。

![Character Controller组件](http://img17.poco.cn/mypoco/myphoto/20150806/15/17800049220150806151533027.png)

&#160; &#160; &#160; &#160;添加箱子模型。从Assets/Mecanim Example/Finished/Models目录下找到propCrate，将其拖到场景中，重置其Transform组件。移除其Animation组件，Position的Y值为0.3，X、Z值自行随意设置。

&#160; &#160; &#160; &#160;复制箱子模型。选中propCrate对象，按【Ctrl+D】复制propCrate对象，复制多份，调节X、Z值，让箱子在场景中均匀分布。

&#160; &#160; &#160; &#160;整理propCrate对象。创建一个空的游戏对象Crates，重置其Transform组件，讲之前生成的propCrate对象拖动到Crates下，使他们成为Crates的子对象。

&#160; &#160; &#160; &#160;添加跨栏模型。从Assets/Mecanim Example/Finished/Models目录下找到propHurdle，将其拖动到场景中，重置其Transform组件，调整X、Z值使之位于合适的位置。

&#160; &#160; &#160; &#160;如果模型显示如下：

![跨栏显示错误](http://img17.poco.cn/mypoco/myphoto/20150806/15/17800049220150806152555021.png)

那是由于propHurdle的子对象propHurdle_collision开启了Mesh Renderer，将其Mesh Renderer组件移除即可，移除后如下所示：

![正常显示跨栏](http://img17.poco.cn/mypoco/myphoto/20150806/15/1780004922015080615261907.png)

&#160; &#160; &#160; &#160;复制和整理跨栏模型。复制多份跨栏模型，修改X、Z值使跨栏模型均匀分布，也可以修改其Y轴旋转角度，使朝向不同，然后新建一个空对象Hurdles，讲之前的所有propHurdle都移动到Hurdles下，成为其子对象。

##2.创建动画控制器

&#160; &#160; &#160; &#160;在Project试图的Assets/_Animators目录下新建一个Animator Controller，重命名为MyAnimatorAC，并将其拖动到Player对象上，由于本场景将通过动画来控制角色的运动，所以需要勾选Animator Controller组件的Apply Root Motion选项。

&#160; &#160; &#160; &#160;在Animator编辑器中添加如下参数

1. 表示速度的float型变量Speed
2. 表示角色运动方向的float型变量Direction
3. 表示角色是否处于跳跃状态的bool型变量Jump
4. 表示角色是否处于打招呼状态的bool型变量Hi

![动画参数](http://img17.poco.cn/mypoco/myphoto/20150806/15/17800049220150806153801028.png)

在Base Layer中新建一个空的动画状态，重命名为Run，并双击进行编辑。入下图所示：

![动画状态Run](http://img17.poco.cn/mypoco/myphoto/20150806/15/17800049220150806154020042.png)

&#160; &#160; &#160; &#160;选择Parameter为Direction，勾选Automate Thresholds选项，在motion中依次添加RunLeft、Run、RunRight三个动画片段，设置Direction范围为(-1,1)。

&#160; &#160; &#160; &#160;设置完成后，Blend Tree的状态如下。

![Blend Tree的Motion列表](http://img17.poco.cn/mypoco/myphoto/20150806/15/17800049220150806154044015.png)

&#160; &#160; &#160; &#160;在Animator编辑器中返回Base Layer动画层。在Project视图中搜索动画片段Jump，然后将其拖放到Animator编辑器中创建状态Jump。

&#160; &#160; &#160; &#160;创建状态间的Transitions：

1. 当角色从状态Idel过渡到Run的条件为Speed大于0.1，同时除去HasExitTime勾选
2. 反过来从Run过渡到Idel的条件为Speed的值小于0.1，同时除去HasExitTime勾选
3. 当角色从Run过渡到Jump的条件为Jump等于true，同时除去HasExitTime勾选
4. 反过来由状态Jump过渡到Run时，保留默认条件，即ExitTime值约为0.9

创建后动画状态如下图所示：

![动画状态机](http://img17.poco.cn/mypoco/myphoto/20150806/15/17800049220150806155037038.png)

&#160; &#160; &#160; &#160;在本例中还有一个Say Hi打招呼的动作，该动作只需挥手臂，其余保持不动。在Animator编辑器重新建一个动画层Arm Layer，创建两个状态，一个Empty，一个Wave，Empty为空状态，Wave状态的Motion属性为动画片段Wave。

设置Empty和Wave两个状态之间的过渡条件：

1. 角色从状态Empty过渡到Wave的条件为参数Hi等于True，同时取消勾选HasExitTime选项
2. 角色从状态Wave过渡到Empty的条件为默认条件，即ExitTime约等于0.93

&#160; &#160; &#160; &#160;在Project试图的Assets/_AvatarMasks目录下新建一个Avatar Mask对象，重命名为RightArmAvatar，设置为只有右手可以运动，如下：

![设置Humanoid](http://img17.poco.cn/mypoco/myphoto/20150806/15/17800049220150806155826091.png)

&#160; &#160; &#160; &#160;在Arm Layer中，设置Weight的值为1，并设置Mask属性为前面创建的RightArmAvatar。在Mecanim动画系统中默认动画层Base Layer的权重总是1，其他层权重需要手动设置。

![Arm Layer](http://img17.poco.cn/mypoco/myphoto/20150806/16/17800049220150806160157011.png)

##3.角色运动控制

&#160; &#160; &#160; &#160;在Project试图的Assets/_Scripts目录下新建一个子目录Animator Controller，然后在该目录内新建一个脚本AnimatorMove.cs，并将其绑定到Player对象上。

&#160; &#160; &#160; &#160;在AnimatorMove类中加入如下变量

<pre><code>
	public float DirectionDampTime = .25f;
	private Animator animator;
</code></pre>

&#160; &#160; &#160; &#160;在Start中初始化变量Animator

<pre><code>
	void Start () {
		animator = GetComponent&lt;Animator> ();
	}
</code></pre>

&#160; &#160; &#160; &#160;在Update中加入对方向键处理的功能：

<pre><code>
	void Update () {
		if (animator == null) {
			return;
		}
		AnimatorStateInfo stateInfo = animator.GetCurrentAnimatorStateInfo (0);
		if (stateInfo.IsName ("Base Layer.Run")) {
			if (Input.GetButton ("Fire1")) {
				animator.SetBool ("Jump", true);
			}
		} else {
			animator.SetBool("Jump", false);
		}
		if (Input.GetButtonDown ("Fire2") && animator.layerCount >= 2) {
			animator.SetBool("Hi", !animator.GetBool("Hi"));
		}
		float h = Input.GetAxis("Horizontal");
		float v = Input.GetAxis("Vertical");
		animator.SetFloat ("Speed", h * h + v * v);
		animator.SetFloat ("Direction", h, DirectionDampTime, Time.deltaTime);
	}
</code></pre>

* `animator。SetFloat("Speed,h*h+v*v")`表示水平和垂直输入轴的值为Speed赋值，是一个近似值，可以用速度严格合成值`Mathf.Sqrt(h*h+v*v)`，看看效果。

&#160; &#160; &#160; &#160;运行游戏，按下方向键，角色移动，按下左【Alt】键角色挥手，跑动状态下按左【Ctrl】键角色跳跃。但由于摄像机位置固定，很容易跑出视野，接下来添加摄像机跟随角色运动。

##4.摄像机运动控制

&#160; &#160; &#160; &#160;在Assets/_Script下新建脚本CameraMover.cs，并绑定到Main Camera上，由于该脚本以后会常用，所以直接放在_Script目录下。

&#160; &#160; &#160; &#160;在CameraMover类中加入如下变量：

<pre><code>
	public Transform follow;
	public float distanceAway = 5.0f;
	public float distanceUp = 2.0f;
	public float smooth = 1.0f;
	private Vector3 targetPosition;
</code></pre>

&#160; &#160; &#160; &#160;在函数LateUpdate中加入如下代码：

<pre><code>
	void LateUpdate () {
		targetPosition = follow.position + 
			Vector3.up * distanceUp - 
			follow.forward * distanceAway;
		transform.position = Vector3.Lerp (transform.position, targetPosition,
			 Time.deltaTime * smooth);
		transform.LookAt (follow);
	}
</code></pre>

&#160; &#160; &#160; &#160;从Hierarchy中拖动对象Player到脚本CameraMover的变量follow上完成赋值

&#160; &#160; &#160; &#160;运行游戏，角色移动，摄像机会跟随角色运动。

##5.添加提示文字

&#160; &#160; &#160; &#160;在Project视图的Assets/_Scripts/Animator Controller目录下新建一个脚本MyAnimatorUI.cs，并将其拖动到对象Environment上

&#160; &#160; &#160; &#160;在MyAnimatorUI类中加入函数OnGUI，并添加提示代码：

<pre><code>
	void OnGUI()
	{
		GUILayout.Label("运动时按Fire1键(左Ctrl)实现跳跃动作。"
			+"按Fire2键(左Alt键）实现打招呼的动作。");
	}
</code></pre>


##6.实际运行效果

<script type="text/javascript" src="http://cdn.bootcss.com/jquery/2.1.4/jquery.min.js"></script>
<script type="text/javascript">
$(document).ready(function(){
	$("iframe").toggle();
  $(".btn1").click(function(){
  $("iframe").toggle();
  });
});
</script>
<iframe style="overflow: hidden;" src="http://acidg.pub/MyMecanim/MyAnimation/MyAnimation.html" width="630px" height="420px" scrolling="no"  frameborder="0"></iframe>
<button class="btn1">运行UnityWebPlayer版本</button>

&#160;运行[WebGL版本](http://acidg.pub/MyMecanim/MyAnimation2/index.html)