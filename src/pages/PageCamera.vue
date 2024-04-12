<template>
  <q-page class="constrain-more q-pa-md">
    <div class="camera-frame q-pa-md">
      <img
        class="full-width"
        src="https://cdn.quasar.dev/img/parallax2.jpg" >
    </div>
    <div class="text-center q-pa-md">
      <q-btn
        round 
        color="black"
        size="lg"
        icon="eva-camera" />
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
        class="col col-sm-8"
        label="Location"
      >
        <template v-slot:append>
          <q-btn
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
      }
    }
  }
}
</script>

<style lang="sass" scoped>

</script>