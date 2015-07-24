// バージョン1.9.2対応

// 開始時
START_HOURS = 4.5; // 時刻が何時か
START_PLAYER_X = 0; // プレイヤーの位置。地形の大きさで補正がかかるためタイル単位ではない
START_PLAYER_Y = 19;
HAS_START_ITEM = false; // 所持品があるか
DELETE_OLD_RECIPE = true; // 既存のレシピを削除するか
START_RAIN_CHANCE = 15; // 雨が降っている確率(%)
IS_START_PLAYER_STATUS_FULL = false; // スタート時のhunger, thirstが最大値
// マップ
MAP_SIZE_RATE = 1.5; // 地形の大きさの倍率
ITEM_SPAWN_CHANCE = 3; // 漂着物の湧きやすさ
// モンスター
NUMBER_CHICKEN = 3;
NUMBER_FISH = 3;
// 表示
LIGHT_AT_NIGHT = 0.8; // 夜の暗さ。1だと真っ暗。0だと真昼の明るさになる
RAIN_RENDER_QUANTITY = 1; // 表示上の雨の多さ
RAIN_RENDER_STOP_SECONDS = 15; // 入力が無くなってから雨の表示を止めるまでの秒数
DECAY_SOON_DISPLAY_HOURS = 3; // 腐る何時間前にアイテムに緑枠を表示するか
// 時間
FRAMES_PER_SECOND = 30; // 1秒間のフレーム数
MOVE_INTERVAL = 100; // 移動してから次に移動できるまでの時間（ミリ秒）
SECONDS_PER_WAIT = 1; // 時間の進む早さ。1ウェイト当たりのゲーム内の秒数。ウェイトとは消費時間の単位
WAITS_PER_STAY = 30; // その場で足踏みした時にかかる時間（ウェイト数）
// 睡眠等でターンをスキップしている時のターン当たりのゲーム内の経過時間（ウェイト数）
WAITS_PER_SKIP_TURN = {
    sleep: 900,
    rest: 100,
    craft: 200,};
// スタミナ
STAMINA_REGENERATION_PER_MINUTE_BY_REST = 7; // 休憩
STAMINA_BY_MOVE = 0.3; // 移動の基本消費量
// その他
CAN_LOG_DISPLAY = false; // ログの表示を行うか
// 画像
IMAGE_ITEMSET_NEW =
    'https://raw.githubusercontent.com/hagoro/el/master/itemset2.png';
IMAGE_ITEMSET_SMALL_NEW =
    'https://raw.githubusercontent.com/hagoro/el/master/itemset2_small.png';
NEW_ITEM_BEGIN_ID = 251; // アイテムIDがこれ以上のアイテムは追加のアイテム
NEW_ITEM_IMAGE_BEGIN_ID = 500; // アイテムIDがこれ以上だと追加のアイテム画像を使用する
// NEW_ITEM_BEGIN_ID以上でNEW_ITEM_IMAGE_BEGIN_ID未満のアイテムは既存の画像を使用した追加アイテム
// リリース時にアイテム画像を更新すること
IS_RELEASE_VERSION = true; // リリース版ではこれをtrueにする。falseにすると開発環境でないと動作しない

// 以下の定数は開発版でしか機能しない
if (!IS_RELEASE_VERSION) {
    FRAMES_PER_SECOND = 8;
//    MOVE_INTERVAL = 100;
    // 画像ファイル名を変えないとfirefoxで画像が更新されない
    HAS_START_ITEM = true;
    CAN_LOG_DISPLAY = false;
    IMAGE_ITEMSET = 'images/itemset.png';
    IMAGE_ITEMSET_SMALL = 'images/itemset_small.png';
    IMAGE_ITEMSET_NEW = 'images/itemset2.png';
    IMAGE_ITEMSET_SMALL_NEW = 'images/itemset2_small.png';
    WAITS_PER_STAY = 30;
    IS_START_PLAYER_STATUS_FULL = false;
    var log = {}; // 関数内の変数をコンソールで見るために、これに渡す
} else {
    IMAGE_ITEMSET = itemset.src;
    IMAGE_ITEMSET_SMALL = itemSetSmall.src;
}

/**
 * modの初期化処理を行う
 * このmodがロードされた時に実行する
 */
