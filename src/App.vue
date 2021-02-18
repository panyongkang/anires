<template>
  <div id="app">
    <StyleEditor ref="styleEditor" :code="currentStyle"></StyleEditor>
    <ResumeEditor ref="resumeEditor" :markdown="currentMarkdown" :enableHtml="enableHtml"></ResumeEditor>
  </div>
</template>

<script>
import StyleEditor from "./components/StyleEditor";
import ResumeEditor from "./components/ResumeEditor";
import "./assets/reset.css";

export default {
  name: "app",
  components: {
    StyleEditor,
    ResumeEditor
  },
  data() {
    return {
      // 动画延时
      interval: 3,
      currentStyle: "",
      enableHtml: false,
      fullStyle: [
        `/*
* Inspired by http://strml.net/
* 源码地址 https://gitee.com/zhousiwei/anires
* 大家好，我是向阳榆木。
* 我来写一份简历！
*/

/* 给所有元素加上过渡效果 */
* {
  transition: all .1s;
}
/* 设置背景颜色 */
html {
  color: rgb(222,222,222); background: rgb(0,43,54);
}

/* 设置边框 */
.styleEditor {
  padding: .5em;
  box-shadow: -8px -8px 10px #0300ff, -8px 8px 10px #0300ff;
  margin: .5em;
  overflow: auto;
  width: 45vw; 
  height: 90vh;
  background: rgb(42 12 206);
}
/* 代码高亮 */
.token.selector{ color: rgb(133,153,0); }
.token.property{ color: rgb(187,137,0); }
.token.punctuation{ color: yellow; }
.token.function{ color: rgb(42,161,152); }

/* 加3D效果 */
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

/* 准备一个编辑器 */
.resumeEditor{
  position: fixed; 
  right: 0; top: 0;
  margin: .5em;
  width: 48vw; 
  height: 90vh;
  box-shadow: 0px 0px 10px 5px #aaa;
  background: white; 
  color: #222;
  overflow: auto;
}
/* 开始写简历 */
`,
        `
/*将Markdown格式翻译成HTML
 *再对HTML加点样式
*/
.resumeEditor{
  padding: 2em;
}
.resumeEditor h2{
  display: inline-block;
  border-bottom: 1px solid;
  margin: 1em 0 .5em;
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
`
      ],
      currentMarkdown: "",
      fullMarkdown: `
潘永康
----
24岁，Java软件工程师，两年Java EE 从业经验，目前在贵阳工作。
有良好的文档编写和代码书写规范，能独立解决问题、细节控
<br/>

- [个人博客](https://www.xyym.cf/)

技能
----
* 后端开发
* 公众号/小程序开发
* 资源整合
* 开源爱好者
* 博客系统

技术及语言
----
  - **Java**: SpringBoot、SpringCloud、SpringMVC、MyBatis
  - **前端**: VueJs、Bootstrap、NodeJs、jQuery UI、Element UI
  - **数据库**: MySQL、DB2、Oracle、Redis、EsgynDB
  - **web 服务器**: Nginx、Tomcat、Apache、Jetty
  - **OS**: Linux、Windows
  - **Others**: Git、Svn、Eclipse、PrcessOn、IDEA

工作经历
----
1. [深圳市长亮科技股份有限公司](http://www.sunline.cn/)
2. [北京科蓝软件系统股份有限公司](http://www.csii.com.cn/)

开源项目
----
1. [百脑科技商城](https://github.com/panyongkang/gitRepository/tree/master/EStore-master)
2. [电子购物商城项目](https://github.com/panyongkang/gitRepository/tree/master/e-shop)
3. [基于Vue图书管理小项目](https://github.com/panyongkang/vueProjects)
4. [一个会动的简历](https://gitee.com/zhousiwei/anires)
5. [向阳榆木的博客网站](https://github.com/panyongkang/panyongkang.github.io)

链接
----
* [向阳榆木博客](https://www.xyym.cf/)
* [GitHub](https://github.com/panyongkang)
* [主页导航](http://dwz.date/c6qh)
* [综合导航](https://www.xyym.cf/mypages/xkzhdh/Integrated.html)
* [软件分享](https://www.xyym.cf/mypages/software/index.html)

联系我
----
* 联系QQ：**2549315545** | 微信：**18385763177**
* 主要涉及技术：**Java后端开发**、**资源整合**、**公众号开发**、**博客订制**、**Live 2D**

> 如果你喜欢这个效果，Fork [我的项目](https://gitee.com/zhousiwei/anires)，打造你自己的简历！
`
    };
  },
  created() {
    this.makeResume();
  },

  methods: {
    makeResume: async function() {
      await this.progressivelyShowStyle(0);
      await this.progressivelyShowResume();
      await this.progressivelyShowStyle(1);
      await this.showHtml();
      await this.progressivelyShowStyle(2);
    },
    showHtml() {
      return new Promise((resolve, reject) => {
        this.enableHtml = true;
        resolve();
      });
    },
    progressivelyShowStyle(n) {
      return new Promise((resolve, reject) => {
        let interval = this.interval;
        let showStyle = async function() {
          let style = this.fullStyle[n];
          if (!style) {
            return;
          }
          // 计算前 n 个 style 的字符总数
          let length = this.fullStyle
            .filter((_, index) => index <= n)
            .map(item => item.length)
            .reduce((p, c) => p + c, 0);
          let prefixLength = length - style.length;
          if (this.currentStyle.length < length) {
            let l = this.currentStyle.length - prefixLength;
            let char = style.substring(l, l + 1) || " ";
            this.currentStyle += char;
            if (style.substring(l - 1, l) === "\n" && this.$refs.styleEditor) {
              this.$nextTick(() => {
                this.$refs.styleEditor.goBottom();
              });
            }
            setTimeout(showStyle, interval);
          } else {
            resolve();
          }
        }.bind(this);
        showStyle();
      });
    },
    progressivelyShowResume() {
      return new Promise((resolve, reject) => {
        let length = this.fullMarkdown.length;
        let interval = this.interval;
        let showResume = () => {
          if (this.currentMarkdown.length < length) {
            this.currentMarkdown = this.fullMarkdown.substring(
              0,
              this.currentMarkdown.length + 1
            );
            let lastChar = this.currentMarkdown[
              this.currentMarkdown.length - 1
            ];
            let prevChar = this.currentMarkdown[
              this.currentMarkdown.length - 2
            ];
            if (prevChar === "\n" && this.$refs.resumeEditor) {
              this.$nextTick(() => this.$refs.resumeEditor.goBottom());
            }
            setTimeout(showResume, interval);
          } else {
            resolve();
          }
        };
        showResume();
      });
    }
  }
};
</script>

<style scoped>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

html {
  min-height: 100vh;
}

* {
  box-sizing: border-box;
}
</style>
