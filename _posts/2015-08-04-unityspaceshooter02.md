---
layout: post
title: "Space Shooter开发全过程记录02"
description: "Space Shooter开发全过程记录02"
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

在Inspector中设置Bolt的Speed为20

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

&#160; &#160; &#160; &#160;2. 调整Boundary的X、Z值为0和5，使Boundary位于游戏区中心

&#160; &#160; &#160; &#160;3. 设置Boundary的Scale属性值为(15,1,20)

&#160; &#160; &#160; &#160;4. 设置Boundary的Box Collider组件为触发器。由于不需要显示Boundary，移除Mesh Renderer组件

&#160; &#160; &#160; &#160;5. 在Project视图中Assets/_Scripts的目录下新建一个脚本DestroyByBoundary.cs，并将其拖到Boundary对象上。

&#160; &#160; &#160; &#160;6. 打开脚本DestroyByBoundary.cs脚本，添加如下事件响应函数：

<pre><code>
void OnTriggerExit (Collider other) {
		Destroy (other.gameObject);
	}
</code></pre>

&#160; &#160; &#160; &#160;7. 运行游戏，在Hierarchy视图中可以发现，当子弹飞出场景后，自动消除了

###2.3.添加小行星(Asteroid)

要实现的目标：

1. 小行星随机产生，且以随机的角度旋转
2. 当飞船发射子弹击中小行星，小行星会爆炸并销毁
3. 若飞船碰撞上小行星，则飞船爆炸，游戏结束

####2.3.1.创建小行星对象

&#160; &#160; &#160; &#160;1. 在Hierarchy视图中新建一个空的游戏对象Asteroid，重置其Transform组件。设置其Position为(0,0,10)。添加Rigidbody组件，取消勾选Use Gravity选项框。添加Capsule Collider组件，勾选Is Trigger选项。

&#160; &#160; &#160; &#160;2. 从Project视图中Assets/Models的目录下拖动模型prop_asteroid_01到Asteroid上，是之成为Asteroid的子对象。

&#160; &#160; &#160; &#160;3. 为了使碰撞体(Capsule Collider)更接近模型的几何体形状，选中Asteroid后在Inspector视图中调整碰撞体的属性值。设置Radius的值为0.5，Height的值为1.6，Direction的值为Z轴

####2.3.2.添加小行星随机旋转的功能

&#160; &#160; &#160; &#160;1. 在Project视图中Assets/_Scripts目录下新建一个脚本RandomRotator.cs，并将其绑定到Asteroid对象上。

&#160; &#160; &#160; &#160;2. 在脚本中添加一个表示小行星旋转系数的变量tumble，再重载Start函数，为刚体组件的角速度赋上随机值。

<pre><code>
	public float tumble = 10.0f;
	void Start () {
		GetComponent&lt;Rigidbody> ().angularVelocity = 
			Random.insideUnitSphere * tumble;
	}
</code></pre>

angularVelocity表示刚体的角速度，在3D图形学中，角速度描述的是做圆周运动的物体单位时间转过的角度。Random.insideUnitSphere表示单位长度半径球体内的一个随机点(向量)，这个随机点与旋转系数tumble的乘积描述了在半径长度为tumble的球体内的随机点，由此就可以实现刚体以一个随机的角速度旋转。

&#160; &#160; &#160; &#160;3. 运行游戏，发现小行星在飞船前方随机旋转。

&#160; &#160; &#160; &#160;但仔细观察会发现，旋转越来越慢，这是因为刚体设置了角阻力(angular drag)，为了使小行星不受影响，把Asteroid对象Rigidbody中Angular Drag的值设置为0

####2.3.3.添加控制射击小行星的功能

&#160; &#160; &#160; &#160;1. 在Project试图中Assets/_Scripts目录下新建脚本DestroyByContact.cs,并绑定到Asteroid上

&#160; &#160; &#160; &#160;2. 在脚本中加入事件函数OnTriggerEnter，并在函数中销毁Asteroid对象(即下面脚本追踪的gameObject)和与Asteroid发生碰撞的对象(即other.gameObejct)

