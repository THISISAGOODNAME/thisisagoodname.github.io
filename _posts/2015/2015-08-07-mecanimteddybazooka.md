---
layout: post
title: "Mecanim研究之Teddy Bazooka"
description: "Mecanim研究之Teddy Bazooka"
category: Unity3D
tags: [Unity3D]
---

* Table of Contents
{:toc}

&#160; &#160; &#160; &#160;在这个Teddy熊火箭炮场景中要实现的目标是：

1. 方向键控制Player运动
2. 按Fire2键Player拿起武器或卸下武器
3. 按Fire1键火箭炮发射炮弹，若击中Teddy，则Teddy倒地毙命

<!-- more -->

##创建场景

&#160; &#160; &#160; &#160;复制Assets/_Scenes目录下的场景MyCrowd，并重命名为MyBazooka

&#160; &#160; &#160; &#160;在Hierarchy视图中选中Player对象，移除Character Controller组件和Animator Move脚本；选中Environment对象，移除脚本PlayerGenerator.cs；选中Teddy对象，移除脚本CrowdMovement.cs

&#160; &#160; &#160; &#160;为Player对象添加动画控制器：在Project视图的Assets/_Animators目录下将MyCrowdAC复制一份，重命名为MyWeaponAC，并将其拖动到Player对象上

##编辑动画控制器

&#160; &#160; &#160; &#160;双击MyWeaponAC进入Animator编辑器，勾选Base Layer上的IK Pass选项。新建一个动画层HandIK。勾选HandIK上的IK Pass选项，用以在此动画层上实现角色抬火箭炮时双手抬起的动作

设置完成后，此时两个动画层均支持IK

&#160; &#160; &#160; &#160;在Animator编辑器中修改参数如下：

