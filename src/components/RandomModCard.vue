<script setup>
import { onMounted } from "vue";

  const props = defineProps(["isRandomProposition", "mod", "isLoading", "modUrl"])

  const emit = defineEmits(['add', 'reroll'])

  function skipMod () {
    emit('reroll');
  }

  function addMod () {
    emit('add');
  }

  onMounted(() => {
    emit('reroll');
  })
</script>

<template>
  <v-card
      class="rounded-xl"
      :disabled="props.isLoading"
      :loading="props.isLoading"
  >
    <v-container class="d-flex justify-center align-center">
      <v-img
        width="150"
        aspect-ratio="1/1"
        :src='props.mod.icon_url'
        class="ma-1 rounded-lg"
      />
      <v-container >
        <v-row>
          <v-col>
            <v-card-title class="justify-start">{{ props.mod.title }}</v-card-title>
            <v-card-subtitle :disabled="props.isLoading" class="justify-start">{{ props.mod.project_type.toUpperCase() }}</v-card-subtitle>
            <v-card-text class="text-justify">{{ props.mod.description }}</v-card-text>
            <v-row class="align-center justify-space-around">
              <div class="d-flex flex-row align-center">
                <v-icon icon="mdi-download"/>
                <div>{{ Intl.NumberFormat().format(props.mod.downloads) }}</div>
              </div>
              <v-divider vertical inset/>
              <div class="d-flex flex-row align-center">
                <!-- List of loaders -->
                <v-list
                    class="d-flex flex-row align-center"
                    :items="props.mod.loaders"
                    disabled
                />
              </div>
              <v-divider vertical inset/>
              <div class="d-flex flex-row align-center">
                <!-- List of categories -->
                <v-list
                    class="d-flex flex-row align-center"
                    :items="props.mod.categories"
                    disabled
                />
              </div>
              <v-divider vertical inset/>
              <v-tooltip text="Open modpage">
                <template v-slot:activator="{ props }">
                  <v-btn
                      icon="mdi-open-in-new"
                      variant="text"
                      :href = "isRandomProposition
                        ? modUrl
                        : `https://modrinth.com/${mod.project_type}/${mod.slug}`"
                      target="_blank"
                      rel="noopener"
                      v-bind="props"
                  />
                </template>
              </v-tooltip>
            </v-row>
          </v-col>
        </v-row>
      </v-container>
    </v-container>

    <v-card-actions v-if="isRandomProposition" class="mt-n8 align-baseline">
      <v-container class="d-flex justify-center align-center">
        <v-btn variant="elevated" base-color="red-darken-4" class="mx-4" @click="skipMod">Skip</v-btn>
        <v-btn variant="elevated" base-color="green-darken-4" class="mx-4" @click="addMod">Add</v-btn>
      </v-container>
    </v-card-actions>
  </v-card>
</template>