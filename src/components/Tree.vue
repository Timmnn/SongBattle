<script lang="ts" setup>
const props = defineProps<{
   tree: any;
}>();
</script>

<template>
   <div class="tree">
      <div class="children">
         <div class="left">
            <Tree v-if="tree.left" :tree="tree.left" />
         </div>
         <div class="right">
            <Tree v-if="tree.right" :tree="tree.right" />
         </div>
      </div>
      <div class="node">
         <div class="name">
            {{ props.tree.round_name ? "?" : props.tree.name }}
         </div>
      </div>
   </div>
</template>

<style scoped lang="scss">
.tree {
   display: flex;
   flex-direction: row;
   align-items: stretch;
   width: 100%;
   gap: 1rem;
   height: fit-content;
   border: 1px solid black;
}

.children {
   display: flex;
   flex-direction: column;
   justify-content: space-between;
   flex-grow: 1;
   gap: 1rem;
}

.lines {
   display: flex;
   justify-content: center;
   align-items: center;

   .up {
      width: 1px;
      height: 1rem;
      background-color: black;
   }

   .horizontal {
      width: 1rem;
      height: 1px;
      background-color: black;
   }
}

.node {
   flex-grow: 1;
   display: flex;
   flex-direction: column;
   align-items: center;
   justify-content: center;
   .name {
      border: 1px solid black;
      background-color: rgb(14, 24, 29);
      padding: 1rem;
   }
}
</style>
