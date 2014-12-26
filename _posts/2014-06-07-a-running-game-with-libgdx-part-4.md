--- 
title: LibGDX Tutorial - A Running Game with libGDX - Part 4
date: "2014-06-07T10:04:00.002-03:00"
author: William Mora
tags: 
- Java
- software development
- scene2d
- libGDX
- video games
- Android
permalink: /2014/06/a-running-game-with-libgdx-part-4.html
assets_url: /assets/libgdx-cityescape-tutorial
---

Check out [part 1](http://www.williammora.com/2014/06/a-running-game-with-libgdx-part-1.html) for the project and world setup!
Check out [part 2](http://www.williammora.com/2014/06/a-running-game-with-libgdx-part-2.html) for implementing controls for our runner!
We introduced our enemies on [part 3](http://www.williammora.com/2014/06/a-running-game-with-libgdx-part-3.html).

This is part 4 of a tutorial on writing a 2d running game. Remember the code is on [GitHub](https://github.com/wmora/cityescape). Also, a final version based on this tutorial is on [Google Play](https://play.google.com/store/apps/details?id=com.gamestudio24.cityescape.android).

<!--more-->
## Graphics Would Be Nice!
This is the part where you should really make the game your own instead of using the resources from here; I am going to be adding textures to the game. We are going to do the following:

*   Add a moving background
*   Add texture to the ground

All graphics have been made by [Kenney](http://www.kenney.nl/).

### Add a moving background

We will be adding the following background to the game:

<img src="{{ page.assets_url }}/background.png" width=400 />

But it would be nice to make the background look as if it is moving instead of a static one. For that I will be using the following technique: I will show two copies of the image and keep two rectangles that will determine the current position where each image should be rendered. We will save the background image inside the `$PROJECT_ROOT/android/assets/` path, and name it `background.png`. Let's keep track of the image path in our `Constants` class: 

```java
package com.gamestudio24.cityescape.utils;

import com.badlogic.gdx.math.Vector2;

public class Constants {

    ...

    public static final String BACKGROUND_IMAGE_PATH = "background.png";
}
```

Now let's create our `Background` class. Like I said before, it will keep a reference to a `TextureRegion` objects with two `Rectangle` objects. It will update the bounds of each `Rectangle` in the -x direction at a constant speed and when one of the `Rectangle` bounds reaches the end of the screen, reset both boundaries. Note that this class extends `Actor` and not our `GameActor`. This is because we don't really need any of the `box2d` elements since the background is not really a game character: 

```java
package com.gamestudio24.cityescape.actors;

import com.badlogic.gdx.Gdx;
import com.badlogic.gdx.graphics.Texture;
import com.badlogic.gdx.graphics.g2d.Batch;
import com.badlogic.gdx.graphics.g2d.TextureRegion;
import com.badlogic.gdx.math.Rectangle;
import com.badlogic.gdx.scenes.scene2d.Actor;
import com.gamestudio24.cityescape.utils.Constants;

public class Background extends Actor {

    private final TextureRegion textureRegion;
    private Rectangle textureRegionBounds1;
    private Rectangle textureRegionBounds2;
    private int speed = 100;

    public Background() {
        textureRegion = new TextureRegion(new Texture(Gdx.files.internal(Constants.BACKGROUND_IMAGE_PATH)));
        textureRegionBounds1 = new Rectangle(0 - Constants.APP_WIDTH / 2, 0, Constants.APP_WIDTH, Constants.APP_HEIGHT);
        textureRegionBounds2 = new Rectangle(Constants.APP_WIDTH / 2, 0, Constants.APP_WIDTH, Constants.APP_HEIGHT);
    }

    @Override
    public void act(float delta) {
        if (leftBoundsReached(delta)) {
            resetBounds();
        } else {
            updateXBounds(-delta);
        }
    }

    @Override
    public void draw(Batch batch, float parentAlpha) {
        super.draw(batch, parentAlpha);
        batch.draw(textureRegion, textureRegionBounds1.x, textureRegionBounds1.y, Constants.APP_WIDTH,
                Constants.APP_HEIGHT);
        batch.draw(textureRegion, textureRegionBounds2.x, textureRegionBounds2.y, Constants.APP_WIDTH,
                Constants.APP_HEIGHT);
    }

    private boolean leftBoundsReached(float delta) {
        return (textureRegionBounds2.x - (delta * speed)) <= 0;
    }

    private void updateXBounds(float delta) {
        textureRegionBounds1.x += delta * speed;
        textureRegionBounds2.x += delta * speed;
    }

    private void resetBounds() {
        textureRegionBounds1 = textureRegionBounds2;
        textureRegionBounds2 = new Rectangle(Constants.APP_WIDTH, 0, Constants.APP_WIDTH, Constants.APP_HEIGHT);
    }

}
```

Now on to the `GameStage`. We need to get rid of our `Box2dDebugRenderer` (the ugly boxes where just for debugging purposes). We will also update the camera viewport and begin translating our `box2d` coordinates to screen coordinates (in pixels). 

For now, we will only render our newly created `Background`: 

```java
package com.gamestudio24.cityescape.stages;

import ...
import com.badlogic.gdx.utils.Scaling;
import com.badlogic.gdx.utils.viewport.ScalingViewport;
import com.gamestudio24.cityescape.actors.Background;
import com.gamestudio24.cityescape.actors.Enemy;
import com.gamestudio24.cityescape.actors.Ground;
import com.gamestudio24.cityescape.actors.Runner;
import com.gamestudio24.cityescape.utils.BodyUtils;
import com.gamestudio24.cityescape.utils.Constants;
import com.gamestudio24.cityescape.utils.WorldUtils;

public class GameStage extends Stage implements ContactListener {

    private static final int VIEWPORT_WIDTH = Constants.APP_WIDTH;
    private static final int VIEWPORT_HEIGHT = Constants.APP_HEIGHT;

    ...

    public GameStage() {
        super(new ScalingViewport(Scaling.stretch, VIEWPORT_WIDTH, VIEWPORT_HEIGHT,
                new OrthographicCamera(VIEWPORT_WIDTH, VIEWPORT_HEIGHT)));
        setUpWorld();
        setupCamera();
        setupTouchControlAreas();
    }

    private void setUpWorld() {
        world = WorldUtils.createWorld();
        world.setContactListener(this);
        setUpBackground();
        setUpGround();
        setUpRunner();
        createEnemy();
    }

    private void setUpBackground() {
        addActor(new Background());
    }

}
```

Run the game and you will see a nice, smooth animation of a moving background just like the video below:  

<div class="separator" style="clear: both; text-align: center;"><object width="320" height="266" class="BLOG_video_class" id="BLOG_video-b6ef51add911a795" classid="clsid:D27CDB6E-AE6D-11cf-96B8-444553540000" codebase="http://download.macromedia.com/pub/shockwave/cabs/flash/swflash.cab#version=6,0,40,0"><param name="movie" value="//www.youtube.com/get_player"><param name="bgcolor" value="#FFFFFF"><param name="allowfullscreen" value="true"><param name="flashvars" value="flvurl=http://redirector.googlevideo.com/videoplayback?id%3Db6ef51add911a795%26itag%3D5%26source%3Dblogger%26app%3Dblogger%26cmo%3Dsensitive_content%253Dyes%26ip%3D0.0.0.0%26ipbits%3D0%26expire%3D1421596483%26sparams%3Did,itag,source,ip,ipbits,expire%26signature%3D9BDB5665F6D7733F472AF2C3EC92062787A6B748.3212499AC4104F49A3E8D4642951DFBE974E27B8%26key%3Dck2&amp;iurl=http://video.google.com/ThumbnailServer2?app%3Dblogger%26contentid%3Db6ef51add911a795%26offsetms%3D5000%26itag%3Dw160%26sigh%3DvOcyKUcNdZQknhX1XSdl7AOYk3E&amp;autoplay=0&amp;ps=blogger"><embed src="//www.youtube.com/get_player" type="application/x-shockwave-flash" width="320" height="266" bgcolor="#FFFFFF" flashvars="flvurl=http://redirector.googlevideo.com/videoplayback?id%3Db6ef51add911a795%26itag%3D5%26source%3Dblogger%26app%3Dblogger%26cmo%3Dsensitive_content%253Dyes%26ip%3D0.0.0.0%26ipbits%3D0%26expire%3D1421596483%26sparams%3Did,itag,source,ip,ipbits,expire%26signature%3D9BDB5665F6D7733F472AF2C3EC92062787A6B748.3212499AC4104F49A3E8D4642951DFBE974E27B8%26key%3Dck2&iurl=http://video.google.com/ThumbnailServer2?app%3Dblogger%26contentid%3Db6ef51add911a795%26offsetms%3D5000%26itag%3Dw160%26sigh%3DvOcyKUcNdZQknhX1XSdl7AOYk3E&autoplay=0&ps=blogger" allowFullScreen="true" /></object> </div>

Whoa! So what happened to the game? The ground, runner and enemies are still there, we are just not rendering them anymore with the `Box2dDebugRenderer`. That means the game started, an enemy came towards the runner and probably hit him without us noticing :D. 

### Add Texture to the Ground
For the ground, I will pretty much follow the same logic we used for the background. This will be our image for the ground:

[![]({{ page.assets_url }}/ground.png)]({{ page.assets_url }}/ground.png)

I know that I should have all my textures in an atlas but I am making an exception with the ground so I can have a good image quality since I will stretch the image a lot. Let's keep track of the image name (`ground.png` inside the `assets` directory) in our `Constants` class along with the previous one:

```java
package com.gamestudio24.cityescape.utils;

import com.badlogic.gdx.math.Vector2;

public class Constants {

    ...

    public static final String BACKGROUND_IMAGE_PATH = "background.png";
    public static final String GROUND_IMAGE_PATH = "ground.png";   

}
```

Now, one thing to note is that when we rendered the `Background` we used rectangles the size of the screen to know where to render. That was fine since we got a nice image which has the same size as the screen measurements we are working with. We don't really want to do that for the other `GameActor` objects since we will be working with smaller images. In order to know the current `Rectangle` that we are supposed to render in screen coordinates, I am going to add a `screenRectangle` to my `GameActor` and update it in its `act()` method. I'm also going to check if the `GameActor`'s `Body` was destroyed by the `GameStage` (this happens when an `Enemy` or the `Runner` go out of bounds:  

```java
package com.gamestudio24.cityescape.actors;

import com.badlogic.gdx.math.Rectangle;
import com.badlogic.gdx.physics.box2d.Body;
import com.badlogic.gdx.scenes.scene2d.Actor;
import com.gamestudio24.cityescape.box2d.UserData;
import com.gamestudio24.cityescape.utils.Constants;

public abstract class GameActor extends Actor {

    protected Body body;
    protected UserData userData;
    protected Rectangle screenRectangle;

    public GameActor() {

    }

    public GameActor(Body body) {
        this.body = body;
        this.userData = (UserData) body.getUserData();
        screenRectangle = new Rectangle();
    }

    @Override
    public void act(float delta) {
        super.act(delta);

        if (body.getUserData() != null) {
            updateRectangle();
        } else {
            // This means the world destroyed the body (enemy or runner went out of bounds)
            remove();
        }

    }

    public abstract UserData getUserData();

    private void updateRectangle() {
        screenRectangle.x = transformToScreen(body.getPosition().x - userData.getWidth() / 2);
        screenRectangle.y = transformToScreen(body.getPosition().y - userData.getHeight() / 2);
        screenRectangle.width = transformToScreen(userData.getWidth());
        screenRectangle.height = transformToScreen(userData.getHeight());
    }

    protected float transformToScreen(float n) {
        return Constants.WORLD_TO_SCREEN * n;
    }

}
```

I mentioned before that I am translating 1 meter as 32 pixels. You can translate the units using whatever ratio works for you. The value goes in the `Constants`: 

```java
package com.gamestudio24.cityescape.utils;

import com.badlogic.gdx.math.Vector2;

public class Constants {

    public static final int APP_WIDTH = 800;
    public static final int APP_HEIGHT = 480;
    public static final float WORLD_TO_SCREEN = 32;

    ...
```

Since we are going to be working with the body's position, we should change the way we create our `Ground` in `WorldUtils`. Before doing so, we have to match the `GroundUserData` constructor with its parent's to store the `width` and `height`: 

```java
package com.gamestudio24.cityescape.box2d;

import com.gamestudio24.cityescape.enums.UserDataType;

public class GroundUserData extends UserData {

    public GroundUserData(float width, float height) {
        super(width, height);
        userDataType = UserDataType.GROUND;
    }

}
```

Now we can make the change in `WorldUtils`: 

```java
package com.gamestudio24.cityescape.utils;

import ...

public class WorldUtils {

    ...

    public static Body createGround(World world) {
        ...
        body.setUserData(new GroundUserData(Constants.GROUND_WIDTH, Constants.GROUND_HEIGHT));
        shape.dispose();
        return body;
    }

    ...

}
```

And now we apply pretty much the same logic we applied to the background in order to render the `Ground` with the exception of the texture loading (Note that I'm being lazy and should reuse the same methods from the `Background` class instead of copying them): 

```java
package com.gamestudio24.cityescape.actors;

import com.badlogic.gdx.Gdx;
import com.badlogic.gdx.graphics.Texture;
import com.badlogic.gdx.graphics.g2d.Batch;
import com.badlogic.gdx.graphics.g2d.TextureRegion;
import com.badlogic.gdx.math.Rectangle;
import com.badlogic.gdx.physics.box2d.Body;
import com.gamestudio24.cityescape.box2d.GroundUserData;
import com.gamestudio24.cityescape.utils.Constants;

public class Ground extends GameActor {

    private final TextureRegion textureRegion;
    private Rectangle textureRegionBounds1;
    private Rectangle textureRegionBounds2;
    private int speed = 10;

    public Ground(Body body) {
        super(body);
        textureRegion = new TextureRegion(new Texture(Gdx.files.internal(Constants.GROUND_IMAGE_PATH)));
        textureRegionBounds1 = new Rectangle(0 - getUserData().getWidth() / 2, 0, getUserData().getWidth(),
                getUserData().getHeight());
        textureRegionBounds2 = new Rectangle(getUserData().getWidth() / 2, 0, getUserData().getWidth(),
                getUserData().getHeight());
    }

    @Override
    public GroundUserData getUserData() {
        return (GroundUserData) userData;
    }

    @Override
    public void act(float delta) {
        super.act(delta);
        if (leftBoundsReached(delta)) {
            resetBounds();
        } else {
            updateXBounds(-delta);
        }
    }

    @Override
    public void draw(Batch batch, float parentAlpha) {
        super.draw(batch, parentAlpha);
        batch.draw(textureRegion, textureRegionBounds1.x, screenRectangle.y, screenRectangle.getWidth(),
                screenRectangle.getHeight());
        batch.draw(textureRegion, textureRegionBounds2.x, screenRectangle.y, screenRectangle.getWidth(),
                screenRectangle.getHeight());
    }

    private boolean leftBoundsReached(float delta) {
        return (textureRegionBounds2.x - transformToScreen(delta * speed)) <= 0;
    }

    private void updateXBounds(float delta) {
        textureRegionBounds1.x += transformToScreen(delta * speed);
        textureRegionBounds2.x += transformToScreen(delta * speed);
    }

    private void resetBounds() {
        textureRegionBounds1 = textureRegionBounds2;
        textureRegionBounds2 = new Rectangle(textureRegionBounds1.x + screenRectangle.width, 0, screenRectangle.width,
                screenRectangle.height);
    }

}
```

Now run the game and you should see the `Background` getting rendered along with the `Ground` just like it's shown on the video below: 

<div style="text-align: center;"><object width="320" height="266" class="BLOG_video_class" id="BLOG_video-b758ffe285faf2cb" classid="clsid:D27CDB6E-AE6D-11cf-96B8-444553540000" codebase="http://download.macromedia.com/pub/shockwave/cabs/flash/swflash.cab#version=6,0,40,0"><param name="movie" value="//www.youtube.com/get_player"><param name="bgcolor" value="#FFFFFF"><param name="allowfullscreen" value="true"><param name="flashvars" value="flvurl=http://redirector.googlevideo.com/videoplayback?id%3Db758ffe285faf2cb%26itag%3D5%26source%3Dblogger%26app%3Dblogger%26cmo%3Dsensitive_content%253Dyes%26ip%3D0.0.0.0%26ipbits%3D0%26expire%3D1421596483%26sparams%3Did,itag,source,ip,ipbits,expire%26signature%3D958DFE0010FF9D5FB813FE7428D2775D1EB19EB7.4A29AC29C99297F1B6C619F18AEEFA4DF076D641%26key%3Dck2&amp;iurl=http://video.google.com/ThumbnailServer2?app%3Dblogger%26contentid%3Db758ffe285faf2cb%26offsetms%3D5000%26itag%3Dw160%26sigh%3DEYlfHsqxE8EGf8h5XsV_DsasWkc&amp;autoplay=0&amp;ps=blogger"><embed src="//www.youtube.com/get_player" type="application/x-shockwave-flash" width="320" height="266" bgcolor="#FFFFFF" flashvars="flvurl=http://redirector.googlevideo.com/videoplayback?id%3Db758ffe285faf2cb%26itag%3D5%26source%3Dblogger%26app%3Dblogger%26cmo%3Dsensitive_content%253Dyes%26ip%3D0.0.0.0%26ipbits%3D0%26expire%3D1421596483%26sparams%3Did,itag,source,ip,ipbits,expire%26signature%3D958DFE0010FF9D5FB813FE7428D2775D1EB19EB7.4A29AC29C99297F1B6C619F18AEEFA4DF076D641%26key%3Dck2&iurl=http://video.google.com/ThumbnailServer2?app%3Dblogger%26contentid%3Db758ffe285faf2cb%26offsetms%3D5000%26itag%3Dw160%26sigh%3DEYlfHsqxE8EGf8h5XsV_DsasWkc&autoplay=0&ps=blogger" allowFullScreen="true" /></object></div>

The game looks really good so far, even though we lost our characters :D. Let's work with that on the next part.

If you have any questions or comments please let me know in the comments section. Please keep in mind that this was my way of solving the box2d to scene2d graphics conversion and I know that there are probably better ways to implement these kind of animations.

[Click here](http://www.williammora.com/2014/06/a-running-game-with-libgdx-part-5.html) for part 5. Cheers!