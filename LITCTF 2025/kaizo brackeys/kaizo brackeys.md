# kaizo brackeys

Category: `rev`

Description:

> only real ones copied those amazing tutorials \nNote: the flag matches this regex: ^LITCTF\{[A-Z]+(?:_[A-Z]+)*\}$


---

For this flag, we are given the game files for a game called Kaizo Brackeys. Going through the directorys, we can see that there is a file called UnityCrashHandler.exe. This tells us that this game was made with Unity. My first idea was to just run the Kaizo Brackeys.exe file. Running it brings up a menu where you can start the game.
<img src="../images/kaizo_brackeys_menu.png" alt="Kaizo Brackeys Menu" width="400">

Upon starting the game, you are placed into a game where you must avoid obstacles in order to reach the end. 
<img src="../images/kaizo_brackeys_game.png" alt="Kaizo Brackeys Menu" width="400">

The first level is completable by going to the right. Once a level has been beaten, this screen comes up.
<img src="../images/kaizo_brackeys_complete.png" alt="Kaizo Brackeys Menu" width="400">

We are then brought to the next level, which is impractical to beat without cheats. Doing all of this helps us understand how this game works. If you reach the end of a level, you are brought to the next level. With this in mind, it is viable (*but not true, more on that later) to believe that once we beat the game, we get the flag.

Now to the actual rev part of the problem. We need some way to either let us see what you see when you win, or we need to to make it so getting past levels is a lot easier. A quick google search will tell us that there is no point looking/editing the actual .exe file, because that file only contains information about Unity's compiler. With another google search, we can find out that scripts that the .exe file reads can be found in the `_data/Managed/Assembly-CSharp.dll file`. To read dll files, I like using a neat tool called DnSpy (http://dnspy.org/). 

To get to the scripts, open up DnSpy and go to `File` >> `Open` and then select the `Assembly-CSharp.dll` in `kaizobrackeys-export\kaizobrackeys-export\x86-64\Kaizo Brackeys_Data\Managed`. Once you open this file up, open up the file tree for `Assembly-CSharp/Assembly-CSharp.dll`. You should see a bunch of files like "PE", "Type References", and "References". If you want to know more about what each of these files hold, google.com. For our purposes, we want to open the "_" folder. In there, we can see a whole bunch of scripts that were written specifically for the game. None of these scripts are very long, so I highly suggest you look through all of them and see how this game runs. For the purpose of beating the game, there are a couple names that stand out.

Take a look at Credits, Level Complete, and Player Movement.

In the Credits script, it seems just to be code for a button, and does not have much use to us.
In the Level Complete scipt, it simply tells Unity to go to the next scene. The game will crash if you try and load a scene that doesn't exist, so this is for the best for now.
The Player Movement script, we can see how the player input is recieved and used. This is the script that I edited first. Instead of moving foreward at a constantly increasing speed, how about we change the script to allow for wasd movement and a key that moves the player up? This can be done either through googling commands or chatGPT. To edit these scripts, simply right click the Player Movement script and click `Edit Class`. Once you are done editing the file, hit the `compile` button on the bottom right. If you have any issues compiling, try to edit each function and class seperately. 

This is the script I used:

```
using System;
using UnityEngine;

// Token: 0x02000009 RID: 9
public class PlayerMovement : MonoBehaviour
{
	// Token: 0x06000011 RID: 17 RVA: 0x0000327E File Offset: 0x0000147E
	private void Start()
	{
	}

	// Token: 0x06000012 RID: 18 RVA: 0x00003E0C File Offset: 0x0000200C
	private void FixedUpdate()
	{
		if (Input.GetKey("w"))
		{
			this.rb.AddForce(0f, 0f, this.forwardForce * Time.deltaTime);
		}
		if (Input.GetKey("s"))
		{
			this.rb.AddForce(0f, 0f, this.forwardForce * -1f * Time.deltaTime);
		}
		if (Input.GetKey("d"))
		{
			this.rb.AddForce(this.sidewaysForce * Time.deltaTime, 0f, 0f, ForceMode.VelocityChange);
		}
		if (Input.GetKey("a"))
		{
			this.rb.AddForce(-this.sidewaysForce * Time.deltaTime, 0f, 0f, ForceMode.VelocityChange);
		}
		if (Input.GetKey("q"))
		{
			if (!this.wHover)
			{
				this.wHover = true;
				this.wHoverBaseY = this.rb.position.y;
				Vector3 velocity = this.rb.velocity;
				velocity.y = 0f;
				this.rb.velocity = velocity;
				this.rb.useGravity = false;
			}
			float target = this.wHoverBaseY + this.wHoverHeight;
			Vector3 position = this.rb.position;
			position.y = Mathf.MoveTowards(position.y, target, this.wHoverSnapSpeed * Time.fixedDeltaTime);
			this.rb.MovePosition(position);
		}
		else if (this.wHover)
		{
			this.wHover = false;
			this.rb.useGravity = true;
		}
		if (this.rb.position.y < -1f)
		{
			Object.FindAnyObjectByType<GameManager>().EndGame();
		}
	}

	// Token: 0x06000013 RID: 19 RVA: 0x00003280 File Offset: 0x00001480
	public PlayerMovement()
	{
		this.forwardForce = 4000f;
		this.sidewaysForce = 100f;
		base..ctor();
	}

	// Token: 0x04000008 RID: 8
	public Rigidbody rb;

	// Token: 0x04000009 RID: 9
	public float forwardForce;

	// Token: 0x0400000A RID: 10
	public float sidewaysForce;

	// Token: 0x0400000B RID: 11
	private bool wHover;

	// Token: 0x0400000C RID: 12
	private float wHoverBaseY;

	// Token: 0x0400000D RID: 13
	private float wHoverHeight = 3f;

	// Token: 0x0400000E RID: 14
	private float wHoverSnapSpeed = 10f;
}
```

This script lets you move around with wasd controls and lets you move up with q. Once you have compiled your script with no errors, save your file by going to `File` >> `Save Module...`. 

With that, you can now run the kaizo_brackeys.exe file again and breeze through all of the levels until you get to this screen: 
<img src="../images/kaizo_brackeys_credits.png" alt="Kaizo Brackeys Menu" width="400">

By pressing the exit button, all that happens is the game closes. It seems our previous assumption of the flag being in the credits was wrong. Our best bet now is to find more information about the game, and a good way of doing that is through a software called Asset Scraper.