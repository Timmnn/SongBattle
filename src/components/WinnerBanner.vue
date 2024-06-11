<script lang="ts" setup>
import { ref, onMounted } from "vue";
const hidden = ref(false);
import WinSound from "/win.mp3";

const props = defineProps<{
   winner: string;
}>();

const audio = new Audio(WinSound);

onMounted(() => {
   hidden.value = false;
   audio.volume = 0.3;
   audio.play();
});

function onClosed() {
   hidden.value = true;
   audio.pause();
   window.location.reload();
}
</script>

<template>
   <div class="backdrop" @click="onClosed" v-if="!hidden">
      <div class="winner">
         <div class="text">{{ props.winner }} wins!</div>
      </div>
   </div>
</template>

<style scoped lang="scss">
.backdrop {
   position: fixed;
   top: 0;
   left: 0;
   width: 100%;
   height: 100%;
   background-color: rgba(0, 0, 0, 0.5);
   display: flex;
   justify-content: center;
   align-items: center;
   z-index: 100;
}

.winner {
   background-color: rgb(27, 27, 46);
   padding: 1rem;
   border-radius: 5px;
   box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
}

.text {
   font-size: 2rem;
   font-weight: bold;
   color: white;
}
</style>
