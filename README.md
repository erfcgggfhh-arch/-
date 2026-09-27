<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport"
      content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no">

<title>Robot Arena</title>

<style>
* {
    box-sizing: border-box;
    touch-action: none;
}

html, body {
    margin: 0;
    width: 100%;
    height: 100%;
    overflow: hidden;
    background: #111;
    font-family: Arial, sans-serif;
}

#game {
    width: 100vw;
    height: 100vh;
    position: relative;
    overflow: hidden;
}

canvas {
    width: 100%;
    height: 100%;
    display: block;
}

.screen {
    position: absolute;
    inset: 0;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    background: linear-gradient(#18244fee,#090e1cee);
    z-index: 10;
}

.screen h1 {
    font-size: clamp(40px,8vw,80px);
    color: white;
    text-shadow: 0 5px #000;
    margin-bottom: 25px;
}

button {
    min-width: 240px;
    margin: 8px;
    padding: 16px 30px;
    border: none;
    border-radius: 18px;
    background: #e79b21;
    color: white;
    font-size: 24px;
    font-weight: bold;
    box-shadow: 0 6px #814b09;
}

button:active {
    transform: translateY(5px);
    box-shadow: 0 1px #814b09;
}

.hidden {
    display: none !important;
}

#hud {
    position: absolute;
    top: 15px;
    left: 15px;
    right: 15px;
    display: flex;
    justify-content: space-between;
    color: white;
    font-size: 20px;
    font-weight: bold;
    text-shadow: 2px 2px #000;
    z-index: 5;
}

.health {
    width: 210px;
    height: 22px;
    background: #222;
    border: 3px solid #111;
    border-radius: 12px;
    overflow: hidden;
}

#healthBar {
    height: 100%;
    width: 100%;
    background: #40d957;
}

#controls {
    position: absolute;
    inset: 0;
    z-index: 6;
}

#joystick {
    position: absolute;
    left: 25px;
    bottom: 25px;
    width: 135px;
    height: 135px;
    border-radius: 50%;
    background: #ffffff35;
    border: 3px solid #ffffff55;
}

#stick {
    position: absolute;
    width: 60px;
    height: 60px;
    left: 35px;
    top: 35px;
    border-radius: 50%;
    background: #ffffff77;
}

.action {
    position: absolute;
    right: 25px;
    bottom: 30px;
    width: 105px;
    height: 105px;
    min-width: 0;
    border-radius: 50%;
    padding: 5px;
    font-size: 17px;
}

#superButton {
    right: 145px;
    bottom: 105px;
    width: 80px;
    height: 80px;
    background: #8846d5;
}

#message {
    position: absolute;
    inset: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    color: white;
    font-size: 55px;
    font-weight: bold;
    text-shadow: 4px 4px #000;
    pointer-events: none;
    z-index: 20;
}
</style>
</head>

<body>

<div id="game">

<canvas id="canvas"></canvas>

<!-- МЕНЮ -->
<div id="menu" class="screen">

<h1>ROBOT ARENA</h1>

<button onclick="startGame()">▶ ИГРАТЬ</button>

<button onclick="openEmpty('МАГАЗИН')">
🛒 МАГАЗИН
</button>

<button onclick="openEmpty('СКИНЫ')">
🎨 СКИНЫ
</button>

</div>

<!-- ПУСТЫЕ РАЗДЕЛЫ -->
<div id="emptyScreen" class="screen hidden">

<button
style="position:absolute;top:15px;left:15px;min-width:auto;font-size:18px"
onclick="backMenu()">
← НАЗАД
</button>

<h1 id="emptyTitle">ПУСТО</h1>

<p style="font-size:25px;color:white">
Здесь пока ничего нет.
</p>

</div>

<!-- HUD -->
<div id="hud" class="hidden">

<div>
❤️
<div class="health">
<div id="healthBar"></div>
</div>
</div>

<div>
🤖 Роботов: <span id="robotCount">0</span>
</div>

</div>

<!-- УПРАВЛЕНИЕ -->
<div id="controls" class="hidden">

<div id="joystick">
<div id="stick"></div>
</div>

<button class="action" id="fireButton">
🔫<br>ОГОНЬ
</button>

<button class="action" id="superButton">
💥<br>СУПЕР
</button>

</div>

<div id="message"></div>

</div>

<script>

const canvas = document.getElementById("canvas");
const ctx = canvas.getContext("2d");

let W;
let H;

let playing = false;

let player;

let robots = [];
let bullets = [];
let particles = [];

let moveX = 0;
let moveY = 0;

let shooting = false;

let lastTime = 0;


/* -----------------------
   РАЗМЕР CANVAS
----------------------- */

