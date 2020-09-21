<template>
<div class="3jRnDcp6ebWPhc9uy3xDuQ">
  <div class="container">
    <div class="field">
      <label class="label">Playlist-ID</label>
      <div class="control has-icons-right">
        <input v-on:change="getPlaylist" v-model="playlistid" class="input" type="text" placeholder="Playlist-ID from Spotify">
        <span class="icon is-small is-right">
          <i class="fas fa-music"></i>
        </span>
      </div>
    </div>
    <div class="field">
      <button v-on:click="saveID" class="button is-link" style="margin-right: 2mm">Save Playlist-ID</button>
      <button v-on:click="getAnalysis" class="button" style="margin-right: 2mm">Reload Track-Analysis</button>
      <button v-on:click="getPlaylist" class="button">Reload Playlist</button>
    </div>
  </div>
  <hr>
  <div class="columns is-multiline" style="margin-top: 1cm" :key="playlistid">
    <div class="card column is-one-fifth-desktop is-full-mobile is-one-third-tablet" style="padding: 6mm; margin-bottom: 7mm" v-for="track in tracks" :key="track.id">
      <div class="card-image">
        <figure class="image is-square">
          <img :src="track.album.images[0].url" alt="Album cover">
        </figure>
        <figure class="is-image">
          <img :src="'https://scannables.scdn.co/uri/plain/png/ffffff/black/800/spotify:track:'+track.id">
        </figure>
      </div>
      <div class="card-content">
        <div class="media">
          <div class="media-content">
            <p class="title is-4">{{ track.name }}</p>
            <p class="subtitle is-6">by {{ track.artists[0].name }}</p>
          </div>
        </div>

        <div class="content"  :key="analysisloaded">
          <p v-if="analysis[track.id] != undefined">Tempo {{ analysis[track.id].track.tempo }} BPM</p>
          <br>
          Released <time>{{ track.album.release_date}}</time>
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
  name: 'Playlist',
  data() {
    return {
      oauthtoken: null,
      playlistid: '',
      tracks: [],
      analysis: {},
      analysisloaded: 0
    }
  },
  created: function() {
    if (getFragment('access_token') === null) {
      if (getCookie('spotify-auth') === null) {
        var scopes = 'playlist-read-private user-read-private user-read-playback-state'
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
  },
  methods: {
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
    }
  }
}
</script>
