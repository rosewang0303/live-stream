<template>
  <div class="home">
    <!-- pc -->
    <div class="pc">
      <div class="pc__wrap">
        <div class="pc__video-wrap">
          <video ref="myVideo" />
          <!-- 視訊鏡頭關閉頭像 -->
          <div v-if="!stream.video && !stream.close" class="pc__video-empty-wrap">
            <img src="@/assets/icon/user.svg"/>
          </div>
          <!-- 視訊已結束 -->
          <div v-if="stream.close" class="pc__video-close">
            <div>視訊已結束</div>
          </div>
          <!-- live/時間 -->
          <div class="pc__video-label">
            <div v-if="!stream.close" ref="live" class="pc__video-label--live">LIVE</div>
            <div v-else class="pc__video-label--live pc__video-label--close">LIVE</div>
            <div class="pc__video-label--time" :class="{'pc__video-label--time-end': stream.close}">{{timeLabel}}</div>
            <div v-if="!stream.close" class="pc__video-label--watch">
              <img src="@/assets/icon/eye.svg"/>
              <div>{{watch}}</div>
            </div>
          </div>
          <!-- 控制麥克風/關閉視訊/鏡頭 -->
          <div class="pc__video-tool-wrap">
            <button class="pc__video-tool pointer" ref="audioIcon" :disabled="stream.close" @click="audioHandler()">
              <img v-if="stream.audio" class="pc__video-tool--audio" src="@/assets/icon/microphone-2.svg"/>
              <img v-else class="pc__video-tool--audio" src="@/assets/icon/microphone-slash.svg"/>
            </button>
            <button class="pc__video-tool pointer" ref="closeIcon" :disabled="stream.close" @click="closeStreemHandler()">
              <img class="pc__video-tool--close" src="@/assets/icon/add.svg"/>
            </button>
            <button class="pc__video-tool pointer" ref="videoIcon" :disabled="stream.close" @click="videoHandler()">
              <img v-if="stream.video" class="pc__video-tool--video" src="@/assets/icon/video.svg"/>
              <img v-else class="pc__video-tool--video" src="@/assets/icon/video-slash.svg"/>
            </button>
          </div>
          <!-- 視訊emoji -->
          <div class="pc__video-emoji-wrap">
            <transition-group @enter="emojiEnter()">
              <div v-for="(emoji, index) in emojiArr" :key="index" class="pc__video-emoji" :ref="`streamEmoji_${index}`">{{emoji}}</div>
            </transition-group>
          </div>
        </div>
        <div class="pc__comment-wrap">
          <div class="pc__comment-item-wrap" ref="comments">
            <!-- 尚未有任何留言 -->
            <div v-if="commentList.length == 0" class="pc__comment-item--empty">尚未有任何留言</div>
            <!-- 留言串 -->
            <div v-else>
              <div class="pc__comment-item" v-for="(row, index) in commentList" :key="index">
                <div class="pc__comment-item--img-wrap">
                  <img src="@/assets/icon/user.svg"/>
                </div>
                <div class="pc__comment-item--text-wrap">
                  <span class="pc__comment-item--user-name">{{row.account}}</span>
                  <span class="pc__comment-item--comment">{{row.comment}}</span>
                </div>
              </div>
              <div v-if="stream.close" class="pc__comment-item--end">已結束視訊</div>
            </div>
          </div>
          <!-- 控制留言 -->
          <div class="pc__comment-bar-wrap">
            <div class="pc__comment-bar">
              <!-- 留言 -->
              <div class="pc__comment-bar--input-wrap">
                <input v-model="user.comment" placeholder="留言..." @keyup.enter="submitComment()"/>
                <button class="pc__comment-bar--input-icon" ref="commentIcon" :disabled="stream.close" @click="submitComment()">
                  <img src="@/assets/icon/direct-right.svg"/>
                </button>
              </div>
              <!-- 送出emoji -->
              <div class="pc__comment-bar--tool-wrap">
                <!-- emoji -->
                <button ref="emojiIcon" :disabled="stream.close" @click="emojiMenuHandler()">
                  <img src="@/assets/icon/smileys.svg"/>
                </button>
                <!-- 愛心 -->
                <button ref="heartIcon" :disabled="stream.close" @click="submitHeart()">
                  <img src="@/assets/icon/heart.svg"/>
                </button>
                <!-- emoji選單 -->
                <transition name="fade">
                  <div v-if="menu.emoji" class="pc__comment-bar--emoji-wrap">
                    <span v-for="(emoji, index) in emojis" :key="index" class="pointer" :ref="`emoji_${index}`" @click="submitEmoji(index)">{{emoji}}</span>
                    <div class="pc__comment-bar--triangle"></div>
                  </div>
                </transition>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <!-- mb -->
    <div class="mb">
      <div class="mb__wrap">
        <div class="mb__video-wrap">
          <video ref="myVideo" />
          <!-- live/時間 -->
          <div class="pc__video-label mb__video-label">
            <div class="h-align mb__video-label-wrap">
              <div v-if="!stream.close" ref="live" class="pc__video-label--live">LIVE</div>
              <div v-else class="pc__video-label--live pc__video-label--close">LIVE</div>
              <div class="pc__video-label--time" :class="{'pc__video-label--time-end': stream.close}">{{timeLabel}}</div>
              <div v-if="!stream.close" class="pc__video-label--watch">
                <img src="@/assets/icon/eye.svg"/>
                <div>{{watch}}</div>
              </div>
            </div>
            <div class="mb__video-label--setting">
              <button class="mb__btn-wrap" ref="settingIcon" @click="settingMenuHandler()">
                <img src="@/assets/icon/more.svg"/>
              </button>
              <!-- 控制麥克風/關閉視訊/鏡頭 -->
              <div v-if="menu.setting" class="pc__video-tool-wrap mb__video-tool-wrap">
                <button class="pc__video-tool mb__video-tool pointer" ref="audioIcon" :disabled="stream.close" @click="audioHandler()">
                  <img v-if="stream.audio" class="pc__video-tool--audio" src="@/assets/icon/microphone-2.svg"/>
                  <img v-else class="pc__video-tool--audio" src="@/assets/icon/microphone-slash.svg"/>
                </button>
                <button class="pc__video-tool mb__video-tool pointer" ref="closeIcon" :disabled="stream.close" @click="closeStreemHandler()">
                  <img class="pc__video-tool--close" src="@/assets/icon/add.svg"/>
                </button>
                <button class="pc__video-tool mb__video-tool pointer" ref="videoIcon" :disabled="stream.close" @click="videoHandler()">
                  <img v-if="stream.video" class="pc__video-tool--video" src="@/assets/icon/video.svg"/>
                  <img v-else class="pc__video-tool--video" src="@/assets/icon/video-slash.svg"/>
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import {TweenMax, TimelineMax, Power0, Power1, Power4} from 'gsap';

