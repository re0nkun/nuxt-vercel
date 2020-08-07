<template>
  <div>
    <v-app-bar
      flat
      dense
    >
      <nuxt-link to="/" class="toolbar-title mx-5">
        <v-toolbar-title @click="clear" v-text="title" />
        <!-- {{ filteredDatas.length }} -->
      </nuxt-link>

      <v-chip
        v-for="(c, i) in checked"
        :key="i"
        label
        small
        close
        class="mx-1 px-1 grey--text text--darken-2"
        @click:close="closeTag(c)"
      >
        <span>{{ c }}</span>
      </v-chip>
    </v-app-bar>

    <v-main class="my-2">
      <v-container class="py-0">
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
                @click="addTag(tag)"
              >
                {{ tag }}
              </v-chip>
            </div>
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </div>
</template>

<script>
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
      checked: null
    }
  },
  computed: {
    filteredDatas () {
      return this.datas.filter((d) => {
        return !this.checked.some((c) => {
          return d.subject.some((s) => {
            return s === c
          })
        })
      })
    }
  },
  created () {
    this.checked = this.$route.query.t ? Array.isArray(this.$route.query.t) ? this.$route.query.t : [this.$route.query.t] : []
  },
  methods: {
    addTag (tag) {
      const t = this.$route.query.t
      if (!t) {
        this.$router.push({ query: { t: tag } })
        this.checked = [tag]
      } else if (!Array.isArray(t)) {
        this.$router.push({ query: { t: [...new Set([t, tag])] } })
        this.checked = [t, tag]
      } else {
        this.$router.push({ query: { t: [...new Set([...t, tag])] } })
        this.checked = [...t, tag]
      }
    },
    closeTag (c) {
      const tag = this.$route.query.t
      console.log(tag, c)

      if (!tag) {
        return 0
      } else if (!Array.isArray(tag)) {
        this.$router.push({ query: { t: [] } })
        this.checked = []
      } else {
        const newTag = tag.filter((n) => { return n !== c })
        console.log(newTag)
        this.$router.push({ query: { t: newTag } })
        this.checked = newTag
      }
    },
    clear () {
      this.checked = []
      this.$router.push({ query: { t: [] } })
    }
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
</style>
