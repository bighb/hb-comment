<template>
  <view class="hb-comment">
    <!-- 评论主体-start -->
    <view class="comment-list" v-if="commentData.length != 0">
      <!-- 评论主体-顶部数量及发表评论按钮-start -->
      <view class="comment-num">
        <view class="num_text_wrap">
          <view class="text_key">评论</view>
          <view>{{ commentSize }}</view>
        </view>
        <view class="add-btn" @click="commentInput"> 写评论 </view>
      </view>
      <!-- 评论主体-顶部数量及发表评论按钮-end -->
      <!-- 评论列表-start -->
      <view
        class="comment-box"
        v-for="(item, index) in commentData"
        :key="item.id"
      >
        <view class="comment-box-item">
          <view>
            <image
              :src="item.headimgUrl"
              mode="aspectFill"
              class="avatar"
            ></image>
          </view>
          <view class="comment-main">
            <!-- 父评论体-start -->
            <view class="comment-main-top">
              <view class="nick-name-box">
                <view class="nick-name">{{ item.nickName }}</view>
              </view>
              <view class="zan-box" @click="like(item)">
                <view
                  :class="item.like ? 'isLike' : 'notLike'"
                  v-if="item.likes > 0"
                  >{{ item.likes }}</view
                >
                <image
                  class="icon_like"
                  v-if="!item.like"
                  src="../../../../static/like.png"
                />
                <image
                  class="icon_like"
                  v-else
                  src="../../../../static/liked.png"
                />
              </view>
            </view>
            <view class="comment-main-content">
              {{ item.discuss }}
            </view>
            <view class="comment-main-foot">
              <view class="foot-time">{{ item.publishTime }}</view>
              <view
                class="foot-btn"
                @click="reply(item.nickName, item.id, item.wxUserId)"
                >回复</view
              >
              <view
                class="foot-btn"
                v-if="item.owner"
                @click="confirmDelete(item.id)"
                >删除</view
              >
            </view>
            <!-- 父评论体-end -->
            <!-- 子评论列表-start -->
            <view class="comment-sub-box">
              <view
                v-if="item.discussNum && !item.hasShowMore"
                class="open"
                @click="openComment(item)"
              >
                <text>展开{{ item.discussNum }}条回复</text>
                <image src="../../../../static/comment_open.png"></image>
              </view>
              <view
                class="comment-sub-item"
                v-for="each in item.children"
                :key="each.id"
              >
                <view>
                  <image
                    :src="each.headimgUrl"
                    mode="aspectFill"
                    class="avatar"
                  >
                  </image>
                </view>
                <view class="comment-main">
                  <view class="sub-comment-main-top">
                    <view class="nick_name_warp">
                      <view class="nick-name">{{ each.nickName }}</view>
                      <view class="reply_wrap" v-if="each.replyUserName">
                        <image
                          class="reply_icon"
                          src="../../../../static/reply.png"
                        ></image>
                        <view class="nick-name">{{ each.replyUserName }}</view>
                      </view>
                    </view>

                    <view class="zan-box" @click="like(each)">
                      <view
                        :class="each.like ? 'isLike' : 'notLike'"
                        v-if="each.likes > 0"
                        >{{ each.likes }}</view
                      >
                      <image
                        class="icon_like"
                        v-if="!each.like"
                        src="../../../../static/like.png"
                      />
                      <image
                        class="icon_like"
                        v-else
                        src="../../../../static/liked.png"
                      />
                    </view>
                  </view>
                  <view class="comment-main-content">
                    {{ each.discuss }}
                  </view>
                  <view class="comment-main-foot">
                    <view class="foot-time">{{ each.publishTime }}</view>
                    <view
                      class="foot-btn"
                      @click="reply(item.nickName, item.id, item.wxUserId)"
                    >
                      回复</view
                    >
                    <view
                      class="foot-btn"
                      v-if="each.owner"
                      @click="confirmDelete(each.id)"
                      >删除
                    </view>
                  </view>
                </view>
              </view>
            </view>
            <!-- 子评论列表-end -->
          </view>
        </view>
      </view>
      <!-- 评论列表-end -->
    </view>
    <!-- 评论主体-end -->
    <!-- 无评论-start -->
    <view class="comment-none" v-else>
      <view class="comment-none-key"> 暂无评论,</view>
      <view class="comment-none-val" @click="commentInput">抢沙发</view>
    </view>
    <!-- 无评论-end -->
    <!-- 新增评论-start -->
    <view class="comment-submit-box" v-if="submit" @click="closeInput">
      <!-- 下边的click.stop.prevent用于让上边的click不传下去，以防点到下边的空白处触发closeInput方法 -->
      <view
        class="comment-add"
        @click.stop.prevent="stopPrevent"
        :style="'bottom:' + KeyboardHeight + 'px'"
      >
        <view class="comment-submit">
          <view class="replayTag">
            <view class="replay_key" v-if="commentReq.reUser">正在回复</view>
            <view v-if="commentReq.reUser">{{ commentReq.reUser }}</view>
          </view>
          <view class="btn-click" @click="add">发布</view>
        </view>
        <textarea
          class="textarea"
          v-model="commentReq.content"
          :placeholder="placeholder"
          :adjust-position="false"
          :show-confirm-bar="false"
          @blur="blur"
          :focus="focus"
          maxlength="100"
          :styles="styles"
        ></textarea>
      </view>
    </view>
    <!-- 新增评论-end -->
  </view>
