<html lang="en">

<head>

<meta charset="UTF-8">

<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">

<meta name="theme-color" content="#100e15">

<title>Midnight Music ♡</title>

<style>

:root {

  --bg: #100e15;

  --panel: #1b1822;

  --text: #fffaff;

  --muted: #aaa2b5;

  --accent: #d8a5ff;

}

* { box-sizing: border-box; }

body {

  margin: 0;

  min-height: 100vh;

  padding: 24px 18px calc(32px + env(safe-area-inset-bottom));

  background: radial-gradient(ellipse at top, #392344, transparent 55%), var(--bg);

  color: var(--text);

  font-family: -apple-system, BlinkMacSystemFont, sans-serif;

}

main { max-width: 520px; margin: auto; }

header { text-align: center; margin: 15px 0 28px; }

h1 { font-size: 29px; margin: 0 0 7px; }

p, .muted { color: var(--muted); font-size: 13px; }

.panel {

  background: rgba(27,24,34,.95);

  border: 1px solid #393141;

  border-radius: 23px;

  padding: 21px;

  margin-bottom: 17px;

}

.art {

  width: 100%;

  aspect-ratio: 1;

  max-height: 300px;

  display: grid;

  place-items: center;

  border-radius: 18px;

  background: linear-gradient(140deg,#76528e,#30213e,#17141e);

  font-size: 80px;

  margin-bottom: 22px;

}

#song { font-size: 21px; font-weight: 700; overflow-wrap: anywhere; }

#artist { margin-top: 7px; color: var(--muted); font-size: 14px; }

button, .upload {

  border: 0;

  border-radius: 14px;

  padding: 13px 16px;

  background: #302a3b;

  color: var(--text);

  font-size: 14px;

  font-weight: 600;

  cursor: pointer;

}

button:active { transform: scale(.97); }

.upload {

  display: block;

  text-align: center;

  background: var(--accent);

  color: #17111f;

  padding: 16px;

  margin-bottom: 20px;

}

input[type=file] { display: none; }

input[type=range] { width: 100%; accent-color: var(--accent); }

.times {

  display: flex;

  justify-content: space-between;

  color: var(--muted);

  font-size: 12px;

}

.controls {

  display: flex;

  justify-content: center;

  align-items: center;

  gap: 17px;

  margin: 22px 0;

}

.controls button { font-size: 22px; }

.controls .play {

  width: 64px;

  height: 64px;

  border-radius: 50%;

  background: var(--accent);

  color: #17111f;

  font-size: 26px;

}

.options {

  display: flex;

  gap: 9px;

  justify-content: center;

  margin: 12px 0 20px;

}

.options button { font-size: 12px; }

.options button.active { color: var(--accent); outline: 1px solid var(--accent); }

h2 { font-size: 16px; margin: 0 0 15px; }

.track {

  display: flex;

  align-items: center;

  gap: 10px;

  padding: 12px 8px;

  border-radius: 12px;

  cursor: pointer;

}

.track.active { background: #342a43; }

.track-icon { font-size: 20px; }

.track-info { flex: 1; min-width: 0; }

.track-name { font-size: 14px; overflow-wrap: anywhere; }

.track small { color: var(--muted); }

.remove { padding: 7px 10px; background: transparent; color: var(--muted); }

#status { text-align: center; font-size: 12px; color: var(--muted); }

.notice { font-size: 12px; line-height: 1.6; color: var(--muted); }

</style>

</head>

<body>

<main>

  <header>

    <h1>Midnight Music ♡</h1>

    <p>Your songs, your little universe.</p>

  </header>

  <section class="panel">

    <div class="art" aria-hidden="true">♫</div>

    <div id="song">Choose your first song</div>

    <div id="artist">Your personal music player</div>

    <div style="margin-top:25px">

      <input id="seek" type="range" min="0" max="1000" value="0"

        aria-label="Song progress">

      <div class="times">

        <span id="elapsed">0:00</span>

        <span id="duration">0:00</span>

      </div>

    </div>

    <div class="controls">

      <button id="prev" aria-label="Previous song">⏮</button>

      <button id="play" class="play" aria-label="Play">▶</button>

      <button id="next" aria-label="Next song">⏭</button>

    </div>

    <div class="options">

      <button id="shuffle" aria-pressed="false">🔀 Shuffle off</button>

      <button id="repeat" aria-pressed="false">🔁 Repeat off</button>

    </div>

    <label for="volume" class="muted">Volume</label>

    <input id="volume" type="range" min="0" max="1" step="0.05"

      value="1" aria-label="Volume">

    <p id="status" aria-live="polite">Ready when you are ♡</p>

  </section>

  <section class="panel">

    <label for="files" class="upload">＋ Add music from your iPhone</label>

    <input id="files" type="file" accept="audio/*,.mp3,.m4a,.aac,.wav,.ogg,.flac" multiple>

    <h2>Your playlist <span id="count" class="muted">(0)</span></h2>

    <div id="playlist">

      <p class="muted">Choose some audio files to get started.</p>

    </div>

  </section>

  <p class="notice">

    ♡ Your selected files are played locally in this browser.

    They are not uploaded to a server. Keep your original files safe.

    Browser sessions may clear your playlist when you close the page.

  </p>

</main>

<audio id="audio" preload="metadata" playsinline></audio>

<script>

(() => {

  const audio = document.getElementById("audio");

  const filesInput = document.getElementById("files");

  const playlistEl = document.getElementById("playlist");

  const songEl = document.getElementById("song");

  const artistEl = document.getElementById("artist");

  const playBtn = document.getElementById("play");

  const seek = document.getElementById("seek");

  const elapsed = document.getElementById("elapsed");

  const duration = document.getElementById("duration");

  const count = document.getElementById("count");

  const status = document.getElementById("status");

  const shuffleBtn = document.getElementById("shuffle");

  const repeatBtn = document.getElementById("repeat");

  let tracks = [];

  let current = -1;

  let shuffle = false;

  let repeat = false;

  let seeking = false;

  let sessionId = 0;

  function formatTime(seconds) {

    if (!Number.isFinite(seconds)) return "0:00";

    const m = Math.floor(seconds / 60);

    const s = Math.floor(seconds % 60);

    return `${m}:${String(s).padStart(2, "0")}`;

  }

  function cleanName(name) {

    return name.replace(/\.[^/.]+$/, "").replace(/[_-]+/g, " ");

  }

  function updateMediaSession() {

    if (!("mediaSession" in navigator) || current < 0) return;

    const track = tracks[current];

    navigator.mediaSession.metadata = new MediaMetadata({

      title: track.title,

      artist: "My Music",

      album: "Midnight Music"

    });

    navigator.mediaSession.setActionHandler("play", () => {

      audio.play().catch(() => {

        status.textContent = "Tap play to resume.";

      });

    });

    navigator.mediaSession.setActionHandler("pause", () => audio.pause());

    navigator.mediaSession.setActionHandler("previoustrack", previous);

    navigator.mediaSession.setActionHandler("nexttrack", next);

    navigator.mediaSession.setActionHandler("seekbackward", details => {

      audio.currentTime = Math.max(

        0, audio.currentTime - (details.seekOffset || 10)

      );

    });

    navigator.mediaSession.setActionHandler("seekforward", details => {

      audio.currentTime = Math.min(

        audio.duration || Infinity,

        audio.currentTime + (details.seekOffset || 10)

      );

    });

    navigator.mediaSession.setActionHandler("seekto", details => {

      if (Number.isFinite(details.seekTime)) {

        audio.currentTime = details.seekTime;

      }

    });

  }

  function renderPlaylist() {

    count.textContent = `(${tracks.length})`;

    playlistEl.replaceChildren();

    if (!tracks.length) {

      const p = document.createElement("p");

      p.className = "muted";

      p.textContent = "Choose some audio files to get started.";

      playlistEl.append(p);

      return;

    }

    tracks.forEach((track, index) => {

      const row = document.createElement("div");

      row.className = "track" + (index === current ? " active" : "");

      const icon = document.createElement("span");

      icon.className = "track-icon";

      icon.textContent = index === current && !audio.paused ? "♫" : "♪";

      const info = document.createElement("div");

      info.className = "track-info";

      const name = document.createElement("div");

      name.className = "track-name";

      name.textContent = track.title;

      const sub = document.createElement("small");

      sub.textContent = index === current && !audio.paused

        ? "Now playing"

        : "Audio file";

      info.append(name, sub);

      const remove = document.createElement("button");

      remove.className = "remove";

      remove.textContent = "✕";

      remove.setAttribute("aria-label", "Remove " + track.title);

      remove.addEventListener("click", event => {

        event.stopPropagation();

        removeTrack(index);

      });

      row.append(icon, info, remove);

      row.addEventListener("click", () => loadTrack(index, true));

      playlistEl.append(row);

    });

  }

  function loadTrack(index, autoplay = false) {

    if (index < 0 || index >= tracks.length) return;

    current = index;

    sessionId++;

    audio.src = tracks[index].url;

    songEl.textContent = tracks[index].title;

    artistEl.textContent = "Your personal playlist";

    seek.value = 0;

    elapsed.textContent = "0:00";

    duration.textContent = "0:00";

    playBtn.textContent = "▶";

    status.textContent = "Loading song…";

    updateMediaSession();

    renderPlaylist();

    if (autoplay) {

      audio.play().then(() => {

        status.textContent = "Now playing ♡";

      }).catch(() => {

        status.textContent = "Tap play to start this song.";

      });

    }

  }

  async function togglePlay() {

    if (!tracks.length) {

      status.textContent = "Add some music first ♡";

      filesInput.click();

      return;

    }

    if (audio.paused) {

      try {

        await audio.play();

      } catch {

        status.textContent = "Unable to play this file. Try another format.";

      }

    } else {

      audio.pause();

    }

  }

  function next() {

    if (!tracks.length) return;

    if (shuffle && tracks.length > 1) {

      let index;

      do {

        index = Math.floor(Math.random() * tracks.length);

      } while (index === current);

      loadTrack(index, true);

    } else if (current < tracks.length - 1) {

      loadTrack(current + 1, true);

    } else if (repeat) {

      loadTrack(0, true);

    } else {

      audio.pause();

      audio.currentTime = 0;

    }

  }

  function previous() {

    if (!tracks.length) return;

    if (audio.currentTime > 3) {

      audio.currentTime = 0;

    } else {

      const index = current > 0 ? current - 1 : tracks.length - 1;

      loadTrack(index, true);

    }

  }

  function removeTrack(index) {

    const wasCurrent = index === current;

    const oldCurrent = current;

    if (wasCurrent) audio.pause();

    URL.revokeObjectURL(tracks[index].url);

    tracks.splice(index, 1);

    if (!tracks.length) {

      current = -1;

      audio.removeAttribute("src");

      audio.load();

      songEl.textContent = "Choose your first song";

      artistEl.textContent = "Your personal music player";

      playBtn.textContent = "▶";

      seek.value = 0;

      elapsed.textContent = duration.textContent = "0:00";

      status.textContent = "Playlist empty.";

    } else if (wasCurrent) {

      current = -1;

      loadTrack(Math.min(index, tracks.length - 1), false);

      status.textContent = "Track removed.";

    } else {

      if (index < oldCurrent) current--;

    }

    renderPlaylist();

  }

  filesInput.addEventListener("change", event => {

    const chosen = Array.from(event.target.files || []);

    const added = [];

    chosen.forEach(file => {

      if (!file.type.startsWith("audio/") &&

          !/\.(mp3|m4a|aac|wav|ogg|flac|aiff?)$/i.test(file.name)) {

        return;

      }

      const track = {

        title: cleanName(file.name),

        url: URL.createObjectURL(file)

      };

      tracks.push(track);

      added.push(tracks.length - 1);

    });

    if (added.length) {

      status.textContent = `${added.length} song(s) added ♡`;

      if (current === -1) loadTrack(added[0], false);

      else renderPlaylist();

    } else if (chosen.length) {

      status.textContent = "No supported audio files selected.";

    }

    // Allows selecting the same file again later.

    filesInput.value = "";

  });

  playBtn.addEventListener("click", togglePlay);

  document.getElementById("next").addEventListener("click", next);

  document.getElementById("prev").addEventListener("click", previous);

  document.getElementById("volume").addEventListener("input", event => {

    audio.volume = Number(event.target.value);

  });

  seek.addEventListener("input", () => {

    seeking = true;

    if (Number.isFinite(audio.duration)) {

      elapsed.textContent = formatTime(

        Number(seek.value) / 1000 * audio.duration

      );

    }

  });

  seek.addEventListener("change", () => {

    if (Number.isFinite(audio.duration)) {

      audio.currentTime = Number(seek.value) / 1000 * audio.duration;

    }

    seeking = false;

  });

  shuffleBtn.addEventListener("click", () => {

    shuffle = !shuffle;

    shuffleBtn.classList.toggle("active", shuffle);

    shuffleBtn.textContent = shuffle ? "🔀 Shuffle on" : "🔀 Shuffle off";

    shuffleBtn.setAttribute("aria-pressed", String(shuffle));

  });

  repeatBtn.addEventListener("click", () => {

    repeat = !repeat;

    repeatBtn.classList.toggle("active", repeat);

    repeatBtn.textContent = repeat ? "🔁 Repeat on" : "🔁 Repeat off";

    repeatBtn.setAttribute("aria-pressed", String(repeat));

  });

  audio.addEventListener("loadedmetadata", () => {

    duration.textContent = formatTime(audio.duration);

    updateMediaSession();

  });

  audio.addEventListener("timeupdate", () => {

    if (!seeking && Number.isFinite(audio.duration) && audio.duration > 0) {

      seek.value = String(audio.currentTime / audio.duration * 1000);

      elapsed.textContent = formatTime(audio.currentTime);

    }

    if ("mediaSession" in navigator &&

        Number.isFinite(audio.duration) && audio.duration > 0 &&

        navigator.mediaSession.setPositionState) {

      try {

        navigator.mediaSession.setPositionState({

          duration: audio.duration,

          playbackRate: audio.playbackRate,

          position: audio.currentTime

        });

      } catch {}

    }

  });

  audio.addEventListener("play", () => {

    playBtn.textContent = "Ⅱ";

    playBtn.setAttribute("aria-label", "Pause");

    status.textContent = "Now playing ♡";

    if ("mediaSession" in navigator) {

      navigator.mediaSession.playbackState = "playing";

    }

    renderPlaylist();

  });

  audio.addEventListener("pause", () => {

    playBtn.textContent = "▶";

    playBtn.setAttribute("aria-label", "Play");

    if ("mediaSession" in navigator) {

      navigator.mediaSession.playbackState = "paused";

    }

    renderPlaylist();

  });

  audio.addEventListener("ended", () => {

    if (repeat) {

      audio.currentTime = 0;

      audio.play().catch(() => {});

    } else if (shuffle || current < tracks.length - 1) {

      next();

    } else {

      playBtn.textContent = "▶";

      status.textContent = "Playlist finished ♡";

    }

  });

  audio.addEventListener("error", () => {

    status.textContent = "This file could not be played. Try MP3 or M4A.";

  });

  window.addEventListener("pagehide", () => {

    // Object URLs are intentionally not revoked here: playback may continue

    // while the browser keeps this page alive in the background.

  });

  if ("mediaSession" in navigator) {

    navigator.mediaSession.playbackState = "none";

  }

  renderPlaylist();

})();

</script>

</body>

</html>
