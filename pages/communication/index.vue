<template>
  <view>
    <!-- 聊天区域 -->
    <scroll-view :style="{'height':`${height}px`}" :refresher-threshold="10" :scroll-top="scrollTop" :refresher-triggered="triggered" refresher-background="#efefef" refresher-enabled scroll-y @refresherrefresh="nextPage">
      <view class="chat-area" id="chat-area">
        <block v-for="(item,index) of msgList" :key="index">
          <view class="message-box">
            <!-- 左头像 -->
            <view class="avatar">
              <template v-if="item.flow == 'in'">
                <image src="@/static/logo.png" />
              </template>
            </view>
            <!-- 内容 -->
            <view class="content" :class="{'we':item.flow == 'out'}">
              <!-- 文本消息 -->
              <template v-if="item.type == 'TIMTextElem'">
                <view class="message">
                  <view class="text">
                    <text>{{item.payload.text}}</text>
                  </view>
                </view>
              </template>
              <!-- 图片消息 -->
              <template v-else-if="item.type == 'TIMImageElem'">
                <view class="message" @click="previewImage(item)">
                  <image :src="item.payload.imageInfoArray[1].url" mode="widthFix" :style="{'width':`${item.payload.imageInfoArray[1].width}px`}" />
                </view>
              </template>
              <!-- 视频消息 -->
              <template v-else-if="item.type == 'TIMVideoFileElem'">
                <view class="message">
                  <view class="video" @click="linkVideoPage(item)">
                    <text class="iconfont iconiconset0481" />
                  </view>
                </view>
              </template>
              <!-- 音频消息 -->
              <template v-else-if="item.type == 'TIMSoundElem'">
                <view class="message">
                  <view class="text" @click="playAudio(item)">
                    <template v-if="item.flow == 'in'">
                      <text class="iconfont iconertongerbihou-shengyinyichang" />
                      <text>{{`${item.payload.second}"`}}</text>
                    </template>
                    <template v-else>
                      <text>{{`${item.payload.second}"`}}</text>
                      <text class="iconfont iconertongerbihou-shengyinyichang" />
                    </template>
                  </view>
                </view>
              </template>
            </view>
            <!-- 右头像 -->
            <view class="avatar">
              <template v-if="item.flow == 'out'">
                <image src="@/static/uni-logo.png" />
              </template>
            </view>
          </view>
        </block>
      </view>
    </scroll-view>
    <!-- 底部输入区域 -->
    <view class="bottom-area">
      <view class="input-message">
        <template v-if="recorder">
          <text class="iconfont iconjianpan" @click="changeRecorder" />
          <view class="input" :class="{'recorder':recorder}">
            <text>按住说话</text>
          </view>
        </template>
        <template v-else>
          <text class="iconfont iconyuyintubiao" @click="changeRecorder" />
          <view class="input">
            <input v-model="message" type="text" :cursor-spacing="20" />
          </view>
        </template>
        <view class="send-message" v-if="isInput" @click="sendMessage">
          <text>发送</text>
        </view>
        <view class="add-media" v-else @click="openOptions">
          <text class="iconfont iconjiahao" />
        </view>
      </view>
      <view class="options" :class="{'open':open}">
        <block v-for="(item,index) of options" :key="index">
          <view class="option-box" @click="handleClick(index)">
            <view>
              <text class="iconfont" :class="item.icon" />
            </view>
            <text>{{item.text}}</text>
          </view>
        </block>
      </view>
    </view>
  </view>
</template>

<script>
import { mock1 } from '@/assets/js/mock1.js'
import { mock2 } from '@/assets/js/mock2.js'
import { mock3 } from '@/assets/js/mock3.js'
export default {
  data() {
    return {
      msgList: [],
      options: [{
        icon: "icontupian",
        text: "图片"
      }, {
        icon: "iconduanshipin",
        text: "短视频"
      }],
      open: false,
      isInput: false,
      recorder: false,
      triggered: false,
      _freshing: false,
      height: 0,
      scrollTop: 0,
      userCode: "",
      message: "",
      nextReqMessageID: "",
      systemInfo: {},
      page: 0,
      pages: 2
    }
  },
  onLoad({ userCode }) {
    uni.getSystemInfo({
      success: (result) => {
        this.systemInfo = result
      }
    })
    this.getMessageList(this.page, this.scrollViewBottom)
  },
  onReady() {
    this.scrollViewHeight()
  },
  watch: {
    open: function (value) {
      this.scrollViewHeight()
    },
    message: function (value) {
      this.isInput = value ? true : false
    },
  },
  methods: {
    // 获取会话消息
    async getMessageList(page, callback = "") {
      if (this._freshing) return;
      this._freshing = true;
      this.triggered = true;
      const result = await this.getMockData(page)
      const { msgList } = this
      this.msgList = [...result, ...msgList]
      this.triggered = false;
      this._freshing = false;
      if (callback) callback()
    },
    getMockData(page) {
      return new Promise((resolve, reject) => {
        const data = [mock1, mock2, mock3]
        setTimeout(() => {
          resolve(data[page])
        }, 1000)
      })
    },
    // 打开底部媒体选项
    openOptions() {
      this.open = !this.open
      this.scrollViewHeight()
    },
    // 切换语音
    changeRecorder() {
      this.recorder = !this.recorder
    },
    // 聊天区域高度
    scrollViewHeight() {
      this.$nextTick(() => {
        setTimeout(() => {
          const inputMessage = uni.createSelectorQuery().in(this).select(".input-message")
          inputMessage.boundingClientRect(({ top }) => {
            const { windowHeight } = this.systemInfo
            this.height = top
          }).exec()
        }, 100)
      })
    },
    // 回到底部
    scrollViewBottom() {
      this.$nextTick(() => {
        setTimeout(() => {
          const chatArea = uni.createSelectorQuery().in(this).select("#chat-area")
          chatArea.boundingClientRect(({ height }) => {
            this.scrollTop = height
          }).exec()
        }, 100)
      })
    },
    // 下一页消息记录
    nextPage() {
      if (this._freshing) return;
      if (this.page == this.pages) return
      this.page++
      this.getMessageList(this.page)
    },
  },
}
</script>

