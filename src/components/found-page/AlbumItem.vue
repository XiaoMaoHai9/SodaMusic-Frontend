<template>
  <div class="album-iten bs">
    <div class="album-cover-container br" @click="playAlbum(data.sid || data.id)">
      <div class="cover-mask mask-style-b">
        <a-icon type="play-circle" class="play-icon" title="播放" theme="filled"/>
      </div>
      <img class="album-cover lazy-img" v-lazy="data.imgUrl || data.cover_url + '?param=90y90'">
    </div>
    <div class="album-info">
      <span class="album-name">{{data.name || data.song_name}}</span>
      <span class="singer-name">{{data.singer.join(' / ')}}</span>
    </div>
    <div class="album-tools">
      <slot></slot>
    </div>
  </div>
</template>

<script>
import { mapActions } from 'vuex'

export default {
  name: 'NewAlbum',
  data () {
    return {
    }
  },
  props: [
    'data'
  ],
  methods: {
    ...mapActions(['checkMusic']),

    // 播放专辑
    async playAlbum (info) {
      if (this.$store.state.sodaAccount.isLogin && this.$route.fullPath === '/myMicLib/micMangage') {
        // 检查播放列表是否已存在该歌曲
        const { isHas, arrIndex } = this.$store.getters.songIsExist(info)
        // 如果该歌曲已在播放列表
        if (isHas) {
          // 如果歌曲已在末尾
          if (arrIndex === this.$store.state.audioPlayer.songsList.length - 1) {
          // 重新播放
            this.$store.commit('changePlayNow', 'replay')
            return
          }
          // 歌曲置尾
          this.$store.commit('endSong', arrIndex)
          // 切歌
          this.$store.commit('changePlayNow', 'end')
          return
        }

        this.$store.dispatch('playSong', this.data)
        return
      }
      // 获取歌曲详情
      this.$store.dispatch('getAlbumAllSongs', info)
    }
  }
}
</script>

<style lang="less" scoped>

.album-iten{
  position: relative;
  display: flex;
  width: 310px;
  height: 110px;
  margin-right: 15px;
  padding: 10px;
  border-radius: 14px;
  transition: .4s;
  background-color: #fff;

  .album-cover-container{
    position: relative;
    width: 90px;
    height: 90px;
    margin-right: 15px;
    border: 1px solid @color-grey;
    overflow: hidden;
    cursor: pointer;
    .lazy-img-container-mixins(90px, 90px);

    .album-cover{
      width: 100%;
      height: 100%;
    }

    .cover-mask{
      position: absolute;
      width: 100%;
      height: 100%;
      line-height: 90px;
      text-align: center;
      z-index: 3;
      opacity: 0;
      transition: .2s;

      .play-icon{
        font-size: 35px;
        color: #fff;
        vertical-align: middle;
        transition: .2s;

        &:hover{
          font-size: 40px;
        }
      }
    }

    .album-cover{
      transition: .3s transform;
    }

    &:hover{
      .cover-mask{
        opacity: 1;
      }

      .album-cover{
        transform: scale(1.1);
      }
    }
  }

  .album-info{
    display: flex;
    flex-direction: column;
    width: 33%;
    height: 90px;
    font-size: 0;
    padding: 15px 0;

    .album-name, .singer-name{
      display: inline-block;
      height: 30px;
      line-height: 30px;
      font-size: 20px;
      text-overflow: ellipsis;
      overflow: hidden;
      white-space: nowrap;
    }
    .singer-name{
      color: #9a9fa9;
    }
  }

  .album-tools{
    position: absolute;
    right: 0;
    // width: 105px;
    height: 90px;
    line-height: 90px;
    text-align: center;

    .tools-icon{
      vertical-align: middle;
      color: #999;
      transition: all 0.2s;
    }

    .tools-icon:hover{
      color: rgba(0, 0, 0, 0.65);
      cursor: pointer;
    }
  }
}
</style>
