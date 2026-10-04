const STORAGE_KEY = 'tomodachi-lite-save-v1';

const THOUGHTS = {
  Happy: ['This is great! 😊', 'I feel amazing!', 'What a day!', 'So happy right now!', 'Haha, nice!'],
  Shy: ['Oh... hi', 'Um... *blushes*', 'That\'s nice...', 'M-maybe...', 'I\'m okay...'],
  Energetic: ['WOOOAH! 🎉', 'Let\'s DO something!', 'This is AWESOME!', 'Yes yes YES!', 'PUMP IT UP!'],
  Creative: ['Hmm, interesting...', 'I like this...', 'Creative vibes', 'Inspiring!', 'Let me draw...'],
  Calm: ['How peaceful...', 'I needed this', 'Serenity...', 'So quiet', 'Ahhh, nice'],
  Goofy: ['HAHAHAHA! 😆', 'Goofy time!', 'Wee hee!', 'That\'s silly!', 'Wheee!']
};

const ACTIVITIES = [
  'stretching',
  'dancing',
  'jumping',
  'waving',
  'thinking',
  'eating'
];

const defaultState = {
  day: 1,
  money: 200,
  selectedId: null,
  zone: 'plaza',
  isPaused: false,
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
      x: 150,
      y: 200,
      vx: 0.5,
      vy: 0,
      activity: 'dancing'
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
      icon: '😊',
      x: 450,
      y: 300,
      vx: -0.3,
      vy: 0,
      activity: 'thinking'
    }
  ]
};

const state = loadState();
let gameRunning = true;
let lastThoughtTime = {};
let selectedCharacterId = null;

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
  return state.characters.find(c => c.id === id);
}

function updateCharacterAnimation() {
  state.characters.forEach(character => {
    character.x += character.vx;
    character.y += character.vy;

    // Bounce off edges
    const canvasWidth = elements.gameCanvas.offsetWidth;
    const canvasHeight = elements.gameCanvas.offsetHeight;
    const charSize = 60;

    if (character.x < 0 || character.x > canvasWidth - charSize) {
      character.vx *= -1;
    }
    if (character.y < 0 || character.y > canvasHeight - charSize) {
      character.vy *= -1;
    }

    character.x = clamp(character.x, 0, canvasWidth - charSize);
    character.y = clamp(character.y, 0, canvasHeight - charSize);

    // Random direction changes
    if (Math.random() < 0.02) {
      character.vx = (Math.random() - 0.5) * 1.2;
      character.vy = (Math.random() - 0.5) * 0.8;
    }

    // Random activities
    if (Math.random() < 0.05) {
      character.activity = ACTIVITIES[randomInt(0, ACTIVITIES.length - 1)];
    }
  });
}

function renderCanvas() {
  elements.gameCanvas.innerHTML = '';

  state.characters.forEach(character => {
    const sprite = document.createElement('div');
    sprite.className = `character-sprite ${selectedCharacterId === character.id ? 'active' : ''}`;
    sprite.style.left = character.x + 'px';
    sprite.style.top = character.y + 'px';

    sprite.innerHTML = `
      <div class="sprite-body">${character.icon}</div>
      <div class="sprite-name">${character.name}</div>
    `;

    sprite.addEventListener('click', () => {
      selectedCharacterId = character.id;
      renderSelectedCharacter();
      renderCanvas();
    });

    elements.gameCanvas.appendChild(sprite);

    // Random thoughts
    if (Math.random() < 0.015 && !lastThoughtTime[character.id]) {
      const thoughts = THOUGHTS[character.personality] || THOUGHTS.Happy;
      const thought = thoughts[randomInt(0, thoughts.length - 1)];
      showThought(character, thought);
      lastThoughtTime[character.id] = Date.now();
      setTimeout(() => delete lastThoughtTime[character.id], 4000);
    }
  });
}

function showThought(character, text) {
  const bubble = document.createElement('div');
  bubble.className = 'thought-bubble';
  bubble.style.left = (character.x + 30) + 'px';
  bubble.style.top = (character.y - 60) + 'px';
  bubble.innerHTML = `<p style="margin:0;">${text}</p>`;

  elements.gameCanvas.appendChild(bubble);

  setTimeout(() => bubble.remove(), 3500);
}

