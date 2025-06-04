# Basketball-retro
A basketball game
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Basketball Phaser Game</title>
  <style>body,html { margin: 0; padding: 0; overflow: hidden; }</style>
</head>
<body>
  <div id="game-container"></div>
  <script type="module" src="main.js"></script>
</body>
</html>

import Phaser from 'phaser';
import firebaseInit, { auth, onAuthStateChanged } from './firebase.js';

import LoginScene from './scenes/LoginScene.js';
import MainMenuScene from './scenes/MainMenuScene.js';
import LobbyScene from './scenes/LobbyScene.js';
import GameScene from './scenes/GameScene.js';
import InventoryScene from './scenes/InventoryScene.js';
import ShopScene from './scenes/ShopScene.js';
import ProfileScene from './scenes/ProfileScene.js';
import SettingsScene from './scenes/SettingsScene.js';
import TutorialScene from './scenes/TutorialScene.js';
import DailyRewardsScene from './scenes/DailyRewardsScene.js';

firebaseInit();

const config = {
  type: Phaser.AUTO,
  width: 800,
  height: 600,
  parent: 'game-container',
  physics: { default: 'arcade', arcade: { debug: false } },
  scene: [
    LoginScene,
    MainMenuScene,
    LobbyScene,
    GameScene,
    InventoryScene,
    ShopScene,
    ProfileScene,
    SettingsScene,
    TutorialScene,
    DailyRewardsScene,
  ],
};

const game = new Phaser.Game(config);

// Handle auth state changes to auto-start LoginScene or MainMenuScene
onAuthStateChanged(auth, user => {
  if (user) {
    game.scene.start('MainMenuScene');
  } else {
    game.scene.start('LoginScene');
  }
});

import { initializeApp } from 'firebase/app';
import {
  getAuth,
  signInWithEmailAndPassword,
  createUserWithEmailAndPassword,
  signOut,
  onAuthStateChanged,
  updateProfile,
} from 'firebase/auth';

import {
  getFirestore,
  doc,
  getDoc,
  setDoc,
  updateDoc,
  collection,
  query,
  where,
  getDocs,
  addDoc,
} from 'firebase/firestore';

const firebaseConfig = {
  apiKey: 'YOUR_API_KEY',
  authDomain: 'YOUR_PROJECT.firebaseapp.com',
  projectId: 'YOUR_PROJECT_ID',
  storageBucket: 'YOUR_PROJECT.appspot.com',
  messagingSenderId: 'SENDER_ID',
  appId: 'APP_ID',
};

let app, auth, db;

export default function firebaseInit() {
  app = initializeApp(firebaseConfig);
  auth = getAuth(app);
  db = getFirestore(app);
}

export {
  auth,
  db,
  signInWithEmailAndPassword,
  createUserWithEmailAndPassword,
  signOut,
  onAuthStateChanged,
  updateProfile,
  doc,
  getDoc,
  setDoc,
  updateDoc,
  collection,
  query,
  where,
  getDocs,
  addDoc,
};

import Phaser from 'phaser';
import {
  auth,
  signInWithEmailAndPassword,
  createUserWithEmailAndPassword,
} from '../firebase.js';

export default class LoginScene extends Phaser.Scene {
  constructor() {
    super('LoginScene');
  }

  create() {
    this.add.text(400, 100, 'Login / Signup', { fontSize: '32px' }).setOrigin(0.5);

    this.emailInput = this.createInput(400, 180, 'Email');
    this.passwordInput = this.createInput(400, 230, 'Password', true);

    this.messageText = this.add.text(400, 320, '', { fontSize: '16px', color: '#f00' }).setOrigin(0.5);

    const loginBtn = this.createButton(300, 280, 'Login', () => this.login());
    const signupBtn = this.createButton(500, 280, 'Signup', () => this.signup());
  }

  createInput(x, y, placeholder, isPassword = false) {
    const input = document.createElement('input');
    input.type = isPassword ? 'password' : 'text';
    input.placeholder = placeholder;
    input.style.position = 'absolute';
    input.style.left = `${x - 100}px`;
    input.style.top = `${y + this.game.canvas.offsetTop}px`;
    input.style.width = '200px';
    input.style.fontSize = '16px';
    document.body.appendChild(input);
    this.sys.game.events.on('destroy', () => input.remove());
    return input;
  }

