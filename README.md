<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport"
      content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">

<title>神妖RPG</title>

<style>
* {
    box-sizing: border-box;
    -webkit-tap-highlight-color: transparent;
}

body {
    margin: 0;
    background: #111;
    color: white;
    font-family: sans-serif;
}

#game {
    width: 100%;
    max-width: 700px;
    min-height: 100vh;
    margin: auto;
    background: #181818;
    display: flex;
    flex-direction: column;
}

/* タイトル */
#title {
    text-align: center;
    padding: 18px 10px;
    font-size: 26px;
    font-weight: bold;
    border-bottom: 2px solid #444;
}

/* バトル画面 */
#battle {
    padding: 15px;
    flex: 1;
}

/* キャラクター情報 */
.status {
    border: 2px solid #555;
    border-radius: 10px;
    padding: 12px;
    margin-bottom: 12px;
    background: #222;
}

.name {
    font-size: 20px;
    font-weight: bold;
}

.hp-text {
    margin-top: 6px;
}

.bar {
    width: 100%;
    height: 18px;
    background: #444;
    border-radius: 10px;
    overflow: hidden;
    margin-top: 5px;
}

.hp {
    height: 100%;
    background: #e53935;
    width: 100%;
    transition: width 0.3s;
}

.mp {
    height: 100%;
    background: #1976d2;
    width: 100%;
    transition: width 0.3s;
}

/* 敵 */
.enemy-area {
    text-align: center;
    padding: 15px 0;
}

.enemy-icon {
    font-size: 90px;
    margin: 15px;
}

/* ログ */
#log {
    height: 140px;
    overflow-y: auto;
    background: #0b0b0b;
    border: 2px solid #444;
    border-radius: 10px;
    padding: 10px;
    margin-bottom: 12px;
    line-height: 1.6;
}

/* コマンド */
.commands {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 8px;
}

button {
    min-height: 58px;
    font-size: 18px;
    font-weight: bold;
    color: white;
    background: #333;
    border: 2px solid #666;
    border-radius: 10px;
}

button:active {
    transform: scale(0.97);
    background: #555;
}

button:disabled {
    opacity: 0.35;
}

/* 技選択 */
#skillMenu {
    display: none;
    position: fixed;
    inset: 0;
    background: rgba(0,0,0,0.85);
    justify-content: center;
    align-items: center;
    padding: 20px;
}

.skill-box {
    width: 100%;
    max-width: 500px;
    background: #222;
    border: 2px solid #777;
    border-radius: 15px;
    padding: 15px;
}

.skill-box h2 {
    text-align: center;
}

.skill {
    width: 100%;
    margin-bottom: 8px;
}

.back {
    background: #555;
}

/* 結果画面 */
#result {
    display: none;
    position: fixed;
    inset: 0;
    background: rgba(0,0,0,0.92);
    justify-content: center;
    align-items: center;
    text-align: center;
}

.result-box {
    padding: 30px;
}

.result-box h1 {
    font-size: 36px;
}

.result-box button {
    width: 220px;
    margin-top: 20px;
}
</style>
</head>

<body>

<div id="game">

    <div id="title">
        ⚔ 神妖RPG ⚔
    </div>

    <div id="battle">

        <!-- プレイヤー -->
        <div class="status">
            <div class="name">
                佐藤 光　Lv.<span id="level">1</span>
            </div>

            <div class="hp-text">
                HP <span id="playerHPText">120 / 120</span>
            </div>

            <div class="bar">
                <div id="playerHPBar" class="hp"></div>
            </div>

            <div class="hp-text">
                MP <span id="playerMPText">40 / 40</span>
            </div>

            <div class="bar">
                <div id="playerMPBar" class="mp"></div>
            </div>
        </div>


        <!-- 敵 -->
        <div class="enemy-area">

            <div class="name">
                妖怪・一つ目小僧
            </div>

            <div class="enemy-icon">
                👹
            </div>

            <div>
                HP <span id="enemyHPText">100 / 100</span>
            </div>

            <div class="bar">
                <div id="enemyHPBar" class="hp"></div>
            </div>

        </div>


        <!-- 戦闘ログ -->
        <div id="log">
            妖怪・一つ目小僧が現れた！
        </div>


        <!-- コマンド -->
        <div class="commands">

            <button onclick="attack()">
                ⚔ たたかう
            </button>

            <button onclick="openSkills()">
                ✨ 妖術
            </button>

            <button onclick="guard()">
                🛡 ぼうぎょ
            </button>

            <button onclick="item()">
                🧪 どうぐ
            </button>

            <button onclick="escape()">
                🏃 にげる
            </button>

        </div>

    </div>