function resize() {

    W = canvas.width = window.innerWidth;
    H = canvas.height = window.innerHeight;

}

window.addEventListener("resize", resize);

resize();


/* -----------------------
   МЕНЮ
----------------------- */

function openEmpty(title) {

    document.getElementById("menu")
        .classList.add("hidden");

    document.getElementById("emptyScreen")
        .classList.remove("hidden");

    document.getElementById("emptyTitle")
        .textContent = title;

}


function backMenu() {

    document.getElementById("emptyScreen")
        .classList.add("hidden");

    document.getElementById("menu")
        .classList.remove("hidden");

}


/* -----------------------
   НАЧАЛО ИГРЫ
----------------------- */

function startGame() {

    document.getElementById("menu")
        .classList.add("hidden");

    document.getElementById("emptyScreen")
        .classList.add("hidden");

    document.getElementById("hud")
        .classList.remove("hidden");

    document.getElementById("controls")
        .classList.remove("hidden");


    player = {

        x: W / 2,
        y: H / 2,

        radius: 23,

        health: 100,
        maxHealth: 100,

        cooldown: 0,

        super: 0,

        invincible: 0

    };


    robots = [];
    bullets = [];
    particles = [];


    /* создаём роботов */

    for (let i = 0; i < 7; i++) {

        let angle =
            Math.random() * Math.PI * 2;

        let distance =
            280 + Math.random() * 250;

        robots.push({

            x:
                W / 2 +
                Math.cos(angle) * distance,

            y:
                H / 2 +
                Math.sin(angle) * distance,

            radius: 20,

            health: 70,
            maxHealth: 70,

            cooldown:
                Math.random(),

            hit: 0

        });

    }


    playing = true;

    document.getElementById("message")
        .textContent = "";

    requestAnimationFrame(gameLoop);

}


/* -----------------------
   ЗАВЕРШЕНИЕ
----------------------- */

function endGame(text) {

    playing = false;

    document.getElementById("message")
        .textContent = text;

    setTimeout(() => {

        document.getElementById("hud")
            .classList.add("hidden");

        document.getElementById("controls")
            .classList.add("hidden");

        document.getElementById("menu")
            .classList.remove("hidden");

        document.getElementById("message")
            .textContent = "";

    }, 1500);

}


/* -----------------------
   РАССТОЯНИЕ
----------------------- */

function distance(a,b) {

    return Math.hypot(
        a.x - b.x,
        a.y - b.y
    );

}


/* -----------------------
   СТРЕЛЬБА
----------------------- */

function shoot() {

    if (!playing) return;

    if (player.cooldown > 0) return;

    player.cooldown = 0.55;


    let dx = moveX;
    let dy = moveY;


    /* если игрок не двигается —
       стреляем в ближайшего робота */

    if (Math.hypot(dx,dy) < 0.2) {

        let target = null;

        for (let robot of robots) {

            if (
                !target ||
                distance(player,robot)
                <
                distance(player,target)
            ) {
                target = robot;
            }

        }


        if (target) {

            dx = target.x - player.x;
            dy = target.y - player.y;

        } else {

            dx = 1;
            dy = 0;

        }

    }


    let length =
        Math.hypot(dx,dy);

    dx /= length;
    dy /= length;


    /* дробовик */

    for (let i = -2; i <= 2; i++) {

        let angle =
            Math.atan2(dy,dx)
            +
            i * 0.12;

        bullets.push({

            x:
                player.x +
                dx * 25,

            y:
                player.y +
                dy * 25,

            vx:
                Math.cos(angle) * 650,

            vy:
                Math.sin(angle) * 650,

            life: 0.45,

            damage: 18,

            enemy: false

        });

    }

}


/* -----------------------
   СУПЕР
----------------------- */

function superAttack() {

    if (!playing) return;

    if (player.super < 100) return;


    player.super = 0;


    for (let robot of robots) {

        if (distance(player,robot) < 160) {

            robot.health -= 75;

        }

    }


    for (let i = 0; i < 40; i++) {

        particles.push({

            x: player.x,
            y: player.y,

            angle:
                Math.random() *
                Math.PI * 2,

            speed:
                80 +
                Math.random() * 250,

            life: 0.5

        });

    }

}


/* -----------------------
   ОБНОВЛЕНИЕ
----------------------- */