  createButton(x, y, text, callback) {
    const btn = this.add.text(x, y, text, {
      fontSize: '22px',
      backgroundColor: '#008800',
      padding: { left: 10, right: 10, top: 5, bottom: 5 },
    }).setOrigin(0.5).setInteractive();

    btn.on('pointerup', callback);
    return btn;
  }

  async login() {
    const email = this.emailInput.value.trim();
    const password = this.passwordInput.value.trim();
    if (!email || !password) {
      this.messageText.setText('Enter email and password');
      return;
    }
    try {
      await signInWithEmailAndPassword(auth, email, password);
      this.messageText.setText('');
      this.scene.start('MainMenuScene');
      this.cleanupInputs();
    } catch (e) {
      this.messageText.setText('Login failed: ' + e.message);
    }
  }

  async signup() {
    const email = this.emailInput.value.trim();
    const password = this.passwordInput.value.trim();
    if (!email || !password) {
      this.messageText.setText('Enter email and password');
      return;
    }
    try {
      await createUserWithEmailAndPassword(auth, email, password);
      this.messageText.setText('Signup successful, please login');
    } catch (e) {
      this.messageText.setText('Signup failed: ' + e.message);
    }
  }

  cleanupInputs() {
    this.emailInput.remove();
    this.passwordInput.remove();
  }
}

import Phaser from 'phaser';
import { auth, signOut } from '../firebase.js';

export default class MainMenuScene extends Phaser.Scene {
  constructor() {
    super('MainMenuScene');
  }

  create() {
    const user = auth.currentUser;
    this.add.text(400, 50, `Welcome ${user?.email}`, { fontSize: '28px', color: '#fff' }).setOrigin(0.5);

    this.createMenuButton(400, 150, 'Start Matchmaking', () => this.scene.start('LobbyScene'));
    this.createMenuButton(400, 220, 'Inventory', () => this.scene.start('InventoryScene'));
    this.createMenuButton(400, 290, 'Shop', () => this.scene.start('ShopScene'));
    this.createMenuButton(400, 360, 'Profile', () => this.scene.start('ProfileScene'));
    this.createMenuButton(400, 430, 'Settings', () => this.scene.start('SettingsScene'));
    this.createMenuButton(400, 500, 'Daily Rewards', () => this.scene.start('DailyRewardsScene'));
    this.createMenuButton(400, 570, 'Logout', () => {
      signOut(auth).then(() => this.scene.start('LoginScene'));
    });
  }

  createMenuButton(x, y, label, callback) {
    const btn = this.add.text(x, y, label, {
      fontSize: '24px',
      backgroundColor: '#0066cc',
      color: '#fff',
      padding: { left: 20, right: 20, top: 10, bottom: 10 },
      align: 'center',
      fixedWidth: 250,
    }).setOrigin(0.5).setInteractive();

    btn.on('pointerup', callback);
  }
}

import Phaser from 'phaser';
import { auth, db, collection, query, where, getDocs, addDoc, updateDoc, doc, onSnapshot } from '../firebase.js';

export default class LobbyScene extends Phaser.Scene {
  constructor() {
    super('LobbyScene');
  }

  init() {
    this.matchDocRef = null;
  }

  async create() {
    this.add.text(400, 50, 'Finding Match...', { fontSize: '32px' }).setOrigin(0.5);
    this.statusText = this.add.text(400, 100, '', { fontSize: '20px' }).setOrigin(0.5);

    // Start matchmaking: look for waiting match or create one
    await this.startMatchmaking();
  }

