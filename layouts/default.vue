<template>
  <div class="site">
    <!-- <PageHeader /> -->

    <div class="site-content">
      <nuxt />
    </div>

    <!-- <PageFooter /> -->
  </div>
</template>

<style lang="scss">
.site {
  display: flex;
  min-height: 100vh;
  flex-direction: column;
}

.site-content {
  flex: 1;
}
</style>
