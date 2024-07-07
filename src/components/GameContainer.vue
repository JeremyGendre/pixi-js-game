<script setup>
import { onMounted, ref } from 'vue';
import { Application, Assets, Sprite } from 'pixi.js';

onMounted(() => {
  launchGame()
});

function loadAssets() {
  return Promise.all([
    Assets.load('/images/player.png'),
    Assets.load('/images/bullet_black.png')
  ]);
}

const loadingAssets = ref(true);

async function launchGame() {
  // Create a new application
  const app = new Application();

  let keys = {};
  let bullets = [];
  let pointerDown = false;
  let bulletFireRateLimitTimeElapsed = 0;
  const bulletSpeed = 7;
  const magicRotation = 0; // old value 1.6;

  // Initialize the application
  await app.init({
    background: '#FFFEEE',
    resizeTo: window,
    eventMode: 'dynamic'
  });

  // Append the application canvas to the document body
  document.body.appendChild(app.canvas);

  // Load textures
  const [playerTexture, bulletTexture] = await loadAssets();
  loadingAssets.value = false;

  // Create players Sprites
  const player = new Sprite(playerTexture);
  player.scale = { x: 0.8, y: 0.8 };
  const ennemy = new Sprite(playerTexture);

  // Center the sprite's anchor point
  player.anchor.set(0.5);
  ennemy.anchor.set(0.5);

  // Move the sprite to the center of the screen
  player.x = app.screen.width / 2;
  player.y = app.screen.height / 2;
  ennemy.x = player.x + 100;
  ennemy.y = player.y + 100;

  app.stage.addChild(player);
  app.stage.addChild(ennemy);

  // Listen for animate update
  app.ticker.add(gameLoop);

  // rotate the player to follow the mouse
  app.stage.eventMode = 'static';
  app.stage.hitArea = app.screen;
  app.stage.on('mousemove', onMouseMove);
  app.stage.on('pointerdown', onPointerDown);
  app.stage.on('pointerup', onPointerUp);

  window.addEventListener('keydown', onKeyDown);
  window.addEventListener('keyup', onKeyUp);

  function gameLoop(delta) {
    if (keys['z'] || keys['w']) {
      player.y -= 2;
    }
    if (keys['s']) {
      player.y += 2;
    }
    if (keys['q'] || keys['a']) {
      player.x -= 2;
    }
    if (keys['d']) {
      player.x += 2;
    }

    // fire a bullet if the pointer is down
    // use a rate limiter to avoid firing too many bullets
    if (pointerDown && bulletFireRateLimitTimeElapsed > 100) {
      bulletFireRateLimitTimeElapsed = 0;
      fireBullet();
    } else {
      bulletFireRateLimitTimeElapsed += delta.elapsedMS;
    }

    bullets.forEach((bullet) => {
      bullet.x += Math.cos(bullet.rotation) * bulletSpeed;
      bullet.y += Math.sin(bullet.rotation) * bulletSpeed;
      // if the bullet is out of the screen, remove it
      if (
        bullet.x < 0 ||
        bullet.x > app.screen.width ||
        bullet.y < 0 ||
        bullet.y > app.screen.height
      ) {
        app.stage.removeChild(bullet);
        bullets.splice(bullets.indexOf(bullet), 1);
      }

      if (!ennemy.dead && handleCollision(bullet, ennemy)) {
        console.log('bullet hit ennemy');
        app.stage.removeChild(bullet);
        bullets.splice(bullets.indexOf(bullet), 1);
        ennemy.dead = true;
        ennemy.destroy({ children: true, texture: true });
      }
    });

    if (!ennemy.dead && !player.dead && handleCollision(player, ennemy)) {
      console.log('collision player ennemy');
      // stop the player from moving
      player.x = player.x - Math.cos(player.rotation) * 2;
      player.y = player.y - Math.sin(player.rotation) * 2;
    }
  }

  function onMouseMove(event) {
    player.rotation = Math.atan2(
      event.global.y - player.y,
      event.global.x - player.x
    ) + magicRotation;
  }

  function onKeyDown(event) {
    keys[event.key] = true;
  }
  function onKeyUp(event) {
    keys[event.key] = false;
  }

  function onPointerDown() {
    pointerDown = true;
  }

  function onPointerUp() {
    pointerDown = false;
  }

  function fireBullet() {
    const bullet = new Sprite(bulletTexture);
    bullet.anchor.set(0.5);
    bullet.x = player.x;
    bullet.y = player.y;
    bullet.rotation = player.rotation - magicRotation;
    bullet.speed = bulletSpeed;
    app.stage.addChild(bullet);
    bullets.push(bullet);
  }

  function handleCollision(a, b) {
    const aBox = a.getBounds();
    const bBox = b.getBounds();
    return aBox.x + aBox.width > bBox.x &&
      aBox.x < bBox.x + bBox.width &&
      aBox.y + aBox.height > bBox.y &&
      aBox.y < bBox.y + bBox.height;
  }
}

</script>
<template>
  <div>

  </div>
</template>