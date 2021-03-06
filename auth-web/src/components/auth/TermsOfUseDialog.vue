<template>
  <v-row>
    <v-col sm="12">
      <v-tooltip bottom>
        <template v-slot:activator="{ on }">
          <v-checkbox
            v-on:change="emitStatus()"
            v-on="on"
            class="terms-checkbox"
            color="default"
            v-model="termsAccepted"
            required
            :disabled="termsAccepted || !canCheckTerms"
          >
            <template v-slot:label>
              <div class="terms-checkbox-label" v-on="on">
                <span>I have read and agreed to the</span>
                <v-btn
                  text
                  link
                  color="primary"
                  class="pr-1 pl-1"
                  @click.stop="openDialog()"
                  data-test="terms-of-use-checkbox"
                >Terms of Use</v-btn>
              </div>
            </template>
          </v-checkbox>
        </template>
        <span>{{tooltipTxt}}</span>
      </v-tooltip>
      <v-dialog scrollable width="1024" v-model="termsDialog" :persistent="true">
        <v-card>
          <v-card-title>Terms of Use</v-card-title>
          <v-card-text id="scroll-target" data-test="scroll-area">
            <div v-scroll:#scroll-target="onScroll" style="height: 2000px;">
              <terms-of-use
                :content="content"
              ></terms-of-use>
            </div>
          </v-card-text>
          <v-card-actions>
            <v-btn
              large
              depressed
              color="primary"
              class="agree-btn"
              :disabled="!atBottom"
              @click="termsDialog = false; termsAccepted = true; emitStatus()"
              data-test="accept-button"
            >
              <span>Agree to Terms</span>
            </v-btn>
            <v-btn
              large
              depressed
              color="primary"
              class="agree-btn"
              @click="termsDialog = false"
              data-test="close-button"
            >
              <span>Close</span>
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-col>
  </v-row>
</template>

<script lang="ts">
import { Component, Emit, Prop, Vue, Watch } from 'vue-property-decorator'
import TermsOfUse from './TermsOfUse.vue'
import { User } from '@/models/user'
import documentService from '@/services/document.services.ts'
import { getModule } from 'vuex-module-decorators'
import { mapState } from 'vuex'

@Component({
  components: {
    TermsOfUse
  },
  computed: {
    ...mapState('user', ['userProfile'])
  }
})
export default class TermsOfServiceDialog extends Vue {
  private readonly userProfile!: User
  private termsDialog = false
  private termsAccepted = false
  private content: string = ''
  private version: string = ''
  private canCheckTerms: boolean = false
  private atBottom = false

  async mounted () {
    const response = await documentService.getTermsOfService('termsofuse')
    this.content = response.data.content
    this.version = response.data.version_id
  }

  get tooltipTxt () {
    return 'Please read and agree to the Terms Of Use'
  }

  @Watch('version')
  onDocVersionChanged (val: string, oldVal: string) {
    if (val === this.userProfile.userTerms.termsOfUseAcceptedVersion) {
      this.termsAccepted = true
      this.canCheckTerms = true
      this.emitStatus()
    }
  }

  private openDialog () {
    this.termsDialog = true
  }

  private onScroll (e) {
    this.atBottom = (e.target.scrollHeight - e.target.scrollTop) <= (e.target.offsetHeight + 25)
  }

  @Emit('terms-updated')
  private emitStatus () {
    return {
      termsVersion: this.version,
      isTermsAccepted: this.termsAccepted
    }
  }
}
</script>

<style lang="scss" scoped>
@import '$assets/scss/theme.scss';

// Tighten up some of the spacing between rows
[class^='col'] {
  padding-top: 0;
  padding-bottom: 0;
}

.terms-checkbox {
  pointer-events: auto !important;
}

.form__btns {
  display: flex;
  justify-content: flex-end;
}

.terms-checkbox-label {
  display: flex;
  align-items: center;
  margin-top: -0.1rem;

  .v-btn {
    height: auto !important;
    margin-left: 0.1rem;
    padding: 0;
    text-decoration: underline;
    font-size: 1rem;
  }
}

.v-card__actions {
  justify-content: center;

  .v-btn {
    width: 10rem;
  }
}

.terms-container ::v-deep {
  article {
    background: $gray1;
  }
}
</style>
