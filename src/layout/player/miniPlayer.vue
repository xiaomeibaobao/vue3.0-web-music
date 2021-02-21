<template>
  <div class="mini-player">
    <div class="left">
      <div class="song-img">
        <i class="iconfont iconiconset0441"></i>
        <img :src="picUrl" alt="">
      </div>
      <div class="name">
        <div class="song-name" :title="songName + aliasStr">
          {{songName}}
          <span class="song-name-alias">{{aliasStr}}</span>
        </div>
        <div class="artist-name" >
          <template v-for="item of artistList" :key="item.id">
            <span class="artist-item" :title="item.name">{{item.name}}</span>
          </template>
        </div>
      </div>
    </div>
    <div class="middle">
      <ul class="btn-select">
        <li class="btn-item">
          <div class="bofang-type" :style="{display: showBofangType ? 'block' : 'none'}"><span>{{listenTypeTitle}}</span></div>
          <i
            class="iconfont turn-red"
            :class="[listenTypeClass]"
            :title="listenTypeTitle"
            @click="changeBofangType"
          ></i>
        </li>
        <li class="btn-item">
          <i class="iconfont iconshangyishoushangyige turn-red" @click="preSong"></i>
        </li>
        <li class="btn-item">
          <div class="bofang" @click="clickBofang">
            <i class="iconfont icon-item" :class="{iconicon_bofang: playing, icon65zanting: !playing}"></i>
          </div>
        </li>
        <li class="btn-item">
          <i class="iconfont iconxiayigexiayishou turn-red" @click="nextSong"></i>
        </li>
        <li class="btn-item">
          <span class="song-geci turn-red">词</span>
        </li>
      </ul>
      <div class="progress-bar">
        <div class="progress-bar-time">{{startTimeStr}}</div>
        <div class="progress-bar-middle">
          <ProgressBar
            :percentage="percentage"
            @change="changeProgress"
          />
        </div>
        <div class="progress-bar-time">{{endTimeStr}}</div>
      </div>
    </div>
    <div class="right">
      <i class="iconfont iconvolume"></i>
      <div class="progress-bar-right">
        <ProgressBar
          :percentage="volume"
          @change="changeVolume"
        />
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { player } from '../../store/modules/palyer'
import ProgressBar from '@/components/progressBar/index.vue'
import { transformTime } from '@/utils/util'
import { ref, computed, nextTick } from 'vue'

export default {
  name: 'miniPlayer',
  components: {
    ProgressBar
  },
  props: {
    percentage: {
      type: Number
    },
    startTime: {
      type: Number
    },
    endTime: {
      type: Number
    },
    volume: {
      type: Number
    }
  },
  emits: ['changeProgress', 'switchMusic', 'changeVolume'],
  setup (props, context) {
    const startTimeStr = computed(() => transformTime(props.startTime))
    const endTimeStr = computed(() => transformTime(props.endTime))
    const songName = computed(() => player.songDetails.name)
    const artistList = computed(() => player.songDetails.artistsList)
    const aliasStr = computed(() => player.songDetails.aliasStr)
    const picUrl = computed(() => player.songDetails.albumPicUrl)
    const playing = computed(() => player.playing)
    const selectListenType = computed(() => player.selectListenType)
    const listenTypeTitle = computed(() => {
      switch (player.selectListenType) {
        case 'order': return '顺序播放'
        case 'random': return '随机播放'
        case 'single': return '单曲循环'
        default: return '顺序播放'
      }
    })
    const listenTypeClass = computed(() => {
      switch (player.selectListenType) {
        case 'order': return 'iconbaocunshunxu'
        case 'random': return 'iconsuijibofang01'
        case 'single': return 'iconhanhan-01-01'
        default: return 'iconbaocunshunxu'
      }
    })
    const listenTypeList = computed(() => player.listenTypeList)
    const showBofangType = ref<boolean>(false)
    const changeVolume = (percentage) => {
      context.emit('changeVolume', percentage)
    }
    const changeProgress = (percentage) => {
      context.emit('changeProgress', percentage)
    }
    const clickBofang = () => {
      context.emit('switchMusic')
    }

    const nextSong = () => {
      player.nextSong()
    }

    const preSong = () => {
      player.preSong()
    }

    const changeBofangType = () => {
      const index = listenTypeList.value.indexOf(selectListenType.value)
      if (index >= listenTypeList.value.length - 1) {
        player.changeListenType(0)
      } else {
        player.changeListenType(index + 1)
      }
      showBofangType.value = true
      setTimeout(() => {
        showBofangType.value = false
      }, 1000)
    }
    return {
      startTimeStr,
      endTimeStr,
      showBofangType,
      songName,
      artistList,
      aliasStr,
      picUrl,
      playing,
      listenTypeList,
      selectListenType,
      listenTypeTitle,
      listenTypeClass,
      changeVolume,
      changeProgress,
      clickBofang,
      nextSong,
      preSong,
      changeBofangType
    }
  }
}
</script>

