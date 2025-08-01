<template>
  <div class="container mt-4">
    <h1 class="text-center mb-4">Google Sheets Flashcards</h1>
    <hr>

    <!-- toogle tool -->
    <div v-if="groups.length > 0" class="mb-4 text-center toggle-container">
      <template v-if="!flashcardMode">
        <div class="form-check form-check-inline flag-container">
          <input
            type="radio"
            value="japanese"
            v-model="flashcardModeSetting"
            class="form-check-input"
          />
          <label for="showJapanese" class="form-check-label flag">🇯🇵</label>
          <!-- <label class="form-check-label" v-html="japaneseEmoji"></label> -->
        </div>
        <div class="form-check form-check-inline flag-container">
          <input
            type="radio"
            value="english"
            v-model="flashcardModeSetting"
            class="form-check-input"
          />
          <label for="showEnglish" class="form-check-label flag">🇬🇧</label>
        </div>
        |
        <div class="form-check form-check-inline">
          <input
            type="radio"
            :value="true"
            v-model="displayingAll"
            class="form-check-input"
          />
          <label for="showJapanese" class="form-check-label">All</label>
        </div>
        <div class="form-check form-check-inline">
          <input
            type="radio"
            :value="false"
            v-model="displayingAll"
            class="form-check-input"
          />
          <label for="showEnglish" class="form-check-label">Marked only</label>
        </div>
      </template>
      
      
    </div>

    <!-- group looping -->
    <div v-if="groups.length > 0 && !flashcardMode" class="row gx-2 gy-2 mb-3">
      <div
        v-for="(group, index) in computedGroups"
        :key="index"
        class="col-6"
      >
        <div 
          :style="{
              backgroundColor: group.name === 'unchecked' ? 'LightSlateGray' : '',
              opacity: group.counter === 0 ? 0.5 : 1
          }"
          class="card shadow" 
          @click="startFlashcards(group)" 
          style="cursor: pointer;"
        >
          <div class="card-body">
            <h5 class="card-title text-center">{{ group.name }}</h5>
            <hr>
            <section class="flex-container" style="width: 80%; margin: auto;">
              <small><i class="fa-solid fa-list"></i> <strong>{{ group.words.length }}</strong></small>
              <small v-if="group.name !== 'unchecked'  && group.name !== 'Marked Words'"><i class="fa-solid fa-bookmark"></i> <strong>{{ group.words.filter(word => word.marked)?.length }}</strong></small>
              <small v-if="group.name !== 'unchecked' && group.name !== 'Marked Words'"><i class="fa-regular fa-clock"></i> <strong>{{ group.counter }}</strong></small>
            </section>
          </div>
        </div>
      </div>
    </div>


    <div v-if="groups?.length == 0" class="text-center fixed left-1/2 top-1/4 transform -translate-x-1/2 -translate-y-1/2">
      <div class="relative w-[22.5vw] aspect-square m-8">
          <div class="absolute w-[80%] h-[80%] border-4 border-indigo-300 rounded-full animate-[ripple_1.5s_infinite] left-1/2 top-1/2 text-center fixed left-1/2 top-1/2 transform -translate-x-1/2 -translate-y-1/2"></div>
          <div class="absolute w-[80%] h-[80%] border-4 border-indigo-300 rounded-full animate-[ripple_1.5s_infinite] [animation-delay:0.5s] left-1/2 top-1/2 text-center fixed left-1/2 top-1/2 transform -translate-x-1/2 -translate-y-1/2"></div>
      </div>
    </div>

    <!-- Flashcard View -->
    <div v-if="flashcardMode" class="flashcard text-center p-4 shadow bg-light rωounded">
      <div class="flex-container">
        <span 
          class="badge text-dark p-2"
          :style="{
            backgroundColor: `rgba(255, 193, 7, ${Math.min(1, currentWord.counter / 25)})`
          }"
        >
          <i class="fa-solid fa-flag"></i> x {{ currentWord.counter }}
        </span>
        <h4 class="mb-4 text-primary">
          {{ currentGroup.name }}: {{currentWordIndex+1}} / {{ shuffledWords.length }} 
        </h4>
        <button class="ms-3 btn btn-danger" @click="exitFlashcards"><i class="fas fa-sign-out"></i></button>
      </div>

      <template v-if="currentGroup.name !== 'unchecked'">
        <!-- main word  -->
        <div style="background-color: lightgray;" class="rounded my-4 py-2" @click="showTranslation = !showTranslation">
          <div class="display-6 mb-6">
            {{ flashcardModeSetting === 'english' ? currentWord.it : (showTranslation ? currentWord.it : '???') }}
          </div>
          <div class="h5 mb-4">
            {{ flashcardModeSetting === 'english' ? (showTranslation ? currentWord.en : '???') : currentWord.en }}
          </div>
          <div class="h6 mb-4 text-secondary" v-if="currentWord.exampleSetence">
            {{currentWord.exampleSetence}}
          </div>
          <div class="h6 mb-4 text-secondary" v-if="currentWord.exampleSetenceEn">
            {{ flashcardModeSetting === 'english' ? (showTranslation ? currentWord.exampleSetenceEn : '???') : currentWord.exampleSetenceEn }}
          </div>
        </div>
  
        <!-- buttons  -->
        <div>
          <button class="btn btn-primary me-2" @click="backWord" :disabled="currentWordIndex === 0">
            <i class="fa-solid fa-left-long"></i>
          </button>
  
          <button 
            class="btn btn-secondary me-2" 
            @click="editWord"
          >
            <i class="fa-solid fa-pen-to-square"></i>
          </button>
  
          <button 
            class="btn btn-warning me-2" 
            @click="toggleFlag(currentWord)"
          >
            <i :class="currentWord.flag ? 'fa-solid fa-flag' : 'fa-regular fa-flag'"></i>
          </button>
  
          <button 
            class="btn btn-warning me-2" 
            @click="toggleMark(currentWord)"
          >
            <i :class="currentWord.marked ? 'fa-solid fa-bookmark' : 'fa-regular fa-bookmark'"></i>
          </button>

          <button class="btn btn-info me-2" @click="speakWord(currentWord)">
              <i class="fa-solid fa-volume-high"></i>
          </button>

  
          <button v-if="currentWordIndex !== shuffledWords.length -1" class="btn btn-success me-2" @click="nextWord">
            <i class="fa-solid fa-right-long"></i>
          </button>
         <button v-else class="btn btn-danger" @click="finishRound">Done</button>
          
        </div>
      </template>

      <!-- editing page -->
      <template v-else>
        <!-- main  -->
        <div class="container py-4">
          <div class="mb-3">
            <label for="jpInput" class="form-label h5">🇯🇵</label>
            <input
              id="jpInput"
              type="text"
              class="form-control form-control-lg"
              placeholder="Enter Japanese word"
              v-model="currentWord.updatedIt"
            />
          </div>
          <div class="mb-3">
            <label for="enInput" class="form-label h5">🇬🇧</label>
            <input
              id="enInput"
              type="text"
              class="form-control form-control-lg"
              placeholder="Enter English word"
              v-model="currentWord.updatedEn"
            />
          </div>
        </div>

  
        <!-- buttons  -->
        <div>
          <button class="btn btn-danger me-2" @click="deleteRow">
            <i class="fa-solid fa-trash-can"></i>
          </button>
  
          <button class="btn btn-success me-2" @click="finishChecking">
            <i class="fa-solid fa-check"></i>
          </button>
          
        </div>
      </template>
    </div>

  </div>

