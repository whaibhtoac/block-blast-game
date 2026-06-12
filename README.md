# block-blast-game
block-blast-game
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>Block Blast (Enhanced)</title>
    <style>
        body { background: #111; margin: 0; display: flex; justify-content: center; align-items: center; height: 100vh; font-family: sans-serif; }
        canvas { background: #222; border: 2px solid #555; }
    </style>
</head>
<body>
<canvas id="game" width="480" height="320"></canvas>
<script>
// ====== Canvas ======
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");

// ====== Ball ======
let x = canvas.width / 2;
let y = canvas.height - 30;
let dx = 3;
let dy = -3;
const ballRadius = 8;

// ====== Paddle ======
const paddleWidth = 80;
const paddleHeight = 10;
const paddleBottomMargin = 5; // 下からの浮き
let paddleX = (canvas.width - paddleWidth) / 2;
let rightPressed = false;
let leftPressed = false;

// ====== Bricks ======
const brickRowCount = 5;
const brickColumnCount = 7;
const brickWidth = 55;
const brickHeight = 20;
const brickPadding = 10;
const brickOffsetTop = 40; // スコア表示用に少し下げる
const brickOffsetLeft = 15;
let bricks = [];
for (let c = 0; c < brickColumnCount; c++) {
    bricks[c] = [];
    for (let r = 0; r < brickRowCount; r++) {
        bricks[c][r] = { x: 0, y: 0, status: 1 };
    }
}

// ====== Game State ======
let score = 0;

// ====== Controls ======
document.addEventListener("keydown", e => {
    if (e.key === "ArrowRight") rightPressed = true;
    if (e.key === "ArrowLeft") leftPressed = true;
});
document.addEventListener("keyup", e => {
    if (e.key === "ArrowRight") rightPressed = false;
    if (e.key === "ArrowLeft") leftPressed = false;
});

// ====== Collision ======
function collisionDetection() {
    for (let c = 0; c < brickColumnCount; c++) {
        for (let r = 0; r < brickRowCount; r++) {
            const b = bricks[c][r];
            if (b.status === 1) {
                if (x > b.x && x < b.x + brickWidth && y > b.y && y < b.y + brickHeight) {
                    dy = -dy;
                    b.status = 0;
                    score++;
                    // クリア判定（すべてのブロックを破壊）
                    if (score === brickRowCount * brickColumnCount) {
                        alert("おめでとうございます！クリアです！");
                        document.location.reload();
                    }
                }
            }
        }
    }
}

// ====== Draw ======
function drawBall() {
    ctx.beginPath();
    ctx.arc(x, y, ballRadius, 0, Math.PI * 2);
    ctx.fillStyle = "#00eaff";
    ctx.fill();
    ctx.closePath();
}

function drawPaddle() {
    ctx.beginPath();
    ctx.rect(paddleX, canvas.height - paddleHeight - paddleBottomMargin, paddleWidth, paddleHeight);
    ctx.fillStyle = "#fff";
    ctx.fill();
    ctx.closePath();
}

function drawBricks() {
    for (let c = 0; c < brickColumnCount; c++) {
        for (let r = 0; r < brickRowCount; r++) {
            if (bricks[c][r].status === 1) {
                const brickX = c * (brickWidth + brickPadding) + brickOffsetLeft;
                const brickY = r * (brickHeight + brickPadding) + brickOffsetTop;
                bricks[c][r].x = brickX;
                bricks[c][r].y = brickY;
                ctx.beginPath();
                ctx.rect(brickX, brickY, brickWidth, brickHeight);
                ctx.fillStyle = "#ff6";
                ctx.fill();
                ctx.closePath();
            }
        }
    }
}

function drawScore() {
    ctx.font = "16px Arial";
    ctx.fillStyle = "#fff";
    ctx.fillText(`Score: ${score}`, 8, 20);
}

// ====== Main Loop ======
function draw() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    drawBricks();
    drawBall();
    drawPaddle();
    drawScore();
    collisionDetection();

    // 壁の反射（左右）
    if (x + dx > canvas.width - ballRadius || x + dx < ballRadius) {
        dx = -dx;
    }
    // 壁の反射（天井）
    if (y + dy < ballRadius) {
        dy = -dy;
    } 
    // パドルの高さに基づく衝突判定（修正部分）
    else if (y + dy > canvas.height - paddleHeight - paddleBottomMargin - ballRadius) {
        if (x > paddleX && x < paddleX + paddleWidth) {
            dy = -dy;
            // 補正：パドルにめり込まないように位置を戻す
            y = canvas.height - paddleHeight - paddleBottomMargin - ballRadius;
        } else if (y + dy > canvas.height - ballRadius) {
            alert("GAME OVER");
            document.location.reload();
            return; // ループを停止
        }
    }

    // パドルの移動
    if (rightPressed && paddleX < canvas.width - paddleWidth) paddleX += 6;
    if (leftPressed && paddleX > 0) paddleX -= 6;

    x += dx;
    y += dy;
    requestAnimationFrame(draw);
}

draw();
</script>
</body>
</html>