function initMod() {
    Messages.none = '_0_'; // showMessage用の初期化
    /* 機能しない
    if (IS_RELEASE_VERSION && isLoaded) {
        showMessage('This mod has been already loaded.');
        return;
    }
    isLoaded = true;
    */
    showMessage('Enjoy life!');

//
/// 内部処理を書き換え
//
    // すでに実行されていたらスキップして、二重に書き換えるのを防ぐ
    var canSkip = false;
    if (window['rendering'].toString().
        indexOf('// EnjoyLife Mod Loaded') != -1) {
        canSkip = true;
    }
    if (!canSkip) {
        changeFunc('rendering', "\n", "// EnjoyLife Mod Loaded\n");
// 以下インデントを左端に戻します。一行を長くして改行を減らすため
//
///     モンスターの処理
//
///         モンスターを初期化
changeFunc('spawnMonster', "tile[x][y].monster = game.monsters.length - 1;", " \
    tile[x][y].monster = game.monsters.length - 1; \
    var mons = game.monsters[game.monsters.length - 1]; \
    mons.act = {status: 'thinkDest', dest: {}}; \
    mons.last = {};");
// ウェイトを初期化
changeFunc('spawnMonster', 'y: y,', 'y:  y, wait: 0,');
changeFunc('spawnMonster', 'y: y,', 'y:  y, wait: 0,');

///         モンスターの行動
// 行動を遅く  *罠関係がおかしくなるかも  すり抜けたり

///         monsMoveループ前
// にループを足す
changeFunc('monsterMove', 'for (var monster = 0;', " \
    game.updateMovement = true; \
    // 初期化 \n\
    for (var i = 0; i < game.monsters.length; i++) { \
        if (!game.monsters[i]) continue; \
        game.monsters[i].route = null; \
        game.monsters[i].draw = null; \
    } \
    // 現在の時刻を経過させる \n\
    game.time.now.seconds += game.time.passInThisTurn = \
        player.wait * SECONDS_PER_WAIT; \
    // プレイヤーのウェイトが残っている間モンスターを行動させる \n\
    for (; player.wait > 0; player.wait--) \
        for (var monster = 0; \
");
///         ループ内
changeFunc('monsterMove', "if (game.monsters[monster]) {"," \
    if (game.monsters[monster]) { \
        var mons = game.monsters[monster]; \
        var lastX = mons.x, lastY = mons.y; \
//        if (mons.wait == -1) mons.wait = 1; \n\
        if (--mons.wait > 0) continue; \
        switch (mons.type) { \
            case 'chicken': \
                mons.wait += 32; \
                break; \
            case 'zombie': \
                mons.wait += 40; \
                break; \
            default: \
                mons.wait += 32; \
        } \
        // 移動速度を固定させない \n\
        if (random(8) == 0) \
            mons.wait += random(Math.floor(mons.wait * 0.2)); \
");
// 攻撃しないモンスターを設定
changeFunc('monsterMove', "skipCombat = (", " \
    skipCombat  = mons.type == 'chicken' || (");

/*
changeFunc('monsterMove', 'if (game.monsters[monster].ai !== "stationary")', ' \
    moveMonster(monster); \
    if (false)');
*/
changeFunc('monsterMove', "//Remove monsters", "\
    var retObj = moveMonster(monster); \
    x = retObj.x; \
    y = retObj.y; \
    // Remove monsters");

changeFunc('monsterMove', "if (game.monsters[monster].hp >=", " \
    if (false && game.monsters[monster].hp >=");

///         ループ内の最後
// 記述が同じ部分を二回スキップする
changeFunc('monsterMove', "deleteMonsters(monster);"," \
deleteMonsters(monster // skipped \n\
    ); ");
changeFunc('monsterMove', "deleteMonsters(monster);"," \
deleteMonsters(monster // skipped \n\
    ); ");
changeFunc('monsterMove', "deleteMonsters(monster);"," \
    deleteMonsters(monster); }}} \
    if (mons.x != lastX || mons.y != lastY) { \
        if (!mons.route) { \
            mons.route = []; /* 自分の一行動内にモンスターが移動したルート */ \
        } \
        if (mons.x >= player.x - game.windowHalfWidth - 1 && \
            mons.x <= player.x + game.windowHalfWidth + 1 && \
            mons.y >= player.y - game.windowHalfHeight - 1 && \
            mons.y <= player.y + game.windowHalfHeight + 1) { \
            mons.route.push({x: mons.x, y: mons.y}); \
            /* 向きを変えたあと動くと最初の一瞬位置がおかしくなるバグに対処 */ \
            if (!mons.draw) mons.draw = {x: mons.x, y: mons.y}; \
        } else { \
            mons.route.push(null); \
        } \
    } }} \
\
    for (var i = 0; i < game.monsters.length; i++) { \
        var monster = game.monsters[i]; \
        if (!monster || !monster.route) \
            continue; \
        // []内はそのモンスターが動いた回数 \n\
        /* timeRateをセット */ \
        for (var j = 0; j < monster.route.length; j++) { \
            if (!monster.route[j]) continue; \
            monster.route[j].timeRate = j / monster.route.length; \
        } \
        /* 空の要素を削除 */ \
        for (var j = 0; j < monster.route.length; j++) \
            if (!monster.route[j]) { \
                monster.route.splice(j, 1); \
                j--; \
            } \
        if (monster.route.length == 0) monster.route = null; \
    } \
    {{{{{ // ←空の閉じ括弧を作り閉じ括弧の位置を移動させる \n\
");
///         モンスターの発生
changeFunc('passTurn', 'game.monsterSpawnTimer++;','');

///     アイテムの処理
///         アイテムの製作
changeFunc('craftItem', "if (player.skills[items[craftType].recipe.skill].percent >= (skillChance + chance - (75 + qualityBonus))) {", " \
\
    switch (items[craftType].recipe.level) { \
        case 'simple': \
            chance = 90; break; \
        case 'intermediate': \
            chance = 30; break; \
    } \
    if (random(100) < chance) {");
changeEventFunc(ui.$craft, $('#craft'), ['mouseup', '.craft'],
    "if (craftItem(craftClass)) {", " \
    // ターンをスキップする \n\
    // ターン数をセット \n\
    if (items[craftClass].recipe.craftTime) \
        player.skipTurns.num = \
            Math.round(items[craftClass].recipe.craftTime.getTotalSeconds() \
            * random(7, 13) / 10 / \
            SECONDS_PER_WAIT / WAITS_PER_SKIP_TURN.craft); \
    else player.skipTurns.num = Math.round(random(20, 40) * 60 / \
        SECONDS_PER_WAIT / WAITS_PER_SKIP_TURN.craft); \
    player.skipTurns.type = 'craft'; \
    player.skipTurns.craftItem = $(this).attr('data-item'); \
    $('#loading').show(); \
    if (false) {");
// 死亡時にスキップを終了する
changeFunc('death', "player.died = true;", " \
    player.skipTurns.num = 0; \
    $('#loading').hide(); \
    player.died  = true;")
// レシピを発見しない
changeFunc('craftItem', 'discoverItem("skill", items[craftType].recipe.skill);',
    'if (false) discoverItem ("skill", items[craftType].recipe.skill);');
/*
changeFunc('Actions', 'if (type === "sleep") {', ' \
    if (type == "craft") { \
        cycles = 100; \
    } \
    if (type === "sleep") {')
for (var i = 0; i < 3; i++)
    changeFunc('restCycle', "ui.message(", " \
        if (game.loadingData.type != 'craft') ui.message (");
*/
///         アイテムが腐る
// 時刻が経過した分だけ腐る
changeFunc('passTurn', "player.invItems[invItem].decay--;", " \
    var item = player.invItems[invItem]; \
    item.decay -= game.time.passInThisTurn; \
    if (item.decay < 0) item.decay = 0;");
changeFunc('passTurn', "tileItems[tileItemsKey[i2]].decay--;", " \
    var item = tileItems[tileItemsKey[i2]]; \
    item.decay -= game.time.passInThisTurn; \
    if (item.decay < 0) item.decay = 0;");
///         アイテム情報表示
changeFunc(['ui', 'itemTip'], 'Messages.decay + " " + itemList[invId].decay', ' \
    Messages.decay + " " + \
    Math.floor(itemList[invId].decay / 3600) + " hour" + \
    (Math.floor(itemList[invId].decay / 3600) != 1 ? "s" : "")');
// メッセージを非表示
changeFunc('passTurn', 'ui.message("itemDecay",', ' \
    // ui.message("itemDecay",');
changeFunc('itemGet', "ui.message(msgType,", " \
    if (msgType != 'itemPickup') \
        ui.message(msgType,");

///     自分の処理
///         ステータス
// ステータス表示
changeFunc('stats', '[player.health + "/" + player.strength + " (" + healthStatPercent + "%)"]',
    '[healthStatPercent + "% " + ( \
        // 時刻を表示 \n\
        IS_RELEASE_VERSION ? "" : \
            // ヘルスゲージに時間を表示 \n\
            " (" + (game.time.now.getHours() < 10 ? "0" : "") + \
            game.time.now.getHours() + ":" + \
            (game.time.now.getMinutes() < 10? "0" : "") + \
            game.time.now.getMinutes() + ")" \
    )]');
changeFunc('stats', '[player.stamina + "/" + player.dexterity + " (" + staminaStatPercent + "%)"]', '[staminaStatPercent + "%"]');
changeFunc('stats',
    '[player.hunger + "/" + player.starvation + " (" + hungerStatPercent + "%)"]',
    '[hungerStatPercent + "%"]');
changeFunc('stats',
    '[player.thirst + "/" + player.dehydration + " (" + thirstStatPercent + "%)"]',
    '[thirstStatPercent + "%"]');
// 空腹・渇きがマイナスにならないようにする
changeFunc('passTurn', 'player.hunger -= 1;', ' \
    player.hunger  -= 1; \
    if (player.hunger < 0) player.hunger = 0; \
');
changeFunc('passTurn', 'player.thirst -= 1;', ' \
    player.thirst  -= 1; \
    if (player.thirst < 0) player.thirst = 0; \
');
changeFunc('edible', "if (thirst !== 0) {", " \
    if (player.thirst + thirst < 0) thirst = 0; \
    if (thirst !== 0) {");

///     render内の書き換え
changeFunc('render', "if (ui.keyTimer >= game.delay && game.scroll.x === 0 && game.scroll.y === 0) {"," \
    // 休憩時等にターンを進める \n\
    if (player.skipTurns.type) \
        passTurn(true); \
    // 生物の移動中なら移動する生物がいるか調べて表示座標をセットする \n\
    var nowTime = new Date().getTime(); \
    var nowTimeRate = (nowTime - player.lastActTime) / game.actTime; \
    for (var i = 0; i < game.monsters.length; i++) { \
        var monster = game.monsters[i]; \
        if (!monster || !monster.route) continue; \
        // 表示座標をセットする  複数該当する場合はスキップする \n\
        while (monster.route.length > 0) { \
            if (monster.route[0].timeRate > nowTimeRate) { \
                for (var v = 0; v < monster.route.length; v++) { \
                } \
                break; \
            } \
            monster.draw = {x: monster.route[0].x, y: monster.route[0].y}; \
            monster.route.splice(0, 1); \
            game.updateMovement = true; \
            game.updateTiles = true; // これは負荷が大きいかも \n\
        } \
    } \
    if (nowTime > player.nextActTime && \
        // player.wait <= 0 && ui.keyTimer >= game.delay \n\
        game.scroll.x === 0 && game.scroll.y === 0) { \
");
changeFunc('render', "if (pickups.length >= 1 ||", " \
    // ピックアップアイテムの表示が更新される間は雨を止めない \n\
    var updatesPickups = false; \
    for (var i = 0; i < pickups.length; i++) \
        if (pickups[i] && pickups[i].type != 'rain' && \
            pickups[i].opacity >= 0) \
            updatesPickups = true; \
    // ウィンドウ切替時等に雨を降らせ過ぎない \n\
    if (game.weather.type == 'rain' && \
        (nowTime - player.lastActTime) > \
        RAIN_RENDER_STOP_SECONDS * 1000) \
        game.weather.lastDrawTime = nowTime; \
    // 雨を降らせる \n\
    if (game.weather.type == 'rain' && \
        // 雨の表示を一定時間に限定。雨が降ると何も行動しなくてもレンダリングが必要になりCPUの負担になるため \n\
        (updatesPickups || \
        (nowTime - player.lastActTime) < \
        RAIN_RENDER_STOP_SECONDS * 1000)) { \
        var passTime = nowTime - game.weather.lastDrawTime; \
        // ウィンドウ切替時などに雨を降らせ過ぎないように \n\
        if (passTime > 200) passTime = 200; \
        game.weather.lastDrawTime = nowTime; \
        // 雨粒を生成 \n\
        createParticles(50, 50, 255, 'rain', 0, 0, \
            Math.floor(passTime / 1000 / 100000 * RAIN_RENDER_QUANTITY * \
                titleCanvas.canvas.width * titleCanvas.canvas.height)); \
    } \
\
    if (game.weather.type != 'rain' || updatesPickups || \
        (nowTime - player.lastActTime) < RAIN_RENDER_STOP_SECONDS * 1000) \
        if (pickups.length >= 1 ||");

changeFunc('render', "if (tile[worldX][worldY].monster) {", " \
    var monsterId = -1; \
    for (var _i = 0; _i < game.monsters.length; _i++) { \
        var _monster = game.monsters[_i]; \
        if (!_monster) continue; \
        if (_monster.draw) { \
            if (_monster.draw.x == worldX && _monster.draw.y == worldY) { \
                monsterId = _i; \
            } \
        } else { \
            if (_monster.x == worldX && _monster.y == worldY) { \
                monsterId = _i; \
            } \
        } \
    } \
    if (monsterId != -1) { \
");
changeFunc('render', "var monster = tile[worldX][worldY].monster;", " \
    var monster = monsterId;");
// 追加アイテム画像を表示
changeFunc('render', "overlayCanvas.drawImage(itemset,", " \
    overlayCanvas.drawImage( \
        items[pickups[pickup].type].id >= NEW_ITEM_IMAGE_BEGIN_ID ? \
        game.image.itemsetNew : itemset, ");
changeFunc('render', "gameCanvas.drawImage(itemSetSmall,", " \
    gameCanvas.drawImage( \
        items[tileItems[tileItemsKey[i]].type].id >= NEW_ITEM_IMAGE_BEGIN_ID ? \
        game.image.itemSetSmallNew : itemSetSmall, ");
// パーティクルを開放。降雨時は絶え間なくパーティクルが生成されて開放されないため
changeFunc('render', "particles[particle].size);", " \
    particles[particle].size); }} \
    if (particles[particle].type == 'rain' && particles[particle].times >= 14) \
        delete particles[particle]; {{ ");
changeFunc('render', "particle++) {", " \
    particle++) { \
    if (!particles[particle]) continue; ");
///         移動
// 移動と向きを変える
changeFunc('render', "if (player.direction.x !== 1) {", " \
    player.direction.x = 1; \
    player.direction.y = 0; \
    player.lastActTime = new Date().getTime(); \
    player.nextActTime = player.lastActTime + game.actTime; \
    if (false && player.direction.x !== 1) {");
changeFunc('render', "if (player.direction.x !== -1) {", " \
    player.direction.x = -1; \
    player.direction.y = 0; \
    player.lastActTime = new Date().getTime(); \
    player.nextActTime = player.lastActTime + game.actTime; \
    if (false && player.direction.x !== -1) {");
changeFunc('render', "if (player.direction.y !== -1) {", " \
    player.direction.x = 0; \
    player.direction.y = -1; \
    player.lastActTime = new Date().getTime(); \
    player.nextActTime = player.lastActTime + game.actTime; \
    if (false && player.direction.y !== -1) {");
changeFunc('render', "if (player.direction.y !== 1) {", " \
    player.direction.x = 0; \
    player.direction.y = 1; \
    player.lastActTime = new Date().getTime(); \
    player.nextActTime = player.lastActTime + game.actTime; \
    if (false && player.direction.y !== 1) {");
// プレイヤーの移動・攻撃時にウェイトをかける
changeFunc('move', "if (!monsterCombat) {", " \
    /* 空腹と渇きによって移動速度を変える */ \
    var hungerSpeedRate = player.hunger / player.starvation * 2; \
    if (hungerSpeedRate > 1) hungerSpeedRate = 1; \
    var thirstSpeedRate = player.thirst / player.dehydration * 2; \
    if (thirstSpeedRate > 1) thirstSpeedRate = 1; \
    var staminaSpeedRate = player.stamina <= 0 ? 0.5 : 1; \
    var calc = new Calculator(); \
    calc.rate = 1 / (0.2 + hungerSpeedRate * 0.5 + thirstSpeedRate * 0.3) / \
        staminaSpeedRate; \
\
    // 攻撃時のウェイト \n\
    if (monsterCombat) player.wait += calc.get(15); \
    // 移動時のウェイト \n\
    if (!monsterCombat) { \
        switch (tile[player.x][player.y].type) { \
            case 'grass': \
                player.stamina -= 0.5 * STAMINA_BY_MOVE; \
                player.wait += calc.get(10); break; \
            case 'sand': \
                player.stamina -= 2 * STAMINA_BY_MOVE; \
                player.wait += calc.get(30); break; \
            case 'shallowwater': \
                player.stamina -= 6 * STAMINA_BY_MOVE; \
                player.wait += calc.get(50); break; \
            case 'water': \
                player.stamina -= 20 * STAMINA_BY_MOVE; \
                player.wait += calc.get(100); break; \
            case 'deepwater': \
                player.stamina -= 30 * STAMINA_BY_MOVE; \
                player.wait += calc.get(130); break; \
            default: \
                player.stamina -= 0.5 * STAMINA_BY_MOVE; \
                player.wait += calc.get(10); \
        }");
// 移動・向き変え時の表示ウェイトを減らす
for (var i = 0; i < 4; i++) {
    changeFunc('render', 'ui.keyTimer = game.delay - 12;',' \
        ui.keyTimer = game.delay - 1;');
}
changeFunc('move', '} else if (!tiletypes',' \
    game.delay = 0; \
    }  else if (!tiletypes \
');
// スタミナが切れても移動可能
changeFunc('move', "if (player.stamina > 0) {", "if (true) {");
// 足踏み
changeFunc('render',
    "else if (ui.keyState[32] || ui.keyState[190] || ui.mouseState === 5) {", "\
    else  if (ui.keyState[32] || ui.keyState[190] || ui.mouseState === 5) { \
        player.lastActTime = new Date().getTime(); \
        player.nextActTime = player.lastActTime + game.actTime / 8; \
        player.wait += WAITS_PER_STAY; \
        if (tiletypes[tile[player.x][player.y].type].water) { \
            // 海上で足踏みするとスタミナを消費する \n\
            player.stamina -= WAITS_PER_STAY * SECONDS_PER_WAIT / 60 * \
                STAMINA_BY_MOVE * 6; \
        } else { \
            player.stamina += WAITS_PER_STAY * SECONDS_PER_WAIT / 60 * \
                STAMINA_REGENERATION_PER_MINUTE_BY_REST; \
        } ");

///         睡眠
// アクションメニューの表示
changeFunc('toggleActionsMenu', "buttons += '<button class=\"rest\">' + Messages.rest + '</button>';", " \
    buttons += '<button class=\"rest\">' + Messages.rest + '</button>'; \
    if (!(tiletypes[tile[player.x][player.y].type].shallowWater)) \
        buttons += '<button class=\"sleep\">' + 'Sleep' + '</button>'; \
    ");

ui.$actionsMenu = $('#actionsmenu').
    on('mouseup', '.sleep', function () {
        $(this).parent().hide();
        elSleep();}).
    off('mouseup', '.rest').
    on('mouseup', '.rest', function () {
        $(this).parent().hide();
        elRest();});

///     日照量
// monsterSpawnerを使う場合、ここで処理をスキップしているので注意
changeFunc('passTurn', "if (player.light >= 0.95)", " \
    if (false) \
        if (player.light >= 0.95)");
changeFunc('passTurn', "if (player.lightSwitch === 0)", " \
    if (false) \
        if (player.lightSwitch === 0)");

// ゾンビが朝に消える
changeFunc('monsterMove', 'if (player.light < 0.75)', 'if (player.light < 0.55)');
changeFunc('monsterMove', 'if (!(game.monsters[monster].x', ' \
    if (true || !(game.monsters[monster].x');

///     その他
if (IS_RELEASE_VERSION) {
    // 死亡時等に初期化する
    changeFunc('play', "closeContainer();", " \
        closeContainer(); \
        initMod();");
}
// FPSを下げると拾ったアイテムの表示が遅くなるのを防ぐ
changeFunc('render', "if (pickups[pickup]) {", " \
    for (var i = Math.ceil(60 / FRAMES_PER_SECOND); i > 0; i--) \
        if (pickups[pickup]) {");
changeFunc('render', "if (pickups[pickup].opacity >= 0) {", " \
    if (i == 1) \
        if (pickups[pickup].opacity >= 0) {");
// 降雨
changeFunc('createParticles', "opacity: 1", " \
    opacity:  1 \
    }); \
    if (type == 'rain') { \
        var par = particles[particles.length - 1]; \
        par.x = random(0, gameCanvas.canvas.width); \
        par.y = random(0, gameCanvas.canvas.height); \
        par.xoff = 0; \
        par.yoff = 8; \
        par.size = random(4, 6); \
        par.type = 'rain'; \
    } \
    ({");
changeFunc('play', "game.fadeFromBlack = 1;", "game.fadeFromBlack = 0;");
// ターンごとの処理
changeFunc('passTurn', 'if (move) {', 'turnExec(); if (move) {');
// 内部処理の地形の生成をスキップする modロード前はスキップできない
changeFunc('mapLoadSpawn', "var c,", " \
    var startTime = new Date().getTime(); \
    game.startX = game.startY = game.mapSize / 2; \
    player.x = player.y = game.mapSize / 2; \
    for (var mapX = 0; mapX < game.mapSize * 2; mapX++) { \
        tile[mapX] = {}; \
        for (var mapY = 0; mapY < game.mapSize; mapY++) \
            tile[mapX][mapY] = {type: 'deepwater'}; \
    } \
    if (CAN_LOG_DISPLAY) \
        console.log('mapLoad:' + (new Date().getTime() - startTime) + 'ms'); \
    return; \
    var  c,");
/*
changeFunc('mapLoadSpawn', "generateExit(caveX + game.mapSize, caveY);", " \
    generateExit(caveX + game.mapSize, caveY); }} \
    console.log('mapLoad:' + (new Date().getTime() - startTime)); \
    {{");
*/

///     アイテムの枠表示
// 腐敗寸前
changeFunc('itemGet', "ui.$inventory.append('<li data-item=\"' + item.type + '\" data-itemid=\"' + len + '\" class=\"tooltip item ' + item.type + group + quality + damaged + '\"></li>');", " \
    var decay = ''; \
    if (item.decay < DECAY_SOON_DISPLAY_HOURS * 3600) { \
        decay = ' decay'; \
    } \n\
    ui.$inventory.append('<li data-item=\"' + item.type + \
        '\" data-itemid=\"' + len + '\" class=\"tooltip item ' + \
        item.type + group + quality + damaged + decay + '\"></li>');");
/*    ui.$inventory.append('<li data-item=\"' + item.type + \
        '\" data-itemid=\"' + len + '\" class=\"tooltip item ' + \
        (items[item.type].id > NEW_ITEM_IMAGE_BEGIN_ID ? 'newitem ' : '') + \
        item.type + group + quality + damaged + decay + '\"></li>');");
*/
///     イベントドリブン関数を書き換え
changeEventFunc(ui.$inventoryEquip, $('#inventory, .equip, .quickslot'), ['mouseup', '.tooltip'],
    "if (items[invClass].decayable &&", " \
    if (items[invClass].decayable && items[invClass].use &&");

//
/// アイテムを初期化
//
// 既に実行されていたら関数の書き換えと共にスキップされる
// アイテムグループを追加・変更
groups.ropelike.name = 'String-like';
groups.cleaningThings = {name: 'Cleaning-Things'};
groups.bedding = {name: 'Bedding'};
groups.cushioning = {name: 'Cushioning'};

// レシピを削除
var keepRecipes = ['branch', 'woodenpole', 'woodenspear'];

for (var i in items) {
    var item = items[i];
    if (DELETE_OLD_RECIPE)
        if (keepRecipes.indexOf(i) == -1)
            delete item.recipe;
}
///     アイテムを追加
/*items.helmetOfSpelunker = {
    name: 'The Helmet Of Spelunker', id: 501, x: 59, y: 3,
    weight: 1, durability: 100, equip: 'head',
    defense: { base: 1, resist: [['blunt', 1],], vulnerable: []}};
*/
items.skullBowl = {
    name: 'A Skull Bowl', id: 502, x: 0, y: 0,
    weight: 0.5, durability: 20, group: ['container'],
    recipe: {skill: 'woodworking', level: 'simple',
        requires: [['sharpeneditem', 1, 0], ['animalskull', 1, 1]],}};
items.freshWaterInSkullBowl = {
    name: 'Fresh Water in Skull Bowl', id: 503, x: 0, y: 1,
    weight: 1, durability: 20, group: ['water'],
    use: ['drink', 'pour'], onUse: {drink: [0, 0, 0, 8]},
    returnOnUse: 'skullBowl'};
items.broom = {
    name: 'A Broom', id: 504, x: 1, y: 0, decayable: [3],
    weight: 1, durability: 20, group: ['cleaningThings'],
    equip: "held", attack: 1, damageType: ['blunt'],
    recipe: {skill: 'tinkering', level: 'intermediate',
        requires: [['polelike', 1, 1], ['twigs', 2, 2], ['ropelike', 1, 1]],}};
/*items.thinBamboo = {
    name: 'A Thin Bamboo', id: 505, x: 2, y: 0,
    weight: 0.5, durability: 20, group: ['polelike'],};
*/
// 関数名を省略
var type = ItemRegister.setType,
    base = ItemRegister.setBase,
    recipe = ItemRegister.setRecipe;
    weapon = ItemRegister.setWeapon;
type('thinBamboo');
base('A Thin Bamboo', 2, 0, 0.5, null, ['polelike']);
weapon(1, ['blunt']);
type('wovenPillow');
base('A Woven Pillow', 5, 0, 1.0, null, ['bedding']);
recipe('tailoring', 'simple', [['wovenfabric', 1, 1], ['cushioning', 3, 3]]);
type('handheldNet');
base('A Handheld Net', 4, 0, 1.0, null);
recipe('tailoring', 'intermediate',
    [['wovenfabric', 1, 1], ['rope', 1, 1], ['polelike', 1, 1]]);
type('skullPan');
base('A Skull Pan', 3, 0, 1.0, null, ['utensil', 'container']);
weapon(3, ['blunt']);
recipe('tinkering', 'simple',
    [['skullBowl', 1, 1], ['polelike', 1, 1], ['ropelike', 1, 1]]);
type('twigs');
recipe('woodworking', 'simple', [['branch', 1, 1]]);

///     アイテムを変更
type('woodenpole');
weapon(1, ['blunt']);
delete items.branch.equip;
delete items.branch.attack;
delete items.branch.damageType;
items.bone.group[1] = 'polelike';
items.log.decayable = [5];
items.grassblades.recipe = {skill: 'tailoring', level: 'simple',
    requires:[['palmleaf', 1, 1]],};
items.rope.recipe = {skill: 'tailoring', level: 'simple',
    requires:[['ropelike', 2, 2]],};
items.woodenfence.recipe = {skill: 'tinkering', level: 'simple',
    requires:[['branch', 2, 2], ['rope', 1, 1]],};
items.woodenfence.decayable = [7];
items.wovenfabric.recipe = {skill: 'tailoring', level: 'simple',
    requires:[['ropelike', 3, 3]],};
delete items.seaweed.group;
items.seaweed.onUse.eat = [0, 0, 10, -5];
items.rawtrout.onUse.eat = [0, 0, 20, -5];
delete items.seaweed.decayable; // decayが大きすぎるバグに対応
items.plantroots.onUse.eat = [0, 0, 5, 0];
items.woodenchest.name = 'A Wooden Box';
items.woodenchest.container = false;
delete items.woodenchest.use;
items.woodenspear.recipe.craftTime = new Time().setHours(1.5);
items.woodenspear.decayable = [3];
delete items.palmleaf.group;
items.glassbottle.decayable = [0.5, 'sharpglass'];
type('glassbottle');
weapon(4, ['blunt']);
items.sharpglass.decayable = [0.5];
type('sharpglass');
weapon(1, ['slashing']);
items.feather.decayable = [0.5];
items.feather.group = ['cushioning'];
items.leaves.group = ['cushioning', 'compost']
// decayを修正
for (var i in items) {
    var item = items[i];
    if (!item.decayable) {
        item.decayable = [];
        item.decayable[0] = 3 * 3600 * 24;
    } else item.decayable[0] *= 3600 * 24
    if (item.decayable[1] == 'rottingvegetation')
        delete item.decayable[1];
}
//items.glassbottle.decayable[1] = 'sharpglass';

} // if (!canSkip)

/// 内部処理を書き換え（毎回実行）
// 目覚める時に一旦暗くなるのをやめる
//changeFunc(['player', 'actions', 'rest'], "game.fadeFromBlack = 1;", "game.fadeFromBlack = 0;");
// 以下は毎回実行しなければいけない。死んだ時にplayerが初期化されてリセットされるのかよくわからない
///         攻撃
changeFunc(['player', 'actions', 'attack'],
    "if (player.skills[attackSkill].percent >= (hitChance - 50)) {", " \
    hitChance = 80; \
    if (monsterType == 'trout') { \
        hitChance = 5; \
        switch (attackType) { \
            case 'melee': \
                if (weapon.damageType.indexOf('piercing') >= 0) \
                    hitChance = 15; \
                break; \
        } \
    } \
    if (random(100) < hitChance) {");

///     スタミナが切れた場合
changeFunc(['player', 'staminaCheck'], "if (this.stamina <= 0) {", " \
    // スタミナが切れても移動可能 \n\
    if (this.stamina <= 0) { \
        this.stamina = 0; \
        // 溺れ死ぬ \n\
        if (tiletypes[tile[player.x][player.y].type].water) \
            player.health = 0; \
//        ui.message('exhaustion', 'bad'); \n\
//        ui.textAbove('tired', 0, 255, 0); \n\
//        audio.queueSfx('hurt'); \n\
\
    } \
    if (false) {");

//
/// 初期設定
//
// game, playerオブジェクトを拡張
game.interval = Math.round(1000 / FRAMES_PER_SECOND); // 秒間フレーム数
game.spawnItem = { // 漂着物
    tile: [], // 漂着物が湧く場所
    lot: {}, // 漂着するアイテムのの種類を決めるくじ
};
game.base = {}; // 島の中心座標

///     時間を設定
game.time = {now: new Time(START_HOURS * 3600), // ゲーム内の時刻。初日の0時から何秒経過したか
    passInThisTurn: 0, }; // プレイヤーの現在のターンで経過する時間

game.actTime = MOVE_INTERVAL; // 行動にかかる時間  ミリ秒
game.weather = {type: random(100) < START_RAIN_CHANCE ? 'rain' : 'fine',
    lastDrawTime: new Date().getTime()};

game.dailyChallenge = true; // セーブできないようにdailyモードにする
//game.fadeFromBlack = 0;
ui.$mainMenu.find('#in-game').hide();
ui.$mainMenu.find('#new-game').show();
ui.$mainMenu.find('#reset').hide();
ui.$mainMenu.find('#continue-game').hide();

player.crafted = []; // 入手済みレシピをクリア
player.wait = 0; // モンスターを行動させるためのウェイト
player.nextActTime = new Date().getTime(); // 次に行動できる時刻
player.skipTurns = {}; // 製作時等に睡眠時のように連続でターンが進む処理用

// ウィンドウを表示・サイズ変更
ui.$inventoryWindow.dialog('open');
ui.$craftWindow.dialog('open');
if (IS_RELEASE_VERSION)
    ui.$optionsWindow.dialog('close');
else {
    ui.$inventoryWindow.dialog({width: 425, height: 175});
    ui.$craftWindow.dialog({width: 265});
}
ui.options.hints = false; // ヒントを表示しない

///     ステータスを設定
player.strength = 100;
player.dexterity = 100;
player.starvation = 100;
player.dehydration = 100;
player.health = random(8, 18);
player.stamina = player.dexterity;
player.hunger = random(15, 25);
player.thirst = random(5, 25);
if (IS_START_PLAYER_STATUS_FULL) {
    player.hunger = 100;
    player.thirst = 100;
}

///     所持品を設定
// 所持品を削除
itemCnt = player.invItems.length;
for (var i = 0; i < itemCnt; i++) {
    removeItem(i, 'INV', false);
}
// 所持品を追加
// numの数だけtypeの種類のアイテムを入手する。numを省略すると1個
function getItem(type, num) {
    if (num == null) num = 1;
    for (var i = 0; i < num; i++)
        itemGet({type: type, decay: 1000000}, 'silent');
}
if (HAS_START_ITEM) {
    getItem('sharpglass');
    getItem('branch');
    getItem('bone');
    getItem('thinBamboo');
    getItem('broom');
    getItem('grassblades');
    getItem('treevine', 3);
//    getItem('strippedbark');
    getItem('palmleaf');
    getItem('leaves', 3);
    getItem('sapling');
    getItem('plantroots');
    getItem('seaweed');
    getItem('animalskull');
    getItem('glassbottle');
    getItem('log', 3);
    getItem('woodenchest');
    getItem('handheldNet');
    getItem('string');
/*
    getItem('');
    getItem('');
*/
}
///     漂着物を設定
// 湧く漂着物をくじで決める
game.spawnItem.lot.normal = new Lot();
var lot = game.spawnItem.lot.normal;
lot.set('branch', 3);
lot.set('thinBamboo', 1);
lot.set('grassblades', 3);
lot.set('treevine', 3);
lot.set('palmleaf', 1);
lot.set('leaves', 3);
//lot.set('sapling', 3);
lot.set('plantroots', 2);
lot.set('bone', 1);
lot.set('seaweed', 2);
lot.set('animalskull', 2);
lot.set('glassbottle', 2);
game.spawnItem.lot.southCoast = game.spawnItem.lot.normal.copy();
lot = game.spawnItem.lot.southCoast;
lot.set('log', 4);
lot.set('woodenchest', 2);
/*
lot.set('', );
lot.set('', );
*/

//
/// マップを初期化
//
if (IS_RELEASE_VERSION) {
///     マップ全域を海だけにする
for (var mapX = 0; mapX < game.mapSize; mapX++) {
    for (var mapY = 0; mapY < game.mapSize; mapY++) {
        tile[mapX][mapY].type = 'deepwater';
        // アイテム消去
        if (tile[mapX][mapY].tileitems) {
            var keys = Object.keys(tile[mapX][mapY].tileitems);
            var length = keys.length;
            for (var i = 0; i < length; i++)
                removeItem(keys[length - i - 1], 'TILE', false);
        }
        // 消去
        if (tile[mapX][mapY].envItemList) {
            var keys = Object.keys(tile[mapX][mapY].envItemList);
            var length = keys.length;
            for (var i = 0; i < length; i++)
                removeItem(tile[mapX][mapY].envItemList[keys[length - i - 1]], 'ENV', false);
        }
    }
}
}
// モンスターを消去
for (var i = 0; i < game.monsters.length; i++)
    if (game.monsters[i]) deleteMonsters(i);

// 島を作る基準になる座標
game.base.x = Math.floor(game.mapSize / 2);
game.base.y = Math.floor(game.mapSize / 2);
var base = {x: game.base.x, y: game.base.y};

// 地形の大きさを変えられるように
var calc = new Calculator();
calc.rate = MAP_SIZE_RATE; // 大きさの倍率
function mag(n) {
    return calc.get(n);
}
///     地形とモンスターを配置
///         自分
player.x = base.x + mag(START_PLAYER_X);
player.y = base.y + mag(START_PLAYER_Y);
///         島
createMapCircle('island', base.x, base.y, mag(15));
createMapCircle('sand', base.x, base.y + mag(5), mag(10), mag(15));
createMapCircle('island', base.x + mag(9), base.y - mag(9), mag(8));
///         鶏
for (var i = 0; i < NUMBER_CHICKEN; i++) {
    var monsterX = 0;
    var monsterY = 0;
//    monsterX = player.x - 1 + i * 2;
//    monsterY = player.y - 2;
    monsterX = random(base.x - mag(10), base.x + mag(10));
    monsterY = random(base.y - mag(10), base.y + mag(10));
    if (tile[monsterX] && tile[monsterX][monsterY])
        // 海上でなければ配置
        if (!tiletypes[tile[monsterX][monsterY].type].water) {
            spawnMonster("chicken", monsterX, monsterY);
            game.monsters[game.monsters.length - 1].act.base =
                {x: base.x, y: base.y, range: mag(15)};
        }
// if (spawnMonster("chicken", monsterX, monsterY)) ←これでモンスターを配置できたか確認できる
}
///         浅瀬
createMapCircle('shallows', base.x + mag(3), base.y - mag(16), mag(10));
///         魚
for (var i = 0; i < NUMBER_FISH; i++) {
    monsterX = random(base.x + mag(3) - mag(7), base.x + mag(3) + mag(7));
    monsterY = random(base.y - mag(23) - mag(5), base.y - mag(23) + mag(5));
//    monsterY = random(base.y - mag(26) - mag(3), base.y - mag(26) + mag(3));
    if (tile[monsterX] && tile[monsterX][monsterY]) {
//        if (tiletypes[tile[monsterX][monsterY].type].water)
            spawnMonster("trout", monsterX, monsterY);
            game.monsters[game.monsters.length - 1].act.base =
                {x: base.x + mag(3), y: base.y - mag(16),
                range: i == 0 ? mag(22) : mag(10)};
    }
}
///     ミニマップに現在の地形を反映
for (var mapX = 0; mapX < game.mapSize; mapX++) {
    for (var mapY = 0; mapY < game.mapSize; mapY++) {
        if (tiletypes[tile[mapX][mapY].type]) {
            mapCanvas.fillStyle = tiletypes[tile[mapX][mapY].type].color;
            mapCanvas.fillRect(mapX, mapY, 1, 1);
        }
    }
}
///     漂着物が湧く海岸を設定
for (var x = 1; x < game.mapSize - 1; x++)
    for (var y = 1; y < game.mapSize - 1; y++)
        if (tile[x][y].type == 'sand' && (
            tile[x - 1][y].type == 'shallowwater'
            || tile[x + 1][y].type == 'shallowwater'
            || tile[x][y - 1].type == 'shallowwater'
            || tile[x][y + 1].type == 'shallowwater'
        )) {
            game.spawnItem.tile.push({x: x, y: y});
            tile[x][y].type = 'spone'; // 湧く場所を一時的にマークしておく。すぐに元に戻す
        }
// 隣が湧く場合は湧く
for (var x = 1; x < game.mapSize - 1; x++)
    for (var y = 1; y < game.mapSize - 1; y++)
        if (tile[x][y].type == 'sand' && (
            tile[x - 1][y].type == 'spone'
            || tile[x + 1][y].type == 'spone'
            || tile[x][y - 1].type == 'spone'
            || tile[x][y + 1].type == 'spone'
        )) {
            game.spawnItem.tile.push({ x: x, y: y });
        }
for (var i = 0; i < game.spawnItem.tile.length; i++) {
    var itemSpawn = game.spawnItem.tile[i];
    // チェックを戻す
    if (tile[itemSpawn.x][itemSpawn.y].type = 'sand')
    // 場所によって湧くアイテムが変わるのでその種類を設定
//    var itemSpawn = game.spawnItem.tile[game.spawnItem.tile.length - 1];
    if (itemSpawn.y > base.y + mag(14))
        itemSpawn.type = 'southCoast';
    else itemSpawn.type = 'normal';
}

/// cssを追加
    var style = '<style type="text/css">';
    style += " \n\
#loading { \n\
    background: rgba(4, 4, 4, 0.5); \n\
} \n";
///     アイテムの色枠を追加
// 腐る寸前
    style += " \n\
.decay { \n\
    border-color: #007700 !important; \n\
    box-shadow: inset 0 0 10px rgba(0, 200, 0, 0.6); \n\
} \n";
///     アイテム画像を差し替え
    style += '.item { \n \
background: url(' + IMAGE_ITEMSET + '); \n \
background-size: 2048px 128px; \n \
}\n';
/*    style += '.newitem { \n\
    background: url(' + IMAGE_ITEMSET_NEW + '); \n\
    background-size: 2048px 160px; \n}';
*/   style += '</style>';
    $('head').append(style);

    //Dynamically add the item css positions
    // 画像を差し替える場合、cssの変更より後に実行すること
    style = '<style type="text/css">';
    var itemsKey = Object.keys(items);
    for (var i = 0; i < itemsKey.length; i++) {
//        if (items[itemsKey[i]].id < NEW_ITEM_BEGIN_ID) continue;
        style += '.' + itemsKey[i] + ' { ' + (
            items[itemsKey[i]].id >= NEW_ITEM_IMAGE_BEGIN_ID ?
            '\n    background: url(' + IMAGE_ITEMSET_NEW + '); \n' +
            '    background-size: 512px 128px; \n    ' : ''
        ) + 'background-position: -' + (items[itemsKey[i]].y * 64 + items[itemsKey[i]].x) * 32 + 'px -' + items[itemsKey[i]].y * 32 + 'px; }\n';
    }
    style += '</style>';
    $('head').append(style);

// ウィンドウ外のアイテム画像
itemset.src = IMAGE_ITEMSET;
itemSetSmall.src = IMAGE_ITEMSET_SMALL;
game.image = {};
game.image.itemsetNew = new Image();
game.image.itemsetNew.src = IMAGE_ITEMSET_NEW;
game.image.itemSetSmallNew = new Image();
game.image.itemSetSmallNew.src = IMAGE_ITEMSET_SMALL_NEW;

// $('.item').css('background', 'url(http://m2.upup.be/f/r/V5HPphNV29.png)');

///     アイテムを設置
/*
for (var i = 0; i < 3; i++) {
    placeItem({type: "helmetOfSpelunker", x: game.base.x + 9, y: game.base.y - 9, decay: -1, durability: "Random"}, 'TILE', false);
//    placeItem({type: "helmetOfSpelunker", x: player.x, y: player.y - 1, decay: -1, durability: "Random"}, 'TILE', false);
}
*/

elSleep(); // 睡眠状態でスタート
passTurn(); // 画面を更新するため

} // initMod()