<pre><code>
void OnTriggerEnter (Collider other) {
		Destroy (other.gameObject);
		Destroy (gameObject);
	}
</code></pre>

&#160; &#160; &#160; &#160;3. 运行游戏，飞船没有射击，小行星也自动销毁了，原因是小行星和Boundary发生了碰撞

&#160; &#160; &#160; &#160;4. 在Hierarchy试图选中Boundary对象，在Inspector中指定其Tag为Boundary(若该Tag不存在则自行创建)

&#160; &#160; &#160; &#160;5. 在DestroyByContact.cs的碰撞事件处理函数中加入以下代码

<pre><code>
void OnTriggerEnter (Collider other){
		if (other.tag == "Boundary")
			return;
		···
	}
</code></pre>

&#160; &#160; &#160; &#160;6. 运行游戏，发现击中小行星后，子弹和小行星都销毁；飞船与小行星碰撞时，两者也都销毁

####2.3.4.添加小行星爆炸时的效果

&#160; &#160; &#160; &#160;1. 打开脚本DestroyByContact.cs，添加量过变量

<pre><code>
	public GameObject explosion;
	public GameObject playerExplosion;
</code></pre>

1. explosion表示小行星碰撞后爆炸时的粒子对象
2. playerExplosion表示飞船与小行星碰撞飞船爆炸时的粒子对象

&#160; &#160; &#160; &#160;2. 在碰撞事件函数中加入以下代码

