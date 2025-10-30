<script setup>
import {onMounted, ref, toRaw} from 'vue'
import RandomModCard from "@/components/RandomModCard.vue";
  import axios from "axios";

const apiUrl = "https://api.modrinth.com/v2/"
  const modUrl = ref("https://www.google.com")

  const theme = ref('dark')
  function ToggleTheme () {
    theme.value = theme.value === 'light' ? 'dark' : 'light'
  }

  const loading = ref(true);
  const currentMod = ref({
    title : "Placeholder",
    description : "Placeholder",
    project_type : "Placeholder",
    downloads : -1
  })

  const items = ref([])

  //Criteria
  const minDownloads = ref(0)
  const allowedModloader = ref("forge")
  const allowedVersion = ref("1.20")
  const allowedCategories = ref([])

  // arrays for ComboBox
  const modLoaders = ref([])
  const versions = ref([])
  const categories = ref([])

  const filterModloaders = ['babric', 'bta-babric', 'bungeecord', 'canvas', 'folia', 'iris', 'java-agent', 'legacy-fabric', 'vanilla', 'liteloader', 'modloader', 'nilloader', 'optifine', 'ornithe', 'rift', 'sponge', 'velocity', 'waterfall']

  function addModToModpack() {
    const newMod = structuredClone(toRaw(currentMod.value))

    const res = items.value.every((item) => {
      return item.slug !== newMod.slug
    })

    if (res) {
      items.value.push(newMod)
    }

    getNewMod()
  }

  async function getNewMod() {
    loading.value = true

    axios.get(apiUrl + 'projects_random?count=1')
      .then( function (response) {
        if (response.data.length > 0) {
          if ( isValidMod(response.data[0]) ) {
            loading.value = false
            currentMod.value = response.data[0]
            modUrl.value = "https://modrinth.com/" + currentMod.value.project_type + "/" + currentMod.value.slug
          }
          else {
            getNewMod()
          }
        }

      })
      .catch(function (error) {
        console.log(error)
      })
  }

  function isValidMod(mod) {
    console.log(mod)

    if (!items.value.every( function (item) { return item.slug !== mod.slug })) return false
    if (mod.project_type === "modpack" || mod.project_type === "shader") return false
    if (mod.downloads <= minDownloads) return false

    if (allowedModloader.value) {
      let res = false

      console.log("checked modloaders")

      console.log(allowedModloader.value)

      mod.loaders.forEach(function (item) {
        res = res || allowedModloader.value === item
      })

      if (!res) return false
    }

    if(allowedVersion.value) {
      let res = false

      console.log("checked versions")

      console.log(allowedVersion.value)

      mod.game_versions.forEach(function (item) {
        // look if the loader is in the allowed loader list or is vanilla (so not to skip ressource pack)
        res = res || allowedVersion.value === item || item === "minecraft"
      })

      if (!res) return false
    }

    if(allowedCategories.value.length > 0) {
      let res = false

      console.log("checked categories")

      console.log(allowedCategories.value)

      allowedCategories.value.forEach((item) => {
        res = res || mod.categories.includes(item.name)
      })

      if (!res) return false
    }

    return true
  }

  onMounted(() => {
    axios.get(apiUrl + "tag/loader")
        .then( function (response) {
          modLoaders.value = response.data
          modLoaders.value = modLoaders.value.filter(item => !filterModloaders.includes(item.name))
        })
        .catch(function (error) {
          console.log(error)
        })

    axios.get(apiUrl + "tag/game_version")
        .then( function (response) {
          versions.value = response.data
          versions.value = versions.value.filter((item) => {
            return item.version_type === "release"
          })
        })
        .catch(function (error) {
          console.log(error)
        })

    axios.get(apiUrl + "tag/category")
        .then( function (response) {
          categories.value = response.data
          categories.value = categories.value.filter((item) => {
            return item.project_type === "mod"
          })

        })
        .catch(function (error) {
          console.log(error)
        })
  })
</script>

<template>
  <v-app :theme="theme">
    <v-app-bar class="px-3">
      <h1>Random Minecraft modpack generator</h1>
      <v-spacer></v-spacer>
      <!-- Theme switcher -->
      <v-btn
          :prepend-icon="theme === 'light' ? 'mdi-weather-sunny' : 'mdi-weather-night'"
          text="Toggle Theme"
          slim
          @click="ToggleTheme"
      ></v-btn>
    </v-app-bar>

    <v-main>
      <!-- Random Mod Card -->
      <v-container class="d-flex justify-center">
        <RandomModCard
            width="40%"
            height="100%"
            :mod="currentMod"
            :isRandomProposition="true"
            :isLoading="loading"
            :modUrl="modUrl"
            isDetailed="detailed"
            @add="addModToModpack"
            @reroll="getNewMod"
        />
      </v-container>
      <!-- Filters -->
      <v-container class="d-flex">
        <v-select
            class="mx-4"
            density="compact"
            variant="outlined"
            hint="Check if it is any of these Modloaders"
            label="Modloader"
            v-model="allowedModloader"
            :items="modLoaders"
            item-title="name"
            item-value="name"
            persistent-hint
        />
        <v-select
            class="mx-4"
            density="compact"
            variant="outlined"
            hint="Check if it is any of these versions"
            label="Version"
            v-model="allowedVersion"
            :items="versions"
            item-title="version"
            item-value="version"
            persistent-hint
        />
        <v-combobox
            class="mx-4"
            density="compact"
            variant="outlined"
            hint="Check if it has any of these categories"
            label="Categories"
            v-model="allowedCategories"
            :items="categories"
            item-title="name"
            item-value="name"
            persistent-hint
            closable-chips
            multiple
            chips
        />
      </v-container>
      <!-- Mod list -->
      <v-container class="overflow-y-scroll mod-list">
        <v-row>
          <v-col
              v-for="(item, index) in items"
              :key="index"
              cols="12" sm="6" md="4"
          >
            <RandomModCard
                height="100%"
                :mod="item"
                :isRandomProposition="false"
            />
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>

<style scoped>
  .mod-list {
    height: calc(100vh - 45vh);
    overflow-y: auto;
  }
</style>