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
		GetComponent<Rigidbody> ().velocity = movement;
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
GetComponent<Rigidbody> ().velocity = movement * speed;
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
		//GetComponent<Rigidbody> ().velocity = movement * speed;
		Rigidbody rb = GetComponent<Rigidbody> ();
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
