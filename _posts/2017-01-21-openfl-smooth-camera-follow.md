---
layout: post
published: false
title: 'OpenFL: Smooth camera follow'
subtitle: Everything is possible with linear interpolation
---
You've probably heard of [OpenFL](http://www.openfl.org/); it's basically a framework to code visual applications (2D games, basically) written in [Haxe](http://haxe.org) with an API extremely similar to *ActionScript 3*, in Flash. Quality OpenSource content! 

After reading a book or some tutorial, you're now and expert on *Haxe* and *OpenFL*. You're coding your awesome platformer videogame, and you need a camera which follows the main character, centering the attention on the player's character.

Let's say we have a player class called `Player` (*such imagination*), which moves across the screen when pressing some buttons. Something like this:

```hx
// Of course, this is a sample snippet, which may not event compile.
class Player extends Sprite {
  // Some cool bitmap and resources
  private var _bitmap: Bitmap;
  
  public function new() {
    // Load resources
    _bitmap = new Bitmap(Assets.getBitmapData("img/player.png"));
    // ...
  }
  
  // Apply some velocity to the object.
  public function move(vx: Float, vy: Float) {
    this.x += vx;
    this.y += vy;
  }
  
  // Update method, called on each frame.
  public function void update() {
    // Handle events, etc
  }
}
```

Our player will move through a world; some class called `World` (*so inteligent*):

```hx
class World extends Sprite {
  // Here's our player reference
  public var player: Player;
  
  public function new() {
    // Create a player, and add it to the scene.
    player = new Player();
    addChild(player);
  }
  
  // Update method, called on each frame.
  public function void update() {
    // Delegate to player
    player.update();
    
    // ...
  }
  
  // ...
}
```

Our `Player` can now move across the screen! But, since the camera here is the screen, which is static, if the player moves, it'll actually move across the screen, instead of being always centered. To make the camera *"move"*, we need to actually *"move everything on screen"*; i.e. instead of moving the camera **to the right**, me move the world **to the left**, creating the illusion that the world has moved. Tricky, isn't it? This is concept is present in almost every game. To understand why this is like it's, you may want to check [this StackOverflow question](http://gamedev.stackexchange.com/questions/40741/why-do-we-move-the-world-instead-of-the-camera).

Now, if we want the player to be followed by *"the camera"* (note the quotes, since there's really not any cameras), we need to move the `World` while the `Player` moves. One approach is to create a reference to the world in the `Player` class, and update it's position with something like this:

```hx
class Player extends Sprite {
  // World reference
  private var _world: World;

  public function new(world: World) {
    _world = world;
    // ...
  }

  public function move(vx: Float, vy: Float) {
    this.x += vx;
    this.y += vy;

    // Move the world in the opposite direction to
    // center the main character on the screen.
    _world.x -= vx;
    _world.y -= vy;
  }

  // ...
}
```

But that's nasty, don't you think? Why should the player care about moving the world when it moves? Nah, that's not a good solution at all.

We need to move that behaviour away, and set a unique class whose only purpose it's to update the *"camera"* position; a class called `Camera` (*so unique*):

```hx
// Since the camera is not visual (only logical), it's
// not necessary to inherit it from `Sprite`.
class Camera {
  private var _player: Player;
  private var _world: World;

  // Keep track of the target (player) and the scene (world).
  public function new(player: Player, world: World) {
    _player = player;
    _world = world;
  }

  // Update the world's position to center the player.
  public function update() {
    // Do something to move the world and center the player on
    // the screen.
    // ...
  }
  
  // ...
}

// We also need to update the World to include the camera:
class World extends Sprite {
  // ...
  
  private var _camera: Camera;
  
  public function new() {
    _player = new Player();
    _camera = new Camera(_player, this);
    // ...
  }

  public function void update() {
    player.update();
    camera.update();
  }
  
  // ...
}
```

Here's the tricky part; how does `Camera.update()` work?

...