<style>
page {
  background: #efefef;
}
</style>
<style lang="scss" scoped>
.chat-area {
  padding: 15rpx 0;
  box-sizing: border-box;

  .message-box {
    display: flex;
    padding: 15rpx 0;

    .avatar {
      flex: 0 0 120rpx;

      > image {
        display: block;
        width: 80rpx;
        height: 80rpx;
        margin: 0 auto;
        border-radius: 100rpx;
      }
    }

    .content {
      flex: 1;
      display: flex;

      &.we {
        justify-content: flex-end;

        .message {
          .text {
            border-radius: 40rpx 40rpx 0 40rpx;
            background: #ff8965;

            text {
              color: #fff;
            }

            .iconfont {
              display: inline-block;
              font-size: 40rpx;
              margin-left: 10rpx;
            }
          }
        }
      }

      .message {
        max-width: 508rpx;

        .text {
          display: flex;
          align-items: center;
          padding: 20rpx 30rpx;
          border-radius: 40rpx 40rpx 40rpx 0;
          background: #fff;

          text {
            color: #333333;
            font-size: 30rpx;
            white-space: normal;
            word-break: break-all;
          }

          .iconfont {
            display: inline-block;
            font-size: 40rpx;
            margin-right: 10rpx;
          }
        }

        image {
          display: block;
          max-width: 100%;
        }

        .video {
          width: 200rpx;
          height: 200rpx;
          position: relative;
          background: #000;

          .iconfont {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            z-index: 10;
            color: #fff;
            font-size: 100rpx;
          }
        }
      }
    }
  }
}

.bottom-area {
  position: fixed;
  left: 0;
  right: 0;
  bottom: 0;
  background: #efefef;

  .input-message {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 40rpx 35rpx;
    border: 2rpx solid #d3d3d3;

    > text {
      color: #666666;
      font-size: 48rpx;
    }

    .input {
      flex: 1;
      margin: 0 35rpx;
      height: 60rpx;

      > input {
        background: #ffffff;
        border-radius: 100rpx;
        height: 60rpx;
        padding: 0 30rpx;
      }

      &.recorder {
        display: flex;
        align-items: center;
        justify-content: center;
        background: #ffffff;
        border-radius: 100rpx;

        > text {
          color: #666666;
          font-size: 30rpx;
        }
      }
    }

    .add-media {
      width: 48rpx;
      height: 48rpx;
      display: flex;
      align-items: center;
      justify-content: center;
      background: linear-gradient(0deg, #ff424e, #fd6b59);
      border-radius: 100rpx;

      > text {
        color: #fff;
        font-size: 30rpx;
      }
    }

    .send-message {
      background: linear-gradient(0deg, #ff424e, #fd6b59);
      width: 94rpx;
      height: 60rpx;
      display: flex;
      align-items: center;
      justify-content: center;
      border-radius: 12rpx;

      > text {
        color: #ffffff;
        font-size: 24rpx;
      }
    }
  }

  .options {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
    padding: 0 25rpx;
    height: 0;
    max-height: 0;
    overflow: hidden;
    transition: all 0.2s;

    .option-box {
      flex: 0 0 25%;
      display: flex;
      flex-direction: column;
      align-items: center;
      padding: 25rpx 0;
      border-radius: 10rpx;

      > view {
        width: 112rpx;
        height: 112rpx;
        background: #fff;
        display: flex;
        align-items: center;
        justify-content: center;
        margin-bottom: 10rpx;

        .iconfont {
          font-size: 54rpx;
          color: #333;
        }
      }

      > text {
        color: #333333;
        font-size: 22rpx;
      }
    }

    &::after {
      content: "";
      flex: auto;
    }

    &.open {
      height: auto;
      max-height: 1000rpx;
    }
  }
}
</style>