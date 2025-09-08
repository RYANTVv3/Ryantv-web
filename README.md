<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>RYANTV Streaming</title>
  <style>
    body {
      margin: 0;
      background: #0f0f0f;
      color: #fff;
      font-family: 'Segoe UI', sans-serif;
    }
    header {
      background: #1c1c1c;
      padding: 20px;
      text-align: center;
      font-size: 2em;
      color: #00ffe7;
      box-shadow: 0 2px 5px rgba(0,0,0,0.5);
    }
    #search {
      display: block;
      margin: 20px auto;
      padding: 10px;
      width: 80%;
      max-width: 400px;
      font-size: 16px;
      border-radius: 5px;
      border: 2px solid #00ffe7;
      background: #1f1f1f;
      color: #00ffe7;
    }
    .categories {
      text-align: center;
      margin-bottom: 20px;
    }
    .categories button {
      margin: 5px;
      padding: 8px 16px;
      background: #1f1f1f;
      border: 2px solid #00ffe7;
      color: #00ffe7;
      border-radius: 5px;
      cursor: pointer;
    }
    .categories button.active {
      background: #00ffe7;
      color: #000;
    }
    .channel-grid {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(180px, 1fr));
      gap: 20px;
      padding: 20px;
    }
    .channel-card {
      background: #1f1f1f;
      border: 2px solid #00ffe7;
      border-radius: 10px;
      overflow: hidden;
      cursor: pointer;
      transition: transform 0.2s;
    }
    .channel-card:hover {
      transform: scale(1.05);
    }
    .channel-card img {
      width: 100%;
      height: 120px;
      object-fit: contain;
      background: #000;
    }
    .channel-name {
      padding: 10px;
      font-size: 1em;
      text-align: center;
      color: #00ffe7;
    }
    video {
      display: block;
      margin: 20px auto;
      width: 90%;
      max-width: 900px;
      border: 3px solid #00ffe7;
      border-radius: 10px;
    }
  </style>
</head>
<body>
  <header>🎥 RYANTV Streaming</header>
  <video id="video" controls autoplay></video>
  <input type="text" id="search" placeholder="🔍 Hanapin ang channel...">
  <div class="categories" id="categoryButtons"></div>
  <div class="channel-grid" id="channelGrid">Loading channels...</div>

  <script src="https://cdn.jsdelivr.net/npm/hls.js@latest"></script>
  <script>
    const video = document.getElementById('video');
    const search = document.getElementById('search');
    const grid = document.getElementById('channelGrid');
    const categoryButtons = document.getElementById('categoryButtons');
    let hls;
    let allChannels = [];
    let currentCategory = 'All';

    fetch('https://raw.githubusercontent.com/RYANTVv3/Ryantv/main/RYANTVV3%20UPDATE')
      .then(res => res.text())
      .then(data => {
        const lines = data.split('\n');
        for (let i = 0; i < lines.length; i++) {
          if (lines[i].startsWith('#EXTINF')) {
            const nameMatch = lines[i].match(/,(.*)/);
            const logoMatch = lines[i].match(/tvg-logo="(.*?)"/);
            const name = nameMatch ? nameMatch[1].trim() : `Channel ${i}`;
            const logo = logoMatch ? logoMatch[1] : '';
            const url = lines[i + 1]?.trim();
            if (url && url.startsWith('http')) {
              const category = getCategory(name);
              allChannels.push({ name, logo, url, category });
            }
          }
        }
        renderCategories();
        renderChannels(allChannels);
      });

    function getCategory(name) {
      const lower = name.toLowerCase();
      if (lower.includes('news')) return 'News';
      if (lower.includes('sports')) return 'Sports';
      if (lower.includes('movie') || lower.includes('cinema')) return 'Movies';
      if (lower.includes('kids') || lower.includes('cartoon')) return 'Kids';
      return 'Others';
    }

    function renderCategories() {
      const categories = ['All', ...new Set(allChannels.map(c => c.category))];
      categoryButtons.innerHTML = '';
      categories.forEach(cat => {
        const btn = document.createElement('button');
        btn.textContent = cat;
        btn.classList.toggle('active', cat === currentCategory);
        btn.onclick = () => {
          currentCategory = cat;
          document.querySelectorAll('.categories button').forEach(b => b.classList.remove('active'));
          btn.classList.add('active');
          filterChannels();
        };
        categoryButtons.appendChild(btn);
      });
    }

    function renderChannels(channels) {
      grid.innerHTML = '';
      channels.forEach(channel => {
        const card = document.createElement('div');
        card.className = 'channel-card';
        card.innerHTML = `
          <img src="${channel.logo || 'https://via.placeholder.com/180x120?text=No+Logo'}" alt="${channel.name}">
          <div class="channel-name">${channel.name}</div>
        `;
        card.onclick = () => playChannel(channel.url);
        grid.appendChild(card);
      });
    }

    function playChannel(url) {
      if (hls) hls.destroy();
      hls = new Hls();
      hls.loadSource(url);
      hls.attachMedia(video);
    }

    function filterChannels() {
      const keyword = search.value.toLowerCase();
      const filtered = allChannels.filter(c =>
        (currentCategory === 'All' || c.category === currentCategory) &&
        c.name.toLowerCase().includes(keyword)
      );
      renderChannels(filtered);
    }

    search.addEventListener('input', filterChannels);
  </script>
</body>
</html>
