const STORAGE_KEY = 'tomodachi-lite-save-v1';

const THOUGHTS = {
  Happy: ['This is great! 😊', 'I feel amazing!', 'What a day!', 'So happy!', 'Wonderful!'],
  Shy: ['Oh... hi', 'Um... hello', 'That is nice...', 'M-maybe...', 'I am okay...'],
  Energetic: ['Let\'s go! 🎉', 'This is awesome!', 'WOOOAH!', 'Time to move!', 'YEAH!'],
  Creative: ['I have an idea...', 'This is inspiring!', 'Let me think.', 'Nice vibe!', 'I like this!'],
  Calm: ['So peaceful...', 'I needed this.', 'Nice and quiet.', 'Feels good.', 'Ahh...'],
  Goofy: ['Haha! 😆', 'Silly time!', 'Heehee!', 'This is funny!', 'Wheee!']
};

const zoneNames = {
  plaza: 'Plaza',
  park: 'Park',
  cafe: 'Cafe',
  beach: 'Beach'
};

const defaultState = {
  day: 1,
  money: 200,
  selectedId: null,
  zone: 'plaza',
  paused: false,
  log: ['Welcome to Sunset Bay!'],
  characters: [
    {
      id: 1,
      name: 'Milo',
      personality: 'Happy',
      hobby: 'Drawing',
      color: '#ff9ecf',
      mood: 76,
      hunger: 72,
      energy: 70,
      happiness: 82,
      social: 68,
      health: 80,
      age: 18,
      relationship: 0,
      favoriteFood: 'berry parfait',
      icon: '😊',
      x: 120,
      y: 190,
      targetX: 180,
      targetY: 230,
      vx: 0.7,
      vy: 0.2,
      activity: 'walking'
    },
    {
      id: 2,
      name: 'Luna',
      personality: 'Shy',
      hobby: 'Reading',
      color: '#91d4ff',
      mood: 72,
      hunger: 68,
      energy: 74,
      happiness: 78,
      social: 63,
      health: 81,
      age: 19,
      relationship: 0,
      favoriteFood: 'tea cake',
      icon: '🙂',
      x: 420,
      y: 320,
      targetX: 500,
      targetY: 260,
      vx: -0.5,
      vy: 0.3,
      activity: 'walking'
    }
  ]
};

const state = loadState();
let selectedCharacterId = state.selectedId ?? state.characters[0]?.id ?? null;
let lastThoughtAt = {};

const elements = {
  dayCounter: document.querySelector('#dayCounter'),
  moneyCounter: document.querySelector('#moneyCounter'),
  gameCanvas: document.querySelector('#gameCanvas'),
  selectedCharacterPanel: document.querySelector('#selectedCharacterPanel'),
  characterForm: document.querySelector('#characterForm'),
  nextDayBtn: document.querySelector('#nextDayBtn'),
  pauseBtn: document.querySelector('#pauseBtn'),
  saveBtn: document.querySelector('#saveBtn'),
  resetBtn: document.querySelector('#resetBtn')
};

function uid() {
  return Date.now() + Math.random().toString(16).slice(2);
}

function clamp(value, min = 0, max = 100) {
  return Math.max(min, Math.min(max, value));
}

function randomInt(min, max) {
  return Math.floor(Math.random() * (max - min + 1)) + min;
}

function loadState() {
  const saved = localStorage.getItem(STORAGE_KEY);
  if (!saved) return structuredClone(defaultState);

  try {
    return JSON.parse(saved);
  } catch {
    return structuredClone(defaultState);
  }
}

function saveState() {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(state));
}

function addLog(message) {
  state.log.unshift(message);
  if (state.log.length > 8) state.log = state.log.slice(0, 8);
}

function getCharacterById(id) {
  return state.characters.find((character) => character.id === id) || null;
}

function updateHud() {
  elements.dayCounter.textContent = String(state.day);
  elements.moneyCounter.textContent = `$${state.money}`;
}

