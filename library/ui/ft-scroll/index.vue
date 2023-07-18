<template>
  <el-scrollbar class="ft-scroll">
    <slot></slot>
  </el-scrollbar>
</template>

<script>
export default {
  name: "FtScroll"
};
</script>

<style lang="scss">
.ft-scroll {
  flex: 1;
  > .el-scrollbar__wrap {
    overflow-x: hidden;
  }
}
</style>