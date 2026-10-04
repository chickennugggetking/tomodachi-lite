const STORAGE_KEY = 'tomodachi-lite-save-v1';

const defaultState = {
  day: 1,
  money: 200,
  selectedId: 1,
  zone: 'home',
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
      icon: '🙂'
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
      icon: '😊'
    }
  ]
};

const state = loadState();

const elements = {
  dayCounter: document.querySelector('#dayCounter'),
  townName: document.querySelector('#townName'),
  moneyCounter: document.querySelector('#moneyCounter'),
  characterList: document.querySelector('#characterList'),
  selectedCharacterPanel: document.querySelector('#selectedCharacterPanel'),
  characterForm: document.querySelector('#characterForm'),
  nextDayBtn: document.querySelector('#nextDayBtn'),
  saveBtn: document.querySelector('#saveBtn'),
  resetBtn: document.querySelector('#resetBtn'),
  townZone: document.querySelector('#townZone')
};

const zoneLabels = {
  home: 'Home sweet home',
  park: 'Park outing',
  plaza: 'Town plaza',
  shop: 'Town shop',
  studio: 'Music studio'
};

function uid() {
  return Date.now() + Math.random().toString(16).slice(2);
}

function clamp(value) {
  return Math.max(0, Math.min(100, value));
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
  if (state.log.length > 8) {
    state.log = state.log.slice(0, 8);
  }
}

function getSelectedCharacter() {
  return state.characters.find((char) => char.id === state.selectedId) || state.characters[0];
}

function getZonePeople(zone) {
  const chars = state.characters.slice();
  if (zone === 'home') return chars.slice(0, 2);
  if (zone === 'park') return chars.slice().reverse();
  if (zone === 'plaza') return chars.slice(1).concat(chars[0]);
  if (zone === 'shop') return chars.slice(0, 1);
  return chars.slice();
}

function renderZone() {
  const zone = state.zone || 'home';
  const people = getZonePeople(zone);

  elements.townZone.innerHTML = `
    <div class="zone-card">
      <div class="zone-identity">
        <div>
          <p class="eyebrow">Current area</p>
          <h3>${zoneLabels[zone]}</h3>
        </div>
        <span class="character-personality">${people.length} residents</span>
      </div>
      <div class="zone-banner"></div>
      <div class="zone-icons">
        ${people.map((person) => `<div class="zone-people" style="background:${person.color}55;">${person.icon}</div>`).join('')}
      </div>
    </div>
  `;
}

