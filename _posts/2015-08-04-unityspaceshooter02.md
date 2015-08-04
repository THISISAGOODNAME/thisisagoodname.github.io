---
layout: post
title: "Space Shooter开发全过程记录02"
description: "Space Shooter开发全过程记录"
category: Unity3D
tags: [Unity3D,C#]
---

* Table of Contents
{:toc}

&#160; &#160; &#160; &#160;上一节准备了一些场景素材，本节要实现键盘控制飞船行动，射击和敌人(障碍物)。

<!-- more -->

##2.编写脚本代码

###2.1.键盘控制飞船移动

&#160; &#160; &#160; &#160;创建脚本文件夹。在Project视图中选中Assets文件夹，然后Create->Folder，创建文件夹"_Scripts",之后的所有脚本文件都放在该文件夹中。

&#160; &#160; &#160; &#160;在_Scripts文件夹中新建一个C#脚本PlayerController.cs，输入下列代码

<pre><code>
void FixedUpdate() {
		float moveHorizontal = Input.GetAxis("Horizontal");
		float moveVertical = Input.GetAxis("Vertical");

		Vector3 movement = new Vector3 (moveHorizontal, 0.0f, moveVertical);
		GetComponent&lt;Rigidbody> ().velocity = movement;
		}
	}
</code></pre>

选中PlayerController.cs，拖动到Hierarchy视图的Player对象上，完成绑定。

&#160; &#160; &#160; &#160;运行游戏，方向键可以控制飞船运动，但有三个不足

1. 飞船移动速度过慢
2. 飞船运动没有范围限制，可以飞出Viewport
3. 左右移动飞船，没有倾斜效果

接下来依次解决三个问题

####2.1.1飞船移动速度慢

&#160; &#160; &#160; &#160;在PlayerController类中添加一个成员变量Speed，为了方便，设置初值为5.0f 。

<pre><code>
public class PlayerController : MonoBehaviour {
	public float speed = 5.0f;
	···
}
</code></pre>

&#160; &#160; &#160; &#160;修改FixedUpdate函数，将刚体的velocity属性设置为：

<pre><code>
GetComponent&lt;Rigidbody> ().velocity = movement * speed;
</code></pre>

再运行游戏，可以发现飞船运动明显变快了。

####2.1.2.添加运动范围限制

&#160; &#160; &#160; &#160;1. 在PlayerController.cs中，在PlayerController类前面声明Boundary类，用于管理飞船活动的边界值。

<pre><code>
public class Boundary {
	public float xMin, xMax, zMin, zMax;
}

public class PlayerController : MonoBehaviour {
	···
}
</code></pre>

&#160; &#160; &#160; &#160;2. 在PlayerController类中添加一个Boundary类的实例，访问权限为public：

<pre><code>public Boundary boundary;</code></pre>

要将物体位置固定在一个范围内，可以使用Unity提供的Mathf.Clamp函数来实现：

<pre><code>static float Clamp(float value,float min,float max)</code></pre>

若value小于min则返回min；若value大于max，则返回max。

&#160; &#160; &#160; &#160;然后更新FiexdUpdate函数，如下：

<pre><code>
void FixedUpdate() {
		float moveHorizontal = Input.GetAxis("Horizontal");
		float moveVertical = Input.GetAxis("Vertical");

		Vector3 movement = new Vector3 (moveHorizontal, 0.0f, moveVertical);
		//GetComponent&lt;Rigidbody> ().velocity = movement * speed;
		Rigidbody rb = GetComponent&lt;Rigidbody> ();
		if (rb != null) {
			rb.velocity = movement * speed;
			rb.position = new Vector3(
				Mathf.Clamp(rb.position.x, boundary.xMin, boundary.xMax), 
				0.0f, 
				Mathf.Clamp(rb.position.z, boundary.zMin, boundary.zMax)
				);
		}
	}
</code></pre>

&#160; &#160; &#160; &#160;然后为Boundary类添加可序列化的属性，入下所示。

<pre><code>
[System.Serializable]
public class Boundary {
	···
}
</code></pre>

然后返回Inspector视图，可以看到Player的Boundary现在可以修改了，设置Boundary的值为(-6.0f,6.0f,-4.0f,14.0f).

