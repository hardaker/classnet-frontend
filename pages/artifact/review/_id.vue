<template>
  <div>
    <a @click="$router.go(-1)">Back</a>
    <ArtifactCommentView
      :artifact="artifact"
      :comments="comments"
    ></ArtifactCommentView>

    <v-container
      v-if="$auth.loggedIn && !alreadyCommented && artifactReleased"
      fill-height
      fluid
      grid-list-xl
    >
      <v-row justify="center">
        <v-col cols="12">
          <material-card
            color="primary"
            title="Add Review"
            text="Provide your input"
          >
            <v-form ref="comment" v-model="valid">
              <v-container class="py-0">
                <v-row>
                  <v-col class="mx-0">
                    <small>
                      Terrible
                    </small>
                    <small style="margin-left: 105px">
                      Great
                    </small>
                    <v-rating
                      v-model="rating"
                      color="amber"
                      dense
                      hover
                      size="32"
                      required
                    ></v-rating>
                  </v-col>
                  <v-col cols="12">
                    <v-textarea
                      label="Review"
                      placeholder="Add review..."
                      v-model="comment"
                      required
                    >
                    </v-textarea>
                  </v-col>
                  <v-col cols="12" class="text-right">
                    <v-btn
                      color="success"
                      @click="onSubmit"
                      :disabled="!formCheck"
                    >
                      Add Review
                    </v-btn>
                  </v-col>
                </v-row>
              </v-container>
            </v-form>
          </material-card>
        </v-col>
      </v-row>
    </v-container>
    <v-container v-else-if="!$auth.loggedIn" fill-height fluid grid-list-xl>
      <v-row justify="center">
        <v-col cols="12">
          <material-card
            color="primary"
            title="Log in to add a review"
            nuxt
            to="/login"
          >
          </material-card>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script>
import { mapState } from 'vuex'

export default {
  components: {
    ArtifactCommentView: () => import('@/components/ArtifactCommentView')
  },
  head() {
    return {
      title: this.artifact.title,
      meta: [
        {
          hid: 'description',
          name: 'description',
          content: this.artifact.title
        }
      ]
    }
  },
  data() {
    return {
      valid: true,
      comment: '',
      rating: 0,
      ticket_status: '',
      artifactReleased: false
    }
  },
  watch: {
    artifact: {
      immediate: true,
      async handler(newVal, oldVal) {
        if (newVal !== oldVal) {
          await this.updateTicketStatus()
        }
      }
    }
  },
  computed: {
    ...mapState({
      artifact: state => state.artifacts.artifact,
      comments: state => state.artifacts.artifact.rating_review,
      userid: state => state.user.userid
    }),
    formCheck() {
      if (this.rating == 0 || this.comment === '') {
        return false
      }
      return true
    },
    alreadyCommented() {
      if (!this.comments) return false
      if (this.comments.find(c => c.review.reviewer.id == this.userid))
        return true
      return false
    }
  },
  async mounted() {
    this.$store.dispatch('artifacts/fetchArtifact', {
      artifact_group_id: this.$route.params.id
    })
  },
  methods: {
    async onSubmit() {
      if (!this.$auth.loggedIn) {
        this.$router.push('/login')
      } else {
        if (this.rating && this.comment) {
          let rating_payload = {
            rating: this.rating
          }
          // FIXME: backend API
          await this.$ratingsEndpoint.post(
            this.artifact.artifact.artifact_group_id,
            rating_payload
          )
          let comment_payload = {
            review: this.comment
          }
          // FIXME: backend API
          await this.$reviewsEndpoint.post(
            this.artifact.artifact.artifact_group_id,
            comment_payload
          )
          this.$store.dispatch('artifacts/fetchArtifact', {
            artifact_group_id: this.$route.params.id
          })

          this.rating = 0
          if (typeof this.$refs.comment !== 'undefined') {
            this.$refs.comment.reset()
          }
        } else {
        }
      }
    },
    async updateTicketStatus() {
      let response = await this.$artifactRequestStatusEndpoint.show(this.artifact.artifact.artifact_group_id)
      this.ticket_status = response.ticket_status
      this.artifactReleased = this.ticket_status && this.ticket_status == "released"
    },
  },
  
}
</script>
