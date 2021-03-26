<template>
  <q-page class="constrain-more q-pa-md">
    <div class="camera-frame q-pa-md">
      <video 
      v-show="!imageCapture"
      ref="video"
      class="full-width"
      autoplay
      />
      <canvas
        v-show="imageCapture"
        ref="canvas"
        class="full-width"
        height="240"
      />
    </div>
    <div class="text-center q-pa-md">
      <q-btn v-if="hasCameraSupport" :disable="imageCapture" @click="captureImage" round color="grey-10" size="lg" icon="eva-camera" />
      <q-file outlined 
      v-else 
     
      v-model="imageUpload"
      accept="image/*"
       @input="captureImageFallback"
      label="Choose an Image">
        <template v-slot:prepend>
          <q-icon name="eva-attach-outline" />
        </template>
      </q-file>
    </div>
    <div class="row justify-center q-ma-md">
      <q-input v-model="post.caption" label="Caption *" class="col col-sm-6" dense />
    </div>
    <div class="row justify-center q-ma-md">
      <q-input :loading="locationLoading" v-model="post.location" label="Location" class="col col-sm-6" dense 
      >
      <template v-slot:append>
          <q-btn v-if="!locationLoading" @click="getLocation" round dense flat icon="eva-navigation-2-outline" />
        </template>
      </q-input>
    </div>
    <div class="row justify-center q-mt-lg">
    <q-btn :disable="!post.caption || !post.photo" @click="addPost" unelevated rounded color="primary" label="Post Image" />
    </div>
  </q-page>
</template>

<script>
import { uid } from 'quasar'
import axios from 'axios'
require('md-gum-polyfill')

export default {
  name: 'PageCamera',
  data() {
    return {
      post: {
        id: uid(),
        caption: '',
        location: '',
        photo: null,
        date: Date.now()
        },
      imageCapture: false,
      imageUpload: [],
      hasCameraSupport: true,
      locationLoading: false
    }
  },
  mounted() {
  this.initCamera()
  console.log(navigator, 'A')
  },
  beforeDestroy() {
    if (this.hasCameraSupport) {
      this.disableCamera()
    }
  },
  // computed: {
  //   locationSupported(){
  //     if ('geolocation' in navigator) return
  //     true
  //     return false
  //   }
  // },
  methods: {
    initCamera() {
      navigator.mediaDevices.getUserMedia({
        video:true
      }).then(stream => {
        this.$refs.video.srcObject = stream
      }).catch(error => {
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
      this.imageCapture = true
      this.post.photo = this.dataURItoBlob(canvas.toDataURL())
      this.disableCamera()
    },
    captureImageFallback(file) {
      console.log(file, 'FILE')
      
      let canvas = this.$refs.canvas
      let context = canvas.getContext('2d')
      this.post.photo = file
      var reader = new FileReader()
      reader.onload = event => {
          var img = new Image()
          img.onload = () => {
              canvas.width = img.width
              canvas.height = img.height
              context.drawImage(img,0,0)
              this.imageCapture = true
          }
          img.src = event.target.result
      }
      reader.readAsDataURL(file)
    },
    disableCamera() {
      console.log('jalan')
      this.$refs.video.srcObject.getVideoTracks().forEach(track => {
        track.stop()
      })
    },
     dataURItoBlob(dataURI) {
        // convert base64 to raw binary data held in a string
        // doesn't handle URLEncoded DataURIs - see SO answer #6850276 for code that does this
        var byteString = atob(dataURI.split(',')[1]);

        // separate out the mime component
        var mimeString = dataURI.split(',')[0].split(':')[1].split(';')[0]

        // write the bytes of the string to an ArrayBuffer
        var ab = new ArrayBuffer(byteString.length);

        // create a view into the buffer
        var ia = new Uint8Array(ab);

        // set the bytes of the buffer to the correct values
        for (var i = 0; i < byteString.length; i++) {
          ia[i] = byteString.charCodeAt(i);
        }

        // write the ArrayBuffer to a blob, and you're done
        var blob = new Blob([ab], {
          type: mimeString
        });
        return blob;

    },
    getLocation() {
      this.locationLoading = true
        navigator.geolocation.getCurrentPosition(position => {
          this.getCityandCountry(position)
          console.log('position', position)
        }), err => {
          this.locationError()
        }, {
          timeout: 7000
        }

      },
      getCityandCountry(position) {
        let apiUrl = `http://geocode.xyz/${ position.coords.latitude },${ position.coords.longitude }?json=1`
        axios.get(apiUrl).then(result => {
          console.log(result, 'LOC')
          this.getLocationSucces(result)
        }), err => {
          this.locationError()
        }
      },
    getLocationSucces(result) {
      this.post.location = result.data.city
      if (result.data.country) {
        this.post.location += `, ${ result.data.country}`
      }
      this.locationLoading = false
    },
    locationError(){
      this.$q.dialog({
        title: 'Error',
        message: 'Could not find your location'
      })
      this.locationLoading = false
    },
    addPost(){
      this.$q.loading.show()
      let formData = new FormData()
      formData.append('id', this.post.id)
      formData.append('caption', this.post.caption)
      formData.append('location', this.post.location)
      formData.append('date', this.post.date)
      formData.append('file', this.post.photo, this.post.id + '.png')
      axios.post(`${ process.env.API}/createpost`, formData).then(
        response => {
          console.log('response', response)
          this.$router.push('/')
          this.$q.notify({
            message: 'Post Created!',
            actions: [{
              label: 'Dismiss',
              color: 'white'
            }]
          })
          this.$q.loading.hide()
        }).catch(err => {
          console.log('err', err)
          this.$q.dialog({
            title: 'Error',
            message: 'Sorry, Could not create post!'
          })
          this.$q.loading.hide()
        })
    }
  }
}
</script>
<style lang="sass">
  .camera-frame
    border: 2px solid $grey-10
    border-radius: 10px
</style>
