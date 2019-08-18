<template>
  <div id="app">
    <StyleEditor ref="styleEditor" :code="currentStyle"></StyleEditor>
    <ResumeEditor ref="resumeEditor" :markdown="currentMarkdown" :enableHtml="enableHtml"></ResumeEditor>
  </div>
</template>

<script>
  import StyleEditor from './components/StyleEditor'
  import ResumeEditor from './components/ResumeEditor'

  export default {
    name: 'app',
    components: {
      StyleEditor,
      ResumeEditor
    },
    data() {
      return {
        interval: 30,
        currentStyle: '',
        enableHtml: false,
        fullStyle: [
          `/*
* 大家好，我是兵哥
* 看完这份简历需要点时间，不要心急
* 现在就开始做一份酷炫的简历吧！
*/

/* 首先给所有元素加上过渡效果 */
* {
  transition: all .3s;
}
/* 白色背景太单调了，加点背景色 */
html {
  color: rgb(222,222,222); 
  background: rgb(0,43,54);
}
/* 文字离边框太近了，加点间距 */
.styleEditor {
  padding: .5em;
  border: 1px solid;
  margin: .5em;
  overflow: auto;
  width: 46vw; height: 90vh;
  background-color:rgb(16,16,16);
}
/* 代码高亮更好看 */
.token.selector{ color: rgb(133,153,0); }
.token.property{ color: rgb(187,137,0); }
.token.punctuation{ color: yellow; }
.token.function{ color: rgb(42,161,152); }

/* 加点 3D 效果，更高大上 */
html{
  perspective: 1000px;
}
.styleEditor {
  position: fixed; left: 0; top: 0;
  -webkit-transition: none;
  transition: none;
  -webkit-transform: rotateY(10deg) translateZ(-100px) ;
          transform: rotateY(10deg) translateZ(-100px) ;
}

/* 接下来给自己准备一个html编辑器 */
.resumeEditor{
  position: fixed; 
  right: 0; 
  top: 0;
  padding: .5em;  
  margin: .5em;
  width: 48vw; 
  height: 90vh;
  border: 1px solid;
  background: #f0f4f0; 
  color: #333;
  overflow: auto;
}
/* 接下来就开始写简历内容了 */

`,
          `
/* 给简历加上一点样式，更好看
*需要先把markdown格式的简历转成HTML格式的
 */
`
          ,
          `
/* 再对 HTML 加点样式 */
.resumeEditor{
  padding: 2em;
}
.resumeEditor h2{
  display: inline-block;
  border-bottom: 1px solid;
  margin: 1em 0 .5em;
  font-size: 1em;
}
.resumeEditor ul,.resumeEditor ol{
  list-style: none;
}
.resumeEditor ul> li::before{
  content: '•';
  margin-right: .5em;
}
.resumeEditor ol {
  counter-reset: section;
}
.resumeEditor ol li::before {
  counter-increment: section;
  content: counters(section, ".") " ";
  margin-right: .5em;
}
.resumeEditor blockquote {
  margin: 1em;
  padding: .5em;
  background: #ddd;
}
`],
        currentMarkdown: '',
        fullMarkdown: `周兵
----

高级前端工程师，2013年毕业于湖北民族大学计算机系，一直在从事web前端项目的开发;


技能
----

* 前端技术栈(javascript/html5/vue等)开发
* python 开发
* java、php、android、node都有一定的熟悉

工作经历
----

1. [森果](https://senguo.cc)
2. 北京辰安科技武汉分公司应用研发部
3. 北京高视远景科技有限公司研发部

链接
----

* [GitHub](https://github.com/playboygit)
* [我的博客](https://blog.csdn.net/playboyanta123)

> 如果你喜欢这个效果，Fork [这个项目](https://github.com/jirengu-inc/animating-resume)！

`
      }
    },
    created: async function() {
      await this.makeResume()
      this.ajustLink()
    },

    methods: {
      makeResume: async function () {
        //样式输出的第一段
        await this.progressivelyShowStyle(0)
        await this.progressivelyShowResume()
        await this.progressivelyShowStyle(1)
        await this.showHtml()
        await this.progressivelyShowStyle(2)
      },
      showHtml: function () {
        return new Promise((resolve, reject) => {
          this.enableHtml = true
          resolve()
        })
      },
      ajustLink: function(){
        console.log("finished resume")
        let alinks = document.querySelectorAll('a')
        for(let i=0; i<alinks.length; i++){
          alinks[i].setAttribute("target","_blank")
        }
      },
      progressivelyShowStyle(n) {
        return new Promise((resolve, reject) => {
          let interval = this.interval
          let showStyle = (async function () {
            let style = this.fullStyle[n]
            if (!style) { return }
            // 计算前 n 个 style 的字符总数
            let length = this.fullStyle
            .filter((_, index) => index <= n) //过滤当前要遍历的样式字符串
            .map((item) => item.length)  //分别计算字符串的长度
            .reduce((p, c) => p + c, 0)  //计算数组字符串的总长度
            let prefixLength = length - style.length
            if (this.currentStyle.length < length) {
              let l = this.currentStyle.length - prefixLength
              let char = style.substring(l, l + 1) || ' '
              this.currentStyle += char
              if (style.substring(l - 1, l) === '\n' && this.$refs.styleEditor) {
                //vue自带方法，dom更新之后执行
                this.$nextTick(() => {
                  this.$refs.styleEditor.goBottom()
                })
              }
              setTimeout(showStyle, interval)
            } else {
              resolve()
            }
          }).bind(this)
          showStyle()
        })
      },
      progressivelyShowResume() {
        return new Promise((resolve, reject) => {
          let length = this.fullMarkdown.length
          let interval = this.interval
          let showResume = () => {
            if (this.currentMarkdown.length < length) {
              this.currentMarkdown = this.fullMarkdown.substring(0, this.currentMarkdown.length + 1)
              let currentMarkdownLength = this.currentMarkdown.length
              let curChar = ''
              if(currentMarkdownLength>1){
                curChar = this.currentMarkdown[currentMarkdownLength - 2]
              }else{
                curChar = this.currentMarkdown[currentMarkdownLength - 1]
              }
              if (curChar === '\n' && this.$refs.resumeEditor) {
                this.$nextTick(() => this.$refs.resumeEditor.goBottom())
              }
              setTimeout(showResume, interval)
            } else {
              resolve()
            }
          }
          showResume()
        })
      }
    }
  }


</script>

<style scoped>
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
  }

  html {
    min-height: 100vh;
  }
  *{
    box-sizing: border-box;
  }
</style>
