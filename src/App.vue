<script setup lang="ts">
import { ref, onMounted } from "vue";
import Tree from "./components/Tree.vue";

const playlist_url = ref("https://open.spotify.com/playlist/0IuCDOVnrPhX8KRf6kLyxn");
const songs = ref([]) as any;
const numberOfSongs = ref(0);

const game = ref(null) as any;

class Game {
   songs: any;
   game_tree: any;
   current_round: any;
   current_round_path: boolean[] = [];
   seen_songs: any = [];

   constructor(songs: any) {
      this.songs = songs;
      this.game_tree = this.buildGameTree();

      console.log(this.game_tree);

      this.current_round = this.findCurrentRound();

      console.log(this.current_round);
   }

   buildGameTree() {
      // pick n random songs

      let picked_songs = JSON.parse(JSON.stringify(songs.value));
      while (picked_songs.length > numberOfSongs.value) {
         const rand_index = Math.floor(Math.random() * picked_songs.length);
         picked_songs = picked_songs.filter((_: any, index: number) => index !== rand_index);
      }

      // shuffle the songs
      picked_songs = picked_songs.sort(() => Math.random() - 0.5);

      const depth = Math.log2(numberOfSongs.value);

      this.current_round_path = new Array(depth - 1).fill(false);

      while (picked_songs.length > 1) {
         const left = picked_songs.shift();
         const right = picked_songs.shift();

         const battle = {
            round_name: `Round ${depth - Math.log2(picked_songs.length)}`,
            left,
            right,
         };

         battle.left.parent = battle;
         battle.right.parent = battle;

         console.log(battle);

         picked_songs.push(battle);
      }

      console.log(picked_songs);

      return picked_songs[0];
   }

   setWinner(winner_id: any) {
      const winner =
         this.current_round.left.id === winner_id
            ? this.current_round.left
            : this.current_round.right;

      let parent = this.current_round.parent;

      winner.parent = winner.parent.parent;

      if (this.game_tree.left.name && this.game_tree.right.name) {
         alert("Winner: " + winner.name);
         console.log("WINNERXXX", this.game_tree);

         game.value = null;
         return;
      }

      if (parent) {
         if (parent.left === this.current_round) {
            parent.left = winner;
         } else {
            parent.right = winner;
         }
      }

      console.log("WINNER", this.game_tree);

      this.current_round = this.findCurrentRound();
   }

   calculateCurrentRoundDepth() {
      let count = 0;
      let curr = this.current_round;
      while (curr.parent) {
         count++;
         curr = curr.parent;
      }

      return count;
   }

   getRoundName() {
      const round_names = {
         "0": "Final",
         "1": "Semi-Finals",
         "2": "Quarter-Finals",
      };

      const depth = this.calculateCurrentRoundDepth();

      if (depth > 2) {
         return `Round of ${2 ** (depth + 1)}`;
      }

      return round_names[depth.toString() as "0" | "1" | "2"];
   }

   findCurrentRound() {
      let current_round = this.game_tree;

      console.log("PATH", JSON.stringify(this.current_round_path));

      //walk down the tree
      for (let i = 0; i < this.current_round_path.length; i++) {
         const direction = this.current_round_path[i];
         current_round = direction ? current_round.right : current_round.left;
      }

      // if we are at the end of the tree, shorten the path
      if (!this.current_round_path.some(path => !path) && this.current_round_path.length > 0) {
         this.current_round_path = new Array(this.current_round_path.length - 1).fill(false);
         return current_round;
      }

      // edit the path
      for (let i = this.current_round_path.length - 1; i >= 0; i--) {
         this.current_round_path[i] = !this.current_round_path[i];
         if (this.current_round_path[i]) {
            break;
         }
      }

      return current_round;
   }
}

function newGame() {
   game.value = new Game(songs.value);
}

function getAccessToken() {
   let token = localStorage.getItem("access_token") as any;
   const expires_at = localStorage.getItem("expires_at");
   if (expires_at && Date.now() > parseInt(expires_at)) {
      localStorage.removeItem("access_token");
      localStorage.removeItem("expires_at");
      return authenticate();
   }
   if (token) {
      return token;
   }
   // get from hash
   const hash = window.location.hash
      .substring(1)
      .split("&")
      .reduce(function (initial, item) {
         if (item) {
            var parts = item.split("=") as any;
            // @ts-ignore
            initial[parts[0]] = decodeURIComponent(parts[1]);
         }
         return initial;
      }, {}) as any;
   token = hash.access_token;
   if (token) {
      localStorage.setItem("access_token", token);
      localStorage.setItem("expires_at", (Date.now() + 3600 * 1000).toString());
      return token;
   }

   if (
      !confirm(
         "Du musst dich bei Spotify anmelden, um eine Playlist zu laden. (Premium Account benötigt)"
      )
   ) {
      return;
   }

   return authenticate();
}

function onPlayerRightClick(e: MouseEvent) {
   console.log("right click");
   const target = e.target as HTMLIFrameElement;
   target.setAttribute("src", target.getAttribute("src") as string);
}

function authenticate() {
   window.location.href = `https://accounts.spotify.com/authorize?client_id=f30887f7bc6749b99807c2540d408a3c&response_type=token&redirect_uri=${window.location.origin}`;
}

