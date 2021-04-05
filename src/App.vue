<!--
-Add animation to background image slide over screen
-Animate playlist

-->
<template>
  <div id="app" :style="typeof playlist[0] != 'undefined' && typeof playlist[0].info != 'undefined' ? 'background-image: url(' + playlist[0].info.thumbnail_url + ')' : ''">
    <main :class="(typeof status != 'undefined' && !status.lastMessageRecived ? 'blur' : '') || (showPlaylist ? 'overlay-tone' : '')">
      <div class="top-bar">
        <div class="search-box" v-if="!showPlaylist">
          <div class="search">
            <input
                type="text"
                class="search-bar"
                placeholder="Search..."
                @keypress="fetchSearch"
                v-model="query"
            />
            <button @click="fetchSearch({key: 'Enter'})"></button>
          </div>
        </div>
      </div>
      <div class="controls-container">
        <div class="top">
          <div class="song-title-container" :class="typeof playlist[0] != 'undefined' && typeof playlist[0].info != 'undefined' && playlist[0].info.song.length > 20 ? 'marquee' : ''">
              <span class="song-title" v-if="typeof playlist[0] != 'undefined' && typeof playlist[0].info != 'undefined'">
                {{ playlist[0].info.song }}
              </span>
              <span class="song-title" v-else>Not playing</span>
          </div>
          <div class="playlist-button" @click="showPlaylist = !showPlaylist">
            <div class="button"></div>
          </div>
        </div>
        <div class="controls">
          <div class="button shuffle" @click="fetchAction('shuffle')"></div>
          <div class="button clear" @click="fetchAction('clear')"></div>
          <div class="button play" :class="typeof status != 'undefined' && !status.paused ? 'pause' : ''" @click="fetchAction('playtoggle')"></div>
          <div class="button next" @click="fetchAction('skip')"></div>
          <div class="button loop" :class="typeof status != 'undefined' && status.loop ? 'green-filter' : ''" @click="fetchAction('loop')"></div>
        </div>
      </div>
    </main>
    <div class="list-container" v-if="showPlaylist">
      <div class="preset-container">
        <div class="title">
          <h2>Presets:</h2>
          <div class="hr"></div>
        </div>
        <div class="presets">
          <button class="text-button" @click="fetchPreset(1)">Klassiker</button>
          <button class="text-button" @click="fetchPreset(2)">Zombies</button>
          <button class="text-button" @click="fetchPreset(3)">Radio</button>
        </div>
      </div>
      <div class="title">
        <h2>Playlist:</h2>
        <div class="hr"></div>
      </div>
      <ul id="v-for-object" class="list-queue">
        <li class="list-queue-item" v-for="(value,index) in playlist" :key="index">
          <p @click="songActionPopup(index)" v-if="typeof value.info != 'undefined'">{{ value.info.song }}</p>
          <p v-else>Loading...</p>
        </li>
      </ul>
    </div>
    <div class="connection-alert" v-if="typeof status != 'undefined' && !status.lastMessageRecived">
      <h1>Bot Status:</h1>
      <h1 class="text-danger">Not connected!</h1>
      <h3>Use ?join to start playing!</h3>
    </div>
    <div class="song-action" v-if="songAction != null" @click="() => {this.songAction = null; this.showPlaylist = true}">
      <div class="title">
        <p>Song: {{playlist[songAction].info.song}}</p>
      </div>
      <div class="song-action-buttons">
        <button class="text-button" @click="fetchSpecific(songAction)">Play now</button>
        <button class="text-button" @click="fetchNextInQueue(songAction)">Next in queue</button>
        <button class="text-button" @click="fetchDelete(songAction)">Delete</button>
        <button class="text-button" @click="() => {this.songAction = null; this.showPlaylist = true}">Return</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "app",
  data() {
    return {
      //DOMAIN here vvv
      url_base: "",
      playlist: [],
      status: [],
      query: "",
      polling: null,
      showPlaylist: false,
      songAction: null,
    };
  },
  methods: {
    fetchApi(e) {
      if (e.key == "Enter") {
        fetch(`${this.url_base}/api`)
            .then((res) => {
              return res.json();
            })
            .then(this.setResults);
      }
    },
    setResults(results) {
      this.playlist = results.queue;
    },
    fetchStatus() {
      fetch(`${this.url_base}/status`)
          .then((res) => {
            return res.json();
          })
          .then(this.setStatus);
    },
    setStatus(status_results) {
      this.status = status_results;
      //console.log(status_results);
    },
    pollData() {
      this.polling = setInterval(() => {
        this.fetchApi({key: "Enter"});
        this.fetchStatus();
      }, 500);
    },
    fetchAction(data) {
      fetch(`${this.url_base}/${data}`);
    },
    fetchSearch(e) {
      if (e.key == "Enter") {
        fetch(`${this.url_base}/search?q=${encodeURIComponent(this.query)}`);
        this.query = "";
      }
    },
    fetchPreset(id) {
      fetch(`${this.url_base}/preset?id=${id}`);
    },
    fetchSpecific(id) {
        fetch(`${this.url_base}/playnow?id=${id}`);
    },
    fetchNextInQueue(id) {
      fetch(`${this.url_base}/move?from=${id}&to=1`);
    },
    fetchDelete(id) {
        fetch(`${this.url_base}/remove?id=${id}`);
    },
    songActionPopup(id) {
      this.showPlaylist = false;
      this.songAction = id;
    }
  },
  created() {
    this.pollData();
  },
};
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: "montserrat", sans-serif;
}

