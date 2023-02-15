<script setup>
import HelloWorld from './components/HelloWorld.vue'
import QueryChainbase from './components/QueryChainbase.vue';
</script>

<template>
  <div>
    <a href="https://chainbase.com/" target="_blank">
      <img src="/chainbase.svg" class="logo chainbase" alt="Chainbase logo" />
    </a>
    <a href="https://next.id/" target="_blank">
      <img src="./assets/nextid.svg" class="logo nextid" alt="NextIDs logo" />
    </a>
  </div>
  <QueryChainbase msg="Chainbase <> NextID" />
  <!-- <HelloWorld msg="Vite + Vue"/> -->
  <p>
    Check out
    <a href="https://chainbase.com/" target="_blank">chainbase.com</a>,
    a leading Web3 blockchain interaction layer infrastructure.
  </p>
  <p>
    Check out
    <a href="https://next.id/" target="_blank">Next.ID</a>,
    an open-sourced protocol that synergises your Web2 and Web3 profiles.
  </p>
  <p class="read-the-docs">Click on the Chainbase and NextID logos to learn more.</p>

</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}

.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}

.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
.read-the-docs {
    color: #888;
}

</style>