<pre><code>
	void OnTriggerEnter (Collider other){
		···
		Instantiate (explosion, transform.position, transform.rotation);
		if (other.tag == "Player") {
			Instantiate(playerExplosion, other.transform.position, other.transform.rotation);
		···
		}
</code></pre>

&#160; &#160; &#160; &#160;3. 在Hierarchy试图中选择对象Player，指定其Tag为Player

&#160; &#160; &#160; &#160;4. 从Project视图Assets/Prefabs/VFX的目录下，拖动explosion_asteroid到变量explosion上，拖动explosion_Player到变量player Explosion上，完成赋值。

&#160; &#160; &#160; &#160;5. 运行游戏，可以看到子弹击中小行星爆炸效果和飞船撞击小行星的爆炸效果。

####2.3.5.添加控制小行星运动的功能

&#160; &#160; &#160; &#160;1. 将Project视图Assets/_Scripts下的脚本Mover.cs拖动到Asteroid对象上，在Inspector视图中设定Speed的值为-5。

&#160; &#160; &#160; &#160;2. 运行游戏，小行星向着飞船飞过来，飞出边界时自动销毁。

####2.3.6.添加小行星随机产生的逻辑功能

&#160; &#160; &#160; &#160;1. 把Asteroid对象从Hierarchy视图拖动到Project视图的Assets/Prefabs目录下，生成预制体Asteroid，然后从Hierarchy视图中将其删除。

&#160; &#160; &#160; &#160;2. 在Project视图创建一个空的游戏对象GameController，重置其Transform组件，指定其Tag为GameController(若不存在该名称Tag则自行创建)

&#160; &#160; &#160; &#160;3. 在Assets/_Scripts目录下新建GameController.cs脚本，拖动到GameController上

&#160; &#160; &#160; &#160;4. 在脚本中输入以下代码

<pre><code>
	public GameObject hazard;
	public Vector3 spawnValues;
	private Vector3 spawnPosition = Vector3.zero;
	private Quaternion spawnRotation;
	void SpawnWaves() {
		spawnPosition.x = Random.Range (-spawnValues.x, spawnValues.x);
		spawnPosition.z = spawnValues.z;
		spawnRotation = Quaternion.identity;
		Instantiate (hazard, spawnPosition, spawnRotation);
	}
	
	void Start () {
		SpawnWaves ();
	}
</code></pre>

在上面的代码中

1. hazard对象是准备实例化的障碍物对象，这里指Asteroid预制体
2. spawPosition表示实例化hazard对象的位置
3. Quaternion.identity表示无旋转
4. SpawnWaves函数用于在X轴方向上随机实例化hazard对象

&#160; &#160; &#160; &#160;5. 在Inspector视图中为各个变量赋值

1. 将Asteroid预制体从Project拖动到Hazard变量上
2. Spawn Values设置为(6,0,14.5)（也是Asteroid在最右上角的情况）

####2.3.7.添加小行星批量产生功能

&#160; &#160; &#160; &#160;1. 打开脚本GameController.cs，加入变量hazardCount，用来表示小行星的数量

<pre><code>public int hazardCount;</code></pre>

&#160; &#160; &#160; &#160;2. 修改函数SpawnWaves如下

<pre><code>
for (int i = 0; i < hazardCount; ++i) {
	spawnPosition.x = Random.Range (-spawnValues.x, spawnValues.x);
	spawnPosition.z = spawnValues.z;
	spawnRotation = Quaternion.identity;
	Instantiate (hazard, spawnPosition, spawnRotation);
	}
</code></pre>

&#160; &#160; &#160; &#160;3. 在Inspector视图中设定hazardCount的值为10

&#160; &#160; &#160; &#160;4. 运行游戏，发现同时生成的小行星很多，而且数量近，有些小行星相互碰撞直接销毁了

&#160; &#160; &#160; &#160;要解决这个问题，可以是每个小行星生成后等待一段时间，Unity中提供协程类WaitForSeconds可以实现这样的功能

&#160; &#160; &#160; &#160;5. 在GameController.cs中添加一个变量spawnWait用于表示每次生成小行星对象后延迟的时间，单位为妙

<pre><code>public float spawnWait;</code></pre>

&#160; &#160; &#160; &#160;6. 修改SpawnWaves返回值类型为IEmunerator，然后用yield return语句返回WaitForSeconds的实力，修改代码如下

<pre><code>
	IEnumerator SpawnWaves() {
		for (int i = 0; i < hazardCount; ++i) {
			···
			yield return new WaitForSeconds(spawnWait);
		}
	}
</code></pre>

&#160; &#160; &#160; &#160;7. 修改Start中SpawnWaves的调用方法

<pre><code>
	void Start () {
		StartCoroutine (SpawnWaves ());
	}
</code></pre>

&#160; &#160; &#160; &#160;8. 在Inspector中设定spawnWait的值为0.5

&#160; &#160; &#160; &#160;9. 由于不希望一开始就立刻产生小行星，而是等待一段时间，在GameController.cs中再加入一个变量startWait，用来表示开始生成小行星对象前等待的时间

<pre><code>public float startWait = 1.0f;</code></pre>

&#160; &#160; &#160; &#160;10. 修改函数SpawnWaves，在for循环之前加上下列语句，表示等待startWait秒

<pre><code>yield return new WaitForSeconds (startWait);</code></pre>

&#160; &#160; &#160; &#160;11. 在真实游戏中，小行星应该不断产生，直到游戏结束，在GameController.cs中再加入一个变量waveWait，表示两批小行星阵列间的时间间隔

<pre><code>public float waveWait = 2.0f;</code></pre>

&#160; &#160; &#160; &#160;修改SpawnWaves的代码，加入无限循环功能

<pre><code>
	IEnumerator SpawnWaves() {
		yield return new WaitForSeconds (startWait);
		while (true) {
			for (int i = 0; i < hazardCount; ++i) {
				···
			}
		}
		yield return new WaitForSeconds(waveWait);
	}
</code></pre>

&#160; &#160; &#160; &#160;12. 运行游戏，发现不断生成小行星，但随着游戏进行，小行星的爆炸粒子效果一直没有销毁

&#160; &#160; &#160; &#160;13. 在Project视图的Assets/_Scripts目录下新建DestroyByTime.cs脚本，并绑定到粒子效果explosion_asteroid和explosion_player上，其代码为：

<pre><code>
public class DestroyByTime : MonoBehaviour {
	public float lifetime = 2.0f;
	void Start () {
		Destroy (gameObject, lifetime);
	}
}
</code></pre>

&#160; &#160; &#160; &#160;14. 运行游戏，发现爆炸粒子效果过一会后自行销毁
