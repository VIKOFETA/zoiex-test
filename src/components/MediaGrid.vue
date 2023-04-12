<template>
  <v-alert class="media-grid__alert" closable v-if="showAlert" @click:close="showAlert = false" text="Link copied to clipboard!" type="success"></v-alert>
 <v-container fluid>
  <div class="media-grid__container">
    <transition-group name="media-grid">
      <template 
          v-for="item in medias"  
          :key="item.id">
        <MediaComp
          v-if="item !== undefined"
          :media="item"
          class="media-grid__item"
          @toClipboard="copyToClipboard"
        /> 
      </template>   
    </transition-group>
  </div>
  <v-progress-circular v-if="loading" :size="70" :width="7" indeterminate color="primary"></v-progress-circular>
  <div ref="observerTarget"></div>
</v-container>
  
</template>

<script lang="ts">
import useClipboard from 'vue-clipboard3'
import { defineComponent, ref, onMounted, watch, onUnmounted} from 'vue';
import axios from 'axios';

interface TMedia {
  type: string;
  id: string;
  url: string,
  title: string,
  
  bitly_gif_url: string;
  bitly_url: string;
  content_url: string;
  embed_url: string;
  images: {
    original: any;
    downsized: any;
    downsized_large: any;
    downsized_medium: any;
    downsized_small: any;
  };
  rating: string;
  slug: string;
  username: string;
}

import MediaComp from './Media.vue';
import { helperNameMap } from '@vue/compiler-core';

export default defineComponent({
  name: 'MediaGrid',
  components: {
    MediaComp,
  },

  props: {
    searchString: {
      type: String,
      default: ''
    },
  },

  setup(props) {
    const apiKey = '8THsQtNfqLgBTaJNpzM73sBh3SM31IMx';
    let slimit = 24;
    let soffset = 0;

    let limit = 24;
    let offset = 0;
  
    let medias = ref<TMedia[] | null>([]);
  
    let searching =  ref(false);
    let loading =  ref(false);
    let loadingMore =  ref(false);

    let showAlert = ref(false);

    let observer = ref<IntersectionObserver | null>(null);
    const observerTarget = ref<Element | null>(null);

    const { toClipboard } = useClipboard()
 
    const search = async (searchStr: string ) => {
      loading.value = true;
      let data: TMedia[] | null = null;

      if (loadingMore.value == true) {
        offset = limit;
        limit += slimit;
      }

      if(searchStr !== '') {
        try {
          const resp = await axios.get(`https://api.giphy.com/v1/gifs/search?api_key=${apiKey}&q=${searchStr}&limit=${limit}&offset=${offset}&rating=g&lang=en`)
          data = resp.data.data as TMedia[];          
        } catch (error) {
          console.error('Failed to fetch data:', error);
        }
      } else {
        try {
          const resp = await axios.get(`https://api.giphy.com/v1/gifs/trending?api_key=${apiKey}&limit=${limit}&rating=r&offset=${offset}`)
          data = resp.data.data as TMedia[];
        } catch (error) {
          console.error('Failed to fetch data:', error);
        }
      }

      loading.value = false;
      loadingMore.value = false;
      return data;
    }

    const onIntersect = (entries: any, observer: any) => {

      entries.forEach(async (entry: { isIntersecting: any; target: any; }) => {
        if (entry.isIntersecting && loading.value == false && searching.value == false) {

          loadingMore.value = true;
          let more = await search(props.searchString);
          medias.value = (medias.value ?? []).concat(more ?? []);

        }
      });

    };

    onMounted(() => {
      observer.value = new IntersectionObserver(onIntersect, {
        root: null,
        rootMargin: '0px',
        threshold: 0.1,
      });
      if (observerTarget.value) {
        observer.value.observe(observerTarget.value);
      }
    });

    onUnmounted(() => {
      if (observer.value) {
        observer.value.disconnect();
      }
    });

    onMounted(async () => {
      medias.value = await search('');
    });

    watch(
      () => props.searchString,
      async (newValue) => {
        limit = slimit;
        offset = soffset;

        searching.value = true;
        medias.value = await search(newValue);

        if((medias.value)?.length == 0) {
          medias.value = await search('Not Found');
        }
        searching.value = false;
      }
    );

    const copyToClipboard = async (e: {title: string, link: string}) => {
      try {
        await toClipboard(e.link)
        console.log('Copied to clipboard')
        showAlert.value = true;
      } catch (e) {
        console.error(e)
      }
    }

    return {
      medias,
      copyToClipboard,
      observer,
      observerTarget,
      loading,
      showAlert
    };
  },

});
</script>

<style scoped>
  .media-grid{
    
  }

  .media-grid__alert{
    position: fixed;
    z-index: 2;
  }
  .media-grid__container{
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
  }

  .media-grid__item{
    margin: 10px;
    max-width: 33.33%;
    border-radius: 3px;
  }

  .media-grid-enter-active,
  .media-grid-leave-active {
    transition: all 0.5s ease;
  }
  .media-grid-enter-from,
  .media-grid-leave-to {
    opacity: 0;
    /* transform: translateX(30px); */
  }

  @media (max-width: 768px) {
    .media-grid__item{
      max-width: 40%;
    }
  }

</style>