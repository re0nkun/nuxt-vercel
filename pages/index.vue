<template>
  <div>
    <v-app-bar>
      <v-toolbar-title>NuxtVercle</v-toolbar-title>
    </v-app-bar>

    {{ datas }}
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
            // link: items[i].link[0] || null,
            // description: items[i].description[0] || null,

            // subject: items[i]['dc:subject'] || null,
            bookmarkcount: Number(items[i]['hatena:bookmarkcount'][0]) || null,
            // bookmarkCommentListPageUrl: items[i]['hatena:bookmarkCommentListPageUrl'][0] || null,
            // imageurl: items[i]['hatena:imageurl'] || null
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
  }
}
</script>