function update(dt) {


    /* движение */

    let length =
        Math.hypot(moveX,moveY);


    if (length > 0.1) {

        player.x +=
            moveX / length *
            250 *
            dt;

        player.y +=
            moveY / length *
            250 *
            dt;

    }


    player.x =
        Math.max(
            25,
            Math.min(W - 25,player.x)
        );

    player.y =
        Math.max(
            50,
            Math.min(H - 25,player.y)
        );


    player.cooldown =
        Math.max(
            0,
            player.cooldown - dt
        );


    player.invincible =
        Math.max(
            0,
            player.invincible - dt
        );


    if (shooting) {

        shoot();

    }


    /* пули */

    for (let bullet of bullets) {

        bullet.x +=
            bullet.vx * dt;

        bullet.y +=
            bullet.vy * dt;

        bullet.life -= dt;

    }


    bullets =
        bullets.filter(
            b =>
                b.life > 0 &&
                b.x > -30 &&
                b.x < W + 30 &&
                b.y > -30 &&
                b.y < H + 30
        );


    /* роботы */

    for (let robot of robots) {

        let dx =
            player.x - robot.x;

        let dy =
            player.y - robot.y;

        let d =
            Math.hypot(dx,dy);


        if (d > 100) {

            robot.x +=
                dx / d *
                72 *
                dt;

            robot.y +=
                dy / d *
                72 *
                dt;

        }


        robot.cooldown -= dt;


        /* робот стреляет */

        if (
            d < 380 &&
            robot.cooldown <= 0
        ) {

            robot.cooldown =
                1.5 +
                Math.random();

            bullets.push({

                x: robot.x,
                y: robot.y,

                vx:
                    dx / d * 260,

                vy:
                    dy / d * 260,

                life: 1.7,

                damage: 10,

                enemy: true

            });

        }

    }


    /* столкновения */

    for (let bullet of bullets) {

        /* в игрока */

        if (bullet.enemy) {

            if (
                distance(bullet,player)
                <
                player.radius + 5
            ) {

                if (player.invincible <= 0) {

                    player.health -=
                        bullet.damage;

                    player.invincible = 0.25;

                }

                bullet.life = 0;

            }

        }


        /* в робота */

        else {

            for (let robot of robots) {

                if (
                    distance(bullet,robot)
                    <
                    robot.radius
                ) {

                    robot.health -=
                        bullet.damage;

                    bullet.life = 0;

                    player.super =
                        Math.min(
                            100,
                            player.super + 8
                        );

                    break;

                }

            }

        }

    }


    robots =
        robots.filter(
            robot =>
                robot.health > 0
        );


    /* частицы */

    for (let p of particles) {

        p.x +=
            Math.cos(p.angle) *
            p.speed *
            dt;

        p.y +=
            Math.sin(p.angle) *
            p.speed *
            dt;

        p.life -= dt;

    }


    particles =
        particles.filter(
            p => p.life > 0
        );


    /* HUD */

    document.getElementById(
        "healthBar"
    ).style.width =
        Math.max(
            0,
            player.health
        ) + "%";


    document.getElementById(
        "robotCount"
    ).textContent =
        robots.length;


    if (player.health <= 0) {

        endGame("ПОРАЖЕНИЕ");

    }


    if (robots.length === 0) {

        endGame("ПОБЕДА!");

    }

}


/* -----------------------
   РИСОВАНИЕ
----------------------- */