/**
 * 睡眠の時に呼ばれる
 */
function elSleep() {
    // ターンをスキップする
    // ターン数をセット
    // 少し眠って、昼なら起きる。夜ならそのまま朝まで眠る
    var wakeTime = game.time.now.getTimeAfter(
        new Time({minutes: random(75, 105)}));
    if (wakeTime.getHoursFloat() > 17.5) {
        wakeTime = new Time({days: wakeTime.getDays() + 1,
            hours: 6, minutes: random(30, 90)});
    } else if (wakeTime.getHoursFloat() < 6.5) {
        wakeTime = new Time({days: wakeTime.getDays(),
            hours: 6, minutes: random(30, 90)});
    }
    player.skipTurns.num = Math.round(
        wakeTime.getTimeFrom(game.time.now).getTotalSeconds() /
        SECONDS_PER_WAIT / WAITS_PER_SKIP_TURN.sleep);
    player.skipTurns.type = 'sleep';
    $('#loading').show();
}
/**
 * 休憩の時に呼ばれる
 */
function elRest() {
    player.skipTurns.type = 'rest';
    $('#loading').show();
}
/**
 * 乱数を返す
 * 引数一つで呼んだ場合 0以上n1 - 1以下の整数
 * 引数二つで呼んだ場合 n1以上n2以下の整数
 * 大きい数だと機能しないかも
 * @param {int} n1
 * @param {int} [n2]
 * @returns {int} 乱数
 */
