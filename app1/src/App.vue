<template>
  <!-- <div id="app">
    {{ nextImage }}
  </div> -->

  <img :src="`${nextImage}`" />


  <!-- <div id="app">
    {{ timerCount }}
  </div> -->

</template>

<script>
import axios from 'axios';

export default {
  name: 'App',
  components: {
  },
  data() {
    return {
      objectids: [], //List of object ids
      timerCount: 10, //Timer counter
      objectListIndexCounter: 0, //Object list indexer counter
      objectIDToCheck: "", //This is the ID of the image which will be verified
      objectUrlToCheck: "",
      foundNext: false,
      nextImage: "https://cdn-icons-png.flaticon.com/512/2748/2748558.png",
      dispImg: "https://cdn-icons-png.flaticon.com/512/2748/2748558.png",
      objectIDLoadStatus: false //boolean to let us know if the list of object ids have been loaded
    }
  },
  mounted() {
    axios
      .get('https://collectionapi.metmuseum.org/public/collection/v1/objects')
      .then(response => (this.objectids = response.data.objectIDs))
  },
  watch: {
    timerCount: {
      async handler(value) {
        if (value > -1) {
          setTimeout(() => {
            this.timerCount--;
          }, 1000);

          //In addition to counting down 10s, check if the object ids have been loaded
          if (this.objectids.length > 0 & this.objectIDLoadStatus === false) {
            //Object IDs are loaded
            this.objectIDLoadStatus = true;
          }

          //If object IDs have been loaded, check for valid image URLs
          if (this.objectIDLoadStatus & this.foundNext === false) {
            this.objectIDToCheck = this.objectids[this.objectListIndexCounter];

            //Get the object details and check if this object id has a valid image url
            try {
              await axios
                .get('https://collectionapi.metmuseum.org/public/collection/v1/objects/' + this.objectIDToCheck)
                .then(response => (this.objectUrlToCheck = response.data.primaryImageSmall));
            } catch (err) {
              console.log(err);
            }

            //validate if the url for the image is valid or not  
            if (this.objectUrlToCheck.length > 0) {
              this.foundNext = true;
              this.objectListIndexCounter++;
            }

            //If within 10s the next valid image is not found
            if (this.foundNext === false) {
              this.objectListIndexCounter++;
            } else {
              //The current URL is valid, display image
              this.dispImg = this.objectUrlToCheck;
            }
          }
        }
        else {
          //reset timer back to 10s
          this.timerCount = 10;

          if (this.foundNext === true) {
            this.nextImage = this.dispImg;
            this.foundNext = false;
          }
        }
      },
      immediate: true // This ensures the watcher is triggered upon creation
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
