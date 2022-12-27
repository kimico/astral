<template>
  <div ref="src" class="hidden break-word-wrap"><slot /></div>
  <div ref="append" class="hidden break-word-wrap"><slot name="append" /></div>
  <div v-if="metadatadownloading">
    <h1>METADATA</h1>
  </div>
  <div v-if="torrent">
    <p v-if="torrent.progress < 0.01" @click="download">Download</p>
    <p>
      {{ Math.round(torrent.downloaded / 1024 / 1024) }}/{{
        Math.round(torrent.length / 1024 / 1024)
      }}
      MB
    </p>
    <div
      v-for="(item) in files"
      :key="item.name"
    >
      <div v-html=media(item)  ></div>
    </div>
  </div>
  <p v-if="progress > 0.01">{{ Math.ceil(progress * 100) }}%</p>
  <div
    v-html="html"
    ref="html"
    class="break-word-wrap dynamic-content"
    @click="handleClicks"
  />
</template>

<script>
import helpersMixin from '../utils/mixin'
import WebTorrent from 'webtorrent'

const torrent_client = new WebTorrent()

export default {
  name: 'BaseMagnetLink',
  mixins: [helpersMixin],
  emits: ['expand', 'resized'],
  components: {},

  data() {
    return {
      magnet: false,
      html: '',
      torrent: false,
      size: 0,
      files: {},
      metadatadownloading: false,
      progress: 0,
    }
  },

  props: {
    content: {
      type: String,
      default: 'todo needs to be updated', //plz?
    },
    longForm: {
      type: Boolean,
      default: false,
    },
  },

  computed: {},
  beforeMount() {
    var parts = this.$props.content.split(' ').filter((part) => {
      return part.startsWith('magnet:?xt=')
    })
    var self = this
    if (parts.length && !self.torrent && !self.metadatadownloading) {
      self.magnet = parts[0]
      self.magnet = 'magnet:?xt=urn:btih:08ada5a7a6183aae1e09d831df6748d566095a10&dn=Sintel&tr=udp%3A%2F%2Fexplodie.org%3A6969&tr=udp%3A%2F%2Ftracker.coppersurfer.tk%3A6969&tr=udp%3A%2F%2Ftracker.empire-js.us%3A1337&tr=udp%3A%2F%2Ftracker.leechers-paradise.org%3A6969&tr=udp%3A%2F%2Ftracker.opentrackr.org%3A1337&tr=wss%3A%2F%2Ftracker.btorrent.xyz&tr=wss%3A%2F%2Ftracker.fastcast.nz&tr=wss%3A%2F%2Ftracker.openwebtorrent.com&ws=https%3A%2F%2Fwebtorrent.io%2Ftorrents%2F&xs=https%3A%2F%2Fwebtorrent.io%2Ftorrents%2Fsintel.torrent'
      self.magnet = 'magnet:?xt=urn:btih:48f05e03bb65927751b774d3579f130753822bf1&dn=YosemiteNightSky_1920x1080.jpg&tr=wss%3A%2F%2Ftracker.btorrent.xyz&tr=wss%3A%2F%2Ftracker.openwebtorrent.com&tr=udp%3A%2F%2Ftracker.leechers-paradise.org%3A6969&tr=udp%3A%2F%2Ftracker.coppersurfer.tk%3A6969&tr=udp%3A%2F%2Ftracker.opentrackr.org%3A1337&tr=udp%3A%2F%2Fexplodie.org%3A6969&tr=udp%3A%2F%2Ftracker.empire-js.us%3A1337'
      self.magnet = 'magnet:?xt=urn:btih:53ee4a197c8e7c0a2008e1250016024433e619fc&dn=SolarImpulse2.jpg&tr=wss%3A%2F%2Ftracker.btorrent.xyz&tr=wss%3A%2F%2Ftracker.openwebtorrent.com&tr=udp%3A%2F%2Ftracker.leechers-paradise.org%3A6969&tr=udp%3A%2F%2Ftracker.coppersurfer.tk%3A6969&tr=udp%3A%2F%2Ftracker.opentrackr.org%3A1337&tr=udp%3A%2F%2Fexplodie.org%3A6969&tr=udp%3A%2F%2Ftracker.empire-js.us%3A1337'

      self.metadatadownloading = true
      let torrents = torrent_client.torrents.filter((tr) => {
        return this.magnet.indexOf(tr.infoHash) !== -1
      })
      if (!torrents.length) {
        console.log('Add Magnet link', this.magnet)
        torrent_client.add(self.magnet, {}, function (torrent_metadata) {
          self.metadatadownloading = false
          self.torrent = torrent_metadata
          if (self.torrent.length / 1024 / 1024 > 0.25) self.torrent.pause()
          else self.download()
        })
      }
    }
  },
  mounted() {
    return this.render()
  },
  watch: {},

  updated() {
    this.render()
  },

  methods: {
    download() {
      var self = this
      self.torrent.resume()


      self.torrent.on('done', function () {
        self.metadatadownloading = false
      })
      self.torrent.on('download', function (bytes) {
        console.log('self.torrent.progress', self.torrent.progress)
        self.progress = self.torrent.progress
        if (self.progress > 0.1) {
          self.metadatadownloading = false
        }

        var new_files = self.torrent.files.filter(function (file) {
          if (file.done) {
            return true
          }
        })


        new_files.forEach((a) => {
          if (!self.files[a.name]) {
            a.getBlobURL(function (err, url) {
            if (err) return
            self.files[a.name] = {
              name: a.name,
              url: url
            }
          })
          }
        })


        console.log('nf', self.files)
      })
    },
    render() {
      //this.$refs.html.innerHTML = this.html
      // if (this.links.length === 0) {
      //   this.$refs.html.querySelectorAll('a').forEach(link => this.links.push(link.href))
      //   // if (links[0] && links[0].href) this.links.push(links[0].href)
      //   // links.forEach(link => this.links.push(link.href))
      //   console.log('links', this.links)
      // }
    },

    handleClicks(event) {
      // ensure we use the link, in case the click has been received by a subelement
      let { target } = event
      console.log('Click', target)
      // while (target && target.tagName !== 'A') target = target.parentNode
      // handle only links that occur inside the component and do not reference external resources
    },
    media(e) {
      if (
        e.name.endsWith('.gif') ||
        e.name.endsWith('.png') ||
        e.name.endsWith('.jpeg') ||
        e.name.endsWith('.jpg') ||
        e.name.endsWith('.svg')
      ) {
        return `<img src="${e.url}" crossorigin async loading='lazy' style="max-width: 90%; max-height: 50vh;">`
      }
    },
    expand(event) {
      // document.querySelector('#long-form-button').style.name.display = 'none'
      // document.querySelector('#post-text').classList.remove('long-form')
      this.$emit('expand')
    },
  },
}
</script>

