<template>
  <v-container class="my-4 my-sm-8">
    <img
      src="~assets/img/logo.png"
      alt="BuildChester"
      style="height: 64px; max-width: 100%;"
      class="mb-4 mb-sm-6 mx-auto d-block"
    />

    <v-row justify="center" justify-lg="space-around">
      <!-- main -->
      <v-col
        cols="12"
        lg="8"
        class="max-width-600 elevation-10 rounded-2 pt-3 pt-sm-5 px-sm-5 pb-4 pb-lg-6"
      >
        <h1
          class="headline text-uppercase mb-3 primary--text mb-3 mb-sm-4 font-weight-bold"
        >
          Housing Scheme Application
        </h1>

        <v-alert
          v-text="errorResponseMessage"
          :value="!!errorResponse"
          text
          dense
          color="error"
          class="mb-3"
          elevation="5"
          border="left"
          transition="slide-y-reverse-transition"
        />

        <v-form ref="formEl" v-model="formIsValid" @submit.prevent="submit">
          <!-- v-text-field email -->
          <v-text-field
            v-model="form.fullName"
            :rules="formRules.fullName"
            name="full-name"
            class="form__name"
            label="Full Name"
            type="text"
            dense
            outlined
            required
            autofocus
          />

          <!-- v-text-field phone -->
          <!-- FIXME: v-mask required 1 input but found 2 -->
          <!-- <v-text-field
            v-model="form.phoneNumber"
            v-mask="'### ### ####'"
            :rules="formRules.phoneNumber"
            :prefix="country.dialCode"
            name="phoneNumber"
            class="form__phone-number"
            label="Phone Number"
            type="tel"
            placeholder="8030041152"
            dense
            outlined
            required
          > -->
          <v-text-field
            v-model="form.phoneNumber"
            :rules="formRules.phoneNumber"
            :prefix="country.dialCode"
            required
            name="phoneNumber"
            class="form__phone-number"
            label="Phone Number"
            type="tel"
            placeholder="8030041152"
            dense
            outlined
          >
            <template #prepend>
              <v-autocomplete
                v-model="country"
                :items="countries"
                item-text="code"
                item-value="code"
                label="Country"
                return-object
                dense
                outlined
                style="width: 110px;"
              >
                <template #prepend-inner>
                  <v-img
                    :src="`/icons/svg/flags/${country.code.toLowerCase()}.svg`"
                    :height="20"
                    :width="26.6667"
                    class="rounded-1"
                  />
                </template>

                <template #item="{ item, on }">
                  <v-list-item-content v-on="on">
                    <v-list-item-title class="d-inline-flex align-center">
                      <v-img
                        :src="`/icons/svg/flags/${item.code}.svg`"
                        :height="20"
                        :width="26.6667"
                        class="flex-grow-0"
                      />
                      <span class="text-ellipsis ml-2">
                        {{ item.dialCode }}
                      </span>
                    </v-list-item-title>
                    <!-- <v-list-item-title v-text="item.name" /> -->

                    <v-list-item-subtitle v-text="item.name" class="mt-1" />
                  </v-list-item-content>
                </template>
              </v-autocomplete>
            </template>
          </v-text-field>

          <!-- v-text-field email -->
          <v-text-field
            v-model="form.email"
            :rules="formRules.email"
            name="email"
            class="form__email"
            label="Email"
            type="email"
            dense
            outlined
            required
          />

          <v-row dense>
            <v-col cols="3" class="mb-0">
              <!-- v-text-field occupation -->
              <v-menu :close-on-content-click="false" offset-y>
                <template #activator="{ on }">
                  <v-text-field
                    v-model="age"
                    :rules="formRules.age"
                    v-on="on"
                    name="age"
                    class="form__age"
                    label="Age"
                    type="number"
                    dense
                    outlined
                    required
                    readonly
                  />
                </template>

                <template #default>
                  <v-date-picker
                    v-model="pickedDate"
                    class="mt-4"
                    min="1860-01-01"
                    max="2010-01-01"
                    color="secondary"
                    header-color="primary"
                  ></v-date-picker>
                </template>
              </v-menu>
            </v-col>

            <v-col cols="9" class="mb-0">
              <!-- v-text-field occupation -->
              <v-text-field
                v-model="form.occupation"
                :rules="formRules.occupation"
                name="occupation"
                class="form__occupation"
                label="Occupation"
                type="text"
                dense
                outlined
                required
                auto-grow
              />
            </v-col>
          </v-row>

          <!-- v-text-area address -->
          <v-textarea
            v-model="form.address"
            :rules="formRules.address"
            name="address"
            class="form__address"
            label="Address"
            type="text"
            rows="1"
            auto-grow
            dense
            outlined
            required
          />

          <v-layout align-baseline wrap>
            <!-- <v-flex shrink class="mr-8 mb-4 mt-4">
              <nuxt-link to="/sign-in" class="text--primary"
                >Have an account instead?
              </nuxt-link>
            </v-flex> -->
            <v-flex shrink>
              <!-- <v-btn
                id="sign-up-btn"
                :loading="formIsProcessing"
                large
                color="primary"
                class="mx-auto"
              >
                Create an Account
              </v-btn> -->
              <v-btn
                id="sign-up-btn"
                :loading="formIsProcessing"
                large
                color="secondary"
                type="submit"
              >
                Submit
              </v-btn>
            </v-flex>
          </v-layout>

          <v-dialog
            v-model="shouldShowDialog"
            max-width="400px"
            content-class="white position-relative px-3 py-4"
          >
            <v-icon size="64" color="success" class="mb-2 d-block mx-auto">
              {{ mdiCheckCircleOutline }}
            </v-icon>

            <h4 class="success--text text--darken-2 mb-1">
              Submitted Successfully
            </h4>

            <p class="mb-2">
              Your request has been received. We'll get back to you as soon as
              possible.
            </p>

            <p class="mb-0 ">
              <small>Here's your <strong>Reference ID:</strong></small>
              <code>{{ referenceId }}</code>
            </p>

            <p class="subtitle text--secondary">
              We've sent a copy of the
              <span class="text--primary">Reference ID</span> to your email.
            </p>
          </v-dialog>
        </v-form>
      </v-col>
      <!-- main -->

      <!-- sidebar -->
      <v-col cols="12" lg="4" class="mt-6 mt-md-9">
        <h2 class="headline mb-2">
          Don't hesitate to get your housing today
        </h2>

        <p>
          BuildChester is a Registered Multi-Purpose Construction & Civil
          Engineering Company
        </p>
      </v-col>
      <!-- sidebar -->
    </v-row>
  </v-container>
