<template>
  <div>
    <div>
      <div class="flex justify-center text-center">
        <div class="m-2 m-auto">
          <label class="inline-flex items-center mt-3">
            <span class="mr-2 text-gray-700">Record Testing Data</span>
            <input type="checkbox" class="form-checkbox h-5 w-5 text-gray-600" v-model="testrecord" id="checkbox">
          </label>
        </div>
        <div class="m-2 m-auto">
          <label class="inline-flex items-center mt-3">
            <span class="mr-2 text-gray-700">Select User</span>
            <div class="inline-flex relative w-64">
              <select class="block appearance-none w-full bg-white border border-gray-400 hover:border-gray-500 px-4 py-2 pr-8 rounded shadow leading-tight focus:outline-none focus:shadow-outline" @change="getRecorded" v-model="user">
                <option v-for="user in users" :key="user.id" :value="user.Id">{{ user.name }}</option>
              </select>
              <div class="pointer-events-none absolute inset-y-0 right-0 flex items-center px-2 text-gray-700">
                <i class="fill-current h-4 w-4 fas fa-arrow-down"></i>
              </div>
            </div>
          </label>
        </div>
      </div>
      <div class="flex justify-center py-4">
        <hr class="w-1/3">
      </div>
      <div class="flex justify-center text-center">
        <div class="m-2">
          <label class="block text-gray-700 text-sm font-bold mb-2" for="username">
        Add User
          </label>
          <div class="flex justify-center">
            <input class="border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline" id="name" v-model="username" type="text" placeholder="Name">
            <button v-on:click="addUser(username)" class="ml-2 bg-gray-300 hover:bg-blue-700 font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline" type="button">Add</button>
          </div>
        </div>
      </div>


    </div>
    <div class="py-4">
      <hr>
    </div>


    <div class="flex justify-center text-center" v-if="playing">
      <div class="max-w-xl rounded overflow-hidden shadow-lg" v-if="'album' in playing">
        <div class="px-6 pt-4 pb-2">
          <button v-on:click="record()" class="bg-gray-300 hover:bg-gray-400 mr-2 text-gray-800 font-bold py-2 px-4 rounded inline-flex items-center"><i class="mr-4 fas fa-brain"></i><span>Record EEG</span></button>
          <button v-on:click="previousTrack()" class="bg-gray-300 hover:bg-gray-400 mx-2 text-gray-800 font-bold py-2 px-4 rounded inline-flex items-center"><i class="mr-4 fas fa-backward"></i><span>Previous Track</span></button>
          <button v-on:click="nextTrack()" class="bg-gray-300 hover:bg-gray-400 ml-2 text-gray-800 font-bold py-2 px-4 rounded inline-flex items-center"><i class="mr-4 fas fa-forward"></i><span>Next Track</span></button>
        </div>
        <img class="object-none" :src="playing.album.images[0].url" :alt="playing.album.name">
        <div class="px-6 py-4">
          <div class="font-bold text-xl mb-2">{{ playing.name }}</div>
          <p class="text-gray-700 text-base">
            {{ playing.album.name }}<br>
            {{ playing.album.artists[0].name }}<br>
            {{ playing.album.release_date }}
          </p>
        </div>
        <div class="px-6 pt-4 pb-2">
          <figure class="is-image">
            <img :src="'https://scannables.scdn.co/uri/plain/png/ffffff/black/800/spotify:track:'+playing.id">
          </figure>
        </div>
      </div>
    </div>
    <div class="flex justify-center text-center" v-else>
      <h3 class="text-4xl">No song playing</h3>
    </div>
    <div class="py-4">
      <hr>
    </div>

    <div class="text-center">
      <div class="rounded flex justify-center overflow-hidden">
        <div>
          <h3 class="text-xl"><b>EEG Stream</b></h3>
          <div class="" v-if="stream">
            <div class="bg-green-100 border border-green-400 text-green-700 px-4 py-3 rounded relative" role="alert">
              <strong class="font-bold">EEG Streaming </strong>
              <span class="block sm:inline">You can start recording EEG-Data</span>
            </div>
          </div>
          <div class="" v-else>
            <p>EEG not streaming</p><button v-on:click="checkStream()" class="bg-gray-300 hover:bg-gray-400 ml-2 text-gray-800 font-bold py-2 px-4 rounded inline-flex items-center"><i class="mr-4 fas fa-sync"></i><span>Check again</span></button>
          </div>
        </div>
        <div v-if="!stream">
          <h3 class="text-xl"><b>Available Muses</b></h3>
          <div class="" v-if="muses.length !== 0 && !stream">
            <span v-for="muse in muses" :key="muse.name">{{ muse.name }}</span>
          </div>
          <div class="" v-else>
            <p>No muse available!</p><button v-on:click="listmuses()" class="bg-gray-300 hover:bg-gray-400 ml-2 text-gray-800 font-bold py-2 px-4 rounded inline-flex items-center"><i class="mr-4 fas fa-sync"></i><span>Scan again</span></button>
          </div>
        </div>
      </div>
    </div>
    <div class="py-4">
      <hr>
    </div>
    <div class="w-full shadow-lg">
      <h1 class="text-4xl text-center py-6 cursor-pointer" v-on:click="getRecorded()">Songs Recorded <i class="ml-4 fas inline fa-sync"></i></h1>
      <div class="flex justify-center">
        <button v-on:click="train()" class="bg-gray-300 hover:bg-gray-400 ml-2 text-gray-800 flex justify-center font-bold py-2 px-4 rounded inline-flex items-center"><i class="mr-4 fas fa-running"></i><span>Train selection</span></button>
        <button v-on:click="predict()" class="bg-gray-300 hover:bg-gray-400 ml-2 text-gray-800 flex justify-center font-bold py-2 px-4 rounded inline-flex items-center"><i class="mr-4 fas fa-running"></i><span>Predict (select songs)</span></button>
      </div>
      <div class="flex flex-wrap justify-center" :key="changedselections">
        <div class="text-center m-6" v-bind:class="{ 'shadow-2xl bg-green-100': selections[id] }" :key="track.id" v-for="(track, id) in recorded" v-on:click="toggleSelection(id)">
          <div class="max-w-xs rounded overflow-hidden shadow-lg" v-if="'album' in track">
            <img class="object-none" :src="track.album.images[0].url" :alt="track.album.name">
            <div class="px-6 py-4">
              <div class="font-bold text-xl mb-2">{{ track.name }}</div>
              <p class="text-gray-700 text-base">
                {{ track.album.name }}<br>
                {{ track.album.artists[0].name }}<br>
                {{ track.album.release_date }}
              </p>
            </div>
            <div class="px-6 pt-4 pb-2">
              <figure class="is-image">
                <img :src="'https://scannables.scdn.co/uri/plain/png/ffffff/black/800/spotify:track:'+track.id">
              </figure>
            </div>
          </div>
        </div>
      </div>
    </div>

  </div>