<style lang="scss">
a {
  text-decoration: underline;
  color: #448195;
}
p {
  margin-block-end: 0.5rem;
}
ul {
  list-style-type: disc;
}
ol {
  list-style-type: decimal;
}
ul,
ol {
  list-style-position: outside;
  padding-inline-start: 1.5rem;
  margin-block-start: 0.5rem;
  margin-block-end: 0.5rem;
  text-align: left;
}
.post-highlighted ul,
.post-highlighted ol {
  padding-inline-start: 2rem;
}
ul ul,
ol ul {
  list-style-type: circle;
  list-style-position: outside;
  margin-left: 0.5rem;
}
ol ol,
ul ol {
  list-style-type: lower-latin;
  list-style-position: outside;
  margin-left: 0.5rem;
}

.break-word-wrap p:last-of-type {
  margin: 0;
}
.break-word-wrap {
  word-wrap: break-word;
  word-break: break-word;
}
.break-word-wrap p:has(img),
.break-word-wrap p:has(video) {
  display: inline-block;
}
.break-word-wrap img,
.break-word-wrap video {
  border-radius: 1rem;
  border: 1px solid var(--q-accent);
  display: block;
}
.break-word-wrap pre {
  overflow: auto;
}
.long-form {
  max-height: 10rem;
  overflow-y: hidden;

  /* Permalink - use to edit and share this gradient: https://colorzilla.com/gradient-editor/#000000+0,000000+100&1+0,1+51,0.7+58,0+100 */
  background: -moz-linear-gradient(
    top,
    rgba(0, 0, 0, 1) 0%,
    rgba(0, 0, 0, 1) 51%,
    rgba(0, 0, 0, 0.7) 58%,
    rgba(0, 0, 0, 0) 100%
  ); /* FF3.6-15 */
  background: -webkit-linear-gradient(
    top,
    rgba(0, 0, 0, 1) 0%,
    rgba(0, 0, 0, 1) 51%,
    rgba(0, 0, 0, 0.7) 58%,
    rgba(0, 0, 0, 0) 100%
  ); /* Chrome10-25,Safari5.1-6 */
  background: linear-gradient(
    to bottom,
    rgba(0, 0, 0, 1) 0%,
    rgba(0, 0, 0, 1) 51%,
    rgba(0, 0, 0, 0.7) 58%,
    rgba(0, 0, 0, 0) 100%
  ); /* W3C, IE10+, FF16+, Chrome26+, Opera12+, Safari7+ */
  filter: progid:DXImageTransform.Microsoft.gradient( startColorstr='#000000', endColorstr='#00000000',GradientType=0 ); /* IE6-9 */

  margin: 0;
  padding: 0;
  background-clip: text;
  -webkit-background-clip: text;
  color: transparent;
}
.body--dark .long-form {
  /* Permalink - use to edit and share this gradient: https://colorzilla.com/gradient-editor/#000000+0,000000+100&1+0,1+51,0.7+58,0+100 */
  background: -moz-linear-gradient(
    top,
    rgba(255, 255, 255, 1) 0%,
    rgba(255, 255, 255, 1) 51%,
    rgba(255, 255, 255, 0.7) 58%,
    rgba(255, 255, 255, 0) 100%
  ); /* FF3.6-15 */
  background: -webkit-linear-gradient(
    top,
    rgba(255, 255, 255, 1) 0%,
    rgba(255, 255, 255, 1) 51%,
    rgba(255, 255, 255, 0.7) 58%,
    rgba(255, 255, 255, 0) 100%
  ); /* Chrome10-25,Safari5.1-6 */
  background: linear-gradient(
    to bottom,
    rgba(255, 255, 255, 1) 0%,
    rgba(255, 255, 255, 1) 51%,
    rgba(255, 255, 255, 0.7) 58%,
    rgba(255, 255, 255, 0) 100%
  ); /* W3C, IE10+, FF16+, Chrome26+, Opera12+, Safari7+ */
  filter: progid:DXImageTransform.Microsoft.gradient( startColorstr='#000000', endColorstr='#00000000',GradientType=0 ); /* IE6-9 */

  background-clip: text;
  -webkit-background-clip: text;
}
</style>