</template>

<script>
import { mdiCheckCircleOutline } from '@mdi/js'
import countryCodes from '@/data/country-codes.json'

export default {
  name: 'ApplyNow',

  head: {
    title: 'Apply Now'
  },

  data() {
    return {
      countries: countryCodes,

      form: {
        fullName: '',
        // firstName: '',
        // lastName: '',
        email: '',
        phoneNumber: '',
        occupation: '',
        address: ''
      },

      pickedDate: null,

      formIsValid: false,

      formRules: {
        fullName: [
          (value) => this.isFieldEmpty(value, 'Full Name'),
          (value) => this.isFieldLengthy(value, 60, 'Full Name'),
          (value) =>
            /^([a-zA-Z]+|[a-zA-Z]+\s{1}[a-zA-Z]{1,}|[a-zA-Z]+\s{1}[a-zA-Z]{3,}\s{1}[a-zA-Z]{1,})$/g.test(
              value
            ) || 'Use only Alphabets, and Space to separate names'
          // (value) =>
          //   /^[a-zA-Z][a-zA-Z ]+[a-zA-Z]$/g.test(value) ||
          //   'Use only Alphabets, and Space to separate names'
        ],
        /* firstName: [
          (value) =>
            this.isFieldEmpty(value, 'First Name'),
          (value) =>
            this.isFieldLengthy(value, 30, 'First Name'),
          (value) =>
            !/[^A-Za-z]+/g.test(value) || 'Use only Alphabets'
        ],
        lastName: [
          (value) =>
            this.isFieldEmpty(value, 'Last Name'),
          (value) =>
            this.isFieldLengthy(value, 30, 'Last Name')
        ], */
        email: [
          (value) => this.isFieldEmpty(value, 'Email'),
          (value) => this.isFieldLengthy(value, 100, 'Email'),
          (value) => /.+@.+/.test(value) || 'Type a valid email address'
        ],
        phoneNumber: [
          (value) => this.isFieldEmpty(value, 'Phone Number'),
          (value) => this.isFieldLengthy(value, 11, 'Phone Number'),
          (value) => !/[^0-9]+/.test(value) || 'Should only contain numbers'
        ],
        age: [
          (value) => this.isFieldEmpty(value, 'Age'),
          (value) => Number(value) > 10 || 'Too young'
        ],
        address: [
          (value) => this.isFieldEmpty(value, 'Address'),
          (value) => this.isFieldLengthy(value, 300, 'Address')
        ]
      },

      shouldShowCountries: false,

      shouldShowDialog: false,

      formIsProcessing: false,

      errorResponse: '',

      referenceId: '',

      mdiCheckCircleOutline
    }
  },

  computed: {
    age() {
      if (!this.pickedDate) return ''

      const ageDiffMs = new Date() - new Date(this.pickedDate)
      const ageDate = new Date(ageDiffMs) // miliseconds from epoch
      return Math.abs(ageDate.getUTCFullYear() - 1970)
    },

    errorResponseMessage() {
      if (this.errorResponse === 'EMAIL EXISTS') {
        return `You've already used this email to apply for housing scheme. In case you are yet to get our response, we'll get back to you soon.`
      } else if (this.errorResponse === '') {
        return ''
      } else {
        return 'An Error occurred. Please reload the page or try again later.'
      }
    }
  },

  watch: {
    pickedDate(oldValue, newValue) {
      // eslint-disable-next-line no-console
      console.info(
        `About to chane pickedDate from '${oldValue}' to '${newValue}';`
      )
    }
  },

  async asyncData({ app, route }) {
    const data = {}

    const countryIndex = countryCodes.findIndex(
      (country) => country.code === 'NG'
    )

    data.country = countryCodes[countryIndex]

    try {
      const { data: responseData } = await app.$axios({
        method: 'get',
        url: 'http://ip-api.com/json/?fields=countryCode'
      })

      const countryIndex = countryCodes.findIndex(
        (country) => country.code === responseData.countryCode
      )

      if (countryIndex !== -1) {
        data.country = countryCodes[countryIndex]
      }
    } catch (error) {}

    // console.info(data)
    return data
  },

  methods: {
    async submit() {
      if (!this.formIsValid || this.formIsProcessing) return

      // eslint-disable-next-line no-console
      console.info('The form is being processed')

      this.formIsProcessing = true
      this.errorResponse = ''
      this.referenceId = ''

      const data = {
        ...this.form,
        dateOfBirth: new Date(this.pickedDate),
        age: this.age,
        dateCreated: new Date()
      }

      data.phoneNumber = this.country.dialCode + this.form.phoneNumber

      try {
        const { data: response } = await this.$axios({
          method: 'post',
          url: `${process.env.DB_HOST}/add`,
          data
        })

        if (response.error) {
          // eslint-disable-next-line no-console
          console.error(response.error)
          this.errorResponse = response.error.message
        } else {
          this.referenceId = response.referenceId
          this.shouldShowDialog = true
        }
      } catch (error) {
        // eslint-disable-next-line no-console
        console.error(error)
        // this.errorResponse = error.error.message
      }

      this.formIsProcessing = false
    },

    isFieldEmpty(value, fieldName) {
      return !!value || `${fieldName} is required`
    },

    isFieldLengthy(value, maxLength, fieldName) {
      return (
        value.length <= maxLength ||
        `${fieldName} must be less than ${maxLength} characters`
      )
    },

    reset() {
      // @ts-ignore
      this.$refs.formEl.reset()
      // this.form = {
      //   firstName: '',
      //   lastName: '',
      //   email: '',
      //   phoneNumber: '',
      //   password: '',
      //   confirmPassword: ''
      // }
    }
  }
}
</script>

<style lang="scss" scoped></style>
