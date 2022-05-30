<template>
    <RouterView :key="this.$route.fullPath" />
</template>

<style>
    @import '../style/reset.css';
    @import '../style/fonts.css';
    @import '../style/variables.css';
    @import '../style/style.css';
</style>