</template>

<script>
export default {
  name: "hb-comment",
  props: {
    cmData: {
      type: Array,
      default: () => {
        return [];
      },
    },
    commentSize: {
      type: Number,
      default: () => 0,
    },
    deleteTip: {
      type: String,
      default: () => {
        return "操作不可逆，如果评论下有子评论，也将被一并删除，确认？";
      },
    },
  },
  data() {
    return {
      styles: {
        padding: "16rpx",
        "text-align": "justify",
      },
      placeholder: "请输入评论",
      commentReq: {
        pId: null, // 评论父id
        content: "", // 评论内容
        reUser: "", //被回复的人昵称
        wxUserId: "", //被回复的人wxid
      },
      focus: false, // 输入框自动聚焦
      submit: false, // 弹出评论
      KeyboardHeight: 0, // 键盘高度
      commentData: [],
    };
  },
  watch: {
    cmData: {
      handler: function (newVal, oldVal) {
        this.init(newVal);
      },
      immediate: true,
    },
  },
  mounted: function () {
    uni.onKeyboardHeightChange((res) => {
      this.KeyboardHeight = res.height;
    });
  },
  methods: {
    // 初始化评论
    init(cmData) {
      this.commentData = cmData;
    },
    // 展开评论
    openComment(item) {
      this.$emit("openComment", item);
    },
    // 查询子评论完成-本地将子评论增加到父评论children中去
    addSubList(data, item) {
      // 将子评论增加到父评论children中去
      let array = this.commentData;
      let subArr = data;
      let resultArr = [];
      for (let j = 0; j < array.length; j++) {
        const element = array[j];
        let obj = {
          ...element,
          children: element.children ? element.children : [],
        };
        for (let i = 0; i < subArr.length; i++) {
          const subElement = subArr[i];
          if (subElement.parentId === element.id) {
            obj.children.push(subElement);
          }
        }
        resultArr.push(obj);
      }
      this.commentData = resultArr;
      let idx = this.commentData.findIndex((e) => {
        return item.id === e.id;
      });
      this.commentData[idx].hasShowMore = true;
    },
    // 没用的方法，但不要删
    stopPrevent() {},
    // 回复评论
    reply(reUser, pId, wxUserId) {
      this.commentReq.pId = pId;
      this.commentReq.wxUserId = wxUserId;
      this.commentReq.content = "";
      this.commentReq.reUser = reUser;
      if (reUser) {
        //先判断当前回复父级是否包含子评论，是否展开评论
        let item = this.commentData.find((e) => {
          return e.id === pId;
        });
        this.commentReq.currentItem = item;
      }

      this.commentInput();
    },
    // 删除评论前确认
    confirmDelete(commentId) {
      var that = this;
      uni.showModal({
        title: "警告",
        content: that.deleteTip,
        confirmText: "确认删除",
        success: function (res) {
          if (res.confirm) {
            that.$emit("del", commentId);
          }
        },
      });
    },
    // 新增评论
    add() {
      if (!this.commentReq.content) {
        return uni.showToast({
          title: "请输入评论",
          duration: 2000,
          icon: "none",
        });
      }
      this.$emit("add", this.commentReq);
    },
    // 点赞评论
    like(item) {
      this.$emit("like", item);
    },
    // 新增完成，本地修改数据结构
    addComplete(comment) {
      if (comment.parentId == 0) {
        // 写评论
        this.commentData.unshift(comment);
      } else {
        // 回复
        let resultArr = [];
        for (let i = 0; i < this.commentData.length; i++) {
          let element = this.commentData[i];
          let obj = {
            ...element,
            children: element.children ? element.children : [],
          };
          if (element.id === comment.parentId) {
            obj.children.push(comment);
          }
          resultArr.push(obj);
        }
        this.commentData = [...resultArr];
      }
      this.commentReq.content = null;
      this.tagClose();
      this.closeInput();
    },
    // 点赞完成-本地修改点赞结果
    likeComplete(commentId) {
      for (let i = 0; i < this.commentData.length; i++) {
        if (this.commentData[i].id == commentId) {
          this.commentData[i].like
            ? this.commentData[i].likes--
            : this.commentData[i].likes++;
          this.commentData[i].like = !this.commentData[i].like;
          break;
        }
        for (let j = 0; j < this.commentData[i].children.length; j++) {
          if (this.commentData[i].children[j].id == commentId) {
            this.commentData[i].children[j].like
              ? this.commentData[i].children[j].likes--
              : this.commentData[i].children[j].likes++;
            this.commentData[i].children[j].like =
              !this.commentData[i].children[j].like;
            break;
          }
        }
      }
    },
    // 删除完成-本地删除评论
    deleteComplete(commentId) {
      for (let i = 0; i < this.commentData.length; i++) {
        if (this.commentData[i].id == commentId) {
          this.commentData.splice(i, 1);
          break;
        }
        for (let j = 0; j < this.commentData[i].children.length; j++) {
          if (this.commentData[i].children[j].id == commentId) {
            this.commentData[i].children.splice(j, 1);
            break;
          }
        }
      }
    },

    // 输入框失去焦点
    blur() {
      this.focus = false;
    },
    // 标签关闭
    tagClose() {
      this.commentReq.reUser = "";
      this.commentReq.pId = null;
      this.commentReq.wxUserId = "";
    },
    // 输入评论
    commentInput() {
      this.submit = true;
      setTimeout(() => {
        this.focus = true;
      }, 50);
    },
    // 关闭输入评论
    closeInput() {
      this.focus = false;
      this.submit = false;
    },
  },
};
</script>

