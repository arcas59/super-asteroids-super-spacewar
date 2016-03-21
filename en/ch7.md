# SUPERPOWERS TUTORIAL #4
## SUPER ASTEROIDS and SUPER SPACEWAR, Chapter 7

### **Writing Ship and Missile behavior**

#### Initialize ship whit Game.start

Before to write the ship behavior, we need to write when the ship appear in the game (spawn). We do that in the Game.start function of the global script.

We check first what game have been chosen from Menu in nameIndex variable and set the game accordingly.

```ts
[...]
namespace Game{
  [...]
  export function start(){
    [...]
    // If the game is Asteroids, load the Ship 1 only
    if (nameIndex === 0) {
      // Set ship 1 in game
      setShip(Ships.index.ship1);
      // Set visible false the HUD display for ship 2
      Sup.getActor("HUD").getChild("Ship2").setVisible(false);
      // Set visible true the HUD display for alien
      Sup.getActor("HUD").getChild("Alien").setVisible(true);
      // Set Timer HUD visible false
      Sup.getActor("HUD").getChild("Timer").setVisible(true);
    }
    // If the game is Spacewar, load two ships
    if (nameIndex === 1) {
      // Set ship 1 in game
      setShip(Ships.index.ship1);
      // Set ship 2 in game
      setShip(Ships.index.ship2);
      // Set visible true the HUD display for ship 2
      Sup.getActor("HUD").getChild("Ship2").setVisible(true);
      // Set visible false the HUD display for alien
      Sup.getActor("HUD").getChild("Alien").setVisible(false);
      // Set Timer HUD visible false
      Sup.getActor("HUD").getChild("Timer").setVisible(false);
    }
[...]
```
It won't work yet because we need the setShip local function which is used to load the Ship actor in the game.

#### the setShip funtion

We write a new function setShip which is not exported because we will use it only from the local module Game.

```ts
[...]
namespace Game{
  function setShip(shipIndex:number){
    // Initialize a new variable of type Sup.Actor
    let Ship: Sup.Actor;
    // Add the ship 1 or 2 to game scene depending the index
    if (shipIndex === 0) {
      // Create Ship and set the ship variable with the Ship1 actor
      Ship = Sup.appendScene("Ship/0/Prefab")[0];
    }
    else {
      // Create Ship and set the ship variable with the Ship2 actor
      Ship = Sup.appendScene("Ship/1/Prefab")[0];
    }
    // Set behavior variables
    Ship.getBehavior(ShipBehavior).index = shipIndex;
    // Set spawn position of the ship accordingly to the game an ship index
    // If game is Asteroids, set position to center
    if (nameIndex === 0) {
      Ship.setLocalPosition(Ships.spawns[0]);
    }
    // If game is Spacewar and Ship is 1, set position to corner down left
    else if (shipIndex === 0) {
      Ship.setLocalPosition(Ships.spawns[1]);
    }
    // Else it is Ship 2, set position to corner up right
    else {
      Ship.setLocalPosition(Ships.spawns[2]);
    }
    // Set the ship default size to half size
    Ship.setLocalScale(Ships.size, Ships.size, Ships.size);
    // Set the ship default body to half size
    Ship.getChild("Body").arcadeBody2D.setSize(Ships.body, Ships.body);
  }
}
[...]
```
To test this code we also need to add index:number; in the ShipBehavior class of the Ship script.

We don't have menu yet to choose game, we still can do test by changing temporarily the nameIndex variable.

If we want to try the Asteroids game :

```ts
[...]
  export let nameIndex: number = 0; // Temporary allocation
[...]
```

If we want to try the Spacewar game :

```ts
[...]
  export let nameIndex: number = 0; // Temporary allocation
[...]
```

#### the shipBehavior class

##### ship datas

We open the Ship script and first initialize the ship local datas we will use in our code.

```ts
[...]
class ShipBehavior extends Sup.Behavior {
  // Ship index, 0 is ship 1, 1 is ship 2
  index: number;
  // Ship current life
  lifes: number;
  // Ship current score
  score: number;
  // Spawn position
  spawnPosition: Sup.Math.Vector2;
  // Current position
  position: Sup.Math.Vector2;
  // Current movement speed
  linearVelocity = new Sup.Math.Vector2();
  // Current rotation speed
  angularVelocity: number;
  // Angle position
  angle: number;
  // Timer before shooting
  shootCooldown: number;
  // Timer before respawn
  spawnCooldown: number;
  // Timer before vulnerability
  invincibilityCooldown: number;
  // Timer before gameover
  gameOverCooldown: number;
  awake() {
  }
  update() {
  }
}
Sup.registerBehavior(ShipBehavior);
[...]
```

##### ship awakening

When the ship awake, we need to set some variables.