<style lang="scss" scoped>
.mini-player {
  position: fixed;
  bottom: 0;
  left: 0;
  width: 100%;
  height: $player-height;
  box-sizing: border-box;
  padding: 0 10px;
  background-color: #fff;
  display: flex;
  align-items: center;
  flex-direction: row;
  flex-wrap: nowrap;

  .left {
    flex: 0 0 25%;
    display: flex;
    flex-direction: row;
    align-items: center;

    .song-img {
      position: relative;

      >img {
        width: 44px;
        height: 44px;
        border-radius: 5px;
        cursor: pointer;
      }

      >i {
        position: absolute;
        width: 36px;
        height: 36px;
        top: 4px;
        left: 4px;
        z-index: 5;
        font-size: 36px;
        color: #676767;
        transform: scale(0);
        animation: all .2s ease-in-out;
      }

      &:hover {
        >i {
          transform: scale(1);
        }
        >img {
          filter: blur(3px);
        }
      }
    }

    .name {
      margin-left: 10px;
      display: flex;
      flex-direction: column;

      .song-name {
        font-size: 16px;
        display: inline-block;
        max-width: 150px;
        text-overflow: ellipsis;
        overflow: hidden;
        white-space: nowrap;
        margin-bottom: 10px;
        cursor: pointer;

        .song-name-alias {
          color: #989898;
        }
      }
      .artist-name {
        font-size: 12px;

        .artist-item {
          padding-right: 10px;
          cursor: pointer;
        }
      }
    }
  }

  .middle {
    flex: 0 0 50%;
    display: flex;
    flex-direction: column;
    align-items: center;

    .btn-select {
      display: flex;
      height: 36px;
      flex-direction: row;
      justify-content: center;
      margin-bottom: 2px;

      .btn-item {
        margin: 0 12px;
        position: relative;
        font-size: 18px;
        display: flex;
        align-items: center;

        .bofang-type {
          position: absolute;
          top: -20px;
          display: none;
          left: 50%;
          padding: 6px 12px;
          font-size: 12px;
          border-radius: 5px;
          color: #f2f3f4;
          background-color: rgba(77, 77, 79, 0.9);
          transform: translateX(-50%);
          animation: all 0.6s ease-in-out;

          >span {
            white-space: nowrap;
          }
        }

        .bofang {
          background-color: #f5f5f6;
          height: 36px;
          width: 36px;
          border-radius: 50%;
          color: #000000;
          cursor: pointer;
          position: relative;

          .icon-item {
            position: absolute;
            font-size: 20px;
            width: 20px;
            height: 20px;
            line-height: 1;
            top: 8px;
            left: 8px;
          }

          &:hover {
            background-color: #e9e9e9;
          }
        }

        .song-geci {
          font-size: 14px;
        }

        .turn-red {

          &:hover {
            color: #ec4141;
          }
        }
      }
    }

    .progress-bar {
      display: flex;
      flex-direction: row;
      justify-content: center;
      flex-wrap: nowrap;
      align-items: center;

      .progress-bar-time {
        font-size: 12px;
        color: #989898;
      }

      .progress-bar-middle {
        margin: 0 10px;
        width: 500px;
      }
    }
  }

  .right {
    flex: 0 0 25%;
    display: flex;
    flex-direction: row;
    align-items: center;

    > i {
      font-size: 18px;
    }

    .progress-bar-right {
      width: 100px;
      padding: 0 10px;
    }
  }
}
</style>