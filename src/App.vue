<script setup lang="ts">
import { computed, ref, watch } from "vue";
import PokemonItem from "./components/PokemonItem.vue";
import { useAsyncState, useInfiniteScroll } from "@vueuse/core";
import Pagination from "./components/Pagination.vue";

const offset = ref(0);
const limit = ref(6);
const hasMore = ref(true);
const isPaged = ref(true);
const total = ref(0);
const data = ref([] as any[]);

const keyword = ref("");

const { execute: load, isLoading: loading } = useAsyncState(
  async () => {
    if (keyword.value) {
      const url = `https://pokeapi.co/api/v2/pokemon/${keyword.value.toLowerCase()}/`;
      const res = await fetch(url);
      const result = await res.json();
      console.log(result);
      if (!result) return;
      data.value = [
        {
          name: result.name,
          url: `https://pokeapi.co/api/v2/pokemon/${result.id}`,
        },
      ];
      total.value = 1;
      return;
    }
    const url = `https://pokeapi.co/api/v2/pokemon?offset=${offset.value}&limit=${limit.value}`;
    const res = await fetch(url);
    const { results, count } = await res.json();
    total.value = count;
    hasMore.value = results.length >= limit.value;
    if (offset.value > 0 && !isPaged.value) {
      data.value.push(...results);
    } else {
      data.value = results;
    }
  },
  [],
  {
    onError(e) {
      data.value = [];
      total.value = 0;
    },
  }
);

const loadMore = async () => {
  if (isPaged.value) return;
  if (!hasMore.value) return;
  offset.value += data.value.length;
};

watch(offset, load);

const currentPage = computed({
  get() {
    return Math.floor(offset.value / limit.value) + 1;
  },
  set(n) {
    offset.value = limit.value * (n - 1);
  },
});

useInfiniteScroll(window, loadMore);

watch(isPaged, () => {
  offset.value = 0;
  load();
  if (!isPaged.value) {
    loadMore();
  }
});
</script>
<template>
  <div class="">
    <div class="navbar bg-base-200 px-3 flex-wrap">
      <div class="avatar bg-slate-500 text-base-100 mask mask-hexagon p-3">
        P
      </div>
      <a class="btn btn-ghost text-xl mx-3">Pokemon Feed</a>
      <span class="text-xs text-slate-500">by Johnny Wu</span>
      <div class="flex-none ml-auto gap-2">
        <form class="flex gap-2" @submit.prevent="load">
          <div class="form-control">
            <input
              v-model="keyword"
              type="text"
              placeholder="Search"
              class="input input-bordered w-auto flex-shrink-0"
            />
          </div>
          <button :disabled="loading" class="btn btn-primary" type="submit">Search</button>
        </form>
      </div>
    </div>
    <div class="px-3">
      <div class="py-5 flex justify-center">
        <div role="tablist" class="tabs tabs-boxed">
          <a
            role="tab"
            class="tab"
            :class="{ 'tab-active': !isPaged }"
            @click="isPaged = false"
            >Infinite Scroll</a
          >
          <a
            role="tab"
            class="tab"
            :class="{ 'tab-active': isPaged }"
            @click="isPaged = true"
            >Pagination</a
          >
        </div>
      </div>
      <div class="grid md:grid-cols-2 lg:grid-cols-3 gap-3 pb-5" v-if="data.length">
        <PokemonItem
          v-for="item in data"
          :key="item.name"
          :url="item.url"
        ></PokemonItem>
      </div>
      <div v-else class="pb-5 text-center">
        <span class="text-sm text-slate-500">Nothing...</span>
        
      </div>
      <Pagination
        v-if="isPaged"
        :loading="loading"
        v-model="currentPage"
        :total="total"
        :limit="limit"
      ></Pagination>

      <div class="text-center py-3" v-show="loading">
        <span class="loading loading-bars"></span>
      </div>
    </div>
  </div>
</template>
