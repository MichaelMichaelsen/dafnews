<template>
  <div>
    <q-linear-progress :value="progress" class="q-mt-md" />
    <q-space />
    <div class="row">
      <div class="col-12 col-md-3"
          v-for="message in messages"
          :key="message.id">
        <q-card
          class="feed-card"
          bordered
          >
          <q-card-section :style="{ 'background-color': titleBackground(message.title)}">
            <div class="text-h6">{{ message.id }}. {{ message.title }}
              <q-icon :name="message.statusIcon" style="font-size: 1.2em;" />
            </div>
          </q-card-section>
          <q-separator />
          <q-card-section>
            <div v-html="message.message"></div>
          </q-card-section>
          <q-card-actions align="around">
              <a :href="message.url" outline color="purple">{{ message.url }} </a>
          </q-card-actions>
        </q-card>
      </div>
    </div>
  </div>
</template>

<script>
const RSSparser = require('rss-parser')
const parser = new RSSparser()
export default {
  name: 'PageIndex',
  data () {
    return {
      messages: [],
      progress: 0.0
    }
  },
  mounted () {
    const url = 'https://cors-anywhere.herokuapp.com/https://datafordeler.dk/api/operations/serviceMessages/atom'

    parser.parseURL(url, (error, feed) => {
      if (error) throw error
      console.log(feed)
      let id = 1
      const igang = new RegExp('Status: I gang$')
      const loest = new RegExp('Status: Løst$')
      const besked = new RegExp(/Besked: /ig)
      const service = new RegExp(/Service: /ig)
      const numberOfMessage = feed.items.length
      const step = 100.0 / numberOfMessage
      console.log('Step ', step)
      const timerId = setInterval(() => { this.progress = Math.min(1, this.progress + 0.1) }, 500)
      // const kommende = new RegExp('Status: Kommende$')
      feed.items.forEach(item => {
        console.log(item.title + ':' + item.link)
        let statusIcon = ''
        if (loest.test(item.content)) {
          console.log('Status løst')
          statusIcon = 'thumb_up'
        }
        if (igang.test(item.content)) {
          console.log('Status i gang')
          statusIcon = 'warning'
        }
        this.messages.push({
          id: id++,
          title: item.title,
          message: item.content.replace(besked, '').replace(service, '<br/>'),
          url: item.link.toLowerCase(),
          statusIcon: statusIcon
        })
        clearInterval(timerId)
        this.progress = 1.0
      })
    })
  },
  methods: {
    titleBackground (title) {
      console.log(title)
      if (title === 'Meddelelser') return 'lightblue'
      if (title === 'Danske Stednavne') return 'lightgreen'
      if (title === 'GeoDanmark Vektor') return 'lightyellow'
      if (title === 'Det Centrale Virksomhedsregister (CVR)') return 'lightsalmon'
      if (title === 'Danmarks Administrative Geografiske Inddelinger (DAGI)') return 'khaki'
      return 'grey'
    }
  }
}
</script>
<style  scoped>
.feed-card{
  width: 80%;
  max-width: 80%;
}
</style>
