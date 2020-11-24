<template>
  <div class="container">
    <div class="main box">
      <Header />
      <ValidationObserver ref="observer" v-slot="{ passes, invalid }">
        <transition name="fade" mode="out-in">
          <div v-if="state === 'result'" key="loading" class="results">
            <div class="result-box">{{ decriptedData }}</div>
          </div>

          <div v-else key="form" class="body">
            <form class="form" @submit.prevent="passes()">
              <validation-provider
                v-slot="{ errors }"
                rules="required"
                name="Permittee private key"
                :class="'form-group'"
              >
                <input
                  v-model="formData.privateKey"
                  type="password"
                  class="form-field"
                  placeholder="Enter your private key"
                >
                <label for="id" v-text="'Enter your private key'" />
                <div class="alert" v-text="errors[0]" />
                <div class="form-hint">
                  This is an Ethereum private key of a permittee.
                </div>
              </validation-provider>

              <validation-provider
                v-slot="{ errors }"
                rules="required"
                name="Encrypted file data"
                :class="'form-group'"
              >
                <input
                  v-model="formData.encryptedData"
                  type="textarea"
                  class="form-field"
                  placeholder="Enter encrypted document source"
                >
                <label for="id" v-text="'Enter encrypted document source'" />
                <div class="alert" v-text="errors[0]" />
                <div class="form-hint">
                  This is an encrypted source of a document.
                </div>
              </validation-provider>

              <br>
              <div class="button-field">
                <button :disabled="invalid" class="button" @click="decrypt()">
                  Decrypt
                </button>
                Download the file using <a href="https://admin.genobank.io/test">admin.genobank.io</a>, populate this
                form with the douwnloaded data and press <b>Decrypt</b>.
              </div>
            </form>
          </div>
        </transition>
      </ValidationObserver>
    </div>
    <Footer />
  </div>
</template>

<script>
import { ValidationObserver, ValidationProvider, extend, localize } from 'vee-validate'
import { required, email, between } from 'vee-validate/dist/rules'
import en from 'vee-validate/dist/locale/en.json'
import * as eccryptoJS from 'eccrypto-js'
import Footer from '~/components/Footer'
import Header from '~/components/Header'

extend('required', required)
extend('email', email)
extend('between', between)
extend('wallet', (address) => {
  return /^(0x)?[0-9a-f]{40}$/i.test(address.toLowerCase())
})
extend('url', (url) => {
  return url.substr(0, 7) === 'http://' || url.substr(0, 8) === 'https://'
})
extend('date', (date) => {
  return !isNaN(new Date(date).getDate())
})
localize({ en })

export default {
  components: {
    Header,
    Footer,
    ValidationProvider,
    ValidationObserver
  },
  data () {
    return {
      state: 'form',
      formData: {
        encryptedData: '',
        privateKey: ''
      },
      decriptedData: 'Unable to decode'
    }
  },
  methods: {
    async decrypt () {
      try {
        const data = this.formData.encryptedData
        const privateKey = Buffer.from(this.formData.privateKey, 'hex')
        const encrypted = eccryptoJS.deserialize(Buffer.from(data, 'hex'))
        const decrypted = await eccryptoJS.decrypt(privateKey, encrypted)
        window.eval(`var jsondata = ${decrypted.toString()}`)
        jsondata.signatureImage = '<HIDDEN>'
        const json = JSON.stringify(jsondata, null, 2)
        this.decriptedData = json
        this.state = 'result'
      } catch (e) {
        console.error(e)
        this.state = 'result'
      }
    }
  }
}
</script>

<style lang="scss">
@import '~assets/_forms.scss';

.main {
    min-width: 500px;
}

.field {
  margin-top: 2px;
  background-color: #ECEFF1;
  border-radius: 4px;
  padding: 14px 30px 10px 14px;
  position: relative;

  p {
    padding-right: 1.5rem;
    word-break: break-all;
  }
}

.results {
  min-height: 462px;
  display: flex;
  flex-flow: column;
  justify-content: center;
  align-items: center;

  .button, .button-text {
    width: 100%;
    display: block;
    text-align: center;
  }

  .result-box {
    white-space: pre-wrap;
    overflow-x: auto;
    max-width: 1000px;
  }
}


</style>
