<template>
  <q-page class="constrain q-pa-md">
    <div class="row q-col-gutter-lg">
      <div class="col-12 col-sm-8">
        <q-card flat bordered v-for="post in posts" :key="post.id" class="card-post q-mb-md">
          <q-item>
            <q-item-section avatar>
              <q-avatar>
                <img src="https://cdn.quasar.dev/img/boy-avatar.png">
              </q-avatar>
            </q-item-section>

            <q-item-section>
              <q-item-label>Auto</q-item-label>
              <q-item-label caption>
                {{post.location}}
              </q-item-label>
            </q-item-section>
          </q-item>

          <q-separator />
          <q-img :src="post.imageUrl" />
          <q-card-section>
            <div>{{post.Caption}}</div>
            <div class="text-caption text-grey">{{post.date | niceDate}}</div>
          </q-card-section>
        </q-card>
      </div>
      <div class="col-4 large-screen-only">
        <q-item class="fixed">
            <q-item-section avatar>
              <q-avatar size="48px">
                <img src="https://cdn.quasar.dev/img/boy-avatar.png">
              </q-avatar>
            </q-item-section>

            <q-item-section>
              <q-item-label class="text-bold">Auto Prananda</q-item-label>
              <q-item-label caption>
                auto_prananda
              </q-item-label>
            </q-item-section>
          </q-item>
      </div>
    </div>
    
  </q-page>
</template>

<script>
import { date } from 'quasar'
export default {
  name: 'Home',
  data () {
    return {
      posts: [
        {
        id: 1,
        Caption: 'Golden Gate',
        date: 1616571845759,
        location: 'Jakarta',
        imageUrl: 'https://cdn.quasar.dev/img/parallax2.jpg'
        },
        {
        id: 2,
        Caption: 'Golden Gate',
        date: 1616571845759,
        location: 'Jakarta',
        imageUrl: 'https://cdn.quasar.dev/img/parallax2.jpg'
        },
        {
        id: 3,
        Caption: 'Golden Gate',
        date: 1616571845759,
        location: 'Jakarta',
        imageUrl: 'https://cdn.quasar.dev/img/parallax2.jpg'
        },
        {
        id: 4,
        Caption: 'Golden Gate',
        date: 1616571845759,
        location: 'Jakarta',
        imageUrl: 'https://cdn.quasar.dev/img/parallax2.jpg'
        }
      ]
    }
  },
  filters: {
    niceDate(value) {
      return date.formatDate(value, 'D MMMM h:mmA')
    }
  }
}
</script>
<style lang="sass">
  .card-post
    .q-img
      min-height: 200px
</style>
