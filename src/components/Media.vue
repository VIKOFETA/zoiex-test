<template>
  <div class="media" v-if="media">
    <div class="media__info">
      <v-icon class="media__share" @click="share" >mdi-share-variant</v-icon>
    </div>
    <div class="media__image-wrapper">
      <v-img
        :width="300"
        :aspect-ratio="1"
        :lazy-src="cover"
        :src="url"
        :alt="title"
        cover
        class="media__image"
      ></v-img>
      <div class="media__image-cascades">        
        <div class="media__image-cascade"></div>
        <div class="media__image-cascade"></div>
        <div class="media__image-cascade"></div>
      </div>
    </div>
  </div>
  
</template>

<script lang="ts">
import { defineComponent, ref, provide} from 'vue';

export default defineComponent({
  name: 'Media',

  props: {
    media: {
      type: Object,
      default: () => { return {}; }
    },
  },

  setup(props) {
    const type = props.media.type as string;
    const id = props.media.id as string;
    const title = props.media.title as string;
    const url = props.media.images.fixed_height.url as string;
    const cover = props.media.images.preview_webp.url as string;
    const link = props.media.url as string;
    // const urlWebp = props.media.images.original.webp as String;

    // console.log(props.media);


    return {
      type,
      id,
      url,
      title,
      cover,
      link,
    };
  },

  methods: {
    share() {
      this.$emit('toClipboard', { title: this.title,  link: this.link})
    }
  }

});
</script>

<style scoped>
  .media{
    position: relative;
  }

  .media:hover .media__info{
    opacity: 1;
  }

  .media__info{
    display: flex;
    justify-content: flex-end;
    position: absolute;
    top: 5px;
    right: 5px;
    width: 100%;
    z-index: 2;
    opacity: 0;
    transition: opacity .3s;
  }

  .media__share{
    color: white;
  }

  .media__image-wrapper{
    display: flex;
    flex-direction: column;
    position: relative;
    opacity: 1;
    text-shadow: rgba(0, 0, 0, 0.25) 0px 2px 10px;
  }
  .media__image-cascades{
    display: flex;
    flex-direction: column;
  }
  .media__image-cascade{
    background: rgb(var(--v-theme-primary));
  }

  .media__image-cascade:first-child{    
    box-shadow: rgba(0, 0, 0, 0.35) 0px 1px 1px 0px;
    height: 3px;
    opacity: 0.8;
  }
  .media__image-cascade:nth-child(2){    
    box-shadow: rgba(0, 0, 0, 0.35) 0px 1px 1px 0px;
    height: 6px;
    margin: 0px 6px;
    opacity: 0.6;
  }
  .media__image-cascade:last-child{    
    height: 6px;
    margin: 0px 12px;
    opacity: 0.4;
  }


</style>