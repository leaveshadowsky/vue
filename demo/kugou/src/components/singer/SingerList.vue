<template>
  <section class="singer_list">
    <PubModuleTitle :title="singerList.info.name"/>
    <ul class="singer_list__list" @scroll="$_xsl__loadImgLazy($refs.lazyImages)">
      <li
        class="singer_list__item main_border_bottom"
        v-for="(item,index) in singerList.data"
        :key="index"
      >
        <router-link :to="item.path" class="singer_list__link">
          <img
            class="singer_list__img lazy_image"
            ref="lazyImages"
            :src="logo__grey"
            :data-src="item.imgUrl"
            :data-is-loaded="isLoaded"
          >
          <div class="singer_list__name">{{item.name}}</div>
        </router-link>
      </li>
    </ul>
  </section>
</template>

<script>
import PubModuleTitle from '../public/PubModuleTitle'
import axios from 'axios'
import api from '../../assets/js/api'
import mixin from '../../mixins/index'
import loading from '../../mixins/loading'
import { mapState, mapMutations } from 'vuex'

export default {
  name: 'SingerList',
  mixins: [mixin, loading],
  components: {
    PubModuleTitle
  },
  computed: {
    ...mapState('images', ['logo__grey']),
    ...mapState('singer', ['singerList']),
    ...mapState('loading', {
      isLoadingShow: 'isShow'
    })
  },
  created() {
    // todo add scrollRemember
    let singerListId = this.$route.path.split('/').pop()
    if (Number(singerListId) !== this.singerList.info.id) {
      this.setLoadingExcludeHeader()
      this.startLoading()
      this.getSingerList(singerListId)
    }
  },
  mounted() {
    let lazyImages = this.$refs.lazyImages
    this.$_xsl__detectToLoadImgLazy(lazyImages, this.$el, '.lazy_image')
  },
  methods: {
    ...mapMutations(['replaceProperty']),
    getSingerList(singerListId) {
      axios
        .get(api.singerList.replace(/singerListId?/i, singerListId))
        .then(({ data }) => {
          let singerList = {
            info: {
              id: data.classid,
              name: data.classname,
              count: data.singers.total
            },
            data: data.singers.list.info
          }
          data.singers.list.info.forEach(obj => {
            obj.id = obj.singerid
            obj.name = obj.singername
            obj.imgUrl = this.$_xsl__replaceImgUrlSize(obj.imgurl)
            obj.path = '/singer/info/' + obj.id
          })
          this.replaceProperty({
            paths: 'singer.singerList',
            data: singerList
          })
          this.stopLoading()
        })
        .catch(er => {
          alert(er)
        })
    }
  }
}
</script>

<style scoped lang="less">
@import (reference) '../../assets/css/constant.less';
.singer_list {
  font-size: 19px;
}
.singer_list__list {
  overflow: scroll;

  box-sizing: border-box;
  height: calc(100% - @module_title_height);
}
.singer_list__item {
  height: 88px;
}
.singer_list__link {
  display: flex;
  align-items: center;

  height: 100%;
}
.singer_list__img {
  display: block;

  width: 61px;
  height: 61px;
  margin: 0 18px 0 @padding_width;
}
</style>
