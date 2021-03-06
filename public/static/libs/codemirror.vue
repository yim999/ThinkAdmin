<template>
<textarea></textarea>
</template>

<script>
const theme = 'base16-dark'
const mode = 'javascript'
const CodeMirror = require('codemirror/lib/codemirror.js')
require('codemirror/lib/codemirror.css')

require.ensure([], function (require) {
  require(`codemirror/theme/${theme}.css`)
  require('codemirror/addon/display/fullscreen.css')
  require('codemirror/addon/display/fullscreen.js')
})

export default {
  data: function () {
    return {
      content: '',
      editor: null
    }
  },

  props: {
    code: String,
    value: String,
    options: {
      type: Object,
      default: function () {
        return {
          mode: mode,
          line: true,
          lineNumbers: true,
          lineWrapping: true,
          lint: true,
          gutters: ['CodeMirror-linenumbers', 'CodeMirror-lint-markers']
        }
      }
    },
    lintOptions: {
      type: Object,
      default: function () {
        return { sub: true }
      }
    },
    replaceRange: {
      type: Object,
      default: function () {
        return null
      }
    }
  },

  created () {
    // Require language mode config & basic addons.
    require(`codemirror/mode/javascript/javascript.js`)
     require(`codemirror/mode/css/css.js`)
    require('codemirror/addon/lint/lint.js')
    require('codemirror/addon/lint/lint.css')
    require('codemirror/addon/lint/javascript-lint.js')
  },

  mounted () {
    const options = { ...this.options, lint: this.lintOptions, theme: theme }
    this.editor = CodeMirror.fromTextArea(this.$el, options)
    this.editor.setValue(this.code || this.value || this.content)

    this.editor.on('change', (cm) => {
      this.content = cm.getValue()
      if (!!this.$emit) {
        this.$emit('changed', this.content)
        this.$emit('input', this.content)
      }
    })

    if (!window.JSHINT) window.JSHINT = require('jshint').JSHINT
  },

  beforeDestroy () { window.JSHINT = null },

  watch: {
    code (newVal, oldVal) {
      const editorValue = this.editor.getValue()
      if (newVal !== editorValue) {
        let scrollInfo = this.editor.getScrollInfo()
        this.editor.setValue(newVal)
        this.content = newVal
        this.editor.scrollTo(scrollInfo.left, scrollInfo.top)
      }
    },

    value (newVal, oldVal) {
      const editorValue = this.editor.getValue()
      if (newVal !== editorValue) {
        let scrollInfo = this.editor.getScrollInfo()
        this.editor.setValue(newVal)
        this.content = newVal
        this.editor.scrollTo(scrollInfo.left, scrollInfo.top)
      }
    },

    /**
     * Replace string from & to a certain range. If `to` is equal to `from` it will just insert a string at that position.
     * @see https://codemirror.net/doc/manual.html#replaceRange
     */
    replaceRange: {
      handler (newVal, oldVal) {
        if (newVal && this.editor) {
          const cursor = this.editor.getCursor()
          const cursorLineAndCh = {
            line: cursor.line,
            ch: cursor.ch
          }
          const insert = {
            ...newVal,
            from: newVal.from || cursorLineAndCh,
            to: newVal.from || cursorLineAndCh
          }

          this.editor.replaceRange(
            insert.replacement,
            insert.from,
            insert.to
          )
        }
      },
      deep: true
    }
  }
}
</script>

<style scoped>
.CodeMirror-code {
  line-height: 1.6em;
  font-family: Menlo, Monaco, Consolas, "Courier New", monospace;
}
</style>
