<template>
  <div v-if="history.length">
    <nav class="panel">
      <p class="panel-heading has-text-centered">
        {{ msg }}
      </p>
      <div class="panel-block">
        <p class="control has-icons-left">
          <input
            class="input"
            type="text"
            placeholder="Search"
            @input="inputChange"
          />
          <span class="icon is-left">
            <i class="fas fa-search" aria-hidden="true"></i>
          </span>
        </p>
      </div>
      <p
        v-for="data in Newhistory"
        :key="data"
        class="panel-block is-clickable ellipsis hover "
        @click="Itemclicked(data)"
      >
      <em>
        {{ data.copiedtext }}
      </em>
      <br>
      <small>{{data.copiedDate}}</small>
      </p>
      <div class="panel-block">
        <button
          class="button is-link is-outlined is-fullwidth"
          @click="Resetdatabase()"
        >
          Reset Clipboard
        </button>
      </div>
    </nav>
  </div>
  <div v-if="!history.length">
    <p class="panel-heading">Sorry You clipboard is empty</p>
    <img
      class="nodataimage"
      alt="Clipboard logo"
      src="../assets/ClipboardImage.jpg"
    />
    <h4 class="subtitle is-4">
      Please start copying Data from the clipboard :)
    </h4>
  </div>
</template>

<script>
const { ipcRenderer } = require('electron')
const { clipboard } = require('electron')
import sound from '../../public/CopySound.wav'
const audio = new Audio(sound)
var Datastore = require('nedb')
const database = new Datastore('database.db')
database.loadDatabase()

// ****************************** If you wanna add images, Hints are in the Read.me/Article  below******************************

export default {
  name: 'HelloWorld',
  props: {
    msg: String,
  },
  data() {
    return {
      history: [],
      searchQuery: null,
    }
  },
  created() {
    document.title = 'ClipBoard Manager'
  },
  mounted() {
    setInterval(this.checkClipboard, 500)
    // const buffer =  clipboard.readImage('clipboard')
    // console.log(buffer.toDataURL())
    database.find({}, (err, data) => {
      console.log(data)
      this.history = data
    })
  },
  computed: {
    Newhistory() {
      if (this.searchQuery) {
        return this.history.filter((his) =>
          his.copiedtext.toLowerCase().includes(this.searchQuery),
        )
      } else {
        return this.history.slice().reverse()
      }
    },
  },
  methods: {
    Resetdatabase() {
      this.history = []
      database.remove({}, { multi: true }, function (err, numRemoved) {
        database.loadDatabase(function (err) {})
      })
    },
    checkClipboard() {
      const copiedtext = clipboard.readText()
      if (
        this.history.some((value) => value['copiedtext'] === copiedtext) != true &&
        copiedtext.trim() != ''
      ) {
        this.history.push({
          copiedtext,
          copiedDate: new Date().toLocaleString(),
        })
        database.insert({ name: 'Data', copiedtext: copiedtext, copiedDate: new Date().toLocaleString() })
        audio.currentTime = 0
        audio.play()
      }
    },
    Itemclicked(item) {
      clipboard.writeText(item.copiedtext)
      ipcRenderer.send('minimize')
    },
    inputChange: function (event) {
      this.searchQuery = event.target.value
    },
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
.nodataimage {
  width: 250px;
  height: 350px;
}
.hover:hover {
  background-color: #d3d3d3;
}
.ellipsis {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
  text-overflow: ellipsis;
}
</style>
