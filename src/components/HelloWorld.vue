<template>
  <div class="hello">
    <h1>{{ msg }}</h1>

    <div class="container">
      <div class="player">
        <div class="player__controls">
          <div class="player__btn player__btn--small" id="previous">
            <i class="fas fa-arrow-left"></i>
          </div>
          <h5 class="player__title">playing now</h5>
          <div class="player__btn player__btn--small" id="icon-menu">
            <i class="fas fa-bars"></i>
          </div>
        </div>
        <div class="player__album">
          <img
            src="https://i1.sndcdn.com/artworks-000501934485-vavvhy-t500x500.jpg"
            alt="Album Cover"
            class="player__img"
            loading="lazy"
          />
        </div>

  


        <h2 class="player__artist">Nombre grupo</h2>
        <h3 class="player__song">{{ songTrack }}</h3>

        <input
          v-model="seek"
          type="range"
          min="0"
          :max="seekMax"
          class="player__level"
          id="range"
          controls="true"
        />
        <div class="audio-duration">
          <div class="start">{{ currentTime }} </div>
          <div class="start">{{ duration }}</div>
        </div>

        <audio  class="player__audio" controls id="audio" >
          <source src="" type="audio/mpeg" />
        </audio>

        <div class="player__controls">
          <div
            @click="skip('')"
            class="player__btn player__btn--medium"
            id="backward"
          >
            <i class="fas fa-backward"></i>
          </div>

          <div
            
            @click="playing ? pause() : play()"
            class="player__btn player__btn--medium blue play"
            id="player"
          >
         
            <i v-if="!playing" class="fas fa-play play-btn"></i>
            <i v-else class="fas fa-pause pause-btn "></i>
          </div>

          <div
            @click="skip('next')"
            class="player__btn player__btn--medium"
            id="forward"
          >
            <i class="fas fa-forward"></i>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import {Howl} from 'howler';
import axios from "axios";


