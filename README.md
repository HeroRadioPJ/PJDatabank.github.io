<html lang="de">
<head>
  <meta charset="UTF-8">
  <title>Yu-Gi-Oh! Cardpool-Viewer</title>
  <style>
    body { font-family: sans-serif; margin: 2rem; background: #f2f2f2; color: #333; }
    body.darkmode { background: #121212; color: #f2f2f2; }
    input, button, select { padding: 0.4rem; margin: 0.3rem; }
    input, select { background: #fff; border: 1px solid #ccc; }
    body.darkmode input, body.darkmode select { background: #333; color: #f2f2f2; border-color: #444; }
    .view-toggle { float: right; }
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
  </style>
</head>
<body>
  <h1>Yu-Gi-Oh! Cardpool-Viewer</h1>
  <div class="controls">
    <div>
      <label for="poolSelect">Cardpool wählen:</label>
      <select id="poolSelect"></select>
    </div>
    <div class="view-toggle">
      <button class="toggle-btn" onclick="toggleView()">Ansicht wechseln</button>
    </div>
    <div class="view-toggle">
      <button class="toggle-btn" onclick="toggleDarkMode()"><span id="darkModeIcon">🌙</span></button>
    </div>
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
    const poolSelect = document.getElementById('poolSelect');
    const filterInput = document.getElementById('filterInput');
    const cardTable = document.getElementById('cardTable');
    const gallery = document.getElementById('gallery');
    const sortBar = document.getElementById('sortBar');
    const darkModeIcon = document.getElementById('darkModeIcon');
    const cardCounter = document.getElementById('cardCounter');

    let pools = JSON.parse(localStorage.getItem('cardPools') || '{}');
    let currentPool = Object.keys(pools)[0] || '';
    let view = 'list';
    let sortKey = 'name';
    let sortAsc = true;

    function toggleDarkMode() {
      document.body.classList.toggle('darkmode');
      darkModeIcon.textContent = document.body.classList.contains('darkmode') ? '☀️' : '🌙';
    }

    function toggleView() {
      view = view === 'list' ? 'gallery' : 'list';
      render();
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

    function updateCounter() {
      const count = pools[currentPool]?.length || 0;
      cardCounter.textContent = `Karten: ${count}`;
    }

    function render() {
      const filterValue = filterInput.value.toLowerCase();
      let cards = pools[currentPool]?.filter(card =>
        card.name.toLowerCase().includes(filterValue) || card.type.toLowerCase().includes(filterValue)
      ) || [];

      const sortFuncs = {
        name: (a, b) => a.name.localeCompare(b.name),
        type: (a, b) => a.type.localeCompare(b.type),
        level: (a, b) => a.level - b.level,
        atk: (a, b) => b.atk - a.atk,
        def: (a, b) => b.def - a.def,
      };
      cards.sort((a, b) => {
        const res = sortFuncs[sortKey](a, b);
        return sortAsc ? res : -res;
      });

      sortBar.innerHTML = '';
      ['name', 'type', 'level', 'atk', 'def'].forEach(key => {
        const btn = document.createElement('button');
        btn.className = 'sort-button';
        btn.innerHTML = key.toUpperCase() + '<span class="arrow">⇅</span>';
        btn.onclick = () => {
          if (sortKey === key) sortAsc = !sortAsc;
          else { sortKey = key; sortAsc = true; }
          render();
        };
        sortBar.appendChild(btn);
      });

      if (view === 'list') {
        cardTable.style.display = 'table';
        gallery.style.display = 'none';
        const tbody = cardTable.querySelector('tbody');
        tbody.innerHTML = '';
        cards.forEach(card => {
          const tr = document.createElement('tr');
          tr.innerHTML = `
            <td><img src="${card.image}" alt="${card.name}"></td>
            <td>${card.name}</td>
            <td>${card.type}</td>
            <td>${card.level || ''}</td>
            <td>${card.atk || ''}</td>
            <td>${card.def || ''}</td>
            <td>${card.effect || ''}</td>
          `;
          tbody.appendChild(tr);
        });
      } else {
        cardTable.style.display = 'none';
        gallery.style.display = 'flex';
        gallery.innerHTML = '';
        cards.forEach(card => {
          const div = document.createElement('div');
          div.className = 'card';
          div.innerHTML = `<img src="${card.image}" alt="${card.name}"><div>${card.name}</div>`;
          gallery.appendChild(div);
        });
      }
      updateCounter();
    }

    renderSelect();
    render();
  </script>
</body>
</html>
