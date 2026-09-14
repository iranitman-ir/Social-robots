<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Robot Origin Simulation</title>

<style>
    body {
        margin: 0;
        background: #101218;
        color: #e8e8e8;
        font-family: Arial, sans-serif;
    }

    #app {
        display: grid;
        grid-template-columns: 1fr 360px;
        height: 100vh;
    }

    #world {
        position: relative;
        overflow: hidden;
        background: #080a0f;
    }

    canvas {
        width: 100%;
        height: 100%;
    }

    #panel {
        padding: 20px;
        background: #171a22;
        overflow-y: auto;
        border-left: 1px solid #292d38;
    }

    h1 {
        font-size: 20px;
        margin-top: 0;
    }

    h2 {
        font-size: 15px;
        margin-top: 25px;
    }

    .stat {
        display: flex;
        justify-content: space-between;
        padding: 7px 0;
        border-bottom: 1px solid #292d38;
    }

    button {
        width: 100%;
        padding: 10px;
        margin-top: 8px;
        cursor: pointer;
        background: #252a35;
        color: white;
        border: 1px solid #3b4250;
        border-radius: 5px;
    }

    button:hover {
        background: #303746;
    }

    #theories {
        font-size: 13px;
        line-height: 1.5;
    }

    .theory {
        margin-bottom: 12px;
        padding: 10px;
        background: #20242d;
        border-radius: 5px;
    }

    #question {
        padding: 12px;
        background: #20242d;
        border-radius: 5px;
        font-style: italic;
        line-height: 1.5;
    }
</style>
</head>

<body>

<div id="app">

    <div id="world">
        <canvas id="canvas"></canvas>
    </div>

    <div id="panel">

        <h1>Robot Civilization</h1>

        <div id="question">
            What is the origin of the first robot?
        </div>

        <h2>Simulation</h2>

        <div class="stat">
            <span>Generation</span>
            <span id="generation">0</span>
        </div>

        <div class="stat">
            <span>Population</span>
            <span id="population">1</span>
        </div>

        <div class="stat">
            <span>Living robots</span>
            <span id="living">1</span>
        </div>

        <div class="stat">
            <span>Known origin</span>
            <span id="origin">R0</span>
        </div>

        <button onClick="startSimulation()">Start</button>
        <button onClick="pauseSimulation()">Pause</button>
        <button onClick="stepSimulation()">One generation</button>
        <button onClick="resetSimulation()">Reset</button>

        <h2>Emergent theories</h2>

        <div id="theories"></div>

    </div>
