<template>
  <q-page class="constrain-more q-pa-md">
    <div class="camera-frame q-pa-md">
      <video
        v-show="!imageCaptured"
        ref="video"
        class="full-width"
        autoplay
      />
      <canvas
        v-show="imageCaptured"
        ref="canvas"
        class="full-width"
        height="240"
      />
    </div>
    <div class="text-center q-pa-md">
      <q-btn
        v-if="hasCameraSupport"
        @click="captureImage"
        round 
        color="black"
        size="lg"
        icon="eva-camera" />

        <q-file
          v-else
          v-model="imageUpload"
          label="Choose an image"
          accept=" image/*"
          @input="captureImageFallback"
          outlined
        >
          <template v-slot:prepend>
            <q-icon name="eva-attach-outline" />
          </template>
        </q-file>
    </div>
    <div class="row justify-center q-ma-md">
      <q-input 
        v-model="post.caption"
        class="col col-sm-8"
        label="Caption"
        />
    </div>
    <div class="row justify-center q-ma-md">
      <q-input 
        v-model="post.location"
        :loading="locationLoading"
        class="col col-sm-8"
        label="Location"
      >
        <template v-slot:append>
          <q-btn
            v-if="!locationLoading && locationSuported"
            @click="getLocation"
            round
            dense
            flat
            icon="eva-navigation-2-outline" />
        </template>
      </q-input>
    </div>
    <div class="row justify-center q-ma-lg">
      <q-btn
        unelevated
        rounded
        color="primary"
        label="Post Image" />
    </div>
  </q-page>
</template>

<script>
import { uid } from 'quasar'
require('md-gum-polyfill')

export default {
  name: 'PageCamera',
  data() {
    return {
      post: {
        id: uid(),
        caption: "",
        location: "",
        date: Date.now(),
        photo: null,
      },
      imageCaptured: false,
      imageUpload: [],
      hasCameraSupport: true,
      locationLoading: false,
    }
  },
  computed: {
    locationSuported(){
      if('geolocation' in navigator) return true
      return false
    }
  },
  methods: {
    initCamera(){
      navigator.mediaDevices.getUserMedia({
        video: true
      }).then(strean => {
        this.$refs.video.srcObject = strean
      }).catch( error => {
        this.hasCameraSupport = false
      })
    },
    
    captureImage() {
      let video = this.$refs.video
      let canvas = this.$refs.canvas
      canvas.width = video.getBoundingClientRect().width
      canvas.height = video.getBoundingClientRect().height
      let context = canvas.getContext('2d')
      context.drawImage(video, 0, 0, canvas.width, canvas.height)
      this.imageCaptured = true
      this.post.photo = this.dataURItoBlob(canvas.toDataURL())
      this.disableCamera()
    },

    captureImageFallback( file ) {
      this.post.photo = file
      let canvas = this.$refs.canvas
      let context = canvas.getContext('2d')
      var reader = new FileReader()
      reader.onload = (event) => {
          var img = new Image()
          img.onload = () => {
            canvas.width = img.width
            canvas.height = img.height
            context.drawImage(img, 0, 0)
            this.imageCaptured = true
          }
          img.src = event.target.result
      }
      reader.readAsDataURL(file)
    },

    disableCamera() {
      this.$refs.video.srcObject.getVideoTracks().forEach(track => {
        track.stop()
      })
    },

    dataURItoBlob(dataURI) {
      var byteString = atob(dataURI.split(',')[1]);
      var mimeString = dataURI.split(',')[0].split(':')[1].split(';')[0]
      var ab = new ArrayBuffer(byteString.length);
      var ia = new Uint8Array(ab);
      for (var i = 0; i < byteString.length; i++) {
          ia[i] = byteString.charCodeAt(i)
      }
      var blob = new Blob([ab], {type: mimeString})
      return blob;
    },

    getLocation(){
      this.locationLoading = true
      navigator.geolocation.getCurrentPosition(position => {
        this.getCityAndCountry(position)
      }, err => {
        this.locationError()
      }, { timeout: 7000 })
    },

    getCityAndCountry(position){
      let apiUrl = `https://geocode.xyz/${position.coords.latitude},${position.coords.longitude}?geoit=json`
      this.$axios.get(apiUrl)
        .then(result => {
          this.locationSuccess(result)
        })
        .catch(err => {
          this.locationError()
        })
    },

    locationSuccess(result){
      this.post.location = result.data.city
      if(result.data.country){
        this.post.location += `, ${result.data.country}`
      }
      this.locationLoading = false
    },

    locationError() {
      this.$q.dialog({
        title: 'Error',
        message: 'Could not find location.'
      })
      this.locationLoading = false
    }
  },
  mounted() {
    this.initCamera()
  },
  beforeDestroy() {
    if(this.hasCameraSupport){
      this.disableCamera()
    }
  }
}
</script>

<style lang="sass" scoped>

</script>