<template>
  <!-- 页面需要监听滚动事件，滚动到某个位置时标题栏要固定 -->
  <!-- 这里需要增加动态的改变样式信息，不是只有显示隐藏！！！！！！！！！ -->
  <div class="wrapper"
       @scroll="scrollList">
    <!-- 由于歌单页和今日推荐页面的顶部展示区域高度不同，所以通过动态的 height 进行设置 -->
    <div class="container-top"
         :style="{height}">
      <div class="blur-bg"
           :style="{backgroundImage:'url('+imgUrl+')'}">
      </div>
      <!-- 通过传值 isAlbum 的布尔值进行判断，因为在今日推荐页面的页面标题是通过滚动显示隐藏的 -->
      <global-nav class="fixed pd23"
                  v-if="!isAlbum"
                  @returnPage="returnPage"
                  :imgUrl="imgUrl"
                  :opacity="opacity">
        <!-- 通过改变 listFixed 来控制 title 的显示与否-->
        <span class="text"
              v-show="listFixed">{{iTitle}}</span>
      </global-nav>
      <!-- 这里是在歌单页面时，页面标题是一直显示的 -->
      <global-nav class="fixed pd23"
                  v-if="isAlbum"
                  @returnPage="returnPage"
                  :imgUrl="imgUrl"
                  :opacity="opacity">
        <span class="text">{{iTitle}}</span>
      </global-nav>
      <!-- 这里包裹的是每日推荐页面额外显示的日期信息 -->
      <div class="pd23"
           v-if="!isAlbum">
        <div class="date">
          <span class="day">{{day}}</span>
          <span class="month">{{month}}</span>
        </div>
        <div class="info">查收属于您的今日推荐</div>
      </div>
      <!-- 这里包裹的是歌单页面的图片，作者，介绍等信息 -->
      <div class="album-info pd23"
           v-if="isAlbum">
        <div class="info-top">
          <div class="img-info">
            <img v-lazy="imgUrl + '?param=200y200'"
                 :key="imgUrl"
                 alt />
            <span class="play-count">
              <i class="date-song bofang"></i>
              {{playCount | setPlay}}
            </span>
          </div>
          <div class="info-con">
            <p class="album-title">{{iAlbumTitle}}</p>
            <div class="creator">
              <div class="img-info">
                <img v-lazy="creatorImgUrl + '?param=200y200'"
                     :key="imgUrl"
                     alt />
              </div>
              <span>
                {{author}}
                <i class="date-song iconfontjiantou5"></i>
              </span>
            </div>
            <div class="desc-wrapper"
                 v-if="description">
              <span class="desc">{{description}}</span>
              <i class="date-song iconfontjiantou5"></i>
            </div>
          </div>
        </div>
        <div class="icons">
          <div class="comments"
               @click="goComments">
            <i class="date-song pinglun"></i>
            <span>{{commentCount | setCom}}</span>
          </div>
          <div class="comments">
            <i class="date-song fenxiang"></i>
            <span>{{shareCount | setShare}}</span>
          </div>
          <div class="comments">
            <i class="date-song xiazai"></i>
            <span>下载</span>
          </div>
          <div class="comments">
            <i class="date-song duoxuankuang"></i>
            <span>多选</span>
          </div>
        </div>
      </div>
    </div>
    <!-- 这里是 播放全部 那一行的信息，因为要操作是否固定，所以需要单独设置 -->
    <div class="title pd23"
         :class="[listFixed?'listFixed':'']">
      <span>
        <span @click="beginAudio">
          <i class="date-song cbofang"></i>
          播放全部
        </span>
        <!-- 当歌单组件时，需要显示当前歌单总共有多少首歌曲的信息 -->
        <span class="count"
              v-if="isAlbum">(共{{trackCount}}首)</span>
      </span>
      <!-- 当在每日推荐界面时，这里显示的是一个多选按钮 -->
      <!-- 这里的多选需要设置一个多选页面的组件！！！！！！！！！ -->
      <span v-if="!isAlbum">
        <i class="date-song caidan"></i>
        多选
      </span>
      <!-- 当时歌单组件时有收藏歌单的按钮选项 -->
      <!-- 这里需要添加判断用户是否已经收藏歌单！！！！来显示不同的样式 -->
      <div class="collection"
           :class="{ 'bg': !isSubInItem }"
           ref="collection"
           v-if="isAlbum">
        <span v-show="!isSubInItem"
              @click="addPlaylist">+ 收藏({{subscribedCountItem | setCol}})</span>
        <span v-show="isSubInItem"
              @click="deletePlaylist">
          <i class="date-song wenjianjia"></i>
          {{subscribedCountItem | setCol}}
        </span>
      </div>
    </div>
    <!-- 这里将列表进行包裹统一的通过 load 属性进行判断是否展示 -->
    <div class="list-info"
         v-bind:style="[paddingHeight,{'marginTop': top}]"
         v-show="!load">
      <slot>
      </slot>
    </div>
    <!-- 页面 loading 组件 -->
    <page-loading v-show="load"></page-loading>
  </div>