  async startMatchmaking() {
    const matchesRef = collection(db, 'matches');
    // Look for a match with 1 player waiting
    const q = query(matchesRef, where('status', '==', 'waiting'));
    const querySnapshot = await getDocs(q);

    let matchFound = false;
    for (const docSnap of querySnapshot.docs) {
      const matchData = docSnap.data();
      if (matchData.players.length === 1) {
        // Join this match
        this.matchDocRef = doc(db, 'matches', docSnap.id);
        await updateDoc(this.matchDocRef, {
          players: [...matchData.players, auth.currentUser.uid],
          status: 'ready',
        });
        matchFound = true;
        break;
      }
    }

    if (!matchFound) {
      // Create new match with this player waiting
      this.matchDocRef = await addDoc(matchesRef, {
        players: [auth.currentUser.uid],
        status: 'waiting',
        createdAt: Date.now(),
      });
    }

    // Listen for match status updates
    this.unsubscribe = onSnapshot(this.matchDocRef, (docSnap) => {
      const data = docSnap.data();
      if (data.status === 'ready' && data.players.length === 2) {
        this.statusText.setText('Match found! Starting game...');
        setTimeout(() => {
          this.scene.start('GameScene', { matchId: this.matchDocRef.id, players: data.players });
        }, 2000);
      } else if (data.status === 'waiting') {
        this.statusText.setText('Waiting for another player...');
      }
    });
  }

  shutdown() {
    if (this.unsubscribe) this.unsubscribe();
  }

  destroy() {
    if (this.unsubscribe) this.unsubscribe();
  }
}


import Phaser from 'phaser';
import { auth, db, doc, updateDoc, getDoc } from '../firebase.js';

export default class GameScene extends Phaser.Scene {
  constructor() {
    super('GameScene');
  }

  init(data) {
    this.matchId = data.matchId;
    this.players = data.players;
    this.playerId = auth.currentUser.uid;
    this.opponentId = this.players.find(p => p !== this.playerId);
    this.score = { [this.playerId]: 0, [this.opponentId]: 0 };
    this.timeLeft = 60; // seconds
  }

  async create() {
    this.add.text(400, 30, 'Game Started!', { fontSize: '28px' }).setOrigin(0.5);
    this.scoreText = this.add.text(400, 70, `Score: 0 - 0`, { fontSize: '24px' }).setOrigin(0.5);
    this.timerText = this.add.text(400, 110, `Time Left: ${this.timeLeft}`, { fontSize: '24px' }).setOrigin(0.5);

    this.input.keyboard.on('keydown-SPACE', () => {
      this.score[this.playerId]++;
      this.updateScore();
    });

    this.timer = this.time.addEvent({
      delay: 1000,
      callback: this.updateTimer,
      callbackScope: this,
      loop: true,
    });
  }

  updateScore() {
    this.scoreText.setText(`Score: ${this.score[this.playerId]} - ${this.score[this.opponentId]}`);
  }

  updateTimer() {
    this.timeLeft--;
    this.timerText.setText(`Time Left: ${this.timeLeft}`);

    if (this.timeLeft <= 0) {
      this.timer.remove(false);
      this.endMatch();
    }
  }

  async endMatch() {
    let resultText;
    if (this.score[this.playerId] > this.score[this.opponentId]) {
      resultText = 'You Win!';
      await this.giveRewards(true);
    } else if (this.score[this.playerId] < this.score[this.opponentId]) {
      resultText = 'You Lose!';
      await this.giveRewards(false);
    } else {
      resultText = 'Draw!';
      await this.giveRewards(false);
    }

    this.add.text(400, 200, resultText, { fontSize: '32px' }).setOrigin(0.5);

    setTimeout(() => {
      this.scene.start('MainMenuScene');
    }, 5000);
  }

  async giveRewards(win) {
    const userDoc = doc(db, 'users', this.playerId);
    const userSnap = await getDoc(userDoc);
    let data = userSnap.exists() ? userSnap.data() : {};

    data.coins = (data.coins || 0) + (win ? 10 : 2);
    data.xp = (data.xp || 0) + (win ? 15 : 5);

    await updateDoc(userDoc, data);
  }
}

import Phaser from 'phaser';
import { auth, db, doc, getDoc } from '../firebase.js';

const RARITY_COLORS = {
  gray: 0x888888,
  green: 0x00ff00,
  blue: 0x0077ff,
  purple: 0x7700ff,
  gold: 0xffcc00,
};

export default class InventoryScene extends Phaser.Scene {
  constructor() {
    super('InventoryScene');
  }

