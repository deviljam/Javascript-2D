# Javascript-2D
Simple 2D tile based engine for Javascript

This simple engine will allow you to create games with sound, graphics and physics using Javascript's Canvas 2d.

Getting started.

First create a canvas element on your page. These steps are done for you in the provided index.html file.
<canvas id="game" width="512" height="480"></canvas>

Then initalise a game object.
window.game = new Game( document.getElementById( "game" ) );

A level is represented as an array of arrays.

game.tiles = [[...],[...]]
Each array is a layer.

Collision, by default is detecred by none-zero tiles the second layer. game.tiles[1].

The simplest level you can create is:
var tileSize = 16;
game.tiles = [[1,1,1,1,0,1,1,1,1],[1,1,1,1,0,1,1,1,1]];
game.tileDimension = new Line(
		0,0,3,3
);
game.bounds = game.tileDimension.scale(tileSize);

The variable, tileDimension specifies the starting position width and height of the level. We have 
specified above that the level starts at (0,0) and is 3 tiles wide, and 3 tiles tall. The bounds 
specifies the space in pixels of level, this is used to create a binary space partition for more 
efficient object look ups.

Now lets load in some graphics.
For simplicity sake, create an object.
window.sprites = {
  "tiles" : new Sprite("/img/tiles.png", {offset:new Point(0,0), width:tileSize, height:tileSize}),
  "player" : new Sprite(/img/player.png", {offset:new Point(8,8), width:16, height:16)
}
window.game.tileSprite = window.sprites.tiles;
Assuming you have a tile sheet you wish to use, you should now see some a simple doughnut shaped level.
When creanting a new Sprite object, remember the parameters: Sprite(urlForFile, options)
Some options include:
offset <Point> specifies the center of the sprite, 0,0 is the top left corner of the sprite.
width <int> If you want to divide the sprite into a tile sheet, this is the width of each tile.
height <int> If you want to divide the sprite into a tile sheet, this is the height of each tile.

Lets create an object for our game.

Player.prototype = new GameObject();
Player.prototype.constructor = GameObject;
function Player(x, y){	
	this.constructor();
	
	this.position.x = x;
	this.position.y = y;
	
	this.width = 14;
	this.height = 14;
	
	this.sprite = window.sprites.player;
}

This object will extend the GameObject class, the game object class contains many useful functions needed
for this engine to run.

We need to create some logic...

Player.prototype.update = function(){
  var force = new Point();
  var speed = 2;
  if( input.state("left") > 0 ) force.x -= speed;
  if( input.state("right") > 0 ) force.x += speed;
  if( input.state("up") > 0 ) force.y -= speed;
  if( input.state("down") > 0 ) force.y += speed;
  
  window.game.t_move( this, force.x, force.y );
}

We now have a basic object we can more. The "input" object is an object that tracks the user's input. 
You can customise these controls in the input.js file.

We need to add this object to our game. so...

window.game.addObject( new Player(16,16); );

Refresh, and you should now have a player in the center of our dougnut. It will bounce against the walls, 
but there's not a lot of room for this character to move. So lets change our level size.

var tileSize = 16;
game.tiles = [[1,1,1,1,1,1,0,0,0,1,1,0,0,0,1,1,0,0,0,1,1,1,1,1,1],[1,1,1,1,1,1,0,0,0,1,1,0,0,0,1,1,0,0,0,1,1,1,1,1,1]];
game.tileDimension = new Line(
		0,0,5,5
);
game.bounds = game.tileDimension.scale(tileSize);

We have a little more room to move. But the camera does move. Let's create a module for our camera.

window.mod_camera = {
  "init" : function() {},
  "update" : function(){
    window.game.camera.x = this.position.x - 128; //player position minus half the screen size
    window.game.camera.y = this.position.y - 120; //player position minus half the screen size
  }
}

We need to attach this to our player. So in our player's constuctor we add the line.

...
  this.position.x = x;
	this.position.y = y;
	
	this.addModule(mod_camera);
...

When we refresh, the player should fixed in the center of the screen.

That's the basics of getting the engine set up. Some other features worth noting.

You can which sprite in the sprite sheet to use for an object with:
this.frame = 0;
this.frame_row = 0;

Objects will go to sleep (stop executing their script) if they leave the bounds of the screen.
You can prevent this with.

player.prototype.idle = function(){}

You can remove objects from the game world with.
this.destroy();

You can attach listeners and trigger events with.
this.on("something", function(a,b){ console.log(a); console.log(b); /*outputs "some" and "varaibles here" */ } );
this.trigger("something", "some" "variables here");

There are some predefied triggers, such as...
"added":
Called when object is first added to the game world

"objectCollide": object_this_collided_with <GameObject>
Called when another object moves over this object

"collideHorizontal": direction <float>
Called when object bumps into a wall while moving laterally, the direction defines what direction the object was 
trying to move in.

"collideVertical": direction <float>
Called when object bumps into a wall while moving vertically, the direction defines what direction the object was 
trying to move in.

"awake": 
Called when the object enters the screen

"sleep":
Called when object exits the screen
