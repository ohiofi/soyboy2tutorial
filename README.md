# Super Soy Boy, Part 2

Getting started again!

1. I want to make sure everyone is in the same place, so download and then extract **Starter2.zip**.
2. Open the **Game** scene, located in the **Scenes** folder. You should see that the project looks very similar to the previous version of Soy Boy, except there are several more building block items created for you in the **Resources/Prefabs** folder
3. Click **Edit\ProjectSettings\Tags &amp; Layers** to open the layers editor in the Inspector.
4. Use **User Layer 8** and **User Layer 9** to create two new layers called **Player** and **Hazards**.
5. Click **Edit\Project Settings\Physics2D** to open its Inspector. At the bottom you will see **Layer Collision Matrix**. Expand it if necessary.
    * **Uncheck** the boxes where **Hazards** intersects with **Hazards**.
    * **Check** the box where **Player** intersects with **Hazard** s.
    * **Uncheck** the box where **Default** intersects with **Hazards**.
6. In the **Game** scene, select **SoyBoy** and change the GameObject layer to the new **Player** physics layer.

 ![SoyBoy inspector](/pics2/sb21.png)

## Player Controller Code

1. 2D raycasting will be used to determine when you are touching the ground, jumping up against a wall, etc. so you will see new code associated with the raycast concept. Open **SoyBoyController** and add the following new public fields to control jumping.

```c#
public bool isJumping;
public float jumpSpeed = 8f;
private float rayCastLengthCheck = 0.005f;
private float width, height;
```

2. At the bottom of the **Awake()** method, add the following code. The additional 0.1f and 0.2f added to the end of the calculations will give a little buffer when raycasting later.

```c#
width = GetComponent<Collider2D>().bounds.extents.x + 0.1f;
height = GetComponent<Collider2D>().bounds.extents.y + 0.2f;
```

3. Let&#39;s get SoyBoy jumping. Two things need to be checked before he is allowed to jump; we need to make sure he is not already jumping, and a check to insure he is currently touching the ground. Add the following new method to the **SoyBoyController** script. The first check, sends a ray from the bottom of the sprite, while the other two are slightly to the left and right of center of the SoyBoy sprite.

```c#
public bool PlayerIsOnGround()
{
   bool groundCheck1 = Physics2D.Raycast (new Vector2 (transform.position.x, transform.position.y - height), -Vector2.up, rayCastLengthCheck);
   bool groundCheck2 = Physics2D.Raycast (new Vector2 (transform.position.x + (width - 0.2f), transform.position.y - height), -Vector2.up, rayCastLengthCheck);
   bool groundCheck3 = Physics2D.Raycast (new Vector2 (transform.position.x - (width - 0.2f), transform.position.y - height), -Vector2.up, rayCastLengthCheck);
   if (groundCheck1 || groundCheck2 || groundCheck3)
      return true;
   else
      return false;
}
```

4. At the end of your existing **Update()** method, add the following code.

```c#
// determine if player is touching the ground or not and make sure the player is not currently in a jump
if(PlayerIsOnGround() && isJumping == false)
{
   if(input.y > 0f)
      isJumping = true;
}
```

5. Before we add force to SoyBoy to allow him to jump, we need a way to control how high he should jump, based on how long the jump button is held. This will allow for some very accurate jumps over or around obstacles. Add the following new fields to the top of the class.

```c#
public float jumpDurationThreshold = 0.25f;
private float jumpDuration;
```

6. At the bottom of the **Update()** method, add the following. If the jump button is held longer than 0.25 seconds, then the jump is cancelled, meaning the player can only jump to a certain height.

```c#
if(jumpDuration > jumpDurationThreshold)
   input.y = 0f;
```

7. The next code goes at the bottom of **FixedUpdate()**  and converts the horizontal velocity to vertical velocity set to the jumpSpeed.

```c#
if(isJumping && jumpDuration < jumpDurationThreshold)
   rb.velocity = new Vector2(rb.velocity.x, jumpSpeed);
```

8. Enter the follow code to the **Update()** method, just below the sprite flipping code, and just above the **PlayerIsOnGround()** check. As long as the jump button is held down, **jumpDuration** is counted up using the time the previous frame took to complete. If jump is released, **jumpDuration** is set back to 0.

```c#
if(input.y >= 1f)
{
   jumpDuration += Time.deltaTime;
}
else
{
   isJumping = false;
   jumpDuration = 0f;
}
```

9. **Save** your code and run the game to try it out. Use the **space** key for jumping.

## Handling Air Control

When Soy Boy is in the air, you don&#39;t want him to handle as quickly as he normally does when he runs along the ground, so you&#39;ll change the **acceleration** variable value based on whether the player is in the air or not.

