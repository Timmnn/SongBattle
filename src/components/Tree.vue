<script lang="ts" setup>
const props = defineProps<{
   tree: any;
}>();
</script>

<template>
   <div class="tree" :class="{ leaf: !tree.left && !tree.right }">
      <div class="children" v-if="tree.left || tree.right">
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
   border: 2px solid rgb(14, 24, 29);
   border-radius: 5px;
}

.children {
   display: flex;
   flex-direction: column;
   justify-content: space-between;
   flex-grow: 1;
   gap: 1rem;
}

.left,
.right {
   display: flex;
   flex-direction: row;
   align-items: center;
}

.node {
   flex-grow: 1;
   display: flex;
   flex-direction: column;
   align-items: stretch;
   justify-content: center;
   .name {
      background-color: rgb(14, 24, 29);
      height: 60px;
      width: 150px;
      display: flex;
      justify-content: center;
      align-items: center;
   }
}
</style>
