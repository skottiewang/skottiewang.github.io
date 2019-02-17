---
id: 296
title: 操作反馈类组件（Modal，Toast，ActionSheet）小结
date: 2018-01-25T16:01:48+00:00
author: skottiewang
layout: post
guid: http://skottiewang.com/?p=296
permalink: '/2018/01/25/%e6%93%8d%e4%bd%9c%e5%8f%8d%e9%a6%88%e7%b1%bb%e7%bb%84%e4%bb%b6%ef%bc%88modal%ef%bc%8ctoast%ef%bc%8cactionsheet%ef%bc%89%e5%b0%8f%e7%bb%93/'
categories:
  - Code
---
Modal,Toast,ActionSheet这三个组件的写法思路几乎完全相同，这里以ActionSheet为例做一个小结。
    
- ActionSheet子组件的vue文件

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-html">  &lt;div style="bottom: 0px; position: fixed;width: 100%;"&gt;
    &lt;transition name="fade" &gt;
      &lt;div v-if="visible" style="z-index: 100;position: relative;"&gt;
        &lt;div class="um-action-sheet" v-if="!isShare"&gt;
          &lt;h3 class="um-action-sheet-title" v-if="title !== undefined"&gt;{{ title }}&lt;/h3&gt;
          &lt;div class="um-action-sheet-message" v-if="message !== undefined"&gt;{{ message }}&lt;/div&gt;
          &lt;div class="um-action-sheet-button-list"&gt;
            &lt;div :class="wrapCls(index)"
                 v-for="(item, index, text) in btnGroup"
                 @click="btnClick(item)"&gt;
         &lt;span v-if="cancelButtonIndex === index"
               class="um-action-sheet-button-list-item-cancel-button-mask"&gt;
         &lt;/span&gt;
              {{ btnGroup[index].text }}
            &lt;/div&gt;
          &lt;/div&gt;
        &lt;/div&gt;
        &lt;div class="um-action-sheet-share" v-if="isShare"&gt;
          &lt;div class="um-action-sheet-share-message"&gt;{{message}}&lt;/div&gt;
          &lt;div class='um-action-sheet-share-list' v-if="multipleLine" v-for="(row, rowIndex, text) in iconGroup"&gt;
            &lt;div class="um-action-sheet-share-list-item" v-for="(col, colIndex, text) in row"
                 @click="iconClick(rowIndex, colIndex)"&gt;
              &lt;div class="um-action-sheet-share-list-item-icon"&gt;
                &lt;img :src="col.icon" style="width: 36px; height: 36px"/&gt;
              &lt;/div&gt;
              &lt;div class="um-action-sheet-share-list-item-title"&gt;{{ col.title }}&lt;/div&gt;
            &lt;/div&gt;
          &lt;/div&gt;

          &lt;div class='um-action-sheet-share-list' v-if="!multipleLine"&gt;
            &lt;div class="um-action-sheet-share-list-item" v-for="(item, index, text) in iconGroup"&gt;
              &lt;div class="um-action-sheet-share-list-item-icon" @click="iconClick(item)"&gt;
                &lt;img :src="item.icon" style="width: 36px; height: 36px"/&gt;
              &lt;/div&gt;
              &lt;div class="um-action-sheet-share-list-item-title"&gt;{{ item.title }}&lt;/div&gt;
            &lt;/div&gt;
          &lt;/div&gt;

          &lt;div class="um-action-sheet-share-cancel-button" @click="cancelClick"&gt;{{ cancelButtonText }}&lt;/div&gt;
        &lt;/div&gt;
      &lt;/div&gt;
    &lt;/transition&gt;
    &lt;div class="um-action-sheet-mask" v-if='visible' @click="maskClick"&gt;&lt;/div&gt;
  &lt;/div&gt;
</code></pre>