</template>

<script>
import globalNav from 'base/generalNav'
import pageLoading from 'base/pageLoading'
import { mapGetters } from 'vuex'
import api from 'api'
const REM = document.body.clientWidth * 0.14
const setNum = function (val) {
  if (!val) {
    return ''
  }
  if (val > 100000000) {
    val = ((val / 100000000).toFixed(1)) + '亿'
  } else if (val > 10000) {
    val = ((val / 10000).toFixed(1)) + '万'
  }
  return val
}

export default {
  name: '',
  components: {
    globalNav,
    pageLoading
  },
  /**
   * 这里使用 data 把 props 接收的值进行存储，因为 vue 不允许子组件直接修改父组件传过来的值
   */
  data () {
    return {
      opacity: '0',
      fixedBg: false,
      iTitle: this.title,
      iAlbumTitle: this.albumTitle,
      listFixed: false,
      top: '0',
      isSubInItem: false,
      subscribedCountItem: 0
    }
  },
  /**
   * 当已经加载了一次页面后，再次进入页面时标题信息不能更新
   *
   * 这里需要监听 props 的改变，如果改变了将新值接受进行改变
   */
  watch: {
    title: function (val) {
      this.iTitle = val
    },
    albumTitle: function (val) {
      this.iAlbumTitle = val
    },
    subscribedCount: function (val) {
      this.subscribedCountItem = val
      if (val === 0 || !val) {
        this.$refs.collection.style.display = 'none'
      } else {
        this.$refs.collection.style.display = ''
      }
    },
    isSubIn: function (val) {
      this.isSubInItem = val
    }
  },
  /**
   * 所有的 props 值信息
   */
  props: {
    albumId: {
      type: Number
    },
    idxId: {
      type: Number
    },
    // 排行榜id，用来正确获取评论信息
    idxComId: {
      type: Number
    },
    dishId: {
      type: Number
    },
    playCount: {
      type: Number
    },
    height: {
      // 顶部展示区域高度
      type: String,
      default: '6rem'
    },
    subscribedCount: {
      type: Number
    },
    subscribed: {
      type: Boolean
    },
    trackCount: {
      type: Number
    },
    shareCount: {
      type: Number
    },
    commentCount: {
      type: Number
    },
    description: {
      type: String
    },
    author: {
      type: String
    },
    creatorImgUrl: {
      type: String
    },
    albumTitle: {
      type: String
    },
    imgUrl: {
      type: String
    },
    title: {
      type: String
    },
    isAlbum: {
      type: Boolean,
      default: true
    },
    load: {
      type: Boolean,
      default: true
    },
    isSubIn: {
      type: Boolean,
      default: false
    }
  },
  filters: {
    setCom: function (val) {
      if (!val) {
        return '评论'
      }
      return setNum(val)
    },
    setShare: function (val) {
      if (!val) {
        return '分享'
      }
      return setNum(val)
    },
    setCol: function (val) {
      if (!val) {
        return ''
      }
      return setNum(val)
    },
    setPlay: function (val) {
      if (!val) {
        return ''
      }
      return setNum(val)
    }
  },
  // 对日期信息提取展示
  computed: {
    ...mapGetters(['AUDIO_LIST']),
    /**
     * 返回日
     */
    day: function () {
      return new Date().getDate() < 10
        ? '0' + new Date().getDate()
        : new Date().getDate()
    },
    /**
     * 返回月份
     */
    month: function () {
      return new Date().getMonth() + 1 < 10
        ? '0' + (new Date().getMonth() + 1)
        : new Date().getMonth() + 1
    },
    paddingHeight () {
      return this.AUDIO_LIST.length ? { 'paddingBottom': '1rem' } : {}
    }
  },
  methods: {
    /**
     * 收藏歌单
     */
    addPlaylist () {
      const id = this.albumId || this.dishId || this.idxId
      api.addOrDeletePlaylistFn(1, id)
        .then(res => {
          const data = res.data
          if (data.code === 200) {
            this.isSubInItem = true
            ++this.subscribedCountItem
          }
        })
    },
    /**
     * 取消收藏歌单
     */
    deletePlaylist () {
      const id = this.albumId || this.dishId || this.idxId
      api.addOrDeletePlaylistFn(2, id)
        .then(res => {
          const data = res.data
          if (data.code === 200) {
            this.isSubInItem = false
            --this.subscribedCountItem
          }
        })
    },
    /**
     * 播放全部，向父组件传递事件
     */
    beginAudio () {
      this.$emit('startPlayAll')
    },
    /**
     * 去评论页面
     * 通过传不同的params的属性来判断资源是歌单还是专辑
     */
    goComments () {
      const playlistId = this.albumId ? this.albumId : this.idxComId
      const albumId = this.dishId
      const imgUrl = this.imgUrl
      const title = this.albumTitle
      const author = this.author
      this.$router.push({ name: 'comments', params: { playlistId, albumId, imgUrl, title, author } })
    },
    /**
     * 返回上一页
     */
    returnPage () {
      this.listFixed = false
      this.top = '0'
      this.opacity = '0'
      this.$router.go(-1)
    },
    /**
     * 定义页面滚动事件，
     * 这里需要添加在滚动过程中样式的变化
     */
    scrollList (e) {
      // 获取到 top 值
      let top = e.target.scrollTop
      // 当当前组件不是歌单组件时，就是每日推荐页面
      if (!this.isAlbum) {
        this.opacity = this.opacity <= 1 ? top / (2.6 * REM) + '' : '1'
        if (top >= 136) {
          this.listFixed = true
          this.top = '1rem'
        } else {
          this.listFixed = false
          this.top = '0'
        }
      } else {
        this.opacity = this.opacity <= 1 ? top / (5 * REM) + '' : '1'
        // 当是歌单组件时，当页面滚动到一定位置的时候顶部的标题会变
        if (top >= 2.6 * REM) {
          // 这里使用data存下了props的值进行修改，子组件不能直接修改props传过来的值
          this.iTitle = this.albumTitle
        } else {
          this.iTitle = this.title
        }
        // 当 top 到了 250 的时候会改变标题行的是否固定样式
        if (top >= 5 * REM) {
          this.listFixed = true
          this.top = '1rem'
        } else {
          this.listFixed = false
          this.top = '0rem'
        }
      }
    }
  }
}
</script>