export default {
  name: "HelloWorld",
  props: {
    msg: String,
  },
  data() {
    return {
      trackList: [],
      //urltrack: "http://localhost:3000/tracks/6340ae963543d07d7e184889",
      player: new Audio(),
     // currentTrack: "6340ae963543d07d7e184889",
      songTrack: "",
      position: 0,
      seekbar: {
        max: 100,
        value: 0,
      },
      duration: "0:00",
      currentTime: "0:00",
      music: {
        element: HTMLMediaElement,
        paused: false,
        loop: false,
        volume: 50,
        favourite: false,
      },
      playing:false,
      audio: new Audio(),
      playlist:[],
      index:0,
      seek:0,
      seekMax:0
    };
  },
  mounted() {
    let x=false
    if (x) {
      setTimeout(()=>{
      this.music.element = document.querySelector(".player__audio");

      console.log("duracion cancion mounted "+this.music.element.duration)
    },2000)  
    }
    
    
  },
  computed: {
    currentTrack() {
      //Tema actual
      return this.trackList[this.index]
    },
    trackProgress() {
      if (this.playing) {
        return this.progress  * 10  
      }
      return 0;
      
    },
    progress () {
      if (this.currentTrack.howl.duration() === 0) return 0
      return this.seek / this.currentTrack.howl.duration()
    }
       
  },
  watch:{
    playing(playing) {
      
      this.seek = this.currentTrack.howl.seek()
      let updateSeek
      if (playing) {
        updateSeek = setInterval(() => {
          this.seek = this.currentTrack.howl.seek()
          this.currentTime = this.formatTime(this.seek)
        }, 1000)
      } else {
        clearInterval(updateSeek)
      }
      this.seek= updateSeek
  },
    
  },
  methods: {
    async getTracks() {

      try {
        let response = await axios.get("https://backend-spotyrap-production.up.railway.app/tracks");
        this.trackList = response.data;
        this.createList()  

      } catch (error) {
        console.log(error)
      }
      
    },
    formatTime(secs) {
       //s if (!secs || typeof value !== "number") return "00:00"

        let min = parseInt(secs / 60),
            sec = parseInt(secs % 60)
        min = min < 10 ? "0" + min : min
        sec = sec < 10 ? "0" + sec : sec
        secs = min + ":" + sec
        
        return secs

    },
    createList () {
      this.trackList.forEach(  (track) => {
          let fileUrl = track._id
                    track.howl = new Howl({
            src: [`https://backend-spotyrap-production.up.railway.app/tracks/${fileUrl}`],
            onend: () => {
              if (this.loop) {
                this.play(this.index)
              } else {
                this.skip('next')
              }
            },
            preload:true,
            html5:true
            
          })
        })
        
    },
    play (index) {
     // index=''
      let selectedTrackIndex = this.trackList.findIndex(track => track === this.selectedTrack)
      if (typeof index === 'number') {
        //index = index

      } else if (this.selectedTrack) {
        if (this.selectedTrack != this.currentTrack) {
          this.stop()

        }
        index = selectedTrackIndex
      } else {
        index = this.index
      }
    
      let track = this.trackList[index].howl

      if (track.playing()) {
        return
      } else {
        track.play()
      }
      
      this.selectedTrack = this.trackList[index]
      this.playing = true
      this.index = index
      this.songTrack = this.currentTrack.filename
      
      setTimeout(()=>{
            
          this.duration = this.formatTime(track.duration())
            //console.log(parseInt(track.duration()))
            this.seekMax= this.currentTrack.howl.duration();
            let x = this.currentTrack.howl.duration()
            return x;
            //console.log(this.currentTrack.howl.duration())
          },3000)
    },
    pause () {
      this.currentTrack.howl.pause()
      this.playing = false
      console.log("pause")

    },
    stop () {
      this.currentTrack.howl.stop()
      this.playing = false
    },
    skip (direction) {

      let index = 0
      if (direction === "next") {
        index = this.index + 1
        if (index >= this.trackList.length) {
          index-=-1
          return;
          //index = 0
        }
      } else {
        index = this.index - 1
        if (index < 0) {
          index=0
          return;
          //index = this.trackList.length - 1
        }
      }

      this.skipTo(index)
    },
    skipTo (index) {
      if (this.currentTrack) {
        this.currentTrack.howl.stop()
        this.playing = false
      }
     this.play(index)
    },
    
  },

  created() {
    this.getTracks();
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
@import url("https://fonts.googleapis.com/css2?family=Raleway:wght@400;500&display=swap");

:root {
  --background: #e0e5ec;
  --gray: #797d7f;
}

*,
*::before,
*::after {
  box-sizing: border-box;
}

body {
  margin: 0;
  font-family: "Raleway", sans-serif;
}

img {
  max-width: 100%;
}

.container {
  background-color: var(--background);
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
}

.player {
  width: 300px;
  height: 550px;
  background-color: var(--background);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  border-radius: 12px;
  box-shadow: 9px 9px 16px rgba(163, 177, 198, 0.6),
    -9px -9px 16px rgba(255, 255, 255, 0.5);
}

.player__controls {
  display: flex;
  width: 95%;
  justify-content: space-evenly;
  align-items: center;
  margin-bottom: 25px;
}

.player__btn {
  cursor: pointer;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  box-shadow: -8px -8px 20px 0px #fff9, -6px -6px 6px 0px #fff9,
    8px 8px 20px #0001, 5px 5px 6px 0px #0001;
  color: var(--gray);
}

.player__btn:active {
  box-shadow: inset -8px -8px 20px #fff9, inset -5px -5px 6px #fff9,
    inset 8px 8px 20px #0003, inset 5px 5px 6px #0001;
}

.player__btn--small {
  min-width: 50px;
  min-height: 50px;
}

.player__title {
  font-weight: 600;
  font-size: 0.8em;
  color: #a1a1a1;
  text-transform: uppercase;
  letter-spacing: 2px;
  margin: 0;
}

.player__album {
  width: 150px;
}

.player__img {
  border-radius: 50%;
  box-shadow: 2px 2px 7px rgb(163, 177, 198), -2px -2px 7px rgb(163, 177, 198),
    -8px -8px 50px rgba(255, 255, 255, 0.8), 3px 3px 25px rgba(0, 0, 0, 0.6);
}

.player__artist {
  font-size: 1.2em;
  font-weight: 500;
  opacity: 0.5;
  margin: 20px 0px 0px 0px;
}

.player__song {
  position: relative;
  width: 100%;
  text-align: center;
  font-weight: 400;
  font-size: 1em;
  opacity: 0.5;
  margin: 0;
}

.player__level {
  width: 80%;
  -webkit-appearance: none;
  outline: none;
  border: none;
  padding: 0;
  margin-top: 40px;

}




.player__level::-webkit-slider-runnable-track {
  background-color: #d7dbdd;
  height: 6px;
  border-radius: 3px;
}

.player__level::-webkit-slider-thumb {
  -webkit-appearance: none;
  border-radius: 100%;
  background-color: #5c87fe;
  height: 18px;
  width: 18px;
  margin-top: -7px;
}

.audio-duration {
  margin-top: -30px;
  display: flex;
  width: 78%;
  font-size: 10px;
  font-weight: 600;
  color: #a1a1a1;
  letter-spacing: 1px;
}

.start {
  flex: 1;
  
}

.player__audio {
  visibility: hidden;
}

.player__btn--medium {
  min-height: 70px;
  min-width: 70px;
}

.blue {
  background-color: #5c87fe;
  color: #fff;
}

.hide {
  display: none;
}
</style>
