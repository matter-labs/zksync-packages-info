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
  <main class="max-w-7xl mx-auto text-lg space-y-4 py-16">
    <a href="https://zksync.io/" target="_blank">
      <img
        src="./assets/era-logo.png"
        class="mx-auto h-64 py-12"
        alt="zkSync logo"
      />
    </a>
    <div class="text-center text-xl space-y-4">
      <h1 class="text-3xl md:text-5xl font-medium text-center mb-12">
        zkSync Era developer tools
      </h1>
      <h3 class="mb-8 text-center">
        Information about the different packages and SDKs by MatterLabs
        to interact with zkSync Era
      </h3>
      <p>
        Check out the
        <a
          href="https://v2-docs.zksync.io/api/"
          target="_blank"
          class="text-indigo-400 underline"
          >SDK/Tools section of the docs</a
        >
        to learn more.
      </p>
      <p><p>
        Update this page 
        <a
          href="https://github.com/matter-labs/zksync-packages-info/"
          target="_blank"
          class="text-indigo-400 underline"
          >creating a PR in GitHub</a
        >
        .
      </p></p>
    </div>
    <div class="text-center space-y-4 pt-12">
      <h3 class="text-2xl md:text-3xl font-medium text-center">Compilers</h3>
      <p><a href="https://era.zksync.io/docs/api/compiler-toolchain/overview.html" target="_blank" class="text-indigo-400 underline"> See documentation</a></p>
    </div>

    <table class="border p-4 mx-auto text-left">
      <tr class="border-b-2">
        <th class="border p-4">Package name</th>
        <th class="border p-4">Latest version</th>
        <th class="border p-4">Minimal required version</th>
        <th class="border p-4">Repository</th>
      </tr>
      <tr v-for="c in compilersInfo" :key="c.name" class="border-b">
        <td class="border p-4 font-medium">{{ c.name }}</td>
        <td class="border p-4">{{ c.versions.latest }}</td>
        <td class="border p-4">v{{ c.versions.minVersion }}</td>
        <td class="border p-4">
          <a :href="c.repo" class="text-indigo-400 underline" target="_blank"
            >repository</a
          >
        </td>
      </tr>
    </table>
    <div class="text-center space-y-4 pt-12">
      <h3 class="text-2xl md:text-3xl font-medium text-center pt-12">
      Plugins and libraries
      </h3>
      <p><a href="https://era.zksync.io/docs/api/" target="_blank" class="text-indigo-400 underline"> See documentation</a></p>
    </div>

    <table class="border p-4 mx-auto text-left">
      <tr class="border-b-2">
        <th class="border p-4">Package name</th>
        <th class="border p-4">Description</th>
        <th class="border p-4">Latest version</th>
        <th class="border p-4">Repository</th>
      </tr>
      <tr v-for="p in packagesInfo" :key="p.name" class="border-b">
        <td class="border p-4 font-medium">{{ p.name }}</td>
        <td class="border p-4">{{ p.description }}</td>
        <td class="border p-4">v{{ p.version }}</td>
        <td class="border p-4">
          <a class="text-indigo-400 underline" :href="p.repo" target="_blank"
            >repository</a
          >
        </td>
      </tr>
    </table>
  </main>
</template>

<style scoped>

</style>
