<template>
  <main>
    <div class="container mx-auto sm:px-12">
      <header>
        <div class="px-4">
          <div class="text-4xl font-bold mt-10 inline-block">
            Countries Catalog
          </div>
          <!-- Search -->
          <div class="relative mt-8">
            <div>
              <div
                class="flex absolute inset-y-0 left-0 items-center pl-3 pointer-events-none"
              >
                <svg
                  aria-hidden="true"
                  class="w-5 h-5"
                  fill="none"
                  stroke="currentColor"
                  viewBox="0 0 24 24"
                  xmlns="http://www.w3.org/2000/svg"
                >
                  <path
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    stroke-width="2"
                    d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"
                  ></path>
                </svg>
              </div>
              <input
                id="default-search"
                v-model="searchCriteria"
                type="search"
                class="block p-4 pl-10 w-full rounded-sm border border-gray-300 focus:ring-blue-500 focus:border-blue-500"
                placeholder="Country Name"
              />
              <div class="text-red-400 mt-2.5 absolute">
                {{ error?.response?.data.message }}
              </div>
            </div>
          </div>
          <!-- Sorting -->
          <div class="mt-12">Sort by name</div>
          <div class="flex">
            <div class="flex gap-x-2 flex-wrap">
              <button
                :class="{
                  'bg-sky-700 text-white': orderNameBy === OrderBy.ASC,
                }"
                class="button-alt border-2 border-sky-700 hover:text-white"
                @click="orderCountryNameBy(OrderBy.ASC)"
              >
                Ascending
              </button>
              <button
                :class="{
                  'bg-sky-700 text-white': orderNameBy === OrderBy.DESC,
                }"
                class="button-alt border-2 border-sky-700 hover:text-white"
                @click="orderCountryNameBy(OrderBy.DESC)"
              >
                Descending
              </button>
              <button class="button" @click="reset">Reset</button>
            </div>

            <div class="hidden ml-auto md:flex gap-x-2">
              <button
                :disabled="isFirstPage"
                :class="{
                  'bg-sky-700 text-white': orderNameBy === OrderBy.ASC,
                  '!cursor-not-allowed !bg-gray-500 hover:!bg-gray-500':
                    isFirstPage,
                }"
                class="button"
                @click="prev"
              >
                Previous
              </button>
              <button
                :disabled="isLastPage"
                :class="{
                  'bg-sky-700 text-white': orderNameBy === OrderBy.DESC,
                  '!cursor-not-allowed !bg-gray-500 hover:!bg-gray-500':
                    isLastPage,
                }"
                class="button"
                @click="next"
              >
                Next
              </button>
            </div>
          </div>
        </div>
      </header>

      <!-- Country Catalog Wrapper -->
      <div
        class="grid grid-cols-2 md:grid-cols-3 xl:grid-cols-4 2xl:grid-cols-5 xl:gap-6 my-4 md:mb-12"
      >
        <!-- Country Card -->
        <div
          v-for="(country, index) in displayCountries"
          :key="index"
          class="rounded-xl"
        >
          <div
            class="aspect-[1.5] p-4 cursor-pointer"
            @click="viewCountryDetails(country)"
          >
            <img :src="country.flags.png" class="w-full h-full" />
          </div>
          <div class="p-4">
            <div class="font-bold truncate" :title="country.name.official">
              {{ country.name.official }}
            </div>
            <div>{{ country.cca2 }}</div>
            <div>{{ country.cca3 }}</div>
            <button class="button w-full" @click="viewCountryDetails(country)">
              View Details
            </button>
          </div>
        </div>
      </div>

      <!-- Mobile Pagination -->
      <div class="flex gap-x-2 justify-end mb-12 md:hidden px-4">
        <button
          :disabled="isFirstPage"
          :class="{
            'bg-sky-700 text-white': orderNameBy === OrderBy.ASC,
            '!cursor-not-allowed !bg-gray-500 hover:!bg-gray-500': isFirstPage,
          }"
          class="button"
          @click="prev"
        >
          Previous
        </button>
        <button
          :disabled="isLastPage"
          :class="{
            'bg-sky-700 text-white': orderNameBy === OrderBy.DESC,
            '!cursor-not-allowed !bg-gray-500 hover:!bg-gray-500': isLastPage,
          }"
          class="button"
          @click="next"
        >
          Next
        </button>
      </div>

      <!-- Country Dialog -->
      <TransitionRoot appear :show="isDialogOpen" as="template">
        <Dialog as="div" class="relative z-10" @close="closeModal">
          <TransitionChild
            as="template"
            enter="duration-300 ease-out"
            enter-from="opacity-0"
            enter-to="opacity-100"
            leave="duration-200 ease-in"
            leave-from="opacity-100"
            leave-to="opacity-0"
          >
            <div class="fixed inset-0 bg-black bg-opacity-25" />
          </TransitionChild>

          <div class="fixed inset-0 overflow-y-auto">
            <div
              class="flex min-h-full items-center justify-center p-4 text-center"
            >
              <TransitionChild
                as="template"
                enter="duration-300 ease-out"
                enter-from="opacity-0 scale-95"
                enter-to="opacity-100 scale-100"
                leave="duration-200 ease-in"
                leave-from="opacity-100 scale-100"
                leave-to="opacity-0 scale-95"
              >
                <DialogPanel
                  class="w-full max-w-md transform overflow-hidden rounded-2xl bg-white p-6 text-left align-middle shadow-xl transition-all"
                >
                  <div class="aspect-[1.5]">
                    <img
                      :src="currentCountry?.flags.png"
                      class="w-full h-full"
                    />
                  </div>
                  <DialogTitle
                    as="h3"
                    class="text-lg font-bold leading-6 text-gray-900 mt-4"
                  >
                    {{ currentCountry?.name.official }}
                  </DialogTitle>
                  <div class="mt-2">
                    <div>
                      <span class="font-medium">CCA2:</span>
                      {{ currentCountry?.cca2 }}
                    </div>
                    <div>
                      <span class="font-medium">CCA3:</span>
                      {{ currentCountry?.cca3 }}
                    </div>
                    <div>
                      <span class="font-medium">Zho Translation:</span>
                      {{
                        currentCountry?.name.nativeName.zho
                          ? currentCountry?.name.nativeName.zho.official
                          : currentCountry?.translations.zho.official
                      }}
                    </div>
                    <div>
                      <span class="font-medium">Alternative Spellings:</span>
                      {{ currentCountry?.altSpellings.join(", ") }}
                    </div>
                    <div>
                      <span class="font-medium">IDD Root:</span>
                      {{ currentCountry?.idd.root }}
                    </div>
                    <div>
                      <span class="font-medium">IDD Suffixes:</span>
                      {{ currentCountry?.idd.suffixes.join(", ") }}
                    </div>
                  </div>

                  <div class="mt-4">
                    <button
                      type="button"
                      class="button w-full"
                      @click="closeModal"
                    >
                      Got it, thanks!
                    </button>
                  </div>
                </DialogPanel>
              </TransitionChild>
            </div>
          </div>
        </Dialog>
      </TransitionRoot>
    </div>

    <!-- Loading Spinner -->
    <div
      v-if="isLoading || loading"
      class="fixed inset-0 w-full h-full grid place-items-center bg-sky-100/60"
    >
      <half-circle-spinner
        :animation-duration="1000"
        :size="60"
        color="#0ea5e9"
      />
    </div>
  </main>