1. Locate the check to see if the player is moving horizontally in the FixedUpdate() method and change it to make sure the xVelocity is only set to 0 if the player is on the ground and not using the left or right controls.

```c#
if(PlayerIsOnGround() && input.x == 0)
```

2. Next, add a new variable to the top of the class to hold a value that will be used for air control. Notice the airAccel variable is half that of the running accel variable.

```c#
public float airAccel = 3f;
```

3. At the top of the **FixedUpdate()** method, replace the line var acceleration = accel; with the following code block. This code sets acceleration to 0f to start with, then looks to see if the player is on the ground or not and sets the variables accordingly.

```c#
var acceleration = 0f;
if(PlayerIsOnGround())
{
   acceleration = accel;
}
else
{
   acceleration = airAccel;
}
```

## Jumping Off Walls

1. First of all, you need to know if a wall is on the left or right of the player. This way, when the player jumps off the wall, you know to propel him away from the wall he is on. In the **SoyBoyController** script just below the **PlayerIsOnGround()** method, add the new method below.

```c#
public bool IsWallToLeftOrRight()
{
	bool wallOnLeft = Physics2D.Raycast (new Vector2 (transform.position.x - width, transform.position.y), -Vector2.right, rayCastLengthCheck);
	bool wallOnRight = Physics2D.Raycast (new Vector2 (transform.position.x + width, transform.position.y), Vector2.right, rayCastLengthCheck);
	if (wallOnLeft || wallOnRight)
		return true;
	else
		return false;
}
```

2. Add another field variable to control the distance of each jump.

```c#
public float jump = 14f;
```

3. Now, just below the IsWallToLeftOrRight() method, add a new method to check to see if Soy Boy is touching a wall or the ground.

```c#
public bool PlayerIsTouchingGroundOrWall()
{
   if(PlayerIsOnGround() || IsWallToLeftOrRight())
      return true;
   else
      return false;
}
```

4. The **yVelocity** value is set to the jump value of 14f when the character is jumping from the ground, or wall. Otherwise it is set to the current velocity of the rigidbody. In the **FixedUpdate()** method, just above the **rb.AddForce(…)** line, add the following:

```c#
var yVelocity = 0f;
if(PlayerIsTouchingGroundOrWall() && input.y == 1)
   yVelocity = jump;
else
   yVelocity = rb.velocity.y;
```

5. Still in the **FixedUpdaet()** method, change the line that reads:
```c#
rb.velocity = new Vector2(xVelocity, rb.velocity.y);
```
to instead read:
```c#
rb.velocity = new Vector2(xVelocity, yVelocity);
```

6. At this point, your character can jump up on walls, but he will just move straight up next to them when jumping. This definitely doesn&#39;t look right. The right way would be to propel him away from the walls in the opposite direction when jumping. Just below the **PlayerIsTouchingGroundOrWall()** method, add another new method:

```c#
public int GetWallDirection()
{
	bool isWallLeft = Physics2D.Raycast (new Vector2 (transform.position.x - width, transform.position.y), -Vector2.right, rayCastLengthCheck);
	bool isWallRight = Physics2D.Raycast (new Vector2 (transform.position.x + width, transform.position.y), Vector2.right, rayCastLengthCheck);
	if (isWallLeft)
		return -1;
	else if(isWallRight)
        return 1;
	else
        return 0;
}
```

7. The GetWallDirection() method returns an integer based on whether the wall is left (-1), right (1), or neither (0). You&#39;ll use the returned value to multiply against the character&#39;s xVelocity to either make him go left or right (when wall jumping) based on which side the wall is on. Add the following statements in the FixedUpdate() method just below the line that now reads velocity = new Vector2(xVelocity, yVelocity);.

```c#
if(IsWallToLeftOrRight() && !PlayerIsOnGround() && input.y == 1)
{
   rb.velocity = new Vector2(-GetWallDirection() * speed * 0.75f, rb.velocity.y);
}
```


8. Save your changes and switch back to Unity. Use the Scene editor to duplicate some single-block prefabs and move the copies up on top of each other to build some walls. Remember to hold CTRL when moving them in the Scene editor to keep them snapped to 1 unit size.
9. Run the game, and jump against the wall. As you land on the wall, jump again, moving in the other direction. With this technique your can now scale walls!

 ![Soy Boy with walls](/pics2/sb22.png)
 
## Hooking Up Character Animations

The character controller is now complete in regard to input, so now we will focus on animations.

1. Click the **SoyBoy** GameObject, then open the **Window\Animator**** Parameter** window.
2. There are no parameters yet. Click the + icon three times and add the following parameters.
    * Speed as a Float with value 0.0
    * IsJumping as a Bool, unchecked
    * IsOnWall as a Bool, unchecked

 ![](/pics2/sb23.png)

