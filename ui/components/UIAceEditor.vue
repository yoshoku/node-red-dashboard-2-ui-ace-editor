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

<script setup>
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
import { computed, inject, onMounted, ref, watch } from 'vue'
import { VAceEditor } from 'vue3-ace-editor'
import { useStore } from 'vuex'

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

const props = defineProps({
    id: { type: String, required: true },
    props: { type: Object, default: () => ({}) },
    state: { type: Object, default: () => ({ enabled: false, visible: false }) }
})

const $socket = inject('$socket')
const $dataTracker = inject('$dataTracker')
const store = useStore()

const messages = computed(() => store.state.data.messages)
// Sometimes `props.id` is not yet registered in `store.messages` when onMounted is triggered.
// This watcher monitors the `messages` state until `props.id`  is present.
const unwatch = watch(messages, () => {
    $dataTracker(props.id)
    if (messages.value[props.id]) {
        unwatch()
    }
}, { deep: true, immediate: true })

const content = ref(messages.value[props.id] ? messages.value[props.id].payload : '')
// Watch the input payload and apply it to the editor.
watch(messages, (newMessages) => {
    if (messages.value[props.id]) {
        if (newMessages[props.id].payload !== content.value) {
            content.value = newMessages[props.id].payload.toString() || ''
        }
    }
}, { deep: true })

const mode = computed(() => {
    return props.props.mode || 'javascript'
})

const theme = computed(() => {
    return props.props.theme || 'chrome'
})

const send = () => {
    $socket.emit('widget-change', props.id, content.value)
}

const onBlur = () => {
    if (props.props.sendOnBlur) {
        send()
    }
}

const onEnter = () => {
    if (props.props.sendOnEnter) {
        send()
    }
}

onMounted(() => {
    $dataTracker(props.id)
})
</script>

<style scoped>
.ui-ace-editor-wrapper {
    padding: 0;
    margin: 0;
    border: 1px solid #d1d1d1;
    border-radius: 4px;
}
</style>