function getPlaylist() {
   // @ts-ignore
   const id = playlist_url.value.split("/").pop()!.split("?")[0];

   fetch("https://api.spotify.com/v1/playlists/" + id, {
      headers: {
         Authorization: "Bearer " + getAccessToken(),
      },
   })
      .then(response => response.json())
      .then(data => {
         console.log(data);
         if (data.error?.status === 401) {
            localStorage.removeItem("access_token");
            window.location.reload();
         }
         songs.value = data.tracks.items.map((item: any) => {
            return {
               name: item.track.name,
               artist: item.track.artists[0].name,
               album: item.track.album.name,
               image: item.track.album.images[0].url,
               id: item.track.id,
            };
         });
      });
}

function generatePowersOf2(inputArray: any) {
   const n = inputArray.length;
   const powersOf2 = [];
   let power = 2;
   while (power < n) {
      powersOf2.push(power);
      power *= 2;
   }
   return powersOf2;
}

function exportSongs() {
   const json = JSON.stringify(songs.value);
   const blob = new Blob([json], { type: "application/json" });
   const url = URL.createObjectURL(blob);
   const a = document.createElement("a");
   a.href = url;
   a.download = "songs.json";
   a.click();
}

function importSongs() {
   const input = document.createElement("input");
   input.type = "file";
   input.accept = ".json";
   input.onchange = (e: any) => {
      const file = e.target.files[0];
      const reader = new FileReader();
      reader.onload = (e: any) => {
         songs.value = JSON.parse(e.target.result);
      };
      reader.readAsText(file);
   };
   input.click();
}
</script>

<template>
   <div class="load-playlist">
      <h1>Load Playlist</h1>
      <label>URL: <input type="text" v-model="playlist_url" class="playlist-url-input" /></label>
      <button @click="getPlaylist">Load</button>
   </div>
   <div class="songs">
      <button @click="exportSongs" v-if="songs.length">Export Song List</button>
      <button @click="importSongs">Import Song List</button>

      <div v-for="song in songs" class="song">
         <a :href="`https://open.spotify.com/track/${song.id}`" target="_blank">
            <img :src="song.image" />
            <div>
               <h5>{{ song.name }}</h5>
               <p>{{ song.artist }}</p>
            </div>
         </a>
      </div>
   </div>

   <div class="new-game" v-if="songs.length">
      <h1>New Game</h1>
      <label>
         Anzahl der Songs:
         <select v-model="numberOfSongs">
            <option v-for="n in generatePowersOf2(songs)" :key="n" :value="n">
               {{ n }}
            </option>
         </select>
      </label>

      <button @click="newGame">Start</button>
   </div>

   <div class="game-tree" v-if="game">
      <h1>Game Tree</h1>
      <Tree :tree="game.game_tree" class="tree" />
   </div>

   <div class="current-round" v-if="game?.current_round">
      <h1>Current Round</h1>

      <h2>
         {{ game.getRoundName() }}
      </h2>

      <div class="battle">
         <div class="left">
            <iframe
               @contextmenu="onPlayerRightClick"
               style="border-radius: 12px"
               :src="`https://open.spotify.com/embed/track/${game.current_round.left.id}?utm_source=generator&theme=0`"
               width="100%"
               height="152"
               frameBorder="0"
               allow="autoplay; clipboard-write; encrypted-media; fullscreen; picture-in-picture"
               loading="lazy"></iframe>
         </div>
         <div class="right">
            <iframe
               @contextmenu="onPlayerRightClick"
               style="border-radius: 12px"
               :src="`https://open.spotify.com/embed/track/${game.current_round.right.id}?utm_source=generator&theme=0`"
               width="100%"
               height="152"
               frameBorder="0"
               allow="autoplay; clipboard-write; encrypted-media; fullscreen; picture-in-picture"
               loading="lazy"></iframe>
         </div>
      </div>
      <div class="options">
         <button @click="game.setWinner(game.current_round.left.id)">Left</button>
         <button @click="game.setWinner(game.current_round.right.id)">Right</button>
      </div>
   </div>
</template>

<style scoped>
.playlist-url-input {
   width: 500px;
   height: 2rem;
   margin-right: 1rem;
}

.battle {
   display: flex;
   gap: 1rem;
   width: 100%;
}
.left,
.right {
   flex-grow: 1;
}
.songs {
   display: grid;
   grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
   gap: 1rem;
   width: 1000px;
   border-radius: 12px;

   .song {
      border: 1px solid #ccc;
      border-radius: 12px;
      padding: 1rem;
      max-height: 280px;
      a {
         text-decoration: none;
         color: initial;
      }
   }

   img {
      border-radius: 12px;
      width: 100%;
      aspect-ratio: 1;
   }
}

.tree {
   width: 500px;
}

.current-round {
   display: flex;
   flex-direction: column;
   align-items: center;
   width: 100%;

   img {
      border-radius: 12px;
      width: 100px;
      aspect-ratio: 1;
   }

   button {
      margin: 1rem;
      padding: 1rem;
      border-radius: 12px;
      border: none;
      background-color: #007bff;
      color: white;
   }
}

h5,
p {
   margin: 0;
}

.options {
   display: flex;
   width: 100%;
   gap: 1rem;

   button {
      flex-grow: 1;
   }
}
</style>
