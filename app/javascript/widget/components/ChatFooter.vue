<template>
  <div>
    <footer v-if="!hideReplyBox" class="footer">
      <ChatInputWrap
        :on-send-message="handleSendMessage"
        :on-send-attachment="handleSendAttachment"
      />
    </footer>
    <div v-else>
      <custom-button
        class="font-medium"
        block
        :bg-color="widgetColor"
        :text-color="textColor"
        @click="startNewConversation"
      >
        {{ $t('START_NEW_CONVERSATION') }}
      </custom-button>
      <custom-button
        v-if="showEmailTranscriptButton"
        type="clear"
        class="font-normal"
        @click="sendTranscript"
      >
        {{ $t('EMAIL_TRANSCRIPT.BUTTON_TEXT') }}
      </custom-button>
    </div>
  </div>
</template>

<script>
import { mapActions, mapGetters } from 'vuex';
import { getContrastingTextColor } from '@chatwoot/utils';
import CustomButton from 'shared/components/Button';
import ChatInputWrap from 'widget/components/ChatInputWrap.vue';
import { BUS_EVENTS } from 'shared/constants/busEvents';
import { sendEmailTranscript } from 'widget/api/conversation';

export default {
  components: {
    ChatInputWrap,
    CustomButton,
  },
  props: {
    msg: {
      type: String,
      default: '',
    },
  },
  computed: {
    ...mapGetters({
      conversationAttributes: 'conversationAttributes/getConversationParams',
      widgetColor: 'appConfig/getWidgetColor',
      getConversationSize: 'conversation/getConversationSize',
      currentUser: 'contacts/getCurrentUser',
    }),
    textColor() {
      return getContrastingTextColor(this.widgetColor);
    },
    hideReplyBox() {
      const { csatSurveyEnabled } = window.chatwootWebChannel;
      const { status } = this.conversationAttributes;
      return csatSurveyEnabled && status === 'resolved';
    },
    showEmailTranscriptButton() {
      return this.currentUser && this.currentUser.email;
    },
  },
  methods: {
    ...mapActions('conversation', [
      'sendMessage',
      'sendAttachment',
      'clearConversations',
    ]),
    ...mapActions('conversationAttributes', [
      'getAttributes',
      'clearConversationAttributes',
    ]),
    async handleSendMessage(content) {
      const conversationSize = this.getConversationSize;
      await this.sendMessage({
        content,
      });
      // Update conversation attributes on new conversation
      if (conversationSize === 0) {
        this.getAttributes();
      }
    },
    handleSendAttachment(attachment) {
      this.sendAttachment({ attachment });
    },
    startNewConversation() {
      this.clearConversations();
      this.clearConversationAttributes();
      window.bus.$emit(BUS_EVENTS.START_NEW_CONVERSATION);
    },
    async sendTranscript() {
      const { email } = this.currentUser;
      if (email) {
        try {
          await sendEmailTranscript({
            email,
          });
          window.bus.$emit(BUS_EVENTS.SHOW_ALERT, {
            message: this.$t('EMAIL_TRANSCRIPT.SEND_EMAIL_SUCCESS'),
            type: 'success',
          });
        } catch (error) {
          window.bus.$emit(BUS_EVENTS.SHOW_ALERT, {
            message: this.$t('EMAIL_TRANSCRIPT.SEND_EMAIL_ERROR'),
          });
        }
      }
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
@import '~widget/assets/scss/variables.scss';
@import '~widget/assets/scss/mixins.scss';

.footer {
  background: $color-white;
  box-sizing: border-box;
  padding: $space-small $space-slab;
  margin-top: 1rem;
  width: 100%;
  border-radius: 7px;
  @include shadow-big;
}

.branding {
  align-items: center;
  color: $color-body;
  display: flex;
  font-size: $font-size-default;
  justify-content: center;
  padding: $space-one;
  text-align: center;
  text-decoration: none;

  img {
    margin-right: $space-small;
    max-width: $space-two;
  }
}
</style>
