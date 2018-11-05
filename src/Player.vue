<template>
  <v-app dark>
    <v-content>
      <v-container>
        <player-title-bar></player-title-bar>
        <player-controls-bars @playtrack="play" @pausetrack="pause" @stoptrack="stop" @skiptrack="skip"></player-controls-bars>
        <player-playlist-panel :playlist="playlist" :selectedTrack="selectedTrack" @selecttrack="selectTrack" @playtrack="play"></player-playlist-panel>
      </v-container>
    </v-content>
  </v-app>
</template>
 
<script>
import PlayerTitleBar from './components/PlayerTitleBar.vue'
import PlayerPlaylistPanel from './components/PlayerPlaylistPanel.vue'
import PlayerControlsBars from './components/PlayerControlsBars.vue'
  export default {
    components: {
      PlayerTitleBar, // REGISTER the component
      PlayerPlaylistPanel,
      PlayerControlsBars,
    },
    data () {
      return {
        playlist: [
          {title: "I Don't Want to Miss a Thing", artist: "Aerosmith", howl: null, display: true},
          {title: "Best Day Of My Life", artist: "American Authors", howl: null, display: true},
        ],
        selectedTrack: null,
        index: 0,
        playing: false,
      }
    },
    created: function () {
      this.playlist.forEach( (track) => {
        let file = track.title.replace(/\s/g, "_")
        track.howl = new Howl({
          src: [`./playlist/${file}.mp3`]
        })
      })
    },
    methods: {
      selectTrack (track) {
        this.selectedTrack = track
      },
      play (index) {
        let selectedTrackIndex = this.playlist.findIndex(track => track === this.selectedTrack)
 
        if (typeof index === 'number') {
          index = index
        } else if (this.selectedTrack) {
          if (this.selectedTrack != this.currentTrack) {
            this.stop()
          }
          index = selectedTrackIndex
        } else {
          index = this.index
        }
 
        let track = this.playlist[index].howl
 
        if (track.playing()) {
          return
        } else {
          track.play()
        }
   
        this.selectedTrack = this.playlist[index]
        this.playing = true
        this.index = index
      },
      pause () {
        this.currentTrack.howl.pause()
        this.playing = false
      },
      stop () {
        this.currentTrack.howl.stop()
        this.playing = false
      },
      skip (direction) {
        let index = 0
 
        if (direction === "next") {
          index = this.index + 1
          if (index >= this.playlist.length) {
            index = 0
          }
        } else {
          index = this.index - 1
          if (index < 0) {
            index = this.playlist.length - 1
          }
        }
 
        this.skipTo(index)
      },
      skipTo (index) {
        if (this.currentTrack) {
          this.currentTrack.howl.stop()
        }
 
        this.play(index)
      },
    },
    computed: {
      currentTrack () {
        return this.playlist[this.index]
      }
    }
  }
</script>