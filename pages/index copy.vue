<template>
  <div>
    <v-app-bar
      app
      flat
      height="auto"
    >
      <v-row justify="space-around" align="center">
        <nuxt-link to="/" class="toolbar-title">
          <v-toolbar-title v-text="title" />
        </nuxt-link>

        <v-checkbox
          v-for="(c, index) in categories"
          :key="index"
          v-model="checked"
          :value="c"
          :label="c"
          hide-details
          off-icon="mdi-checkbox-blank-circle-outline"
          on-icon="mdi-minus-circle-outline"
          background-color="none"
          color="rgba(0, 0, 0, 0.54)"
          :ripple="false"
          @change="$router.push( {query: {q: checked} } )"
        />
      </v-row>
    </v-app-bar>

    <v-content class="pt-8 mt-12 mt-md-6">
      <v-container fluid class="py-0">
        <v-row>
          <v-col
            v-for="(d, index) in filteredDatas"
            :key="index"
            cols="12"
          >
            <div class="item inline-block">
              <img :src="'https://cdn-ak2.favicon.st-hatena.com/?url=' + d.link">

              <a :href="d.link" class="text-decoration-none">
                <span class="ml-3 grey--text text--darken-3 title">{{ d.title }}</span>
              </a>

              <a :href="d.bookmarkCommentListPageUrl" class="bookmarkcount red--text text-no-wrap">
                <span class="font-weight-black title ml-2">{{ d.bookmarkcount }}</span>
                <span class="body-2">users</span>
              </a>

              <v-chip
                v-for="(tag, i) in d.subject"
                :key="i"
                label
                x-small
                class="mx-1 px-1 grey--text text--darken-2"
              >
                {{ tag }}
              </v-chip>
            </div>
          </v-col>
        </v-row>
      </v-container>
    </v-content>
  </div>
</template>

<script>
// subjectの二つ目以降にもフィルターをかけれるようにする
import { parseString } from 'xml2js'
export default {
  async asyncData ({ $axios }) {
    try {
      const results = []
      for (const c of ['social', 'economics', 'life', 'knowledge', 'it', 'fun', 'entertainment', 'game']) {
        const xml = $axios.$get(`https://b.hatena.ne.jp/hotentry/${c}.rss`)
        results.push(xml)
      }
      const xmls = await Promise.all(results)

      let datas = []
      for (const xml of xmls) {
        let items
        parseString(xml, (err, res) => {
          if (err) { console.log(err) }
          items = res['rdf:RDF'].item
        })

        for (let i = 0; i < items.length; i++) {
          datas.push({
            title: items[i].title[0] || null,
            link: items[i].link[0] || null,
            description: items[i].description[0] || null,

            subject: items[i]['dc:subject'] || null,
            bookmarkcount: Number(items[i]['hatena:bookmarkcount'][0]) || null,
            bookmarkCommentListPageUrl: items[i]['hatena:bookmarkCommentListPageUrl'][0] || null,
            imageurl: items[i]['hatena:imageurl'] || null
          })
        }
      }

      datas.sort((a, b) => {
        if (a.bookmarkcount < b.bookmarkcount) { return 1 }
        if (a.bookmarkcount > b.bookmarkcount) { return -1 }
        return 0
      })

      datas = datas.filter((d) => {
        return d.bookmarkcount >= 100
      })

      return { datas }
    } catch (err) {
      console.log(err)
    }
  },
  data () {
    return {
      title: 'SimpleHB',
      checked: null,
      categories: ['世の中', '政治と経済', '暮らし', '学び', 'テクノロジー', 'おもしろ', 'エンタメ', 'アニメとゲーム']
    }
  },
  computed: {
    filteredDatas () {
      return this.datas.filter((d) => {
        return !this.checked.some((c) => {
          return c === d.subject[0]
        })
        // return !d.subject.some((s) => {
        //   return s === this.subject
        // })
      })
    }
  },
  created () {
    this.checked = this.$route.query.q ? Array.isArray(this.$route.query.q) ? this.$route.query.q : [this.$route.query.q] : []
  }
}
</script>
<style scoped>
.toolbar-title {
  color: inherit;
  text-decoration: inherit;
}
.text-decoration-none {
  text-decoration: none;
}
.item img {
  width: 22px;
  vertical-align: -0.25em;
}
a.bookmarkcount {
  vertical-align: .1em;
}
/* .card-title {
  overflow: hidden;
  width: 80%;
}
.imageurl {
  width: 200px;
  height: 150px;
  object-fit: scale-down;
} */
</style>
