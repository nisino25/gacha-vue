<template>
  <div class="p-6">

    <!-- ✅ 年選択 -->
    <div class="mb-6">
      <select v-if="!showLikedOnly" v-model="showingYear" class="border p-2 rounded">
        <option
          v-for="item in availableYears"
          :key="item.year"
          :value="item.year"
        >
          {{ item.year }}年 ({{ item.count }})
        </option>
      </select>


      <!-- 🔍 open button -->
      <button
        @click="openSearch"
        class="ml-4 px-4 py-2 bg-indigo-600 text-white rounded shadow hover:bg-indigo-700"
      >
        🔍 Search
      </button>

      <!-- ⭐ Liked toggle -->
      <button
        @click="showLikedOnly = !showLikedOnly"
        class="ml-2 px-4 py-2 rounded shadow transition"
        :class="showLikedOnly 
          ? 'bg-yellow-400 text-black' 
          : 'bg-gray-200 hover:bg-gray-300'"
      >
        ⭐ Liked
      </button>

      <!-- 🪟 modal -->
      <div
        v-if="isSearchOpen"
        class="fixed inset-0 bg-black/50 flex items-center justify-center z-50"
      >
        <div class="bg-white w-[500px] max-h-[80vh] rounded-xl p-4 shadow-lg overflow-hidden">

          <!-- header -->
          <div class="flex justify-between items-center mb-3">
            <h2 class="font-bold text-lg">Search</h2>
            <button @click="closeSearch">✖</button>
          </div>

          <!-- input -->
          <input
            v-model="search"
            placeholder="検索..."
            class="w-full border p-2 rounded mb-3"
          />

          <!-- results -->
          <div class="overflow-y-auto max-h-[60vh]">
            <div
              v-for="item in filteredItems"
              :key="item.url"
              class="flex gap-3 items-center p-2 hover:bg-gray-100 rounded cursor-pointer"
            >
              <img v-if="filteredItems.length <= 25" :src="item.thumbnail" class="w-12 h-12 object-cover rounded" />

              <div class="flex-1">
                <p class="font-semibold">{{ item.name }}</p>
                <p class="text-xs text-gray-500">
                  {{ item.yearMonth }} / Week {{ item.week }}
                </p>
              </div>

              <a
                :href="item.url"
                target="_blank"
                class="text-blue-500 text-sm"
              >
                詳細
              </a>
            </div>
          </div>

        </div>
      </div>

    </div>


    <!-- ✅ カンバン -->
    <div class="flex gap-6 overflow-x-auto">
      <div
        v-for="(items, yearMonth) in groupedByYearMonth"
        :key="yearMonth"
        class="min-w-[300px] bg-gray-100 p-4 rounded"
      >
        <h2 class="text-xl font-bold mb-4">{{ yearMonth }}（{{ items.length }}）</h2>
        <div
        v-for="item in items"
        :key="item.url"
        class="bg-white p-3 rounded mb-3 shadow"
        >
          <a :href="item.url" target="_blank" rel="noopener noreferrer">
            <img :src="item.thumbnail" class="w-full mb-4" />
          </a>

          <div class="flex justify-between">
            <strong class="">第{{ item.week }}週・{{ item.kinds }}種</strong>
            <div>
              <button
                @click="toggleLike(item)"
                class="text-xl transition"
              >
                <span v-if="isLiked(item)" class="text-yellow-400">⭐</span>
                <span v-else class="text-gray-300">☆</span>
              </button>
              <a
                :href="'https://jp.mercari.com/search?keyword=' + item.name + ' めじるしアクセサリー'"
                target="_blank"
                rel="noopener noreferrer"
                class="ml-4 inline-block px-2 py-2 bg-red-500 text-white text-sm font-semibold rounded-lg shadow hover:bg-red-600 hover:shadow-md transition"
              >
                メルカリ
              </a>

            </div>
          </div>
          <p class="font-bold">{{ item.name }}</p>

        </div>
      </div>
    </div>

  </div>
</template>


<script>
  import data from "./const/data.json"

  export default {
    name: 'App',
    data(){
      return{
        gachadata: data,
        showingYear: 2026,

        isSearchOpen: false,
        search: "",

        likedItems: {},
        showLikedOnly: false
      }
    },
    computed: {
      availableYears() {
          const map = {}

          this.gachadata.forEach(item => {
              const year = item.yearMonth.split("-")[0]

              if (!map[year]) {
                  map[year] = 0
              }

              map[year]++
          })

          return Object.entries(map)
              .map(([year, count]) => ({
                  year: Number(year),
                  count
              }))
              .sort((a, b) => b.year - a.year)
      },


      groupedByYearMonth() {
    const grouped = {}

    let filtered = this.gachadata

    // ⭐ お気に入りモード
    if (this.showLikedOnly) {
        filtered = filtered.filter(item => this.likedItems[item.url])

        // 👉 年ごとにグループ
        filtered.forEach(item => {
            const year = item.yearMonth.split("-")[0]

            if (!grouped[year]) {
                grouped[year] = []
            }

            grouped[year].push(item)
        })

        // 👉 年内で week昇順
        Object.keys(grouped).forEach(key => {
            grouped[key].sort((a, b) => a.week - b.week)
        })

        // 👉 年で新しい順
        return Object.keys(grouped)
            .sort((a, b) => Number(b) - Number(a))
            .reduce((acc, key) => {
                acc[key] = grouped[key]
                return acc
            }, {})
    }

    // =========================
    // 通常モード（今まで通り）
    // =========================

    filtered = filtered.filter(item =>
        item.yearMonth.startsWith(String(this.showingYear))
    )

    filtered.forEach(item => {
        const yearMonth = item.yearMonth

        if (!grouped[yearMonth]) {
            grouped[yearMonth] = []
        }

        grouped[yearMonth].push(item)
    })

    Object.keys(grouped).forEach(key => {
        grouped[key].sort((a, b) => a.week - b.week)
    })

    return Object.keys(grouped)
        .sort((a, b) => {
            const [yearA, monthA] = a.split("-").map(Number)
            const [yearB, monthB] = b.split("-").map(Number)

            return yearB - yearA || monthB - monthA
        })
        .reduce((acc, key) => {
            acc[key] = grouped[key]
            return acc
        }, {})
}
,

      filteredItems() {
        if (!this.search) return this.gachadata

        const keyword = this.search.toLowerCase()

        return this.gachadata.filter(item =>
          item.name.toLowerCase().includes(keyword)
        )
      },
    },
    methods: {
      openSearch() {
        this.isSearchOpen = true
      },
      closeSearch() {
        this.isSearchOpen = false
        this.search = ""
      },
      toggleLike(item) {
        if (this.likedItems[item.url]) {
          // 解除
          this.$delete
            ? this.$delete(this.likedItems, item.url) // Vue2対応
            : delete this.likedItems[item.url]        // Vue3でもOK
        } else {
          // 追加
          this.likedItems[item.url] = true
        }

        // 💾 保存
        localStorage.setItem("likedItems", JSON.stringify(this.likedItems))
      },

      isLiked(item) {
        return !!this.likedItems[item.url]
      },


    },
    mounted() {
      const saved = localStorage.getItem("likedItems")
      if (saved) {
        this.likedItems = JSON.parse(saved)
      }
    },



  }
</script>

<style>
  #app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    /* text-align: center; */
    color: #2c3e50;
    margin-top: 60px;
  }
</style>
