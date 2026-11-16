<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<title>X: The Final Escape</title>

<style>

* {
    box-sizing: border-box;
}

body {
    margin: 0;
    overflow: hidden;
    background: #020208;
    color: white;
    font-family: Arial, sans-serif;
    touch-action: none;
}

canvas {
    display: block;
    touch-action: none;
}

#hud {
    position: absolute;
    top: 15px;
    left: 15px;
    z-index: 5;
    background: rgba(0,0,0,0.65);
    padding: 13px;
    border-radius: 12px;
    font-size: 15px;
    line-height: 1.8;
    border: 1px solid #554477;
}

#message {
    position: absolute;
    top: 40%;
    left: 5%;
    width: 90%;
    text-align: center;
    z-index: 8;
    pointer-events: none;
}

#crosshair {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%,-50%);
    z-index: 4;
    font-size: 23px;
    pointer-events: none;
}

#joystick {
    position: absolute;
    bottom: 25px;
    left: 25px;
    width: 125px;
    height: 125px;
    border-radius: 50%;
    background: rgba(180,180,200,0.18);
    border: 2px solid rgba(255,255,255,0.4);
    z-index: 10;
    touch-action: none;
}

#stick {
    position: absolute;
    width: 45px;
    height: 45px;
    background: #aaaacc;
    border-radius: 50%;
    left: 38px;
    top: 38px;
}

.gameButton {
    position: absolute;
    z-index: 10;
    border: 2px solid #aa77ff;
    background: rgba(30,15,55,0.9);
    color: white;
    border-radius: 50%;
    width: 75px;
    height: 75px;
    font-weight: bold;
    touch-action: none;
}

#interact {
    right: 25px;
    bottom: 125px;
}

#throw {
    right: 25px;
    bottom: 35px;
    display: none;
}

#puzzle {
    display: none;
    position: absolute;
    z-index: 20;
    top: 50%;
    left: 50%;
    transform: translate(-50%,-50%);
    width: 85%;
    max-width: 400px;
    padding: 25px;
    background: rgba(8,5,20,0.97);
    border: 2px solid #a35cff;
    border-radius: 18px;
    text-align: center;
    box-shadow: 0 0 35px #7300ff66;
}

#answer {
    width: 100%;
    padding: 13px;
    margin-top: 15px;
    background: #171326;
    color: white;
    border: 1px solid #a35cff;
    border-radius: 8px;
    font-size: 18px;
}

#submit {
    margin-top: 12px;
    padding: 12px 25px;
    background: #842bce;
    border: none;
    border-radius: 8px;
    color: white;
    font-weight: bold;
}

#closePuzzle {
    background: #333;
    color: white;
    border: none;
    padding: 9px 20px;
    margin-left: 8px;
    border-radius: 8px;
}

#ending {
    display: none;
    position: absolute;
    inset: 0;
    z-index: 30;
    background: rgba(8,0,20,0.97);
    align-items: center;
    justify-content: center;
    text-align: center;
    padding: 25px;
}

#endingText {
    font-size: 22px;
    line-height: 2;
    max-width: 500px;
}

#endingTitle {
    color: #e4aaff;
    font-size: 30px;
}

</style>
</head>

<body>

<div id="hud">
    <div>✕ TOKENS: <span id="count">0</span>/5</div>
    <div>PUZZLES: <span id="puzzleCount">0</span>/3</div>
    <div id="status">Find the X tokens.</div>
</div>

<div id="message">
    <h2>✕ THE FINAL ESCAPE</h2>
    <p>Something is waiting for you...</p>
</div>

<div id="crosshair">+</div>

<div id="joystick">
    <div id="stick"></div>
</div>

<button id="interact" class="gameButton">INTERACT</button>

<button id="throw" class="gameButton">THROW X</button>

<div id="puzzle">

    <h2 id="puzzleTitle">MYSTERY</h2>

    <p id="puzzleQuestion"></p>

    <input id="answer" placeholder="Your answer">

    <br>

    <button id="submit">SUBMIT</button>

    <button id="closePuzzle">CLOSE</button>

    <p id="feedback"></p>

</div>

<div id="ending">

    <div id="endingText">

        <h1 id="endingTitle">THE END</h1>

        <p id="endingStory"></p>

        <button onclick="location.reload()"
        style="padding:15px 30px;
        background:#922cff;
        color:white;
        border:0;
        border-radius:10px;
        font-size:16px;">
            PLAY AGAIN
        </button>

    </div>

</div>


<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>

<script>

// ==================================
// 1. GAME SETUP
// ==================================

