<script setup lang="ts">
import { computed,defineEmits } from 'vue'


export interface VFlexPaginationProps {
  itemPerPage: number
  totalItems: number
  currentPage?: number
  maxLinksDisplayed?: number
}
const emit = defineEmits(['update:modelValue'])
const props = withDefaults(defineProps<VFlexPaginationProps>(), {
  currentPage: 1,
  maxLinksDisplayed: 4,
  itemPerPage: 100
})
const lastPage = computed(
    () => Math.ceil(props.totalItems / props.itemPerPage) || 1
)
const totalPageDisplayed = computed(() =>
    lastPage.value > props.maxLinksDisplayed - 2
        ? props.maxLinksDisplayed 
        : lastPage.value
)
const changePosition = (key) => {
  emit('update:page', key)
}

const pages = computed(() => {
  const _pages = []
  let firstButton = props.currentPage - Math.floor(totalPageDisplayed.value / 2)
  let lastButton =
      firstButton +
      (totalPageDisplayed.value - Math.ceil(totalPageDisplayed.value % 2))

  if (firstButton < 1) {
    firstButton = 1
    lastButton = firstButton + (totalPageDisplayed.value - 1)
  }

  if (lastButton > lastPage.value) {
    lastButton = lastPage.value
    firstButton = lastButton - (totalPageDisplayed.value - 1)
  }

  for (let page = firstButton; page <= lastButton; page += 1) {
    if (page === firstButton || page === lastButton) {
      continue
    }

    _pages.push(page)
  }

  return _pages
})
</script>



<template>
  <nav
      role="navigation"
      class="flex-pagination pagination is-rounded"
      aria-label="pagination"
      data-filter-hide
  >
    <div
        v-if="lastPage > 1"
        class="pagination-previous has-chevron"
        @click="changePosition(currentPage - 1)"
    >
      <i
          aria-hidden="true"
          class="iconify"
          data-icon="feather:chevron-left"
      ></i>
    </div>
    <div
        v-if="lastPage > 1"
        class="pagination-next has-chevron"
        @click="changePosition(currentPage + 1)"
    >
      <i
          aria-hidden="true"
          class="iconify"
          data-icon="feather:chevron-right"
      ></i>
    </div>

    <ul class="pagination-list">
      <li>
        <div
            class="pagination-link"

            :class="[currentPage === 1 && 'is-current']"
            @click="changePosition(1)"
        >
          1
        </div>
      </li>

      <li v-if="pages.length === 0 || pages[0] > 2">
        <span class="pagination-ellipsis">…</span>
      </li>

      <li v-for="page in pages" :key="page">
        <div
            class="pagination-link"

            :aria-current="currentPage === page ? 'page' : undefined"
            :class="[currentPage === page && 'is-current']"
            @click="changePosition(page)"
        >
          {{ page }}
        </div>
      </li>

      <li v-if="pages[pages.length - 1] < lastPage - 1">
        <span class="pagination-ellipsis">…</span>
      </li>

      <li>
        <div
            class="pagination-link"

            :class="[currentPage === lastPage && 'is-current']"
            @click="changePosition(lastPage)"
        >
          {{ lastPage }}
        </div>
      </li>
    </ul>
  </nav>
</template>

<style lang="scss" scoped>
.pagination-link {
  width: 23px;
  cursor: pointer;
  border-radius: 10px;
  border: #646cff solid 1px;
  margin: 10px;
}
.is-current {
 background: #ea7e20;

}
.pagination-list {
  margin-left: 550px;
  display: flex;
}
li {
  list-style-type: none; /* Убираем маркеры */
}
ul {
  margin-left: 0; /* Отступ слева в браузере IE и Opera */
  padding-left: 0; /* Отступ слева в браузере Firefox, Safari, Chrome */
}

</style>
