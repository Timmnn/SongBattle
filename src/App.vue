<script setup lang="ts">
import { ref } from "vue";
import Tree from "./components/Tree.vue";

const playlist_url = ref("https://open.spotify.com/playlist/7jm9uTjAE8kq7Pvk6s5qko");
const songs = ref([]) as any;
const numberOfSongs = ref(0);

const game = ref(null) as any;

class Game {
   songs: any;
   game_tree: any;
   current_round: any;

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

      const depth = Math.log2(numberOfSongs.value);

      while (picked_songs.length > 1) {
         const left = picked_songs.shift();
         const right = picked_songs.shift();

         const battle = {
            round_name: `Round ${depth - Math.log2(picked_songs.length)}`,
            left,
            right,
         };

         left.parent = battle;
         right.parent = battle;

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

      if (this.current_round.parent) {
         if (this.current_round.parent.left === this.current_round) {
            this.current_round.parent.left = winner;
         } else {
            this.current_round.parent.right = winner;
         }
      } else {
         alert("Winner: " + winner.name);
      }
      console.log(this.game_tree);

      this.current_round = this.findCurrentRound();
   }

   findCurrentRound() {
      let current_round = this.game_tree;

      console.log("AAA", current_round);
      while (true) {
         if (current_round.left.round_name) {
            current_round = current_round.left;
            continue;
         }

         if (current_round.right.round_name) {
            current_round = current_round.right;
            continue;
         }
         break;
      }

      return current_round;
   }
}

function getSpotifyAuthUrl() {
   return `https://accounts.spotify.com/authorize?client_id=f30887f7bc6749b99807c2540d408a3c&response_type=token&redirect_uri=${window.location.origin}`;
}

function newGame() {
   game.value = new Game(songs.value);
}

function getAccessToken() {
   let token = localStorage.getItem("access_token") as any;
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
      return token;
   }
   return authenticate();
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
   let power = 1;
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
   {{ getSpotifyAuthUrl() }}
   <div class="load-playlist">
      <h1>Load Playlist</h1>
      URL: <input type="text" v-model="playlist_url" />
      <button @click="getPlaylist">Load</button>
   </div>
   <div class="songs">
      <button @click="exportSongs">Export</button>
      <button @click="importSongs">Import</button>

      <div v-for="song in songs" class="song">
         <img :src="song.image" />
         <div>
            <h2>{{ song.name }}</h2>
            <p>{{ song.artist }} - {{ song.album }}</p>
         </div>
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

   <div class="game-tree">
      <h1>Game Tree</h1>
      <Tree v-if="game && false" :tree="game.game_tree" />
   </div>

   <div class="current-round" v-if="game?.current_round">
      <h1>Current Round</h1>
      <div>
         <div>
            <div class="left">
               <iframe
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
                  style="border-radius: 12px"
                  :src="`https://open.spotify.com/embed/track/${game.current_round.right.id}?utm_source=generator&theme=0`"
                  width="100%"
                  height="152"
                  frameBorder="0"
                  allow="autoplay; clipboard-write; encrypted-media; fullscreen; picture-in-picture"
                  loading="lazy"></iframe>
            </div>
         </div>
         <button @click="game.setWinner(game.current_round.left.id)">Left</button>
         <button @click="game.setWinner(game.current_round.right.id)">Right</button>
      </div>
   </div>
</template>

<style scoped>
.songs {
   display: grid;
   grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
   gap: 1rem;
   width: 1000px;
   border-radius: 12px;

   .song {
      border: 1px solid #ccc;
      border-radius: 12px;
      padding: 1rem;
   }

   img {
      border-radius: 12px;
      width: 100%;
      aspect-ratio: 1;
   }
}

.current-round {
   display: flex;
   flex-direction: column;
   align-items: center;

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
</style>