let emojiList = "😆,🤣,😍,🙄,😡,😱";

export default {
  name: 'Home',
  data() {
    return {
      videoStream: null,
      interval: null,
      time: 0,
      watch: 330,
      isMb: false,
      menu: {
        emoji: false,
        setting: false,
      },
      emojiArr: [],
      videoConstraints: { 
        audio: true, 
        video: true
      },
      animation: {
        live: null,
      },
      stream: {
        audio: true, 
        video: true,
        close: false,
      },
      user : {
        comment: '',
      },
      emojis: emojiList.split(','),
      commentList: [],
    }
  },
  mounted() {
    this.videoInit();
    // 判斷裝置
    this.isMb = this.checkIsMb();
  },
  components: {
  },
  methods: {
    checkIsMb(){
      let isIos = !!navigator.userAgent.match(/(iphone|ipad|ios|ipad)/ig);
      let isAndroid = !!navigator.userAgent.match(/(android)/ig);

      if(isIos || isAndroid){
        return true;
      }else{
        return false;
      }
    },
    // 視訊鏡頭 初始
    videoInit() {
      if(navigator.mediaDevices === undefined){
        console.log("沒有支援鏡頭")
      }else{
        navigator.mediaDevices.getUserMedia(this.videoConstraints)
        .then((mediaStream) => {
          this.videoStream = mediaStream;
          var video = this.$refs.myVideo;
          video.srcObject = mediaStream;
          video.onloadedmetadata = (e) => {
            video.play();
            this.liveAnimation();
            // 計數
            this.countTime();
            console.log(e)
          };
        })
        .catch(function(err) {
          console.log(err.name + ": " + err.message); 
        });
      }
    },
    // 電腦版 留言串自動滾動到最新(最底)
    commentScrollToBottom(){
      var element = this.$refs.comments;
      if(element){
        element.scrollTop = element.scrollHeight;
      }
    },
    // 按鈕點擊 動畫
    btnClickAnimation(ref) {
      let tl = new TimelineMax();
      let _ref = this.$refs[ref];
      tl.to(_ref, 0.1, {
        css: {
          transform: "scale(0.8)",
        },
        ease: Power0.easeNone,
        yoyo: true,
      }).to(_ref, 0.1, {
        css: {
          transform: "scale(1)",
        },
        ease: Power0.easeNone,
        yoyo: true,
      });
    },
    // live 動畫
    liveAnimation() {
      this.$refs.live.style.backgroundColor = "#f84e64";
      this.animation.live = TweenMax.to(this.$refs.live, 1, {
        css: {
          backgroundColor: "rgb(48, 48, 48)",
        },
        ease: Power0.easeNone,
        repeat: -1,
        yoyo: true,
      });
    },
    emojiEnter() {
      this.emojiAnimation();
    },
    // emoji 動畫
    emojiAnimation() {
      let tl = new TimelineMax();
      let _ref = this.$refs[`streamEmoji_${this.emojiArr.length-1}`];
      tl.set(_ref, {
        scale: 0.2,
        x: Math.random() * -100 + 30,
      }).to(_ref, 1, {
        y: -150 + Math.random() * -100,
        ease: Power4.easeOut,
        scale: 1,
      }).to(_ref, 0.6, {
        y: -200 + Math.random() * -10,
        ease: Power0.easeNone,
        repeat: -1,
        yoyo: true,
      }).to(_ref, 3, {
        x: -1000,
        ease: Power1.easeIn,
        scale: 0.6,
      });
    },
    // 送出留言
    submitComment() {
      if(this.user.comment != "" && !this.stream.close){
        this.btnClickAnimation("commentIcon");
        let obj = {
          account: 'account',
          comment: this.user.comment,
        }
        this.commentList.push(obj);
        // 清空
        this.user.comment = "";
      }
    },
    // emoji選單
    emojiMenuHandler() {
      this.btnClickAnimation("emojiIcon");
      this.menu.emoji = !this.menu.emoji;
    },
    // emoji選單
    settingMenuHandler() {
      this.btnClickAnimation("settingIcon");
      this.menu.setting = !this.menu.setting;
    },
    // 送出愛心
    submitHeart() {
      this.btnClickAnimation("heartIcon");
      this.emojiArr.push("❤️");
    },
    // 送出emoji
    submitEmoji(index) {
      this.btnClickAnimation("emoji_" + index);
      this.emojiArr.push(this.emojis[index]);
      // 關閉 emoji選單
      this.menu.emoji = false;
    },
    // audio 開關
    audioHandler() {
      this.btnClickAnimation("audioIcon");
      // audio
      if(this.stream.audio) {
        this.videoStream.getTracks()[0].enabled = false;
        this.stream.audio = false;
      }else {
        this.stream.audio = true;
        this.videoStream.getTracks()[0].enabled = true;
      }
    },
    // video 開關
    videoHandler() {
      this.btnClickAnimation("videoIcon");
      // video
      if(this.stream.video) {
        this.videoStream.getTracks()[1].enabled = false;
        this.stream.video = false;
      }else {
        this.stream.video = true;
        this.videoStream.getTracks()[1].enabled = true;
      }
    },
    // 串流 開關
    closeStreemHandler() {
      this.btnClickAnimation("closeIcon");
      this.stream.close = true;
      this.animation.live.kill();
      // 全部關閉
      this.videoStream.getTracks().forEach(
        function(track) { 
          track.stop(); 
        }
      );
      // 計數
      clearInterval(this.interval);
    },
    // 計數
    countTime() {
      this.interval = setInterval(() => {
        this.time++;
      }, 1000);
    },
  },
  watch: {
    commentList: {
      handler: function () {
        this.$nextTick(() => {
          this.commentScrollToBottom();
        });
      },
      deep: true,
      immediate: true,
    },
    time: {
      handler: function (val) {
        if(val == 0){
          clearInterval(this.interval);
        }
      },
      deep: true,
      immediate: true,
    },
  },
  computed: {
    timeLabel() {
      let sec = this.time % 60;
      let min = Math.floor(this.time / 60) % 60;
      let hour = Math.floor(this.time / 3600) % 24;
      let pd = (num) => (num + "").padStart(2, "0");
      return `${pd(hour)}：${pd(min)}：${pd(sec)}`;
    },
  },
}
</script>
<style lang="scss" scoped>
.fade-enter-active, .fade-leave-active {
  transition: opacity .5s;
}
.fade-enter, .fade-leave-to {
  opacity: 0;
}
.home {
  background-color: $bg-black;
  width: 100%;
  height: 100%;
  .pc {
    display: block;
    &__wrap {
      display: flex;
      align-items: flex-start;
      position: absolute;
      top: 10vh;
      left: 50%;
      transform: translateX(-50%);
      width: 100%;
      padding: 0px 50px;
      height: 80vh
    }
    &__video-wrap {
      width: 70%;
      height: 100%;
      overflow: hidden;
      margin-right: 20px;
      border-radius: 8px;
      video {
        height: 80vh;
        left: 50%;
        transform: translate(-50%);
      }
    }
    &__video-empty-wrap {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      width: 100px;
      height: 100px;
      border-radius: 50%;
      margin-right: 8px;
      background-color: $border-black;
      display: flex;
      align-items: center;
      justify-content: center;
      img {
        width: 50px;
      }
    }
    &__video-label {
      position: absolute;
      top: 20px;
      left: 50%;
      transform: translateX(-50%);
      display: flex;
      align-items: center;
      background-color: rgba($border-black, 0.8);
      padding: 2px 14px;
      border-radius: 30px;
      &--live {
        color: $white;
        background-color: rgba($red, 0.8);
        padding: 2px 10px;
        font-size: 10px;
        font-weight: 900;
        border-radius: 20px;
        letter-spacing: 2px;
        margin-right: 12px;
      }
      &--close {
        color: rgba($text-gray, 0.6);
        background-color: rgba($border-black, 0.8) !important;
      }
      &--time {
        color: $white;
        // background-color: rgba($border-black, 0.8);
        padding: 5px 14px;
        font-size: 14px;
        font-weight: 500;
        border-radius: 20px;
      }
      &--time-end {
        color: rgba($text-gray, 0.6);
      }
      &--watch {
        display: flex;
        align-items: center;
        color: $white;
        font-size: 10px;
        // background-color: rgba($border-black, 0.8);
        border-radius: 20px;
        padding: 5px 14px;
        font-weight: 500;
        margin-left: 12px;
        img {
          margin-right: 4px;
          width: 14px;
          height: 14px;
        }
      }
    }
    &__video-close {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      color: $text-gray;
      font-weight: 500;
      font-size: 20px;
    }
    &__video-tool-wrap {
      position: absolute;
      bottom: 30px;
      left: 50%;
      transform: translateX(-50%);
      display: flex;
      align-items: center;
      z-index: 2;
    }
    &__video-tool {
      background-color: rgba($border-black, 0.8);
      border-radius: 50%;
      width: 50px;
      height: 50px;
      margin-right: 20px;
      display: flex;
      align-items: center;
      justify-content: center;
      &:hover {
        border: 1px solid $text-gray;
      }
      &--audio {
        
      }
      &--video {

      }
      &--close {
        transform: rotate(45deg);
      }
    }
    &__video-emoji-wrap {
      position: absolute;
      right: 60px;
      bottom: 20px;
      display: flex;
      align-items: center;
      z-index: 1;
      > span {
        display: flex;
        align-items: center;
      }
    }
    &__video-emoji {
      font-size: 30px;
    }
    &__comment-wrap {
      background-color: $bg-black;
      border: 2px solid $border-black;
      width: 30%;
      height: 100%;
      border-radius: 8px;
      padding: 20px 0px 0px;
    }
    &__comment-item-wrap {
      padding: 0px 20px 20px;
      height: calc( 80vh - 70px);
      overflow-y: scroll;
    }
    &__comment-item {
      display: flex;
      align-items: flex-start;
      margin-bottom: 14px;
      &--end {
        color: $text-gray;
        font-size: 16px;
        margin-top: 30px;
      }
      &--empty {
        color: $text-gray;
        font-size: 16px;
        margin-top: 50px;
      }
      &--img-wrap {
        width: 30px;
        height: 30px;
        border-radius: 50%;
        margin-right: 8px;
        background-color: $border-black;
        display: flex;
        align-items: center;
        justify-content: center;
        img {
          width: 18px;
        }
      }
      &--text-wrap {
        padding-top: 3px;
        width: calc(100% - 30px);
        text-align: left;
      }
      &--user-name {
        color: $text-gray;
        margin-right: 12px;
      }
      &--comment {
        color: $white;
        font-weight: 500;
        word-break: break-word;
      }
    }
    &__comment-bar-wrap {
      position: absolute;
      left: 0;
      bottom: 0;
      width: 100%;
      border-top: 2px solid $border-black;
      background-color: $block-black;
      padding: 10px 14px;
      border-bottom-left-radius: 8px;
      border-bottom-right-radius: 8px;
      display: flex;
      align-items: center;
    }
    &__comment-bar {
      display: flex;
      align-items: center;
      justify-content: flex-start;
      width: 100%;
      &--input-wrap {
        width: 80%;
        background-color: $border-black;
        border-radius: 8px;
      }
      input {
        outline: none;
        border: none;
        color: $white;
        width: 100%;
        font-size: 16px;
        padding: 10px 45px 10px 12px;
        background-color: $border-black;
        border-radius: 8px;
        &::placeholder {
          color: $text-gray;
        }
      }
      &--input-icon {
        position: absolute;
        top: 5px;
        right: 8px;
      }
      &--tool-wrap {
        width: 20%;
        display: flex;
        align-items: center;
        justify-content: space-between;
        padding-left: 16px;
      }
      &--emoji-wrap {
        position: absolute;
        top: -70px;
        right: 0;
        background-color: rgba($border-black, 0.8);
        font-size: 28px;
        padding: 5px 15px;
        display: flex;
        align-items: center;
        justify-content: space-evenly;
        border-radius: 30px;
        span {
          padding: 0px 5px;
        }
      }
      &--triangle {
        width: 0;
        height: 0;
        border-style: solid;
        border-width: 10px 8px 0 8px;
        border-color: rgba($border-black, 0.8) transparent transparent transparent;
        position: absolute;
        bottom: -8px;
        right: 30px;
      }
    }
  }
  .mb {
    display: none;
    &__wrap {
      position: absolute;
      top: 0;
      left: 50%;
      transform: translateX(-50%);
      overflow: hidden;
      width: 100%;
      height: 100vh;
    }
    &__video-wrap {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      video {}
    }
    &__video-label-wrap {
      background-color: rgba($border-black, 0.8);
      padding: 2px 14px;
      border-radius: 20px;
    }
    &__video-label {
      top: 50px;
      width: 100vw;
      justify-content: space-between;
      background-color: unset;
      padding: 0 20px;
      &--time {
        background-color: rgba($border-black, 0.8);
      }
    }
    &__btn-wrap {
      background-color: rgba($border-black, 0.8);
      width: 40px;
      height: 40px;
      border-radius: 50%;
      display: flex;
      align-items: center;
      justify-content: center;
    }
    &__video-tool-wrap {
      top: 0;
      display: block;
      margin-top: 50px;
      right: -20px;
    }
    &__video-tool {
      width: 40px;
      height: 40px;
      margin-bottom: 12px;
      &:hover {
        border: none;
      }
    }
  }
}
@media screen and (max-width: 992px){
  .home {
    .pc {
      display: none;
    }
    .mb {
      display: block;
    }
  }
}
</style>