const scene = new THREE.Scene();

scene.background = new THREE.Color(0x020208);

scene.fog = new THREE.FogExp2(
    0x020208,
    0.035
);

const camera = new THREE.PerspectiveCamera(
    75,
    innerWidth / innerHeight,
    0.1,
    100
);

camera.position.set(1.5,1.6,1.5);

const renderer = new THREE.WebGLRenderer({
    antialias:true
});

renderer.setSize(innerWidth,innerHeight);

renderer.setPixelRatio(
    Math.min(devicePixelRatio,2)
);

renderer.shadowMap.enabled = true;

document.body.appendChild(renderer.domElement);


// ==================================
// 2. LIGHTING
// ==================================

scene.add(
    new THREE.AmbientLight(0x343044,0.65)
);

const flashlight = new THREE.PointLight(
    0xffffff,
    2.5,
    12
);

camera.add(flashlight);

scene.add(camera);


// Mysterious purple lights

const mysteryLights = [];

function createMysteryLight(x,z) {

    const light = new THREE.PointLight(
        0x7a00ff,
        1.5,
        7
    );

    light.position.set(x,2,z);

    scene.add(light);

    mysteryLights.push(light);

}

createMysteryLight(-8,-8);
createMysteryLight(8,0);
createMysteryLight(-8,8);
createMysteryLight(8,8);


// ==================================
// 3. FLOOR
// ==================================

const floor = new THREE.Mesh(
    new THREE.PlaneGeometry(40,40),
    new THREE.MeshStandardMaterial({
        color:0x17151f,
        roughness:0.95
    })
);

floor.rotation.x = -Math.PI/2;

scene.add(floor);


// ==================================
// 4. MAZE WALLS
// ==================================

const walls = [];

function createWall(x,z) {

    const wall = new THREE.Mesh(
        new THREE.BoxGeometry(2,3,2),
        new THREE.MeshStandardMaterial({
            color:0x292632,
            roughness:0.9
        })
    );

    wall.position.set(x,1.5,z);

    wall.castShadow = true;

    scene.add(wall);

    walls.push(wall);

}


// Outer walls

for(let i=-10;i<=10;i+=2) {

    createWall(i,10);

    // Leave an opening for the exit

    if(i !== 0) {
        createWall(i,-10);
    }

    createWall(-10,i);
    createWall(10,i);

}


// Maze layout

const wallPositions = [

    [-4,-8],
    [-4,-6],
    [-4,-4],
    [-4,-2],

    [0,-8],
    [0,-6],
    [0,-4],

    [4,-6],
    [4,-4],
    [4,-2],

    [-6,2],
    [-4,2],
    [-2,2],

    [2,2],
    [4,2],

    [-4,6],
    [-2,6],
    [0,6],

    [6,6],
    [6,4]

];

wallPositions.forEach(p => {
    createWall(p[0],p[1]);
});


// ==================================
// 5. TOKENS
// ==================================

let tokens = [];

let collected = 0;

function createX(x,z,color=0x00ffff) {

    const group = new THREE.Group();

    const material = new THREE.MeshStandardMaterial({
        color:color,
        emissive:color,
        emissiveIntensity:3
    });

    const a = new THREE.Mesh(
        new THREE.BoxGeometry(0.18,0.85,0.18),
        material
    );

    const b = new THREE.Mesh(
        new THREE.BoxGeometry(0.18,0.85,0.18),
        material
    );

    a.rotation.z = Math.PI/4;
    b.rotation.z = -Math.PI/4;

    group.add(a,b);

    group.position.set(x,1.2,z);

    scene.add(group);

    return group;

}

function createToken(x,z) {

    tokens.push(createX(x,z));

}

createToken(-6,-6);
createToken(2,-8);
createToken(6,-2);
createToken(-8,4);
createToken(2,8);


// ==================================
// 6. PUZZLE SYSTEM
// ==================================

let puzzlesSolved = 0;

let currentPuzzle = null;

const puzzles = [

    {
        x:-8,
        z:-2,
        title:"THE FIRST RIDDLE",
        question:
        "I have keys but no locks. I have space but no room. What am I?",
        answer:"keyboard",
        clue:"Somewhere, someone is typing..."
    },

    {
        x:8,
        z:-8,
        title:"THE MATHEMATICAL TEST",
        question:
        "A number multiplied by 4 equals 36. What is the number?",
        answer:"9",
        clue:"The numbers never lie."
    },

    {
        x:-8,
        z:8,
        title:"THE FINAL MYSTERY",
        question:
        "I follow you everywhere, but disappear in darkness. What am I?",
        answer:"shadow",
        clue:"You are never completely alone..."
    }

];

