<template>
  <main class="bg-sky-100">
    <div class="container mx-auto sm:px-12">
      <header>
        <div class="px-4">
          <div class="text-4xl font-bold mt-10 inline-block">
            Countries Catalog
          </div>
        </div>
      </header>

      <div
        class="grid grid-cols-2 md:grid-cols-3 xl:grid-cols-4 2xl:grid-cols-5 xl:gap-6 mt-8"
      >
        <!-- Country Card -->
        <div
          v-for="(country, index) in countries"
          :key="index"
          class="rounded-xl"
        >
          <div class="aspect-[1.5] p-4">
            <img :src="country.flags.png" class="w-full h-full" />
          </div>
          <div class="p-4">
            <div class="font-bold truncate" :title="country.name.official">
              {{ country.name.official }}
            </div>
            <div>{{ country.cca2 }}</div>
            <div>{{ country.cca3 }}</div>
            <button
              class="py-2.5 mt-4 text-white font-medium rounded-sm text-sm w-full bg-sky-700 hover:bg-sky-800 focus:ring-2 focus:ring-blue-300"
            >
              View Details
            </button>
          </div>
        </div>
      </div>
    </div>
  </main>
</template>

<script lang="ts" setup>
import { useAxios } from "@vueuse/integrations/useAxios";

import { axiosInstance } from "./libs/http";
import { Country } from "./interfaces/Country";

const { data: countries } = useAxios<Country[]>("/all", axiosInstance);
</script>