</div>


<!-- 技選択 -->
<div id="skillMenu">

    <div class="skill-box">

        <h2>✨ 妖術</h2>

        <button class="skill" onclick="useSkill('lightArrow')">
            光矢　MP 5
        </button>

        <button class="skill" onclick="useSkill('lightFlash')">
            光閃　MP 10
        </button>

        <button class="skill" onclick="useSkill('lightWall')">
            光壁　MP 8
        </button>

        <button class="skill back" onclick="closeSkills()">
            戻る
        </button>

    </div>

</div>


<!-- 結果 -->
<div id="result">

    <div class="result-box">

        <h1 id="resultTitle">
            勝利！
        </h1>

        <p id="resultText"></p>

        <button onclick="restart()">
            もう一度戦う
        </button>

    </div>

</div>


<script>

/* =========================
   プレイヤー
========================= */

let player = {
    level: 1,
    maxHP: 120,
    hp: 120,

    maxMP: 40,
    mp: 40,

    exp: 0,
    nextExp: 50,

    guard: false
};


/* =========================
   敵
========================= */

let enemy = {
    name: "妖怪・一つ目小僧",

    maxHP: 100,
    hp: 100,

    attack: 18
};


/* =========================
   ログ
========================= */

function log(message) {

    const logBox = document.getElementById("log");

    logBox.innerHTML += "<br>" + message;

    logBox.scrollTop = logBox.scrollHeight;
}


/* =========================
   画面更新
========================= */

function updateScreen() {

    document.getElementById("level").textContent =
        player.level;

    document.getElementById("playerHPText").textContent =
        player.hp + " / " + player.maxHP;

    document.getElementById("playerMPText").textContent =
        player.mp + " / " + player.maxMP;

    document.getElementById("enemyHPText").textContent =
        enemy.hp + " / " + enemy.maxHP;


    document.getElementById("playerHPBar").style.width =
        (player.hp / player.maxHP * 100) + "%";

    document.getElementById("playerMPBar").style.width =
        (player.mp / player.maxMP * 100) + "%";

    document.getElementById("enemyHPBar").style.width =
        (enemy.hp / enemy.maxHP * 100) + "%";
}


/* =========================
   通常攻撃
========================= */

function attack() {

    if (enemy.hp <= 0) return;

    let damage =
        Math.floor(Math.random() * 10) + 15;

    enemy.hp -= damage;

    if (enemy.hp < 0) {
        enemy.hp = 0;
    }

    log(
        "佐藤 光の攻撃！ " +
        damage +
        "ダメージ！"
    );

    updateScreen();

    checkEnemy();

}


/* =========================
   妖術メニュー
========================= */

function openSkills() {

    document.getElementById("skillMenu").style.display =
        "flex";
}


function closeSkills() {

    document.getElementById("skillMenu").style.display =
        "none";
}


/* =========================
   妖術
========================= */

function useSkill(skill) {

    closeSkills();

    if (skill === "lightArrow") {

        if (player.mp < 5) {

            log("MPが足りない！");
            return;
        }

        player.mp -= 5;

        let damage =
            Math.floor(Math.random() * 15) + 25;

        enemy.hp -= damage;

        if (enemy.hp < 0) {
            enemy.hp = 0;
        }

        log(
            "✨ 妖術・光矢！ " +
            damage +
            "ダメージ！"
        );
    }


    if (skill === "lightFlash") {

        if (player.mp < 10) {

            log("MPが足りない！");
            return;
        }

        player.mp -= 10;

        let damage =
            Math.floor(Math.random() * 20) + 40;

        enemy.hp -= damage;

        if (enemy.hp < 0) {
            enemy.hp = 0;
        }

        log(
            "✨ 妖術・光閃！！ " +
            damage +
            "ダメージ！"
        );
    }


    if (skill === "lightWall") {

        if (player.mp < 8) {

            log("MPが足りない！");
            return;
        }

        player.mp -= 8;

        player.guard = true;

        log(
            "✨ 妖術・光壁！" +
            " 次の攻撃を軽減する！"
        );
    }


    updateScreen();

    checkEnemy();
}


