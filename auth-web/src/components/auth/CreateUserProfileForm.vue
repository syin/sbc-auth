<template>
  <div>
    <v-form ref="form" lazy-validation>
      <v-expand-transition>
        <div class="form_alert-container" v-show="formError">
          <v-alert type="error" class="mb-0"
                  :value="true"
          >
            {{formError}}
          </v-alert>
        </div>
      </v-expand-transition>
      <!-- Username -->
      <v-row>
        <v-col cols="12" class="pt-0 pb-0">
          <v-text-field
              filled
              label="Username"
              req
              persistent-hint
              :rules="usernameRules"
              v-model="username"
              data-test="username"
          >
          </v-text-field>
        </v-col>
      </v-row>
      <!-- Password -->
      <v-row>
        <v-col cols="12" class="pt-0 pb-0">
          <v-text-field
              filled
              label="Password"
              req
              persistent-hint
              :rules="passwordRules"
              v-model="password"
              data-test="password"
          >
          </v-text-field>
        </v-col>
      </v-row>
      <!-- Confirm Password -->
      <v-row>
        <v-col cols="12" class="pt-0 pb-0">
          <v-text-field
              filled
              label="Confirm Password"
              req
              persistent-hint
              :error-messages="passwordMustMatch()"
              v-model="confirmPassword"
              data-test="confirm-password"
          >
          </v-text-field>
        </v-col>
      </v-row>

      <v-row>
        <v-col cols="12" class="form__btns pt-5">
            <v-btn
              large
              color="primary"
              class="save-continue-button"
              :loading="isLoading"
              :disabled='!isFormValid()'
              @click="nextStep"
              data-test="next-button"
            > Next
            </v-btn>
            <v-btn
              large
              depressed
              @click="cancel"
              data-test="cancel-button"
              class="cancel-button"
            > Cancel
            </v-btn>
        </v-col>
      </v-row>
    </v-form>
    <!-- Error Dialog -->
    <ModalDialog
      ref="errorDialog"
      :title="dialogTitle"
      :text="dialogText"
      dialog-class="notify-dialog"
      max-width="640"
    >
      <template v-slot:icon>
        <v-icon large color="error">mdi-alert-circle-outline</v-icon>
      </template>
      <template v-slot:actions>
        <v-btn large color="error" @click="close()" data-test="dialog-ok-button">OK</v-btn>
      </template>
    </ModalDialog>
  </div>
</template>

<script lang="ts">
import { Component, Mixins, Prop, Vue } from 'vue-property-decorator'
import ConfigHelper from '@/util/config-helper'
import ModalDialog from '@/components/auth/ModalDialog.vue'
import NextPageMixin from '@/components/auth/NextPageMixin.vue'
import { Organization } from '@/models/Organization'
import { UserProfileRequestBody } from '@/models/user'
import UserService from '@/services/user.services'

@Component({
  components: {
    ModalDialog
  }
})
export default class CreateUserProfileForm extends Mixins(NextPageMixin) {
    private username = ''
    private password = ''
    private confirmPassword = ''
    private formError = ''
    private isLoading = false
    private dialogTitle = ''
    private dialogText = ''

    @Prop() token: string

    $refs: {
      form: HTMLFormElement,
      errorDialog: ModalDialog
    }

    private usernameRules = [
      v => !!v || 'Username is required'
    ]

    private passwordRules = [
      v => !!v || 'Password is required'
    ]

    private passwordMustMatch (): string {
      return (this.password === this.confirmPassword) ? '' : 'Passwords must'
    }

    private isFormValid (): boolean {
      return this.$refs.form &&
        this.$refs.form.validate() &&
        !this.passwordMustMatch()
    }

    private async mounted () {
    }

    private async nextStep () {
      if (this.isFormValid()) {
        this.isLoading = true
        const requestBody: UserProfileRequestBody = {
          username: this.username,
          password: this.password
        }
        try {
          const response = await UserService.createUserProfile(this.token, requestBody)
          if (response?.data?.users?.length) {
            this.redirectToSignin()
          }
        } catch (error) {
          this.isLoading = false
          // TODO: Handle cases according to the type of the error
          if (error?.response?.data?.code) {
            switch (error.response.data.code) {
              case 'FAILED_ADDING_USER_IN_KEYCLOAK':
                this.showErrorModal('Failed to add the user, please try again')
                break
              case 'DATA_ALREADY_EXISTS':
                this.showErrorModal('There is already an user existing with the given username, please try with a new one')
                break
              default: this.showErrorModal()
            }
          } else {
            this.$emit('show-error-message')
          }
        }
      }
    }

    private redirectToSignin () {
      let redirectUrl = ConfigHelper.getSelfURL() + '/confirmtoken/' + this.token
      this.$router.push('/signin/bcros/' + encodeURIComponent(redirectUrl))
    }

    private cancel () {
      window.history.back()
    }

    close () {
      this.$refs.errorDialog.close()
    }

    showErrorModal (msg?) {
      this.dialogTitle = 'An error has occured'
      this.dialogText = msg || 'Something went wrong while attempting to create this profile. Please try again later.'
      this.$refs.errorDialog.open()
    }
}
</script>

<style lang="scss" scoped>
  @import '$assets/scss/theme.scss';

  .form__btns {
    display: flex;
    justify-content: flex-end;

    .v-btn + .v-btn {
      margin-left: 0.5rem;
    }
  }
</style>