</template>

<script>
import axios from 'axios';

function getCookie(a) {
  var b = document.cookie.match('(^|[^;]+)\\s*' + a + '\\s*=\\s*([^;]+)')
  return b ? b.pop() : null
}

function getParam(parameterName) {
  var result = null,
    tmp = [];
  location.search
    .substr(1)
    .split("&")
    .forEach(function(item) {
      tmp = item.split("=");
      if (tmp[0] === parameterName) result = decodeURIComponent(tmp[1]);
    });
  return result;
}
console.log(getParam(1))

function getFragment(parameterName) {
  var result = null,
    tmp = [];
  window.location.hash.substr(1)
    .split("&")
    .forEach(function(item) {
      tmp = item.split("=");
      if (tmp[0] === parameterName) result = decodeURIComponent(tmp[1]);
    });
  return result;
}
export default {
  name: 'Main',
  data() {
    return {
      backend: "http://127.0.0.1:5000",
      oauthtoken: null,
      playlistid: '',
      tracks: [],
      analysis: {},
      analysisloaded: 0,
      playing: {},
      muses: [],
      recorded: [],
      songids: [],
      stream: false,
      testrecord: false,
      username: "",
      users: [],
      selections: [],
      user: 1,
      changedselections: 0,
      isplaying: false
    }
  },
  created: function() {

    this.getToken()
    this.getPlaying()
    this.getRecorded()
    this.getUsers()
    this.checkStream()
  },
  mounted: function () {
        this.$nextTick(function () {
            window.setInterval(() => {
                this.getPlaying();
            },5000);
        })
    },

  methods: {
    train: function () {
      axios.post(this.backend + '/train', { user: this.user, songs: this.selectedids() }).then(result => {
        console.log(result)
      })
    },
    predict: function () {
      axios.post(this.backend + '/predict', { user: this.user, songs: this.selectedids() }).then(result => {
        result.data.predictions.forEach(pred => {
          console.log(pred[0])
        })
      })
    },
    selectedids: function () {
      var selection = []
      this.selections.forEach((bool, id) => {
        if (bool) {
          selection.push(this.songids[id].Id)
        }
      })
      return selection
    },
    toggleSelection: function(id) {
      this.selections[id] = !this.selections[id]
      this.changedselections += 1
    },

    record: function() {

      axios.post(this.backend + '/record', { trackid: this.playing.id, test: this.testrecord, user: this.user }).then(result => {
        console.log(result)
        this.getRecorded()
      })

    },
    listmuses: function() {
      axios.get(this.backend + '/listmuses').then(result => {
        this.muses = result.data.muses
      })
    },
    checkStream: function() {
      axios.get(this.backend + '/stream/check').then(result => {
        this.stream = result.data.data
      })
    },
    addUser: function() {
      axios.post(this.backend + '/users/add', { name: this.username }).then(result => {
        this.users = result.data.users
        this.user = result.data.inserted
        this.getRecorded()
      })
    },
    getUsers: function() {
      axios.get(this.backend + '/users').then(result => {
        this.users = result.data.data

      })
    },
    getRecorded: function() {
      let config = {
        headers: {
          Authorization: 'Bearer ' + this.oauthtoken,
        }
      }
      axios.post(this.backend + '/songs', { user: this.user }).then(result => {
        this.songids = result.data.data
        if (result.data.data.length === 0) {
          this.recorded = []
        } else {
          axios.get('https://api.spotify.com/v1/tracks?ids='+result.data.data.map(track => track.trackid).join(), config).then(result => {
            this.recorded = result.data.tracks
            this.selections = []
            result.data.tracks.forEach(() => {
              this.selections.push(false)
            })
          })
        }
      })
    },
    getPlaying: function() {
      let config = {
        headers: {
          Authorization: 'Bearer ' + this.oauthtoken,
        }
      }

      axios.get('https://api.spotify.com/v1/me/player/currently-playing', config).then(result => {
        if (result.data.item !== undefined) {
          if (this.playing.id !== result.data.item.id) {
            axios.get('https://api.spotify.com/v1/tracks/'+result.data.item.id, config).then(result => {
              this.playing = result.data
              this.isplaying = true
            })

          }
        } else {
          this.playing = {}
          this.isplaying = false
        }

    })
    },
    previousTrack: function() {
      let config = {
        headers: {
          Authorization: 'Bearer ' + this.oauthtoken,
        }
      }
      axios.post('https://api.spotify.com/v1/me/player/previous', {}, config)
      setTimeout(this.getPlaying(), 1000)
    },
    nextTrack: function() {
      let config = {
        headers: {
          Authorization: 'Bearer ' + this.oauthtoken,
        }
      }
      axios.post('https://api.spotify.com/v1/me/player/next', {}, config)
      setTimeout(this.getPlaying(), 1000)
    },
    getPlaylist: function() {
      let config = {
        headers: {
          Authorization: 'Bearer ' + this.oauthtoken,
        }
      }
      axios.get('https://api.spotify.com/v1/playlists/' + this.playlistid + '/tracks', config).then(result => {
        axios.get('https://api.spotify.com/v1/tracks?ids=' + result.data.items.map(track => track.track.id), config).then(res => {
          this.tracks = res.data.tracks;
          this.getAnalysis()
        })
      })

    },
    getAnalysis: function() {
      let config = {
        headers: {
          Authorization: 'Bearer ' + this.oauthtoken,
        }
      }
      this.tracks.forEach(track => {
        axios.get('https://api.spotify.com/v1/audio-analysis/' + track.id, config).then(res => this.analysis[track.id] = res.data)
      })
      this.analysisloaded += 1
    },
    saveID: function() {
      document.cookie = 'playlist=' + this.playlistid + '; expires=31556926'
    },
    getToken: function() {
      if (getFragment('access_token') === null) {
        if (getCookie('spotify-auth') === null) {
          var scopes = 'playlist-read-private user-read-private user-read-playback-state user-read-currently-playing user-modify-playback-state'
          window.location = 'https://accounts.spotify.com/authorize?response_type=token&client_id=9236ab5c0f3c4d418c87f199faeae2c9' + (scopes ? '&scope=' + encodeURIComponent(scopes) : '') + '&redirect_uri=http://192.168.1.149:8080/'
        } else {
          this.oauthtoken = getCookie('spotify-auth')
          if (getCookie('playlist') !== null) {
            this.playlistid = getCookie('playlist')
            this.getPlaylist()
          }
        }
      } else {
        document.cookie = 'spotify-auth=' + getFragment('access_token') + '; expires=1209600'

        window.location = window.location.pathname
      }
    }
  }
}
</script>