<style lang='less' scoped>
@import url("//at.alicdn.com/t/font_1394963_t6jt71rtm9.css");
@import url("~styles/global.less");
.topFixed {
  position: fixed;
  width: 100%;
  height: 1rem;
  z-index: 9;
}
@textColor: #ccc;
.fixed {
  .topFixed();
  top: 0;
}
.listFixed {
  .topFixed();
  top: 1rem;
}

.wrapper {
  height: 100vh;
  position: relative;
  overflow: scroll;
  // 组件的title
  .title {
    font-size: 0.3rem;
    height: 1rem;
    line-height: 1rem;
    .flex-between();
    background-color: #fff;
    .count {
      color: #999;
      font-size: small;
    }
    .collection {
      .pd23();
      font-size: smaller;
      margin-top: 0.125rem;
      height: 0.75rem;
      line-height: 0.75rem;
      border-radius: 0.4rem;
      color: #999;
      &.bg {
        background: linear-gradient(to bottom, #ff1a00 19%, #ff1a00 50%);
        color: #fff;
      }
    }
  }
  .container-top {
    width: 100%;
    color: #fff;
    position: relative;
    background-color: rgba(0, 0, 0, 0.5);
    .faker-bg {
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      height: 1rem;
      background: #fff;
    }
    .blur-bg {
      position: absolute;
      top: 0;
      left: 0;
      bottom: 0;
      right: 0;
      z-index: -1;
      background-size: cover;
      filter: blur(50px);
      background-position: center;
    }
    .date {
      padding-top: 1.5rem;
      .day {
        font-size: 0.7rem;
      }
      .month {
        font-size: 0.4rem;
        color: #c8d6e5;
        &::before {
          content: "/";
        }
      }
    }
    .info {
      margin-top: 0.7rem;
    }
    .text {
      padding-left: 0.1rem;
      font-size: 0.3rem;
      font-weight: 600;
      vertical-align: 5px;
      width: 8rem;
      .ellipsis();
    }
    .album-info {
      padding-top: 1.5rem;
      .info-top {
        height: 3rem;
        .flex-between();
        overflow: hidden;
        .img-info {
          @size: 2.6rem;
          width: @size;
          height: @size;
          position: relative;
          border-radius: @imgBorderRadius;
          overflow: hidden;
          img {
            width: @size;
            height: @size;
          }
          .play-count {
            position: absolute;
            top: 0.1rem;
            right: 0.1rem;
            .bofang {
              font-size: 0.24rem;
            }
          }
        }
        .info-con {
          width: 3.6rem;
          height: 2.6rem;
          display: flex;
          flex-direction: column;
          overflow: hidden;
          .album-title {
            font-size: 0.36rem;
            line-height: 1.5;
            .twoLinesEllipsis();
          }
          .creator {
            height: 1rem;
            color: @textColor;
            display: flex;
            align-items: center;
            .img-info {
              @size: 0.6rem;
              width: @size;
              height: @size;
              margin-right: 8px;
              border-radius: 50%;
              overflow: hidden;
              img {
                width: @size;
                height: @size;
              }
            }
          }
          .desc-wrapper {
            display: flex;
            align-items: center;
            color: @textColor;
            .desc {
              width: 3rem;
              .twoLinesEllipsis();
            }
          }
        }
      }
      .icons {
        margin-top: 8px;
        .flex-around();
        width: 100%;
        .comments {
          display: flex;
          justify-content: center;
          align-items: center;
          flex-direction: column;
          .date-song {
            font-size: 0.4rem;
            margin-bottom: 5px;
          }
        }
      }
    }
  }
  .list-info {
    position: relative;
    z-index: 4;
    width: 100%;
    box-sizing: border-box;
    padding: 0.1rem 0.23rem;
    background-color: #fff;
  }
}
</style>