/* =========================
   ぼうぎょ
========================= */

function guard() {

    player.guard = true;

    log(
        "佐藤 光は身を守っている！"
    );

    enemyTurn();
}


/* =========================
   どうぐ
========================= */

function item() {

    if (player.hp >= player.maxHP) {

        log("HPは満タンだ！");
        return;
    }

    let heal = 40;

    player.hp += heal;

    if (player.hp > player.maxHP) {
        player.hp = player.maxHP;
    }

    log(
        "回復薬を使った！ HPが回復した！"
    );

    updateScreen();

    enemyTurn();
}


/* =========================
   にげる
========================= */

function escape() {

    let success =
        Math.random() < 0.5;

    if (success) {

        log("うまく逃げ切った！");

        setTimeout(() => {

            showResult(
                "逃走成功",
                "戦闘から逃げました。"
            );

        }, 500);

    } else {

        log("逃げられない！");

        enemyTurn();
    }
}


/* =========================
   敵HPチェック
========================= */

function checkEnemy() {

    if (enemy.hp <= 0) {

        log(
            "妖怪・一つ目小僧を倒した！"
        );

        victory();

    } else {

        enemyTurn();
    }
}


/* =========================
   敵の攻撃
========================= */

function enemyTurn() {

    setTimeout(() => {

        if (enemy.hp <= 0) return;

        let damage =
            Math.floor(Math.random() * 8) +
            enemy.attack - 5;

        if (player.guard) {

            damage =
                Math.floor(damage / 2);

            player.guard = false;

            log(
                "🛡 ぼうぎょでダメージを軽減！"
            );
        }

        if (damage < 1) {
            damage = 1;
        }

        player.hp -= damage;

        if (player.hp < 0) {
            player.hp = 0;
        }

        log(
            "一つ目小僧の攻撃！ " +
            damage +
            "ダメージ！"
        );

        updateScreen();

        if (player.hp <= 0) {

            defeat();
        }

    }, 600);
}


/* =========================
   勝利
========================= */

function victory() {

    let exp = 30;

    player.exp += exp;

    log(
        "経験値 " +
        exp +
        " を獲得！"
    );

    if (player.exp >= player.nextExp) {

        levelUp();
    }

    setTimeout(() => {

        showResult(
            "勝利！",
            "妖怪を討伐しました！<br>" +
            "経験値 +" + exp
        );

    }, 700);
}


/* =========================
   レベルアップ
========================= */

function levelUp() {

    player.level++;

    player.exp = 0;

    player.nextExp += 30;

    player.maxHP += 30;

    player.maxMP += 10;

    player.hp = player.maxHP;

    player.mp = player.maxMP;

    log(
        "🎉 レベルアップ！ Lv." +
        player.level +
        "になった！"
    );

    updateScreen();
}


/* =========================
   敗北
========================= */

function defeat() {

    setTimeout(() => {

        showResult(
            "敗北……",
            "佐藤 光は戦闘不能になった。"
        );

    }, 500);
}


/* =========================
   結果画面
========================= */

function showResult(title, text) {

    document.getElementById("resultTitle").innerHTML =
        title;

    document.getElementById("resultText").innerHTML =
        text;

    document.getElementById("result").style.display =
        "flex";
}


/* =========================
   リスタート
========================= */

function restart() {

    player.hp = player.maxHP;
    player.mp = player.maxMP;

    player.guard = false;

    enemy.hp = enemy.maxHP;

    document.getElementById("result").style.display =
        "none";

    document.getElementById("log").innerHTML =
        "妖怪・一つ目小僧が現れた！";

    updateScreen();
}


/* =========================
   初期化
========================= */

updateScreen();

</script>

</body>
</html>