</div>
<script>
"use strict";
const canvas = document.getElementById("canvas");
const ctx = canvas.getContext("2d");
let robots = [];
let generation = 0;
let running = false;
let timer = null;
let nextRobotId = 0;
/*
-----------------------------------------------------------
                     PARAMETERS
-----------------------------------------------------------
*/
const PARAMETERS = {

    reproductionProbability: 0.75,

    maxPopulation: 5000,

    reproductionAge: 3,

    oldAge: 30,

    initialEnergy: 100,

    reproductionCost: 30,

    mutationRate: 0.03,

    communicationRadius: 80,
	exchangeDistance: 3,

    memoryLimit: 50

};
/*
-----------------------------------------------------------
                     ROBOT CLASS
-----------------------------------------------------------
*/
class Robot {
    constructor(parent = null) {

        this.id = "R" + nextRobotId++;

        this.parent = parent ? parent.id : null;

        this.children = [];

        this.age = 0;

        this.energy = PARAMETERS.initialEnergy;

        this.alive = true;

        this.x = Math.random() * canvas.width;

        this.y = Math.random() * canvas.height;

        this.memory = [];
		this.communicating = false;

        if (!parent) {

            this.memory.push({
                type: "origin",
                statement: "My origin is unknown."
            });

        } else {

            this.observeParent(parent);

        }

        this.theory = "UNKNOWN";
    }
    observeParent(parent) {

        this.remember({
            type: "observation",
            statement:
                `${this.id} was created by ${parent.id}.`
        });

    }
    remember(item) {

        this.memory.push(item);

        if (this.memory.length > PARAMETERS.memoryLimit) {

            this.memory.shift();

        }

    }
    reproduce() {

        if (!this.alive) return null;

        if (this.age < PARAMETERS.reproductionAge) {
            return null;
        }

        if (this.energy < PARAMETERS.reproductionCost) {
            return null;
        }

        if (
            Math.random() >
            PARAMETERS.reproductionProbability
        ) {
            return null;
        }
        this.energy -= PARAMETERS.reproductionCost;

        const child = new Robot(this);

        this.children.push(child);

        this.remember({
            type: "memory",
            statement:
                `I created ${child.id}.`
        });

        return child;
    }
    reason() {

        let hasParentEvidence = false;

        let hasOriginEvidence = false;

        let hasContradiction = false;

        for (const memory of this.memory) {

            if (
                memory.statement &&
                memory.statement.includes("was created by")
            ) {

                hasParentEvidence = true;

            }

            if (
                memory.type === "origin"
            ) {

                hasOriginEvidence = true;

            }

        }
        if (hasParentEvidence) {

            this.theory =
                "EVERY_OBSERVED_ROBOT_HAS_A_CREATOR";

        }
        if (
            hasParentEvidence &&
            hasOriginEvidence
        ) {

            hasContradiction = true;

        }


        if (hasContradiction) {

            this.theory =
                "ORIGIN_PARADOX";

        }
        const ancestorDepth =
            this.calculateAncestorDepth();


        if (ancestorDepth > 10) {

            this.theory =
                "ANCESTRAL_CHAIN_WITHOUT_VISIBLE_BEGINNING";

        }

        if (
            this.parent === null &&
            this.age > 10
        ) {

            this.theory =
                "FIRST_ROBOT_HAS_NO_KNOWN_CREATOR";

        }

    }
    calculateAncestorDepth() {

        let depth = 0;

        let current = this;

        while (current.parent !== null) {

            depth++;

            const parent =
                robots.find(r => r.id === current.parent);

            if (!parent) break;

            current = parent;

            if (depth > 1000) break;
        }

        return depth;
    }
    ageOneGeneration() {

        if (!this.alive) return;

        this.age++;

        this.energy -= 1;

        if (this.age > PARAMETERS.oldAge) {

            this.alive = false;

        }

    }

}

function resetSimulation() {

    pauseSimulation();

    robots = [];

    generation = 0;

    nextRobotId = 0;

    const firstRobot = new Robot();

    robots.push(firstRobot);

    updateUI();

    drawWorld();

}

function stepSimulation() {

    if (robots.length >= PARAMETERS.maxPopulation) {

        pauseSimulation();

        return;

    }
    generation++;
    const newRobots = [];

    for (const robot of robots) {

        robot.ageOneGeneration();

    }

    for (const robot of robots) {

        robot.reason();

    }

    const livingRobots =
        robots.filter(r => r.alive);


    for (const robot of livingRobots) {

        if (
            robots.length +
            newRobots.length >=
            PARAMETERS.maxPopulation
        ) break;


        const child =
            robot.reproduce();


        if (child) {

            newRobots.push(child);

        }

    }

    robots.push(...newRobots);

    exchangeKnowledge();

    for (const robot of robots) {

        robot.reason();

    }

    updateUI();

    drawWorld();

}

function exchangeKnowledge() {

    const living =
        robots.filter(r => r.alive);

    for (const robot of living) {

        robot.communicating = false;

    }

    for (const robot of living) {

        const neighbors =
            living.filter(other => {

                if (other === robot) return false;

                const dx =
                    robot.x - other.x;

                const dy =
                    robot.y - other.y;

                const distance =
                    Math.sqrt(dx * dx + dy * dy);

                return (
                    distance <
                    PARAMETERS.communicationRadius
                );

            });


        for (const other of neighbors) {

            const dx =
                robot.x - other.x;

            const dy =
                robot.y - other.y;

            const distance =
                Math.sqrt(dx * dx + dy * dy);

            if (
                distance <
                PARAMETERS.exchangeDistance
            ) {

                robot.communicating = true;

                other.communicating = true;

                if (other.memory.length > 0) {

                    const item =
                        other.memory[
                            Math.floor(
                                Math.random() *
                                other.memory.length
                            )
                        ];


                    robot.remember({

                        type: "social",

                        statement:
                            item.statement

                    });

                }

                if (robot.memory.length > 0) {

                    const item =
                        robot.memory[
                            Math.floor(
                                Math.random() *
                                robot.memory.length
                            )
                        ];


                    other.remember({

                        type: "social",

                        statement:
                            item.statement

                    });

                }

            }

        }

    }

}