const puzzleObjects = [];

function createPuzzleStation(p) {

    const group = new THREE.Group();

    const base = new THREE.Mesh(
        new THREE.BoxGeometry(0.8,0.8,0.8),
        new THREE.MeshStandardMaterial({
            color:0x24113b,
            emissive:0x5700a8,
            emissiveIntensity:1
        })
    );

    const crystal = new THREE.Mesh(
        new THREE.OctahedronGeometry(0.4),
        new THREE.MeshStandardMaterial({
            color:0xc000ff,
            emissive:0x8500ff,
            emissiveIntensity:4
        })
    );

    crystal.position.y = 0.9;

    group.add(base,crystal);

    group.position.set(p.x,0,p.z);

    scene.add(group);

    puzzleObjects.push({
        group:group,
        data:p,
        solved:false
    });

}

puzzles.forEach(createPuzzleStation);


// ==================================
// 7. SPECIAL PURPLE X
// ==================================

let specialX = null;

let holdingX = false;

function createSpecialX() {

    specialX = createX(8,8,0xff00ff);

    document.getElementById("message").innerHTML =
        "<h2>ALL CHALLENGES COMPLETE!</h2>" +
        "<p>Something purple is waiting for you...</p>";

    document.getElementById("status").innerText =
        "Find the FINAL X.";

}


// ==================================
// 8. INTERACT WITH PUZZLES
// ==================================

function openPuzzle(p) {

    currentPuzzle = p;

    document.getElementById("puzzle").style.display =
        "block";

    document.getElementById("puzzleTitle").innerText =
        p.title;

    document.getElementById("puzzleQuestion").innerText =
        p.question;

    document.getElementById("feedback").innerText = "";

    document.getElementById("answer").value = "";

    document.getElementById("answer").focus();

}

document.getElementById("submit").onclick = function() {

    if(!currentPuzzle) return;

    const answer = document.getElementById("answer")
        .value.trim().toLowerCase();

    if(answer === currentPuzzle.answer) {

        const station = puzzleObjects.find(
            o => o.data === currentPuzzle
        );

        station.solved = true;

        puzzlesSolved++;

        document.getElementById("puzzleCount").innerText =
            puzzlesSolved;

        document.getElementById("feedback").innerText =
            "CORRECT! " + currentPuzzle.clue;

        document.getElementById("feedback").style.color =
            "#8dffb0";

        document.getElementById("status").innerText =
            "Puzzle solved!";

        setTimeout(() => {

            document.getElementById("puzzle").style.display =
                "none";

            currentPuzzle = null;

            checkProgress();

        },1800);

    }

    else {

        document.getElementById("feedback").innerText =
            "Incorrect. The maze is still watching...";

        document.getElementById("feedback").style.color =
            "#ff5555";

    }

};

document.getElementById("closePuzzle").onclick = () => {

    document.getElementById("puzzle").style.display =
        "none";

    currentPuzzle = null;

};


// ==================================
// 9. PROGRESS
// ==================================

function checkProgress() {

    if(collected === 5 && puzzlesSolved === 3) {

        if(!specialX) {

            createSpecialX();

        }

    }

    else if(collected === 5) {

        document.getElementById("status").innerText =
            "Solve all 3 puzzles to reveal the final X.";

    }

    else if(puzzlesSolved === 3) {

        document.getElementById("status").innerText =
            "Now find all 5 X tokens.";

    }

}


// ==================================
// 10. PLAYER CONTROLS
// ==================================

let yaw = 0;
let pitch = 0;

let moveForward = 0;
let moveSide = 0;

const speed = 0.10;

const keys = {};

document.addEventListener("keydown",e => {

    keys[e.key.toLowerCase()] = true;

    if(e.key.toLowerCase() === "e") {
        interact();
    }

});

document.addEventListener("keyup",e => {
    keys[e.key.toLowerCase()] = false;
});


// ==================================
// 11. WORKING MOBILE JOYSTICK
// ==================================

const joystick = document.getElementById("joystick");

const stick = document.getElementById("stick");

let joystickActive = false;

joystick.addEventListener("pointerdown",e => {

    e.preventDefault();

    joystickActive = true;

    joystick.setPointerCapture(e.pointerId);

    updateJoystick(e);

});

joystick.addEventListener("pointermove",e => {

    if(joystickActive) {
        updateJoystick(e);
    }

});

joystick.addEventListener("pointerup",resetJoystick);