<!--more-->

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-javascript">&lt;script&gt;
  export default {
    name: 'ActionSheetVue',
    data () {
      return {
        visible: '',
        title: '',
        message: '',
        callback: '',
        isShare: false,
        btnGroup: '',
        iconGroup: '',
        multipleLine: '',
        maskClosable: true,
        cancelButtonText: '取消',
        cancelButtonIndex: '',
        destructiveButtonIndex: ''
      }
    },
    methods: {
      cancelClick () {
        this.visible = false
      },
      maskClick () {
        if (this.maskClosable) {
          this.visible = false
        }
      },
      iconClick (rowIndex, colIndex) {
        console.log(rowIndex, colIndex)
        if (this.callback && typeof this.callback === 'function') {
          const promise = this.callback(rowIndex, colIndex)
          promise.then(() =&gt; {
            this.visible = false
          })
        } else {
          this.visible = false
        }
      },
      btnClick (item) {
        if (this.callback && typeof this.callback === 'function') {
          const promise = this.callback(item)
          promise.then(() =&gt; {
            this.visible = false
          })
        } else {
          this.visible = false
        }
      },
      wrapCls (index) {
        return [
          'um-action-sheet-button-list-item',
          {
            [`um-action-sheet-button-list-item-destructive-button`]: this.destructiveButtonIndex === index,
            [`um-action-sheet-button-list-item-cancel-button`]: this.cancelButtonIndex === index
          }
        ]
      }
    }
  }
&lt;/script&gt;
</code></pre>

  * ActionSheet.js文件中定义ActionSheet的不同方法
  * 不同的方法对应的就是ActionSheet的不同类型
  * 不同的方法分别对子组件data中的属性进行不同的定义

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-javascript">let ActionSheet = function (options) {
  if (!instance) {
    initInstance()
  }
  ShowActionSheet(options)
}

ActionSheet.share = function (options) {
  if (!instance) {
    initInstance()
  }
  instance.isShare = true
  if (options.iconGroup) {
    instance.multipleLine = Array.isArray(options.iconGroup[0])
  }
  ShowActionSheet(options)
}
</code></pre>

  * 将父组件传来的options的数据赋给instance,appendChild

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-javascript">let ShowActionSheet = function (options) {
  for (let prop in options) {
    if (options.hasOwnProperty(prop)) {
      instance[prop] = options[prop]
    }
  }
  document.body.appendChild(instance.$el)
  Vue.nextTick(() =&gt; {
    instance.visible = true
  })
}
</code></pre>

完整js文件

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-javascript">import Vue from 'vue'
import ActionSheetVue from './ActionSheet.vue'

const ActionSheetVueConstructor = Vue.extend(ActionSheetVue)

let instance
let initInstance = function () {
  instance = new ActionSheetVueConstructor({
    el: document.createElement('div')
  })
}

let ActionSheet = function (options) {
  if (!instance) {
    initInstance()
  }
  ShowActionSheet(options)
}

ActionSheet.share = function (options) {
  if (!instance) {
    initInstance()
  }
  instance.isShare = true
  if (options.iconGroup) {
    instance.multipleLine = Array.isArray(options.iconGroup[0])
  }
  ShowActionSheet(options)
}

let ShowActionSheet = function (options) {
  for (let prop in options) {
    if (options.hasOwnProperty(prop)) {
      instance[prop] = options[prop]
    }
  }
  document.body.appendChild(instance.$el)
  Vue.nextTick(() =&gt; {
    instance.visible = true
  })
}

export default ActionSheet

</code></pre>

  * 父组件中的调用

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-html">  &lt;div&gt;
    &lt;WingBlank&gt;
      &lt;h3&gt;Baisc&lt;/h3&gt;
      &lt;Button @click.native="showActionSheet"&gt;ShowActionSheet&lt;/Button&gt;
      &lt;WhiteSpace&gt;&lt;/WhiteSpace&gt;
      &lt;Button @click.native="shareActionSheet"&gt;ShareActionSheet&lt;/Button&gt;
      &lt;WhiteSpace&gt;&lt;/WhiteSpace&gt;
      &lt;Button @click.native="shareActionSheetMulpitleLine"&gt;ShareActionSheetMulpitleLine&lt;/Button&gt;
    &lt;/WingBlank&gt;
  &lt;/div&gt;
