<template>
  <v-app light>
    <v-content>
      <v-container>
        <player-title-bar></player-title-bar>
        <player-search-bar :playlist="playlist"></player-search-bar>
        <player-info-panel :trackInfo="getTrackInfo"></player-info-panel>
        <player-controls-bars @playtrack="play" @pausetrack="pause" @stoptrack="stop" @skiptrack="skip" @toggleloop="toggleLoop" :loop="loop" :shuffle="shuffle" @toggleshuffle="toggleShuffle" :progress="progress" @updateseek="setSeek"></player-controls-bars>
        <player-playlist-panel :playlist="playlist" :selectedTrack="selectedTrack" @selecttrack="selectTrack" @playtrack="play"></player-playlist-panel>
      </v-container>
    </v-content>
  </v-app>
</template>
 
<script>
import PlayerTitleBar from './components/PlayerTitleBar.vue'
import PlayerPlaylistPanel from './components/PlayerPlaylistPanel.vue'
import PlayerControlsBars from './components/PlayerControlsBars.vue'
import PlayerInfoPanel from './components/PlayerInfoPanel.vue'
import PlayerSearchBar from './components/PlayerSearchBar.vue'
  export default {
    components: {
      PlayerTitleBar, // REGISTER the component
      PlayerPlaylistPanel,
      PlayerControlsBars,
      PlayerInfoPanel,
      PlayerSearchBar,
    },
    data () {
      return {
        playlist: [
          {title: "I Don't Want to Miss a Thing", artist: "Aerosmith", howl: null, display: true},
          {title: "Best Day Of My Life", artist: "American Authors", howl: null, display: true},
          {title: "Home", artist: "Michael Bublé", howl: null, display: true},
          {title: "I Feel Good", artist: "Pablo López Jahvel Johnson", howl: null, display: true},
          {title: "Bohemian Rhapsody", artist: "Queen", howl: null, display: true},
        ],
        selectedTrack: null,
        index: 0,
        playing: false,
        loop: false,
        shuffle: false,
        seek: 0,
      }
    },
    created: function () {
      this.playlist.forEach( (track) => {
        let file = track.title.replace(/\s/g, "_")
        track.howl = new Howl({
          src: [`./playlist/${file}.mp3`],
          onend: () => {
            if (this.loop) {
              this.play(this.index)
            } else {
              this.skip('next')
            }
          }
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
        
        let lastIndex = this.playlist.length - 1
 
        if (this.shuffle) {
          index = Math.round(Math.random() * lastIndex)
          while (index === this.index) {
            index = Math.round(Math.random() * lastIndex)
          }
        } else if (direction === "next") {
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
      toggleLoop (value) {
        this.loop = value
      },
      toggleShuffle (value) {
        this.shuffle = value
      },
      setSeek (percents) {
  let track = this.currentTrack.howl
 
  if (track.playing()) {
    track.seek((track.duration() / 100) * percents)
  }
}
    },
    computed: {
      currentTrack () {
        return this.playlist[this.index]
      },
      progress () {
        if (this.currentTrack.howl.duration() === 0) return 0
          return this.seek / this.currentTrack.howl.duration()
      },
      getTrackInfo () {
        let artist = this.currentTrack.artist,
        title = this.currentTrack.title,
        seek = this.seek,
        duration = this.currentTrack.howl.duration()
        return {
          artist,
          title,
          seek,
          duration,
        }
      }
    },
    watch: {
  playing(playing) {
    this.seek = this.currentTrack.howl.seek()
    let updateSeek
    if (playing) {
      updateSeek = setInterval(() => {
        this.seek = this.currentTrack.howl.seek()
      }, 250)
    } else {
      clearInterval(updateSeek)
    }
  },
},
  }
</script>