function random(n1, n2) {
    if (n2 == undefined)
        return Math.floor(Math.random() * n1);
    else
        return n1 + Math.floor(Math.random() * (n2 - n1 + 1));
}

/**
 * イベントドリブン関数を上書きする
 * interface.jsのUiオブジェクト内部の関数等を書き換えるのに使います
 * @param {Object} object - ui.$inventoryEquip等
 * @param {Object} dom
 * @param {Object(Array)} event - イベントの種類。要素が2の配列を使用する
 */
function changeEventFunc(object, dom, event, searchStr, replaceStr) {
    var baseFunc = $._data(object.get(0), 'events')
        [event[0]/*このあたり一部の関数で不具合があるかも*/][0].handler.toString();
    if (baseFunc.indexOf(searchStr) == -1)
        console.log('changeFunc not find: ' + event);
    object = dom.off(event[0], event[1]).on(event[0], event[1], eval('(' +
        baseFunc.replace(searchStr, replaceStr) + ')'))
}

/**
 * アイテムの種類を登録する
 */
function ItemRegister() {
    // クラス変数
    ItemRegister.nextId; // 新しく登録するアイテムのID
    ItemRegister.item; // アイテムの種類
}
    ItemRegister.nextId = 600;
    /**
     * アイテムの種類をセットします。その種類が無い場合は追加します
     * この後他の関数を呼ぶと、ここでセットしたアイテムの種類が使用されます
     * @param {String} type - items.の後に繋がる文字列
     */
    ItemRegister.setType = function(type) {
        if (!items[type]) items[type] = {};
        ItemRegister.item = items[type];
    }
    /**
     * アイテムを追加して基本情報をセットする。IDも自動的にセットする
     * @param [x], [y] - 画像ファイル内の座標
     * @param [weight]
     * @param [durability]
     */
    ItemRegister.setBase = function(name, x, y, weight, durability, group) {
        if (x == null) x = 32;
        if (y == null) y = 3;
        if (weight == null) weight = 0.5;
        if (durability == null) durability = 20;
        var item = ItemRegister.item;
        item.id = ItemRegister.nextId++;
        item.name = name;
        item.x = x;
        item.y = y;
        item.weight = weight;
        item.durability = durability;
        item.group = group;
    }
    /**
     * アイテムのレシピをセットする
     */
    ItemRegister.setRecipe = function(skill, level, requires) {
        if (!ItemRegister.item.recipe)
            ItemRegister.item.recipe = {};
        var recipe = ItemRegister.item.recipe;
        recipe.skill = skill;
        recipe.level = level;
        recipe.requires = requires;
    }
    /**
     * アイテムの武器情報をセットする
     */
    ItemRegister.setWeapon = function(attackPoints, damageType) {
        ItemRegister.item.equip = 'held';
        ItemRegister.item.attack = attackPoints;
        ItemRegister.item.damageType = damageType;
    }

