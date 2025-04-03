### "p5.js × 生成AI でパックマンを作ろう" の記事が面白かったので、生成AIに編集させてパックマンの口をぱくぱくさせてみた。

元の記事はこちら。

[１．はじめに：p5.js × 生成AI でパックマンを作ろう - Note](https://note.com/nice_llama936/n/n1f884905fe43?sub_rt=share_pw)


生成AIで2000行以上のコードを編集するのがどれだけ大変か自分も体感したかったので、
こちらのシリーズの最後にあるソースコードを生成AIに渡して、パックマンの口をぱくぱくさせる変更を加えさせてみました。

若干の修正（ぱくぱくの速度など）は必要でしたが、ほぼそのまま動きました。
流石にコードの生成には5分以上かかってた気がします笑

元の記事のソースコードも以下のソースコードもp5.jsのエディタにコピペすることでそのまま動きます↓

[p5.js Web Editor](https://editor.p5js.org/)



<details>
<summary>Diff形式で読みにくいですが、変更差分も載せておきます。</summary>

<pre>
65a66,71
> // ▼▼▼ ここから追加：パックマンの口のアニメ用変数 ▼▼▼
> let pacmanMouthOpen = 0;     // 口の開き具合(度数法)
> let pacmanMouthDelta = 10;    // 口の開閉スピード
> let pacmanMouthMax = 45;     // 最大開き角度
> // ▲▲▲ 追加ここまで ▲▲▲
> 
194a201,210
>   // ▼▼▼ ここから追加：パックマンの口の開閉を更新 ▼▼▼
>   // 開き具合をフレームごとに増減させる
>   pacmanMouthOpen += pacmanMouthDelta;
>   if(pacmanMouthOpen < 0 || pacmanMouthOpen > pacmanMouthMax){
>     pacmanMouthDelta *= -1;
>     // はみ出た場合は端で止める
>     pacmanMouthOpen = constrain(pacmanMouthOpen, 0, pacmanMouthMax);
>   }
>   // ▲▲▲ 追加ここまで ▲▲▲
> 
269a286,289
>   // ▼▼▼ 追加：口のアニメ変数リセット ▼▼▼
>   pacmanMouthOpen = 0;
>   // ▲▲▲ 追加ここまで ▲▲▲
> 
329a350,355
> 
>   // ▼▼▼ ここから変更：角度を pacmanMouthOpen を使って可変に ▼▼▼
>   // baseAngle はもとの基本角度差(=90度)の1/2に相当する角度 (45度)
>   // そこから pacmanMouthOpen を加算/減算して口の開き具合を変化させる
>   let baseAngle = 45;                // 元のコードで使われていた固定角度(90/2)
>   let start, end;                    // arc の開始・終了角度(度)
331,333d356
<     case 'left':
<       arc(x,y,size,size, radians(225),radians(135), PIE);
<       break;
335c358,365
<       arc(x,y,size,size, radians(45),radians(315), PIE);
---
>       start = baseAngle - pacmanMouthOpen;       // 45 - α
>       end   = 360 - baseAngle + pacmanMouthOpen; // 315 + α
>       arc(x, y, size, size, radians(start), radians(end), PIE);
>       break;
>     case 'left':
>       start = 180 + baseAngle - pacmanMouthOpen; // 225 - α
>       end   = 540 - baseAngle + pacmanMouthOpen; // 495 + α
>       arc(x, y, size, size, radians(start), radians(end), PIE);
338c368,370
<       arc(x,y,size,size, radians(315),radians(225), PIE);
---
>       start = 270 + baseAngle - pacmanMouthOpen; // 315 - α
>       end   = 630 - baseAngle + pacmanMouthOpen; // 585 + α
>       arc(x, y, size, size, radians(start), radians(end), PIE);
341c373,378
<       arc(x,y,size,size, radians(135),radians(405), PIE);
---
>       // down は元が [135, 405], つまり 270度 の弧
>       // → 中心を 270deg(=135+135) で考えると ± baseAngle
>       //   ここも pacmanMouthOpen を加減してみる
>       start = 90 + baseAngle - pacmanMouthOpen;  // 135 - α
>       end   = 450 - baseAngle + pacmanMouthOpen; // 405 + α
>       arc(x, y, size, size, radians(start), radians(end), PIE);
343a381
>   // ▲▲▲ 変更ここまで ▲▲▲
</pre>

</details>

## ソースコード

```js
// ==================== 迷路関連 ====================
const cols = 28;
const rows = 36;
const cellSize = 20;

// 迷路配列 (省略なし)
const mazeOriginal = [
  [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
  [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
  [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
  [1, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 2, 1, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 2],
  [6, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 6, 6, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 6],
  [6, 8, 1, 5, 5, 2, 8, 1, 5, 5, 5, 2, 8, 6, 6, 8, 1, 5, 5, 5, 2, 8, 1, 5, 5, 2, 8, 6],
  [6, 9, 6, 0, 0, 6, 8, 6, 0, 0, 0, 6, 8, 6, 6, 8, 6, 0, 0, 0, 6, 8, 6, 0, 0, 6, 9, 6],
  [6, 8, 3, 5, 5, 4, 8, 3, 5, 5, 5, 4, 8, 3, 4, 8, 3, 5, 5, 5, 4, 8, 3, 5, 5, 4, 8, 6],
  [6, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 6],
  [6, 8, 1, 5, 5, 2, 8, 1, 2, 8, 1, 5, 5, 5, 5, 5, 5, 2, 8, 1, 2, 8, 1, 5, 5, 2, 8, 6],
  [6, 8, 3, 5, 5, 4, 8, 6, 6, 8, 3, 5, 5, 2, 1, 5, 5, 4, 8, 6, 6, 8, 3, 5, 5, 4, 8, 6],
  [6, 8, 8, 8, 8, 8, 8, 6, 6, 8, 8, 8, 8, 6, 6, 8, 8, 8, 8, 6, 6, 8, 8, 8, 8, 8, 8, 6],
  [3, 5, 5, 5, 5, 2, 8, 6, 3, 5, 5, 2, 0, 6, 6, 0, 1, 5, 5, 4, 6, 8, 1, 5, 5, 5, 5, 4],
  [0, 0, 0, 0, 0, 6, 8, 6, 1, 5, 5, 4, 0, 3, 4, 0, 3, 5, 5, 2, 6, 8, 6, 0, 0, 0, 0, 0],
  [0, 0, 0, 0, 0, 6, 8, 6, 6, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 6, 6, 8, 6, 0, 0, 0, 0, 0],
  [0, 0, 0, 0, 0, 6, 8, 6, 6, 0, 1, 5, 5, 7, 7, 5, 5, 2, 0, 6, 6, 8, 6, 0, 0, 0, 0, 0],
  [5, 5, 5, 5, 5, 4, 8, 3, 4, 0, 6, 0, 0, 0, 0, 0, 0, 6, 0, 3, 4, 8, 3, 5, 5, 5, 5, 5],
  [0, 0, 0, 0, 0, 0, 8, 0, 0, 0, 6, 0, 0, 0, 0, 0, 0, 6, 0, 0, 0, 8, 0, 0, 0, 0, 0, 0],
  [5, 5, 5, 5, 5, 2, 8, 1, 2, 0, 6, 0, 0, 0, 0, 0, 0, 6, 0, 1, 2, 8, 1, 5, 5, 5, 5, 5],
  [0, 0, 0, 0, 0, 6, 8, 6, 6, 0, 3, 5, 5, 5, 5, 5, 5, 4, 0, 6, 6, 8, 6, 0, 0, 0, 0, 0],
  [0, 0, 0, 0, 0, 6, 8, 6, 6, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 6, 6, 8, 6, 0, 0, 0, 0, 0],
  [0, 0, 0, 0, 0, 6, 8, 6, 6, 0, 1, 5, 5, 5, 5, 5, 5, 2, 0, 6, 6, 8, 6, 0, 0, 0, 0, 0],
  [1, 5, 5, 5, 5, 4, 8, 3, 4, 0, 3, 5, 5, 2, 1, 5, 5, 4, 0, 3, 4, 8, 3, 5, 5, 5, 5, 2],
  [6, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 6, 6, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 6],
  [6, 8, 1, 5, 5, 2, 8, 1, 5, 5, 5, 2, 8, 6, 6, 8, 1, 5, 5, 5, 2, 8, 1, 5, 5, 2, 8, 6],
  [6, 8, 3, 5, 2, 6, 8, 3, 5, 5, 5, 4, 8, 3, 4, 8, 3, 5, 5, 5, 4, 8, 6, 1, 5, 4, 8, 6],
  [6, 9, 8, 8, 6, 6, 8, 8, 8, 8, 8, 8, 8, 0, 0, 8, 8, 8, 8, 8, 8, 8, 6, 6, 8, 8, 9, 6],
  [3, 5, 2, 8, 6, 6, 8, 1, 2, 8, 1, 5, 5, 5, 5, 5, 5, 2, 8, 1, 2, 8, 6, 6, 8, 1, 5, 4],
  [1, 5, 4, 8, 3, 4, 8, 6, 6, 8, 3, 5, 5, 2, 1, 5, 5, 4, 8, 6, 6, 8, 3, 4, 8, 3, 5, 2],
  [6, 8, 8, 8, 8, 8, 8, 6, 6, 8, 8, 8, 8, 6, 6, 8, 8, 8, 8, 6, 6, 8, 8, 8, 8, 8, 8, 6],
  [6, 8, 1, 5, 5, 5, 5, 4, 3, 5, 5, 2, 8, 6, 6, 8, 1, 5, 5, 4, 3, 5, 5, 5, 5, 2, 8, 6],
  [6, 8, 3, 5, 5, 5, 5, 5, 5, 5, 5, 4, 8, 3, 4, 8, 3, 5, 5, 5, 5, 5, 5, 5, 5, 4, 8, 6],
  [6, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 8, 6],
  [3, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 5, 4],
  [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
  [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
];

// 実際に使う迷路
let maze = JSON.parse(JSON.stringify(mazeOriginal));

// ==================== ゲーム全体のフラグ ====================
let isGameStarted = false;
let isGameOver    = false;
let isGameCleared = false;

// === ドットカウンター ===
let dotCounter = 0;

// パックマンの変数
let pacmanPixelX, pacmanPixelY;
let pacmanVelX=0, pacmanVelY=0;
let pacmanDirection='left';
let nextDirection='left';

const pacmanSpeed=2.0;
const pacmanSpeedFrightened=2.4;

// ▼▼▼ ここから追加：パックマンの口のアニメ用変数 ▼▼▼
let pacmanMouthOpen = 0;     // 口の開き具合(度数法)
let pacmanMouthDelta = 15;    // 口の開閉スピード
let pacmanMouthMax = 45;     // 最大開き角度
// ▲▲▲ 追加ここまで ▲▲▲

// アカベエ
let akabeePixelX, akabeePixelY;
let akabeeVelX=0, akabeeVelY=0;
let akabeeState='normal'; // 'normal','house','eaten'
let akabeeIsFrightened=false;
const akabeeSpeedNormal=1.6;
const akabeeSpeedFrightened=1.0;
const akabeeSpeedEaten=5.6;
let akabeeSpeed= akabeeSpeedNormal;
let akabeeReturnPath=[];
let akabeeReturnIndex=0;
let akabeeTargetX=0, akabeeTargetY=0;
const AKABEE_SCATTER_ROW=0;
const AKABEE_SCATTER_COL=25;
const AKABEE_HOUSE_ROW=17;
const AKABEE_HOUSE_COL=14;

// ピンキー
let pinkyPixelX, pinkyPixelY;
let pinkyVelX=0, pinkyVelY=0;
let pinkyState='house'; // 'house','normal','eaten'
let pinkyIsFrightened=false;
const pinkySpeedNormal=1.6;
const pinkySpeedFrightened=1.0;
const pinkySpeedEaten=5.6;
let pinkySpeed= pinkySpeedNormal;
let pinkyReturnPath=[];
let pinkyReturnIndex=0;
let pinkyTargetX=0, pinkyTargetY=0;
const PINKY_SCATTER_TARGET= { row:0, col:2 };

// アオスケ
let aosukePixelX, aosukePixelY;
let aosukeVelX=0, aosukeVelY=0;
let aosukeState='house'; // 'house','normal','eaten'
let aosukeIsFrightened=false;
const aosukeSpeedNormal=1.6;
const aosukeSpeedFrightened=1.0;
const aosukeSpeedEaten=5.6;
let aosukeSpeed= aosukeSpeedNormal;
let aosukeReturnPath=[];
let aosukeReturnIndex=0;
let aosukeTargetX=0, aosukeTargetY=0;
const AOSUKE_SCATTER_TARGET= { row:34, col:27 };

// グズタ
let guzzutaPixelX, guzzutaPixelY;
let guzzutaVelX=0, guzzutaVelY=0;
let guzzutaState='house'; // 'house','normal','eaten'
let guzzutaIsFrightened=false;
const guzzutaSpeedNormal=1.6;
const guzzutaSpeedFrightened=1.0;
const guzzutaSpeedEaten=5.6;
let guzzutaSpeed= guzzutaSpeedNormal;
let guzzutaReturnPath=[];
let guzzutaReturnIndex=0;
let guzzutaTargetX=0, guzzutaTargetY=0;
const GUZZUTA_SCATTER_TARGET= { row:34, col:0 };

// ゲームフロー
let gameFlowMode='scatter';
let modeTimer=0;
const scatterDuration=8;
const chaseDuration=20;

const frightenedDuration=8;
const frightenedWhiteTime=1;
let pausedModeBeforeFrightened=null;
let pausedTimerBeforeFrightened=0;

/*--------------------------
  p5.js setup/draw
---------------------------*/
function setup(){
  createCanvas(cols*cellSize, rows*cellSize);
  resetGame();
}

function draw(){
  background(0);

  // 迷路描画
  for(let r=0; r<rows; r++){
    for(let c=0; c<cols; c++){
      drawCell(c*cellSize, r*cellSize, maze[r][c]);
    }
  }

  // GAME CLEAR
  if(isGameCleared){
    fill(255);
    textAlign(CENTER,CENTER);
    textSize(24);
    text("GAME CLEAR", width/2, height/2);

    fill(255);
    textSize(16);
    text("PRESS SPACE TO RESTART", width/2, height-20);

    drawAll();
    return;
  }

  // GAME OVER
  if(isGameOver){
    fill(255);
    textAlign(CENTER,CENTER);
    textSize(24);
    text("GAME OVER", width/2, height/2);

    fill(255);
    textSize(16);
    text("PRESS SPACE TO RESTART", width/2, height-20);

    drawAll();
    return;
  }

  // スタート前
  if(!isGameStarted){
    fill(255);
    textAlign(CENTER,CENTER);
    textSize(16);
    text("PRESS SPACE TO START", width/2, height-20);

    drawAll();
    return;
  }

  // ▼▼▼ ここから追加：パックマンの口の開閉を更新 ▼▼▼
  // 開き具合をフレームごとに増減させる
  pacmanMouthOpen += pacmanMouthDelta;
  if(pacmanMouthOpen < 0 || pacmanMouthOpen > pacmanMouthMax){
    pacmanMouthDelta *= -1;
    // はみ出た場合は端で止める
    pacmanMouthOpen = constrain(pacmanMouthOpen, 0, pacmanMouthMax);
  }
  // ▲▲▲ 追加ここまで ▲▲▲

  // 通常進行
  handlePacman();
  handleGameFlow();

  moveAkabee();
  movePinky();
  moveAosuke();
  moveGuzzuta();

  drawAll();

  // 衝突判定
  checkCollisionPacmanAndAkabee();
  checkCollisionPacmanAndPinky();
  checkCollisionPacmanAndAosuke();
  checkCollisionPacmanAndGuzzuta();

  // ターゲット表示 + 情報
  drawAkabeeTarget();
  drawPinkyTarget();
  drawAosukeTarget();
  drawGuzzutaTarget();
  displayInfo();

  // 全ドット確認
  if(checkAllDotsCleared()){
    isGameCleared=true;
  }
}

function drawAll(){
  drawPacman(pacmanPixelX,pacmanPixelY,pacmanDirection);
  drawAkabee();
  drawPinky();
  drawAosuke();
  drawGuzzuta();
}

/*--------------------------
  keyPressed
---------------------------*/
function keyPressed(){
  if(key===' '){
    if(!isGameStarted || isGameOver || isGameCleared){
      resetGame();
      isGameStarted=true;
      return;
    }
  }
  if(keyCode===LEFT_ARROW)  nextDirection='left';
  if(keyCode===RIGHT_ARROW) nextDirection='right';
  if(keyCode===UP_ARROW)    nextDirection='up';
  if(keyCode===DOWN_ARROW)  nextDirection='down';
}

/*--------------------------
  resetGame
---------------------------*/
function resetGame(){
  isGameStarted=false;
  isGameOver=false;
  isGameCleared=false;

  // 迷路リセット
  maze= JSON.parse(JSON.stringify(mazeOriginal));
  dotCounter=0;

  // パックマン
  pacmanPixelX= 14*cellSize + cellSize/2;
  pacmanPixelY= 26*cellSize + cellSize/2;
  pacmanVelX= -pacmanSpeed;
  pacmanVelY=0;
  pacmanDirection='left';
  nextDirection='left';

  // ▼▼▼ 追加：口のアニメ変数リセット ▼▼▼
  pacmanMouthOpen = 0;
  // ▲▲▲ 追加ここまで ▲▲▲

  // フロー
  gameFlowMode='scatter';
  modeTimer=0;
  pausedModeBeforeFrightened=null;
  pausedTimerBeforeFrightened=0;

  // アカベエ
  akabeePixelX= 14*cellSize+ cellSize/2;
  akabeePixelY= 14*cellSize+ cellSize/2;
  akabeeVelX=0; 
  akabeeVelY=0;
  akabeeState='normal';
  akabeeIsFrightened=false;
  akabeeSpeed= akabeeSpeedNormal;
  akabeeReturnPath=[];
  akabeeReturnIndex=0;

  // ピンキー
  pinkyPixelX= 14*cellSize+ cellSize/2;
  pinkyPixelY= 17*cellSize+ cellSize/2;
  pinkyVelX=0; 
  pinkyVelY=0;
  pinkyState='house';
  pinkyIsFrightened=false;
  pinkySpeed= pinkySpeedNormal;
  pinkyReturnPath=[];
  pinkyReturnIndex=0;

  // アオスケ
  aosukePixelX= 12*cellSize+ cellSize/2; 
  aosukePixelY= 17*cellSize+ cellSize/2;
  aosukeVelX=0; 
  aosukeVelY=0;
  aosukeState='house';
  aosukeIsFrightened=false;
  aosukeSpeed= aosukeSpeedNormal;
  aosukeReturnPath=[];
  aosukeReturnIndex=0;

  // グズタ
  guzzutaPixelX= 16*cellSize+ cellSize/2; 
  guzzutaPixelY= 17*cellSize+ cellSize/2;
  guzzutaVelX=0; 
  guzzutaVelY=0;
  guzzutaState='house';
  guzzutaIsFrightened=false;
  guzzutaSpeed= guzzutaSpeedNormal;
  guzzutaReturnPath=[];
  guzzutaReturnIndex=0;
}

/*--------------------------
  パックマン描画
---------------------------*/
function drawPacman(px,py,direction){
  fill(255,255,0);
  noStroke();
  const size= cellSize*1.2;
  const x= round(px);
  const y= round(py);

  // ▼▼▼ ここから変更：角度を pacmanMouthOpen を使って可変に ▼▼▼
  // baseAngle はもとの基本角度差(=90度)の1/2に相当する角度 (45度)
  // そこから pacmanMouthOpen を加算/減算して口の開き具合を変化させる
  let baseAngle = 45;                // 元のコードで使われていた固定角度(90/2)
  let start, end;                    // arc の開始・終了角度(度)
  switch(direction){
    case 'right':
      start = baseAngle - pacmanMouthOpen;       // 45 - α
      end   = 360 - baseAngle + pacmanMouthOpen; // 315 + α
      arc(x, y, size, size, radians(start), radians(end), PIE);
      break;
    case 'left':
      start = 180 + baseAngle - pacmanMouthOpen; // 225 - α
      end   = 540 - baseAngle + pacmanMouthOpen; // 495 + α
      arc(x, y, size, size, radians(start), radians(end), PIE);
      break;
    case 'up':
      start = 270 + baseAngle - pacmanMouthOpen; // 315 - α
      end   = 630 - baseAngle + pacmanMouthOpen; // 585 + α
      arc(x, y, size, size, radians(start), radians(end), PIE);
      break;
    case 'down':
      // down は元が [135, 405], つまり 270度 の弧
      // → 中心を 270deg(=135+135) で考えると ± baseAngle
      //   ここも pacmanMouthOpen を加減してみる
      start = 90 + baseAngle - pacmanMouthOpen;  // 135 - α
      end   = 450 - baseAngle + pacmanMouthOpen; // 405 + α
      arc(x, y, size, size, radians(start), radians(end), PIE);
      break;
  }
  // ▲▲▲ 変更ここまで ▲▲▲
}

/*--------------------------
  パックマン移動
---------------------------*/
function handlePacman(){
  if(isCenterOfCell(pacmanPixelX,pacmanPixelY)){
    if(canMoveInDirection(nextDirection)){
      pacmanDirection= nextDirection;
      switch(nextDirection){
        case 'left':  pacmanVelX= -pacmanSpeed; pacmanVelY=0;  break;
        case 'right': pacmanVelX=  pacmanSpeed; pacmanVelY=0;  break;
        case 'up':    pacmanVelX=0; pacmanVelY= -pacmanSpeed; break;
        case 'down':  pacmanVelX=0; pacmanVelY=  pacmanSpeed; break;
      }
    }
    if(!canMoveInDirection(pacmanDirection)){
      pacmanVelX=0; 
      pacmanVelY=0;
    }
    eatDotOrPowerPellet();
  }
  pacmanPixelX+= pacmanVelX;
  pacmanPixelY+= pacmanVelY;
  fixPacmanPosition();

  // 左右ワープ
  const gp= getGridPosition(pacmanPixelX,pacmanPixelY);
  if(gp.row===17){
    if(gp.col<0){
      pacmanPixelX= (cols-1)*cellSize+ cellSize/2;
      pacmanPixelY= 17*cellSize+ cellSize/2;
      pacmanVelX= -pacmanSpeed; 
      pacmanVelY=0;
      pacmanDirection='left';
    }
    else if(gp.col>=cols){
      pacmanPixelX= cellSize/2;
      pacmanPixelY= 17*cellSize+ cellSize/2;
      pacmanVelX= pacmanSpeed;  
      pacmanVelY=0;
      pacmanDirection='right';
    }
  }
}

function fixPacmanPosition(){
  const {row,col}= getGridPosition(pacmanPixelX,pacmanPixelY);
  if(pacmanVelX!==0){
    pacmanPixelY= row*cellSize+ cellSize/2;
  }
  if(pacmanVelY!==0){
    pacmanPixelX= col*cellSize+ cellSize/2;
  }
}

/*--------------------------
  イジケモード開始
---------------------------*/
function startFrightenedMode(){
  if(gameFlowMode==='frightened'){
    modeTimer=0;
  } else {
    pausedModeBeforeFrightened= gameFlowMode;
    pausedTimerBeforeFrightened= modeTimer;
    gameFlowMode='frightened';
    modeTimer=0;
  }
  // パックマン速度UP
  pacmanVelX= sign(pacmanVelX)* pacmanSpeedFrightened;
  pacmanVelY= sign(pacmanVelY)* pacmanSpeedFrightened;

  // アカベエ
  akabeeIsFrightened= true;
  if(akabeeState==='normal'){
    akabeeSpeed= akabeeSpeedFrightened;
    akabeeVelX= -akabeeVelX; 
    akabeeVelY= -akabeeVelY;
  }
  // ピンキー
  pinkyIsFrightened= true;
  if(pinkyState==='normal'){
    pinkySpeed= pinkySpeedFrightened;
    pinkyVelX= -pinkyVelX; 
    pinkyVelY= -pinkyVelY;
  }
  // アオスケ
  aosukeIsFrightened= true;
  if(aosukeState==='normal'){
    aosukeSpeed= aosukeSpeedFrightened;
    aosukeVelX= -aosukeVelX; 
    aosukeVelY= -aosukeVelY;
  }
  // グズタ
  guzzutaIsFrightened= true;
  if(guzzutaState==='normal'){
    guzzutaSpeed= guzzutaSpeedFrightened;
    guzzutaVelX= -guzzutaVelX; 
    guzzutaVelY= -guzzutaVelY;
  }
}

/*--------------------------
  ドット・パワーエサ
---------------------------*/
function eatDotOrPowerPellet(){
  const {row,col}= getGridPosition(pacmanPixelX,pacmanPixelY);
  if(maze[row][col]===8){
    maze[row][col]=0;
    dotCounter++;
  }
  else if(maze[row][col]===9){
    maze[row][col]=0;
    dotCounter++;
    startFrightenedMode();
  }
}

/*--------------------------
  アカベエ描画
---------------------------*/
function drawAkabee(){
  const x= round(akabeePixelX);
  const y= round(akabeePixelY);
  const size= cellSize*1.2;
  if(akabeeState==='eaten'){
    drawEatenMonster(x,y,size);
    return;
  }
  if(akabeeIsFrightened && gameFlowMode==='frightened'){
    let remain= frightenedDuration - modeTimer;
    if(remain<= frightenedWhiteTime){
      drawScaredMonster(x,y,size, color(255), color(255,0,0));
    } else {
      drawScaredMonster(x,y,size, color(0,0,255), color(255));
    }
  } else {
    drawMonster(x,y,size, color(255,0,0)); // アカベエ(赤)
  }
}

/*--------------------------
  ピンキー描画
---------------------------*/
function drawPinky(){
  const x= round(pinkyPixelX);
  const y= round(pinkyPixelY);
  const size= cellSize*1.2;
  if(pinkyState==='eaten'){
    drawEatenMonster(x,y,size);
    return;
  }
  if(pinkyIsFrightened && gameFlowMode==='frightened'){
    let remain= frightenedDuration - modeTimer;
    if(remain<= frightenedWhiteTime){
      drawScaredMonster(x,y,size, color(255), color(255,0,0));
    } else {
      drawScaredMonster(x,y,size, color(0,0,255), color(255));
    }
  } else {
    drawMonster(x,y,size, color(255,182,193));
  }
}

/*--------------------------
  アオスケ描画
---------------------------*/
function drawAosuke(){
  const x= round(aosukePixelX);
  const y= round(aosukePixelY);
  const size= cellSize*1.2;
  if(aosukeState==='eaten'){
    drawEatenMonster(x,y,size);
    return;
  }
  if(aosukeIsFrightened && gameFlowMode==='frightened'){
    let remain= frightenedDuration - modeTimer;
    if(remain<= frightenedWhiteTime){
      drawScaredMonster(x,y,size, color(255), color(255,0,0));
    } else {
      drawScaredMonster(x,y,size, color(0,0,255), color(255));
    }
  } else {
    drawMonster(x,y,size, color(0,255,255)); // アオスケ(シアン)
  }
}

/*--------------------------
  グズタ描画
---------------------------*/
function drawGuzzuta(){
  const x= round(guzzutaPixelX);
  const y= round(guzzutaPixelY);
  const size= cellSize*1.2;
  if(guzzutaState==='eaten'){
    drawEatenMonster(x,y,size);
    return;
  }
  if(guzzutaIsFrightened && gameFlowMode==='frightened'){
    let remain= frightenedDuration - modeTimer;
    if(remain<= frightenedWhiteTime){
      drawScaredMonster(x,y,size, color(255), color(255,0,0));
    } else {
      drawScaredMonster(x,y,size, color(0,0,255), color(255));
    }
  } else {
    drawMonster(x,y,size, color(255,165,0)); // グズタ(オレンジ)
  }
}

/*--------------------------
  共通モンスター描画関数
---------------------------*/
function drawMonster(x,y,size, bodyColor){
  const radius= size/2;
  const eyeWidth= size/4;
  const eyeHeight= size/3;
  const pupilSize= eyeWidth/2;

  fill(bodyColor);
  noStroke();
  arc(x,y,size,size, PI,TWO_PI,CHORD);
  rect(x-radius,y,size, radius);

  fill(255);
  ellipse(x- radius/3, y- radius/5, eyeWidth,eyeHeight);
  ellipse(x+ radius/5, y- radius/5, eyeWidth,eyeHeight);

  fill(0);
  ellipse(x- radius/3-(eyeWidth/6), y- radius/5, pupilSize,pupilSize);
  ellipse(x+ radius/5-(eyeWidth/6), y- radius/5, pupilSize,pupilSize);
}

function drawScaredMonster(x,y,size, bodyColor, featureColor){
  const radius= size/2;
  const pupilSize= size/6;
  const waveHeight= size/12;
  const waveY= y+ radius/2 - waveHeight/2;

  fill(bodyColor);
  noStroke();
  arc(x,y,size,size, PI,TWO_PI,CHORD);
  rect(x-radius,y,size, radius);

  fill(featureColor);
  ellipse(x- radius/3, y- radius/5, pupilSize,pupilSize);
  ellipse(x+ radius/5, y- radius/5, pupilSize,pupilSize);

  stroke(featureColor);
  strokeWeight(2);
  noFill();
  beginShape();
  const waveStartX= x-radius + size/10;
  const waveEndX= x+ radius - size/10;
  const step= (waveEndX - waveStartX)/6;
  for(let i=0; i<=6; i++){
    const xPos= waveStartX+ i*step;
    const yPos= waveY+ (i%2===0 ? waveHeight : -waveHeight);
    vertex(xPos,yPos);
  }
  endShape();
}

function drawEatenMonster(x,y,size){
  const eyeSize= size*0.3;
  fill(255);
  noStroke();
  ellipse(x- eyeSize*0.5, y- eyeSize*0.5, eyeSize, eyeSize);
  ellipse(x+ eyeSize*0.5, y- eyeSize*0.5, eyeSize, eyeSize);

  fill(0);
  const pupil= eyeSize*0.4;
  ellipse(x- eyeSize*0.5, y- eyeSize*0.5, pupil,pupil);
  ellipse(x+ eyeSize*0.5, y- eyeSize*0.5, pupil,pupil);
}

/*--------------------------
  アカベエの移動
---------------------------*/
function moveAkabee(){
  switch(akabeeState){
    case 'house':
      moveAkabeeInHouse();
      break;
    case 'normal':
      moveAkabeeNormalOrFrightened();
      break;
    case 'eaten':
      moveAkabeeEaten();
      break;
  }
}

function moveAkabeeInHouse(){
  const gp= getGridPosition(akabeePixelX, akabeePixelY);
  if(gp.row===14 && gp.col===14){
    akabeePixelX= gp.col*cellSize+ cellSize/2;
    akabeePixelY= gp.row*cellSize+ cellSize/2;
    akabeeState='normal';
    akabeeIsFrightened=false; 
    akabeeSpeed= akabeeSpeedNormal;
    akabeeVelX=0; 
    akabeeVelY=0;
    return;
  }
  if(gp.row<14){
    akabeeVelX=0; 
    akabeeVelY= akabeeSpeed;
  }
  else if(gp.row>14){
    akabeeVelX=0; 
    akabeeVelY= -akabeeSpeed;
  }
  else {
    if(gp.col<14){
      akabeeVelX= akabeeSpeed; 
      akabeeVelY=0;
    }
    else if(gp.col>14){
      akabeeVelX= -akabeeSpeed; 
      akabeeVelY=0;
    }
  }
  akabeePixelX+= akabeeVelX;
  akabeePixelY+= akabeeVelY;
}

function moveAkabeeNormalOrFrightened(){
  if(isCenterOfCell(akabeePixelX,akabeePixelY)){
    if(akabeeIsFrightened && gameFlowMode==='frightened'){
      pickAkabeeFrightenedDirection();
    } else {
      pickAkabeeDirection();
    }
  }
  akabeePixelX+= akabeeVelX;
  akabeePixelY+= akabeeVelY;

  // ワープ
  const gp= getGridPosition(akabeePixelX, akabeePixelY);
  if(gp.row===17){
    if(gp.col<0){
      akabeePixelX= (cols-1)*cellSize+ cellSize/2;
      akabeePixelY= 17*cellSize+ cellSize/2;
      akabeeVelX= -akabeeSpeed; 
      akabeeVelY=0;
    }
    else if(gp.col>=cols){
      akabeePixelX= cellSize/2;
      akabeePixelY= 17*cellSize+ cellSize/2;
      akabeeVelX= akabeeSpeed; 
      akabeeVelY=0;
    }
  }
}

function moveAkabeeEaten(){
  if(!akabeeReturnPath || akabeeReturnPath.length===0){
    const gp= getGridPosition(akabeePixelX, akabeePixelY);
    let path= bfsToHouse(gp.row,gp.col,17,14);
    if(!path){
      akabeeVelX=0; 
      akabeeVelY=0;
      return;
    }
    akabeeReturnPath= path;
    akabeeReturnIndex=0;
    akabeeSpeed= akabeeSpeedEaten;
    setNextAkabeeReturnTarget();
  }
  const dx= akabeeTargetX- akabeePixelX;
  const dy= akabeeTargetY- akabeePixelY;
  const dist= sqrt(dx*dx+ dy*dy);
  if(dist< akabeeSpeed){
    akabeePixelX= akabeeTargetX;
    akabeePixelY= akabeeTargetY;
    akabeeReturnIndex++;
    if(akabeeReturnIndex>= akabeeReturnPath.length){
      akabeeState='house';
      akabeeVelX=0; 
      akabeeVelY=0;
      akabeeIsFrightened=false; 
      akabeeSpeed= akabeeSpeedNormal;

      akabeeReturnPath=[];
      akabeeReturnIndex=0;
      return;
    }
    setNextAkabeeReturnTarget();
  }
  else {
    let vx= (dx/dist)* akabeeSpeed;
    let vy= (dy/dist)* akabeeSpeed;
    akabeePixelX+= vx; 
    akabeePixelY+= vy;
  }
}

function setNextAkabeeReturnTarget(){
  if(!akabeeReturnPath) return;
  if(akabeeReturnIndex< akabeeReturnPath.length){
    let cell= akabeeReturnPath[akabeeReturnIndex];
    akabeeTargetX= cell.col* cellSize+ cellSize/2;
    akabeeTargetY= cell.row* cellSize+ cellSize/2;
  }
}

function pickAkabeeDirection(){
  if(akabeeState!=='normal') return;

  if(akabeeIsFrightened && gameFlowMode==='frightened'){
    pickAkabeeFrightenedDirection();
    return;
  }

  const gp= getGridPosition(akabeePixelX, akabeePixelY);
  const currDir= { x: sign(akabeeVelX), y: sign(akabeeVelY) };
  const oppDir=  { x: -currDir.x, y: -currDir.y };

  // gameFlowMode
  let effectiveMode= gameFlowMode;
  if(gameFlowMode==='frightened' && !akabeeIsFrightened){
    if(pausedModeBeforeFrightened==='scatter' || pausedModeBeforeFrightened==='chase'){
      effectiveMode= pausedModeBeforeFrightened;
    }
    else {
      effectiveMode='scatter';
    }
  }

  let targetRow, targetCol;
  if(effectiveMode==='scatter'){
    targetRow= AKABEE_SCATTER_ROW;
    targetCol= AKABEE_SCATTER_COL;
  } else {
    // chase => パックマン位置
    let pm= getGridPosition(pacmanPixelX,pacmanPixelY);
    targetRow= pm.row;
    targetCol= pm.col;
  }

  let directions= [
    {x:0,y:-1},
    {x:0,y:1},
    {x:-1,y:0},
    {x:1,y:0}
  ];
  let candidates=[];
  for(let d of directions){
    if(d.x===oppDir.x && d.y===oppDir.y) continue;
    let nr= gp.row+ d.y;
    let nc= gp.col+ d.x;
    if(isWall(nr,nc)) continue;

    let dx= (nc+0.5)- (targetCol+0.5);
    let dy= (nr+0.5)- (targetRow+0.5);
    let distSq= dx*dx+ dy*dy;
    candidates.push({dir:d, dist: distSq});
  }
  if(candidates.length===0){
    akabeeVelX=0; 
    akabeeVelY=0;
  }
  else {
    candidates.sort((a,b)=> a.dist- b.dist);
    let best= candidates[0].dir;
    akabeeVelX= best.x* akabeeSpeed;
    akabeeVelY= best.y* akabeeSpeed;
  }
}

function pickAkabeeFrightenedDirection(){
  const gp= getGridPosition(akabeePixelX, akabeePixelY);
  const currDir= { x: sign(akabeeVelX), y: sign(akabeeVelY) };
  const oppDir=  { x:-currDir.x, y:-currDir.y };

  let directions= [
    {x:0,y:-1},
    {x:0,y:1},
    {x:-1,y:0},
    {x:1,y:0}
  ];
  let candidates=[];
  for(let d of directions){
    if(d.x===oppDir.x && d.y===oppDir.y) continue;
    let nr= gp.row+ d.y;
    let nc= gp.col+ d.x;
    if(isWall(nr,nc)) continue;
    candidates.push(d);
  }
  if(candidates.length===0){
    akabeeVelX=0; 
    akabeeVelY=0;
  }
  else {
    const idx= floor(random(candidates.length));
    let pick= candidates[idx];
    akabeeVelX= pick.x* akabeeSpeed;
    akabeeVelY= pick.y* akabeeSpeed;
  }
}

/*--------------------------
  ピンキーの移動
---------------------------*/
function movePinky(){
  switch(pinkyState){
    case 'house':
      movePinkyInHouse();
      break;
    case 'normal':
      movePinkyNormalOrFrightened();
      break;
    case 'eaten':
      movePinkyEaten();
      break;
  }
}

function movePinkyInHouse(){
  const gp= getGridPosition(pinkyPixelX,pinkyPixelY);
  if(gp.row===14 && gp.col===14){
    pinkyPixelX= gp.col* cellSize+ cellSize/2;
    pinkyPixelY= gp.row* cellSize+ cellSize/2;
    pinkyState='normal';
    if(pinkyIsFrightened && gameFlowMode==='frightened'){
      pinkySpeed= pinkySpeedFrightened;
    } else {
      pinkyIsFrightened=false;
      pinkySpeed= pinkySpeedNormal;
    }
    pinkyVelX=0; 
    pinkyVelY=0;
    return;
  }
  if(gp.row<14){
    pinkyVelX=0; 
    pinkyVelY= pinkySpeed;
  }
  else if(gp.row>14){
    pinkyVelX=0; 
    pinkyVelY= -pinkySpeed;
  }
  else {
    if(gp.col<14){
      pinkyVelX= pinkySpeed; 
      pinkyVelY=0;
    }
    else if(gp.col>14){
      pinkyVelX= -pinkySpeed; 
      pinkyVelY=0;
    }
  }
  pinkyPixelX+= pinkyVelX; 
  pinkyPixelY+= pinkyVelY;
}

function movePinkyNormalOrFrightened(){
  if(isCenterOfCell(pinkyPixelX,pinkyPixelY)){
    if(pinkyIsFrightened && gameFlowMode==='frightened'){
      pickGhostFrightenedDirection('pinky');
    } else {
      pickPinkyDirection();
    }
  }
  pinkyPixelX+= pinkyVelX; 
  pinkyPixelY+= pinkyVelY;

  const gp= getGridPosition(pinkyPixelX,pinkyPixelY);
  if(gp.row===17){
    if(gp.col<0){
      pinkyPixelX= (cols-1)*cellSize+ cellSize/2;
      pinkyPixelY= 17*cellSize+ cellSize/2;
      pinkyVelX= -pinkySpeed; 
      pinkyVelY=0;
    }
    else if(gp.col>=cols){
      pinkyPixelX= cellSize/2;
      pinkyPixelY= 17*cellSize+ cellSize/2;
      pinkyVelX= pinkySpeed;  
      pinkyVelY=0;
    }
  }
}

function movePinkyEaten(){
  if(!pinkyReturnPath || pinkyReturnPath.length===0){
    const gp= getGridPosition(pinkyPixelX,pinkyPixelY);
    let path= bfsToHouse(gp.row,gp.col,17,14);
    if(!path){
      pinkyVelX=0; 
      pinkyVelY=0;
      return;
    }
    pinkyReturnPath= path;
    pinkyReturnIndex=0;
    pinkySpeed= pinkySpeedEaten;
    setNextPinkyReturnTarget();
  }
  const dx= pinkyTargetX- pinkyPixelX;
  const dy= pinkyTargetY- pinkyPixelY;
  const dist= sqrt(dx*dx+ dy*dy);
  if(dist< pinkySpeed){
    pinkyPixelX= pinkyTargetX;
    pinkyPixelY= pinkyTargetY;
    pinkyReturnIndex++;
    if(pinkyReturnIndex>= pinkyReturnPath.length){
      pinkyState='house';
      pinkyVelX=0; 
      pinkyVelY=0;

      pinkyIsFrightened=false;
      pinkySpeed= pinkySpeedNormal;

      pinkyReturnPath=[];
      pinkyReturnIndex=0;
      return;
    }
    setNextPinkyReturnTarget();
  }
  else {
    let vx= (dx/dist)* pinkySpeed;
    let vy= (dy/dist)* pinkySpeed;
    pinkyPixelX+= vx; 
    pinkyPixelY+= vy;
  }
}

function setNextPinkyReturnTarget(){
  if(!pinkyReturnPath) return;
  if(pinkyReturnIndex< pinkyReturnPath.length){
    let cell= pinkyReturnPath[pinkyReturnIndex];
    pinkyTargetX= cell.col* cellSize+ cellSize/2;
    pinkyTargetY= cell.row* cellSize+ cellSize/2;
  }
}

// 追尾ロジック
function pickPinkyDirection(){
  const gp= getGridPosition(pinkyPixelX,pinkyPixelY);
  const currDir= { x: sign(pinkyVelX), y: sign(pinkyVelY) };
  const oppDir=  { x: -currDir.x, y:-currDir.y };

  // イジケではないが gameFlowMode==='frightened' の場合 => pausedModeBeforeFrightened
  let effectiveMode= gameFlowMode;
  if(gameFlowMode==='frightened' && !pinkyIsFrightened){
    if(pausedModeBeforeFrightened==='scatter' || pausedModeBeforeFrightened==='chase'){
      effectiveMode= pausedModeBeforeFrightened;
    } else {
      effectiveMode='scatter';
    }
  }

  let target=null;
  if(effectiveMode==='scatter'){
    target= { row:PINKY_SCATTER_TARGET.row, col:PINKY_SCATTER_TARGET.col };
  }
  else if(effectiveMode==='chase'){
    let pm= getGridPosition(pacmanPixelX,pacmanPixelY);
    let pdir= pacmanDirection;
    let ahead= {row:pm.row, col:pm.col};
    switch(pdir){
      case 'left':  ahead.col-=4; break;
      case 'right': ahead.col+=4; break;
      case 'up':    ahead.row-=4; break;
      case 'down':  ahead.row+=4; break;
    }
    target= ahead;
  }
  else {
    // 何も該当しないなら抜ける
    return;
  }

  let directions= [
    {x:0,y:-1},
    {x:0,y:1},
    {x:-1,y:0},
    {x:1,y:0}
  ];
  let candidates=[];
  for(let d of directions){
    if(d.x===oppDir.x && d.y===oppDir.y) continue;
    let nr= gp.row+ d.y;
    let nc= gp.col+ d.x;
    if(isWall(nr,nc)) continue;

    let dx= (nc+0.5)-(target.col+0.5);
    let dy= (nr+0.5)-(target.row+0.5);
    let distSq= dx*dx+ dy*dy;
    candidates.push({dir:d, dist:distSq});
  }
  if(candidates.length===0){
    pinkyVelX=0; 
    pinkyVelY=0;
  }
  else {
    candidates.sort((a,b)=> a.dist- b.dist);
    let best= candidates[0].dir;
    pinkyVelX= best.x* pinkySpeed;
    pinkyVelY= best.y* pinkySpeed;
  }
}

/*--------------------------
  アオスケの移動
---------------------------*/
function moveAosuke(){
  switch(aosukeState){
    case 'house':
      moveAosukeInHouse();
      break;
    case 'normal':
      moveAosukeNormalOrFrightened();
      break;
    case 'eaten':
      moveAosukeEaten();
      break;
  }
}

function moveAosukeInHouse(){
  if(dotCounter<30){
    aosukeVelX=0; 
    aosukeVelY=0;
    return;
  }
  const gp= getGridPosition(aosukePixelX,aosukePixelY);
  if(gp.row===14 && gp.col===14){
    aosukePixelX= gp.col* cellSize+ cellSize/2;
    aosukePixelY= gp.row* cellSize+ cellSize/2;
    aosukeState='normal';
    if(aosukeIsFrightened && gameFlowMode==='frightened'){
      aosukeSpeed= aosukeSpeedFrightened;
    } else {
      aosukeIsFrightened=false;
      aosukeSpeed= aosukeSpeedNormal;
    }
    aosukeVelX=0; 
    aosukeVelY=0;
    return;
  }
  if(gp.row<14){
    aosukeVelX=0; 
    aosukeVelY= aosukeSpeed;
  }
  else if(gp.row>14){
    aosukeVelX=0; 
    aosukeVelY= -aosukeSpeed;
  }
  else {
    if(gp.col<14){
      aosukeVelX= aosukeSpeed; 
      aosukeVelY=0;
    }
    else if(gp.col>14){
      aosukeVelX= -aosukeSpeed; 
      aosukeVelY=0;
    }
  }
  aosukePixelX+= aosukeVelX;
  aosukePixelY+= aosukeVelY;
}

function moveAosukeNormalOrFrightened(){
  if(isCenterOfCell(aosukePixelX,aosukePixelY)){
    if(aosukeIsFrightened && gameFlowMode==='frightened'){
      pickGhostFrightenedDirection('aosuke');
    } else {
      pickAosukeDirection();
    }
  }
  aosukePixelX+= aosukeVelX;
  aosukePixelY+= aosukeVelY;

  // ワープ
  const gp= getGridPosition(aosukePixelX,aosukePixelY);
  if(gp.row===17){
    if(gp.col<0){
      aosukePixelX= (cols-1)*cellSize+ cellSize/2;
      aosukePixelY= 17*cellSize+ cellSize/2;
      aosukeVelX= -aosukeSpeed; 
      aosukeVelY=0;
    }
    else if(gp.col>=cols){
      aosukePixelX= cellSize/2;
      aosukePixelY= 17*cellSize+ cellSize/2;
      aosukeVelX= aosukeSpeed;  
      aosukeVelY=0;
    }
  }
}

function moveAosukeEaten(){
  if(!aosukeReturnPath || aosukeReturnPath.length===0){
    const gp= getGridPosition(aosukePixelX,aosukePixelY);
    let path= bfsToHouse(gp.row,gp.col,17,14);
    if(!path){
      aosukeVelX=0; 
      aosukeVelY=0;
      return;
    }
    aosukeReturnPath= path;
    aosukeReturnIndex=0;
    aosukeSpeed= aosukeSpeedEaten;
    setNextAosukeReturnTarget();
  }
  const dx= aosukeTargetX- aosukePixelX;
  const dy= aosukeTargetY- aosukePixelY;
  const dist= sqrt(dx*dx+ dy*dy);
  if(dist< aosukeSpeed){
    aosukePixelX= aosukeTargetX;
    aosukePixelY= aosukeTargetY;
    aosukeReturnIndex++;
    if(aosukeReturnIndex>= aosukeReturnPath.length){
      aosukeState='house';
      aosukeVelX=0; 
      aosukeVelY=0;

      aosukeIsFrightened=false;
      aosukeSpeed= aosukeSpeedNormal;

      aosukeReturnPath=[];
      aosukeReturnIndex=0;
      return;
    }
    setNextAosukeReturnTarget();
  }
  else {
    let vx= (dx/dist)* aosukeSpeed;
    let vy= (dy/dist)* aosukeSpeed;
    aosukePixelX+= vx; 
    aosukePixelY+= vy;
  }
}

function setNextAosukeReturnTarget(){
  if(!aosukeReturnPath) return;
  if(aosukeReturnIndex< aosukeReturnPath.length){
    let cell= aosukeReturnPath[aosukeReturnIndex];
    aosukeTargetX= cell.col* cellSize+ cellSize/2;
    aosukeTargetY= cell.row* cellSize+ cellSize/2;
  }
}

function pickAosukeDirection(){
  const gp= getGridPosition(aosukePixelX,aosukePixelY);
  const currDir= { x: sign(aosukeVelX), y: sign(aosukeVelY) };
  const oppDir= { x:-currDir.x, y:-currDir.y };

  let effectiveMode= gameFlowMode;
  if(gameFlowMode==='frightened' && !aosukeIsFrightened){
    if(pausedModeBeforeFrightened==='scatter' || pausedModeBeforeFrightened==='chase'){
      effectiveMode= pausedModeBeforeFrightened;
    } else {
      effectiveMode='scatter';
    }
  }

  let target=null;
  if(effectiveMode==='scatter'){
    target= AOSUKE_SCATTER_TARGET;
  }
  else if(effectiveMode==='chase'){
    const pm= getGridPosition(pacmanPixelX,pacmanPixelY);
    let ahead= { row:pm.row, col:pm.col };
    switch(pacmanDirection){
      case 'left':  ahead.col-=2; break;
      case 'right': ahead.col+=2; break;
      case 'up':    ahead.row-=2; break;
      case 'down':  ahead.row+=2; break;
    }
    const ak= getGridPosition(akabeePixelX, akabeePixelY);
    const rowT= 2*ahead.row - ak.row;
    const colT= 2*ahead.col - ak.col;
    target= {row:rowT, col:colT};
  }
  else {
    return;
  }

  let dirs= [{x:0,y:-1},{x:0,y:1},{x:-1,y:0},{x:1,y:0}];
  let candidates=[];
  for(let d of dirs){
    if(d.x===oppDir.x && d.y===oppDir.y) continue;
    let nr= gp.row+ d.y;
    let nc= gp.col+ d.x;
    if(isWall(nr,nc)) continue;

    let dx= (nc+0.5)-(target.col+0.5);
    let dy= (nr+0.5)-(target.row+0.5);
    let distSq= dx*dx + dy*dy;
    candidates.push({dir:d, dist: distSq});
  }
  if(candidates.length===0){
    aosukeVelX=0; 
    aosukeVelY=0;
  }
  else {
    candidates.sort((a,b)=> a.dist- b.dist);
    let best= candidates[0].dir;
    aosukeVelX= best.x* aosukeSpeed;
    aosukeVelY= best.y* aosukeSpeed;
  }
}

/*--------------------------
  グズタの移動
---------------------------*/
function moveGuzzuta(){
  switch(guzzutaState){
    case 'house':
      moveGuzzutaInHouse();
      break;
    case 'normal':
      moveGuzzutaNormalOrFrightened();
      break;
    case 'eaten':
      moveGuzzutaEaten();
      break;
  }
}

function moveGuzzutaInHouse(){
  if(dotCounter<60){
    guzzutaVelX=0; 
    guzzutaVelY=0;
    return;
  }
  const gp= getGridPosition(guzzutaPixelX,guzzutaPixelY);
  if(gp.row===14 && gp.col===14){
    guzzutaPixelX= gp.col* cellSize+ cellSize/2;
    guzzutaPixelY= gp.row* cellSize+ cellSize/2;
    guzzutaState='normal';
    if(guzzutaIsFrightened && gameFlowMode==='frightened'){
      guzzutaSpeed= guzzutaSpeedFrightened;
    } else {
      guzzutaIsFrightened=false;
      guzzutaSpeed= guzzutaSpeedNormal;
    }
    guzzutaVelX=0; 
    guzzutaVelY=0;
    return;
  }
  if(gp.row<14){
    guzzutaVelX=0; 
    guzzutaVelY= guzzutaSpeed;
  }
  else if(gp.row>14){
    guzzutaVelX=0; 
    guzzutaVelY= -guzzutaSpeed;
  }
  else {
    if(gp.col<14){
      guzzutaVelX= guzzutaSpeed; 
      guzzutaVelY=0;
    }
    else if(gp.col>14){
      guzzutaVelX= -guzzutaSpeed; 
      guzzutaVelY=0;
    }
  }
  guzzutaPixelX+= guzzutaVelX; 
  guzzutaPixelY+= guzzutaVelY;
}

function moveGuzzutaNormalOrFrightened(){
  if(isCenterOfCell(guzzutaPixelX,guzzutaPixelY)){
    if(guzzutaIsFrightened && gameFlowMode==='frightened'){
      pickGhostFrightenedDirection('guzzuta');
    } else {
      pickGuzzutaDirection();
    }
  }
  guzzutaPixelX+= guzzutaVelX;
  guzzutaPixelY+= guzzutaVelY;

  const gp= getGridPosition(guzzutaPixelX,guzzutaPixelY);
  if(gp.row===17){
    if(gp.col<0){
      guzzutaPixelX= (cols-1)*cellSize+ cellSize/2;
      guzzutaPixelY= 17*cellSize+ cellSize/2;
      guzzutaVelX= -guzzutaSpeed; 
      guzzutaVelY=0;
    }
    else if(gp.col>=cols){
      guzzutaPixelX= cellSize/2;
      guzzutaPixelY= 17*cellSize+ cellSize/2;
      guzzutaVelX= guzzutaSpeed;  
      guzzutaVelY=0;
    }
  }
}

function moveGuzzutaEaten(){
  if(!guzzutaReturnPath || guzzutaReturnPath.length===0){
    const gp= getGridPosition(guzzutaPixelX,guzzutaPixelY);
    let path= bfsToHouse(gp.row,gp.col,17,14);
    if(!path){
      guzzutaVelX=0; 
      guzzutaVelY=0;
      return;
    }
    guzzutaReturnPath= path;
    guzzutaReturnIndex=0;
    guzzutaSpeed= guzzutaSpeedEaten;
    setNextGuzzutaReturnTarget();
  }
  const dx= guzzutaTargetX- guzzutaPixelX;
  const dy= guzzutaTargetY- guzzutaPixelY;
  const dist= sqrt(dx*dx+ dy*dy);
  if(dist< guzzutaSpeed){
    guzzutaPixelX= guzzutaTargetX;
    guzzutaPixelY= guzzutaTargetY;
    guzzutaReturnIndex++;
    if(guzzutaReturnIndex>= guzzutaReturnPath.length){
      guzzutaState='house';
      guzzutaVelX=0; 
      guzzutaVelY=0;

      guzzutaIsFrightened=false;
      guzzutaSpeed= guzzutaSpeedNormal;

      guzzutaReturnPath=[];
      guzzutaReturnIndex=0;
      return;
    }
    setNextGuzzutaReturnTarget();
  }
  else {
    let vx= (dx/dist)* guzzutaSpeed;
    let vy= (dy/dist)* guzzutaSpeed;
    guzzutaPixelX+= vx; 
    guzzutaPixelY+= vy;
  }
}

function setNextGuzzutaReturnTarget(){
  if(!guzzutaReturnPath) return;
  if(guzzutaReturnIndex< guzzutaReturnPath.length){
    let cell= guzzutaReturnPath[guzzutaReturnIndex];
    guzzutaTargetX= cell.col* cellSize + cellSize/2;
    guzzutaTargetY= cell.row* cellSize + cellSize/2;
  }
}

function pickGuzzutaDirection(){
  const gp= getGridPosition(guzzutaPixelX,guzzutaPixelY);
  const currDir= { x: sign(guzzutaVelX), y: sign(guzzutaVelY) };
  const oppDir= { x:-currDir.x, y:-currDir.y };

  // "frightened" だけど isFrightened===false => pausedModeBeforeFrightened
  let effectiveMode= gameFlowMode;
  if(gameFlowMode==='frightened' && !guzzutaIsFrightened){
    if(pausedModeBeforeFrightened==='scatter' || pausedModeBeforeFrightened==='chase'){
      effectiveMode= pausedModeBeforeFrightened;
    }
    else {
      effectiveMode='scatter';
    }
  }

  let target=null;
  if(effectiveMode==='scatter'){
    target= GUZZUTA_SCATTER_TARGET;
  }
  else if(effectiveMode==='chase'){
    // 160px以上離れていればパックマン
    const dx= pacmanPixelX- guzzutaPixelX;
    const dy= pacmanPixelY- guzzutaPixelY;
    const dist= sqrt(dx*dx+ dy*dy);
    if(dist>160){
      let pm= getGridPosition(pacmanPixelX,pacmanPixelY);
      target= {row:pm.row,col:pm.col};
    } else {
      target= {row:GUZZUTA_SCATTER_TARGET.row,col:GUZZUTA_SCATTER_TARGET.col};
    }
  }
  else {
    return;
  }

  let directions= [
    {x:0,y:-1},
    {x:0,y:1},
    {x:-1,y:0},
    {x:1,y:0}
  ];
  let candidates=[];
  for(let d of directions){
    if(d.x===oppDir.x && d.y===oppDir.y) continue;
    let nr= gp.row+ d.y;
    let nc= gp.col+ d.x;
    if(isWall(nr,nc)) continue;
    let ddx= (nc+0.5)-(target.col+0.5);
    let ddy= (nr+0.5)-(target.row+0.5);
    let distSq= ddx*ddx+ ddy*ddy;
    candidates.push({dir:d, dist:distSq});
  }
  if(candidates.length===0){
    guzzutaVelX=0; 
    guzzutaVelY=0;
  }
  else {
    candidates.sort((a,b)=> a.dist- b.dist);
    let best= candidates[0].dir;
    guzzutaVelX= best.x*guzzutaSpeed;
    guzzutaVelY= best.y*guzzutaSpeed;
  }
}

/*--------------------------
  ゴースト共通: イジケランダム
---------------------------*/
function pickGhostFrightenedDirection(who){
  let px,py,vx,vy,speed;
  if(who==='pinky'){
    px= pinkyPixelX; py= pinkyPixelY; vx= pinkyVelX; vy= pinkyVelY; speed= pinkySpeed;
  }
  else if(who==='aosuke'){
    px= aosukePixelX; py= aosukePixelY; vx= aosukeVelX; vy= aosukeVelY; speed= aosukeSpeed;
  }
  else if(who==='guzzuta'){
    px= guzzutaPixelX; py= guzzutaPixelY; vx= guzzutaVelX; vy= guzzutaVelY; speed= guzzutaSpeed;
  }
  else {
    return;
  }

  const gp= getGridPosition(px,py);
  const currDir= { x: sign(vx), y: sign(vy) };
  const oppDir=  { x: -currDir.x, y: -currDir.y };

  let directions= [
    {x:0,y:-1},
    {x:0,y:1},
    {x:-1,y:0},
    {x:1,y:0}
  ];
  let candidates=[];
  for(let d of directions){
    if(d.x===oppDir.x && d.y===oppDir.y) continue;
    let nr= gp.row+ d.y;
    let nc= gp.col+ d.x;
    if(isWall(nr,nc)) continue;
    candidates.push(d);
  }
  if(candidates.length===0){
    vx=0; vy=0;
  }
  else {
    const idx= floor(random(candidates.length));
    let pick= candidates[idx];
    vx= pick.x* speed;
    vy= pick.y* speed;
  }

  if(who==='pinky'){
    pinkyVelX=vx; 
    pinkyVelY=vy;
  }
  else if(who==='aosuke'){
    aosukeVelX=vx; 
    aosukeVelY=vy;
  }
  else if(who==='guzzuta'){
    guzzutaVelX=vx; 
    guzzutaVelY=vy;
  }
}

/*--------------------------
  衝突判定
---------------------------*/
function checkCollisionPacmanAndAkabee(){
  if(akabeeState==='eaten') return;
  const dx= pacmanPixelX- akabeePixelX;
  const dy= pacmanPixelY- akabeePixelY;
  const distSq= dx*dx+ dy*dy;
  const radius=10;
  if(distSq<= radius*radius){
    if(akabeeIsFrightened && akabeeState==='normal'){
      akabeeState='eaten';
      akabeeSpeed= akabeeSpeedEaten;
      akabeeReturnPath=[];
      akabeeReturnIndex=0;
    }
    else if(akabeeState==='normal'){
      isGameOver= true;
    }
  }
}

function checkCollisionPacmanAndPinky(){
  if(pinkyState==='eaten') return;
  const dx= pacmanPixelX- pinkyPixelX;
  const dy= pacmanPixelY- pinkyPixelY;
  const distSq= dx*dx+ dy*dy;
  const radius=10;
  if(distSq<= radius*radius){
    if(pinkyIsFrightened && pinkyState==='normal'){
      pinkyState='eaten';
      pinkySpeed= pinkySpeedEaten;
      pinkyReturnPath=[];
      pinkyReturnIndex=0;
    }
    else if(pinkyState==='normal'){
      isGameOver= true;
    }
  }
}

function checkCollisionPacmanAndAosuke(){
  if(aosukeState==='eaten') return;
  const dx= pacmanPixelX- aosukePixelX;
  const dy= pacmanPixelY- aosukePixelY;
  const distSq= dx*dx+ dy*dy;
  const radius=10;
  if(distSq<= radius*radius){
    if(aosukeIsFrightened && aosukeState==='normal'){
      aosukeState='eaten';
      aosukeSpeed= aosukeSpeedEaten;
      aosukeReturnPath=[];
      aosukeReturnIndex=0;
    }
    else if(aosukeState==='normal'){
      isGameOver= true;
    }
  }
}

function checkCollisionPacmanAndGuzzuta(){
  if(guzzutaState==='eaten') return;
  const dx= pacmanPixelX- guzzutaPixelX;
  const dy= pacmanPixelY- guzzutaPixelY;
  const distSq= dx*dx+ dy*dy;
  const radius=10;
  if(distSq<= radius*radius){
    if(guzzutaIsFrightened && guzzutaState==='normal'){
      guzzutaState='eaten';
      guzzutaSpeed= guzzutaSpeedEaten;
      guzzutaReturnPath=[];
      guzzutaReturnIndex=0;
    }
    else if(guzzutaState==='normal'){
      isGameOver= true;
    }
  }
}

/*--------------------------
  BFS: 巣へ戻る
---------------------------*/
function bfsToHouse(sr,sc, TR,TC){
  return ghostBFSCommon(sr,sc,TR,TC);
}

function ghostBFSCommon(sr,sc,TR,TC){
  let visited= new Set();
  let queue= [];
  queue.push({r:sr,c:sc, path:[{row:sr,col:sc}]});
  visited.add(sr+"-"+sc);

  while(queue.length>0){
    let {r,c,path}= queue.shift();
    if(r===TR && c===TC){
      return path;
    }
    let dirs= [
      {x:0,y:-1},{x:0,y:1},{x:-1,y:0},{x:1,y:0}
    ];
    for(let d of dirs){
      let nr= r+d.y, nc= c+d.x;
      if(!isWallForEatenGhost(nr,nc)){
        let key= nr+"-"+nc;
        if(!visited.has(key)){
          visited.add(key);
          queue.push({
            r:nr,c:nc,
            path:[...path,{row:nr,col:nc}]
          });
        }
      }
    }
  }
  return null;
}

function isWallForEatenGhost(r,c){
  if(r<0 || r>=rows) return true;
  if(c<0 || c>=cols) return true;
  let t= maze[r][c];
  if(t>=1 && t<=7){
    // 7はゲート => 目玉なら通れる
    return (t!==7);
  }
  return false;
}

/*--------------------------
  ゲームフロー
---------------------------*/
function handleGameFlow(){
  if(gameFlowMode==='frightened'){
    modeTimer+= 1/60;
    if(modeTimer>= frightenedDuration){
      gameFlowMode= pausedModeBeforeFrightened;
      modeTimer= pausedTimerBeforeFrightened;

      // アカベエ
      akabeeIsFrightened=false;
      if(akabeeState==='normal'){
        akabeeSpeed= akabeeSpeedNormal;
      }
      // ピンキー
      pinkyIsFrightened=false;
      if(pinkyState==='normal'){
        pinkySpeed= pinkySpeedNormal;
      }
      // アオスケ
      aosukeIsFrightened=false;
      if(aosukeState==='normal'){
        aosukeSpeed= aosukeSpeedNormal;
      }
      // グズタ
      guzzutaIsFrightened=false;
      if(guzzutaState==='normal'){
        guzzutaSpeed= guzzutaSpeedNormal;
      }

      // パックマン速度を戻す
      pacmanVelX= sign(pacmanVelX)* pacmanSpeed;
      pacmanVelY= sign(pacmanVelY)* pacmanSpeed;

      pausedModeBeforeFrightened=null;
      pausedTimerBeforeFrightened=0;
    }
    return;
  }

  modeTimer+=1/60;
  if(gameFlowMode==='scatter'){
    if(modeTimer>= scatterDuration){
      setGameFlowMode('chase');
    }
  }
  else if(gameFlowMode==='chase'){
    if(modeTimer>= chaseDuration){
      setGameFlowMode('scatter');
    }
  }
}

function setGameFlowMode(newMode){
  gameFlowMode= newMode;
  modeTimer=0;

  // normalゴーストなら方向反転
  if(!akabeeIsFrightened && akabeeState==='normal'){
    akabeeVelX= -akabeeVelX; 
    akabeeVelY= -akabeeVelY;
  }
  if(!pinkyIsFrightened && pinkyState==='normal'){
    pinkyVelX= -pinkyVelX; 
    pinkyVelY= -pinkyVelY;
  }
  if(!aosukeIsFrightened && aosukeState==='normal'){
    aosukeVelX= -aosukeVelX; 
    aosukeVelY= -aosukeVelY;
  }
  if(!guzzutaIsFrightened && guzzutaState==='normal'){
    guzzutaVelX= -guzzutaVelX; 
    guzzutaVelY= -guzzutaVelY;
  }
}

/*--------------------------
  情報表示
---------------------------*/
function displayInfo(){
  push();
  fill(255);
  noStroke();
  textAlign(LEFT,TOP);
  textSize(16);

  let info= "Mode: "+gameFlowMode+"  "+modeTimer.toFixed(1)+"s\n"
          + "dotCounter: "+ dotCounter;
  text(info, 12*cellSize, 1*cellSize);
  pop();
}

/*--------------------------
  ターゲット描画
---------------------------*/
// アカベエのターゲット表示
function drawAkabeeTarget(){
  if(akabeeState!=='normal') return;
  if(akabeeIsFrightened) return;

  let effectiveMode= gameFlowMode;
  if(gameFlowMode==='frightened'){
    if(pausedModeBeforeFrightened==='scatter' || pausedModeBeforeFrightened==='chase'){
      effectiveMode= pausedModeBeforeFrightened;
    } else {
      return;
    }
  }

  let targetCell=null;
  if(effectiveMode==='scatter'){
    targetCell= {row:AKABEE_SCATTER_ROW, col:AKABEE_SCATTER_COL};
  }
  else if(effectiveMode==='chase'){
    const pm= getGridPosition(pacmanPixelX,pacmanPixelY);
    targetCell= {row:pm.row, col:pm.col};
  }
  if(!targetCell) return;

  const tx= targetCell.col*cellSize+ cellSize/2;
  const ty= targetCell.row*cellSize+ cellSize/2;
  push();
  fill(255,0,0);
  noStroke();
  rectMode(CENTER);
  rect(tx,ty,5,5);
  stroke(255,0,0);
  line(round(akabeePixelX),round(akabeePixelY), tx,ty);
  pop();
}

// ピンキーのターゲット表示
function drawPinkyTarget(){
  if(pinkyState!=='normal') return;
  if(pinkyIsFrightened) return;

  let effectiveMode= gameFlowMode;
  if(gameFlowMode==='frightened'){
    if(pausedModeBeforeFrightened==='scatter' || pausedModeBeforeFrightened==='chase'){
      effectiveMode= pausedModeBeforeFrightened;
    } else {
      return;
    }
  }

  let targetCell=null;
  if(effectiveMode==='scatter'){
    targetCell= PINKY_SCATTER_TARGET;
  }
  else if(effectiveMode==='chase'){
    // パックマン進行方向4マス先
    const pm= getGridPosition(pacmanPixelX,pacmanPixelY);
    let pdir= pacmanDirection;
    let ahead= {row:pm.row,col:pm.col};
    switch(pdir){
      case 'left':  ahead.col-=4; break;
      case 'right': ahead.col+=4; break;
      case 'up':    ahead.row-=4; break;
      case 'down':  ahead.row+=4; break;
    }
    targetCell= ahead;
  }
  if(!targetCell) return;

  const tx= targetCell.col*cellSize+ cellSize/2;
  const ty= targetCell.row*cellSize+ cellSize/2;

  push();
  fill(255,182,193);
  noStroke();
  rectMode(CENTER);
  rect(tx,ty,5,5);
  stroke(255,182,193);
  line(round(pinkyPixelX),round(pinkyPixelY), tx,ty);
  pop();
}

// アオスケのターゲット表示
function drawAosukeTarget(){
  if(aosukeState!=='normal') return;
  if(aosukeIsFrightened) return;

  let effectiveMode= gameFlowMode;
  if(gameFlowMode==='frightened'){
    if(pausedModeBeforeFrightened==='scatter' || pausedModeBeforeFrightened==='chase'){
      effectiveMode= pausedModeBeforeFrightened;
    } else {
      return;
    }
  }

  let targetCell=null;
  if(effectiveMode==='scatter'){
    targetCell= AOSUKE_SCATTER_TARGET;
  }
  else if(effectiveMode==='chase'){
    // パックマンの2マス先 M
    const pm= getGridPosition(pacmanPixelX,pacmanPixelY);
    let pdir= pacmanDirection;
    let ahead= {row:pm.row,col:pm.col};
    switch(pdir){
      case 'left': ahead.col-=2; break;
      case 'right':ahead.col+=2; break;
      case 'up':   ahead.row-=2; break;
      case 'down': ahead.row+=2; break;
    }
    // アカベエ(A)
    const ak= getGridPosition(akabeePixelX,akabeePixelY);
    const rowT= 2*ahead.row - ak.row;
    const colT= 2*ahead.col - ak.col;
    targetCell= {row:rowT,col:colT};
  }
  if(!targetCell) return;

  const tx= targetCell.col*cellSize+ cellSize/2;
  const ty= targetCell.row*cellSize+ cellSize/2;

  push();
  fill(0,255,255);
  noStroke();
  rectMode(CENTER);
  rect(tx,ty,5,5);
  stroke(0,255,255);
  line(round(aosukePixelX),round(aosukePixelY), tx,ty);
  pop();
}

// グズタのターゲット表示
function drawGuzzutaTarget(){
  if(guzzutaState!=='normal') return;
  if(guzzutaIsFrightened) return;

  let effectiveMode= gameFlowMode;
  if(gameFlowMode==='frightened'){
    if(pausedModeBeforeFrightened==='scatter' || pausedModeBeforeFrightened==='chase'){
      effectiveMode= pausedModeBeforeFrightened;
    } else {
      return;
    }
  }

  let targetCell=null;
  if(effectiveMode==='scatter'){
    targetCell= GUZZUTA_SCATTER_TARGET;
  }
  else if(effectiveMode==='chase'){
    // 160以上離れていればパックマン、未満ならscatter
    const dx= pacmanPixelX- guzzutaPixelX;
    const dy= pacmanPixelY- guzzutaPixelY;
    const dist= sqrt(dx*dx+ dy*dy);
    if(dist>160){
      const pm= getGridPosition(pacmanPixelX,pacmanPixelY);
      targetCell= {row:pm.row,col:pm.col};
    } else {
      targetCell= {row:GUZZUTA_SCATTER_TARGET.row, col:GUZZUTA_SCATTER_TARGET.col};
    }
  }
  if(!targetCell) return;

  const tx= targetCell.col*cellSize + cellSize/2;
  const ty= targetCell.row*cellSize + cellSize/2;

  push();
  fill(255,165,0);
  noStroke();
  rectMode(CENTER);
  rect(tx,ty,5,5);
  stroke(255,165,0);
  line(round(guzzutaPixelX),round(guzzutaPixelY), tx,ty);
  pop();
}

/*--------------------------
  drawCell(壁/ドットなど)
---------------------------*/
function drawCell(x,y,type){
  stroke(255);
  strokeWeight(1);
  noFill();
  const cx= x+cellSize/2;
  const cy= y+cellSize/2;
  switch(type){
    case 1:
      // ┘
      line(cx,cy, cx,y+cellSize);
      line(cx,cy, x+cellSize,cy);
      break;
    case 2:
      // └
      line(cx,cy, cx,y+cellSize);
      line(cx,cy, x,cy);
      break;
    case 3:
      // ┐
      line(cx,cy, cx,y);
      line(cx,cy, x+cellSize,cy);
      break;
    case 4:
      // ┌
      line(cx,cy, cx,y);
      line(cx,cy, x,cy);
      break;
    case 5:
      // ─
      line(x,cy, x+cellSize,cy);
      break;
    case 6:
      // │
      line(cx,y, cx,y+cellSize);
      break;
    case 7:
      // gate(太線)
      strokeWeight(2);
      line(x,cy, x+cellSize,cy);
      strokeWeight(1);
      break;
    case 8:
      // 小ドット
      fill(255);
      noStroke();
      ellipse(cx,cy, cellSize/4);
      break;
    case 9:
      // パワーエサ
      fill(255);
      noStroke();
      ellipse(cx,cy, cellSize*0.8);
      break;
    default:
      // 0 => 通路
      break;
  }
}

/*--------------------------
  補助
---------------------------*/
function getGridPosition(px,py){
  const col= floor(px/cellSize);
  const row= floor(py/cellSize);
  return {row,col};
}

function isCenterOfCell(px,py){
  const ox= px%cellSize;
  const oy= py%cellSize;
  const mid= cellSize/2;
  return (abs(ox-mid)<1 && abs(oy-mid)<1);
}

function canMoveInDirection(dir){
  const {row,col}= getGridPosition(pacmanPixelX,pacmanPixelY);
  let nr=row, nc=col;
  switch(dir){
    case 'left':  nc--; break;
    case 'right': nc++; break;
    case 'up':    nr--; break;
    case 'down':  nr++; break;
  }
  return !isWall(nr,nc);
}

function isWall(r,c){
  if(r<0||r>=rows) return true;
  if(c<0||c>=cols){
    // row=17はワープ通路 => 壁扱いしない
    if(r===17) return false;
    return true;
  }
  const t= maze[r][c];
  return (t>=1 && t<=7);
}

function checkAllDotsCleared(){
  for(let r=0; r<rows; r++){
    for(let c=0; c<cols; c++){
      if(maze[r][c]===8 || maze[r][c]===9){
        return false;
      }
    }
  }
  return true;
}

function sign(x){
  if(x>0) return 1;
  if(x<0) return -1;
  return 0;
}
```
