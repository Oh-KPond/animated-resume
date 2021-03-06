<template>
  <div id="app">
    <StyleEditor ref="styleEditor" :code="currentStyle"></StyleEditor>
    <ResumeEditor ref="resumeEditor" :markdown="currentMarkdown" :enableHtml="enableHtml"></ResumeEditor>
  </div>
</template>

<script>
  import StyleEditor from './components/StyleEditor'
  import ResumeEditor from './components/ResumeEditor'
  import './assets/reset.css'

  export default {
    name: 'app',
    components: {
      StyleEditor,
      ResumeEditor
    },
    data() {
      return {
        interval: 25,
        currentStyle: '',
        enableHtml: false,
        fullStyle: [
          `/*
* Inspired by http://strml.net/ & https://github.com/jirengu-inc/animating-resume
*
* Hello there! My name is Kate. It's a pleasure to e-meet you!
* Today I'd like to build my resume for you.
* If you ask me why, then I'd have to tell you...
* I think it's fun!
*/

/* First we'll need to add a transition effect to all the elements */
* {
  -webkit-transition: all .3s;
  transition: all .3s;
}
/* Then I don't know about you, but
* I find the white background a bit boring. Let's change that, shall we? */
html {
  color: #ececec;
  background: #4b2650;
}
/* Now we'll need a bit of padding to move the text from the edge of the screen
* And wouldn't a border be nice,
* with some margin to keep the text from the edge of the border.
*/
.styleEditor {
  padding: .5em;
  border: 1px solid;
  margin: .5em;
  overflow: auto;
  width: 50vw; height: 90vh;
}
/* I think it would be nice to have the code highlighted as well */
.token.selector { color: #ef41fe; }
.token.property { color: #f1a9a0; }
.token.punctuation { color: #dda0dd; }
.token.function { color: #fffacd; }

/* Those colors were pickout using colorsafe.co
* "Accessible text colors are generated with
* WCAG Guidelines recommend contrast ratio"
*
* How about a little perspective now? */
html {
  -webkit-perspective: 1000px;
          perspective: 1000px;
}
/* Here's some fun with that perspective.  */
.styleEditor {
  position: fixed; left: 10px; top: 0;
  -webkit-transition: none;
  transition: none;
  -webkit-transform: rotateY(10deg) translateZ(-100px) ;
          transform: rotateY(10deg) translateZ(-100px) ;
}

/* Now to make the space for my resume... */
.resumeEditor {
  position: fixed; right: 0; top: 0;
  padding: .5em;  margin: .5em;
  width: 48vw; height: 90vh;
  border: 3px solid #c5eff7;
  background: white; color: #222;
  overflow: auto;
}
/* And actually write it! */


`,
          `
/* This Markdown needs some style now.
 * We'll accomplish this with some open source tools like
 * marked(https://www.npmjs.com/package/marked).
 */
`
          ,
          `
/* Here comes the styling... */
.resumeEditor{
  padding: 2em;
}
.resumeEditor h2 {
  display: inline-block;
  border-bottom: 1px solid;
  margin: 1em 0 .5em;
}
.resumeEditor ul,.resumeEditor ol {
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
        fullMarkdown: `
Kate Pond
----

Genius, Unicorn, Goddess, Gatekeeper of Unlimited Knowledge, Leader

Skills
----
(Some of the things I love in no particular order)

* Hardware (especially soldering & LEDs)
* Event Planning
* Networking (& helping others to connect)
* Git/Github
* Slack
* Jira
* Pair Programming
* Heroku
* Ruby on Rails
* React.js
* Node.js
* HTML5 & CSS3
* Swift & Xcode
* Test Driven Development (TDD)
* Leadership
* Interpersonal Communication
* Education (giving talks & helping at workshops)

Experience
----
(Just a few things I've done)

* Currently Tech Ladies Seattle City Organizer
* Organized 30+ events & classes as a Girl Develop It Chapter Leader & Tech Ladies Phoenix City Organizer
* Supervisory Ranger for Mountains Conservation and Recreation Area & Tongass National Forest
* Given 100+ public talks on natural and cultural history, a few talks on Git, & a fews talks about Women in Tech

Connect
----

* [GitHub](https://github.com/Oh-KPond)
* [Medium](https://medium.com/@OhKPond)
* [Twitter @OhKPond](https://twitter.com/OhKPond)

> To build your own animated resume，Fork [This Project](https://github.com/Oh-KPond/animated-resume)，and make it your own！

`
      }
    },
    created() {
      this.makeResume()
    },

    methods: {
      makeResume: async function () {
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
      progressivelyShowStyle(n) {
        return new Promise((resolve, reject) => {
          let interval = this.interval
          let showStyle = (async function () {
            let style = this.fullStyle[n]
            if (!style) { return }
            // Calculate the total number of characters before the n styles
            let length = this.fullStyle.filter((_, index) => index <= n).map((item) => item.length).reduce((p, c) => p + c, 0)
            let prefixLength = length - style.length
            if (this.currentStyle.length < length) {
              let l = this.currentStyle.length - prefixLength
              let char = style.substring(l, l + 1) || ' '
              this.currentStyle += char
              if (style.substring(l - 1, l) === '\n' && this.$refs.styleEditor) {
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
              let lastChar = this.currentMarkdown[this.currentMarkdown.length - 1]
              let prevChar = this.currentMarkdown[this.currentMarkdown.length - 2]
              console.log(prevChar)
              if (prevChar === '\n' && this.$refs.resumeEditor) {
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

  * {
    -webkit-transition: all .3s;
    transition: all .3s;
  }
</style>
