Collision to see for alien ship script alienbehavior
# SUPERPOWERS TUTORIAL #4
## SUPER ASTEROIDS and SUPER SPACEWAR, Chapter 8

### **Writing Collision, Life and Score systems.**

To give a substance to all our actors and give the main gameplay to our game we need to set the collision system without which the actors would never interact to each other.

#### Collision system

##### The Game.checkCollisionMissile function

We add a new function that we will call each time we need to check a collision, we put it in the Module Game of the Global script.

```ts
namespace Game{
[...] 
  export function checkCollisionMissile(actor: Sup.Actor, actorList: Sup.Actor[], amplitude: number){
    /* 
    We get the distance between actor and all the actors of the actorList
    If the distance if inferior to the amplitude box around the Actor, there is a collision
    */
    
    // Get the position 1 of the actor we check collision with actorList
    let position1: Sup.Math.Vector2 = actor.getLocalPosition().toVector2();
    // Loop through the actors of listToCheck and check if ....
    for(let i = 0; i < actorList.length; i++){
      // Get the position 2 of the current actor of the loop inside actorList
      let position2: Sup.Math.Vector2 = actorList[i].getLocalPosition().toVector2();
      // Get the distance between position 1 and position 2
      let distance: number = Math.round(position1.distanceTo(position2)*100)/100;
      // If the distance is inferior to the amplitude (collision radius), then it is a collision
      if (distance < amplitude) {
        // The current actor of the actorList is destroyed
        actorList[i].destroy();
        // Return true to the behavior which check for collision
        return true;
      }
    }
    // Return false to the behavior which check for collision
    return false;
  }
[...]
```

##### The Game.checkCollisionAsteroid function

##### The Game.checkAsteroidShip function


##### Collision alien ship with player missile

Inside the update loop of the Alien script, we call a Game.checkCollisionMissile with the ship missiles of the player.

```ts
[...]  
  update() {
  [...]
    // If there is player ship1 missiles, call a collision checking for this actor with the list of the player Ship missiles
    if(Ships.missiles[0].length > 0){
      // If the collision checking return true
      if (Game.checkCollisionMissile(this.actor, Ships.missiles[0], this.amplitude)) {
        // Decrease Alien lifes by one
        Alien.lifes--
        // Flag true to check and update the life HUD in Game Script
        Game.checkLifeHUD = true;
      }
    }
  }
[...]
```

##### Collision asteroid with player missile

Inside the update loop of the Asteroid script, we call a Game.checkCollisionMissile with the ship missiles of the player. If the collision is true the asteroid is destroyed and two new asteroids spawn.

```ts
[...]
  update() {
  [...]
    // If there is player ship1 missiles, call a collision checking for this actor with the list of the player Ship missiles
    if(Ships.missiles[0].length > 0){
      // If the collision checking return true
      if (Game.checkCollisionMissile(this.actor, Ships.missiles[0], this.amplitude)) {
        // Destroy this asteroid
        this.die();
        // If the asteroid is not small
        if (this.sizeClass !== "small"){
          // Spawn two new asteroid of the class below
          this.spawnChildren();
        }
      }
    }
  }
[...]
```

##### Collision player ship with asteroids

Inside the Ship script, if the game is Super Asteroids we now check for collision betwen the ship and the alien missiles and all the asteroids in the update loop.

If the game is Super Spacewar we check collision with the missile of other ship or a collision with the other ship.


```ts
[...]
  update() {
  [...]
    // If game is Super Spacewar, check collision with other missiles and other ship
    if (Game.nameIndex === 0) {      
      // if (Game.checkCollisionAsteroids(this.actor, Alien.missiles, this.amplitude)){
      //   this.die();
      // }
    }
[...]
```

##### Collision player ship with alien missile

```ts
[...]
  update() {
  [...]
    // If game is Super Spacewar, check collision with other missiles and other ship
    if (Game.nameIndex === 0) {
    [...]
      if (Game.checkCollisionMissile(this.actor, Alien.missiles, this.amplitude)){
        this.die();
      }
    }
[...]
```

##### Collision player ship with other player missiles

```ts
[...]    
    // If game is Super Spacewar, check collision with other missiles and other ship
    if (Game.nameIndex === 1) {
      // If the ship is 1 check with ship 2 missile
      if (this.index === 0){
        if (Game.checkCollisionMissile(this.actor, Ships.missiles[1], this.amplitude)){
          this.die();
        }
      }
      // Else the ship is 2 check with ship 1 missile
      else {
        if (Game.checkCollisionMissile(this.actor, Ships.missiles[0], this.amplitude)){
          this.die();
        }
      }
    }
[...]
```

##### Collision player ship with other player ship

```ts
[...]
[...]
```

The collision work fine, we need now to update the Game HUD when something happen in the game.

#### Score system

##### Game.addPoints function

```ts
namespace Game{
  [...]
  // Add points to the ship score
  export function addPoints(ship1: boolean, points: number){
    // If ship1 is true, add points to ship1
    if(ship1){
      Sup.getActor("Ship1").getBehavior(ShipBehavior).score += points;
    }
    // If ship1 is false, add points to ship2
    else {
      Sup.getActor("Ship2").getBehavior(ShipBehavior).score += points;
    }
  }
}
[...]
```

##### Game.updateScore function Update HUD score

##### Game script checkHUDscore


#### Life system

#### Game.updateLife function Update HUD lifes

##### Game script checkHUDlife