joystick.addEventListener("pointercancel",resetJoystick);

function updateJoystick(e) {

    const rect = joystick.getBoundingClientRect();

    let x = e.clientX - rect.left - rect.width/2;

    let y = e.clientY - rect.top - rect.height/2;

    const max = 40;

    const distance = Math.sqrt(x*x+y*y);

    if(distance > max) {

        x = x/distance*max;
        y = y/distance*max;

    }

    stick.style.left = (x+40)+"px";
    stick.style.top = (y+40)+"px";

    moveSide = x/max;

    moveForward = -y/max;

}

function resetJoystick() {

    joystickActive = false;

    moveForward = 0;
    moveSide = 0;

    stick.style.left = "38px";
    stick.style.top = "38px";

}


// ==================================
// 12. LOOK AROUND
// ==================================

let looking = false;

let lastX = 0;
let lastY = 0;

renderer.domElement.addEventListener("pointerdown",e => {

    looking = true;

    lastX = e.clientX;
    lastY = e.clientY;

});

document.addEventListener("pointermove",e => {

    if(!looking) return;

    const dx = e.clientX-lastX;
    const dy = e.clientY-lastY;

    yaw -= dx*0.004;

    pitch -= dy*0.004;

    pitch = Math.max(
        -1.3,
        Math.min(1.3,pitch)
    );

    lastX = e.clientX;
    lastY = e.clientY;

});

document.addEventListener("pointerup",() => {
    looking = false;
});


// ==================================
// 13. COLLISION DETECTION
// ==================================

function checkCollision(position) {

    const radius = 0.35;

    for(const wall of walls) {

        const dx = Math.abs(position.x-wall.position.x);
        const dz = Math.abs(position.z-wall.position.z);

        if(dx < 1+radius && dz < 1+radius) {
            return true;
        }

    }

    return false;

}


// ==================================
// 14. INTERACTION SYSTEM
// ==================================

function interact() {

    if(currentPuzzle) return;

    // Pick up special X

    if(specialX && !holdingX) {

        if(camera.position.distanceTo(
            specialX.position
        ) < 2.5) {

            pickupSpecial();

            return;

        }

    }

    // Find nearest unsolved puzzle

    let nearest = null;

    let nearestDistance = 3;

    for(const station of puzzleObjects) {

        if(station.solved) continue;

        const distance = camera.position.distanceTo(
            station.group.position
        );

        if(distance < nearestDistance) {

            nearest = station;
            nearestDistance = distance;

        }

    }

    if(nearest) {

        openPuzzle(nearest.data);

    }

}

document.getElementById("interact").onclick = interact;


// ==================================
// 15. PICK UP FINAL X
// ==================================

function pickupSpecial() {

    holdingX = true;

    scene.remove(specialX);

    specialX = null;

    const ending = document.getElementById("ending");

    ending.style.display = "flex";

    document.getElementById("endingStory").innerHTML = `
        X served us well.<br><br>

        It multiplied.<br>

        It filled up textbooks.<br>

        It taught us math.<br><br>

        But now, we must say goodbye...<br><br>

        <span style="color:#ff55ff">
        Farewell, X.
        </span>
    `;

    document.getElementById("endingTitle").innerText =
        "A FAREWELL TO X";

    setTimeout(() => {

        ending.style.display = "none";

        document.getElementById("message").innerText =
            "THE FINAL THROW AWAITS.";

        document.getElementById("throw").style.display =
            "block";

        document.getElementById("status").innerText =
            "Throw X into the darkness.";

    },6500);

}


// ==================================
// 16. THROWING X
// ==================================

let projectile = null;

document.getElementById("throw").onclick = function() {

    if(!holdingX) return;

    holdingX = false;

    document.getElementById("throw").style.display =
        "none";

    projectile = createX(
        camera.position.x,
        camera.position.z,
        0xff00ff
    );

    projectile.position.copy(camera.position);

    const direction = new THREE.Vector3();

    camera.getWorldDirection(direction);

    projectile.userData.velocity =
        direction.multiplyScalar(0.32);

};


// ==================================
// 17. DARKNESS AND EXIT
// ==================================

const portal = new THREE.Mesh(

    new THREE.CircleGeometry(1.5,48),

    new THREE.MeshBasicMaterial({
        color:0x000000
    })

);

portal.position.set(0,1.6,-9.7);

scene.add(portal);

let exitOpened = false;