  async create() {
    this.add.text(400, 50, 'Inventory', { fontSize: '32px' }).setOrigin(0.5);

    const userDoc = doc(db, 'users', auth.currentUser.uid);
    const userSnap = await getDoc(userDoc);
    const inventory = (userSnap.exists() && userSnap.data().inventory) || [];

    if (!inventory.length) {
      this.add.text(400, 300, 'No players unlocked yet', { fontSize: '24px' }).setOrigin(0.5);
      return;
    }

    inventory.forEach((player, i) => {
      const y = 120 + i * 60;
      const color = RARITY_COLORS[player.rarity] || 0xffffff;
      const bg = this.add.rectangle(400, y, 500, 50, color).setOrigin(0.5);
      const nameText = this.add.text(250, y, player.name, { fontSize: '24px', color: '#000' }).setOrigin(0, 0.5);
      const statsText = this.add.text(300, y, `Speed: ${player.speed} Shot: ${player.shot} Def: ${player.defense}`, { fontSize: '18px', color: '#000' }).setOrigin(0, 0.5);
    });

    this.createButton(700, 550, 'Back', () => this.scene.start('MainMenuScene'));
  }

  createButton(x, y, label, callback) {
    const btn = this.add.text(x, y, label, {
      fontSize: '20px',
      backgroundColor: '#0088cc',
      color: '#fff',
      padding: { left: 10, right: 10, top: 5, bottom: 5 },
    }).setOrigin(0.5).set

    let db, uid;

export async function initializeFirebase() {
  const config = {
    apiKey: "YOUR_API_KEY",
    authDomain: "YOUR_PROJECT.firebaseapp.com",
    projectId: "YOUR_PROJECT_ID",
    storageBucket: "YOUR_PROJECT.appspot.com",
    messagingSenderId: "SENDER_ID",
    appId: "APP_ID"
  };
  firebase.initializeApp(config);

  await firebase.auth().signInAnonymously();
  uid = firebase.auth().currentUser.uid;
  db = firebase.firestore();
}

export async function loadProgress() {
  const doc = await db.collection('players').doc(uid).get();
  const data = doc.exists ? doc.data() : {};

  localStorage.setItem('xp', data.xp || 0);
  localStorage.setItem('coins', data.coins || 0);
  localStorage.setItem('unlocked', JSON.stringify(data.unlocked || ['player_red']));
  localStorage.setItem('selectedCharacter', data.selectedCharacter || 'player_red');
  localStorage.setItem('lastLogin', data.lastLogin || '');
}

export function saveProgress() {
  const xp = parseInt(localStorage.getItem('xp') || '0');
  const coins = parseInt(localStorage.getItem('coins') || '0');
  const unlocked = JSON.parse(localStorage.getItem('unlocked') || '[]');
  const selectedCharacter = localStorage.getItem('selectedCharacter');
  const lastLogin = localStorage.getItem('lastLogin');

  return db.collection('players').doc(uid).set({
    xp, coins, unlocked, selectedCharacter, lastLogin
  });
}

import { initializeFirebase, saveProgress, loadProgress } from './utils/firebase.js';
import MainMenu from './scenes/MainMenuScene.js';
import GameScene from './scenes/GameScene.js';
import GameOver from './scenes/GameOverScene.js';
import ProfileScene from './scenes/ProfileScene.js';
import ShopScene from './scenes/ShopScene.js';
import CharacterSelectScene from './scenes/CharacterSelectScene.js';

await initializeFirebase();
await loadProgress();

const config = {
  type: Phaser.AUTO,
  width: 800,
  height: 600,
  backgroundColor: '#222',
  parent: 'game-container',
  scene: [CharacterSelectScene, MainMenu, GameScene, GameOver, ShopScene, ProfileScene],
  physics: {
    default: 'arcade',
    arcade: { gravity: { y: 900 }, debug: false }
  }
};

new Phaser.Game(config);

firebase.auth().signInAnonymously().then(() => {
  const uid = firebase.auth().currentUser.uid;
  const db = firebase.firestore();

  db.collection('players').doc(uid).get().then(doc => {
    if (doc.exists) {
      const data = doc.data();
      localStorage.setItem('xp', data.xp || 0);
      localStorage.setItem('coins', data.coins || 0);
      localStorage.setItem('unlocked', JSON.stringify(data.unlocked || ['player_red']));
      localStorage.setItem('selectedCharacter', data.selectedCharacter || 'player_red');
    }
  });

  // Save after game
  function saveToCloud(xp, coins, unlocked, selectedCharacter) {
    db.collection('players').doc(uid).set({
      xp,
      coins,
      unlocked,
      selectedCharacter
    });
  }

  // Example usage after game
  // saveToCloud(localXP, localCoins, localUnlockedArray, localSelectedCharacter);
});

const lastLogin = localStorage.getItem('lastLogin');
const today = new Date().toDateString();

if (lastLogin !== today) {
  const bonusXP = 10;
  const bonusCoins = 5;
  const xp = parseInt(localStorage.getItem('xp') || '0') + bonusXP;
  const coins = parseInt(localStorage.getItem('coins') || '0') + bonusCoins;

  localStorage.setItem('xp', xp);
  localStorage.setItem('coins', coins);
  localStorage.setItem('lastLogin', today);

  this.add.text(250, 200, `🎁 Daily Reward! +${bonusXP} XP, +${bonusCoins} Coins`, {
    fontSize: '18px',
    fill: '#0ff'
  });
}

class ProfileScene extends Phaser.Scene {
  constructor() {
    super('ProfileScene');
  }

  create() {
    const coins = parseInt(localStorage.getItem('coins') || '0');
    const xp = parseInt(localStorage.getItem('xp') || '0');
    const character = localStorage.getItem('selectedCharacter') || 'player_red';
    const level = calculateLevel(xp);

    this.add.text(300, 80, '🏀 Player Profile', { fontSize: '28px', fill: '#fff' });

    this.add.image(400, 180, character).setScale(1.2);

    this.add.text(320, 300, `Level: ${level}`, { fontSize: '22px', fill: '#0f0' });
    this.add.text(320, 340, `XP: ${xp}`, { fontSize: '20px', fill: '#0ff' });
    this.add.text(320, 380, `Coins: ${coins}`, { fontSize: '20px', fill: '#ff0' });

    this.add.text(320, 450, '← Back', { fontSize: '20px', fill: '#f88' })
      .setInteractive()
      .on('pointerdown', () => this.scene.start('MainMenu'));
  }
}

const unlocked = JSON.parse(localStorage.getItem('unlocked') || '["player_red"]');

if (unlocked.includes('player_red')) { ... }
if (unlocked.includes('player_green')) { ... }
if (unlocked.includes('player_blue')) { ... }

class ShopScene extends Phaser.Scene {
  constructor() {
    super('ShopScene');
  }

  create() {
    let coins = parseInt(localStorage.getItem('coins') || '0');
    this.add.text(300, 100, `Coins: ${coins}`, { fontSize: '20px', fill: '#ff0' });

    this.add.image(300, 200, 'player_green')
      .setInteractive()
      .on('pointerdown', () => this.tryBuy('player_green', 100));

    this.add.image(500, 200, 'player_blue')
      .setInteractive()
      .on('pointerdown', () => this.tryBuy('player_blue', 200));

    this.add.text(300, 350, 'Tap a character to unlock', { fontSize: '16px', fill: '#fff' });
  }

  tryBuy(key, price) {
    let coins = parseInt(localStorage.getItem('coins') || '0');
    let unlocked = JSON.parse(localStorage.getItem('unlocked') || '[]');

    if (coins >= price && !unlocked.includes(key)) {
      coins -= price;
      unlocked.push(key);
      localStorage.setItem('coins', coins);
      localStorage.setItem('unlocked', JSON.stringify(unlocked));
      alert(`${key} unlocked!`);
    } else {
      alert(coins < price ? 'Not enough coins!' : 'Already unlocked!');
    }
  }
}

const level = calculateLevel(totalXP);
this.add.text(300, 400, `Level: ${level}`, { fontSize: '20px', fill: '#0f0' });

function calculateLevel(xp) {
  if (xp >= 300) return 4;
  if (xp >= 150) return 3;
  if (xp >= 50) return 2;
  return 1;
}

let totalXP = parseInt(localStorage.getItem('xp') || '0');
let totalCoins = parseInt(localStorage.getItem('coins') || '0');

totalXP += this.finalXP;
totalCoins += this.finalCoins;

localStorage.setItem('xp', totalXP);
localStorage.setItem('coins', totalCoins);

this.add.text(300, 340, `Total XP: ${totalXP}`, { fontSize: '18px', fill: '#0ff' });
this.add.text(300, 370, `Total Coins: ${totalCoins}`, { fontSize: '18px', fill: '#ff0' });

// On tap
this.input.on('pointerdown', () => {
  this.jumpSound.play();
  this.ball.setVelocityY(-500);
  this.ball.setVelocityX(Phaser.Math.Between(-200, 200));
});

// On score
this.scoreSound.play();

import Phaser from 'https://cdn.jsdelivr.net/npm/phaser@3.60.0/dist/phaser.esm.js';

let score = 0;

const config = {
  type: Phaser.AUTO,
  width: 800,
  height: 600,
  parent: 'game-container',
  backgroundColor: '#222',
  physics: {
    default: 'arcade',
    arcade: {
      gravity: { y: 800 },
      debug: false
    }
  },
  scene: [MainMenu, GameScene, GameOver]
};

const game = new Phaser.Game(config);

// --- Main Menu Scene ---
class MainMenu extends Phaser.Scene {
  constructor() {
    super('MainMenu');
  }
  create() {
    this.add.text(300, 250, '🏀 BASKETBALL GAME', { fontSize: '24px', fill: '#fff' });
    const startBtn = this.add.text(350, 300, 'Tap to Start', { fontSize: '20px', fill: '#0f0' })
      .setInteractive()
      .on('pointerdown', () => this.scene.start('GameScene'));
  }
}

// --- Game Scene ---
class GameScene extends Phaser.Scene {
  constructor() {
    super('GameScene');
  }

  preload() {
    this.load.image('ball', 'https://i.imgur.com/8XKQA7C.png');
    this.load.image('hoop', 'https://i.imgur.com/kYt1S0y.png'); // transparent hoop
  }

  create() {
    score = 0;

    this.scoreText = this.add.text(10, 10, 'Score: 0', { fontSize: '24px', fill: '#fff' });

    this.ball = this.physics.add.image(400, 500, 'ball').setCircle(32).setBounce(0.6).setCollideWorldBounds(true);
    this.hoop = this.physics.add.staticImage(400, 150, 'hoop').setScale(0.5).refreshBody();

    this.physics.add.overlap(this.ball, this.hoop, this.scorePoint, null, this);

    this.input.on('pointerdown', () => {
      this.ball.setVelocityY(-500);
      this.ball.setVelocityX(Phaser.Math.Between(-200, 200)); // adds randomness
    });

    this.timer = this.time.delayedCall(30000, () => {
      this.scene.start('GameOver', { score });
    }, [], this);
  }

  scorePoint(ball, hoop) {
    if (!ball.hasScored) {
      score++;
      this.scoreText.setText('Score: ' + score);
      ball.hasScored = true;
      this.time.delayedCall(1000, () => { ball.hasScored = false; }, [], this);
    }
  }
}

// --- Game Over Scene ---
class GameOver extends Phaser.Scene {
  constructor() {
    super('GameOver');
  }
  init(data) {
    this.finalScore = data.score;
  }
  create() {
    this.add.text(300, 250, 'Game Over', { fontSize: '32px', fill: '#fff' });
    this.add.text(320, 300, `Score: ${this.finalScore}`, { fontSize: '24px', fill: '#fff' });
    const restartBtn = this.add.text(350, 350, 'Play Again', { fontSize: '20px', fill: '#0f0' })
      .setInteractive()
      .on('pointerdown', () => this.scene.start('GameScene'));
    const menuBtn = this.add.text(360, 400, 'Main Menu', { fontSize: '20px', fill: '#0ff' })
      .setInteractive()
      .on('pointerdown', () => this.scene.start('MainMenu'));
  }
}
