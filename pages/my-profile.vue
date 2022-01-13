<template>
<div class="mh-100v">
  <v-row v-if='loadingPage'>
    <v-col cols='12'>
      <v-skeleton-loader></v-skeleton-loader>
    </v-col>
  </v-row>
  <a-row v-else class="pa-6">
    <v-col :xs="24" :sm="24">
      <div>
        <p class="h4 mb-1"  style="display: flex; align-items: center">
          {{$t('hi')}}, {{name}} <span v-if="!isUser && userRole !== 'guest'" class="user-role ml-1">
        {{$t('your_role')}}:
        <b class="b-white">{{userRoleTxt}}</b>
      </span></p>
      </div>
      <div>
        <a-tabs v-if='showTabs' :default-active-key="tab.toString()">
          <a-tab-pane key='1' tab='Profile Picture'>
            <div class='profile-upload'>
              <div v-if='showUploadPicture' class='upload-box'>
                <a-upload
                  name="avatar"
                  list-type="picture-card"
                  class="avatar-uploader"
                  :show-upload-list="false"
                  :before-upload="beforeUpload"
                  @change="handleChange"
                >
                  <img v-if="src && !loadingUpload" :src="src" alt="avatar" />
                  <div v-else>
                    <a-icon :type="loadingUpload ? 'loading' : 'plus'" />
                    <div class="ant-upload-text">
                      Upload
                    </div>
                  </div>
                </a-upload>
                <div class='buttons'>
                  <a-button v-if='src && src.length' block type='primary' @click='uploadPicture'>
                    <SpinOrText v-model='loadingUpload'>
                      Save
                    </SpinOrText>
                  </a-button>
                  <a-button block type='danger' @click='cancelUpload' >Cancel</a-button>
                </div>
              </div>
              <div v-else >
                <div>
                  <ProfilePicture :full-name='name' :picture='picture'>
                    <div class='profile-overlay' @click='showUploadPicture = true'>
                      <a-icon type='upload'></a-icon>
                    </div>
                  </ProfilePicture>
                </div>
              </div>
            </div>
          </a-tab-pane>
          <a-tab-pane key="2" tab="Personal Information">
            <v-row v-if="!$auth.user.has_phone">
              <v-col>
                <v-alert type="warning">Please add your phone number</v-alert>
              </v-col>
            </v-row>
            <v-form ref="personalForm" v-model="validPersonalForm"  @submit.prevent="handleSubmit">
              <v-row>
                <v-col md="6">
                  <v-text-field v-model="first_name" :placeholder="$t('fn')" :label="$t('fn')" :rules="[v => !!v ||  $t('v.fn_req'), v => !!v && v.length > 2 || $t('v.min_3', v => !!v && v.length <= 30 || $t('v.max_30'))]"></v-text-field>
                </v-col>
                <v-col md="6">
                  <v-text-field v-model="last_name" :placeholder="$t('ln')" :label="$t('ln')" :rules="[v => !!v ||  $t('v.fn_req'), v => !!v && v.length > 2 || $t('v.min_3', v => !!v && v.length <= 30 || $t('v.max_30'))]"></v-text-field>
                </v-col>
              </v-row>
              <v-row>
                <v-col>
                  <v-text-field v-model="email" disabled label="Email" :rules="[v => !!v || $t('v.email_req'), v => !!v && v.length <= 150 || $t('v.max_email_150')]" prepend-inner-icon="mdi-email"></v-text-field>
                </v-col>
              </v-row>
              <v-row>
                <v-col sm="4" md="2">
                  <v-text-field v-model="country_code" placeholder="Country code" label="Country code"></v-text-field>
                </v-col>
                <v-col sm="8" md="10">
                  <v-text-field v-model="phone_no" placeholder="Phone number" label="Phone number" maxlength="10"></v-text-field>
                </v-col>
              </v-row>
              <v-row v-if="isUser">
                <v-col>
                  <hr>
                  <p class="h4">
                    Address
                  </p>
                  <UserAddress></UserAddress>
                </v-col>
              </v-row>
              <v-row>
                <v-col>
                  <v-btn color="primary" tile small type="submit" :loading="loading">
                    {{$t('save_changes')}}
                  </v-btn>
                </v-col>
              </v-row>
            </v-form>
          </a-tab-pane>
          <a-tab-pane key="3" tab="Security" force-render>
            <a-form-item>
              <p class="h4">
                {{$t('pwd_reset')}}
              </p>
              <div>
                <nuxt-link :to='localePath("change-my-password")'>
                  {{$t('change_pwd')}}
                </nuxt-link>
              </div>
            </a-form-item>
            <hr>
            <a-form-item v-if="isModerator">
              <p class="h4">
                PIN
              </p>
              <div>
                <nuxt-link :to='localePath("change-my-pin")'>
                  Change my PIN
                </nuxt-link>
              </div>
            </a-form-item>
          </a-tab-pane>
          <a-tab-pane v-if="isProfessional" key="4" tab="Professional Information" force-render>
            <div v-if='!isComplete'>
              <a-alert message='Please complete your profile information' :show-icon='true' type='warning' :banner='true'></a-alert>
            </div>
            <v-form ref="profForm" v-model="validProfForm" @submit.prevent='saveProfessional'>
              <v-row>
                <v-col md='6'>
                  <v-autocomplete v-model="category" label="Category" clearable :rules="[v => !!v || 'The category is required']" placeholder="Category" :items="categories" item-text="cate_category" item-value="cate_id"></v-autocomplete>
                </v-col>
                <v-col md='6'>
                  <v-text-field v-model="npi" placeholder="NPI" label="NPI" :rules="[v => !!v || 'The NPI is required', v => !!v && v.length === 10 || 'Enter exactly 10 characters']"></v-text-field>
                </v-col>
              </v-row>
              <v-row>
                <v-col md='6'>
                  <v-autocomplete v-model="specialty" label="Specialty" :items="specialties" item-text="spec_specialty" item-value="spec_id" clearable :loading="loadingSpec" placeholder="Specialty"></v-autocomplete>
                </v-col>
                <v-col md='6'>
                  <v-text-field v-model="practice_name" label="Practice Name" placeholder="Practice Name" :rules="[v => !!v || 'The practice name is required', v => !!v && v.length > 0 || 'Enter at least 1 character', value => !!value && value.length <=60 || 'Enter a maximum of 60 characters']">
                  </v-text-field>
                </v-col>
              </v-row>
              <v-row>
                <v-col md='6' lg="4">
                  <v-text-field v-model="medical_license" label="Medical License" placeholder="Medical license" :rules="[v => !!v || 'The Medical License is required', v => !!v && v.length >= 3 || 'Enter at least 3 characters', v => !!v && v.length <=12 || 'Enter a maximum of 12 characters']">
                  </v-text-field>
                </v-col>
                <v-col md='6' lg="4">
                  <v-text-field v-model="license_state" label="License State" placeholder="License State" :rules="[v => !!v || 'The License state is required', v => !!v && v.length > 1 || 'Enter at least 2 characters', v => !!v && v.length <= 30 || 'Enter a maximum of 30 characters']"></v-text-field>
                </v-col>
                <v-col md='6' lg="4">
                  <v-text-field v-model="credentials" label="Credentials" placeholder="Credentials" :rules="[v =>!!v || 'The credentials are required', v => !!v && v.length > 1 || 'Enter at least 2 characters', v=> !!v && v.length <= 30 || 'Enter a maximum of 30 characters']"></v-text-field>
                </v-col>
              </v-row>
              <hr class="mb-1">
              <p class="mb-1 h4">
                Practice Address
              </p>
              <UserAddress ref="profAddr"></UserAddress>
              <a-form-item>
                <a-button type="primary" html-type='submit'>
                  <SpinOrText v-model='loadingSaveP'>
                    {{$t('save_changes')}}
                  </SpinOrText>
                </a-button>
              </a-form-item>
            </v-form>
          </a-tab-pane>
        </a-tabs>
      </div>
    </v-col>
  </a-row>
  <request-modal ref='rmodal'></request-modal>
