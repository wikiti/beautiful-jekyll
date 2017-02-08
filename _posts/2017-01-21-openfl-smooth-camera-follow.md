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
  public function applyVelocity(vx: Float, vy: Float) {
    this.x += vx;
    this.y += vy;
  }
  
  // Update method, called on each frame.
  public function void onUpdate() {
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
    player = new Player();
  }
  
  // Update method, called on each frame.
  public function void onUpdate() {
    // Delegate to player
    player.onUpdate();
  }
  
  // ...
}
```

Our `Player` can now move across the screen! But, since the camera here is the screen, which is static, if the player moves, it'll actually move across the screen, instead of being always centered. To make the camera *"move"*, we need to actually *"move everything on screen"*; i.e. instead of moving the camera **to the right**, me move the world **to the left**, creating the illusion that the world has moved. Tricky, isn't it? This is concept is present in almost every game. To understand why this is like it's, you may want to check [this StackOverflow question](http://gamedev.stackexchange.com/questions/40741/why-do-we-move-the-world-instead-of-the-camera).

Now, if we want the player to be followed by *"the camera"* (note the quotes, since there's really not any cameras), we need to move the `World` while the `Player` moves. But that's nasty, don't you think? Why should the world care about ...