function draw() {


    /* поле */

    ctx.fillStyle = "#70b94d";

    ctx.fillRect(
        0,
        0,
        W,
        H
    );


    /* клетки */

    ctx.strokeStyle =
        "#57933b";

    ctx.lineWidth = 3;


    for (
        let x = 0;
        x < W;
        x += 55
    ) {

        ctx.beginPath();

        ctx.moveTo(x,0);

        ctx.lineTo(x,H);

        ctx.stroke();

    }


    for (
        let y = 0;
        y < H;
        y += 55
    ) {

        ctx.beginPath();

        ctx.moveTo(0,y);

        ctx.lineTo(W,y);

        ctx.stroke();

    }


    /* препятствия */

    for (
        let i = 0;
        i < 7;
        i++
    ) {

        let x =
            80 + i * 170;

        let y =
            100 +
            (i % 2) * 230;


        ctx.fillStyle =
            "#87633f";

        ctx.fillRect(
            x,
            y,
            70,
            55
        );

    }


    /* пули */

    for (let bullet of bullets) {

        ctx.fillStyle =
            bullet.enemy
            ? "#ffcf4a"
            : "#fff3b0";

        ctx.beginPath();

        ctx.arc(
            bullet.x,
            bullet.y,
            5,
            0,
            Math.PI * 2
        );

        ctx.fill();

    }


    /* роботы */

    for (let robot of robots) {

        ctx.fillStyle =
            "#3c4753";

        ctx.beginPath();

        ctx.arc(
            robot.x,
            robot.y,
            robot.radius,
            0,
            Math.PI * 2
        );

        ctx.fill();


        ctx.fillStyle =
            "#e84b4b";

        ctx.fillRect(
            robot.x - 12,
            robot.y - 4,
            24,
            8
        );


        /* здоровье */

        ctx.fillStyle =
            "#111";

        ctx.fillRect(
            robot.x - 20,
            robot.y - 30,
            40,
            5
        );


        ctx.fillStyle =
            "#4be05b";

        ctx.fillRect(
            robot.x - 20,
            robot.y - 30,
            40 *
            robot.health /
            robot.maxHealth,
            5
        );

    }


    /* игрок */

    ctx.save();

    ctx.translate(
        player.x,
        player.y
    );


    let angle =
        Math.atan2(
            moveY,
            moveX
        );


    if (
        Math.hypot(moveX,moveY)
        < 0.2
    ) {

        angle = 0;

    }


    ctx.rotate(angle);


    /* тело */

    ctx.fillStyle =
        player.invincible > 0
        ? "#ffffff"
        : "#8b55e8";

    ctx.beginPath();

    ctx.arc(
        0,
        0,
        player.radius,
        0,
        Math.PI * 2
    );

    ctx.fill();


    /* лицо */

    ctx.fillStyle =
        "#f4c69c";

    ctx.beginPath();

    ctx.arc(
        5,
        -10,
        13,
        0,
        Math.PI * 2
    );

    ctx.fill();


    /* волосы */

    ctx.fillStyle =
        "#6d4027";

    ctx.beginPath();

    ctx.arc(
        2,
        -17,
        11,
        Math.PI,
        Math.PI * 2
    );

    ctx.fill();


    /* дробовик */

    ctx.fillStyle =
        "#5b3824";

    ctx.fillRect(
        15,
        -5,
        31,
        10
    );


    ctx.fillStyle =
        "#333";

    ctx.fillRect(
        38,
        -7,
        12,
        14
    );


    ctx.restore();


    /* частицы */

    for (let p of particles) {

        ctx.fillStyle =
            "#ffd34d";

        ctx.beginPath();

        ctx.arc(
            p.x,
            p.y,
            4,
            0,
            Math.PI * 2
        );

        ctx.fill();

    }

}


/* -----------------------
   ИГРОВОЙ ЦИКЛ
----------------------- */

function gameLoop(time) {

    if (!playing) return;


    let dt =
        Math.min(
            0.035,
            (time - lastTime) / 1000 || 0.016
        );


    lastTime = time;


    update(dt);

    draw();


    requestAnimationFrame(
        gameLoop
    );

}


/* -----------------------
   ДЖОЙСТИК
----------------------- */

const joystick =
    document.getElementById("joystick");

const stick =
    document.getElementById("stick");


function joystickMove(e) {

    let rect =
        joystick.getBoundingClientRect();


    let x =
        e.clientX -
        rect.left -
        rect.width / 2;

    let y =
        e.clientY -
        rect.top -
        rect.height / 2;


    let length =
        Math.hypot(x,y);

    let max = 45;


    if (length > max) {

        x =
            x / length * max;

        y =
            y / length * max;

    }


    moveX = x / max;
    moveY = y / max;


    stick.style.transform =
        `translate(${x}px,${y}px)`;

}


joystick.addEventListener(
    "pointerdown",
    e => {

        joystick.setPointerCapture(
            e.pointerId
        );

        joystickMove(e);

    }
);


joystick.addEventListener(
    "pointermove",
    e => {

        if (e.buttons)
            joystickMove(e);

    }
);


joystick.addEventListener(
    "pointerup",
    resetJoystick
);


function resetJoystick() {

    moveX = 0;
    moveY = 0;

    stick.style.transform =
        "translate(0,0)";

}


/* -----------------------
   КНОПКА ОГОНЬ
----------------------- */

const fireButton =
    document.getElementById(
        "fireButton"
    );


fireButton.addEventListener(
    "pointerdown",
    () => {

        shooting = true;
        shoot();

    }
);


fireButton.addEventListener(
    "pointerup",
    () => {

        shooting = false;

    }
);


fireButton.addEventListener(
    "pointercancel",
    () => {

        shooting = false;

    }
);


/* -----------------------
   СУПЕР
----------------------- */

document.getElementById(
    "superButton"
).addEventListener(
    "pointerdown",
    superAttack
);


/* -----------------------
   КЛАВИАТУРА
----------------------- */

window.addEventListener(
    "keydown",
    e => {

        if (e.key === " ") {

            shooting = true;

            shoot();

        }

        if (
            e.key === "e" ||
            e.key === "E"
        ) {

            superAttack();

        }

    }
);


window.addEventListener(
    "keyup",
    e => {

        if (e.key === " ") {

            shooting = false;

        }

    }
);

</script>

</body>
</html>
