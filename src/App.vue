<template>
  <section
    ref="intersectionObserverRoot"
    class="h-screen w-screen overflow-y-scroll bg-gray-20 p-8"
  >
    <ul
      class="flex flex-col gap-4"
    >
      <li
        v-for="{ id, name, species, house } in items"
        :key="id"
        class="rounded-4 overflow-hidden shadow-5 bg-white"
      >
        <article class="flex gap-4">
          <section
            class="min-h-full w-8 flex-none"
            :class="[
              (() => {
                switch (house) {
                  case 'Gryffindor':
                    return 'bg-red-90'
                  case 'Ravenclaw':
                    return 'bg-blue-70'
                  case 'Hufflepuff':
                    return 'bg-yellow-50'
                  case 'Slytherin':
                    return 'bg-green-70'
                  default:
                    return 'bg-gray-80'
                }
              })()
            ]"
          />
          <section class="flex flex-col gap-3 p-4">
            <span class="text-6 font-6">{{ name }}</span>
            <span class="uppercase tracking-3 font-6 text-3 text-gray-60">{{ species }}</span>
            <span class="uppercase tracking-3 font-6 text-3 text-gray-60">{{ house || 'House unknown' }}</span>
          </section>
        </article>
      </li>
    </ul>
    <div
      ref="observed"
      class="h-8 bg-blue-10 text-blue-90 w-full"
    >
      <span
        v-if="delayable.status === 'delaying'"
      >
        Loading...
      </span>
      <span
        v-else
      >
        Loaded
      </span>
    </div>
    
  </section>
</template>

<script>
import { ref, onMounted } from 'vue'
import { useFetchable, useListenable, useDelayable } from '@baleada/vue-composition'
import allItems from './allItems'

export default {
  setup () {
    onMounted(() => {
      intersectable.value.listen(
        entries => {
          const { 0: { isIntersecting } } = entries

          console.log(entries[0].intersectionRatio)

          if (isIntersecting) {
            fetchMoreItems()
          }
        },
        {
          target: observed.value,
          observer: {
            root: intersectionObserverRoot.value,
          },
        }
      )
    })

    const intersectable = useListenable('intersect'),
          fetchMoreItems = () => delayable.value.delay(),
          delayable = useDelayable(
            () => {
              console.log(items.value)
              items.value = [...items.value, ...allItems.slice(items.value.length, items.value.length + 10)]
            },
            { delay: 2000 },
          ),
          items = ref(allItems.slice(0, 10)),
          observed = ref(null),
          intersectionObserverRoot = ref(null)

    return {
      observed,
      intersectionObserverRoot,
      items,
      delayable,
    }
  }
}
</script>
