<template>
  <div class="idea-part__header__buttons-panel">
    <v-btn
      v-if="editable"
      x-small
      text
      icon
      color="gray"
      class="privacyButton"
      @click="toggleIdeaPrivacy"
    >
      <v-icon v-if="isPrivate">mdi-lock</v-icon>
      <v-icon v-else>mdi-lock-open-variant-outline</v-icon>
    </v-btn>
    <v-btn x-small icon text class="shareButton">
      <v-icon @click="showShareIdeaDialog">share</v-icon>
    </v-btn>
    <v-menu v-if="editable" offset-y left transition="slide-y-transition">
      <template v-slot:activator="{ on }">
        <v-btn x-small icon v-on="on">
          <v-icon class="moreActionsButton">fas fa-ellipsis-v</v-icon>
        </v-btn>
      </template>
      <v-list>
        <v-list-item @click="enableEditMode">
          <v-list-item-title>
            <v-icon small class="editButton">mdi-pencil</v-icon>
            <span>Edit Idea</span>
          </v-list-item-title>
        </v-list-item>
        <v-list-item @click="deleteIdea">
          <v-list-item-title>
            <v-icon small class="deleteButton">mdi-trash-can-outline</v-icon>
            <span>Delete Idea</span>
          </v-list-item-title>
        </v-list-item>
      </v-list>
    </v-menu>
    <act-on-idea
      v-if="!editable"
      :idea="idea"
      action="like"
      @liked-state-changed="onIdeaLikeStateChanged"
    ></act-on-idea>
    <act-on-idea
      v-if="!editable"
      :idea="idea"
      action="save"
      @saved-state-changed="onIdeaSaveStateChanged"
    ></act-on-idea>

    <share-idea-by-email-dialog
      v-model="showEmailShareDialog"
      :idea-id="idea.ideaId"
      :idea-owner-id="idea.userId"
      @success="onSharedIdeaOverEmail"
      @error="onSharedIdeaOverEmailError"
      @onCopyShareLink="onCopyShareLink"
    ></share-idea-by-email-dialog>

    <default-dialog
      v-model="showMakeIdeaPrivate"
      header="Want To Make Your Idea Private?"
      button-ok-text="Make Private"
      :image-path="require('~/assets/images/dialogs/undraw_security_o890.svg')"
      @ok="makeIdeaPrivate"
    >
      <p>That's ok!</p>
      <p>
        Some things aren't ready to be shared. Make this idea private with just
        one click.
      </p>
    </default-dialog>

    <default-dialog
      v-model="showMakeIdeaPublic"
      header="Share Your Idea With The World!"
      button-ok-text="Make Public"
      :image-path="
        require('~/assets/images/dialogs/undraw_spread_love_r9jb.svg')
      "
      @ok="makeIdeaPublic"
    >
      <p>
        Awesome!
      </p>
      <p>Ideas want to be shared! Make this idea public with just one click.</p>
    </default-dialog>
  </div>
</template>

<script>
import { mapMutations } from 'vuex'
import { graphqlOperation } from '@aws-amplify/api'
import ShareIdeaByEmailDialog from '@/components/dialogs/ShareIdeaByEmail'
import ActOnIdea from '@/components/ideaDetail/ActOnIdea'
import DefaultDialog from '@/components/dialogs/DefaultDialog'
import makeIdeaPrivate from '~/graphql/mutations/makeIdeaPrivate'
import makeIdeaPublic from '~/graphql/mutations/makeIdeaPublic'

export default {
  name: 'MenuPanel',
  components: {
    ActOnIdea,
    ShareIdeaByEmailDialog,
    DefaultDialog
  },
  props: {
    editable: {
      type: Boolean,
      default: false
    },
    idea: {
      type: Object,
      required: true
    }
  },
  data() {
    return {
      showEmailShareDialog: false,
      showMakeIdeaPrivate: false,
      showMakeIdeaPublic: false
    }
  },
  computed: {
    isPrivate() {
      return this.idea.visibility === 'PRIVATE'
    }
  },
  methods: {
    ...mapMutations({
      showProgressBar: 'layoutState/showProgressBar',
      hideProgressBar: 'layoutState/hideProgressBar'
    }),

    deleteIdea() {
      this.$emit('on-delete-idea')
    },

    onIdeaSaveStateChanged(val) {
      this.$emit('saved-state-changed', val)
    },

    onIdeaLikeStateChanged(val) {
      this.$emit('liked-state-changed', val)
    },

    async makeIdeaPrivate() {
      this.showProgressBar()
      try {
        const ideaId = this.idea.ideaId
        const result = await this.$amplifyApi.graphql(
          graphqlOperation(makeIdeaPrivate, { ideaId })
        )
        if (result.data.makeIdeaPrivate.ok) {
          this.$emit('on-idea-visibility-changed', { isPrivate: true })
        } else {
          this.$emit('on-idea-visibility-change-error', { isPrivate: true })
        }
      } catch (err) {
        this.$emit('on-idea-visibility-change-error', { isPrivate: true })
      }
      this.hideProgressBar()
      this.showMakeIdeaPrivate = false
    },

    async makeIdeaPublic() {
      this.showProgressBar()
      try {
        const ideaId = this.idea.ideaId
        const result = await this.$amplifyApi.graphql(
          graphqlOperation(makeIdeaPublic, { ideaId })
        )
        if (result.data.makeIdeaPublic.ok) {
          this.$emit('on-idea-visibility-changed', { isPrivate: false })
        } else {
          this.$emit('on-idea-visibility-change-error', { isPrivate: false })
        }
      } catch (err) {
        this.$emit('on-idea-visibility-change-error', { isPrivate: false })
      }
      this.hideProgressBar()
      this.showMakeIdeaPublic = false
    },

    toggleIdeaPrivacy() {
      if (this.isPrivate) {
        this.showMakeIdeaPublic = true
      } else {
        this.showMakeIdeaPrivate = true
      }
    },

    showShareIdeaDialog() {
      if (navigator.share) {
        navigator.share({
          title: this.idea.title,
          url: window.location.href
        })
      } else {
        this.showEmailShareDialog = true
      }
    },

    enableEditMode() {
      this.$emit('enable-edit-mode')
    },

    onSharedIdeaOverEmail() {
      this.$emit('on-notification', {
        type: 'success',
        message: 'Idea shared!'
      })
      this.$emit('on-idea-shared')
    },

    onSharedIdeaOverEmailError() {
      this.$emit('on-notification', {
        type: 'error',
        message: "Can't share Idea"
      })
    },

    onCopyShareLink() {
      this.$emit('on-notification', { type: 'success', message: 'Link copied' })
    }
  }
}
</script>

<style scoped lang="scss">
.idea-part__header__buttons-panel {
  padding-top: 4px; /* this is just to offset the buttons to match the baseline of the title, because the title font is a different height than the button icon height */
  .privacyButton {
    /* color: #1867c0 !important; */
    color: #5cbbf6 !important;
  }
  .shareButton {
    color: #2cbe4e !important;
  }
  .moreActionsButton {
    color: silver !important;
  }
}
</style>
