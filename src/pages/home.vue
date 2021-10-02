<template> 
    <div style="height:93vh;display:flex;flex-direction:column" class="q-pa-md">
        
    <q-card style="flex-grow:1" class="q-a-md">
              <q-tabs
                v-model="tab"
                dense
                class="bg-grey-3 text-grey-7"
                active-color="primary"
                indicator-color="primary"
                align="justify"
                >
          <q-tab name="home" label="Home" no-caps/>
          <q-tab name="history" label="History" no-caps />
        </q-tabs>

        <q-tab-panels v-model="tab" animated>
          <q-tab-panel name="home">
                <div class="row justify-center q-py-md q-gutter-x-xl"> 
                    <q-select v-model="model" :options="languages" class="q-px-md col-5"/>
                    <q-btn icon="mic" rounded class="q-pa-md" color="primary" @click="startTxtToSpeech" dense/>
                </div>
                <div>
                    <q-input class="q-px-xl q-mt-lg" placeholder="Type your text or use the voice button" v-model="transcription_[transcription_.length-1]" filled autogrow></q-input>
                </div>

                <div class="column items-center q-pa-md q-mt-xl">
                    <q-video src="https://www.youtube.com/embed/qcdivQfA41Y" class="video" style="width:300px;height:300px;"/>
                </div>
                
          </q-tab-panel>

          <q-tab-panel name="history">
            <div class="text-h6">History</div>
            Here you will see your previous videos
            <q-separator class="q-mt-sm"/>
            <div v-for="i in transcription_" :key="i" class="row q-gutter-y-md q-mt-lg">
                <q-card class="col-8 col-sm-4  q-ma-md">
                    <q-video src="https://www.youtube.com/embed/qcdivQfA41Y" />

                    <q-card-section class="q-pt-none text-h6">
                       {{i}}
                    </q-card-section>
                </q-card>
            </div>

          </q-tab-panel>
        </q-tab-panels>
   
        <!-- <q-btn class="txt-to-speech" @click="startSpeechToTxt">Txt to speech</q-btn> -->

    </q-card>
    </div>
</template>


<script>


export default {

 data() {
   return {
     tab:"home",
     runtimeTranscription_: "",
     transcription_: ['do it',"here i go", "timewatch"],
     lang_: "En-",
     model:'English',
     languages:['English','Hindi','Marathi'],
     sources: [
        {
          src: 'http://www.peach.themazzone.com/durian/movies/sintel-2048-surround.mp4',
          type: 'video/mp4'
        }
     ],
        playerOptions: {
          // videojs options
          muted: true,
          language: 'en',
          playbackRates: [0.7, 1.0, 1.5, 2.0],
          sources: [{
            type: "video/mp4",
            src: "https://cdn.theguardian.tv/webM/2015/07/20/150716YesMen_synd_768k_vp8.webm"
          }],
          poster: "/static/images/author.jpg",
        }
   };
 },
 methods: {
    startTxtToSpeech() {
      // initialisation of voice
      window.SpeechRecognition =
      window.SpeechRecognition || window.webkitSpeechRecognition;
      const recognition = new window.SpeechRecognition();
      recognition.lang = this.lang_;
      recognition.interimResults = true;

      // event current voice reco word
      recognition.addEventListener("result", event => {
        const text = Array.from(event.results)
          .map(result => result[0])
          .map(result => result.transcript)
          .join("");
        this.runtimeTranscription_ = text;
      });

      // end of transcription
      recognition.addEventListener("end", () => {
        this.transcription_.push(this.runtimeTranscription_);
        this.runtimeTranscription_ = "";
        recognition.stop();
      });
      recognition.start();
    },
    startSpeechToTxt() {
      // start speech to txt
      var utterance = new SpeechSynthesisUtterance(this.transcription_);
      window.speechSynthesis.speak(utterance);
    },

    onPlayerPlay(player) {
        // console.log('player play!', player)
      },
      onPlayerPause(player) {
        // console.log('player pause!', player)
      },
      // ...player event

      // or listen state event
      playerStateChanged(playerCurrentState) {
        // console.log('player current update state', playerCurrentState)
      },

      // player is ready
      playerReadied(player) {
        console.log('the player is readied', player)
        // you can use it to do something...
        // player.[methods]
      }
  }

}
</script>

<style>

</style>