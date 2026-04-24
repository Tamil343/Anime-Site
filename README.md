# Anime-Site
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Anime Max Ultra</title>

<style>
body {
  margin: 0;
  background: #0b0b0b;
  color: white;
  font-family: Arial;
}

header {
  display: flex;
  justify-content: space-between;
  padding: 15px;
  background: #111;
}

.logo {
  color: #00ffc8;
  font-weight: bold;
  cursor: pointer;
}

.search {
  padding: 8px;
  border-radius: 5px;
  border: none;
}

/* GRID */
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
  gap: 15px;
  padding: 15px;
}

.card {
  background: #1a1a1a;
  border-radius: 10px;
  overflow: hidden;
  cursor: pointer;
  transition: 0.3s;
}

.card:hover { transform: scale(1.05); }

.card img {
  width: 100%;
  height: 220px;
  object-fit: cover;
}

.title {
  padding: 10px;
  font-size: 14px;
  text-align: center;
}

/* DETAILS */
.details {
  padding: 20px;
}

.details img {
  width: 180px;
  border-radius: 10px;
}

.btn {
  padding: 10px;
  background: #00ffc8;
  border: none;
  margin: 5px;
  cursor: pointer;
  border-radius: 5px;
  color: black;
  font-weight: bold;
}

.btn:hover {
  background: #00e6b0;
}

/* EPISODES */
.episodes {
  margin-top: 15px;
}

.ep-btn {
  display: inline-block;
  padding: 8px 12px;
  margin: 5px;
  background: #222;
  cursor: pointer;
  border-radius: 5px;
}

.ep-btn:hover {
  background: #00ffc8;
  color: black;
}

/* PLAYER */
.player {
  padding: 20px;
}

video {
  width: 100%;
  border-radius: 10px;
}

.error {
  color: #ff6b6b;
  padding: 20px;
  text-align: center;
}
</style>
</head>

<body>

<header>
  <div class="logo" onclick="goHome()">ANIME MAX</div>
  <input class="search" id="search" placeholder="Search...">
</header>

<div id="app"></div>

<script>
let animeData = [];
let watchlist = JSON.parse(localStorage.getItem("watchlist")) || [];
let currentAnime = null; // Track current anime for watchlist updates

/* LOAD HOME */
async function loadHome() {
  try {
    document.getElementById("search").value = ""; // Clear search box
    let res = await fetch("https://api.jikan.moe/v4/top/anime");
    
    if (!res.ok) throw new Error("Failed to load anime data");
    
    let data = await res.json();
    animeData = data.data || [];
    renderGrid(animeData);
  } catch (error) {
    document.getElementById("app").innerHTML = `<div class="error">Error loading anime: ${error.message}</div>`;
  }
}

/* GRID */
function renderGrid(list) {
  let html = `<div class="grid">`;

  list.forEach(a => {
    // Safely escape anime data
    let title = (a.title || "").replace(/"/g, "&quot;").replace(/'/g, "&#39;");
    html += `
      <div class="card" onclick='openDetails(this)' data-anime='${title}'>
        <img src="${a.images.jpg.image_url}" alt="${title}">
        <div class="title">${a.title}</div>
      </div>
    `;
  });

  html += `</div>`;
  document.getElementById("app").innerHTML = html;
}

/* DETAILS */
function openDetails(cardElement) {
  let title = cardElement.getAttribute("data-anime");
  let anime = animeData.find(a => a.title === title);
  
  if (!anime) return;

  currentAnime = anime; // Store for watchlist updates
  let saved = watchlist.includes(anime.mal_id);
  let synopsis = anime.synopsis || "No description available.";
  let genres = anime.genres?.map(g => g.name).join(", ") || "N/A";
  let episodes = anime.episodes || 12;

  let html = `
    <div class="details">
      <img src="${anime.images.jpg.image_url}" alt="${anime.title}">
      <h2>${anime.title}</h2>
      <p><strong>Genres:</strong> ${genres}</p>
      <p>${synopsis}</p>

      <button class="btn" onclick='toggleWatchlist()'>
        ${saved ? "â¤ï¸ Saved" : "ðŸ¤ Add to Watchlist"}
      </button>

      <button class="btn" onclick='goHome()'>â¬… Back</button>

      <div class="episodes">
        <h3>Episodes</h3>
  `;

  for (let i = 1; i <= Math.min(episodes, 12); i++) {
    html += `<span class="ep-btn" onclick='playEpisode(${i})'>Ep ${i}</span>`;
  }

  html += `</div></div>`;

  document.getElementById("app").innerHTML = html;
}

/* PLAYER */
function playEpisode(ep) {
  if (!currentAnime) return;
  
  let title = currentAnime.title;
  // Using a demo video - in production, you'd use actual episode URLs
  let videoUrl = "https://www.w3schools.com/html/mov_bbb.mp4";

  document.getElementById("app").innerHTML = `
    <div class="player">
      <h2>${title} - Episode ${ep}</h2>
      <video controls>
        <source src="${videoUrl}" type="video/mp4">
        Your browser does not support the video tag.
      </video>
      <br><br>
      <button class="btn" onclick='openDetails(this)'>â¬… Back to Details</button>
      <button class="btn" onclick='goHome()'>ðŸ  Home</button>
    </div>
  `;
}

/* WATCHLIST */
function toggleWatchlist() {
  if (!currentAnime) return;

  let id = currentAnime.mal_id;
  
  if (watchlist.includes(id)) {
    watchlist = watchlist.filter(t => t !== id);
  } else {
    watchlist.push(id);
  }

  localStorage.setItem("watchlist", JSON.stringify(watchlist));
  
  // Refresh details view to show updated button
  openDetails(document.querySelector("[data-anime]"));
}

/* SEARCH */
document.getElementById("search").addEventListener("input", function () {
  let value = this.value.toLowerCase();
  let filtered = animeData.filter(a =>
    a.title.toLowerCase().includes(value)
  );
  renderGrid(filtered);
});

/* INIT */
loadHome();
</script>

</body>
</html>