function openExit() {

    if(exitOpened) return;

    exitOpened = true;

    portal.material = new THREE.MeshBasicMaterial({
        color:0x9b00ff
    });

    portal.scale.set(1.7,1.7,1.7);

    scene.fog.density = 0.012;

    document.getElementById("message").innerHTML =
        "<h1>THE DARKNESS HAS ACCEPTED X.</h1>" +
        "<p>THE EXIT IS OPEN!</p>";

    document.getElementById("status").innerText =
        "RUN TO THE EXIT!";

}


// ==================================
// 18. MAIN GAME LOOP
// ==================================

let won = false;

const clock = new THREE.Clock();

function animate() {

    requestAnimationFrame(animate);

    const time = clock.getElapsedTime();

    // Mysterious flickering lights

    mysteryLights.forEach((light,i) => {

        light.intensity =
            1.2 + Math.sin(time*2+i*3)*0.6;

    });


    // Movement

    let forward = moveForward;

    let side = moveSide;

    if(keys["w"]) forward = 1;
    if(keys["s"]) forward = -1;
    if(keys["a"]) side = -1;
    if(keys["d"]) side = 1;

    const direction = new THREE.Vector3(
        Math.sin(yaw),
        0,
        -Math.cos(yaw)
    );

    const right = new THREE.Vector3(
        Math.cos(yaw),
        0,
        Math.sin(yaw)
    );

    const movement = new THREE.Vector3();

    movement.addScaledVector(
        direction,
        forward*speed
    );

    movement.addScaledVector(
        right,
        side*speed
    );

    // Prevent faster diagonal movement

    if(movement.length() > speed) {
        movement.normalize().multiplyScalar(speed);
    }

    const next = camera.position.clone();

    next.add(movement);

    // Check X and Z separately for smoother wall sliding

    const nextX = camera.position.clone();
    nextX.x = next.x;

    if(!checkCollision(nextX)) {
        camera.position.x = next.x;
    }

    const nextZ = camera.position.clone();
    nextZ.z = next.z;

    if(!checkCollision(nextZ)) {
        camera.position.z = next.z;
    }

    camera.rotation.order = "YXZ";

    camera.rotation.y = yaw;
    camera.rotation.x = pitch;


    // Collect tokens

    tokens.forEach((token,index) => {

        token.rotation.y += 0.035;

        token.position.y =
            1.2+Math.sin(time*3+index)*0.15;

        if(camera.position.distanceTo(token.position)<1.3) {

            scene.remove(token);

            tokens = tokens.filter(t=>t!==token);

            collected++;

            document.getElementById("count").innerText =
                collected;

            document.getElementById("status").innerText =
                "Token collected!";

            checkProgress();

        }

    });


    // Puzzle stations

    puzzleObjects.forEach((station,i) => {

        station.group.children[1].rotation.y += 0.025;

        station.group.children[1].position.y =
            0.9+Math.sin(time*2+i)*0.15;

    });


    // Special X

    if(specialX) {

        specialX.rotation.y += 0.035;

        specialX.position.y =
            1.2+Math.sin(time*3)*0.2;

    }


    // Projectile

    if(projectile) {

        projectile.position.add(
            projectile.userData.velocity
        );

        projectile.userData.velocity.y -= 0.004;

        projectile.rotation.y += 0.1;

        // Check if the X reaches the darkness

        if(projectile.position.distanceTo(
            portal.position
        ) < 1.4) {

            scene.remove(projectile);

            projectile = null;

            openExit();

        }

        else if(projectile.position.length()>40) {

            scene.remove(projectile);

            projectile = null;

        }

    }


    // Escape

    if(
        exitOpened &&
        !won &&
        camera.position.z < -9 &&
        Math.abs(camera.position.x)<1.5
    ) {

        won = true;

        document.getElementById("ending").style.display =
            "flex";

        document.getElementById("endingTitle").innerText =
            "YOU ESCAPED!";

        document.getElementById("endingStory").innerHTML =
            "You collected every X.<br>" +
            "You solved every mystery.<br><br>" +
            "You defeated the darkness.<br><br>" +
            "<span style='color:#bb88ff'>" +
            "BUT SOMETHING TELLS YOU..." +
            "<br>THIS WAS ONLY THE BEGINNING." +
            "</span>";

        document.getElementById("throw").style.display =
            "none";

    }

    renderer.render(scene,camera);

}


// ==================================
// 19. RESIZE
// ==================================

window.addEventListener("resize",() => {

    camera.aspect = innerWidth/innerHeight;

    camera.updateProjectionMatrix();

    renderer.setSize(innerWidth,innerHeight);

});

animate();

</script>

</body>
</html>
