<template>
  <view>
    <form @submit="feedback">
      <text class="info">新建反馈</text>
      <input name="title" placeholder="请输入标题" />
      <picker name="label" @change="pickerHandle" value="{{feedbackIndex}}" range="{{feedbackType}}">
        <view class="picker">
          <text>请选择类型:</text>
          <text>{{feedbackType[feedbackIndex]}}</text>
        </view>
      </picker>
      <textarea name="content" placeholder="请输入内容" />
      <view class="help">
        <text>感谢您的反馈，您的反馈将以issue的形式提交到scuplus-wechat的github，您可以通过访问https://github.com/mohuishou/scuplus-wechat/issues
                    来获取您的反馈动态，同时也可以关注项目的开发进度，当然如果您有更好的实现欢迎提交PR
                  </text>
      </view>
      <button formType="submit">提交反馈</button>
    </form>
  </view>
</template>

<script>
  import wepy from 'wepy'
  import {
    version
  } from 'config'
  import HttpMixin from "mixins/http";
  import ToastMixin from "mixins/toast";
  import db from "util/db"
  export default class Feedback extends wepy.page {
    mixins = [HttpMixin, ToastMixin]
    components = {}
    config = {
      navigationBarTitleText: '反馈'
    }
    data = {
      feedbackType: ['Bug', '功能建议', '其他'],
      feedbackIndex: 0
    }
    methods = {
      pickerHandle(e) {
        this.feedbackIndex = e.detail.value
      },
      async feedback(e) {
        let param = e.detail.value
        if (!param.title || !param.content) {
          this.Alert('标题或内容不能为空')
          return
        }
        try {
          let info = wepy.getSystemInfoSync()
          param.title = `[${db.Get("nickName")}]: ` + param.title
          param.brand = info.brand
          param.model = info.model
          param.system = info.system
          param.version = info.version
          param.SDKVersion = info.SDKVersion
          param.scuplusVersion = version
          param.label = this.feedbackType[param.label]
        } catch (error) {
          this.Alert('获取系统信息错误, 请稍候再试')
          return
        }
        const resp = await this.POST('/user/feedback', param)
        wepy.navigateBack({
          delta: 1
        })
      }
    }
    onLoad(option) {
      if ('index' in option) this.feedbackIndex = option.index
    }
    onShareAppMessage(options) {
      return {
        title: 'we川大-反馈'
      }
    }
  }
</script>

<style lang="less">
  @border: 1px solid #ddd;
  page {
    background: #eee;
  }
  form {
    color: #333;
    .info {
      padding: 0.5rem;
      display: block;
      font-size: 0.8rem;
      color: #999;
    }
    .help {
      padding: 0.5rem;
      font-size: 0.8rem;
      color: #999;
    }
    .picker {
      display: flex;
      justify-content: space-between;
      border-top: @border; // font-size: 0.8rem;
      padding: 0.5rem;
      background: #fff;
    }
    input {
      border-top: @border;
      padding: 0.5rem;
      background: #fff;
    }
    textarea {
      background: #fff;
      border-top: @border;
      padding: 0.5rem;
      border-bottom: @border;
      width: 100%;
      height: 8rem;
    }
    button {
      margin-top: 1rem;
      background: #f06292;
      color: #fff;
    }
  }
</style>