/**
 * ゲーム内の時刻・時間を管理するクラス
 * 現在の時刻やアイテムが腐る時刻などを扱うのに使う
 * @param {Object|int} [properties] - 時刻・時間
 *     hoursは24以上、minutesとsecondsは60以上をそれぞれ指定できる。それぞれを秒換算して加えた時刻になる
 *     例えばminutesを61, secondsを90とすると 1時間2分30秒がセットされる
 */
function Time(properties) {
    if (properties == undefined)
        this.seconds = 0;
    else if (typeof properties == 'number')
        this.seconds = properties;
    else {
        var days = properties.days || 0,
            hours = properties.hours || 0,
            minutes = properties.minutes || 0,
            seconds = properties.seconds || 0;
        this.seconds = Math.round(
            days * 24 * 3600 + hours * 3600 + minutes * 60 + seconds);
    }
    /**
     * 時間数をセットする
     * @param {float} hours - セットする時間数。一時間半なら1.5
     */
    Time.prototype.setHours = function(hours) {
        this.seconds = Math.floor(hours * 3600);
        return this;
    }
    /**
     * その日の0時から経過した秒数を返す
     * @returns {int} 経過した秒数
     */
    Time.prototype.getSecondsOfDay = function() {
        return this.seconds % (3600 * 24);
    }
    /**
     * 全ての時間を秒換算して何秒なのか返す
     * 0日の0時0分0秒から何秒経ったかと同じ意味
     * @returns {int} 秒
     */
    Time.prototype.getTotalSeconds = function() {
        return this.seconds;
    }
    /**
     * 何分なのかを返す
     * @returns {int} 分。例えば01:30なら30
     */
    Time.prototype.getMinutes = function() {
        return Math.floor(this.getSecondsOfDay() / 60) % 60;
    }
    /**
     * その日の0時から経過した時間数を整数で返す
     * @returns {int} その日の0時から経過した時間数
     */
    Time.prototype.getHours = function() {
        return Math.floor(this.getHoursFloat());
    }
    /**
     * その日の0時から経過した時間数を小数で返す
     * @returns {float} その日の0時から経過した時間数。例えば01:30なら1.5時間経過してるので1.5を返す
     */
    Time.prototype.getHoursFloat = function() {
        return this.getSecondsOfDay() / 3600;
    }
    /**
     * 0日から経過した日数を整数で返す
     * @returns {int}
     */
    Time.prototype.getDays = function() {
        return Math.floor(this.seconds / 3600 / 24);
    }
    /**
     * 指定の時刻を基準に、自分の時刻がどれだけ経過したか時間を返す *自分を基準にするように修正したい
     * @param {Time} time - 基準にする時刻
     * @returns {Time} 経過した時間。指定の時間の方が後ならマイナスの値
     */
/*
     * @param {int} hours, [minutes], [seconds] - 時・分・秒を指定する
     * @param {int} [daysLater] - 自分の日付から何日後か。マイナスにすると何日前か指定される
*/
    Time.prototype.getTimeFrom = function(time) {
        return new Time({seconds: this.seconds - time.getTotalSeconds()});
    }
/*
     * 自分の日付を基準に、指定された時刻を返す
     * @param {Object} properties  - [daysAfter]:何日後か。マイナスなら何日前か,
     *     [hours], [minutes], [seconds]: 時刻を指定する
    Time.prototype.getTime = function(properties) {
        var hours, minutes, seconds;
        var days = Math.floor(this.seconds / 24 / 3600);
        if (properties.daysAfter != null)
            days += properties.daysAfter;
        if (properties.hours == null)
            hours = 0;
        else hours = properties.hours;
        if (properties.minutes == null)
            minutes = 0;
        else minutes = properties.minutes;
        if (properties.seconds == null)
            seconds = 0;
        else seconds = properties.seconds = 0;
        return new Time(this.seconds + days * 24 * 3600 + hours * 3600 +
            minutes * 60 + seconds);
    }
*/
    /**
     * 自分の時刻を指定の時間だけ進めた時刻を返す
     * @param {Time} time - 進める時間
     */
    Time.prototype.getTimeAfter = function(time) {
        return new Time({seconds: this.seconds + time.getTotalSeconds()});
    }
    /**
     * 自分のコピーを返す
     * @returns {Object(Time)} 自分のコピー
     */
    Time.prototype.copy = function() {
        return new Time(this.seconds);
    }
}