#app {
  position: fixed;
  top: 0;
  bottom: 0;
  right: 0;
  left: 0;
  background-image: url("./assets/cold-bg.jpg");
  background-size: cover;
  background-position: bottom;
  transition: 0.4s;
  min-height: 100vh;
}

main {
  position: fixed;
  top: 0;
  bottom: 0;
  right: 0;
  left: 0;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  background-image: linear-gradient(
      to bottom,
      rgba(0, 0, 0, 0.25),
      rgba(0, 0, 0, 0.75)
  );
}

.overlay-tone {
  background-image: linear-gradient(
      to bottom,
      rgba(0, 0, 0, 0.75),
      rgba(0, 0, 0, 0.95)
  );
}

.top-bar {
  height: 15%;
}

.search-box {
  padding: 25px;
  width: 100%;
  margin-bottom: 30px;
}

.search {
  display: flex;
  width: 100%;
  font-size: 20px;
  appearance: none;
  border: none;
  outline: none;

  box-shadow: 0px 0px 8px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.5);
  border-radius: 0px 16px 0px 16px;
  transition: 0.4s;
}

.search-bar {
  font-size: 18px;
  margin: 10px 0 10px 10px;
  flex-grow: 2;
  border:none;
  background: none;
}

.search-bar::placeholder {
  color: #282828;
}

.search button {
  border:none;
  background: none;
  outline: none;
}

input:focus {
  /* removing the input focus blue box. Put this on the form if you like. */
  outline: none;
}

.search-bar:focus .search {
  box-shadow: 0px 0px 16px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.75);
  border-radius: 16px 0px 16px 0px;
}

.list-container {
  position: fixed;
  top: 0;
  right: 0;
  left: 0;
  height: 80%;
  z-index: 5;
  overflow: auto;
  padding: 16px;
}

.title {
  color: gray;
  margin-bottom: 16px;
}

.hr {
  border: 1px solid gray;
  height: 2px;
  width: 100%;
}

.presets {
  height: auto;
  display: flex;
  justify-content: space-between;
  flex-wrap: wrap;
  padding-bottom: 16px;
}

.text-button {
  border: none;
  font-size: 18px;
  box-shadow: 0px 0px 8px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.5);
  border-radius: 0px 16px 0px 16px;
  padding: 8px;
}

.list-queue {
  list-style: none;
  transition: 0.4s;
}

.list-queue-item {
  border-radius: 0px 16px 0px 16px;
  padding: 16px;
  margin-bottom: 16px;
  color: #202020;
  background-color: rgba(255, 255, 255, 0.4);
  box-shadow: 0px 0px 8px rgba(0, 0, 0, 0.25);
  transition: 0.4s;
}

.controls-container {
  height: 20%;
  color: white;
  padding: 16px;
  display: flex;
  flex-direction: column;
}

.top {
  display: flex;
  justify-content: space-between;
  margin-bottom: 16px;
  height: 50%;
}

.marquee {
  height: 50px;
  width: 100%;
  overflow: hidden;
  position: relative;
  white-space: nowrap;
}

.marquee span {
  font-size: 20px;
  position: absolute;
  width: auto;
  height: 100%;
  margin: 0;
  line-height: 50px;
  text-align: center;
  transform:translateX(100%);
  animation: example1 15s linear infinite;
}

@keyframes example1 {
  0%   {
    transform: translateX(100%);
  }
  100% {
    transform: translateX(-100%);
  }
}

.controls {
  display: flex;
  justify-content: space-between;
  height: 50%;
}

.button {
  min-height: 48px;
  min-width: 48px;
  width: 20%;
  height: 100%;
  padding: 10px;
  background-repeat: no-repeat;
  background-position: center;
}

.search button {
  height: 48px;
  width: 48px;
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
  background-image: url(./assets/search.png);
}

.playlist-button .button {
  background-image: url(./assets/playlist.png);
}

.shuffle {
  background-image: url(./assets/shuffle.png);
}

.previous {
  background-image: url(./assets/previous.png);
}

.clear {
  background-image: url(./assets/trash.png);
}

.play {
  background-image: url(./assets/play.png);
}

.pause {
  background-image: url(./assets/pause.png);
}

.next {
  background-image: url(./assets/next.png);
}

.loop {
  background-image: url(./assets/loop.png);
}
.green-filter {
  filter: invert(40%) sepia(75%) saturate(345%) hue-rotate(94deg) brightness(97%) contrast(95%);
}

.connection-alert {
  position: fixed;
  top: 0;
  bottom: 0;
  right: 0;
  left: 0;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background-color: rgba(0,0,0,0.9);
  color: white;
  z-index: 10;
}

.blur {
  backdrop-filter: blur(4px);
  filter: blur(4px);
}

.text-danger{
  color: #9f0a0a;
}

.song-action {
  position: absolute;
  top: 0;
  bottom: 0;
  right: 0;
  left: 0;
  z-index: 8;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.song-action-buttons {
  display: flex;
  flex-wrap: wrap;
}

</style>