</code></pre>

<pre class="line-numbers prism-highlight" data-start="1"><code class="language-javascript">&lt;script&gt;
  import WingBlank from '@/components/WingBlank'
  import Button from '@/components/Button'
  import WhiteSpace from '@/components/WhiteSpace'
  import ActionSheet from '@/components/ActionSheet'
  import Toast from '@/components/Toast'

  export default{
    components: {
      WingBlank,
      ActionSheet,
      Button,
      WhiteSpace,
      Toast
    },
    data () {
      return {}
    },
    methods: {
      showActionSheet () {
        ActionSheet({
//          title: '提示',
          message: 'I am description, description, description',
          cancelButtonIndex: 4,
          destructiveButtonIndex: 3,
          btnGroup: [
            {text: 'Operation1'},
            {text: 'Operation2'},
            {text: 'Operation3'},
            {text: 'Delete'},
            {text: 'Cancel'}
          ],
          callback: (item) =&gt; new Promise((resolve) =&gt; {
            console.log(item)
            resolve()
          })
        })
      },
      shareActionSheet () {
        ActionSheet.share({
          message: 'I am description, description, description',
          cancelButtonText: '取消',
          iconGroup: [
            {title: '发送给朋友', icon: 'https://gw.alipayobjects.com/zos/rmsportal/OpHiXAcYzmPQHcdlLFrc.png'},
            {title: '新浪微博', icon: 'https://gw.alipayobjects.com/zos/rmsportal/wvEzCMiDZjthhAOcwTOu.png'},
            {title: '生活圈', icon: 'https://gw.alipayobjects.com/zos/rmsportal/cTTayShKtEIdQVEMuiWt.png'},
            {title: '微信好友', icon: 'https://gw.alipayobjects.com/zos/rmsportal/umnHwvEgSyQtXlZjNJTt.png'},
            {title: 'QQ', icon: 'https://gw.alipayobjects.com/zos/rmsportal/SxpunpETIwdxNjcJamwB.png'}
          ],
          callback: (inputValue) =&gt; new Promise((resolve) =&gt; {
            Toast({
              message: 'closed after 1000s',
              duration: 1000
            })
            setTimeout(() =&gt; {
              console.log(inputValue)
              resolve()
            }, 1000)
          })
        })
      },
      shareActionSheetMulpitleLine () {
        ActionSheet.share({
          message: 'I am description, description, description',
          cancelButtonText: '取消',
          iconGroup: [[
            {title: '发送给朋友', icon: 'https://gw.alipayobjects.com/zos/rmsportal/OpHiXAcYzmPQHcdlLFrc.png'},
            {title: '新浪微博', icon: 'https://gw.alipayobjects.com/zos/rmsportal/wvEzCMiDZjthhAOcwTOu.png'},
            {title: '生活圈', icon: 'https://gw.alipayobjects.com/zos/rmsportal/cTTayShKtEIdQVEMuiWt.png'},
            {title: '微信好友', icon: 'https://gw.alipayobjects.com/zos/rmsportal/umnHwvEgSyQtXlZjNJTt.png'},
            {title: 'QQ', icon: 'https://gw.alipayobjects.com/zos/rmsportal/SxpunpETIwdxNjcJamwB.png'}
          ], [
            {title: '微信好友', icon: 'https://gw.alipayobjects.com/zos/rmsportal/umnHwvEgSyQtXlZjNJTt.png'},
            {title: 'QQ', icon: 'https://gw.alipayobjects.com/zos/rmsportal/SxpunpETIwdxNjcJamwB.png'}
          ]],
          callback: (row, col) =&gt; new Promise((resolve) =&gt; {
            console.log(row, col)
            resolve()
          })
        })
      }
    }
  }
&lt;/script&gt;
</code></pre>