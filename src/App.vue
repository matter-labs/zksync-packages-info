<script setup lang="ts">
// This starter template is using Vue 3 <script setup> SFCs
// Check out https://vuejs.org/api/sfc-script-setup.html#script-setup
import HelloWorld from './components/HelloWorld.vue';
import { onMounted, ref, Ref } from 'vue';

import { Axios } from 'axios';
interface Package {
  name: string;
  description: string;
  repo: string;
  version: string;
}
// const axios = new Axios({baseURL: https://registry.npmjs.org})
const axios = new Axios({});

const packagesList = [
  'zksync-web3',
  '@matterlabs/hardhat-zksync-solc',
  '@matterlabs/hardhat-zksync-deploy',
  '@matterlabs/hardhat-zksync-vyper',
];
const packagesInfo: Ref<Package[]> = ref([]);

async function retrievePackageInfo(name: string) {
  console.log(`Retrieving package info for ${name}`);
  const endpoint = `https://registry.npmjs.org/${name}`;

  const res = await axios.get(endpoint);

  console.log(res.data);
  return res.data;
}

async function loadPackagesData() {
  packagesList.forEach(async (p) => {
    const res = await retrievePackageInfo(p);

    const jsonData = JSON.parse(res);
    // @ts-ignore
    packagesInfo.value.push({
      name: jsonData.name,
      description: jsonData.description,
      version: jsonData['dist-tags'].latest,
      repo: jsonData.bugs.url,
    } as Package);
    // packagesInfo.value.push(await retrievePackageInfo(p));
  });
}

onMounted(() => {
  loadPackagesData();
});
</script>

<template>
  <div>
    <!-- <a href="https://vitejs.dev" target="_blank">
      <img src="/vite.svg" class="logo" alt="Vite logo" />
    </a> -->
    <a href="https://zksync.io/" target="_blank">
      <img src="./assets/zkSync-logo.png" class="logo vue" alt="zkSync logo" />
    </a>
    <h1>zkSync packages info</h1>

    <div v-for="p in packagesInfo" :key="p.name">
      <h2>{{ p.name }}</h2>
      <p>{{ p.description }}</p>
      <p>Latest version : v{{ p.version }}</p>
      <p><a :href="p.repo" target="_blank">repository</a></p>
      <hr />
    </div>
    <!-- {{ packagesInfo }} -->
  </div>
  <!-- <HelloWorld msg="Vite + Vue" /> -->
</template>

<style scoped>
.logo {
  height: 8em;
  padding: 1.5em;
  will-change: filter;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
