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
                    <q-btn icon="mic" rounded class="q-pa-md" color="primary" @click="startTxtToSpeech" dense/>
                </div>
                <div :class="!$q.screen.lt.sm?'row q-px-xl q-mt-xl':'q-mt-lg column'">
                    <q-input class="col-10" placeholder="Type your text or use the voice button" v-model="inputText" filled autogrow></q-input>
                    <q-btn class="col-2" :disable="inputText==''" label="Generate" color="secondary" no-caps @click="generateVideo"/>
                </div>

                <!-- This is video player -->
                <div class="column items-center" :class="!$q.screen.lt.sm?'q-mt-xl q-pt-xl':'q-mt-xl'"  >
                <div v-if="history.length==0">
                  <q-img src="isl2.jpg" style="height:240px;" :style="$q.screen.lt.sm?'width:290px':'width:310px'" />
                </div>
                  <q-card v-else style="height:240px;" :style="$q.screen.lt.sm?'width:290px':'width:310px'" :class="loading? 'bg-grey-3':''" flat>
                      <q-inner-loading :showing="loading">
                        <q-spinner-gears size="50px" color="primary" />
                      </q-inner-loading>
                      <div v-if="!loading" class="q-video " >
                        <video :width="$q.screen.lt.sm?'290':'310'" height="240" controls>
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
            <div v-for="i in history" :key="i">
                <!-- This is video player -->
                <div class="col-4" >
                  <q-card style="min-width:310px;min-height:240px;" width="fit-content" :class="loading? 'bg-grey-3':''" >
                      <q-inner-loading :showing="loading">
                        <q-spinner-gears size="50px" color="primary" />
                      </q-inner-loading>
                      <div v-if="!loading" class="q-video " >
                        <video width="310" height="240" controls>
                          <source :src="historyVideoUrl(i)" type="video/mp4">
                        </video>
                      </div>
                      <q-card-section class="q-pt-none text-h6">
                       {{i}}
                    </q-card-section>             
                  </q-card>
                </div>
            </div>
            </div>

          </q-tab-panel>
        </q-tab-panels>
   
        <!-- <q-btn class="txt-to-speech" @click="startSpeechToTxt">Txt to speech</q-btn> -->

    </q-card>
    </div>
</template>


<script>
const axios = require('axios');


export default {

 data() {
   return {
     inputText:"",
     history:[],
     loading:false,
     tab:"home",
     runtimeTranscription_: "",
     transcription_: [],
     lang_: "En-",
     model:'English',
     videoUrl:"https://lazt009.pythonanywhere.com/media/Videos/ww.webm",
     languages:['English','Hindi','Marathi'],
   };
 },
 methods: {
   historyVideoUrl(i){
     return "https://lazt009.pythonanywhere.com/media/Videos/"+i+".webm"
   },
   generateVideo(){
     this.loading=true;
     this.post();
     this.history.push(this.inputText)
   },
   async post(){
     console.log(this.inputText)
     try{
        const response = await axios.post('https://lazt009.pythonanywhere.com/get-video/', {
          text:this.inputText.replace(/ /g,'')
        })
        console.log(response.data)
        this.videoUrl = "https://lazt009.pythonanywhere.com/"+response.data
     } catch (error){
       console.log(error);
     }
     this.loading=false
   },
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
        this.inputText = text
        console.log("asdasd")
      });

      // end of transcription
      recognition.addEventListener("end", () => {
        this.transcription_.push(this.runtimeTranscription_);
        this.runtimeTranscription_ = "";
        recognition.stop();
      });
      recognition.start();
    },
    // startSpeechToTxt() {
    //   console.log("asdasd")
    //   // start speech to txt
    //   var utterance = new SpeechSynthesisUtterance(this.transcription_);
    //   window.speechSynthesis.speak(utterance);
    // },

  }

}
</script>

<style>

</style>