<template>
  <div v-if="record.artifact">
    <v-card class="mx-auto my-2">
      <v-card-title>{{ record.artifact.title }}</v-card-title>
      <v-card-subtitle>
        <div
          v-if="record.artifact.artifact_group.publication != null &&
                record.artifact.artifact_group.publication.artifact_id != record.artifact.id"
          align="left"
          class="mx-0"
        >
          &nbsp;&nbsp;(<a :href="`/artifact/${record.artifact.artifact_group_id}`">newest version</a>)
        </div>
        <div>
        </div>
<!--
     <span>Upload filled data use agreement here  <input type="file" @change="uploadFile" ref="file"></span><br>    -->

      </v-card-subtitle>

       </v-card>

   <div>
    <transition name="modal-fade">
          <DUAReviewModal
            v-show="isModalVisible"
            @close="closeModal"
            @submitRequest="submitRequest"
            v-bind:duaHTML="duaHTML">
          </DUAReviewModal>
        </transition>
     <form @submit.prevent="submitForm" ref="request_form">
     <!-- <div style="margin-top: 20px; font-weight: bold;">Please download and fill out data use agreement from<a @click="fetchDUA"> this link</a></div>
      <div style="margin-top: 20px; margin-bottom: 20px; font-weight: normal;">Upload filled data use agreement here (in PDF format) <input type="file" @change="uploadFile" ref="file" required accept="application/pdf"></div> -->

      <div style="font-weight: bold;">Enter project name<span style='color: red;'><strong> *</strong></span></div>
      <v-text-field
        name="project"
        v-model="project"
        type="text"
        hint="Enter your project name"
        auto-grow
        clearable
        required
      ></v-text-field>
      <div style="margin-top: 20px; font-weight: bold;">Briefly describe the research to be done with the dataset<span style='color: red;'><strong> *</strong></span></div>
      <v-textarea
        name="project_description"
        v-model="project_description"
        type="text"
        hint="Enter your research purpose"
        auto-grow
        clearable
        required
      ></v-textarea>
      <div v-if="record.artifact.provider == 'USC'">
      <div style="margin-top: 20px; font-weight: bold;">Briefly justify (a few sentences) about why the dataset is needed to advance that research.<span style='color: red;'><strong> *</strong></span></div>
      <v-textarea
        name="project_justification"
        v-model="project_justification"
        type="text"
        hint="Enter your justification for using this dataset."
        auto-grow
        clearable
        required
      ></v-textarea>
      </div>
      <div style="margin-top: 20px; font-weight: bold;">Enter details of the researcher representative</div>
      <v-container>
        <v-row align="center">
          <v-col md="5">
            <v-text-field
              v-model="representative_researcher.name"
              type="text"
              hint="Enter researcher name"
              required>
              <template #label>
                  <span>Name<span style='color: red;'> *</span></span>
              </template>
            </v-text-field>
          </v-col>
          <v-col md="3">
            <v-text-field
              v-model="representative_researcher.email"
              label="Email"
              type="email"
              hint="Enter researcher email"
              required>
              <template #label>
                  <span>Email<span style='color: red;'> *</span></span>
              </template>
            </v-text-field>
          </v-col>
          <v-col md="3">
            <v-text-field
              v-model="representative_researcher.number"
              type="text"
              hint="Enter researcher phone number"
              required
              pattern="^[0-9]+$">
              <template #label>
                  <span>Phone Number (only digits)<span style='color: red;'> *</span></span>
              </template>
            </v-text-field>
          </v-col>
          <v-col md="5">
            <v-text-field
              v-model="representative_researcher.organization"
              type="text"
              hint="Enter researcher organization"
              required>
              <template #label>
                  <span>Organization<span style='color: red;'> *</span></span>
              </template>
            </v-text-field>
          </v-col>
          <v-col md="5">
            <v-text-field
              v-model="representative_researcher.title"
              type="text"
              hint="Enter researcher title"
              required>
              <template #label>
                  <span>Researcher Title<span style='color: red;'> *</span></span>
              </template>
            </v-text-field>
          </v-col>
        </v-row>
      </v-container>

      <div style="margin-top: 20px; font-weight: bold;">Enter details of all other researchers that will interact with the data:</div>
      <div v-for="(researcher, index) in researchers_that_interact" :key="index">
        <v-container>
          <v-row align="center">
            <v-col md="5">
              <v-text-field
                v-model="researcher.name"
                type="text"
                hint="Enter researcher name"
                required>
                <template #label>
                  <span>Name<span style='color: red;'> *</span></span>
                </template>
              </v-text-field>
            </v-col>
            <v-col md="3">
              <v-text-field
                v-model="researcher.email"
                type="email"
                hint="Enter researcher email"
                required>
                <template #label>
                  <span>Email<span style='color: red;'> *</span></span>
                </template>
              </v-text-field>
            </v-col>
            <v-col md="3">
              <v-text-field
                v-model="researcher.number"
                label="Phone Number (only digits)"
                type="text"
                hint="Enter researcher phone number"
                pattern="^[0-9]+$"
              ></v-text-field>
            </v-col>
            <v-col md="1">
              <div>
                <v-icon
                  v-if="researchers_that_interact.length > 0"
                  color="error"
                  @click="deleteResearcher(index)"
                >
                  mdi-delete
                </v-icon>
              </div>
            </v-col>
          </v-row>
        </v-container>
      </div>
      <div style="margin-top: 20px;">
        <v-btn
          class="success ml-2 mb-2"
          fab
          x-small
          @click="addResearcher"
        >
          <v-icon>mdi-plus</v-icon>
        </v-btn>
      </div>
      <div v-if ="record.artifact.irb" style="margin-top: 20px; font-weight: bold;">Upload IRB approval Letter</div>
      <v-file-input v-if ="record.artifact.irb" v-model = "irbContent" clearable label="Upload IRB approval letter" variant="underlined" @change ="getContent"></v-file-input>
      <div>
      <input type="submit" value="Review" class="btn-submit" style="margin-top: 20px;" :disabled="requestMode">
      </div>
    </form>

    <v-dialog
    v-model="formSubmitted"
    width="auto "
    >
      <v-card>
        <v-card-text>
          <div v-if="formSubmittedError" class="form-submit-error">
            <p>{{formSubmittedErrorMessage}}</p>
          </div>
          <div v-else class="form-submit-success">
            <p>Request submitted successfully!</p>
          </div>
        </v-card-text>
        <v-card-actions>
          <v-btn color="primary" block @click="$router.push('/');">Go to homepage</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>

   </div>
  <div v-else>
    {{ loadingMessage }}
  </div>
