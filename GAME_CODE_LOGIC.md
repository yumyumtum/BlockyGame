# Game Code Logic Documentation

This document explains the code logic and architecture of each game in the BlockyGame collection.

## Table of Contents
- [SpaceShip](#spaceship)
- [Dodge_Games](#dodge_games)
- [FlappyPoop](#flappypoop)
- [SniperGame](#snipergame)
- [SigmaSniper](#sigmasniper)
- [DuckAdventure](#duckadventure)
- [Trivia](#trivia)

---

## SpaceShip

### Overview
A space shooter game where you control a spaceship, fight enemies, and collect power-ups.

### Core Components

#### 1. Game State Management
```javascript
let spaceship = { x, y, width, height, speed }
let bullets = []      // Player projectiles
let missiles = []     // Rocket ammunition
let enemies = []      // Enemy ships
let enemyBullets = [] // Enemy projectiles
let stars = []        // Power-up items (rocket pickups)
let assistShips = []  // Helper ships pickup
let shadowShips = []  // Active shadow ships
let hexItems = []     // Golden star pickups
```

#### 2. Player Controls
- **WASD / Arrow Keys**: Movement in 4 directions
- **SPACE**: Fire bullets (hold for laser beam)
- **'2' / Right-click**: Fire rockets
- **Left-click**: Shoot bullets

#### 3. Weapons System

**Regular Bullets:**
- Fast projectiles that travel upward
- Unlimited ammunition
- Basic damage to enemies

**Laser Beam:**
- Hold SPACE for sustained laser
- High damage output
- Cooldown period of 20 seconds
- Drains energy while active

**Rockets:**
- Limited ammunition (collect from star pickups)
- Splash damage with explosion effects
- Can kill multiple enemies in area

#### 4. Enemy Types

**Firing Enemies:**
- Move vertically and shoot projectiles
- Standard HP
- Predictable movement patterns

**Blinking Enemies:**
- Appear/disappear periodically
- Drop power-ups when killed
- Large explosion on death that can damage player

**Spinning Enemies:**
- Rotate while moving
- Split into 2 smaller enemies when destroyed
- Medium explosion effect

**Mini Enemies:**
- Small, fast-moving
- Spawned from splitting enemies
- Chase player directly

#### 5. Power-Up System

**Star Pickups (Yellow):**
- Grant rocket ammunition (+1)
- Common drop from enemies

**Assist Ships (Green):**
- Spawn helper ships that fight alongside player
- Auto-target and shoot enemies
- Limited duration

**Shadow Ships (Purple/Hexagon):**
- Temporary invincibility
- Mirror player's movements
- Visual shadow effect

**Golden Stars (Rare):**
- Special collectibles
- Bonus scoring

#### 6. Difficulty Scaling
- **Level System**: Increases every 60 seconds
- **Speed Multiplier**: Enemies get 20% faster each level (×1.2)
- **Level 5 Reset**: Speed resets at level 5, then scales again
- Applied to:
  - Enemy movement speed
  - Enemy bullet speed
  - Spawn rate adjustments

#### 7. Collision Detection
```javascript
function isColliding(a, b) {
  return !(a.x > b.x + b.width || 
           a.x + a.width < b.x || 
           a.y > b.y + b.height || 
           a.y + a.height < b.y)
}
```
- Rectangle-based collision detection
- Checked every frame for:
  - Player vs enemies
  - Player vs enemy bullets
  - Player bullets vs enemies
  - Player vs power-ups

#### 8. Explosion System
```javascript
createExplosion(x, y, size, color, canKill=true)
```
- Particle-based explosion effects
- Expanding radius animation
- Different sizes based on enemy type:
  - Blinking enemies: 240px radius (can kill player)
  - Spinning enemies: 50px radius
  - Mini enemies: 25px radius
  - Regular enemies: 40px radius
- Particles fade over time

#### 9. Visual Effects

**Background Starfield:**
- 150 stars with parallax scrolling
- Twinkling animation (sine wave)
- Multiple layers of depth

**Spaceship Animations:**
- Three sprite images: normal, left-tilt, right-tilt
- Changes based on movement direction
- Smooth transitions

**Screen Shake:**
- Triggered on explosions
- Intensity based on explosion size

#### 10. Audio System
- **Shoot sound**: Bullet firing
- **Missile sound**: Rocket launch
- **Pickup sound**: Collecting items
- **Explosion sound**: Enemy destruction
- **Laser sound**: Beam weapon
- **BGM**: 3-track rotation (background.mp3, DestroyThemAll.mp3, Starfield.mp3)

#### 11. Game Loop
```javascript
function gameLoop() {
  if (!menuActive && !gameOver && !paused) {
    update()    // Update positions, check collisions
    draw()      // Render all objects
    requestAnimationFrame(gameLoop)
  }
}
```

**Update Phase:**
1. Move player based on key input
2. Update all bullets/missiles positions
3. Update enemy positions and AI
4. Update enemy bullets
5. Process collisions
6. Update power-ups
7. Update explosions
8. Check win/lose conditions

**Draw Phase:**
1. Clear canvas
2. Draw background stars
3. Draw enemies
4. Draw player
5. Draw projectiles
6. Draw power-ups
7. Draw explosions
8. Draw UI (health, score, etc.)

#### 12. Scoring System
- Kill count tracks eliminated enemies
- Every 10 kills grants 1 rocket
- Score displayed in real-time
- Rankings saved to localStorage

---

## Dodge_Games

### Overview
A survival game where you control a yellow smiley face, dodge falling blocks, and collect power-ups to level up.

### Core Components

#### 1. Game State
```javascript
let player = { x, y, width, height, speed, health, level }
let blocks = []       // Falling obstacles
let arrows = []       // Player projectiles (level 5+)
let missiles = []     // Player missiles (level 10+)
let explosions = []   // Visual effects
let score = 0         // Blocks successfully dodged
```

#### 2. Player Controls
- **Arrow Keys / WASD**: Move in 4 directions
- **5**: Fire arrow (requires level ≥ 5)
- **1**: Fire missile (requires level ≥ 10)
- **L / Middle Mouse**: Hold for laser beam
- **H**: Toggle help overlay

#### 3. Block Types

**Red Blocks:**
- Damage player on contact (-1 HP, -1.5 HP after 30s)
- Most common block type (60%)
- Must be dodged

**Blue Blocks:**
- Heal player (+1 HP, max 2)
- Increase player level (+1)
- Should be collected (20%)

**Green Blocks:**
- Instant game over if touched
- Become orange after 60 seconds
- Must be avoided (10%)

**Yellow Blocks (Buff):**
- Regeneration buff (15 seconds)
- Heals 0.5 HP per second
- Rare (4%)

**Purple Blocks (Buff):**
- Speed boost (10 seconds)
- Doubles movement speed
- Rare (3%)

**White Blocks (Buff):**
- Invincibility (8 seconds)
- Immune to damage
- Very rare (2%)

**Pink Blocks (Buff):**
- Shield (12 seconds)
- Absorbs one hit
- Very rare (1%)

#### 4. Projectile System

**Arrows (Level 5+):**
- Fast, simple projectiles
- Destroy blocks on contact
- Fired with '5' key

**Missiles (Level 10+):**
- Slower but more powerful
- Create explosions on impact
- Can activate buffs when hitting buff blocks
- Fired with '1' key

**Laser (Any level):**
- Hold 'L' or middle mouse button
- Continuous beam that destroys blocks
- No cooldown

#### 5. Buff System

**Regeneration (Yellow):**
```javascript
regenEnd = now + 15000
healTick = every 1000ms
health += 0.5 (max 2)
```

**Speed Boost (Purple):**
```javascript
speedEnd = now + 10000
player.speed = baseSpeed * 2
```

**Invincibility (White):**
```javascript
invEnd = now + 8000
// Ignore all damage
```

**Shield (Pink):**
```javascript
shieldEnd = now + 12000
shieldActive = true
// Absorbs next hit, then deactivates
```

#### 6. Collision Logic
```javascript
function checkCollision(a, b) {
  return a.x < b.x + b.width && 
         a.x + a.width > b.x &&
         a.y < b.y + b.height && 
         a.y + a.height > b.y
}
```

**Priority Order:**
1. Check invincibility (ignore damage)
2. Check shield (absorb hit)
3. Apply damage/effect based on block type

#### 7. Scoring System
- **Score**: Number of blocks that fall off screen without collision
- **Level**: Increased by collecting blue blocks
- **Health**: 2.0 maximum, shown as bar

#### 8. Difficulty Progression
- Block spawn rate: Every 800ms
- Block speed: 2-4 pixels per frame (random)
- Speed increases slightly over time
- Green blocks become orange after 60 seconds (still deadly)

#### 9. Visual Feedback

**Player:**
- Yellow circle with black eyes and smile
- Expressions change based on state
- Flashes red when taking damage

**Warning System:**
```javascript
warningTimer = 60  // frames
// Screen flashes red briefly on hit
```

**Explosions:**
- Expanding circles with particle effects
- Green blocks create larger explosions
- Fade out over time

#### 10. Music System
- 5 background tracks rotate randomly
- Files: BANDIT.mp3, ICE_AGE.mp3, TORE_UP.mp3, Toxic.mp3, Understand.mp3
- Volume set to 0.5
- Loops continuously

#### 11. Game Over Conditions
1. Player health reaches 0
2. Player touches green block
3. Display final score and level

#### 12. Game Loop
```javascript
function gameLoop() {
  if (gameRunning) {
    updateBlocks()
    updateProjectiles()
    updateExplosions()
    updateBuffs()
    movePlayer()
    draw()
    requestAnimationFrame(gameLoop)
  }
}
```

---

## FlappyPoop

### Overview
A simplified Flappy Bird clone with a yellow bird navigating through green pipes.

### Core Components

#### 1. Game State
```javascript
let bird = { x: 80, y: 200, width: 40, height: 30, velocity: 0 }
let pipes = []
let score = 0
let gameOver = false
```

#### 2. Physics Constants
```javascript
const GRAVITY = 0.5         // Downward acceleration
const FLAP = -8            // Upward velocity on flap
const PIPE_WIDTH = 60      // Width of obstacles
const PIPE_GAP = 150       // Vertical gap between pipes
```

#### 3. Controls
- **SPACE**: Flap (jump)
  - While playing: Apply upward velocity
  - When game over: Restart game

#### 4. Bird Physics
```javascript
function update() {
  bird.velocity += GRAVITY    // Apply gravity
  bird.y += bird.velocity     // Update position
}

function flap() {
  bird.velocity = FLAP        // Set upward velocity
}
```

#### 5. Pipe Generation
```javascript
function createPipe() {
  const topHeight = Math.random() * (canvas.height - PIPE_GAP - 100) + 20
  pipes.push({
    x: canvas.width,
    top: topHeight,
    bottom: canvas.height - topHeight - PIPE_GAP
  })
}
```
- Generated every 1500ms
- Random vertical position
- Move left at constant speed (2 pixels/frame)

#### 6. Collision Detection

**Boundary Check:**
```javascript
if (bird.y + bird.height > canvas.height || bird.y < 0) {
  gameOver = true
}
```

**Pipe Collision:**
```javascript
if (bird.x < pipe.x + PIPE_WIDTH &&
    bird.x + bird.width > pipe.x &&
    (bird.y < pipe.top || 
     bird.y + bird.height > canvas.height - pipe.bottom)) {
  gameOver = true
}
```

#### 7. Scoring System
```javascript
if (!pipe.passed && pipe.x + PIPE_WIDTH < bird.x) {
  score++
  pipe.passed = true  // Prevent double counting
}
```
- Score increases by 1 for each pipe passed
- Only counted once per pipe

#### 8. Rendering
- **Background**: Sky blue (#70c5ce)
- **Bird**: Yellow rectangle (40×30)
- **Pipes**: Green rectangles (60px wide)
- **Score Overlay**: White text at top center

#### 9. Game Loop
```javascript
function gameLoop() {
  if (!gameOver) {
    update()      // Physics and collision
    draw()        // Render
    requestAnimationFrame(gameLoop)
  } else {
    // Show game over message
  }
}
```

#### 10. Game Reset
```javascript
function resetGame() {
  bird.y = 200
  bird.velocity = 0
  pipes = []
  score = 0
  gameOver = false
}
```

---

## SniperGame

### Overview
A sniper shooting game where you aim through a scope and shoot moving targets before they shoot you.

### Core Components

#### 1. Game State
```javascript
let target = {
  x, y,                    // Position
  width: 30, height: 90,   // Size
  speed: 5,                // Movement speed
  alive: true,             // Status
  nextShotTime: Date.now() + 4000  // When target will shoot
}
let bullet = null          // Player's shot
let bloodSplat = []       // Blood particles on target
let screenBlood = []      // Blood on screen (when player hit)
let gameOver = false
let message = ""
```

#### 2. Controls
- **Mouse Position**: Aim
- **Left Click**: Shoot

#### 3. Visual Elements

**Scope:**
```javascript
function drawScope() {
  // Circle: 200px radius
  // Crosshair: Horizontal and vertical lines
}
```
- White circle (200px radius)
- Crosshair at screen center
- Always centered on screen

**Target:**
```javascript
function drawRealisticTarget(x, y) {
  // Body: Green rectangle (#556B2F)
  // Head: Beige circle (#FFDAB9)
  // Arms: Beige lines extending from shoulders
  // Legs: Dark lines (#2F4F4F)
}
```
- Stick figure style
- 90 pixels tall
- Moves horizontally across screen

#### 4. Game Mechanics

**Target Movement:**
- Moves left-to-right or right-to-left
- Speed: 5 pixels per frame
- Reverses direction at screen edges
- Random vertical position changes

**Shooting:**
```javascript
canvas.addEventListener("click", (e) => {
  if (!bullet && target.alive && !gameOver) {
    bullet = { 
      x: e.clientX, 
      y: e.clientY, 
      speed: 15 
    }
  }
})
```

**Hit Detection:**
```javascript
if (bullet.x >= target.x - 15 && 
    bullet.x <= target.x + 15 &&
    bullet.y >= target.y - 45 && 
    bullet.y <= target.y + 45) {
  // Hit!
}
```

#### 5. Blood Effects

**Blood Splat (on target):**
```javascript
for (let i = 0; i < 50; i++) {
  bloodSplat.push({
    x: hitX + random(-20, 20),
    y: hitY + random(-20, 20),
    size: random(3, 10)
  })
}
```

**Screen Blood (when player hit):**
```javascript
for (let i = 0; i < 100; i++) {
  screenBlood.push({
    x: random(0, canvas.width),
    y: random(0, canvas.height),
    size: random(5, 20),
    opacity: random(0.3, 0.7)
  })
}
// Plus full red overlay
```

#### 6. Target AI
```javascript
if (Date.now() >= target.nextShotTime && target.alive) {
  gameOver = true
  message = "You got shot!"
  // Apply screen blood effect
}
```
- Target shoots after 4 seconds if still alive
- Player must kill target before timer expires

#### 7. Win/Lose Conditions

**Win:**
- Hit target before it shoots you
- Message: "You hit the target!"

**Lose:**
- Target shoots you first
- Message: "You got shot!"
- Screen covered in blood

#### 8. Game Loop
```javascript
function gameLoop() {
  ctx.clearRect(0, 0, canvas.width, canvas.height)
  
  if (target.alive && !gameOver) {
    updateTarget()
  }
  
  drawTarget()
  drawBullet()
  drawBlood()
  drawScreenBlood()
  drawScope()
  drawMessage()
  
  if (!gameOver) {
    requestAnimationFrame(gameLoop)
  }
}
```

---

## SigmaSniper

### Overview
A first-person sniper game with movement, jumping, and advanced scope mechanics.

### Core Components

#### 1. Game State
```javascript
let player = {
  x, y, z,           // Position in 3D space
  vx, vy, vz,        // Velocity
  hp: 100,           // Health
  onGround: true     // Jump status
}
let enemies = []     // Enemy positions
let bullets = []     // Bullets in flight
let score = 0
```

#### 2. Controls
- **Click**: Lock mouse pointer
- **ESC**: Unlock pointer
- **Mouse Movement**: Look around
- **WASD**: Move (forward, left, back, right)
- **SPACE**: Jump
- **Left Click**: Shoot
- **Hold Right Click**: Scope (zoom view)

#### 3. Camera System

**Pointer Lock API:**
```javascript
canvas.addEventListener('click', () => {
  canvas.requestPointerLock()
})

document.addEventListener('pointerlockchange', () => {
  // Update locked state
})
```

**Mouse Look:**
```javascript
document.addEventListener('mousemove', (e) => {
  if (locked) {
    yaw += e.movementX * sensitivity
    pitch += e.movementY * sensitivity
    pitch = clamp(pitch, -89, 89)  // Prevent over-rotation
  }
})
```

#### 4. Movement System

**WASD Movement:**
- Forward/Back: Along view direction
- Left/Right: Perpendicular to view
- Speed affected by ground status

**Jump Mechanics:**
```javascript
if (onGround && keys['Space']) {
  vy = jumpForce
  onGround = false
}

// Gravity
if (!onGround) {
  vy -= gravity * dt
}

// Ground collision
if (y <= 0) {
  y = 0
  vy = 0
  onGround = true
}
```

#### 5. Scope System

**Scope Activation:**
```javascript
let scopeActive = false

canvas.addEventListener('mousedown', (e) => {
  if (e.button === 2) {  // Right click
    scopeActive = true
    showScopeOverlay()
  }
})
```

**Scope Visuals:**
- Radial vignette (dark edges)
- Reticle (crosshair)
- Tick marks around center
- Center dot
- Blur/zoom effect

**FOV Changes:**
```javascript
if (scopeActive) {
  fov = 30  // Zoomed in
} else {
  fov = 75  // Normal
}
```

#### 6. Shooting Mechanics

**Bullet Creation:**
```javascript
canvas.addEventListener('mousedown', (e) => {
  if (e.button === 0 && !reloading) {  // Left click
    bullets.push({
      x: player.x,
      y: player.y + eyeHeight,
      z: player.z,
      vx: Math.cos(yaw) * Math.cos(pitch) * bulletSpeed,
      vy: Math.sin(pitch) * bulletSpeed,
      vz: Math.sin(yaw) * Math.cos(pitch) * bulletSpeed
    })
  }
})
```

**Raycast Hit Detection:**
```javascript
for (let enemy of enemies) {
  let dist = distance(bullet, enemy)
  if (dist < hitRadius) {
    enemy.hp -= damage
    if (enemy.hp <= 0) {
      enemies.remove(enemy)
      score++
    }
  }
}
```

#### 7. Enemy AI

**Spawning:**
- Random positions around map
- Increasing numbers over time
- Different enemy types

**Behavior:**
- Move toward player
- Shoot at player when in range
- Take cover behind obstacles

**Damage to Player:**
```javascript
if (distance(enemy, player) < shootRange) {
  if (random() < hitChance) {
    player.hp -= enemyDamage
    showDamageEffect()
  }
}
```

#### 8. HUD Display
```javascript
<div id="hud">
  HP: <span id="hp">100</span>
  Score: <span id="score">0</span>
</div>
```
- Health display
- Score/kill count
- Control instructions
- Ammo count (if limited)

#### 9. Visual Effects

**Muzzle Flash:**
- Bright flash when shooting
- Fade out quickly
- Position at gun barrel

**Hit Markers:**
- Visual confirmation of hit
- Sound effect
- Blood splatter on enemy

**Damage Vignette:**
- Red screen edges when hit
- Fades over time
- Intensity based on damage

#### 10. 3D Rendering
- Uses canvas 2D context with perspective projection
- Fake 3D using scaling and positioning
- Depth sorting for proper draw order

#### 11. Game Loop
```javascript
function gameLoop(timestamp) {
  dt = (timestamp - lastTime) / 1000
  
  updatePlayer()
  updateEnemies()
  updateBullets()
  checkCollisions()
  
  render3DScene()
  renderHUD()
  
  requestAnimationFrame(gameLoop)
}
```

---

## DuckAdventure

### Overview
A grid-based game where you control a duck, collect stars, and avoid moving puddles that shoot projectiles.

### Core Components

#### 1. Game State
```javascript
let level = 1
let gridSize = 10 + (level - 1)  // Expands with level
let duckPos = 0                   // Grid index
let score = 0
let cells = []                    // Grid cells
let puddlePositions = []          // Enemy positions
let projectilePositions = []      // Enemy shots
```

#### 2. Grid System
```javascript
// Grid is 1D array representing 2D grid
// Position (row, col) = row * gridSize + col

function posToCoords(pos) {
  return {
    row: Math.floor(pos / gridSize),
    col: pos % gridSize
  }
}

function coordsToPos(row, col) {
  return row * gridSize + col
}
```

#### 3. Controls
- **Arrow Keys**: Move duck (up, down, left, right)
- Movement is instant (snaps to adjacent cell)

#### 4. Level Progression
```javascript
function initLevel() {
  gridSize = 10 + (level - 1)  // Grid grows
  starCount = 5 + level * 2     // More stars
  puddleCount = 2 + level       // More enemies
  
  // Timers speed up with level
  puddleMoveTimer = setInterval(movePuddles, 1000)
  puddleShootTimer = setInterval(shootProjectiles, 3000)
  projectileMoveTimer = setInterval(moveProjectiles, 300)
}
```

#### 5. Duck Movement
```javascript
document.addEventListener("keydown", (e) => {
  const { row, col } = posToCoords(duckPos)
  
  if (e.key === "ArrowUp" && row > 0)
    duckPos -= gridSize
  else if (e.key === "ArrowDown" && row < gridSize - 1)
    duckPos += gridSize
  else if (e.key === "ArrowLeft" && col > 0)
    duckPos -= 1
  else if (e.key === "ArrowRight" && col < gridSize - 1)
    duckPos += 1
    
  checkCollisions()
})
```

#### 6. Collectibles

**Stars (⭐):**
- Randomly placed on grid
- Count: 5 + level * 2
- Collecting all stars advances level
- Score +1 per star

**Collection Logic:**
```javascript
if (duckPos === starPos) {
  collectedStars++
  score++
  removeItem(starPos)
  
  if (collectedStars === totalStars) {
    level++
    initLevel()
  }
}
```

#### 7. Enemies (Puddles)

**Puddle Properties:**
```javascript
{
  position: gridIndex,
  type: "puddle",
  emoji: "💧"
}
```

**Puddle Movement:**
```javascript
function movePuddles() {
  for (let puddle of puddlePositions) {
    // Move toward duck using pathfinding
    let nextPos = getNextPositionTowardDuck(puddle.pos)
    if (isValidMove(nextPos)) {
      puddle.pos = nextPos
    }
  }
  checkCollisions()
}
```
- Execute every 1000ms
- Move one cell toward duck
- Use Manhattan distance for pathfinding

**Puddle Shooting:**
```javascript
function shootProjectiles() {
  for (let puddle of puddlePositions) {
    let direction = getDirectionToDuck(puddle.pos)
    projectilePositions.push({
      pos: puddle.pos,
      direction: direction,  // up, down, left, right
      emoji: "💦"
    })
  }
}
```
- Execute every 3000ms
- Shoot in direction of duck

#### 8. Projectiles

**Projectile Movement:**
```javascript
function moveProjectiles() {
  for (let proj of projectilePositions) {
    const { row, col } = posToCoords(proj.pos)
    
    switch (proj.direction) {
      case "up":    proj.pos -= gridSize; break
      case "down":  proj.pos += gridSize; break
      case "left":  proj.pos -= 1; break
      case "right": proj.pos += 1; break
    }
    
    // Remove if out of bounds
    if (isOutOfBounds(proj.pos)) {
      removeProjectile(proj)
    }
  }
  checkCollisions()
}
```
- Execute every 300ms
- Move one cell in direction
- Remove when leaving grid

#### 9. Collision Detection
```javascript
function checkCollisions() {
  // Duck vs Puddle
  for (let puddle of puddlePositions) {
    if (duckPos === puddle.pos) {
      gameOver("Hit by puddle!")
      return
    }
  }
  
  // Duck vs Projectile
  for (let proj of projectilePositions) {
    if (duckPos === proj.pos) {
      gameOver("Hit by projectile!")
      return
    }
  }
}
```

#### 10. Visual Representation
```css
.cell            /* Green grass tile */
.duck            /* Yellow background */
.star            /* Gold background */
.puddle          /* Blue background */
.projectile      /* Cyan background */
```

Each cell displays emoji:
- Duck: 🦆
- Star: ⭐
- Puddle: 💧
- Projectile: 💦

#### 11. HUD Display
```javascript
<div id="level">Level: {level}</div>
<div id="score">Score: {score}</div>
```

#### 12. Pathfinding Algorithm
```javascript
function getNextPositionTowardDuck(puddlePos) {
  const puddle = posToCoords(puddlePos)
  const duck = posToCoords(duckPos)
  
  // Manhattan distance
  const rowDiff = duck.row - puddle.row
  const colDiff = duck.col - puddle.col
  
  // Move in direction of largest difference
  if (Math.abs(rowDiff) > Math.abs(colDiff)) {
    return puddlePos + (rowDiff > 0 ? gridSize : -gridSize)
  } else {
    return puddlePos + (colDiff > 0 ? 1 : -1)
  }
}
```

---

## Trivia

### Overview
An interactive trivia quiz game with multiple-choice questions fetched from an API.

### Core Components

#### 1. Game State
```javascript
let questions = []        // Fetched questions
let currentQuestion = 0   // Index
let score = 0            // Correct answers
let streak = 0           // Consecutive correct
let playerName = ""
let selectedCategory = ""
let selectedDifficulty = ""
```

#### 2. Screen Management
```javascript
const screens = {
  welcome: "welcomeScreen",
  game: "gameScreen",
  results: "resultsScreen"
}

function showScreen(screenName) {
  document.querySelectorAll('.screen').forEach(s => {
    s.classList.remove('active')
  })
  document.getElementById(screenName).classList.add('active')
}
```

#### 3. Question API Integration
```javascript
async function fetchQuestions(category, difficulty, amount = 10) {
  const url = `https://opentdb.com/api.php?amount=${amount}&category=${category}&difficulty=${difficulty}&type=multiple`
  
  const response = await fetch(url)
  const data = await response.json()
  
  questions = data.results.map(q => ({
    question: decodeHTML(q.question),
    correct: decodeHTML(q.correct_answer),
    incorrect: q.incorrect_answers.map(decodeHTML),
    category: q.category,
    difficulty: q.difficulty
  }))
}
```

**HTML Entity Decoding:**
```javascript
function decodeHTML(html) {
  const txt = document.createElement('textarea')
  txt.innerHTML = html
  return txt.value
}
```
- Necessary because API returns HTML entities
- Example: `&quot;` → `"`

#### 4. Welcome Screen

**Name Input:**
```html
<input type="text" id="playerName" placeholder="Enter your name">
```

**Category Selection:**
```html
<select id="category">
  <option value="9">General Knowledge</option>
  <option value="21">Sports</option>
  <option value="23">History</option>
  <!-- etc. -->
</select>
```

**Difficulty Selection:**
```html
<select id="difficulty">
  <option value="easy">Easy</option>
  <option value="medium">Medium</option>
  <option value="hard">Hard</option>
</select>
```

#### 5. Question Display

**Shuffling Answers:**
```javascript
function displayQuestion() {
  const q = questions[currentQuestion]
  const allAnswers = [...q.incorrect, q.correct]
  
  // Fisher-Yates shuffle
  for (let i = allAnswers.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    ;[allAnswers[i], allAnswers[j]] = [allAnswers[j], allAnswers[i]]
  }
  
  // Display shuffled answers
  allAnswers.forEach(answer => {
    createAnswerButton(answer)
  })
}
```

**Question HTML:**
```html
<div id="questionText">What is the capital of France?</div>
<div id="questionInfo">Category: Geography | Difficulty: Easy</div>
<div id="progress">Question 1 of 10</div>
```

#### 6. Answer Selection

**Button Creation:**
```javascript
function createAnswerButton(answer) {
  const button = document.createElement('button')
  button.className = 'choice'
  button.textContent = answer
  button.onclick = () => selectAnswer(answer)
  choicesContainer.appendChild(button)
}
```

**Answer Checking:**
```javascript
function selectAnswer(selected) {
  const correct = questions[currentQuestion].correct
  const buttons = document.querySelectorAll('.choice')
  
  // Disable all buttons
  buttons.forEach(btn => btn.disabled = true)
  
  // Visual feedback
  buttons.forEach(btn => {
    if (btn.textContent === correct) {
      btn.classList.add('correct')  // Green
    }
    if (btn.textContent === selected && selected !== correct) {
      btn.classList.add('wrong')    // Red
    }
  })
  
  // Update score
  if (selected === correct) {
    score++
    streak++
    showFeedback("Correct! ✓", "success")
  } else {
    streak = 0
    showFeedback(`Wrong! The answer was: ${correct}`, "error")
  }
  
  // Next question after delay
  setTimeout(nextQuestion, 2000)
}
```

#### 7. Scoring System

**Points:**
- Easy: 1 point per correct answer
- Medium: 2 points per correct answer
- Hard: 3 points per correct answer

**Streak Bonus:**
```javascript
if (streak >= 3) {
  bonusPoints = streak - 2  // 3 streak = +1, 4 = +2, etc.
  totalScore += bonusPoints
}
```

**Visual Feedback:**
```css
.correct {
  background: #4CAF50;  /* Green */
  border-color: #45a049;
}

.wrong {
  background: #f44336;  /* Red */
  border-color: #da190b;
}
```

#### 8. Results Screen

**Statistics Display:**
```javascript
function showResults() {
  const percentage = (score / questions.length) * 100
  
  resultsHTML = `
    <h2>Quiz Complete, ${playerName}!</h2>
    <p>Score: ${score} / ${questions.length}</p>
    <p>Percentage: ${percentage.toFixed(1)}%</p>
    <p>Best Streak: ${maxStreak}</p>
  `
  
  // Performance message
  if (percentage === 100) {
    resultsHTML += `<p class="perfect">Perfect Score! 🏆</p>`
  } else if (percentage >= 80) {
    resultsHTML += `<p class="great">Great Job! 🎉</p>`
  } else if (percentage >= 60) {
    resultsHTML += `<p class="good">Good Effort! 👍</p>`
  } else {
    resultsHTML += `<p class="tryagain">Keep Trying! 💪</p>`
  }
}
```

**Replay Options:**
```html
<button onclick="restartQuiz()">Play Again</button>
<button onclick="changeSettings()">Change Settings</button>
<button onclick="viewLeaderboard()">Leaderboard</button>
```

#### 9. Data Persistence

**Local Storage:**
```javascript
// Save high score
function saveScore() {
  const scores = JSON.parse(localStorage.getItem('triviaScores') || '[]')
  scores.push({
    name: playerName,
    score: score,
    category: selectedCategory,
    difficulty: selectedDifficulty,
    date: new Date().toISOString()
  })
  scores.sort((a, b) => b.score - a.score)
  scores.splice(10)  // Keep top 10
  localStorage.setItem('triviaScores', JSON.stringify(scores))
}

// Load leaderboard
function loadLeaderboard() {
  return JSON.parse(localStorage.getItem('triviaScores') || '[]')
}
```

#### 10. Timer (Optional Feature)
```javascript
let timeRemaining = 30  // seconds per question

function startTimer() {
  timerInterval = setInterval(() => {
    timeRemaining--
    updateTimerDisplay()
    
    if (timeRemaining <= 0) {
      selectAnswer(null)  // Auto-submit (wrong answer)
    }
  }, 1000)
}

function stopTimer() {
  clearInterval(timerInterval)
}
```

#### 11. Animations
```css
@keyframes fadeIn {
  from { opacity: 0; transform: translateY(20px); }
  to { opacity: 1; transform: translateY(0); }
}

.choice {
  animation: fadeIn 0.3s ease-out;
}

.choice:hover:not(:disabled) {
  transform: scale(1.02);
  transition: all 0.3s;
}
```

#### 12. Error Handling
```javascript
async function fetchQuestions() {
  try {
    const response = await fetch(url)
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`)
    }
    
    const data = await response.json()
    
    if (data.response_code !== 0) {
      throw new Error('No questions available for this category/difficulty')
    }
    
    questions = data.results
  } catch (error) {
    showError('Failed to load questions. Please try again.')
    console.error('Fetch error:', error)
  }
}
```

---

## Common Patterns Across All Games

### 1. Canvas-Based Rendering
Most games use HTML5 Canvas for rendering:
```javascript
const canvas = document.getElementById('gameCanvas')
const ctx = canvas.getContext('2d')

function draw() {
  ctx.clearRect(0, 0, canvas.width, canvas.height)
  // Draw game objects
}
```

### 2. Game Loop Pattern
```javascript
function gameLoop() {
  if (gameRunning) {
    update()    // Game logic
    draw()      // Rendering
    requestAnimationFrame(gameLoop)
  }
}
```

### 3. Collision Detection
Rectangle-based AABB collision:
```javascript
function checkCollision(a, b) {
  return a.x < b.x + b.width &&
         a.x + a.width > b.x &&
         a.y < b.y + b.height &&
         a.y + a.height > b.y
}
```

### 4. Input Handling
```javascript
const keys = {}

document.addEventListener('keydown', e => {
  keys[e.key] = true
})

document.addEventListener('keyup', e => {
  keys[e.key] = false
})

// In update loop
if (keys['ArrowLeft']) player.x -= player.speed
```

### 5. State Management
```javascript
let gameState = 'menu'  // menu, playing, paused, gameover

function changeState(newState) {
  gameState = newState
  // Update UI accordingly
}
```

### 6. Audio System
```javascript
const audio = new Audio('sound.mp3')
audio.volume = 0.5

function playSound() {
  audio.currentTime = 0  // Reset to start
  audio.play()
}
```

### 7. Score Persistence
```javascript
// Save
localStorage.setItem('highScore', JSON.stringify(score))

// Load
const highScore = JSON.parse(localStorage.getItem('highScore') || '0')
```

---

## Technical Architecture

### File Structure
Each game is self-contained in a single `index.html` file containing:
1. HTML structure
2. CSS styling
3. JavaScript game logic
4. Asset references

### Performance Optimizations
1. **Object Pooling**: Reuse bullet/enemy objects
2. **Culling**: Don't render off-screen objects
3. **Lazy Loading**: Load audio on user interaction
4. **RequestAnimationFrame**: Smooth 60 FPS rendering

### Browser Compatibility
- HTML5 Canvas API
- ES6+ JavaScript features
- Web Audio API
- LocalStorage API
- Pointer Lock API (SigmaSniper)

### Asset Management
- Images: PNG format
- Audio: MP3 format
- Preloading: Images loaded before game starts
- Fallbacks: Silent failure for audio issues

---

## Development Guidelines

### Code Style
- Consistent indentation (2 spaces)
- Descriptive variable names
- Comments for complex logic
- Modular function design

### Testing
- Cross-browser testing (Chrome, Firefox, Safari)
- Mobile responsiveness (where applicable)
- Performance profiling
- Audio fallback testing

### Future Enhancements
1. Multiplayer support
2. More levels/content
3. Save game progress
4. Achievements system
5. Mobile touch controls
6. Better graphics/animations
7. Sound effects library
8. Analytics integration

---

## Conclusion

This collection demonstrates various game programming concepts:
- **Physics**: Gravity, velocity, collision
- **AI**: Enemy pathfinding, behavior patterns
- **User Input**: Keyboard, mouse, touch
- **Graphics**: Canvas rendering, particle effects
- **Audio**: Sound effects, background music
- **Persistence**: Local storage, high scores
- **UI/UX**: Menus, HUD, feedback

Each game serves as a learning example for different aspects of browser-based game development.