/**
 * ランダムでくじから景品を引くクラス
 * アイテムをランダムで決める時などに使います
 */
function Lot() {
    /**
     * くじに入れた景品
     */
    this.itemList = [];
    /**
     * 景品をセットする 個数が多いほどその景品が出やすくなる
     * @param {Object} item - くじの景品
     * @param {int} count - 景品を入れる個数
     */
    Lot.prototype.set = function(item, count) {
        for (var i = 0; i < count; i++)
            this.itemList.push(item);
    }
    /**
     * 景品を引く
     * @returns {String|int} 引いた景品
     */
    Lot.prototype.draw = function() {
        return this.itemList[random(this.itemList.length)];
    }
    /**
     * 自分のコピーを返す
     * @returns {Object(Lot)} 自分のコピー
     */
    Lot.prototype.copy = function() {
        var lot = new Lot();
        lot.itemList = this.itemList.concat();
        return lot;
    }
}

/**
 * グローバル関数のプログラムを上書きする
 * wayward内部の関数を書き換えるのに使います
 * searchStrがreplaceStrに含まれる場合コンソールにエラーメッセージを出します
 *
 * \を使って文字列を複数行に渡って書いている途中で//でコメントアウトすると、それ以降の行もコメントアウトされてしまいます
 * プログラムの表示上は改行されていますが、実際の文字列は改行されていないためです
 * その場合は「\n \」と改行コードを入れれば解決します
 *
 * この関数を1回実行するごとに構文チェックが行われるようです
 * 例えば複数行を括弧で囲いたい場合、括弧と閉じ括弧を2回に分けて入れようとすると
 * 1回目の実行で括弧を閉じていないため構文エラーが出ます
 *
 * @param {String|Array} func - 上書きするグローバル関数名
 *     通常はStringだが、Arrayの場合要素数3まで指定できる
 *     例えばActions.rest()の場合、['player','actions','rest']と指定する
 *     Ui.itemTip()の場合、['ui', 'itemTip']と指定する
 * @param {String} searchStr - 置き換え前の文字列
 * @param {String} replaceStr - 置き換える文字列
 */