</div>
</template>

<script>
import userRoleMixin from "~/mixins/userRoleMixin";
import SpinOrText from '~/components/SpinOrText.vue'
import inputMixin from '~/mixins/inputMixin'
import authMixin from '~/mixins/authMixin'
import RequestModal from '~/components/RequestModal'
import ProfilePicture from '~/components/ProfilePicture'
import uploadMixin from '~/mixins/uploadMixin'
import UserAddress from "~/components/UserAddress";
import addressMixin from "~/mixins/addressMixin";

export default {
  name: "MyProfile",
  components: {
    UserAddress,
    ProfilePicture,
    SpinOrText,
    RequestModal,
  },
  mixins: [userRoleMixin, inputMixin, authMixin, uploadMixin, addressMixin],
  middleware: ['authenticated', 'not-blocked', 'not-deleted', 'verified', 'view-set'],
  data (){
    return {
      validPersonalForm: false,
      validProfForm: false,
      picture: '',
      file: null,
      showTabs: true,
      isComplete: false,
      tab: 1,
      form: this.$form.createForm(this, { name: 'user-data-form' }),
      profForm: null,
      loading: false,
      category: null,
      npi: '',
      categories: [],
      country_code: '',
      phone_no: '',
      loadingSaveP: false,
      loadingPage: true,
      isProfessional: false,
      loadingUpload: false,
      imageUrl: '',
      showUploadPicture: false,
      specialties: [],
      loadingSpec: false,
      specialty: null,
      practice_name: '',
      medical_license: '',
      license_state: '',
      credentials: '',
    }
  },
  head() {
    return {
      title: this.$t('my_profile'),
    }
  },
  computed: {

    user() {
      return this.$auth.user
    },
    first_name(){
      return this.user.user_first_name
    },
    last_name(){
      return this.user.last_name
    },
    name (){
      return this.user.user_first_name + ' ' + this.user.last_name
    },
    email (){
      return this.user.email
    }
  },
  watch: {
    categories(){
      this.specialty = null
    },
    category(v){
      if (v && this.isModerator){
        this.loadingSpec = true
        this.$api.get('/specialty/' + v).then(({data})=>{
          this.specialties = data
          if (this.specialties && this.specialties.length <= 0){
            this.$toast.error('No specialties found')
          }
        }).catch(()=>{
          this.$toast.error('Unable to get specialties')
        }).finally(() => {
          setTimeout(() => {
            this.loadingSpec = false
          }, 600)
        })
      }
    },
    zip(v){
      if (v){
        this.zip = this.numbersOnly(v)
      }
    },

  },
  mounted() {
    if (this.myUserId){
      this.$api.get('/user/'+ this.myUserId).then(({data})=>{
        if (data && data.user_uuid){
          this.phone_no = data.user_phone_no
          this.country_code = '+' + data.user_country_code
          this.picture = data.user_picture

          if (!this.$auth.user.has_phone){
            console.log('Dont have a phone no')
            this.showTabs = false
            this.tab = '2'
            setTimeout(() =>{
              this.$nextTick(()=>{
                this.showTabs = true
              })
            }, 200)
          }
        }
      })
    }
    this.$api.get('/category').then(({data})=>{
      if (data){
        this.categories = data
        console.log('Categories --->', data)
        this.getMyRecord()
      }
    }).catch(()=>{
      this.$toast.error('Failed to load categories.')
    })
  },
  methods: {
    uploadPicture(){
      this.loadingUpload = true
      const data = new FormData()

      data.append('file', this.file)

      this.$api.post('/user/picture', data).then(({data})=>{
        this.$toast.success('Your profile picture has been updated.')
        this.src = ''
        this.showUploadPicture = false
        this.picture = data.file

      }).catch((err)=>{
        this.$refs.rmodal.$emit('error',err)
      }).finally(() => {
        this.loadingUpload = false
      })

    },
    cancelUpload(){
      if (!this.loadingUpload){
        this.src = ''
        this.showUploadPicture = false
      }
    },
    handleChange(info) {
      if (info && info.file){
        this.getSrcInfo(info)
        this.file = info.file.originFileObj
      }
    },
    handleUpload() {
      const { fileList } = this
      const formData = new FormData()
      fileList.forEach(file => {
        formData.append('file', file)
      })
      formData.append('type', this.type)
      this.loadingUpload = true
      this.$api.post(this.computedAction, formData, {
        onUploadProgress: (evt) => {
          this.onProgress(evt)
        }
      }).then(() => {
        this.fileList = []
        this.$message.success(this.$t('upl_succ').toString())
        setTimeout(() => {
          this.handleRemove()
          this.umUploadProgress = 0
        }, 1000)

      }).catch(() => {
        this.$message.error(this.$t('upl_fail').toString())
      }).finally(() => {
        this.loadingUpload = false
      })
    },
    beforeUpload(file) {
      const isJpgOrPng = file.type === 'image/jpeg' || file.type === 'image/png';
      if (!isJpgOrPng) {
        this.$message.error('You can only upload JPG file!');
      }
      const isLt2M = file.size / 1024 / 1024 < 6;
      if (!isLt2M) {
        this.$message.error('Image must smaller than 6MB!');
      }
      return isJpgOrPng && isLt2M;
    },
    getMyRecord(){
      this.$api.get('/professional/my-record').then(({data})=>{
        if (data){
          this.isProfessional = this.isModerator
          this.category = data.profe_cate_id
          this.isComplete = data.profe_specialty && data.profe_practice_name && data.profe_medical_license && data.profe_license_state && data.profe_credentials

          if (!this.isComplete) {
            this.showTabs = false
            this.tab = 4
            setTimeout(() =>{
              this.$nextTick(()=>{
                this.showTabs = true
              })
            }, 100)
          }


          if (this.isModerator){
            this.npi = data.profe_npi
            this.practice_name = data.profe_practice_name
            this.medical_license = data.profe_medical_license
            this.license_state = data.profe_license_state
            this.credentials = data.profe_credentials
            if (data.profe_spec_id){
              this.specialty = data.profe_spec_id
            }
          }
          if (this.$refs.profAddr){
            this.$refs.profAddr.$emit('refresh')
          }
        }
      }).finally(() => {
        this.loadingPage = false
      })
    },
    handleSubmit(){

      this.$refs.personalForm.validate()
      if (this.validPersonalForm){
        this.loading = true
        this.$api
          .put('/user', {
            country_code: this.country_code,
            phone_no: this.phone_no,
            is_patient: this.isUser,
            line1: this.line1,
            city: this.city,
            state: this.state,
            zip: this.zip,
            first_name: this.first_name,
            last_name: this.last_name,
          })
          .then(() => {
            setTimeout(async () => {
              await this.$auth.fetchUser()
              console.log('Fetched user --->', this.$auth.user)
              this.$toast.success(this.$t('updated_suc').toString());
            }, 100)
          })
          .catch((e) => {
            this.$refs.rmodal.$emit('error', e)
          })
          .finally(() => {
            setTimeout(() => {
              this.loading = false
            }, 100)
          })
      }
    },
    saveProfessional(){
      this.$refs.profForm.validate()
      if (this.validProfForm){

        const values = {
          category: this.category,
          npi: this.npi,
          specialty: this.specialty,
          practice_name: this.practice_name,
          medical_license: this.medical_license,
          license_state: this.license_state,
          credentials: this.credentials,
          line1: this.line1,
          city: this.city,
          state: this.state,
          zip: this.zip
        }
        this.loadingSaveP = true
        this.$api.put('/professional', values).then(()=>{
          this.$toast.success(this.$t('updated_suc').toString());
        }).catch((e)=>{
          this.$refs.rmodal.$emit('error', e)
        }).finally(() => {
          this.loadingSaveP = false
        })
      }
    },
  }
}
</script>

<style scoped lang="sass">
  .user-role
    background-color: #444
    color: #fff
    border: 0
    border-radius: 0
    padding: 0.2rem 0.5rem
    display: inline-block
    text-transform: capitalize
    font-size: 15px !important
  .profile-upload
    display: flex
    flex-direction: row
    flex-wrap: wrap
    > div
      display: flex
      align-items: center
      justify-content: center
      padding: 0.9rem
    .upload-box
      display: flex
      justify-content: center
      align-items: center
      flex-direction: column
      .buttons
        display: flex
        width: 100%

  .profile-overlay
    opacity: 0
    color: #fff
    position: absolute
    left: 0
    top: 0
    bottom: 0
    width: 100%
    height: 100%
    display: flex
    justify-content: center
    align-items: center
    font-weight: bold
    font-size: 3rem
    &:hover
      cursor: pointer
      background: rgba(0,0,0,0.9)
      opacity: 1

  .avatar-uploader.ant-upload-picture-card-wrapper
    img
      max-width: 120px !important
</style>