<style lang="scss" scoped>
view {
  box-sizing: border-box;
}
.icon_like {
  width: 26rpx;
  height: 26rpx;
}
.hb-comment {
  padding: 10rpx;
  .comment-num {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 20rpx 0;
    .num_text_wrap {
      display: flex;
      font-size: 28rpx;
      font-weight: bolder;
      color: #333333;
      .text_key {
        padding-right: 10rpx;
      }
    }
    .add-btn {
      font-size: 26rpx;
      font-weight: bolder;
      color: #596d96;
    }
  }

  .comment-box {
    padding: 10rpx 0;
    .comment-box-item {
      display: flex;
      justify-content: space-between;
      .avatar {
        width: 62rpx;
        height: 62rpx;
        border-radius: 50%;
      }

      .comment-main {
        padding-left: 20rpx;
        flex-grow: 1;
        .comment-main-top {
          padding-top: 6rpx;
          display: flex;
          justify-content: space-between;
          .nick-name-box {
            display: flex;
            align-items: center;
            flex-grow: 1;
            .nick-name {
              font-size: 26rpx;
              font-weight: 400;
              color: #666666;
              overflow: hidden;
              text-overflow: ellipsis;
              white-space: nowrap;
              max-width: 70%;
            }
          }
          .zan-box {
            display: flex;
            align-items: center;
            flex-shrink: 0;
            .isLike {
              font-size: 24rpx;
              padding-right: 10rpx;
              color: #f97249;
            }

            .notLike {
              font-size: 24rpx;
              padding-right: 10rpx;
              color: #999999;
            }
          }
        }
        .comment-main-content {
          font-size: 28rpx;
          color: #333333;
          padding: 10rpx 0 10rpx 0;
        }

        .comment-main-foot {
          display: flex;
          font-size: 22rpx;
          .foot-time {
            font-size: 24rpx;
            font-weight: 400;
            color: #666666;
          }
          .foot-btn {
            padding-left: 10rpx;
            font-size: 24rpx;
            color: #666666;
          }
        }
        .comment-sub-box {
          .open,
          .close {
            display: flex;
            align-items: center;
            &:before {
              display: block;
              content: "";
              width: 32rpx;
              height: 1px;
              background: #a0a0a0;
              margin-right: 16rpx;
            }

            text {
              font-size: 24rpx;
              font-family: PingFangSC-Medium, PingFang SC;
              font-weight: 500;
              color: #666666;
            }

            image {
              width: 25rpx;
              height: 20rpx;
              margin-left: 10rpx;
            }
          }
          .comment-sub-item {
            display: flex;
            justify-content: space-between;
          }
          padding: 20rpx 0;
          .sub-comment-main-top {
            padding-top: 6rpx;
            display: flex;
            justify-content: space-between;
            .nick_name_warp {
              display: flex;
              align-items: center;
              flex-grow: 1;
              .nick-name {
                font-size: 26rpx;
                font-weight: 400;
                color: #666666;
                overflow: hidden;
                text-overflow: ellipsis;
                white-space: nowrap;
                max-width: 70%;
              }
              .reply_wrap {
                display: flex;
                align-items: center;
                flex-grow: 1;
                .reply_icon {
                  width: 12rpx;
                  height: 19rpx;
                  padding: 0 15rpx;
                }
              }
            }

            .zan-box {
              display: flex;
              align-items: center;
              flex-shrink: 0;
              .isLike {
                font-size: 24rpx;
                padding-right: 10rpx;
                color: #f97249;
              }

              .notLike {
                font-size: 24rpx;
                padding-right: 10rpx;
                color: #999999;
              }
            }
          }
        }
      }
    }
  }
  .comment-none {
    padding: 20rpx;
    width: 100%;
    color: #999999;
    display: flex;
    font-size: 28rpx;
    justify-content: center;
    .comment-none-key {
      margin-right: 10rpx;
    }
    .comment-none-val {
      color: #596d96;
    }
  }
  .comment-submit-box {
    position: fixed;
    display: flex;
    align-items: flex-end;
    z-index: 9900;
    left: 0;
    top: var(--window-top);
    bottom: 0;
    background-color: rgba($color: #000000, $alpha: 0.5);
    width: 100%;
    .comment-add {
      position: fixed;
      background-color: #ffffff;
      width: 100%;
      padding: 5rpx;
      border: 1px solid #ddd;
      transition: 0.3s;
      -webkit-transition: 0.3s;
      .comment-submit {
        padding: 5rpx 20rpx 0 20rpx;
        border-bottom: 1px dashed #ddd;
        display: flex;
        justify-content: space-between;
        align-items: center;
        .btn-click {
          color: #596d96;
          font-size: 28rpx;
          padding: 10rpx;
          font-weight: bold;
        }

        .replayTag {
          display: flex;
          font-size: 24rpx;
          color: #333333;
          .replay_key {
            padding-right: 20rpx;
          }
        }
      }
      .textarea {
        height: 100px;
        width: 100%;
        font-size: 24rpx;
        color: #333333;
      }
    }
  }
}
</style>
