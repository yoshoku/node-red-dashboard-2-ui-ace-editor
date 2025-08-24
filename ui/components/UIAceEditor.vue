<template>
    <!-- Component must be wrapped in a block so props such as className and style can be passed in from parent -->
    <div className="ui-ace-editor-wrapper">
        <v-ace-editor
            v-model:value="content"
            :lang="mode"
            :theme="theme"
            style="height: 100%;"
            @blur="onBlur"
            @keyup.enter="onEnter"
        />
    </div>
</template>

<script>
import ace from 'ace-builds'
/* eslint-disable import/no-unresolved */
import modeAsciiDocUrl from 'ace-builds/src-noconflict/mode-asciidoc?url'
import modeCCppUrl from 'ace-builds/src-noconflict/mode-c_cpp?url'
import modeCSSUrl from 'ace-builds/src-noconflict/mode-css?url'
import modeGoUrl from 'ace-builds/src-noconflict/mode-golang?url'
import modeHTMLUrl from 'ace-builds/src-noconflict/mode-html?url'
import modeJavaUrl from 'ace-builds/src-noconflict/mode-java?url'
import modeJavaScriptUrl from 'ace-builds/src-noconflict/mode-javascript?url'
import modeJSONUrl from 'ace-builds/src-noconflict/mode-json?url'
import modeMarkdownUrl from 'ace-builds/src-noconflict/mode-markdown?url'
import modePHPUrl from 'ace-builds/src-noconflict/mode-php?url'
import modePythonUrl from 'ace-builds/src-noconflict/mode-python?url'
import modeRubyUrl from 'ace-builds/src-noconflict/mode-ruby?url'
import modeRustUrl from 'ace-builds/src-noconflict/mode-rust?url'
import modeShellUrl from 'ace-builds/src-noconflict/mode-sh?url'
import modeSQLUrl from 'ace-builds/src-noconflict/mode-sql?url'
import modeYAMLUrl from 'ace-builds/src-noconflict/mode-yaml?url'
import themeChromeUrl from 'ace-builds/src-noconflict/theme-chrome?url'
import themeCloudsUrl from 'ace-builds/src-noconflict/theme-clouds?url'
import themeGitHubUrl from 'ace-builds/src-noconflict/theme-github?url'
import themeGitHubDarkUrl from 'ace-builds/src-noconflict/theme-github_dark?url'
import themeMonokaiUrl from 'ace-builds/src-noconflict/theme-monokai?url'
import themeSolarizedDarkUrl from 'ace-builds/src-noconflict/theme-solarized_dark?url'
import themeSolarizedLightUrl from 'ace-builds/src-noconflict/theme-solarized_light?url'
/* eslint-enable import/no-unresolved */
import { VAceEditor } from 'vue3-ace-editor'
import { mapState } from 'vuex'

ace.config.setModuleUrl('ace/mode/css', modeCSSUrl)
ace.config.setModuleUrl('ace/mode/javascript', modeJavaScriptUrl)
ace.config.setModuleUrl('ace/mode/python', modePythonUrl)
ace.config.setModuleUrl('ace/mode/ruby', modeRubyUrl)
ace.config.setModuleUrl('ace/mode/php', modePHPUrl)
ace.config.setModuleUrl('ace/mode/java', modeJavaUrl)
ace.config.setModuleUrl('ace/mode/c_cpp', modeCCppUrl)
ace.config.setModuleUrl('ace/mode/golang', modeGoUrl)
ace.config.setModuleUrl('ace/mode/rust', modeRustUrl)
ace.config.setModuleUrl('ace/mode/markdown', modeMarkdownUrl)
ace.config.setModuleUrl('ace/mode/asciidoc', modeAsciiDocUrl)
ace.config.setModuleUrl('ace/mode/json', modeJSONUrl)
ace.config.setModuleUrl('ace/mode/yaml', modeYAMLUrl)
ace.config.setModuleUrl('ace/mode/sql', modeSQLUrl)
ace.config.setModuleUrl('ace/mode/html', modeHTMLUrl)
ace.config.setModuleUrl('ace/mode/sh', modeShellUrl)

ace.config.setModuleUrl('ace/theme/chrome', themeChromeUrl)
ace.config.setModuleUrl('ace/theme/clouds', themeCloudsUrl)
ace.config.setModuleUrl('ace/theme/monokai', themeMonokaiUrl)
ace.config.setModuleUrl('ace/theme/github', themeGitHubUrl)
ace.config.setModuleUrl('ace/theme/github_dark', themeGitHubDarkUrl)
ace.config.setModuleUrl('ace/theme/solarized_light', themeSolarizedLightUrl)
ace.config.setModuleUrl('ace/theme/solarized_dark', themeSolarizedDarkUrl)

export default {
    name: 'UIAceEditor',
    components: {
        VAceEditor
    },
    inject: ['$socket', '$dataTracker'],
    props: {
        /* do not remove entries from this - Dashboard's Layout Manager's will pass this data to your component */
        id: { type: String, required: true },
        props: { type: Object, default: () => ({}) },
        state: { type: Object, default: () => ({ enabled: false, visible: false }) }
    },
    data () {
        return {
            input: {
                title: 'some text here will base turned into title case.'
            },
            vuetifyStyles: [
                { label: 'Responsive Displays', url: 'https://vuetifyjs.com/en/styles/display/#display' },
                { label: 'Flex', url: 'https://vuetifyjs.com/en/styles/flex/' },
                { label: 'Spacing', url: 'https://vuetifyjs.com/en/styles/spacing/#how-it-works' },
                { label: 'Text & Typography', url: 'https://vuetifyjs.com/en/styles/text-and-typography/#typography' }
            ]
        }
    },
    computed: {
        ...mapState('data', ['messages']),
        content: {
            get () {
                return this.messages[this.id]?.payload
            },
            set (val) {
                if (!this.messages[this.id]) {
                    this.messages[this.id] = {}
                }
                this.messages[this.id].payload = val
            }
        },
        mode: function () {
            return this.getProperty('mode') || 'javascript'
        },
        theme: function () {
            return this.getProperty('theme') || 'chrome'
        }
    },
    created () {
        // setup our event handlers, and informs Node-RED that this widget has loaded
        this.$dataTracker(this.id, this.onInput, this.onLoad, this.onDynamicProperties)
    },
    methods: {
        /*
            widget-action just sends a msg to Node-RED, it does not store the msg state server-side
            alternatively, you can use widget-change, which will also store the msg in the Node's datastore
        */
        send () {
            this.$socket.emit('widget-change', this.id, this.content)
        },
        onBlur () {
            if (this.props.sendOnBlur) {
                this.send()
            }
        },
        onEnter () {
            if (this.props.sendOnEnter) {
                this.send()
            }
        }
    }
}
</script>

<style scoped>
.ui-ace-editor-wrapper {
    padding: 0;
    margin: 0;
    border: 1px solid #d1d1d1;
    border-radius: 4px;
}
</style>