function buildGroundObjects() {
  const objects = [];
  const zone = state.zone;

  if (zone === 'plaza') {
    objects.push({ type: 'house', x: 90, y: 90 });
    objects.push({ type: 'house', x: 650, y: 120 });
    objects.push({ type: 'tree', x: 260, y: 360 });
    objects.push({ type: 'tree', x: 560, y: 390 });
  }

  if (zone === 'park') {
    objects.push({ type: 'tree', x: 140, y: 260 });
    objects.push({ type: 'tree', x: 520, y: 190 });
    objects.push({ type: 'tree', x: 690, y: 320 });
    objects.push({ type: 'tree', x: 240, y: 430 });
  }

  if (zone === 'cafe') {
    objects.push({ type: 'house', x: 280, y: 170 });
    objects.push({ type: 'tree', x: 120, y: 420 });
    objects.push({ type: 'tree', x: 680, y: 240 });
  }

  if (zone === 'beach') {
    objects.push({ type: 'tree', x: 180, y: 120 });
    objects.push({ type: 'tree', x: 620, y: 300 });
    objects.push({ type: 'tree', x: 760, y: 150 });
  }

  return objects;
}

function renderScene() {
  const zone = state.zone;
  elements.gameCanvas.dataset.zone = zone;
  elements.gameCanvas.innerHTML = '';

  const ground = document.createElement('div');
  ground.className = 'ground-objects';

  buildGroundObjects().forEach((obj) => {
    const el = document.createElement('div');
    el.className = obj.type;
    el.style.left = `${obj.x}px`;
    el.style.top = `${obj.y}px`;
    ground.appendChild(el);
  });

  elements.gameCanvas.appendChild(ground);

  state.characters.forEach((character) => {
    const sprite = document.createElement('div');
    sprite.className = `character-sprite ${selectedCharacterId === character.id ? 'active' : ''}`;
    sprite.style.left = `${character.x}px`;
    sprite.style.top = `${character.y}px`;
    sprite.innerHTML = `
      <div class="sprite-body" style="background:${character.color}45; border-radius:16px;">${character.icon}</div>
      <div class="sprite-name">${character.name}</div>
    `;

    sprite.addEventListener('click', () => {
      selectedCharacterId = character.id;
      state.selectedId = character.id;
      renderSelectedCharacter();
      renderScene();
    });

    elements.gameCanvas.appendChild(sprite);

    if (Math.random() < 0.013 && Date.now() - (lastThoughtAt[character.id] || 0) > 4000) {
      const thoughtList = THOUGHTS[character.personality] || THOUGHTS.Happy;
      const bubble = document.createElement('div');
      bubble.className = 'thought-bubble';
      bubble.textContent = thoughtList[randomInt(0, thoughtList.length - 1)];
      bubble.style.left = `${character.x + 40}px`;
      bubble.style.top = `${character.y - 36}px`;
      elements.gameCanvas.appendChild(bubble);
      lastThoughtAt[character.id] = Date.now();
      setTimeout(() => bubble.remove(), 2400);
    }
  });
}

function renderStatBar(label, value) {
  return `
    <div class="bar">
      <div class="bar-label"><span>${label}</span><span>${value}%</span></div>
      <div class="bar-track"><div class="bar-fill" style="width:${value}%"></div></div>
    </div>
  `;
}

