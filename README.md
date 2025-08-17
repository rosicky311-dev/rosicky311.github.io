# rosicky311.github.io
The duel of the three kingdoms
// 游戏状态
let gameState = {
    player: {
        hero: '刘备',
        hp: 4,
        hand: [],
        maxHp: 4
    },
    ai: {
        hero: '曹操',
        hp: 4,
        hand: [],
        maxHp: 4
    },
    deck: [],
    currentPlayer: 'player',
    phase: 'draw' // draw, play, discard
};

// 卡牌定义
const cardTypes = {
    '杀': { type: 'basic', color: 'red', effect: '造成1点伤害' },
    '闪': { type: 'basic', color: 'black', effect: '抵消杀的伤害' },
    '桃': { type: 'basic', color: 'red', effect: '回复1点体力' }
};

// 武将技能（简化版）
const heroSkills = {
    '刘备': {
        '仁德': () => {
            // 可以给其他角色牌
            return true;
        }
    },
    '曹操': {
        '奸雄': () => {
            // 受到伤害时获得牌
            return true;
        }
    }
};

// 初始化游戏
function initGame() {
    createDeck();
    shuffleDeck();
    dealCards();
    renderGame();
    
    document.getElementById('end-turn').addEventListener('click', endPlayerTurn);
}

// 创建卡组
function createDeck() {
    const suits = ['♠', '♥', '♦', '♣'];
    const values = ['A', '2', '3', '4', '5', '6', '7', '8', '9', '10', 'J', 'Q', 'K'];
    
    // 基础牌
    for (let i = 0; i < 15; i++) {
        gameState.deck.push({name: '杀', ...cardTypes['杀']});
    }
    for (let i = 0; i < 10; i++) {
        gameState.deck.push({name: '闪', ...cardTypes['闪']});
    }
    for (let i = 0; i < 8; i++) {
        gameState.deck.push({name: '桃', ...cardTypes['桃']});
    }
    
    // 随机打乱卡组
    function shuffleDeck() {
        for (let i = gameState.deck.length - 1; i > 0; i--) {
            const j = Math.floor(Math.random() * (i + 1));
            [gameState.deck[i], gameState.deck[j]] = [gameState.deck[j], gameState.deck[i]];
        }
    }
}

// 发牌
function dealCards() {
    for (let i = 0; i < 4; i++) {
        gameState.player.hand.push(gameState.deck.pop());
        gameState.ai.hand.push(gameState.deck.pop());
    }
}

// 渲染游戏界面
function renderGame() {
    renderHand('player');
    renderHand('ai');
    updateHpDisplay();
}

// 渲染手牌
function renderHand(player) {
    const handElement = document.getElementById(`${player}-hand`);
    handElement.innerHTML = '';
    
    const hand = gameState[player].hand;
    hand.forEach((card, index) => {
        const cardElement = document.createElement('div');
        cardElement.className = `card ${card.color === 'red' ? 'red-card' : 'black-card'}`;
        cardElement.textContent = card.name;
        cardElement.addEventListener('click', () => playCard(player, index));
        handElement.appendChild(cardElement);
    });
}

// 更新血量显示
function updateHpDisplay() {
    document.getElementById('player-hp').textContent = gameState.player.hp;
    document.getElementById('ai-hp').textContent = gameState.ai.hp;
}

// 出牌
function playCard(player, cardIndex) {
    if (gameState.currentPlayer !== player || gameState.phase !== 'play') return;
    
    const card = gameState[player].hand[cardIndex];
    const target = player === 'player' ? 'ai' : 'player';
    
    // 执行卡牌效果
    executeCardEffect(card, player, target);
    
    // 移除手牌
    gameState[player].hand.splice(cardIndex, 1);
    
    // 重新渲染
    renderGame();
    logAction(`${player}使用了${card.name}`);
    
    // 检查游戏结束
    checkGameOver();
}

// 执行卡牌效果
function executeCardEffect(card, source, target) {
    switch(card.name) {
        case '杀':
            if (target === 'ai') {
                gameState.ai.hp--;
                logAction(`AI受到了1点伤害`);
            } else {
                gameState.player.hp--;
                logAction(`玩家受到了1点伤害`);
            }
            break;
        case '桃':
            if (source === 'player') {
                gameState.player.hp = Math.min(gameState.player.maxHp, gameState.player.hp + 1);
                logAction(`玩家恢复了1点体力`);
            } else {
                gameState.ai.hp = Math.min(gameState.ai.maxHp, gameState.ai.hp + 1);
                logAction(`AI恢复了1点体力`);
            }
            break;
        // 其他卡牌效果...
    }
}

// 结束玩家回合
function endPlayerTurn() {
    if (gameState.currentPlayer !== 'player') return;
    
    gameState.currentPlayer = 'ai';
    gameState.phase = 'draw';
    
    // AI回合
    setTimeout(aiTurn, 1000);
}

// AI回合逻辑
function aiTurn() {
    logAction("AI回合开始");
    
    // AI摸牌
    for (let i = 0; i < 2; i++) {
        if (gameState.deck.length > 0) {
            gameState.ai.hand.push(gameState.deck.pop());
        }
    }
    
    // AI出牌逻辑（简化版）
    const attackCard = gameState.ai.hand.find(card => card.name === '杀');
    if (attackCard && gameState.player.hp > 0) {
        const cardIndex = gameState.ai.hand.indexOf(attackCard);
        playCard('ai', cardIndex);
    }
    
    // 结束AI回合
    setTimeout(() => {
        gameState.currentPlayer = 'player';
        gameState.phase = 'draw';
        logAction("玩家回合开始");
        
        // 玩家摸牌
        for (let i = 0; i < 2; i++) {
            if (gameState.deck.length > 0) {
                gameState.player.hand.push(gameState.deck.pop());
            }
        }
        renderGame();
    }, 1500);
}

// 记录操作日志
function logAction(message) {
    const logElement = document.getElementById('action-log');
    const logItem = document.createElement('div');
    logItem.textContent = message;
    logElement.appendChild(logItem);
    logElement.scrollTop = logElement.scrollHeight;
}

// 检查游戏结束
function checkGameOver() {
    if (gameState.player.hp <= 0) {
        alert('你输了！');
        resetGame();
    } else if (gameState.ai.hp <= 0) {
        alert('你赢了！');
        resetGame();
    }
}

// 重置游戏
function resetGame() {
    gameState = {
        player: {
            hero: '刘备',
            hp: 4,
            hand: [],
            maxHp: 4
        },
        ai: {
            hero: '曹操',
            hp: 4,
            hand: [],
            maxHp: 4
        },
        deck: [],
        currentPlayer: 'player',
        phase: 'draw'
    };
    document.getElementById('action-log').innerHTML = '';
    initGame();
}

// 启动游戏
initGame();
