<template>
  <div fluid class="d-flex flex-column">
    <v-card
      class="overflow-y-auto"
      max-height="70vh"
      id="messages-container"
    >
      <div ref="scrollingContent">
        <v-card-text v-if="peerMessages && peerMessages.length > 0">
          <v-row v-for="(message, index) in peerMessages" :key="message.id">
            <v-col
              :class="message.sender === 'me' ? 'text-right' : 'text-left'"
              cols="12"
            >
              <div class="text-caption mb-0 mx-2">{{ getMessageTime(message) }}</div>
              <v-chip
                :class="[
                  message.sender === 'me' ? 'primary' : 'secondary',
                  'white--text',
                  'px-4',
                  'py-2',
                  index === peerMessages.length - 1 ? 'last-peer-message' : ''
                ]"
              >
                {{ message.text }}
              </v-chip>
            </v-col>
          </v-row>
        </v-card-text>
      </div>
    </v-card>
    <div
      ref="inputContainer"
      class="input-container"
    >
      <v-container fluid>
        <v-row>
          <v-col cols="10" class="ml-0 pl-0">
            <v-text-field
              v-model="inputMessage"
              @keydown.enter="sendMessage"
              outlined
              placeholder="Type your message here..."
              single-line
            ></v-text-field>
          </v-col>
          <v-col cols="2" class="d-flex justify-left align-center">
            <v-btn fab color="primary" @click="sendMessage" :disabled="isLocked">
              <v-icon>mdi-send</v-icon>
            </v-btn>
          </v-col>
        </v-row>
      </v-container>
    </div>
  </div>
</template>

<script lang="ts">
import { ref, onMounted, nextTick, computed } from 'vue'
import { mapState } from 'pinia'
import { useAuth } from '@/stores/auth'
import { useMessages } from '~/stores/messages'
import * as _timeago from 'timeago.js'
import { PeerMessage } from '~/stores/types'

export default defineComponent({
  setup() {
    const inputMessage = ref('')
    const inputContainerHeight = ref(0)
    const timeago = ref(_timeago)
    const scrollingContent = ref<HTMLElement | null>(null)

    const scrollToBottom = async () => {
      await nextTick()
      if (scrollingContent.value) {
        scrollingContent.value.scrollIntoView({ behavior: 'smooth', block: 'end' })
      }
    }

    onMounted(scrollToBottom)

    const authStore = useAuth()
    const isLocked = computed(() => authStore.isLocked)

    return { inputMessage, inputContainerHeight, timeago, scrollToBottom, scrollingContent, isLocked }
  },
  methods: {
    async sendMessage() {
      // @ts-ignore
      const text = this.inputMessage.trim()
      if (!text) return
      // @ts-ignore
      const { npub } = this.$route.params
      // @ts-ignore
      const lastMessage = this.peerMessages[this.peerMessages.length - 1]
      const id = lastMessage?.id || null
      // @ts-ignore
      await this.$mostro.submitDirectMessage(text, npub, id)
      // @ts-ignore
      this.inputMessage = ''
      this.scrollToBottom()
    },
    // @ts-ignore
    getMessageTime(message: PeerMessage) {
      // @ts-ignore
      return this.timeago.format(message.created_at * 1e3)
    },
  },
  watch: {
    peerMessages: {
      handler() {
        // @ts-ignore
        this.scrollToBottom();
      },
      // To scroll when the component is mounted, set immediate to true
      immediate: true
    }
  },
  computed: {
    ...mapState(useMessages, ['getPeerMessagesByNpub']),
    peerMessages() : PeerMessage[] {
      const route = useRoute()
      const { npub } = route.params
      // @ts-ignore
      return this.getPeerMessagesByNpub(npub)
    }
  }
})
</script>

<style scoped>
.input-container {
  width: 90%;
  position: absolute;
  bottom: 14px;
  z-index: 1;
}
</style>