function renderSelectedCharacter() {
  const character = selectedCharacterId ? getCharacterById(selectedCharacterId) : null;
  if (!character) {
    elements.selectedCharacterPanel.innerHTML = '<p class="eyebrow">Selected resident</p><h2>Click on a resident</h2>';
    return;
  }

  state.selectedId = character.id;
  elements.selectedCharacterPanel.innerHTML = `
    <div class="selected-header">
      <div>
        <p class="eyebrow">Selected resident</p>
        <h2>${character.name}</h2>
      </div>
      <div class="big-avatar" style="background: linear-gradient(180deg, ${character.color}AA, white);">${character.icon}</div>
    </div>

    <div class="details-grid">
      <div class="detail-item"><strong>Personality:</strong> ${character.personality}</div>
      <div class="detail-item"><strong>Hobby:</strong> ${character.hobby}</div>
      <div class="detail-item"><strong>Favorite snack:</strong> ${character.favoriteFood}</div>
      <div class="detail-item"><strong>Relationship:</strong> ${character.relationship} hearts</div>
    </div>

    <div class="character-stats">
      ${renderStatBar('Mood', character.mood)}
      ${renderStatBar('Hunger', character.hunger)}
      ${renderStatBar('Energy', character.energy)}
      ${renderStatBar('Happiness', character.happiness)}
      ${renderStatBar('Social', character.social)}
      ${renderStatBar('Health', character.health)}
    </div>

    <div class="action-grid">
      <button class="action-btn primary" data-action="feed">Feed</button>
      <button class="action-btn" data-action="play">Play</button>
      <button class="action-btn" data-action="rest">Rest</button>
      <button class="action-btn" data-action="talk">Talk</button>
    </div>

    <div>
      <h3 style="margin-bottom:8px;">Town log</h3>
      <div class="log-list">
        ${state.log.map((entry) => `<div class="log-entry">${entry}</div>`).join('')}
      </div>
    </div>
  `;

  elements.selectedCharacterPanel.querySelectorAll('[data-action]').forEach((button) => {
    button.addEventListener('click', () => applyAction(button.dataset.action, character));
  });
}

function createCharacter(name, personality, hobby, color) {
  const newCharacter = {
    id: Number(uid().replace(/\D/g, '').slice(0, 8)) || Date.now(),
    name,
    personality,
    hobby,
    color,
    mood: 74,
    hunger: 70,
    energy: 72,
    happiness: 78,
    social: 72,
    health: 80,
    age: 18,
    relationship: 0,
    favoriteFood: ['fruit bowl', 'ramen', 'tea cake', 'berry shake'][randomInt(0, 3)],
    icon: ['😊', '🙂', '😄', '😎', '😁'][randomInt(0, 4)],
    x: randomInt(60, 700),
    y: randomInt(80, 470),
    targetX: randomInt(60, 700),
    targetY: randomInt(80, 470),
    vx: (Math.random() - 0.5) * 1.1,
    vy: (Math.random() - 0.5) * 0.8,
    activity: 'walking'
  };

  state.characters.push(newCharacter);
  selectedCharacterId = newCharacter.id;
  addLog(`${newCharacter.name} moved into Sunset Bay!`);
  saveState();
}

function applyAction(action, character) {
  if (!character) return;

  switch (action) {
    case 'feed':
      character.hunger = clamp(character.hunger + 18);
      character.happiness = clamp(character.happiness + 12);
      character.health = clamp(character.health + 8);
      character.activity = 'eating';
      addLog(`${character.name} enjoyed a ${character.favoriteFood}.`);
      break;
    case 'play':
      character.happiness = clamp(character.happiness + 16);
      character.social = clamp(character.social + 14);
      character.energy = clamp(character.energy - 8);
      character.activity = 'playing';
      addLog(`${character.name} had a fun ${character.hobby} session!`);
      break;
    case 'rest':
      character.energy = clamp(character.energy + 18);
      character.mood = clamp(character.mood + 10);
      character.activity = 'resting';
      addLog(`${character.name} took a break and feels refreshed.`);
      break;
    case 'talk':
      character.social = clamp(character.social + 18);
      character.relationship = clamp(character.relationship + 1, 0, 100);
      character.happiness = clamp(character.happiness + 10);
      character.activity = 'talking';
      addLog(`${character.name} caught up with you. Friendship +1!`);
      break;
    default:
      break;
  }

  character.mood = clamp(Math.round((character.happiness + character.social + character.health) / 3));
  saveState();
  renderSelectedCharacter();
}

