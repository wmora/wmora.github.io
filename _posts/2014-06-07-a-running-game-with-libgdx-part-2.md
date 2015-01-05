--- 
title: LibGDX Tutorial - A Running Game with libGDX - Part 2
date: "2014-06-07T10:02:00.001-03:00"
author: William Mora
tags: 
- Java
- box2d
- software development
- libGDX
- video games
- Android
redirect_from: 
- /2014/06/a-running-game-with-libgdx-part-2.html
assets_url: /assets/libgdx-martianrun-tutorial
---

Check out [part 1](/a-running-game-with-libgdx-part-1) for the project and world setup!

This is part 2 of a tutorial on writing a 2d running game. Remember the code is on [GitHub](https://github.com/wmora/martianrun). Also, a final version based on this tutorial is on [Google Play](https://play.google.com/store/apps/details?id=com.gamestudio24.cityescape.android).

<!--more-->
## Run, Jump and Dodge!
The next step is to create our main guy, the runner. He will be a dynamic body with a fixed position on the x-axis with the ability to jump when the right side of the screen is touched and will dodge for as long the left side of the screen is touched.

First, let's build our function inside `WorldUtils` that creates the runner:

```java
package com.gamestudio24.martianrun.utils;

import ...

public class WorldUtils {

    //Rest of the code
    ...

    public static Body createRunner(World world) {
        BodyDef bodyDef = new BodyDef();
        bodyDef.type = BodyDef.BodyType.DynamicBody;
        bodyDef.position.set(new Vector2(Constants.RUNNER_X, Constants.RUNNER_Y));
        PolygonShape shape = new PolygonShape();
        shape.setAsBox(Constants.RUNNER_WIDTH / 2, Constants.RUNNER_HEIGHT / 2);
        Body body = world.createBody(bodyDef);
        body.createFixture(shape, Constants.RUNNER_DENSITY);
        body.resetMassData();
        shape.dispose();
        return body;
    }

}
```

And add the following constants:

```java
package com.gamestudio24.martianrun.utils;

import com.badlogic.gdx.math.Vector2;

public class Constants {

    ...

    public static final float RUNNER_X = 2;
    public static final float RUNNER_Y = GROUND_Y + GROUND_HEIGHT;
    public static final float RUNNER_WIDTH = 1f;
    public static final float RUNNER_HEIGHT = 2f;
    public static float RUNNER_DENSITY = 0.5f;

}
```

How is our runner set up? We created him as a box that is 1 meter wide and 2 meters long. His position will be fixed at 2 meters from the left side of the screen and on top of the ground (duh). Let's make sure that our runner is set up fine by adding him to the `GameStage`:

```java
package com.gamestudio24.martianrun.stages;

import ...

public class GameStage extends Stage {

    ...

    private World world;
    private Body ground;
    private Body runner;

    ...

    public GameStage() {
        world = WorldUtils.createWorld();
        ground = WorldUtils.createGround(world);
        runner = WorldUtils.createRunner(world);
        renderer = new Box2DDebugRenderer();
        setupCamera();
    }

    // The rest
    ...

}
```

Run the game again and you should now see the runner (a box) standing there doing nothing, like the image below:

<img src="{{ page.assets_url }}/box.png" width=480 />

Good! Now, before we add the controls to our game, we should make good use of our `GameStage` and use `Actor` classes for our world components (the ground and the runner). In a newly created `actors` package, let's create a base class for our actors called `GameActor`. Let's start with this:

```java
package com.gamestudio24.martianrun.actors;

import com.badlogic.gdx.physics.box2d.Body;
import com.badlogic.gdx.scenes.scene2d.Actor;

public abstract class GameActor extends Actor {

    protected Body body;

    public GameActor(Body body) {
        this.body = body;
    }

}
```

With the base class done, I'll add a `Ground` class for our ground:

```java
package com.gamestudio24.martianrun.actors;

import com.badlogic.gdx.physics.box2d.Body;
import com.gamestudio24.martianrun.box2d.GroundUserData;

public class Ground extends GameActor {

    public Ground(Body body) {
        super(body);
    }

}
```

And a `Runner` class for our runner:

```java
package com.gamestudio24.martianrun.actors;

import com.badlogic.gdx.physics.box2d.Body;

public class Runner extends GameActor {

    public Runner(Body body) {
        super(body);
    }

}
```

Notice how our constructor is expecting a `Body`. This will make sure each `Actor` is responsible for the physics `Body` it is supposed to render and update. Let's change our `GameStage` so it uses our newly created classes.

```java
package com.gamestudio24.martianrun.stages;

import ...

public class GameStage extends Stage {

    ...

    private World world;
    private Ground ground;
    private Runner runner;

    ...

    public GameStage() {
        setUpWorld();
        setupCamera();
        renderer = new Box2DDebugRenderer();
    }

    private void setUpWorld() {
        world = WorldUtils.createWorld();
        setUpGround();
        setUpRunner();
    }

    private void setUpGround() {
        ground = new Ground(WorldUtils.createGround(world));
        addActor(ground);
    }

    private void setUpRunner() {
        runner = new Runner(WorldUtils.createRunner(world));
        addActor(runner);
    }

    private void setupCamera() ...

    ...    

}
```

If you run the project at this point, you should still see the last posted image (the static runner on the ground).

The first control we'll add to the runner is jumping. The logic is the following: when I touch the right side of the screen, the runner will jump vertically and won't be able to jump again until he's landed.

So, how do I make a body jump? We apply a _linear impulse_ to the body in the y-direction and let box2d do its magic. We'll make use of box2d's _UserData_ to store information needed by our `Body` objects in order to behave properly. We'll also create an `enum` to store the different `UserData` types we have available; this will come in handy when detecting collisions later on.

In an `enums` package, create an enum called `UserDataType`: 

```java
package com.gamestudio24.martianrun.enums;

public enum UserDataType {

    GROUND,
    RUNNER

}
```

In a newly created `box2d` package, create an abstract `UserData` class which will be used by our characters:

```java
package com.gamestudio24.martianrun.box2d;

import com.gamestudio24.martianrun.enums.UserDataType;

public abstract class UserData {

    protected UserDataType userDataType;

    public UserData() {

    }

    public UserDataType getUserDataType() {
        return userDataType;
    }

}
```

And extend it for our `Ground` and our `Runner`:

```java
package com.gamestudio24.martianrun.box2d;

import com.gamestudio24.martianrun.enums.UserDataType;

public class GroundUserData extends UserData {

    public GroundUserData() {
        super();
        userDataType = UserDataType.GROUND;
    }

}
```

Our `RunnerUserData` will store the vector to apply to our body when jumping: 

```java
package com.gamestudio24.martianrun.box2d;

import com.badlogic.gdx.math.Vector2;
import com.gamestudio24.martianrun.enums.UserDataType;
import com.gamestudio24.martianrun.utils.Constants;

public class RunnerUserData extends UserData {

    private Vector2 jumpingLinearImpulse;

    public RunnerUserData() {
        super();
        jumpingLinearImpulse = Constants.RUNNER_JUMPING_LINEAR_IMPULSE;
        userDataType = UserDataType.RUNNER;
    }

    public Vector2 getJumpingLinearImpulse() {
        return jumpingLinearImpulse;
    }

    public void setJumpingLinearImpulse(Vector2 jumpingLinearImpulse) {
        this.jumpingLinearImpulse = jumpingLinearImpulse;
    }

}
```

Here's the new constant I added. Even though it may look like a random value, I actually did a bit of trial and error until the controls felt right. I'm also adding a gravity scale which I'll set in a bit because I want the runner to fall faster so the game has an arcade feel: 

```java
package com.gamestudio24.martianrun.utils;

import com.badlogic.gdx.math.Vector2;

public class Constants {

    ...
    public static final float RUNNER_GRAVITY_SCALE = 3f;
    public static float RUNNER_DENSITY = 0.5f;
    public static final Vector2 RUNNER_JUMPING_LINEAR_IMPULSE = new Vector2(0, 13f);

}
```

Even though I pointed out that `GroundUserData` is pointless, it is important to be consistent with the `UserData` your `Body` objects use to prevent unexpected behavior. We will store an instance of `UserData` in all our `Actor` objects so let's prepare our `GameActor` class for it: 

```java
package com.gamestudio24.martianrun.actors;

import com.badlogic.gdx.physics.box2d.Body;
import com.badlogic.gdx.scenes.scene2d.Actor;
import com.gamestudio24.martianrun.box2d.UserData;

public abstract class GameActor extends Actor {

    protected Body body;
    protected UserData userData;

    public GameActor(Body body) {
        this.body = body;
        this.userData = (UserData) body.getUserData();
    }

    public abstract UserData getUserData();

}
```

I need to override the new `getUserData()` function in our `Ground` class: 

```java
package com.gamestudio24.martianrun.actors;

import com.badlogic.gdx.physics.box2d.Body;

public class Ground extends GameActor {

    public Ground(Body body) {
        super(body);
    }

    @Override
    public GroundUserData getUserData() {
        return (GroundUserData) userData;
    }

}
```

Now we have all we need to make our character jump. We'll do it with a `jump()` function in our `Runner` class. We'll also add a `landed()` function to notify the runner he can jump again: 

```java
package com.gamestudio24.martianrun.actors;

import com.badlogic.gdx.physics.box2d.Body;
import com.gamestudio24.martianrun.box2d.RunnerUserData;

public class Runner extends GameActor {

    private boolean jumping;

    public Runner(Body body) {
        super(body);
    }

    @Override
    public RunnerUserData getUserData() {
        return (RunnerUserData) userData;
    }

    public void jump() {

        if (!jumping) {
            body.applyLinearImpulse(getUserData().getJumpingLinearImpulse(), body.getWorldCenter(), true);
            jumping = true;
        }

    }

    public void landed() {
        jumping = false;
    }

}
```

Pretty straightforward, but we need two more things: set the appropriate `UserData` when setting up our `World` and implementing the controls on the `GameStage` to actually make the character jump. Setting the `UserData` is easy, you just need to add a couple of lines in our `WorldUtils` class. I'm also going to add a gravity scale (already added in the `Constants` class) to the runner so he falls faster: 

```java
package com.gamestudio24.martianrun.utils;

import ...
import com.gamestudio24.martianrun.box2d.GroundUserData;
import com.gamestudio24.martianrun.box2d.RunnerUserData;

public class WorldUtils {

    public static World createWorld() {
        return new World(Constants.WORLD_GRAVITY, true);
    }

    public static Body createGround(World world) {
        ...
        body.setUserData(new GroundUserData());
        shape.dispose();
        return body;
    }

    public static Body createRunner(World world) {
        ...
        Body body = world.createBody(bodyDef);
        body.setGravityScale(Constants.RUNNER_GRAVITY_SCALE);
        body.createFixture(shape, Constants.RUNNER_DENSITY);
        body.resetMassData();
        body.setUserData(new RunnerUserData());
        shape.dispose();
        return body;
    }

}
```

To implement the controls in our `GameStage`, we must override the `touchDown` function, determine if the right side of the screen is touched and, if so, make the runner `jump()`: 

```java
package com.gamestudio24.martianrun.stages;

import com.badlogic.gdx.Gdx;
import com.badlogic.gdx.graphics.OrthographicCamera;
import com.badlogic.gdx.math.Rectangle;
import com.badlogic.gdx.math.Vector3;
import ...

public class GameStage extends Stage {

    ...

    private OrthographicCamera camera;
    private Box2DDebugRenderer renderer;

    private Rectangle screenRightSide;

    private Vector3 touchPoint;

    public GameStage() {
        setUpWorld();
        setupCamera();
        setupTouchControlAreas();
        renderer = new Box2DDebugRenderer();
    }

    ...

    private void setupCamera() {
        ...
    }

    private void setupTouchControlAreas() {
        touchPoint = new Vector3();
        screenRightSide = new Rectangle(getCamera().viewportWidth / 2, 0, getCamera().viewportWidth / 2,
                getCamera().viewportHeight);
        Gdx.input.setInputProcessor(this);
    }

    ...

    @Override
    public void draw() {
        ...
    }

    @Override
    public boolean touchDown(int x, int y, int pointer, int button) {

        // Need to get the actual coordinates
        translateScreenToWorldCoordinates(x, y);

        if (rightSideTouched(touchPoint.x, touchPoint.y)) {
            runner.jump();
        }

        return super.touchDown(x, y, pointer, button);
    }

    private boolean rightSideTouched(float x, float y) {
        return screenRightSide.contains(x, y);
    }

    /**
     * Helper function to get the actual coordinates in my world
     * @param x
     * @param y
     */
    private void translateScreenToWorldCoordinates(int x, int y) {
        getCamera().unproject(touchPoint.set(x, y, 0));
    }

}
```

The controls are almost done, we need one last thing. We want to know when the runner lands on the ground so we are able to jump again. The way we'll do this is by detecting a collision between the `Runner` and the `Ground` and notifying the `Runner` that he's `landed()`. There's a box2d listener for detecting collision between two bodies called `ContactListener` which we'll implement. First, I'll create a `BodyUtils` class inside the `utils` package with a couple of helper functions: 

```java
package com.gamestudio24.martianrun.utils;

import com.badlogic.gdx.physics.box2d.Body;
import com.gamestudio24.martianrun.box2d.UserData;
import com.gamestudio24.martianrun.enums.UserDataType;

public class BodyUtils {

    public static boolean bodyIsRunner(Body body) {
        UserData userData = (UserData) body.getUserData();

        return userData != null && userData.getUserDataType() == UserDataType.RUNNER;
    }

    public static boolean bodyIsGround(Body body) {
        UserData userData = (UserData) body.getUserData();

        return userData != null && userData.getUserDataType() == UserDataType.GROUND;
    }

}
```

And now implement the `ContactListener` in the `GameStage` class: 

```java
package com.gamestudio24.martianrun.stages;

import ...
import com.badlogic.gdx.physics.box2d.*;
import com.badlogic.gdx.scenes.scene2d.Stage;
import com.gamestudio24.martianrun.actors.Ground;
import com.gamestudio24.martianrun.actors.Runner;
import com.gamestudio24.martianrun.utils.BodyUtils;
import com.gamestudio24.martianrun.utils.WorldUtils;

public class GameStage extends Stage implements ContactListener {

    ...

    private void setUpWorld() {
        world = WorldUtils.createWorld();
        // Let the world now you are handling contacts
        world.setContactListener(this);
        setUpGround();
        setUpRunner();
    }

    ...

    @Override
    public void beginContact(Contact contact) {

        Body a = contact.getFixtureA().getBody();
        Body b = contact.getFixtureB().getBody();

        if ((BodyUtils.bodyIsRunner(a) && BodyUtils.bodyIsGround(b)) ||
                (BodyUtils.bodyIsGround(a) && BodyUtils.bodyIsRunner(b))) {
            runner.landed();
        }

    }

    @Override
    public void endContact(Contact contact) {

    }

    @Override
    public void preSolve(Contact contact, Manifold oldManifold) {

    }

    @Override
    public void postSolve(Contact contact, ContactImpulse impulse) {

    }
}
```

We've added a lot of stuff and it's finally ready to run! Run the game and you'll see that when you click/touch the right side of the screen the runner will jump just like is shown in the following video:

<iframe width="320" height="266" src="http://www.youtube.com/embed/RDFV6prbhr8" frameborder="0" allowfullscreen></iframe>

Our next step is to make the runner dodge. Now that we've got our blueprint for implementing the runner's moves it will be much easier to make him dodge. The logic is the following: if the runner is not jumping and we touch the left side of the screen, he will be sliding until we stop touching the left side of the screen. By sliding, we mean rotating our rectangle by -90 degrees and placing it just above the ground. We already know the position of the runner when running; it's in our `Constants` class. Now we also need to make note of the position of the runner while sliding by also adding it in the `Constants` class:

```java
public class Constants {

    ...

    public static float RUNNER_DENSITY = 0.5f;
    public static final float RUNNER_DODGE_X = 2f;
    public static final float RUNNER_DODGE_Y = 1.5f;
    public static final Vector2 RUNNER_JUMPING_LINEAR_IMPULSE = new Vector2(0, 13f);

}
```

And store this info in our `RunnerUserData` class: 

```java
package com.gamestudio24.martianrun.box2d;

import ...

public class RunnerUserData extends UserData {

    private final Vector2 runningPosition = new Vector2(Constants.RUNNER_X, Constants.RUNNER_Y);
    private final Vector2 dodgePosition = new Vector2(Constants.RUNNER_DODGE_X, Constants.RUNNER_DODGE_Y);
    private Vector2 jumpingLinearImpulse;

    public RunnerUserData() {
        super();
        jumpingLinearImpulse = Constants.RUNNER_JUMPING_LINEAR_IMPULSE;
        userDataType = UserDataType.RUNNER;
    }

    public Vector2 getJumpingLinearImpulse() {
        return jumpingLinearImpulse;
    }

    public void setJumpingLinearImpulse(Vector2 jumpingLinearImpulse) {
        this.jumpingLinearImpulse = jumpingLinearImpulse;
    }

    public float getDodgeAngle() {
        // In radians
        return (float) (-90f * (Math.PI / 180f));
    }

    public Vector2 getRunningPosition() {
        return runningPosition;
    }

    public Vector2 getDodgePosition() {
        return dodgePosition;
    }
}
```

In our `Runner` class, we'll add the functions to `dodge()` and `stopDodge()`. We also have to modify our `jump()` function so it does not make the runner jump while dodging: 

```java
package com.gamestudio24.martianrun.actors;

import ...

public class Runner extends GameActor {

    private boolean dodging;
    private boolean jumping;

    ...

    public void jump() {

        if (!(jumping || dodging)) {
            body.applyLinearImpulse(getUserData().getJumpingLinearImpulse(), body.getWorldCenter(), true);
            jumping = true;
        }

    }

    public void landed() {
        jumping = false;
    }

    public void dodge() {
        if (!jumping) {
            body.setTransform(getUserData().getDodgePosition(), getUserData().getDodgeAngle());
            dodging = true;
        }
    }

    public void stopDodge() {
        dodging = false;
        body.setTransform(getUserData().getRunningPosition(), 0f);
    }

    public boolean isDodging() {
        return dodging;
    }

}
```

Great! Now we need to call these functions from our `GameStage` class. When the user touches the left side of the screen we'll call `dodge()` and whenever we stop touching the screen we'll call `stopDodge()` if the runner `isDodging()`. It's easy to know when we stop touching the screen, we just need to override the `touchUp()` function: 

```java
package com.gamestudio24.martianrun.stages;

import ...

public class GameStage extends Stage implements ContactListener {

    ...
    private Rectangle screenLeftSide;
    private Rectangle screenRightSide;

    private void setupTouchControlAreas() {
        touchPoint = new Vector3();
        screenLeftSide = new Rectangle(0, 0, getCamera().viewportWidth / 2, getCamera().viewportHeight);
        screenRightSide = new Rectangle(getCamera().viewportWidth / 2, 0, getCamera().viewportWidth / 2,
                getCamera().viewportHeight);
        Gdx.input.setInputProcessor(this);
    }

    ...

    @Override
    public boolean touchDown(int x, int y, int pointer, int button) {

        // Need to get the actual coordinates
        translateScreenToWorldCoordinates(x, y);

        if (rightSideTouched(touchPoint.x, touchPoint.y)) {
            runner.jump();
        } else if (leftSideTouched(touchPoint.x, touchPoint.y)) {
            runner.dodge();
        }

        return super.touchDown(x, y, pointer, button);
    }

    @Override
    public boolean touchUp(int screenX, int screenY, int pointer, int button) {

        if (runner.isDodging()) {
            runner.stopDodge();
        }

        return super.touchUp(screenX, screenY, pointer, button);
    }

    private boolean rightSideTouched(float x, float y) {
        return screenRightSide.contains(x, y);
    }

    private boolean leftSideTouched(float x, float y) {
        return screenLeftSide.contains(x, y);
    }

    ...
}
```

Our dodging functionality is done! Run the game and go ahead and touch and hold the left side of the screen; you should now be able to dodge. The jump functionality stays the same.  
The video shows how the game is working at this point (ignore the title, the original game name was modified before the first release):  

<iframe width="320" height="266" src="http://www.youtube.com/embed/RDFV6prbhr8" frameborder="0" allowfullscreen></iframe>

Very nice, isn't it? So, what about the running part? We will keep our runner fixed at this position and make the objects move towards him, the animations will give the illusion of the runner moving forward. This means we are done with our runner's controls!

In the [next part](/a-running-game-with-libgdx-part-3), we will be adding our enemies to the game. 

If you have any questions or comments on what we have done so far, feel free to let me know in the comments section. Cheers!