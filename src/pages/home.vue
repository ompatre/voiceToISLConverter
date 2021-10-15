<template> 
    <div style="height:93vh;display:flex;flex-direction:column;" class="q-pa-md">
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
                    <q-btn icon="mic" rounded class="q-pa-md" color="primary" @click="startTxtToSpeech" dense>
                    </q-btn>
                </div>
                <div :class="!$q.screen.lt.sm?'row q-px-xl q-mt-xl':'q-mt-lg column'">
                    <q-input class="col-10" placeholder="Type your text or use the voice button" v-model="inputText" filled autogrow></q-input>
                    <q-btn class="col-2" :disable="inputText=='' || loading" label="Generate" color="secondary" no-caps @click="generateVideo"/>
                </div>

                <!-- This is video player -->
                <div class="column items-center" :class="!$q.screen.lt.sm?'q-mt-xl q-pt-xl':'q-mt-xl'"  >
                <div v-if="history.length==0 && !loading">
                  <q-img src="isl2.jpg" style="height:240px;" :style="$q.screen.lt.sm?'width:290px':'width:310px'" />
                </div>
                  <q-card v-else style="height:240px;" :style="$q.screen.lt.sm?'width:290px':'width:310px'" :class="loading? 'bg-grey-3':''" flat>
                      <q-inner-loading :showing="loading">
                        <q-spinner-gears size="50px" color="primary" />
                      </q-inner-loading>
                      <div v-if="!loading" class="q-video " >
                        <video :width="$q.screen.lt.sm?'290':'310'" height="240" controls autoplay>
                          <source :src="videoUrl" type="video/mp4">
                        </video>
                      </div>             
                  </q-card>
                </div>
          </q-tab-panel>

          <q-tab-panel name="history">
            <div class="text-h6">History</div>
            Here you will see your previous videos
            <q-separator class="q-mt-sm"/>
            <div class="row q-gutter-md q-gutter-y-xl q-mt-sm justify-evenly">
            <div v-for="(i,ind) in history" :key="i">
                <!-- This is video player -->
                <div class="col-4" >
                  <q-card style="min-width:310px;min-height:240px;" width="fit-content" :class="loading? 'bg-grey-3':''" >
                      <q-inner-loading :showing="loading">
                        <q-spinner-gears size="50px" color="primary" />
                      </q-inner-loading>
                      <div v-if="!loading" class="q-video " >
                        <video width="310" height="240" controls>
                          <source :src="i" type="video/mp4">
                        </video>
                      </div>
                      <q-card-section class="row justify-between q-pt-none text-h6">
                        <div>
                       {{historyNames[ind]}}
                        </div>
                        <q-btn class="q-pa-none" dense flat icon="hearing" @click="startSpeechToTxt(historyNames[ind])"/>
                    </q-card-section>             
                  </q-card>
                </div>
            </div>
            </div>

          </q-tab-panel>
        </q-tab-panels>
   
    </q-card>
    </div>
</template>


<script>
const axios = require('axios');
import { QSpinnerBars } from 'quasar'

export default {
 data() {
   return {
     inputText:"",
     history:[],
     historyNames:[],
     loading:false,
     tab:"home",
     runtimeTranscription_: "",
     transcription_: [],
     lang_: "En-",
     model:'English',
     videoUrl:"https://lazt009.pythonanywhere.com/media/Videos/ww.webm",
     languages:['English','Hindi'],
   };
 },

 methods: {
   generateVideo(){
     this.loading=true;
     this.post();
     this.historyNames.push(this.inputText)
   },
   async post(){
     try{
        const response = await axios.post('https://lazt009.pythonanywhere.com/get-video/', {
          text:this.inputText
        })
        this.videoUrl = "https://lazt009.pythonanywhere.com/"+response.data
        this.history.push(this.videoUrl)
     } catch (error){
       console.log(error);
     }
     this.loading=false
   },
   getLang(){
     if (this.model=='English') return 'En-'
     else return 'Hi-'
   },
    startTxtToSpeech() {
        this.$q.loading.show({spinner: QSpinnerBars})
      // initialisation of voice
      window.SpeechRecognition =
      window.SpeechRecognition || window.webkitSpeechRecognition;
      const recognition = new window.SpeechRecognition();
      recognition.lang = this.getLang();
      recognition.interimResults = true;

      // event current voice reco word
      recognition.addEventListener("result", event => {
        const text = Array.from(event.results)
          .map(result => result[0])
          .map(result => result.transcript)
          .join("");
        this.runtimeTranscription_ = text;
        this.inputText = text
      });

      // end of transcription
      recognition.addEventListener("end", () => {
        this.transcription_.push(this.runtimeTranscription_);
        this.runtimeTranscription_ = "";
        recognition.stop();
        this.$q.loading.hide()
      });
      recognition.start();
    },
    startSpeechToTxt(text) {
      // start speech to txt
      var utterance = new SpeechSynthesisUtterance(text);
      window.speechSynthesis.speak(utterance);
    },

  }

}
</script>

<style lang="scss">

</style>