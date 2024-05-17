<script lang="ts" setup>
const props = defineProps<{
   tree: any;
}>();
</script>

<template>
   <div class="tree">
      <div class="node">{{ tree.round_name || tree.name }}</div>
      <div class="children">
         <div class="left">
            <Tree v-if="tree.left" :tree="tree.left" />
         </div>
         <div class="right">
            <Tree v-if="tree.right" :tree="tree.right" />
         </div>
      </div>
   </div>
</template>

<style scoped lang="scss">
.tree {
   display: flex;
   flex-direction: column;
   align-items: center;
   border: 1px solid black;
   padding: 1rem;
   width: fit-content;
}
.children {
   display: flex;
   justify-content: space-between;
}
</style>