![Boundary设置](http://img17.poco.cn/mypoco/myphoto/20150804/23/17800049220150804231934019.png)

此时再运行游戏，飞船将不会飞出屏幕范围。

####2.1.3添加移动旋转效果

&#160; &#160; &#160; &#160;在PlayerController类中添加一个角度倾斜系数，默认设置为4.0f:

<pre><code>public float tilt = 4.0f</code></pre>

&#160; &#160; &#160; &#160;修改FixedUpdate函数，添加如下语句

<pre><code>
void FixedUpdate() {
		···
		if (rb != null) {
			···
			rb.rotation = Quaternion.Euler(0.0f, 0.0f, rb.velocity.x * -tilt);
		}
	}
</code></pre>

运行游戏，左右移动时，飞船出现一定角度的倾斜。

###2.2.实现射击行为

####2.2.1.创建电光子弹

&#160; &#160; &#160; &#160;1. 创建一个空的游戏对象，命名为Bolt，重置Transform组件。为Bolt添加一个Rigidbody组件，并取消勾选Use Gravity。

&#160; &#160; &#160; &#160;2. 新建一个Quad对象，重命名为VFX，将其设置为Bolt的子对象，重置其Transform组件。设置Transform的Rotation属性为(90,0,0),并移除Mesh Collider组件。

&#160; &#160; &#160; &#160;3. 将Project视图Assets/Materials目录下的材质对象fx_bolt_orange拖动到VFX上。

&#160; &#160; &#160; &#160;4. 为Bolt添加一个Capsule Collider(胶囊碰撞体)组件，勾选Is Trigger选项，将其设置为一个触发器。碰撞体要挂在Bolt而不是子对象VFX上，否则可能无法完全销毁Bolt。之后设置Capsule Collider的Direction属性为Z-Axis，并设置Radius为0.04，Height为0.65.

&#160; &#160; &#160; &#160;5. 在Project视图Assets/_Scripts目录下新建一个脚本Mover.cs，如下

<pre><code>
using UnityEngine;
using System.Collections;

public class Mover : MonoBehaviour {

	public float speed;

	// Use this for initialization
	void Start () {
		GetComponent&lt;Rigidbody>().velocity = transform.forward * speed;
	}

}
</code></pre>

&#160; &#160; &#160; &#160;6. 在Project视图的Assets目录下新建一个子目录_Prefabs，用于存储预制体。将对象Bolt从Hierarchy视图拖动到_Prefabs目录下，创建一个新的预制体Bolt。之后可以将Hierarchy视图中的Bolt删除。

&#160; &#160; &#160; &#160;7. 运行游戏，将Bolt预制体拖动到Hierarchy视图中，可以看到一个电光子弹飞速向前运动。

至此，电光子弹预制体基本制作完成，还剩两个问题

1. 电光子弹应该被键盘或者鼠标控制发射
2. 随着发射的子弹越来越多，不会自动销毁。在真实游戏中，数量巨大的子弹必定会消耗非常多的系统资源，影响游戏性能

#####2.2.2.脚本控制发射子弹

&#160; &#160; &#160; &#160;1. 为Player新建一个空的子对象Shot Spawn，设置其Position的值为(0,0,0.7)，将在此位置发射子弹

&#160; &#160; &#160; &#160;2. 双击脚本PlayerController.cs进行编辑。为了实现Fire1触发后即刻实例化Bolt预制体，需要：

1. 存储传入Bolt变量，作为Instantiate的第一个参数
2. 存储发射器的位置，作为实例化Bolt的位置
3. 设置一定的发射频率，只有间隔了一定时间后才继续发射，否则当用户持续按Fire1的时候，会连续的发射子弹。

为PlayerController加入以下变量

<pre><code>
	public float fireRate = 0.5f;
	public GameObject shot;
	public Transform shotSpawn;
	public float nextFire = 0.0f;
</code></pre>

1. fireRate表示发射间隔，默认0.5s
2. shot表示Bolt预设体
3. shotSpwan是Transform类型，这样拖动GameObject类型变量到Inspector上时，会自动提取对象的Transform属性赋值给变量
4. nextFire表示下次射击的最早时间

之后修改PlayerController的Update方法：

<pre><code>
void Update () {
		if (Input.GetButton ("Fire1") && Time.time > nextFire) {
			nextFire = Time.time + fireRate;
			Instantiate(shot, shotSpawn.position, shotSpawn.rotation);
		}
	}
</code></pre>

&#160; &#160; &#160; &#160;3. 为变量赋值，在Hierarchy中选择Player对象，然后将Project中的Bolt预制体拖动到Inspector视图中对应的Bolt变量上；拖动Hierarchy视图中Player下的Shot Spawn子对象到Inspector上对应的shotSpawn变量上

&#160; &#160; &#160; &#160;4. 运行游戏，方向控制移动，鼠标左键或者左【Ctrl】键发射子弹。

####2.2.3.管理电光子弹的生命周期

&#160; &#160; &#160; &#160;1. 在Hierarchy视图中添加一个Cube对象，命名为Boundary，重置其Transform组件。

&#160; &#160; &#160; &#160;2. 调整Boundary的位置