function moveCharacters() {
  const width = elements.gameCanvas.clientWidth || 900;
  const height = elements.gameCanvas.clientHeight || 600;

  state.characters.forEach((character) => {
    if (state.paused) return;

    const dx = character.targetX - character.x;
    const dy = character.targetY - character.y;
    const distance = Math.sqrt(dx * dx + dy * dy);

    if (distance < 10 || Math.random() < 0.01) {
      character.targetX = randomInt(30, width - 70);
      character.targetY = randomInt(40, height - 80);
    }

    const maxSpeed = 0.8;
    const moveX = (dx / (distance || 1)) * maxSpeed;
    const moveY = (dy / (distance || 1)) * maxSpeed;

    character.x = clamp(character.x + moveX, 10, width - 70);
    character.y = clamp(character.y + moveY, 10, height - 90);
    character.activity = distance > 30 ? 'walking' : 'thinking';
  });
}

function advanceDay() {
  state.day += 1;
  state.money = Math.max(0, state.money + 25);

  state.characters.forEach((character) => {
    character.hunger = clamp(character.hunger - randomInt(4, 10));
    character.energy = clamp(character.energy - randomInt(3, 8));
    character.social = clamp(character.social - randomInt(2, 7));
    character.health = clamp(character.health - randomInt(2, 7));
    character.happiness = clamp(character.happiness - randomInt(1, 8));
    character.mood = clamp(Math.round((character.happiness + character.social + character.health) / 3));

    if (character.hunger < 30 || character.energy < 25) {
      character.happiness = clamp(character.happiness - 8);
      addLog(`${character.name} needs a little extra attention today.`);
    } else {
      character.relationship = clamp(character.relationship + randomInt(0, 2), 0, 100);
    }
  });

  addLog(`Day ${state.day} begins in Sunset Bay.`);
  updateHud();
  saveState();
  renderSelectedCharacter();
}

function resetTown() {
  if (!window.confirm('Reset all town progress?')) return;
  localStorage.removeItem(STORAGE_KEY);
  Object.assign(state, structuredClone(defaultState));
  selectedCharacterId = state.characters[0]?.id ?? null;
  updateHud();
  renderSelectedCharacter();
  saveState();
}

function handleFormSubmit(event) {
  event.preventDefault();
  const name = document.querySelector('#nameInput').value.trim();
  const personality = document.querySelector('#personalityInput').value;
  const hobby = document.querySelector('#hobbyInput').value;
  const color = document.querySelector('#colorInput').value;

  if (!name) return;
  createCharacter(name, personality, hobby, color);
  elements.characterForm.reset();
  document.querySelector('#nameInput').value = 'Aiko';
  renderSelectedCharacter();
}

function updateSceneLoop() {
  if (!state.paused) {
    moveCharacters();
    renderScene();
  }
  requestAnimationFrame(updateSceneLoop);
}

function initControls() {
  elements.characterForm.addEventListener('submit', handleFormSubmit);
  elements.nextDayBtn.addEventListener('click', advanceDay);
  elements.pauseBtn.addEventListener('click', () => {
    state.paused = !state.paused;
    elements.pauseBtn.textContent = state.paused ? 'Resume' : 'Pause';
  });
  elements.saveBtn.addEventListener('click', () => {
    saveState();
    addLog('The game was saved.');
    renderSelectedCharacter();
  });
  elements.resetBtn.addEventListener('click', resetTown);

  document.querySelectorAll('.map-tab').forEach((button) => {
    button.addEventListener('click', () => {
      state.zone = button.dataset.zone;
      document.querySelectorAll('.map-tab').forEach((tab) => tab.classList.toggle('active', tab === button));
      renderScene();
    });
  });
}

function init() {
  initControls();
  updateHud();
  renderSelectedCharacter();
  renderScene();
  requestAnimationFrame(updateSceneLoop);
}

init();
