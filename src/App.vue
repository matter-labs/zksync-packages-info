<script setup lang="ts">
// This starter template is using Vue 3 <script setup> SFCs
// Check out https://vuejs.org/api/sfc-script-setup.html#script-setup
import { onMounted, ref, Ref } from "vue";

import { Axios } from "axios";
interface Package {
  name: string;
  description: string;
  repo: string;
  version: string;
}

interface Compiler {
  name: string;
  repo: string;

  versions: any;
}
// const axios = new Axios({baseURL: https://registry.npmjs.org})
const axios = new Axios({});

const packagesList = [
  "zksync-web3",
  "@matterlabs/zksync-contracts",
  "@matterlabs/hardhat-zksync-solc",
  "@matterlabs/hardhat-zksync-deploy",
  "@matterlabs/hardhat-zksync-vyper",
  "@matterlabs/hardhat-zksync-chai-matchers",
  "@matterlabs/hardhat-zksync-upgradable",
  "@matterlabs/hardhat-zksync-verify",
];

const compilersList = ["zksolc", "zkvyper"];
const packagesInfo: Ref<Package[]> = ref([]);
const compilersInfo: Ref<Compiler[]> = ref([]);

async function retrieveCompilerInfo(name: string): Promise<Compiler> {
  const endpoint = `https://raw.githubusercontent.com/matter-labs/${name}-bin/main/version.json`;

  const res = await axios.get(endpoint);

  console.log("compiler info :>> ", res.data);
  return {
    name: name,
    versions: JSON.parse(res.data),
    repo: `https://github.com/matter-labs/${name}-bin`,
  };
}

async function retrievePackageInfo(name: string) {
  console.log(`Retrieving package info for ${name}`);
  const endpoint = `https://registry.npmjs.org/${name}`;

  const res = await axios.get(endpoint);

  // console.log(res.data);
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
      version: jsonData["dist-tags"].latest,
      repo: jsonData.bugs?.url || `https://npmjs.org/${jsonData.name}`,
    } as Package);
    // packagesInfo.value.push(await retrievePackageInfo(p));
  });
  compilersList.forEach(async (c) => {
    const res = await retrieveCompilerInfo(c);
    compilersInfo.value.push(res);
  });
}

onMounted(() => {
  loadPackagesData();
});
</script>

<template>
  <div>
    <a href="https://zksync.io/" target="_blank">
      <img src="./assets/zkSync-logo.png" class="logo vue" alt="zkSync logo" />
    </a>
    <h1>zkSync Era developer tools</h1>
    <h3 style="margin-bottom: 2rem">
      Information about the different packages and SDKs released by MatterLabs
      to interact with zkSync Era
    </h3>
    <p style="margin-bottom: 3rem">
      Check out the
      <a href="https://v2-docs.zksync.io/api/" target="_blank"
        >SDK/Tools section of the docs</a
      >
      to learn more.
    </p>
    <hr />
    <h3>Compilers</h3>
    <div class="grid" style="margin-bottom: 2rem">
      <div v-for="c in compilersInfo" :key="c.name" class="item">
        <h2>{{ c.name }}</h2>
        <!-- <p>{{ p.description }}</p> -->
        <p>
          <b>Latest version : v{{ c.versions.latest }}</b>
        </p>
        <p>Minimal recommended version : v{{ c.versions.minVersion }}</p>

        <p><a :href="c.repo" target="_blank">repository</a></p>
      </div>
    </div>
    <hr />
    <h3>Plugins and libs</h3>

    <div class="grid">
      <div v-for="p in packagesInfo" :key="p.name" class="item">
        <h2>{{ p.name }}</h2>
        <p>{{ p.description }}</p>
        <p>Latest version : v{{ p.version }}</p>
        <p><a :href="p.repo" target="_blank">repository</a></p>
      </div>
    </div>
  </div>
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

.grid {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  width: 100%;
}
.item {
  flex: 32%;
  border: 1px solid gray;
}
</style>
