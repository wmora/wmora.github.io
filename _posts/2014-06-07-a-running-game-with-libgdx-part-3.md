--- 
title: LibGDX Tutorial - A Running Game with libGDX - Part 3
date: "2014-06-07T10:03:00.000-03:00"
author: William Mora
tags: 
- Java
- box2d
- software development
- random
- libGDX
- video games
- enums
- Android
redirect_from: 
- /2014/06/a-running-game-with-libgdx-part-3.html
---

Check out [part 1](/a-running-game-with-libgdx-part-1) for the project and world setup!
Check out [part 2](/a-running-game-with-libgdx-part-2) for implementing controls for our runner!

This is part 3 of a tutorial on writing a 2d running game. Remember the code is on [GitHub](https://github.com/wmora/martianrun). Also, a final version based on this tutorial is on [Google Play](https://play.google.com/store/apps/details?id=com.gamestudio24.cityescape.android).

<!--more-->
## Watch Out for the Enemies!
In this section we'll introduce our enemies, a bunch of running/flying insects. They will all be coming towards our runner in an attempt to hit/stop him. We will have 6 types of enemies: 4 of them will be running so we'll have to jump over them and the other 2 will be flying, so we'll have to dodge them. This is the configuration (width x height in meters) I came up with for all enemies:

*   Running small - 1 x 1
*   Running wide - 2 x 1
*   Running long - 1 x 2
*   Running big - 2 x 2
*   Flying small - 1 x 1
*   Flying wide - 2 x 1

The logic for our enemies will be the following: we'll place an enemy just outside of the right side of the screen and move him towards the left end of the screen. If the runner successfully avoids getting hit by the enemy, we'll create a new one until one of them hits the runner. The type of enemy to create will be randomly determined.

Since we want to generate different types of enemies, we'll store their configuration as an `enum`. In our `enums` package, create an enum called `EnemyType` with the following:

```java
package com.gamestudio24.martianrun.enums;

import com.gamestudio24.martianrun.utils.Constants;

public enum EnemyType {

    RUNNING_SMALL(1f, 1f, Constants.ENEMY_X, Constants.RUNNING_SHORT_ENEMY_Y, Constants.ENEMY_DENSITY),
    RUNNING_WIDE(2f, 1f, Constants.ENEMY_X, Constants.RUNNING_SHORT_ENEMY_Y, Constants.ENEMY_DENSITY),
    RUNNING_LONG(1f, 2f, Constants.ENEMY_X, Constants.RUNNING_LONG_ENEMY_Y, Constants.ENEMY_DENSITY),
    RUNNING_BIG(2f, 2f, Constants.ENEMY_X, Constants.RUNNING_LONG_ENEMY_Y, Constants.ENEMY_DENSITY),
    FLYING_SMALL(1f, 1f, Constants.ENEMY_X, Constants.FLYING_ENEMY_Y, Constants.ENEMY_DENSITY),
    FLYING_WIDE(2f, 1f, Constants.ENEMY_X, Constants.FLYING_ENEMY_Y, Constants.ENEMY_DENSITY);

    private float width;
    private float height;
    private float x;
    private float y;
    private float density;

    EnemyType(float width, float height, float x, float y, float density) {
        this.width = width;
        this.height = height;
        this.x = x;
        this.y = y;
        this.density = density;
    }

    public float getWidth() {
        return width;
    }

    public float getHeight() {
        return height;
    }

    public float getX() {
        return x;
    }

    public float getY() {
        return y;
    }

    public float getDensity() {
        return density;
    }

}
```

The new `Constants` added: 

```java
package com.gamestudio24.martianrun.utils;

import com.badlogic.gdx.math.Vector2;

public class Constants {

    ...

    public static final float ENEMY_X = 25f;
    public static final float ENEMY_DENSITY = RUNNER_DENSITY;
    public static final float RUNNING_SHORT_ENEMY_Y = 1.5f;
    public static final float RUNNING_LONG_ENEMY_Y = 2f;
    public static final float FLYING_ENEMY_Y = 3f;
    public static final Vector2 ENEMY_LINEAR_VELOCITY = new Vector2(-10f, 0);

}
```

To obtain a random value from an `enum`, I created a `RandomUtils` class inside our `utils` package that contains a `private static RandomEnum` class that returns a random value from a given `enum` (credits go to [this SO question](http://stackoverflow.com/a/1973018)): 

```java
package com.gamestudio24.martianrun.utils;

import com.gamestudio24.martianrun.enums.EnemyType;

import java.util.Random;

public class RandomUtils {

    public static EnemyType getRandomEnemyType() {
        RandomEnum<EnemyType> randomEnum = new RandomEnum<EnemyType>(EnemyType.class);
        return randomEnum.random();
    }

    /**
     * @see [Stack Overflow](http://stackoverflow.com/a/1973018)
     * @param <E>
     */

    private static class RandomEnum<E extends Enum> {

        private static final Random RND = new Random();
        private final E[] values;

        public RandomEnum(Class<E> token) {
            values = token.getEnumConstants();
        }

        public E random() {
            return values[RND.nextInt(values.length)];
        }
    }

}
```

Since we want to be consistent with our project setup, we have to create an `EnemyUserData` which extends from our `UserData` as well as an `Enemy` which extends from our `GameActor`. Let's add an `ENEMY` value to our `UserDataType` enum: 

```java
package com.gamestudio24.martianrun.enums;

public enum UserDataType {

    GROUND,
    RUNNER,
    ENEMY

}
```

Hey! What was that `ENEMY_LINEAR_VELOCITY` we added to our `Constants`? Well, remember we are using our custom `UserData` to store any info regarding our characters' movement. Our enemies will always run with a constant _linear velocity_ to the left (negative x-direction) and a `EnemyUserData` class in our `box2d` package will be the perfect place to store it. 

Before creating or `EnemyUserData` class, I'd like to change something in our `UserData` class: when we check if a character is in bounds (we'll add that in a little bit), I need the character's `width` in order to calculate  it. I want to store this info whenever we create a new `UserData` object: 

```java
package com.gamestudio24.martianrun.box2d;

import com.gamestudio24.martianrun.enums.UserDataType;

public abstract class UserData {

    protected UserDataType userDataType;
    protected float width;
    protected float height;

    public UserData() {

    }

    public UserData(float width, float height) {
        this.width = width;
        this.height = height;
    }

    ...

    public float getWidth() {
        return width;
    }

    public void setWidth(float width) {
        this.width = width;
    }

    public float getHeight() {
        return height;
    }

    public void setHeight(float height) {
        this.height = height;
    }

}
```

Let's match this new constructor in our `RunnerUserData`: 

```java
package com.gamestudio24.martianrun.box2d;

import ...

public class RunnerUserData extends UserData {

    ...

    public RunnerUserData(float width, float height) {
        super(width, height);
        jumpingLinearImpulse = Constants.RUNNER_JUMPING_LINEAR_IMPULSE;
        userDataType = UserDataType.RUNNER;
    }

    ...

}
```

And now we can create our `EnemyUserData`: 

```java
package com.gamestudio24.martianrun.box2d;

import com.badlogic.gdx.math.Vector2;
import com.gamestudio24.martianrun.enums.UserDataType;
import com.gamestudio24.martianrun.utils.Constants;

public class EnemyUserData extends UserData {

    private Vector2 linearVelocity;

    public EnemyUserData(float width, float height) {
        super(width, height);
        userDataType = UserDataType.ENEMY;
        linearVelocity = Constants.ENEMY_LINEAR_VELOCITY;
    }

    public void setLinearVelocity(Vector2 linearVelocity) {
        this.linearVelocity = linearVelocity;
    }

    public Vector2 getLinearVelocity() {
        return linearVelocity;
    }

}
```

Let's now add our `Enemy` class that extends from our `GameActor`. Like it's mentioned before, the logic for an enemy is to constantly move in the -x direction: 

```java
package com.gamestudio24.martianrun.actors;

import com.badlogic.gdx.physics.box2d.Body;
import com.gamestudio24.martianrun.box2d.EnemyUserData;

public class Enemy extends GameActor {

    public Enemy(Body body) {
        super(body);
    }

    @Override
    public EnemyUserData getUserData() {
        return (EnemyUserData) userData;
    }

    @Override
    public void act(float delta) {
        super.act(delta);
        body.setLinearVelocity(getUserData().getLinearVelocity());
    }

}
```

We'll add a `createEnemy` function using our `WorldUtils` to create an `Enemy` of a random `EnemyType`. Note that in our `World`, an enemy will be a `KinematicBody`, which means it will move but it won't be affected if it's hit by another object (the `Runner`). While we are at it, we also have to change a line in our `createRunner` function so we store the `Runner width` and `height`: 

```java
package com.gamestudio24.martianrun.utils;

import ...
import com.gamestudio24.martianrun.box2d.EnemyUserData;
import com.gamestudio24.martianrun.box2d.GroundUserData;
import com.gamestudio24.martianrun.box2d.RunnerUserData;
import com.gamestudio24.martianrun.enums.EnemyType;

public class WorldUtils {

    ...

    public static Body createRunner(World world) {
        ...
        body.setUserData(new RunnerUserData(Constants.RUNNER_WIDTH, Constants.RUNNER_HEIGHT));
        shape.dispose();
        return body;
    }

    public static Body createEnemy(World world) {
        EnemyType enemyType = RandomUtils.getRandomEnemyType();
        BodyDef bodyDef = new BodyDef();
        bodyDef.type = BodyDef.BodyType.KinematicBody;
        bodyDef.position.set(new Vector2(enemyType.getX(), enemyType.getY()));
        PolygonShape shape = new PolygonShape();
        shape.setAsBox(enemyType.getWidth() / 2, enemyType.getHeight() / 2);
        Body body = world.createBody(bodyDef);
        body.createFixture(shape, enemyType.getDensity());
        body.resetMassData();
        EnemyUserData userData = new EnemyUserData(enemyType.getWidth(), enemyType.getHeight());
        body.setUserData(userData);
        shape.dispose();
        return body;
    }

}
```

Now, we will add the following logic to our `GameStage`: when the game is running, check if there are any enemies on screen; if there aren't any create a new one. Also, when we check if our enemies are out of bounds, we also want to check if our runner is out of bounds to destroy its `Body`. If an `Enemy` hits our `Runner`, we'll apply an _angular impulse_ to our `Runner` so the hit looks a bit cartoonish. 

Let's store the _angular impulse_ we just mentioned in our `Constants`: 

```java
package com.gamestudio24.martianrun.utils;

import com.badlogic.gdx.math.Vector2;

public class Constants {

    ...
    public static final Vector2 RUNNER_JUMPING_LINEAR_IMPULSE = new Vector2(0, 13f);
    public static final float RUNNER_HIT_ANGULAR_IMPULSE = 10f;

    ...

}
```

And access it through our `RunnerUserData` object: 

```java
package com.gamestudio24.martianrun.box2d;

import ...

public class RunnerUserData extends UserData {

    ...

    public float getHitAngularImpulse() {
        return Constants.RUNNER_HIT_ANGULAR_IMPULSE;
    }

}
```

Our `Runner` now needs to apply this _angular impulse_ whenever he's `hit()`: 

```java
package com.gamestudio24.martianrun.actors;

import com.badlogic.gdx.physics.box2d.Body;
import com.gamestudio24.martianrun.box2d.RunnerUserData;

public class Runner extends GameActor {

    private boolean dodging;
    private boolean jumping;
    private boolean hit;

    ...

    public void jump() {

        if (!(jumping || dodging || hit)) {
            body.applyLinearImpulse(getUserData().getJumpingLinearImpulse(), body.getWorldCenter(), true);
            jumping = true;
        }

    }

    ...

    public void dodge() {
        if (!(jumping || hit)) {
            body.setTransform(getUserData().getDodgePosition(), getUserData().getDodgeAngle());
            dodging = true;
        }
    }

    public void stopDodge() {
        dodging = false;
        // If the runner is hit don't force him back to the running position
        if (!hit) {
            body.setTransform(getUserData().getRunningPosition(), 0f);    
        } 
    }

    public boolean isDodging() {
        return dodging;
    }

    public void hit() {
        body.applyAngularImpulse(getUserData().getHitAngularImpulse(), true);
        hit = true;
    }

    public boolean isHit() {
        return hit;
    }

}
```

Now, add a couple of helper functions in our `BodyUtils` class: `bodyIsEnemy` checks whether the given `Body` belongs to an `Enemy` and `bodyInBounds` checks whether the `Runner` or an `Enemy` is in our screen boundaries: 

```java
package com.gamestudio24.martianrun.utils;

import ...

public class BodyUtils {

    public static boolean bodyInBounds(Body body) {
        UserData userData = (UserData) body.getUserData();

        switch (userData.getUserDataType()) {
            case RUNNER:
            case ENEMY:
                return body.getPosition().x + userData.getWidth() / 2 > 0;
        }

        return true;
    }

    public static boolean bodyIsEnemy(Body body) {
        UserData userData = (UserData) body.getUserData();

        return userData != null && userData.getUserDataType() == UserDataType.ENEMY;
    }

    ...

}
```

Hook it all up in the `GameStage`: 

```java
package com.gamestudio24.martianrun.stages;

import ...
import com.badlogic.gdx.utils.Array;
import com.gamestudio24.martianrun.actors.Enemy;
import com.gamestudio24.martianrun.actors.Ground;
import com.gamestudio24.martianrun.actors.Runner;
import com.gamestudio24.martianrun.utils.BodyUtils;
import com.gamestudio24.martianrun.utils.WorldUtils;

public class GameStage extends Stage implements ContactListener {

    ...

    private void setUpWorld() {
        world = WorldUtils.createWorld();
        world.setContactListener(this);
        setUpGround();
        setUpRunner();
        createEnemy();
    }

    ...

    @Override
    public void act(float delta) {
        super.act(delta);

        Array<Body> bodies = new Array<Body>(world.getBodyCount());
        world.getBodies(bodies);

        for (Body body : bodies) {
            update(body);
        }

        // Fixed timestep
        accumulator += delta;

        while (accumulator >= delta) {
            world.step(TIME_STEP, 6, 2);
            accumulator -= TIME_STEP;
        }

        //TODO: Implement interpolation

    }

    private void update(Body body) {
        if (!BodyUtils.bodyInBounds(body)) {
            if (BodyUtils.bodyIsEnemy(body) && !runner.isHit()) {
                createEnemy();
            }
            world.destroyBody(body);
        }
    }

    private void createEnemy() {
        Enemy enemy = new Enemy(WorldUtils.createEnemy(world));
        addActor(enemy);
    }

    ...

    @Override
    public void beginContact(Contact contact) {

        Body a = contact.getFixtureA().getBody();
        Body b = contact.getFixtureB().getBody();

        if ((BodyUtils.bodyIsRunner(a) && BodyUtils.bodyIsEnemy(b)) ||
                (BodyUtils.bodyIsEnemy(a) && BodyUtils.bodyIsRunner(b))) {
            runner.hit();
        } else if ((BodyUtils.bodyIsRunner(a) && BodyUtils.bodyIsGround(b)) ||
                (BodyUtils.bodyIsGround(a) && BodyUtils.bodyIsRunner(b))) {
            runner.landed();
        }

    }

    ...

}
```

And we've got ourselves a game! Run the game and you'll get "enemies" of different types moving towards the runner. You have to jump or dodge to avoid getting hit by any of them. The following video shows the game at this point (ignore the title, the original game name was modified before the first release): 

<iframe width="320" height="266" src="http://www.youtube.com/embed/X-jEZHDN-gw" frameborder="0" allowfullscreen></iframe>

At this point, I think enough has been covered to understand the basics of implementing a running game. In the next parts of this guide I will add some more stuff to make the game a little more fun, but I recommend you take it from here and customize it as you wish. 
[Click here](/a-running-game-with-libgdx-part-4) for part 4. 
Feel free to drop any questions or comments in the comments section. Cheers!