![MyWeaponAC](http://img17.poco.cn/mypoco/myphoto/20150807/11/17800049220150807111752096.png)

* Jump和Dive两个变量是无用变量，可以删除

&#160; &#160; &#160; &#160;在Hierarchy视图中移除对象Dude，为Teddy对象添加一个Capsule Collider组件，属性如下：

![Teddy的Capsule Collider](http://img17.poco.cn/mypoco/myphoto/20150807/11/17800049220150807112053042.png)

&#160; &#160; &#160; &#160;为Teddy添加动画控制器：把Project视图下Assets/_Animators目录下的MyCrowdAC复制一份，重命名为MyBearAC，拖动到对象Teddy上

&#160; &#160; &#160; &#160;在Animator编辑器中修改MyBearAC的参数如下：

![MyBearAC](http://img17.poco.cn/mypoco/myphoto/20150807/11/17800049220150807112724061.png)

* 其中Dying和Reviving为Assets目录中搜索再拖动到动画状态机中
* Any State到Dying的Transition为Dying为true
* 由状态Dying过渡到Reviving和由Reviving过渡到Idel的Transition均为默认

##Teddy对象随机运动的控制

&#160; &#160; &#160; &#160;在Assets/_Scripts目录下新建一个目录Weapon，在该目录下新建一个脚本MyBear.cs，并绑定到Teddy对象上，脚本如下：

<pre><code>
using UnityEngine;
using System.Collections;

public class MyBear : MonoBehaviour {

	public float AvatarRange = 25;

	private Animator avatar;
	private float SpeedDampTime = .25f;
	private float DirectionDampTime = .25f;
	private Vector3 TargetPosition = new Vector3 (0, 0, 0);

	// Use this for initialization
	void Start () {
		avatar = GetComponent&lt;Animator> ();
	}
	
	// Update is called once per frame
	void Update () {
		if(avatar == null) return;
		
		int r = Random.Range(0, 50);
		avatar.SetBool("Jump", r == 20);
		avatar.SetBool("Dive", r == 30);
			
		if(Vector3.Distance(TargetPosition, avatar.rootPosition) > 5) {
			avatar.SetFloat("Speed", 1, SpeedDampTime, Time.deltaTime);
			Vector3 curentDir = avatar.rootRotation * Vector3.forward;
			Vector3 wantedDir = (TargetPosition - avatar.rootPosition).normalized;
			
			if(Vector3.Dot(curentDir,wantedDir) > 0)
				avatar.SetFloat("Direction",
				                Vector3.Cross(curentDir,wantedDir).y, 
				                DirectionDampTime, Time.deltaTime);
			else
				avatar.SetFloat("Direction", 
				                Vector3.Cross(curentDir,wantedDir).y > 0 ? 1 : -1, 
				                DirectionDampTime, Time.deltaTime);
		} else {
			avatar.SetFloat("Speed", 0, SpeedDampTime, Time.deltaTime);
			if(avatar.GetFloat("Speed") < 0.01f)
				TargetPosition = new Vector3(
					Random.Range(-AvatarRange,AvatarRange), 0, 
					Random.Range(-AvatarRange,AvatarRange));
		}

		var nextStage = avatar.GetNextAnimatorStateInfo (0);
		if (nextStage.IsName ("Base Layer.Dying")) {
			avatar.SetBool ("Dying", false);
		}
	}

	void OnCollisionEnter (Collision collision) {
		if (avatar != null) {
			AnimatorStateInfo currentState = avatar.GetCurrentAnimatorStateInfo(0);
			AnimatorStateInfo nextState = avatar.GetNextAnimatorStateInfo(0);
			if (!currentState.IsName("Base Layer.Dying") && 
				!nextState.IsName("Base Layer.Dying")) {
				avatar.SetBool ("Dying", true);
			}
		}
	}
}
</code></pre>

##抬防火箭炮的控制

&#160; &#160; &#160; &#160;在Project视图的Assets/_Script/Weapon目录下新建一个脚本MyBazooka.cs，并绑定到Player对象上，脚本如下：

<pre><code>
using UnityEngine;
using System.Collections;

public class MyBazooka : MonoBehaviour {

	public GameObject targetA = null;
	public Transform leftHandPos = null;
	public Transform rightHandPos = null;
	public GameObject bazoo = null;
	public GameObject bullet = null;
	public Transform spawn = null;

	private Animator animator;
	private bool load = false;

	// Use this for initialization
	void Start () {
		animator = GetComponent&lt;Animator> ();
	}
	
	// Update is called once per frame
	void Update () {
		if (animator == null)
			return;
		animator.SetFloat ("Aim", load ? 1 : 0, .1f, Time.deltaTime);
		float aim = animator.GetFloat ("Aim");
		if (Input.GetButton ("Fire2")) {
			if (load && aim > 0.99) {
				load = false;
			} else if (!load && aim &lt; 0.01) {
				load = true;
			}
		}

	}

	void OnAnimatorIK (int layerIndex) {
		float aim = animator.GetFloat ("Aim");

		if (layerIndex == 0) {
			if (targetA != null) {
				Vector3 target = targetA.transform.position;
				target.y = target.y + 0.2f * (target - animator.rootPosition).magnitude;
				animator.SetLookAtPosition (target);
				animator.SetLookAtWeight (aim, 0.5f, 0.5f, 0.0f, 0.5f);
				if (bazoo != null) {
					Vector3 pos = new Vector3 (0.195f, -0.0557f, -0.155f);
					Vector3 scale = new Vector3 (0.2f, 0.8f, 0.2f);

					scale = scale * aim;
					bazoo.transform.localScale = scale;
					bazoo.transform.localPosition = pos;
				}
			}
		}

		if (layerIndex == 1) {
			if (leftHandPos != null) {
				animator.SetIKPosition (AvatarIKGoal.LeftHand, leftHandPos.position);
				animator.SetIKRotation (AvatarIKGoal.LeftHand, leftHandPos.rotation);

				animator.SetIKPositionWeight (AvatarIKGoal.LeftHand, aim);
				animator.SetIKRotationWeight (AvatarIKGoal.LeftHand, aim);
			}

			if (rightHandPos != null) {
				animator.SetIKPosition (AvatarIKGoal.RightHand, rightHandPos.position);
				animator.SetIKRotation (AvatarIKGoal.RightHand, rightHandPos.rotation);
				
				animator.SetIKPositionWeight (AvatarIKGoal.RightHand, aim);
				animator.SetIKRotationWeight (AvatarIKGoal.RightHand, aim);
			}
		}
	}

	void OnGUI () {
		GUILayout.Label("按Fire1键发射炮弹");
		GUILayout.Label("按Fire2键抬起或放下火箭炮");
	}
}

</code></pre>

&#160; &#160; &#160; &#160;在Hierarchy视图中新建一个Cylinder对象，重命名为Bazoo，将其拖动到Player对象的子对象joint_Head下面，使之成为后者的子对象。设置其Position值(0.2,-0.06,-0.15)，Rotation的值为(0,180,100)，scale的值为(0.2,0.8,0.2)。设置其Mesh Renderer的Material为mat_metalCorrugated_var03

&#160; &#160; &#160; &#160;选中Bazoo，创三个空子对象，分别命名为LeftHandle、RightHandle和Spawn

> LeftHandle设置如下
> > Position(-0.57,0,-0.65)
> > 
> > Rotation(0,90,-90)
> > 
> > Scale(0.01,0.01,0.01)
>
> RightHandle设置如下
> > Position(-0.57,0,0.65)
> > 
> > Rotation(0,90,90)
> > 
> > Scale(0.01,0.01,0.01)
> 
> Spawn设置如下
> > Position(0.0,1.05,0)
> > 
> > Rotation(-90,0,0)
> > 
> > Scale(0.01,0.01,0.01) 

&#160; &#160; &#160; &#160;在Hierarchy视图中新建一个Sphere对象，重命名为Bullet，设置其Transform组件的Position为(0,-2.2,0)，Rotation为(0,0,0)，Scale为(0.2,0.2,0.2)。为其添加一个Sphere Collider组件，设置其Radius的值为0.5，其他默认，添加一个Rigidbody组件，设置其Mass为0.1，其他默认。最后，从Project视图下搜索材质propHurdle_DEF，赋给Bullet的Mesh Renderer组件的Material属性

&#160; &#160; &#160; &#160;为MyBazooka脚本赋值，结果如下：

![MyBazooka赋值结果](http://img17.poco.cn/mypoco/myphoto/20150807/11/17800049220150807115712056.png)

##射击的控制

&#160; &#160; &#160; &#160;打开脚本MyBazooka.cs，在Update末尾加上如下代码：

<pre><code>
void Update () {
	···
	float fire = animator.GetFloat ("Fire");
	if (Input.GetButton ("Fire1") && fire &lt; 0.01 && aim > 0.99) {
		animator.SetFloat ("Fire", 1);
		if (bullet != null && spawn != null) {
			GameObject newBullet = Instantiate (bullet, 
				spawn.transform.position,
				Quaternion.Euler (0, 0, 0)) as GameObject;
			Rigidbody rb = newBullet.GetComponent&lt;Rigidbody> ();
			if (rb != null) {
				rb.velocity = spawn.transform.TransformDirection (
					Vector3.forward * 20);
			}
		}
	} else {
		animator.SetFloat ("Fire", 0, 0.1f, Time.deltaTime);
	}
}
</code></pre>

&#160; &#160; &#160; &#160;火箭炮发射后由于反作用力会移一段距离,修改OnAnimatorIK中Base Lyaer的处理代码

<pre><code>
if (layerIndex == 0) {
	if (targetA != null) {
		Vector3 target = targetA.transform.position;
		target.y = target.y + 0.2f * (target - 
			animator.rootPosition).magnitude;
		animator.SetLookAtPosition (target);
		animator.SetLookAtWeight (aim, 0.5f, 0.5f, 0.0f, 0.5f);
		if (bazoo != null) {
			float fire = animator.GetFloat ("Fire");
			Vector3 pos = new Vector3 (0.195f, -0.0557f, -0.155f);
			Vector3 scale = new Vector3 (0.2f, 0.8f, 0.2f);

			pos.x -= fire * 0.2f;
			scale = scale * aim;
			bazoo.transform.localScale = scale;
			bazoo.transform.localPosition = pos;
		}
	}
}
</code></pre>

&#160; &#160; &#160; &#160;运行游戏，击中Teddy后，Teddy倒地毙命，然后自动复活：

##实际运行效果

<script type="text/javascript" src="http://cdn.bootcss.com/jquery/2.1.4/jquery.min.js"></script>
<script type="text/javascript">
$(document).ready(function(){
	$("iframe").toggle();
  $(".btn1").click(function(){
  $("iframe").toggle();
  });
});
</script>
<iframe style="overflow: hidden;" src="http://acidg.pub/MyMecanim/MyBazooka/MyBazooka.html" width="630px" height="420px" scrolling="no"  frameborder="0"></iframe>
<button class="btn1">运行UnityWebPlayer版本</button>

&#160;运行[WebGL版本](http://acidg.pub/MyMecanim/MyBazooka2/index.html)