</template>

<script>
import { mapState } from 'vuex'
import { artifactIcon, artifactColor, bytesToSize } from '@/helpers'
import { marked } from 'marked'

export default {
  name: 'KGArtifactLong',
  props: {
    record: {
      type: Object,
      required: true
    }
  },
  components: {
    ArtifactChips: () => import('@/components/ArtifactChips'),
    ArtifactCurationList: () => import('@/components/ArtifactCurationList'),
    JsonPrettyPrint: () => import('@/components/pretty-print'),
    DUAReviewModal: () => import('@/components/DUAReviewModal')
  },
  data() {
    return {
      loading: true,
      loaded: false,
      expanded: false,
      history_expanded: false,
      diff_from: -1,
      diff_to: -1,
      diff_results: [],
      diff_results_dialog: false,
      diff_results_tab: "visual",
      loadingMessage: 'Loading...',
      project: "",
      project_description: "",
      project_justification: "",
      people: "",
      Images: null,
      formSubmitted: false,
      formSubmittedError: false,
      formSubmittedErrorMessage: "",
      representative_researcher: {name: "", email: "", number: "", organization:"", title:""},
      researchers_that_interact: [],
      researchers: [],
      requestMode: false,
      dua: null,
      duaHTML: null,
      isModalVisible: false,
      irbContent : '',
      irbData: '',
    }
  },
  mounted() {
    setTimeout(() => {
      this.loadingMessage = 'Error loading'
    }, 5000)

  },
  computed: {
    ...mapState({
      userid: state => state.user.userid,
      favorites: state => state.artifacts.favoritesIDs,
      user_is_admin: state => state.user.user_is_admin
    }),
    sanitizedDescription: function() {
      return this.$sanitize(this.record.artifact.description)
    },
    favorite: {
      get() {
        return this.favorites[this.record.artifact.artifact_group_id] ? true : false
      },
      set(value) {
        if (value)
          this.$store.commit('artifacts/ADD_FAVORITE', this.record.artifact.artifact_group_id)
        else
          this.$store.commit(
            'artifacts/REMOVE_FAVORITE',
            this.record.artifact.artifact_group_id
          )
      }
    },
    tags() {
      let tags = []
      if (this.record.artifact.tags.length > 0) {
        return this.record.artifact.tags.map(e => e.tag)
      }
      let top = this.record.artifact.meta.find(o => o.name == 'top_keywords')
      if (top) {
        tags = tags.concat(JSON.parse(top.value).map(e => e[0]))
      }
      top = this.record.artifact.meta
        ? this.record.artifact.meta.find(o => o.name == 'top_ngram_keywords')
        : null
      if (top) {
        tags = tags.concat(JSON.parse(top.value).map(e => e[0]))
      }
      // return only unique
      let t = [...new Set(tags)]
      t = t.filter(
        el => !this.record.artifact.tags.map(e => e.tag).includes(el)
      )
      return t
    },
    badgesPresent() {
      return (
        typeof this.record.artifact.badges !== 'undefined' &&
        this.record.artifact.badges.length > 0
      )
    },
    languages() {
      let csv = this.record.artifact.meta.find(o => o.name == 'languages')
      if (csv) {
        return csv.value ? csv.value.split(',') : []
      } else {
        return []
      }
    },
    homepage() {
      let hp = this.record.artifact.meta.find(o => o.name == 'homepage')
      if (!hp) return null
      return hp.value
    },
    stars() {
      let stars = this.record.artifact.meta.find(
        o => o.name == 'stargazers_count'
      )
      if (!stars) return null
      return stars.value
    },
    watchers() {
      let watchers = this.record.artifact.meta.find(
        o => o.name == 'watchers_count'
      )
      if (!watchers) return null
      return watchers.value
    },
    license() {
      return this.record.artifact.license
        ? this.record.artifact.license.short_name +
            ' (' +
            this.record.artifact.license.long_name +
            ')'
        : ''
    },
    markdown() {
      let readmes = {}
      this.record.artifact.files.map(f => {
        readmes = f.members.find(m => m.name.toUpperCase() == 'README.MD')
      })
      console.log(readmes)
      if (typeof readmes !== 'undefined' && readmes.file_content)
         return atob(readmes.file_content.content)
    },
    hideOverflow() {
      return {
        hideoverflow: !this.expanded
      }
    },
    isOverflow() {
      if (!this.loaded) return false
      let element = this.$refs['markdown']
      return element.offsetHeight >= 250
    },
    overflowIcon() {
      if (!this.expanded) return 'mdi-chevron-down'
      else return 'mdi-chevron-up'
    },
    overflowText() {
      if (!this.expanded) return 'Show All'
      else return 'Collapse'
    },
    published() {
      return this.record.artifact.publication ? true : false
    }
  },
  methods: {
    showModal() {
      this.isModalVisible = true;
    },
    closeModal() {
      this.isModalVisible = false;
    },
    async deleteResearcher(index) {
      this.researchers_that_interact.splice(index, 1);
    },
    async addResearcher() {
      let researcherObj = {
        name: "",
        email: "",
        number: ""
      }
      this.researchers_that_interact.push(researcherObj);
    },
    async favoriteThis() {
      if (!this.$auth.loggedIn) {
        this.$router.push('/login')
      } else {
        let action = !this.favorite
        this.favorite = !this.favorite
        if (action) {
          // FIXME: backend API
          await this.$favoritesEndpoint.post(this.record.artifact.artifact_group_id, {})
        } else {
          await this.$favoritesEndpoint.delete(this.record.artifact.artifact_group_id)
        }
      }
    },
    uploadFile() {
        this.Images = this.$refs.file.files[0];
    },
    getContent(){

      if (this.irbContent == null){
        return
      }

      if (!this.irbContent) {this.data = "No File Chosen"}
      var reader = new FileReader();

      // Use the javascript reader object to load the contents
      // of the file in the v-model prop
      reader.readAsBinaryString(this.irbContent);
      reader.onload = () => {
        this.irbData = reader.result;


      }


    },
    submitForm() {
      this.project = this.project.trim();
      this.project_description = this.project_description.trim();
      this.project_justification = this.project_justification.trim();
      this.representative_researcher.name = this.representative_researcher.name.trim();
      this.representative_researcher.email = this.representative_researcher.email.trim();
      this.representative_researcher.number = this.representative_researcher.number.trim();
      this.representative_researcher.organization = this.representative_researcher.organization.trim();
      this.representative_researcher.title = this.representative_researcher.title.trim();
      this.researchers_that_interact.forEach((researcher) => {
        researcher.name = researcher.name.trim();
        researcher.email = researcher.email.trim();
        researcher.number = researcher.number.trim();
      });
      // Researchers is a combination of representative_researcher and the researchers_that_interact list
      this.researchers = [this.representative_researcher].concat(this.researchers_that_interact)
      if (this.$refs.request_form.checkValidity()) {
        // this.submitRequest();
        this.fetchDUA();
      } else {
        this.$refs.request_form.reportValidity();
      }
    },
    iconColor(type) {
      return artifactColor(type)
    },
    iconImage(type) {
      return artifactIcon(type)
    },
    convertSize(size) {
      return bytesToSize(size)
    },
    isAdmin() {
      return this.user_is_admin
    },
    isOwner() {
      if (this.user_is_admin) return true
      return typeof this.record.artifact.owner !== 'undefined'
        ? this.record.artifact.owner.id == this.userid
        : false
    },
    async newVersion() {
      let response = await this.$artifactEndpoint.post(
        [this.record.artifact.artifact_group_id, this.record.artifact.id],{})
      this.$store.dispatch('artifacts/fetchArtifact', {
        artifact_group_id: response.artifact.artifact_group_id,
        id: response.artifact.id
      })
      this.$router.push("/artifact/" + response.artifact.artifact_group_id
        + "/" + response.artifact.id + "?edit=true")
    },
    async reImportNewVersion() {
      let response = await this.$artifactEndpoint.post(
        [this.record.artifact.artifact_group_id, this.record.artifact.id],
        { reimport: true })
      this.$router.push("/import")
    },
    async getDiff(from, to) {
      this.diff_from = from
      this.diff_to = to
      let response = await this.$artifactCompareEndpoint.show(
        [this.record.artifact.artifact_group_id, from],
        { target_artifact_id: to}
      )
      this.diff_results = response.curations.map(
        function(x) { return { curation: x }; }
      )
      console.log(this.diff_results)
      for (var i = 0; i < this.diff_results.length; ++i) {
        // NB: opdata from server is a string, not JSON itself.
        this.diff_results[i].curation.opdata =
          JSON.parse(this.diff_results[i].curation.opdata)

        // NB: curations might not have IDs, as in this case,
        // where the server generated a diff.  So we have to
        // cons up an id for the ArtifactCurationList component.
        this.diff_results[i]._id = i
      }
      this.diff_results_dialog = true
    },
    async submitRequest() {
      this.formSubmitted = false;
      let isEntryEmpty = false;
      if ((this.representative_researcher.name == "" || this.representative_researcher.email == "" || this.representative_researcher.number == "" || this.representative_researcher.organization == "" || this.representative_researcher.title == "")) {
          isEntryEmpty = true;
      }
      this.researchers.forEach((researcher) => {
        if ((researcher.name == "" || researcher.email == "")) {
          isEntryEmpty = true;
        }
      });
      if(this.record.artifact.provider == 'USC') {
        if (this.project_justification == "") {
          isEntryEmpty = true;
        }
      }
      if(!(this.project_description) || isEntryEmpty || !(this.project)) {
        this.formSubmittedError = true;
        this.formSubmittedErrorMessage = "Please fill all the fields";
        this.formSubmitted = true;
        return;
      }

      this.formSubmittedError = false;
      this.formSubmittedErrorMessage = "";
      this.requestMode = true;
      const payload = new FormData();



      let blob = new Blob([this.duaHTML], { type: 'text/plain' });
      let file = new File([blob], "signed_dua.html", {type: "text/plain"});
      let researchersJSON = JSON.stringify(this.researchers);
      payload.append('file', file);
      if(this.record.artifact.irb){
        let pdfBlob = new Blob([this.irbData], {type: 'application/pdf'})
        let pdfFile = new File([pdfBlob], "IRB_approval_letter.pdf", {type: 'application/pdf'})
        payload.append('pdf_file', pdfFile);
      }
      payload.append('project', this.project);
      payload.append('project_description', this.project_description);
      payload.append('project_justification', this.project_justification);
      payload.append('researchers', researchersJSON);
      payload.append('dataset', this.record.artifact.title)
      payload.append('representative_researcher_email', this.representative_researcher['email']);

      let response = await this.$artifactRequestEndpoint.post(
        [this.record.artifact.artifact_group_id, this.record.artifact.id],payload
      );
      if(response.status == 1) {
        this.formSubmittedError = true;
        this.formSubmittedErrorMessage = response.error;
      } else {
        this.formSubmittedError = false;
      }
      this.formSubmitted = true;
      this.requestMode = false;
      this.closeModal();
    },
    async fetchDUA() {
      // get representative from researchers

      let response = await this.$duaEndpoint.show(
        this.record.artifact.artifact_group_id,
        {
          'researchers': JSON.stringify(this.researchers),
          'project': this.project,
          'project_description': this.project_description,
          'dataset_name': this.record.artifact.title,
          'representative_researcher': JSON.stringify(this.representative_researcher)
        }
      );
      this.dua = response.dua;
      this.duaHTML = marked(this.dua);
      this.showModal();
      //TODO: Sign DUA
    },
  }
}
</script>

<style scoped>
.hideoverflow {
  max-height: 250px;
  overflow: hidden;
}
</style>

<style>
  form {
    padding: 10px;
    border-radius: 5px;
  }

  input {
    padding: 4px 8px;
    margin: 4px;
  }

  /* span {
    font-size: 18px;
    margin: 4px;
    font-weight: 500;
  } */

  .submit {
    font-size: 15px;
    color: #fff;
    padding: 6px 12px;
    border: none;
    margin-top: 8px;
    cursor: pointer;
    border-radius: 5px;
  }

  .form-submit-error {
    color: red;
    padding: 10px;
    font-size: 15px;
  }

  .form-submit-success {
    color: green;
    padding: 10px;
    font-size: 15px;
  }

  .btn-submit {
    background-color: #4CAF50;
    color: white;
    font-weight: bold;
    border-radius: 4px;
    padding: 8px;
    width: 100px;
  }



</style>
