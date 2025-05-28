<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8">
  <title>Yu-Gi-Oh! Cardpool-Manager</title>
  <style>
    body { font-family: sans-serif; margin: 2rem; background: #f2f2f2; color: #333; }
    body.darkmode { background: #121212; color: #f2f2f2; }
    input, button, select { padding: 0.4rem; margin: 0.3rem; }
    input, select { background: #fff; border: 1px solid #ccc; }
    body.darkmode input, body.darkmode select { background: #333; color: #f2f2f2; border-color: #444; }
    .view-toggle { float: right; }
    .sort-header { cursor: pointer; user-select: none; }
    .sort-header::after { content: ' ⇅'; font-size: 0.8em; }
    table, .gallery { width: 100%; border-collapse: collapse; margin-top: 1rem; }
    th, td, .card { border: 1px solid #ccc; padding: 0.5rem; text-align: center; background: #fff; }
    body.darkmode table th, body.darkmode table td, body.darkmode .card { background: #333; color: #f2f2f2; }
    img { height: 80px; }
    body.darkmode img { filter: brightness(0.8); }
    .gallery { display: flex; flex-wrap: wrap; gap: 1rem; justify-content: center; }
    .card { width: 140px; cursor: pointer; transition: transform 0.3s ease; }
    .card img { width: 100%; height: auto; }
    .card:hover { transform: scale(1.5); }
    .controls { display: flex; justify-content: space-between; align-items: center; }
    .toggle-btn { background: transparent; border: none; cursor: pointer; color: inherit; font-size: 1.5rem; position: relative; }
    body.darkmode .toggle-btn { color: #f2f2f2; }
    .toggle-btn span { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); transition: all 0.3s; }
    #cardCounter {
      position: fixed;
      bottom: 10px;
      right: 10px;
      background: #f2f2f2;
      color: #121212;
      padding: 0.5rem 1rem;
      border-radius: 8px;
      font-size: 1rem;
      z-index: 1000;
    }
    body.darkmode #cardCounter {
      background: #333;
      color: #fff;
    }
    #sortBar {
      margin-bottom: 1rem;
      display: flex;
      justify-content: space-evenly;
    }
    .sort-button {
      background: #f2f2f2;
      color: #121212;
      padding: 0.5rem 1rem;
      border-radius: 8px;
      border: 1px solid #ccc;
      display: flex;
      align-items: center;
      cursor: pointer;
      transition: background 0.3s ease;
    }
    body.darkmode .sort-button {
      background: #444;
      color: #f2f2f2;
    }
    .sort-button:hover {
      background: #e0e0e0;
    }
    .sort-button:active {
      background: #c0c0c0;
    }
    .sort-button .arrow {
      margin-left: 0.5rem;
      font-size: 0.8em;
    }
    body.darkmode .sort-button .arrow {
      color: #f2f2f2;
    }
  </style>
</head>
<body>
  <h1>Yu-Gi-Oh! Cardpool-Manager</h1>
  <div class="controls">
    <div>
      <label for="poolSelect">Cardpool wählen:</label>
      <select id="poolSelect"></select>
      <button onclick="newPool()">+ Neuer Pool</button>
    </div>
    <div class="view-toggle">
      <button class="toggle-btn" onclick="toggleView()">Ansicht wechseln</button>
    </div>
    <div class="view-toggle">
      <button class="toggle-btn" onclick="toggleDarkMode()"><span id="darkModeIcon">🌙</span></button>
    </div>
  </div>
  <div>
    <input type="text" id="cardInput" placeholder="Kartenname">
    <button onclick="addCard()">Karte hinzufügen</button>
    <button onclick="exportPool()">Exportieren</button>
    <input type="file" id="importFile" accept=".json" style="display:none" onchange="importPool(event)">
    <button onclick="document.getElementById('importFile').click()">Importieren</button>
  </div>
  <div>
    <input type="text" id="filterInput" placeholder="Filter nach Name/Typ..." oninput="render()">
  </div>
  <div id="sortBar"></div>
  <table id="cardTable" style="display:none;">
    <thead>
      <tr>
        <th>Bild</th>
        <th>Name</th>
        <th>Typ</th>
        <th>Stufe</th>
        <th>ATK</th>
        <th>DEF</th>
        <th>Effekt</th>
      </tr>
    </thead>
    <tbody></tbody>
  </table>
  <div id="gallery" class="gallery" style="display:none;"></div>
  <div id="cardCounter">Karten: 0</div>
  <script>
  const API = 'https://db.ygoprodeck.com/api/v7/cardinfo.php?name=';
  let pools = JSON.parse(localStorage.getItem('cardPools') || '{}');
  let currentPool = Object.keys(pools)[0] || 'Standard';
  let view = 'list';
  
  // Initialisierung der Sortiervariablen
  let sortKey = 'name';  // Standard-Sortierung nach Name
  let sortAsc = true;    // Aufsteigend (A-Z)

  let darkMode = false;

  if (!pools[currentPool]) pools[currentPool] = [];

  const poolSelect = document.getElementById('poolSelect');
  const cardTable = document.getElementById('cardTable');
  const gallery = document.getElementById('gallery');
  const filterInput = document.getElementById('filterInput');
  const sortBar = document.getElementById('sortBar');
  const darkModeIcon = document.getElementById('darkModeIcon');
  const cardCounter = document.getElementById('cardCounter');

  function updateCounter() {
    const count = pools[currentPool]?.length || 0;
    cardCounter.textContent = `Karten: ${count}`;
  }

  function save() {
    localStorage.setItem('cardPools', JSON.stringify(pools));
  }

  function normalizeCard(card) {
    const typeMap = {
      'Normal Monster': 1,
      'Effect Monster': 2,
      'Flip Effect Monster': 2,
      'Fusion Monster': 3,
      'Fusion Effect Monster': 3,
      'Ritual Monster': 4,
      'Ritual Effect Monster': 4,
      'Spell Card': 5,
      'Trap Card': 6,
      'Toon Monster': 2
    };
    const getSortVal = (v, t) => {
      if (t === 'Spell Card') return -1;
      if (t === 'Trap Card') return -2;
      return isNaN(v) ? 0 : Number(v);
    };
    card.sortATK = getSortVal(card.atk, card.type);
    card.sortDEF = getSortVal(card.def, card.type);
    card.sortLevel = getSortVal(card.level, card.type);
    card.sortType = typeMap[card.type] || 99;
    return card;
  }

  function renderSelect() {
    poolSelect.innerHTML = '';
    Object.keys(pools).forEach(name => {
      const opt = document.createElement('option');
      opt.value = name;
      opt.textContent = name;
      if (name === currentPool) opt.selected = true;
      poolSelect.appendChild(opt);
    });
  }

  poolSelect.addEventListener('change', () => {
    currentPool = poolSelect.value;
    render();
  });

  function newPool() {
    const name = prompt('Name des neuen Cardpools?');
    if (name && !pools[name]) {
      pools[name] = [];
      currentPool = name;
      save();
      renderSelect();
      render();
    }
  }

  async function addCard() {
    const name = document.getElementById('cardInput').value.trim();
    if (!name) return;
    if (pools[currentPool].some(c => c.name.toLowerCase() === name.toLowerCase())) {
      alert('Karte bereits im Pool.');
      return;
    }
    try {
      const res = await fetch(API + encodeURIComponent(name));
      const data = await res.json();
      const card = data.data[0];
      const entry = normalizeCard({
        name: card.name,
        type: card.type,
        atk: card.atk || 0,
        def: card.def || 0,
        level: card.level || 0,
        effect: card.desc,
        image: card.card_images[0]?.image_url
      });
      pools[currentPool].push(entry);
      save();
      render();
    } catch (error) {
      alert('Karte nicht gefunden.');
    }
  }

function exportPool() {
  const dataStr = JSON.stringify(pools);
  const blob = new Blob([dataStr], { type: 'application/json' });
  const link = document.createElement('a');
  link.href = URL.createObjectURL(blob);
  link.download = `${currentPool}_Cardpool.json`;
  link.click();
}

  function render() {
    const filterValue = filterInput.value.toLowerCase();
    let cards = pools[currentPool].filter(card =>
      card.name.toLowerCase().includes(filterValue) || card.type.toLowerCase().includes(filterValue)
    );

    const sorted = cards.sort((a, b) => {
      const sortFuncs = {
        'name': (a, b) => a.name.localeCompare(b.name),
        'type': (a, b) => {
          if (a.sortType === b.sortType) return b.sortATK - a.sortATK; // If type is the same, sort by ATK
          return a.sortType - b.sortType;
        },
        'level': (a, b) => a.sortLevel - b.sortLevel,
        'atk': (a, b) => b.sortATK - a.sortATK,
        'def': (a, b) => b.sortDEF - a.sortDEF,
      };
      const res = sortFuncs[sortKey](a, b);
      return sortAsc ? res : -res;
    });

    if (view === 'list') {
      cardTable.style.display = 'table';
      gallery.style.display = 'none';
      const tbody = document.querySelector('#cardTable tbody');
      tbody.innerHTML = '';
      sorted.forEach(card => {
        const tr = document.createElement('tr');
        tr.innerHTML = `
          <td><img src="${card.image}" alt="${card.name}"></td>
          <td>${card.name}</td>
          <td>${card.type}</td>
          <td>${card.level}</td>
          <td>${card.atk}</td>
          <td>${card.def}</td>
          <td>${card.effect}</td>
        `;
        tbody.appendChild(tr);
      });
    } else {
      gallery.style.display = 'flex';
      cardTable.style.display = 'none';
      gallery.innerHTML = '';
      sorted.forEach(card => {
        const div = document.createElement('div');
        div.classList.add('card');
        div.innerHTML = `
          <img src="${card.image}" alt="${card.name}">
          <div>${card.name}</div>
        `;
        div.addEventListener('click', () => alert(card.effect));
        gallery.appendChild(div);
      });
    }
    updateCounter();
  }

  function renderSortBar() {
    sortBar.innerHTML = '';
    const sortOptions = ['name', 'type', 'level', 'atk', 'def'];
    sortOptions.forEach(option => {
      const btn = document.createElement('button');
      btn.classList.add('sort-button');
      btn.innerHTML = `${option.charAt(0).toUpperCase() + option.slice(1)}<span class="arrow">${sortAsc ? '↓' : '↑'}</span>`;
      btn.onclick = () => setSort(option);
      sortBar.appendChild(btn);
    });
  }

  function setSort(key) {
    if (sortKey === key) {
      sortAsc = !sortAsc;
    } else {
      sortKey = key;
      sortAsc = true;
    }
    render();
    renderSortBar();
  }

  function toggleDarkMode() {
    darkMode = !darkMode;
    document.body.classList.toggle('darkmode', darkMode);
    darkModeIcon.textContent = darkMode ? '🌞' : '🌙';
    save();
    render();
  }

  function toggleView() {
    view = (view === 'list') ? 'gallery' : 'list';
    render();
  }

  renderSelect();
  render();
  renderSortBar();
</script>
</body>
</html>