function renderSelectedCharacter() {
  const character = selectedCharacterId ? getCharacterById(selectedCharacterId) : null;
  if (!character) {
    elements.selectedCharacterPanel.innerHTML = '<p class="eyebrow">Selected resident</p><h2>Click on a resident</h2>';
    return;
  }

  elements.selectedCharacterPanel.innerHTML = `
    <div class="selected-header">
      <div>
        <p class="eyebrow">Selected resident</p>
        <h2>${character.name}</h2>
      </div>
      <div class="big-avatar">${character.icon}</div>
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

function renderStatBar(label, value) {
  return `
    <div class="bar">
      <div class="bar-label"><span>${label}</span><span>${value}%</span></div>
      <div class="bar-track"><div class="bar-fill" style="width:${value}%"></div></div>
    </div>
  `;
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
    icon: '😊',
    x: randomInt(50, 300),
    y: randomInt(100, 400),
    vx: (Math.random() - 0.5) * 1,
    vy: (Math.random() - 0.5) * 0.6,
    activity: 'dancing'
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
      addLog(`${character.name} enjoyed a ${character.favoriteFood}. Yum!`);
      break;
    case 'play':
      character.happiness = clamp(character.happiness + 16);
      character.social = clamp(character.social + 14);
      character.energy = clamp(character.energy - 8);
      character.activity = 'dancing';
      addLog(`${character.name} had a fun ${character.hobby} session!`);
      break;
    case 'rest':
      character.energy = clamp(character.energy + 18);
      character.mood = clamp(character.mood + 10);
      addLog(`${character.name} took a break and feels refreshed.`);
      break;
    case 'talk':
      character.social = clamp(character.social + 18);
      character.relationship = clamp(character.relationship + 1, 0, 100);
      character.happiness = clamp(character.happiness + 10);
      character.activity = 'waving';
      addLog(`${character.name} caught up with you. Friendship +1!`);
      break;
    default:
      break;
  }

  character.mood = clamp(Math.round((character.happiness + character.social + character.health) / 3));
  saveState();
  renderSelectedCharacter();
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
      character.happiness = clamp(character.happiness - 10);
      addLog(`${character.name} seems a bit sad today...`);
    } else {
      character.relationship = clamp(character.relationship + randomInt(0, 2), 0, 100);
    }
  });

  addLog(`Day ${state.day} begins!`);
  elements.dayCounter.textContent = String(state.day);
  elements.moneyCounter.textContent = `$${state.money}`;
  saveState();
  renderSelectedCharacter();
}

function resetTown() {
  if (!window.confirm('Reset all town progress?')) return;
  localStorage.removeItem(STORAGE_KEY);
  Object.assign(state, structuredClone(defaultState));
  selectedCharacterId = null;
  elements.dayCounter.textContent = String(state.day);
  elements.moneyCounter.textContent = `$${state.money}`;
  renderSelectedCharacter();
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

function gameLoop() {
  if (!state.isPaused) {
    updateCharacterAnimation();
    renderCanvas();
  }
  requestAnimationFrame(gameLoop);
}

elements.characterForm.addEventListener('submit', handleFormSubmit);
elements.nextDayBtn.addEventListener('click', advanceDay);
elements.pauseBtn.addEventListener('click', () => {
  state.isPaused = !state.isPaused;
  elements.pauseBtn.textContent = state.isPaused ? 'Resume' : 'Pause';
});
elements.saveBtn.addEventListener('click', () => {
  saveState();
  addLog('Game saved!');
  renderSelectedCharacter();
});
elements.resetBtn.addEventListener('click', resetTown);

document.querySelectorAll('.map-tab').forEach((button) => {
  button.addEventListener('click', () => {
    state.zone = button.dataset.zone;
    document.querySelectorAll('.map-tab').forEach((tab) => tab.classList.toggle('active', tab === button));
  });
});

elements.dayCounter.textContent = String(state.day);
elements.moneyCounter.textContent = `$${state.money}`;
renderSelectedCharacter();
gameLoop();