3. Now to create states. Open the **Animations** folder and double-click on **SoyBoy**. This will open up the States palette. Make the following changes.
    * Right-click the **Entry** state, select **Make Transition** and then click the **IdleAnimation** block. An arrow is created from Entry pointing to IdleAnimation.
    * Now you need to make the **IdleAnimation** the new default state by executing a right-click **IdleAnimation** and click **Set as Layer Default State**.
    * Now create a transition from **IdleAnimation** to **RunAnimation** , and again from **RunAnimation** to **IdleAnimation**. You may want to drag the state blocks around in the window to position them better. It should look something like the following.

 ![](/pics2/sb24.png)

4. Click the **transition**** arrow **pointing from** IdleAnimation **to** RunAnimation **. The Isnpector window should show you the details of this transition. Click the** + **icon under** Conditions **, and select the** Speed **parameter in the** Condition **drop down. Ensure the check is set to** Greater **than** 0 **. Uncheck the check box for** Has Exit Time**. This makes sure the animation changes state as soon as the parameter check is met.
5. Click the **transition**** arrow **pointing from** RunAnimation **to** IdleAnimation **. Click the** + **icon under** Conditions **, and select the** Speed **parameter in the** Condition **drop down. This time you want to transition from running back to idle, so set the check** Less **than** 0.05 **. Once again uncheck** Has Exit Time** check box.
6. Finish setting up the transition arrows according to the image below.

 ![](/pics2/sb25.png)

7. On every transition you make, ensure you uncheck **Has Exit Time** check box in the Inspector for that transition.
8. There are transitions to and from every state now, except between **SlideAnimation** and **IdleAnimation** where there is only a state change from sliding to idle. Click each arrow in the **Animator** editor and create conditions as specified in the table below.

| **Idle**  **-->**  **Jump** | **Jump**  **-->**  **Idle** | **Run**  **-->**  **Jump** | **Jump**  **-->**  **Run** |
| --- | --- | --- | --- |
| IsJumping = true | Speed Less 0.05 | IsJumping = true | IsJumping = false |
| Speed Less 0.5 | IsJumping = false |   | Speed Greater 0 |
|   | IsOnWall = false |   |   |
| **Run**  **-->**  **Slide** | **Slide**  **-->**  **Run** | **Slide**  **-->**  **Jump** | **Jump**  **-->**  **Slide** |
| IsOnWall = true | IsOnWall = false | IsJumping = true | IsOnWall = true |
|   | Speed Greater 0 | IsOnWall = false | IsJumping = false |
|   |   |   |   |
| **Slide**  **-->**  **Idle** |   |   |   |
| IsOnWall = false |   |   |   |
| Speed Less 0.05 |   |   |   |

9. Lastly, select the **RunAnimation** block, and change the **speed** for the animation from 1 to **2.5**. Close the Animator window.
10. Now we need to change the **SoyBoyController** script to modify your new animation parameters. In the **Update()** method, just below the **Input.GetAxis()** … add the following.

```c#
animator.SetFloat("Speed", Mathf.Abs(input.x));
```

11. In the same **Update()** method, change the block of code that checks for input on the Y axis from:

```c#
if(input.y >= 1f)
{
	jumpDuration += Time.deltaTime;
}
else
{
	isJumping = false;
	jumpDuration = 0f;
}
```

to be the following:

```c#
if(input.y >= 1f)
{
	jumpDuration += Time.deltaTime;
	animator.SetBool("IsJumping", true);
}
else
{
	isJumping = false;
	animator.SetBool("IsJumping", false);
	jumpDuration = 0f;
}
```

12. In the **Update()** method once more, just after the closing brace in the code that says:

 ```c#
 if (input.y > 0f)
 	isJumping = true;
 ```

add the following:

```c#
animator.SetBool("IsOnWall", false);
```

13. Move to the **FixedUpdate()** method next, and replace this code block:

```c#
if(IsWallToLeftOrRight() && !PlayerIsOnGround() && input.y == 1)
{
   rb.velocity = new Vector2(-GetWallDirection() * speed * 0.75f, rb.velocity.y);
}
```

with this:

```c#
if(IsWallToLeftOrRight() && !PlayerIsOnGround() && input.y == 1)
{
   rb.velocity = new Vector2(-GetWallDirection() * speed * 0.75f, rb.velocity.y);
   animator.SetBool("IsOnWall", false);
   animator.SetBool("IsJumping", true);
}
else if(!IsWallToLeftOrRight())
{
   animator.SetBool("IsOnWall", false);
   animator.SetBool("IsJumping", true);
}
if(IsWallToLeftOrRight() && !PlayerIsOnGround())
{
   animator.SetBool("IsOnWall", true);
}
```

14. Save your changes and run the game, and try out some basic controls. Soy Boy should transition between all the different states, including sliding down the wall when you&#39;re up against a wall and falling.