function changeFunc(func, searchStr, replaceStr) {
    // changeFunc not changedのエラーはいらないか
    if (typeof func == 'object') {
        if (func.length == 2) {
            if (window[func[0]][func[1]].toString().indexOf(searchStr) == -1)
                console.log('changeFunc not find: ' + func[0] + '.' + func[1]);
/*        if (replaceStr.indexOf(searchStr) != -1)
            console.log('changeFunc not changed: ' +
                func[0] + '.' + func[1] + ', ' + searchStr);
*/
            window[func[0]][func[1]] =
                eval('(' + window[func[0]][func[1]].toString().
                    replace(searchStr, replaceStr) + ')');
        } else {
            if (window[func[0]][func[1]][func[2]].toString().
                indexOf(searchStr) == -1)
                console.log('changeFunc not find: ' + func[0] + '.' + func[1] +
                    '.' + func[2]);
            window[func[0]][func[1]][func[2]] =
                eval('(' + window[func[0]][func[1]][func[2]].toString().
                    replace(searchStr, replaceStr) + ')');
        }
    } else {
        if (window[func].toString().indexOf(searchStr) == -1)
            console.log('changeFunc not find: ' + func);
/*        if (replaceStr.indexOf(searchStr) != -1)
            console.log('changeFunc not changed: ' +
                func + ', ' + searchStr);
*/        window[func] = eval('(' + window[func].toString()
            .replace(searchStr, replaceStr) + ')');
    }
}

/**
 * 入力された値に一定の計算をして返すクラス
 */
function Calculator() {
    /**
     * 入力された値に掛ける倍率
     */
    this.rate = 0;
    /**
     * valueに、あらかじめ設定した計算をして返す
     * @param {int|float} value - 計算をする値
     */
    Calculator.prototype.get = function(value) {
        return Math.round(value * this.rate);
    }
}

/**
 * 円形または楕円形の地形を生成する
 * ySizeを指定した場合は楕円形に、指定しない場合は円形になる
 * @param {String} mapType - 生成する地形の種類
 *     'island'は島、'sand'は砂浜、'shallows'は浅瀬
 * @param {int} x, y - 生成する中心の座標
 * @param {int} xSize - 円形の場合は円の半径。楕円形の場合はx軸方向の半径
 * @param {int} [ySize] - 楕円形のy軸方向の半径。円形の場合は指定しない
 */
function createMapCircle(mapType, x, y, xSize, ySize) {
    if (ySize == undefined) ySize = xSize;
    var ySizeRate = ySize / xSize;
    for (var tileX = x - xSize * 2; tileX < x + xSize * 2; tileX++) {
        for (var tileY = y - ySize * 2; tileY < y + ySize * 2; tileY++) {
            // 配列の範囲外へのアクセスを防ぐ
            if (!tile[tileX] || !tile[tileX][tileY]) continue;
            var tileType = tile[tileX][tileY].type;
            var dist = getDistance(tileX, y + (tileY - y) / ySizeRate, x, y);
            var size = xSize;
            // グラフィックパターンに変化をつけるための設定
            tile[tileX][tileY].gfx = random(3);

            switch (mapType) {
                case 'island': // breakしない
                case 'sand':
                    if (mapType == 'island')
                        if (dist < size * 0.65) {
                            tile[tileX][tileY].type = 'grass';
                            continue;
                        }
                    if (tileType == 'grass') continue;
                    if (dist < size) {
                        tile[tileX][tileY].type = 'sand';
                        continue;
                    }
                    if (tileType == 'sand') continue;
                    if (dist < size * 1.25) {
                        tile[tileX][tileY].type = 'shallowwater';
                        continue;
                    }
                    if (tileType == 'shallowwater') continue;
                    if (dist < size * 1.7) {
                        tile[tileX][tileY].type = 'water';
                        continue;
                    }
                    if (tileType == 'water') continue;
                    tile[tileX][tileY].type = 'deepwater';
                    break;
                case 'shallows':
                    if (tileType == 'water')
                        if (dist < size) {
                            tile[tileX][tileY].type = 'shallowwater';
                            continue;
                        }
                    if (tileType == 'deepwater')
                        if (dist < size * 1.5)
                            tile[tileX][tileY].type = 'water';
                    break;
            }
        }
    }
};

/**
 * マップ上の2点間の距離を返す
 * @param {int|float} x1, y1 - 距離を測る始点の座標
 * @param {int|float} x2, y2 - 距離を測る終点の座標
 * @returns {float} 2点間の距離
 */