```ts
[...]
  awake() {
    // Starting life to 3
    this.lifes = Ships.startLife;
    // Starting score to 0
    this.score = Ships.startScore;
    // Starting speed movement on x and y axis to 0
    this.linearVelocity.set(0, 0);
    // Starting speed rotation to 0
    this.angularVelocity = 0;
  }
[...]
```  

##### ship starting

Then when the actor is loaded, we can get its parameters and set the local variables of the behavior.
  
```ts
[...]
  start() {
    // Get the starting position to become the spawnPosition of this behavior
    this.spawnPosition = this.actor.getLocalPosition().toVector2();
    // Get the starting position to become the current position of this behavior
    this.position = this.actor.getLocalPosition().toVector2();
    // Get the starting angle to become the current angle of this behavior
    this.angle = this.actor.getLocalEulerZ();
  }
[...]
``` 
##### ship death

We define a method for the death of the player ship.

*Note : Sup.setTimeout is used to have a delay before to jump to the gameOver menu screen.*

```ts
[...]
  die() {
    // Decrease life of one
    this.lifes--;
    // Flag to check and update the life HUD
    Game.checkLifeHUD = true;
    // If life is 0, then the game is over
    if (this.lifes === 0) {
      // If this is the Asteroids game, death of ship mean victory for Alien
      if(Game.nameIndex === 0) {
        Sup.setTimeout(1000, function () { Game.gameOver("alien") });
      }
      else {
        // If this is Spacewar game and death of ship 1 mean victory for ship 2
        if (this.index === 0){
          Sup.setTimeout(1000, function () { Game.gameOver("ship2") });
        }
        // Else, this is ship 2 and it is a victory for ship 1
        else {
          Sup.setTimeout(1000, function () { Game.gameOver() });
        }
      }
    }
    // Check which ship index to see which lose points
    if (this.index === 0) {
      Game.addPoints(true, Game.points.death);
    }
    else {
      Game.addPoints(false, Game.points.death); 
    }
    // Flag to check and update the score HUD
    Game.checkScoreHUD = true;
    // Set timer before respawn
    this.spawnCooldown = Ships.respawnTimer;    
    // Set ship model visibility to false
    this.actor.getChild('Model').setVisible(false);
    // Set sprite animation explosition to play once
    this.actor.getChild('Destruction').spriteRenderer.setAnimation("explode", false);
  }
[...]
``` 

We don't forget to comment the Game.addPoints calls to do tests later without bug.

##### ship respawn

```ts
[...]
  spawn() {
    // The ship respawn to spawn position
    this.position = this.spawnPosition.clone();
    // The ship model visibility to true
    this.actor.getChild('Model').setVisible(true);
    // Set timer for invincibility
    this.invincibilityCooldown = Ships.invincibleTimer;
  }
[...]
``` 


##### ship shooting

```ts
[...]  
shoot() {
    // Initialize a new missile
    let missile: Sup.Actor;
    // If the ship is ship 1 then create a Ship 1 missile and set the variable missile to the Missile actor
    if (this.index === 0) {
      missile = Sup.appendScene("Ship/0/Missile/Prefab")[0];
    } 
    // Else do the same but for the missile of ship 2
    else {
      missile = Sup.appendScene("Ship/1/Missile/Prefab")[0];
    }
    // Set position of the actor to the current position of the ship
    missile.setLocalPosition(this.position);
    // Set local variables of the missile behavior
    // Report the position of the ship to the variable position of the behavior
    missile.getBehavior(ShipMissileBehavior).position = this.position.clone();
    // Report the angle of the ship to the angle direction of the missile
    missile.getBehavior(ShipMissileBehavior).angle = this.angle;
    // Set the shipIndex related to this missile
    missile.getBehavior(ShipMissileBehavior).shipIndex = this.index;
    // Set Shooting timer to be able to shoot again
    this.shootCooldown = Ships.shootingTimer;
  }
[...]
``` 

To make it work, we will need to set the local variable of the Missile behavior in Ship/Missile script.

```ts
class ShipMissileBehavior extends Sup.Behavior {
  shipIndex: number;
  position: Sup.Math.Vector2;
  angle: number;
[...]
```

##### ship boost

We add a little sprite when the ship is moving with this method.

```ts
[...]
  boost(intensity: string) {
    // Create a new variable boost that get the Boost actor child of Ship actor
    let boost:Sup.Actor = this.actor.getChild("Boost");
    // Set the boost visible true
    boost.setVisible(true);
    // Set animation to both sprit with the intensity normal or fast
    boost.getChild("0").spriteRenderer.setAnimation(intensity);
    boost.getChild("1").spriteRenderer.setAnimation(intensity);
  }
[...]
```

##### ship process

Now we have all our methods ready, we can write the whole process of the player ship.

