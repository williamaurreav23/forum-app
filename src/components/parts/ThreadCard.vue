<template>
    <div class="box" :class="{'grid' : !editMode}">
        <header v-if="!editMode" class="header">
            <router-link :to="`/_${board}/${thread.key}/${thread.slug}/`">
                <h2 class="title">{{ thread.title }}</h2>
            </router-link>
            <h3 class="subtitle">Posted by
                <router-link :to="`/~${thread.author}`">{{ thread.author }}</router-link>
                {{ thread.created | timeElapsed }}
            </h3>
        </header>

        <thread-controls v-if="!editMode" :edit-allowed="editAllowed" :drawer-open="drawerOpen" @toggle="drawerOpen = !drawerOpen" @delete="deleteThread()" @archive="archiveThread()" @edit="editThread()" />

        <thread-editor v-if="editMode" :board="board" :thread="thread" @close="updateThread()" />

        <sliding-drawer :hidden="editMode" :drawer-open="drawerOpen" :refresh="refresh" @refreshed="refresh = false" classes="content">
            <article class="can-edit can-break">{{ thread.content }}</article>
        </sliding-drawer>
    </div>
</template>

<script>
import { mapGetters } from 'vuex'
import './ThreadEditor'
import './ThreadControls'

export default Vue.component('thread-card', {
    props: {
        board: {
            type: String,
            required: true
        },
        thread: {
            type: Object,
            required: true
        }
    },
    data() {
        return {
            drawerOpen: false,
            editMode: false,
            refresh: false
        }
    },
    computed: {
        editAllowed: function() {
            return this.username === this.thread.author
        },
        handling: state => state === 'handling',
        ...mapGetters({
            username: 'user/GET_USERNAME',
            state: 'thread/GET_STATE'
        })
    },
    watch: {
        editMode: function(val) {
            if (!val) this.refresh = true
        },
        thread: function() {
            this.refresh = true
        }
    },
    methods: {
        deleteThread: async function() {
            if (confirm(`Are you sure you want to delete ${this.thread.title}?`)) {
                await this.$store.dispatch('thread/deleteThread', {
                    board: this.board,
                    key: this.thread.key,
                    slug: this.thread.slug
                })

                if (!this.handling) {
                    this.$emit('close')
                    await this.$store.dispatch('board/loadThreads')
                }
            }
        },
        editThread: function() {
            this.drawerOpen = true
            this.editMode = true
        },
        updateThread: function() {
            this.editMode = false
            this.drawerOpen = true
        },
        archiveThread: async function() {}
    }
})
</script>

<style lang="sass">
.box.grid
    display: grid
    grid-template-columns: 1fr
    grid-template-rows: auto auto 1fr
    grid-template-areas: "header" "controls" "content"
    grid-row-gap: 1em

.box.grid .header
    grid-area: header

.box.grid .controls
    grid-area: controls

.box.grid .content
    grid-area: content
</style>