</template>



<script>
import twemoji from "twemoji";

export default {
  data() {
    return {
      groups: [], // Stores processed group data
      apiKey: process.env.VUE_APP_API_KEY,
      sheetId: process.env.VUE_APP_SHEET_ID,

      flashcardMode: false, // Toggles flashcard mode
      currentGroup: null, // Current group in flashcard mode
      currentWordIndex: 0, // Index of the current word
      showTranslation: false, // Whether to show the translation
      flashcardModeSetting: "english", // Default to hiding Japanese
      shuffledWords: [],
      displayingAll: true,

      targetCell: "A555", // Default target cell
      cellValue: "Value from Vue", // Default value
      responseMessage: "", // To store the API response

      isFirstRound: false,
      hasEdited: false,

      baseUrl : 'https://script.google.com/macros/s/AKfycbwMWy5Ighh9YSpEEwz1FTnoKruBT36_1G17z6hNhzycJfiRa8YRytigsho71F9ZHmxqzA/exec'
    };
  },
  computed: {
    currentWord() {
        if (!this.shuffledWords || this.shuffledWords.length === 0) {
            return { it: "No words", en: "No words" };
        }
        return this.shuffledWords[this.currentWordIndex];
    },
    filteredGroup() {
      // Create a new array of groups where words are filtered to include only marked ones
      return this.groups.map(group => {
        return {
          ...group,
          words: group.words.filter(word => word.marked) // Include only marked words
        };
      }).filter(group => group.words.length > 0); // Remove groups with no marked words
    },

    filteredWords() {
      return this.groups.flatMap(group => group.words).filter(word => word.marked);
    },

    mistakesInThePast() {
      return this.groups.flatMap(group => group.words).filter(word => word.counter !== 0 );
    },

    computedGroups() {
      const markedWordsOnly = {
        name: "Marked Only",
        words: this.filteredWords,
        counter: '?', // No counter needed
      };


      const mistakesWordsOnly = {
        name: "Mistakes Only",
        words: this.mistakesInThePast,
        counter: '?', // No counter needed
      };
      

      return [mistakesWordsOnly,markedWordsOnly, ...this.groups]; // Add to the start of groups
    },
    

    japaneseEmoji() {
      return twemoji.parse("🇯🇵"); // Japan flag emoji
    },
    englishEmoji() {
      return twemoji.parse("🇺🇸"); // USA flag emoji
    },
  },
  methods: {
    fetchData() {
        console.log(`getting reload`);
        this.groups = [];

        const url = `${this.baseUrl}?callback=jsonpCallback&action=fetchData`;
        console.log(url);

        // Define the callback function globally
        window.jsonpCallback = (data) => {
            console.log("API Response (fetchData):", data);
            if (data.success) {
                this.processData(data.data); // Process and store data
            } else {
                console.error("Error fetching data:", data.message);
            }
        };

        // Dynamically add a <script> tag to call the JSONP API
        const script = document.createElement("script");
        script.src = url; // Set the API URL
        script.async = true; // Load asynchronously
        document.body.appendChild(script);

        // Clean up the <script> tag after the request
        script.onload = () => {
            document.body.removeChild(script); // Remove the script tag
        };
    },
    processData(values) {
        let groups = [];
        let group = { name: "", words: [] };
        let unchecked = { name: "unchecked", words: [] }; // Group for rows with `uncheckedFlag` == 1

        // Helper function to process a word and add it to the appropriate group
        const addWordToGroup = (row, targetGroup) => {
            if (row.it && row.en) {
                targetGroup.words.push({
                    it: row.it,
                    en: row.en,
                    marked: row.marked == 1,  // Ensure boolean conversion
                    counter: row.counter ? Number(row.counter) : 0,
                    exampleSetence: row.exampleSetence || false,
                    exampleSetenceEn: row.exampleSetenceEn || false,
                    audioPath: row.audioPath,

                    updatedIt: row.it,
                    updatedEn: row.en,

                    
                });
            }
        };

        for (let row of values) {
            // Handle "unchecked" words
            if (row.uncheckedFlag == 1) {
                addWordToGroup(row, unchecked);
                continue;
            } 
            // Handle group headers (groupName)
            else if (row.en === "groupName") {
                if (group.name) groups.push(group);
                group = { name: row.it, words: [], counter: row.counter ? Number(row.counter) : 0 };
                continue;
            } 
            // Handle regular words
            else {
                addWordToGroup(row, group);
            }
        }

        if (group.name) groups.push(group);
        if (unchecked.words.length > 0) groups.push(unchecked); // Add "unchecked" group if it has words

        this.groups = null;
        this.groups = groups;
        console.log("Processed Data:", this.groups); // Debugging
        console.log('------------');
        console.log(this.computedGroups)
    },
    shuffleArray(array) {
      // const shuffled = [...array];
      const shuffled = array.map(item => ({ ...item, flag: false }));

      for (let i = shuffled.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]];
      }
      return shuffled;
    },

    startFlashcards(group) {
      this.isFirstRound = true;
      this.hasEdited = false;
      // Shuffle the group's words and store them in shuffledWords
      // this.shuffledWords = this.shuffleArray(group.words);

      this.shuffledWords = this.shuffleArray(
        this.displayingAll ? group.words : group.words.filter(word => word.marked)
      );

      this.flashcardMode = true;
      this.currentGroup = group;
      this.currentWordIndex = 0;
      this.showTranslation = false;
    },

    nextWord() {
      if (this.shuffledWords.length > 0) {
        this.currentWordIndex =
          (this.currentWordIndex + 1) % this.shuffledWords.length;
        this.showTranslation = false;
      }
    },
    backWord() {
      if (this.shuffledWords.length > 0) {
        this.currentWordIndex =
          (this.currentWordIndex - 1) % this.shuffledWords.length;
        this.showTranslation = false;
      }
    },

    finishRound() {
      if(this.isFirstRound && this.currentGroup.name !== 'unchecked'){
        this.currentGroup.counter++
        this.incrementWord({it: this.currentGroup.name, en: 'groupName'})
      }
      this.isFirstRound = false
      // Filter flagged words from shuffledWords
      const flaggedWords = this.shuffledWords.filter(word => word.flag);


      if(this.currentGroup.name == 'unchecked'){
        this.groups = []
        this.exitFlashcards();
        setTimeout(() => {
          this.fetchData();
        }, 1500); // 2000 milliseconds = 2 seconds
        
        return
      }

      if (flaggedWords.length === 0) {
        if(this.hasEdited){
          this.groups = []
          this.exitFlashcards();
          setTimeout(() => {
            this.fetchData();
          }, 1500); // 2000 milliseconds = 2 seconds
          
          return
        }
        return this.exitFlashcards();
      }

      // Shuffle the flagged words and reset state
      this.shuffledWords = this.shuffleArray(flaggedWords);
      this.currentWordIndex = 0;
      this.showTranslation = false;
    },

    toggleMark(word) {
      let checkFlag = word.flag
      // word.flag = !word.flag;
      word.marked = !word.marked;
      if (!checkFlag) this.toggleFlag(word)

      // Build the API URL with the action parameter for toggleMark
      const url = `${this.baseUrl}?callback=jsonpCallback&action=toggleCol&targetCol=5&english=${encodeURIComponent(
        word.en
      )}&italian=${encodeURIComponent(word.it)}`;

      console.log(url)

      // Define the callback function globally
      window.jsonpCallback = (data) => {
        console.log("API Response (toggleMark):", data);
      };


      // Dynamically add a <script> tag to call the JSONP API
      const script = document.createElement("script");
      script.src = url; // Set the API URL
      script.async = true; // Load asynchronously
      document.body.appendChild(script);

      // Clean up the <script> tag after the request
      script.onload = () => {
        document.body.removeChild(script); // Remove the script tag
      };

    },

    toggleFlag(word){
      word.flag = !word.flag
      if(!word.flag) return
      this.incrementWord(word)
    },
    finishChecking(){

      // Build the API URL with the action parameter for toggleMark
      const restURL = `?callback=jsonpCallback&action=finishChecking&english=${encodeURIComponent(this.currentWord.en)}&italian=${encodeURIComponent(this.currentWord.it)}&updatedEnglish=${encodeURIComponent(this.currentWord.updatedEn)}&updatedItalian=${encodeURIComponent(this.currentWord.updatedIt)}`
      const url = `${this.baseUrl}${restURL}`; 

      // Define the callback function globally
      window.jsonpCallback = (data) => {
        console.log("API Response (toggleMark):", data);
      };

      // Dynamically add a <script> tag to call the JSONP API
      const script = document.createElement("script");
      script.src = url; // Set the API URL
      script.async = true; // Load asynchronously
      document.body.appendChild(script);

      // Clean up the <script> tag after the request
      script.onload = () => {
        document.body.removeChild(script); // Remove the script tag
      };

      if(this.currentWordIndex !== this.shuffledWords.length -1){
        this.nextWord()
      }else{
        this.finishRound()
      }
    },

    deleteRow(){
      const isConfirmed = confirm("Are you sure you want to delete this item?");
      if(!isConfirmed) return
      // Build the API URL with the action parameter for toggleMark
      const url = `${this.baseUrl}?callback=jsonpCallback&action=deleteRow&english=${encodeURIComponent(this.currentWord.en)}&italian=${encodeURIComponent(this.currentWord.it)}`; 

      // Define the callback function globally
      window.jsonpCallback = (data) => {
        console.log("API Response (toggleMark):", data);
      };

      // Dynamically add a <script> tag to call the JSONP API
      const script = document.createElement("script");
      script.src = url; // Set the API URL
      script.async = true; // Load asynchronously
      document.body.appendChild(script);

      // Clean up the <script> tag after the request
      script.onload = () => {
        document.body.removeChild(script); // Remove the script tag
      };

      if(this.currentWordIndex !== this.shuffledWords.length -1){
        this.nextWord()
      }else{
        this.finishRound()
      }
    },

    editWord(){
      const isConfirmed = confirm("Do you want edit this word?");
      if(!isConfirmed) return
      
      this.hasEdited = true;

      this.currentWord.flag = false;
      // this.currentWord.un = false;
      // Build the API URL with the action parameter for toggleMark
      const url = `${this.baseUrl}?callback=jsonpCallback&action=toggleCol&targetCol=6&english=${encodeURIComponent(this.currentWord.en)}&italian=${encodeURIComponent(this.currentWord.it)}`;

      // Define the callback function globally
      window.jsonpCallback = (data) => {
        console.log("API Response (toggleMark):", data);
      };

      // Dynamically add a <script> tag to call the JSONP API
      const script = document.createElement("script");
      script.src = url; // Set the API URL
      script.async = true; // Load asynchronously
      document.body.appendChild(script);

      // Clean up the <script> tag after the request
      script.onload = () => {
        document.body.removeChild(script); // Remove the script tag
      };

      if(this.currentWordIndex !== this.shuffledWords.length -1){
        this.nextWord()
      }else{
        this.finishRound()
      }
    },

    incrementWord(word) {
      word.counter++
      // Build the API URL with the action parameter for increment
      const url = `${this.baseUrl}?callback=jsonpCallback&action=increment&english=${encodeURIComponent(word.en)}&italian=${encodeURIComponent(word.it)}`;
      console.log('-------- in crementing word');
      console.log(url);
      console.log('-------');

      // Define the callback function globally
      window.jsonpCallback = (data) => {
        console.log("API Response (increment):", data);
      };

      // Dynamically add a <script> tag to call the JSONP API
      const script = document.createElement("script");
      script.src = url; // Set the API URL
      script.async = true; // Load asynchronously
      document.body.appendChild(script);

      // Clean up the <script> tag after the request
      script.onload = () => {
        document.body.removeChild(script); // Remove the script tag
      };
    },
    exitFlashcards() {
      this.flashcardMode = false;
      this.currentGroup = null;
      this.currentWordIndex = 0;
      this.showTranslation = false;
      this.shuffledWords = [];

      this.fetchData();
    },
    speakWord(word) {
        // if (!word.it) return;

        // const utterance = new SpeechSynthesisUtterance(word.it);
        // utterance.lang = "it-IT"; // Set language to Italian
        // utterance.rate = 0.9; // Adjust speed if needed
        // utterance.pitch = 1; // Adjust pitch if needed
        // speechSynthesis.speak(utterance);

        if (!word.audioPath) return;

        const audio = new Audio(word.audioPath);
        audio.play();
    },
  },
  mounted() {
    console.clear()
    this.fetchData();
  },
  watch: {
    flashcardMode(newVal) {
      const htmlBodyStyles = document.documentElement.style;

      if (newVal) {
        htmlBodyStyles.height = "100%";
        htmlBodyStyles.margin = "0";
        htmlBodyStyles.overflow = "hidden";
      } else {
        htmlBodyStyles.height = "";
        htmlBodyStyles.margin = "";
        htmlBodyStyles.overflow = "";
      }
    },
  }
};


</script>

<style>
#app {
  height: 100vh; /* Ensures the app takes the full viewport height */
  display: flex; /* Optional: For centering or flex layouts */
  flex-direction: column; /* Optional: Ensures children stack vertically */
}

.toggle-container{
  display: flex;
  justify-content: space-around !important;
  align-items: center;
}

.flag-container{
  display: flex !important;
  align-items: center;
  gap: 5px;
  
}

.form-check-inline{
  margin-right: unset !important;
}

.flag{
  font-size: 2em;
}

.flex-container{
  display: flex;
  justify-content: space-around;
  align-items: center;
  margin-bottom: 10px;
}

.text-center{
  justify-content: center;
}

.flex-container span{
  font-size: 1em;
}

.flex-container h4{
  margin-bottom: unset !important;
}

.card-title{
  margin-bottom: unset !important;
}

.card-body{
  padding: 10px !important;
}

.card-body hr{
  margin: .5em 0;
}

.flashcard{
  border: 1px solid lightslategray;
}


@keyframes ripple {
    from {
        transform: translate(-50%,-50%) scale(0);
        opacity: 1;
    }
    to {
        transform: translate(-50%,-50%) scale(1.5);
        opacity: 0;
    }
}
</style>