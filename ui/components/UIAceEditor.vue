<template>
    <!-- Component must be wrapped in a block so props such as className and style can be passed in from parent -->
    <div className="ui-ace-editor-wrapper">
        <v-ace-editor
            v-model:value="content"
            lang="javascript"
            theme="github"
            style="height: 100%;"
            @init="editorInit"
        />
    </div>
</template>

<script>
import ace from 'ace-builds'
import modeJavascriptUrl from 'ace-builds/src-noconflict/mode-javascript'
import themeGitHubUrl from 'ace-builds/src-noconflict/theme-github'
import { VAceEditor } from 'vue3-ace-editor'
import { mapState } from 'vuex'

ace.config.setModuleUrl('ace/mode/javascript', modeJavascriptUrl)
ace.config.setModuleUrl('ace/theme/github', themeGitHubUrl)

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
    setup (props) {
        console.info('UIAceEditor setup with:', props)
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
        // wrap our "example" property in a computed property to ensure it re-calculates when the property changes
        example () {
            // use the globally available "getProperty" function
            return this.getProperty('example')
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
        send (msg) {
            this.$socket.emit('widget-action', this.id, msg)
        },
        /*
            (optional) Custom onInput function to handle incoming messages from Node-RED
        */
        onInput (msg) {
            // load the latest message from the Node-RED datastore when this widget is loaded
            // storing it in our vuex store so that we have it saved as we navigate around
            this.$store.commit('data/bind', {
                widgetId: this.id,
                msg
            })
        },
        /*
            (optional) Custom onLoad function to handle the loading state of the widget
            msg   - the latest message from the Node-RED datastore
            state - The Node-RED config, including any overrides saved to the server-side statestore
        */
        onLoad (msg, state) {
            // loads the last msg received into this node from the Node-RED datastore
            // state is auto-stored into the widget props, but is available here if you want to do anything else
        },
        /*
            (optional) Custom onDynamicProperties function to handle dynamic properties
            msg - the latest message from the Node-RED datastore
        */
        onDynamicProperties (msg) {
            // handle any dynamic properties that are sent from Node-RED
            const updates = msg.ui_update
            if (!updates) {
                return
            }
            if (typeof updates.example !== 'undefined') {
                // use the globally available "setDynamicProperties" function to store any updates to this property
                this.setDynamicProperties({ example: updates.example })
            }
        },
        alert (text) {
            alert(text)
        },
        /*
            You can also emit custom events to Node-RED, which can be handled by a custom event handler
            See the ui-example.js file for how to subscribe to these.
        */
        test () {
            console.info('custom event handler:')
            this.$socket.emit('my-custom-event', this.id, {
                payload: 'Custom Event'
            })
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