function renderCharacterList() {
  elements.characterList.innerHTML = '';

  state.characters.forEach((character) => {
    const card = document.createElement('div');
    card.className = `character-card ${character.id === state.selectedId ? 'active' : ''}`;
    card.innerHTML = `
      <div class="character-avatar" style="background: linear-gradient(180deg, ${character.color}90, white);">
        <span>${character.icon}</span>
      </div>
      <div class="character-card-info">
        <div class="character-header">
          <h3 class="character-name">${character.name}</h3>
          <span class="character-age">Age ${character.age}</span>
        </div>
        <span class="character-personality">${character.personality}</span>
        <div class="character-stats">
          <div class="bar">
            <div class="bar-label"><span>Happy</span><span>${character.happiness}%</span></div>
            <div class="bar-track"><div class="bar-fill" style="width:${character.happiness}%"></div></div>
          </div>
          <div class="bar">
            <div class="bar-label"><span>Energy</span><span>${character.energy}%</span></div>
            <div class="bar-track"><div class="bar-fill" style="width:${character.energy}%"></div></div>
          </div>
        </div>
      </div>
    `;

    card.addEventListener('click', () => {
      state.selectedId = character.id;
      render();
    });

    elements.characterList.appendChild(card);
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
  const character = getSelectedCharacter();
  if (!character) {
    elements.selectedCharacterPanel.innerHTML = '<p class="eyebrow">Selected resident</p><h2>No residents</h2>';
    return;
  }

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
      <div class="detail-item"><strong>Favorite hobby:</strong> ${character.hobby}</div>
      <div class="detail-item"><strong>Favorite snack:</strong> ${character.favoriteFood}</div>
      <div class="detail-item"><strong>Relationship:</strong> ${character.relationship} hearts</div>
    </div>

    <div class="character-stats">
      ${renderStatBar('Mood', character.mood)}
      ${renderStatBar('Hunger', character.hunger)}
      ${renderStatBar('Energy', character.energy)}
      ${renderStatBar('Social', character.social)}
      ${renderStatBar('Health', character.health)}
      ${renderStatBar('Happiness', character.happiness)}
    </div>

    <div class="action-grid">
      <button class="action-btn primary" data-action="feed">Feed</button>
      <button class="action-btn" data-action="play">Play</button>
      <button class="action-btn" data-action="rest">Rest</button>
      <button class="action-btn" data-action="talk">Talk</button>
      <button class="action-btn" data-action="celebrate">Celebrate</button>
      <button class="action-btn" data-action="mini">Mini-game</button>
    </div>

    <div>
      <h3>Town log</h3>
      <div class="log-list">
        ${state.log.map((entry) => `<div class="log-entry">${entry}</div>`).join('')}
      </div>
    </div>
  `;

  elements.selectedCharacterPanel.querySelectorAll('[data-action]').forEach((button) => {
    button.addEventListener('click', () => applyAction(button.dataset.action));
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
    icon: ['🙂', '😊', '😄', '😎', '😁'][randomInt(0, 4)]
  };

  state.characters.push(newCharacter);
  state.selectedId = newCharacter.id;
  addLog(`${newCharacter.name} moved into Sunset Bay.`);
  saveState();
}

function applyAction(action) {
  const character = getSelectedCharacter();
  if (!character) return;

  switch (action) {
    case 'feed':
      character.hunger = clamp(character.hunger + 18);
      character.happiness = clamp(character.happiness + 12);
      character.health = clamp(character.health + 8);
      addLog(`${character.name} enjoyed a ${character.favoriteFood}.`);
      break;
    case 'play':
      character.happiness = clamp(character.happiness + 16);
      character.social = clamp(character.social + 14);
      character.energy = clamp(character.energy - 8);
      addLog(`${character.name} had a fun ${character.hobby} session.`);
      break;
    case 'rest':
      character.energy = clamp(character.energy + 18);
      character.mood = clamp(character.mood + 10);
      addLog(`${character.name} took a break and feels refreshed.`);
      break;
    case 'talk':
      character.social = clamp(character.social + 18);
      character.relationship = clamp(character.relationship + 1);
      character.happiness = clamp(character.happiness + 10);
      addLog(`${character.name} caught up with everyone in town.`);
      break;
    case 'celebrate':
      character.happiness = clamp(character.happiness + 20);
      character.mood = clamp(character.mood + 16);
      state.money += 25;
      addLog(`${character.name} threw a small celebration party.`);
      break;
    case 'mini':
      openMiniGame();
      return;
    default:
      break;
  }

  character.mood = clamp(Math.round((character.happiness + character.social + character.health) / 3));
  saveState();
  render();
}

function openMiniGame() {
  const overlay = document.querySelector('#miniGameTemplate').content.firstElementChild.cloneNode(true);
  document.body.appendChild(overlay);

  const buttons = overlay.querySelectorAll('[data-choice]');
  buttons.forEach((button) => {
    button.addEventListener('click', () => {
      const choice = Number(button.dataset.choice);
      const char = getSelectedCharacter();
      const rewards = {
        1: { mood: 14, social: 10, happiness: 12 },
        2: { mood: 12, happiness: 16 },
        3: { mood: 10, hunger: 8, happiness: 10 }
      };

      const result = rewards[choice] || rewards[1];
      Object.entries(result).forEach(([key, value]) => {
        char[key] = clamp((char[key] || 0) + value);
      });
      char.mood = clamp(Math.round((char.happiness + char.social + char.health) / 3));
      addLog(`${char.name} loved that plan and felt really cared for.`);
      saveState();
      render();
      overlay.remove();
    });
  });

  overlay.querySelector('.close-mini-game').addEventListener('click', () => overlay.remove());
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
      addLog(`${character.name} needs a little extra attention today.`);
    } else {
      character.relationship = clamp(character.relationship + randomInt(0, 2), 0, 100);
    }
  });

  addLog(`Day ${state.day} begins in Sunset Bay.`);
  saveState();
  render();
}

function resetTown() {
  const confirmed = window.confirm('Reset all town progress?');
  if (!confirmed) return;

  localStorage.removeItem(STORAGE_KEY);
  Object.assign(state, structuredClone(defaultState));
  render();
  addLog('The town has been reset.');
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
  render();
}

function updateHud() {
  elements.dayCounter.textContent = String(state.day);
  elements.moneyCounter.textContent = `$${state.money}`;
}

function render() {
  renderZone();
  renderCharacterList();
  renderSelectedCharacter();
  updateHud();
}

document.querySelector('#characterForm').addEventListener('submit', handleFormSubmit);
document.querySelector('#nextDayBtn').addEventListener('click', advanceDay);
document.querySelector('#saveBtn').addEventListener('click', () => {
  saveState();
  addLog('The game was saved.');
  render();
});
document.querySelector('#resetBtn').addEventListener('click', resetTown);

document.querySelectorAll('.map-tab').forEach((button) => {
  button.addEventListener('click', () => {
    state.zone = button.dataset.zone;
    document.querySelectorAll('.map-tab').forEach((tab) => tab.classList.toggle('active', tab === button));
    render();
  });
});

render();