</template>

<script lang="ts" setup>
import { ref, watch, watchEffect, computed } from "vue";
import { useAxios } from "@vueuse/integrations/useAxios";
import { useOffsetPagination } from "@vueuse/core";
import { HalfCircleSpinner } from "epic-spinners";
import {
  TransitionRoot,
  TransitionChild,
  Dialog,
  DialogPanel,
  DialogTitle,
} from "@headlessui/vue";
import _ from "lodash";

import { axiosInstance } from "./libs/http";
import { Country } from "./interfaces/Country";

const loading = ref(false);
const page = ref(1);

// Initial load
const initalLoad = () => {
  displayCountries.value = countries.value?.slice(0, pageSize.value);
  page.value = 1;

  if (countries.value) {
    pageCount.value = Math.max(
      1,
      Math.ceil(countries.value.length / currentPageSize.value)
    );
  }
};

const {
  data: countries,
  isLoading,
  execute,
  error,
} = useAxios<Country[] | any>("/all", axiosInstance);

/**
 * * Search
 */
let searchTimeout: any;
const searchCriteria = ref("");

watch(searchCriteria, async () => {
  if (searchCriteria.value) {
    // avoid excessive calls
    clearTimeout(searchTimeout);
    searchTimeout = await setTimeout(() => {
      execute(`/name/${searchCriteria.value}`);
    }, 1000);
  } else {
    clearTimeout(searchTimeout);
    searchTimeout = await setTimeout(() => {
      execute();
    }, 1000);
  }
});

/**
 * * Order By
 */
enum OrderBy {
  ASC = "ASC",
  DESC = "DESC",
}

const orderNameBy = ref();

const reset = () => {
  loading.value = true;
  orderNameBy.value = "";
  searchCriteria.value = "";
  execute();
  initalLoad();

  setTimeout(() => {
    loading.value = false;
  }, 1000);
};

const orderCountryNameBy = (order: OrderBy) => {
  orderNameBy.value = order;
  loading.value = true;

  if (order === OrderBy.ASC) {
    countries.value = _.orderBy(countries.value, ["name.official"], "asc");
  } else if (order === OrderBy.DESC) {
    countries.value = _.orderBy(countries.value, ["name.official"], "desc");
  }

  // just additional UX
  setTimeout(() => {
    loading.value = false;
  }, 1000);
};

/**
 * * Pagination
 */
const displayCountries = ref();
const pageSize = ref(25);
const pageCount = ref();

const fetchData = ({
  currentPage,
  currentPageSize,
}: {
  currentPage: number;
  currentPageSize: number;
}) => {
  const start = (currentPage - 1) * currentPageSize;
  const end = start + currentPageSize;
  displayCountries.value = countries.value?.slice(start, end);
};

const { currentPage, currentPageSize, isFirstPage, prev, next } =
  useOffsetPagination({
    total: countries.value?.length,
    page,
    pageSize,
    onPageChange: fetchData,
  });

/**
 * * Custom Last Page Logic
 *
 * Due to problem with useOffsetPagination package has bugs that doesn't
 * support async/await to get the actual pageCount
 */
watchEffect(() => {
  initalLoad();
});

const isLastPage = computed(() => currentPage.value === pageCount.value);

/**
 * * Country Dialog
 */
const isDialogOpen = ref(false);
const currentCountry = ref<Country>();

const viewCountryDetails = (country: Country) => {
  currentCountry.value = country;
  isDialogOpen.value = true;
};

const closeModal = () => {
  isDialogOpen.value = false;
};
</script>

<style lang="scss" scoped>
.button {
  @apply px-5 py-2.5 mt-4 text-white font-medium rounded-sm text-sm bg-sky-700 hover:bg-sky-800 focus:ring-2 focus:ring-blue-300;
}

.button-alt {
  @apply px-5 py-2.5 mt-4 font-medium rounded-sm text-sm hover:bg-sky-800 focus:ring-2 focus:ring-blue-300;
}
</style>