function getDistance(x1, y1, x2, y2) {
    return Math.sqrt(Math.pow(x1 - x2, 2) + Math.pow(y1 - y2, 2));
};

/**
 * メッセージを表示する
 * @param {String} message - メッセージの文章
 * @param {String} [type="normal"] - メッセージの種類
 */
function showMessage(message, type) {
    if (type == null) type = 'normal';
    ui.message('none', type, [message]);
}

/**
 * 一時停止する
 * @param {int} time - 一時停止する時間（ミリ秒）
 */
function waitTime(time) {
  var d1 = new Date().getTime();
  var d2 = new Date().getTime();
  while (d2 < d1 + time) {
    d2 = new Date().getTime();
   }
   return;
}

/**
 * 例外オブジェクトからスタックトレースを出力する
 */
function printStackTrace(e) {
  if (e.stack) {
    // 出力方法は、使いやすいように修正する。
    console.log(e.stack);
//    alert(e.stack);
  } else {
    // stackがない場合には、そのままエラー情報を出す。
    console.log(e.message, e);
  }
}

/**
 * モンスターを移動させる
 * @param {int} monsterId - 移動させるモンスターのID
 * @returns {Object} x, y - x, y方向の移動量
 */
function moveMonster(monsterId) {
    var monster = game.monsters[monsterId];
    // 移動できない状態なら、移動先を決め直す
    if (monster.act.status == 'move' &&
        monster.x == monster.last.x && monster.y == monster.last.y)
        monster.act.status = 'thinkDest';
    // 移動先を決める
    if (monster.act.status == 'thinkDest') {
        // 移動先候補を決める
        var dest = [];
        for (var i = 0; i < 3; i++) {
            dest[i] = {};
            // 移動先の方向
            var dir = random(360) * Math.PI / 180;
            // 距離
            var dist = 2 + random(10) / 2;
            dest[i].x = monster.x +
                Math.round(Math.cos(dir) * dist);
            dest[i].y = monster.y +
                Math.round(Math.sin(dir) * dist);
            if (monster.act.base)
                dest[i].dist = getDistance(
                    monster.act.base.x, monster.act.base.y,
                    dest[i].x, dest[i].y);
            else dest[i].dist = 0;
        }
        // baseから遠いほどbaseに戻ろうとする確率が上がる
        if (monster.act.base && (
            random(100) < -100 + 200 *
            getDistance(monster.act.base.x, monster.act.base.y,
                monster.x, monster.y) /
            monster.act.base.range)
        ) {
           // baseからの距離が短い順にdest[]を並び替える
            dest.sort(
                function(a, b) {
                    if (a['dist'] < b['dist']) return -1;
                    return 1;
                }
            );
        }
        monster.act.dest.x = dest[0].x;
        monster.act.dest.y = dest[0].y;
        monster.act.status = 'move';
    }
    if (monster.act.status == 'move') {
        // 移動先までの座標
        var toDestX = monster.act.dest.x - monster.x;
        var toDestY = monster.act.dest.y - monster.y;
        // そのターンのx,y方向の移動座標
        var moveX = 0, moveY = 0;
        if (toDestX == 0 && toDestY == 0)
            monster.act.status = 'thinkDest';
        else {
            if (toDestY < 0) {
                if (Math.abs(toDestY) > Math.abs(toDestX))
                    moveY = -1;
                else {
                    if (toDestX > 0) moveX = 1;
                    else moveX = -1;
                }
            } else {
                if (Math.abs(toDestY) > Math.abs(toDestX))
                    moveY = 1;
                else {
                    if (toDestX > 0) moveX = 1;
                    else moveX = -1;
                }
            }
        }
        // 移動する
        // ！マップ範囲外への移動、モンスターやプレイヤーとのの衝突、移動先の障害物は考慮していない
/*
        if (tile[monster.x][monster.y].monster)
            delete tile[monster.x][monster.y].monster;
        monster.x += moveX;
        monster.y += moveY;
        tile[monster.x][monster.y].monster = monster;
*/
        // 現在の座標を保存。移動できない場合を検出するために使用
        monster.last.x = monster.x;
        monster.last.y = monster.y;
        return {x: moveX, y: moveY};
    }
}

/**
 * プレイヤーのターン毎に行う処理
 */
function turnExec() {
/// 休憩時等にターンを飛ばす
    if (player.skipTurns && player.skipTurns.type) {
        var isFinishedSkipTurns = false; // ターンスキップが終了するか
        if (player.skipTurns.type != 'rest' && --player.skipTurns.num == 0) {
            if (player.skipTurns.type == 'craft')
                craftItem(player.skipTurns.craftItem);
            isFinishedSkipTurns = true;
        }
        switch (player.skipTurns.type) {
            case 'sleep':
                player.wait += WAITS_PER_SKIP_TURN.sleep;
                player.stamina += STAMINA_REGENERATION_PER_MINUTE_BY_REST *
                    WAITS_PER_SKIP_TURN.sleep * SECONDS_PER_WAIT / 60;
                break;
            case 'craft':
                player.wait += WAITS_PER_SKIP_TURN.craft;
                break;
            case 'rest':
                if (player.stamina >= player.dexterity)
                    isFinishedSkipTurns = true;
                else {
                    player.wait += WAITS_PER_SKIP_TURN.rest;
                    player.stamina += STAMINA_REGENERATION_PER_MINUTE_BY_REST *
                        WAITS_PER_SKIP_TURN.rest * SECONDS_PER_WAIT / 60;
                }
                break;
        }
        if (isFinishedSkipTurns) {
            delete player.skipTurns.type;
            $('#loading').hide();
        }
    }
    // 自動回復をさせない
    game.staminaTimer = -1;
    game.healthTimer = -1;
    game.hungerTimer = -1;
    game.thirstTimer = -1;
/// 日照量の変化
    var nowHours = game.time.now.getHoursFloat();
    if (nowHours > 5 && nowHours < 19)
        // 日照量の変化をリアルにするためにsin曲線を使用
        player.light = LIGHT_AT_NIGHT - LIGHT_AT_NIGHT *
            (
                Math.sin(Math.PI * (1.7 + 1.6 * (nowHours - 5) / (19 - 5))) -
                Math.sin(Math.PI * 1.7)
            ) / (1 - Math.sin(Math.PI * 1.7));
    else player.light = LIGHT_AT_NIGHT;

/// アイテムの色枠表示
// 腐る寸前
    for (var i = 0; i < player.invItems.length; i++) {
        var item = player.invItems[i];
        if (!item) continue;
        if (!item.isDispDecay && item.decay < DECAY_SOON_DISPLAY_HOURS * 3600) {
            item.isDispDecay = true;
            // アイテムがコンテナ内にある場合は未実装
            if (false/*containerId*/)
                ui.$container.find('.item[data-itemid="' + i + '"]').
                    addClass('damaged');
            else ui.$inventoryEquip.find('.item[data-itemid="' + i + '"]').
                addClass('decay');
        }
    }

/// 天候
    var weather = game.weather;
    // 雨
    if (weather.type == 'rain') {
        if (random(150) == 0) weather.type = 'fine';
        for (var i = 0; i < tileItems.length; i++) {
            var item = tileItems[i];
            if (!item) continue;
            // ボウルに雨水が貯まる
            if (item.type == 'skullBowl') {
                if (!item.waterFill) item.waterFill = 0;
                if (++item.waterFill > 100) {
                    item.type = 'freshWaterInSkullBowl';
                    item.waterFill = 0;
                }
            }
        }
    } else {
        if (random(500) == 0) weather.type = 'rain';
    }
/// ゾンビが湧く
    if (player.light >= 0.65) {
        if (random(40) == 0) {
            spawnX = game.base.x + random(-17, 17);
            spawnY = game.base.y + random(-17, 17);
            if (tile[spawnX] && tile[spawnX][spawnY])
                if (!tiletypes[tile[spawnX][spawnY].type].water) //海上でなければ配置
                    spawnMonster("zombie", spawnX, spawnY);
//            if (spawnMonster("chicken", spawnX, spawnY)) で配置できたか確認できる
        }
    }
/// 漂着物が湧く
    if (nowHours > 4.5 && nowHours < 16 &&
        random(1000000) <
        1000000 * game.time.passInThisTurn / (1 * 3600) * ITEM_SPAWN_CHANCE) {
        var itemSpawn = game.spawnItem.tile[random(game.spawnItem.tile.length)];
        if (itemSpawn) {
            var itemType = game.spawnItem.lot[itemSpawn.type].draw();
            placeItem({type: itemType,
                x: itemSpawn.x, y: itemSpawn.y,
                decay: items[itemType].decayable ?
                    Math.round(items[itemType].decayable[0] / 5 *
                        random(5, 15) / 10) : -1,
                durability: "Random"}, 'TILE', false);
        }
    }
}

// mod実行
if (IS_RELEASE_VERSION) {
    initMod();
}
