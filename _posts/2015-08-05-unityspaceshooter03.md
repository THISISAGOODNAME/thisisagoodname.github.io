---
layout: post
title: "Space Shooter开发全过程记录03"
description: "Space Shooter开发全过程记录03"
category: Unity3D
tags: [Unity3D,C#]
---

* Table of Contents
{:toc}

&#160; &#160; &#160; &#160;为了增强游戏的“代入感”，绝大部分游戏都会添加合适的音频来增强效果。

&#160; &#160; &#160; &#160;之后，为了提升游戏性，加入计分功能，以及失败后重新开始功能。


<!-- more -->

##3.添加音频

###3.1.添加碰撞爆炸音频

&#160; &#160; &#160; &#160;1. 展开Assets下的目录Audio和Prefabs/VFX/Explosions

&#160; &#160; &#160; &#160;2. 将音频文件explosion_asteroid拖动到预制体explosion_asteroid上

![音频文件拖到预制体上](http://img17.poco.cn/mypoco/myphoto/20150805/09/17800049220150805095735057.png)

&#160; &#160; &#160; &#160;3. 选中预制体对象explosion_asteroid，在Inspector视图中确保Audio Source组件的Play On Awake勾选框处于选中状态，此选项表明了音频文件在唤醒时会自动播放。

&#160; &#160; &#160; &#160;4. 仿照上述操作，吧音频文件explosion_player绑定到预制体explosion_player上

###3.2.添加飞船射击音频

&#160; &#160; &#160; &#160;1. 从Project视图中拖动音频weapon_player到Hierarchy视图的Player对象上，音频文件不能在唤醒时自动播放，否则会出现未发射子弹就发出子弹射击的音效，取消勾选Play On Awake

&#160; &#160; &#160; &#160;2. 编辑脚本PlayerController.cs脚本，在原有代码后加上如下代码：

<pre><code>
	void Update () {
		if (Input.GetButton ("Fire1") && Time.time > nextFire) {
			···
			GetComponent&lt;AudioSource>().Play();
		}
	}
</code></pre>

###3.3.添加背景音效

&#160; &#160; &#160; &#160;1. 从Project视图的Audio目录下拖动音频music_background到GameController上，勾选Play On Awake和Loop选项

![Play On Awake和Loop选项](http://img17.poco.cn/mypoco/myphoto/20150805/10/17800049220150805101216070.png)

&#160; &#160; &#160; &#160;运行游戏，射击音效、爆炸音效和背景音乐都已存在

##4.添加计分文本

###4.1.添加计分Text组件

&#160; &#160; &#160; &#160;1. 点击菜单项GameObject->UI->Text，会添加一个canvas父对象和Text子对象，还有一个EventSystem对象。重命名Text为Score Text对象，在Text组件的Text属性中输入“得分”

![Text Score](http://img17.poco.cn/mypoco/myphoto/20150805/10/17800049220150805101710012.png)

&#160; &#160; &#160; &#160;2. 选中canvas对象，取消勾选Receives Events选项框，因为不需要在此文本中响应事件，同时在Hierarchy视图中删除EventSystem对象

&#160; &#160; &#160; &#160;3. 选中ScoreText，希望其在左上角编辑Rect Transform组件的Anchor属性，打开Anchor Presets对话框，按住【shift】键设置对齐点，按住【alt】键设置对象位置。在Anchor Presets对话框中选择top和left

![Anchor Presets](http://img17.poco.cn/mypoco/myphoto/20150805/10/17800049220150805102024025.png)

&#160; &#160; &#160; &#160;设置Width和Height为(100,30)

&#160; &#160; &#160; &#160;4. 为了使文本与游戏边界有一定距离，调整Rect Transform组件的Pos X和Pos Y值为(10,-10)

###4.2.添加计分功能

&#160; &#160; &#160; &#160;1. 打开GameController.cs脚本文件，添加变量scoreText和score

<pre><code>
	public Text scoreText;
	private int score;
</code></pre>

&#160; &#160; &#160; &#160;2. Text在命名空间UnityEngine.UI中被声明，要使用，必须引用，在脚本开始加上

<pre><code>using UnityEngine.UI;</code></pre>

&#160; &#160; &#160; &#160;3. 加入函数AddScore和UpdateScore更新Text，函数AddScore的访问权限设置为了public，可以在DestroyByContact.cs中调用

<pre><code>
	public void AddScore(int newScoreValue) {
		score += newScoreValue;
		UpdateScore ();
	}

	void UpdateScore() {
		scoreText.text = "得分：" + score;
	}
</code></pre>

&#160; &#160; &#160; &#160;4. 在Start中初始化得分和Text组件

<pre><code>
	void Start () {
		score = 0;
		UpdateScore ();
		StartCoroutine (SpawnWaves ());
	}
</code></pre>

&#160; &#160; &#160; &#160;5. 打开脚本DestroyByContact.cs加入以下变量

<pre><code>
	public int scoreValue;
	private GameController gameController;
</code></pre>

&#160; &#160; &#160; &#160;6. 在小行星碰撞事件函数OnTriggerEnter中加入分值更新语句

<pre><code>gameController.AddScore (scoreValue);</code></pre>

&#160; &#160; &#160; &#160;7. 在Start中初始化变量gameObject

<pre><code>
	void Start () {
		GameObject go = GameObject.FindWithTag("GameController");
		if (go != null)
			gameController = go.GetComponent&lt;GameController> ();
		else
			Debug.Log ("找不到tag为GameController的对象");
		if (gameController == null)
			Debug.Log ("找不到脚本GameController.cs");
	}
</code></pre>

&#160; &#160; &#160; &#160;8. 在GameController对象中将scoreText属性赋值为ScoreText对象，在Asteroid预制体中将scoreValue设为10

##5.游戏结束与重新开始

###5.1.添加游戏结束的Text组件

&#160; &#160; &#160; &#160;1. 在Hierarchy视图中选中对象ScoreText对象，按【Ctrl+D】组合键复制对象，重命名为GameOverText，在Text属性中输入【游戏结束】

&#160; &#160; &#160; &#160;2. 通过Anchor Preset对话框设置Rect Transform的对其点为center和middle

&#160; &#160; &#160; &#160;3. 设置Text组件的Alignment属性为center align和middle align，设置Pos X和Pos Y的值为(-50,50)

###5.2.添加游戏结束功能

&#160; &#160; &#160; &#160;1. 打开GameController.cs，加入如下变量

<pre><code>
	public Text gameOverText;
	private bool gameOver;
</code></pre>

&#160; &#160; &#160; &#160;2. 在Start中初始化变量

<pre><code>
	gameOverText.text = "";
	gameOver = false;
</code></pre>

&#160; &#160; &#160; &#160;3. 在脚本中加入一个函数，用来表示游戏结束

<pre><code>
	public void GameOver() {
		gameOver = true;
		gameOverText.text = "游戏结束";
	}
</code></pre>

&#160; &#160; &#160; &#160;4. 在函数SpawnWaves的开头，加上判断游戏是否已结束，结束便不再产生小行星

<pre><code>
	IEnumerator SpawnWaves() {
		yield return new WaitForSeconds (startWait);
		while (true) {
			if (gameOver) {
				restart = true;
				break;
			}
			···
		}
	}
</code></pre>

&#160; &#160; &#160; &#160;5. 将GameOverText对象拖动到变量gameOverText上

&#160; &#160; &#160; &#160;6. 修改脚本DestroyByContact.cs的OnTriggerEnter函数，加上如下代码

<pre><code>
		···
		if (other.tag == "Player") {
			···
			gameController.GameOver();
		}
		···
</code></pre>

&#160; &#160; &#160; &#160;7. 现在运行游戏，飞船与小行星碰撞后，游戏结束

###5.3.添加重新开始的Text组件

&#160; &#160; &#160; &#160;1. 复制Score Text对象，重命名为Restart Text，然后在其Text属性中输入“按【R】键重新开始”

&#160; &#160; &#160; &#160;2. 设计Rect Transform的Anchor为top和right，设置Alignment的属性为Right Align

&#160; &#160; &#160; &#160;3. 设置Rect Transform组件的Pos X为-110，Pos Y为-10

###5.4.添加重新开始游戏功能

&#160; &#160; &#160; &#160;1. 打开GameController.cs，加入如下变量

<pre><code>
	public Text restartText;
	private bool restart;
</code></pre>

&#160; &#160; &#160; &#160;2. 在函数Start中赋初值

<pre><code>
	restartText.text = "";
	restart = false;
</code></pre>

&#160; &#160; &#160; &#160;3. 修改函数SpwanWaves

<pre><code>
	IEnumerator SpawnWaves() {
		yield return new WaitForSeconds (startWait);
		while (true) {
			if (gameOver) {
				restartText.text = "按R键重新开始";
				restart = true;
				break;
			}
			···
		}
	}
</code></pre>

&#160; &#160; &#160; &#160;4. 修改Update函数，可以按R键重新开始

<pre><code>
	void Update () {
		if (restart) {
			if (Input.GetKeyDown(KeyCode.R))
				Application.LoadLevel(Application.loadedLevel);
		}
	}
</code></pre>

&#160; &#160; &#160; &#160;5. 在Inspector试图中为restartText，方法与前文类型

&#160; &#160; &#160; &#160;6. 运行游戏，失败后可以看到右上角的提示性语言

##6.总结

&#160; &#160; &#160; &#160;对于初学者，亲自参与一次实例开发，然后在学习文档性的教程，可以事半功倍。直接学习文档性质的教程经常迷惑，这些功能都有神么用。而且突破代码恐惧对于新人来说也非常重要。

&#160; &#160; &#160; &#160;这个实例从资源准备结束开始，一点一点完善了一个简单的STG，希望对于各位新人的继续学习能有所帮助。