* Keep respawning, will keep looping and decrease spawn timer
* Keep shooting, call the shoot method when shoot key pressed
* Keep moving, move the ship when key pressed
* Keep rotating, rotate the ship when key pressed
* Keep slowing down, slow down the acceleration of linear and angular velocities
* Stay on the game screen, same as asteroids and alien ship
* Update position and angle, report behavior change to the Actor
* Blinking, to display ship invincibility

```ts
[...]
  update() {
    // Keep respawning
    // If the spawnCooldown timer is more than 0
    if (this.spawnCooldown > 0) {
      // Decrease by one the timer
      this.spawnCooldown--
      // If the spawnCooldown is 0
      if (this.spawnCooldown === 0) {
        // Call the spawn method
        this.spawn();
      }
      //restart update loop to skip the following code
      return;
    }
    
    // Keep shooting
    // If the shootCooldown timer is more than 0
    if (this.shootCooldown > 0) {
      // Decrease by one the timer
      this.shootCooldown--;
    }
    // If the timer is 0 (!0 = true)
    if (!this.shootCooldown) {
      // If the shoot key is pressed
      if(Sup.Input.wasKeyJustPressed(Ships.commands[this.index].shoot)){
        // Call the shoot method
        this.shoot();
      }
    }
    
    // Keep moving
    // If forward key is pressed down
    if (Sup.Input.isKeyDown(Ships.commands[this.index].forward)){
      // Set the impulse with the linearAcceleration
      let impulse = new Sup.Math.Vector2(Ships.linearAcceleration, 0);
      // Convert the impulse to the current angle
      impulse.rotate(this.angle);
      // Add the impulse to the linearVelocity
      this.linearVelocity.add(impulse);
        // Call the boost method with a normal intensity
        this.boost("normal");
        // If the boost key is pressed down
        if (Sup.Input.isKeyDown(Ships.commands[this.index].boost)) {
          // Add a second time the impulse to the linearVelocity
          this.linearVelocity.add(impulse);
        // Call the boost method with a fast intensity
          this.boost("fast");
        }
    }
    else {
      // If the forward key is not pressed, the Boost actor set visible false
      this.actor.getChild("Boost").setVisible(false);
    }
    
    // Keep rotating
    // If left key is pressed down
    if (Sup.Input.isKeyDown(Ships.commands[this.index].left)){
      // The angularVelocity get the angularAcceleration
      this.angularVelocity += Ships.angularAcceleration;
    }
    // If right key is pressed down
    if (Sup.Input.isKeyDown(Ships.commands[this.index].right)){
      // The angularVelocity get the opposite angularAcceleration
      this.angularVelocity -= Ships.angularAcceleration;
    }
    
    // Keep slowing down
    // The linearVelocity multiply the linearDamping
    this.linearVelocity.multiplyScalar(Ships.linearDamping);
    // The angularVelocity multiply the angularDamping
    this.angularVelocity *= Ships.angularDamping;
    
    // Stay on the game screen
    if (this.position.x > Game.bounds.width / 2) {
      this.position.x = -Game.bounds.width / 2;
    }
    
    if (this.position.x < -Game.bounds.width / 2) {
      this.position.x = Game.bounds.width / 2;
    }
    
    if (this.position.y > Game.bounds.height / 2) {
      this.position.y = -Game.bounds.height / 2;
    }
    
    if (this.position.y < -Game.bounds.height / 2) {
      this.position.y = Game.bounds.height / 2;
    }
    
    // Update position and angle
    // Add the linearVelocity to the current position
    this.position.add(this.linearVelocity);
    // Set the new current position to the Ship actor
    this.actor.setLocalPosition(this.position);
    // Add the angularVelocity to the current angle
    this.angle += this.angularVelocity;
    // Set the new angle to the Ship actor
    this.actor.setLocalEulerZ(this.angle);
    
    // Blinking
    // If the invincibilityCooldown Timer is more than 0
    if (this.invincibilityCooldown > 0) {
      // Decrease by one the timer
      this.invincibilityCooldown--;
      // Set actor visible become true every half second, false the other half and stay visible at the end
      this.actor.setVisible(this.invincibilityCooldown % 60 < 30);
      // Restart update loop to skip the collision code
      return;
    }
    
    // Collision code will come here
    if (Sup.Input.wasKeyJustPressed("X")){
      this.die(); // Temporary debug test for ship death, press keyboard X to kill the ship
    }
  }
[...]
``` 

We now have our player ship under control, moving, rotating, slowing down, shooting (but the missile don't have behavior).

We need now to give some behavior to the missiles.

#### Ship missiles
```ts
[...]
[...]
``` 

#### Alien missiles
```ts
[...]
[...]
``` 

Now we have everything we need in place to start an important game mechanic, the collision system, we will see that in the next chapter.