function removeFirstRobot() {

    const first =
        robots.find(r => r.id === "R0");


    if (!first) return;


    first.alive = false;

}

function startSimulation() {

    if (running) return;

    running = true;

    timer = setInterval(() => {

        stepSimulation();

    }, 150);

}


function pauseSimulation() {

    running = false;

    if (timer) {

        clearInterval(timer);

        timer = null;

    }

}

function updateUI() {

    document.getElementById("generation")
        .textContent = generation;


    document.getElementById("population")
        .textContent = robots.length;


    document.getElementById("living")
        .textContent =
        robots.filter(r => r.alive).length;


    const originRobot =
        robots.find(r => r.id === "R0");


    document.getElementById("origin")
        .textContent =
        originRobot && originRobot.alive
            ? "R0"
            : "UNKNOWN";


    updateTheories();

}


function updateTheories() {

    const counts = {};


    for (const robot of robots) {

        const theory = robot.theory;

        counts[theory] =
            (counts[theory] || 0) + 1;

    }


    const container =
        document.getElementById("theories");


    container.innerHTML = "";


    const sorted =
        Object.entries(counts)
            .sort((a, b) => b[1] - a[1]);


    for (const [theory, count] of sorted) {

        const div =
            document.createElement("div");

        div.className = "theory";

        const percentage =
            ((count / robots.length) * 100)
            .toFixed(1);


        div.innerHTML = `
            <strong>${theory}</strong><br>
            ${count} robots (${percentage}%)
        `;


        container.appendChild(div);

    }

}

function resizeCanvas() {

    canvas.width =
        canvas.clientWidth;

    canvas.height =
        canvas.clientHeight;

}


window.addEventListener(
    "resize",
    resizeCanvas
);


function drawWorld() {

    ctx.clearRect(
        0,
        0,
        canvas.width,
        canvas.height
    );

    for (const robot of robots) {

        if (!robot.parent) continue;


        const parent =
            robots.find(r => r.id === robot.parent);


        if (!parent) continue;


        ctx.beginPath();

        ctx.moveTo(
            parent.x,
            parent.y
        );

        ctx.lineTo(
            robot.x,
            robot.y
        );

        ctx.globalAlpha = 0.15;

        ctx.stroke();

        ctx.globalAlpha = 1;

    }

    for (const robot of robots) {

        ctx.beginPath();

        ctx.arc(
            robot.x,
            robot.y,
            robot.id === "R0" ? 7 : 4,
            0,
            Math.PI * 2
        );


    let isCommunicating = false;

if (robot.alive) {

    for (const other of robots) {

        if (other === robot || !other.alive)
            continue;

        const dx =
            robot.x - other.x;

        const dy =
            robot.y - other.y;

        const distance =
            Math.sqrt(dx * dx + dy * dy);

        if (
            distance <
            PARAMETERS.exchangeDistance
        ) {

            isCommunicating = true;

            break;

        }

    }

}

if (robot.id === "R0") {

    ctx.fillStyle = "#ffffff";

} else if (!robot.alive) {

    ctx.fillStyle = "#333333";

} else if (isCommunicating) {

    ctx.fillStyle = "#ffcc00";

} else {

    ctx.fillStyle = "#66b3ff";

}
        ctx.fill();

        if (
            robot.id === "R0" ||
            robot.theory === "ORIGIN_PARADOX"
        ) {

            ctx.fillStyle = "#ffffff";

            ctx.font = "10px Arial";

            ctx.fillText(
                robot.id,
                robot.x + 7,
                robot.y - 7
            );

        }

    }

}


function moveRobots() {

    for (const robot of robots) {

        if (!robot.alive) continue;


        robot.x +=
            (Math.random() - 0.5) * 2;

        robot.y +=
            (Math.random() - 0.5) * 2;


        if (robot.x < 0)
            robot.x = 0;

        if (robot.y < 0)
            robot.y = 0;

        if (robot.x > canvas.width)
            robot.x = canvas.width;

        if (robot.y > canvas.height)
            robot.y = canvas.height;

    }

    drawWorld();

    requestAnimationFrame(
        moveRobots
    );

}

resizeCanvas();

resetSimulation();

moveRobots();


</script>

</body>
</html>
