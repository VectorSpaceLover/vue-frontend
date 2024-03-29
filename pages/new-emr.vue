<template>
  <div class='pa-6 mh-100v'>
    <a-row class='mb-1'>
      <a-col>
        <a-breadcrumb>
          <a-breadcrumb-item>
            <nuxt-link :to="localePath('/dashboard')">{{$t('dashboard')}}</nuxt-link>
          </a-breadcrumb-item>
          <a-breadcrumb-item>
            <nuxt-link :to="localePath('/emr')">{{$t('emr')}}</nuxt-link>
          </a-breadcrumb-item>
          <a-breadcrumb-item>
                      <span v-if='isTemplate'>
                          {{$t('emr_template')}}
                      </span>
            <span v-else>
                          Encounter
                      </span>
          </a-breadcrumb-item>
        </a-breadcrumb>
      </a-col>
    </a-row>
    <div v-if='loadingData'>
      <a-skeleton />
    </div>
    <a-form v-else :form='form' @submit='handleSubmit'>
      <a-row>
        <a-col :xs='24'>
          <p class='h4 mb-1'>
                    <span v-if='isTemplate'>
                        <span v-if='recordId'>{{$t('update_template')}}</span>
                        <span v-else>{{$t('create_template')}}</span>
                    </span>
            <span v-else>
                        <span v-if='recordId'>{{$t('update_record')}}</span>
                        <span v-else>New Encounter</span>
                    </span>
          </p>
        </a-col>
      </a-row>
      <a-row>
        <a-col :xs='24' :sm='24'>
          <a-form-item>
            <a-input v-if='isTemplate'
                     v-decorator="[
                                'template_name',
                                { rules: [
                                    { required: true, message: $t('v.en_tmp_name') },
                                    { min: 2, message: $t('v.min_2')}
                                ] },
                            ]"
                     type='text'
                     :placeholder="$t('template_name')"
                     @input="inputReceived('template_name')">
            </a-input>
            <a-auto-complete v-else v-model='userSearch' :data-source='usersList' placeholder="Search patient"
                             :disabled='!!recordId && !draft'>
              <a-icon slot='suffix' type='search' style='color:rgba(0,0,0,.25)' />
            </a-auto-complete>
          </a-form-item>
        </a-col>
        <a-col :xs='24' :sm='24' :md='12'>
          <a-form-item>
            <a-date-picker v-model='date' :disabled="isDisabled" format='MM-DD-YYYY' @change="onChange"></a-date-picker>
          </a-form-item>
        </a-col>
        <a-col v-if='!isTemplate' :xs='24' :sm='24' :md='12'>
          <a-form-item>
            <a-auto-complete :disabled="isDisabled" :data-source='templatesList' :placeholder="$t('template_name')" allow-clear
                             @change='changeTemplate' />
          </a-form-item>
        </a-col>
      </a-row>
      <a-row>
        <a-col :xs='24'>
          <a-tabs default-active-key='1'>
            <a-tab-pane key='1' :tab="$t('allergies')" force-render>
              <a-form-item>
                <a-textarea v-decorator="['allergies', { rules: [] }]" :placeholder="$t('allergies')" :rows='6' :disabled='isDisabled' @input="inputReceived('allergies')" />
              </a-form-item>
            </a-tab-pane>
            <a-tab-pane key='2' :tab="$t('current_meds')" force-render>
              <a-form-item>
                <a-textarea v-decorator="['current_meds', { rules: [] }]" :placeholder="$t('current_meds')" :rows='6'  :disabled='isDisabled' @input="inputReceived('current_meds')" />
              </a-form-item>
            </a-tab-pane>
            <a-tab-pane key='3' :tab="$t('med_htry')" force-render>
              <a-form-item>
                <a-textarea
                  v-decorator="['medical_history', { rules: [] }]"
                  :placeholder="$t('med_htry')" :rows='6' :disabled='isDisabled' @input="inputReceived('medical_history')" />
              </a-form-item>
            </a-tab-pane>
            <a-tab-pane key='4' :tab="$t('soc_htry')" force-render @input="inputReceived">
              <a-form-item>
                <a-textarea
                  v-decorator="['social_history', { rules: [] }]"
                  :placeholder="$t('soc_htry')" :rows='6' :disabled='isDisabled' @input="inputReceived('social_history')" />
              </a-form-item>
            </a-tab-pane>
            <a-tab-pane key='5' :tab="$t('fam_hry')" force-render>
              <a-form-item>
                <a-textarea
                  v-decorator="['family_history', { rules: [] }]"
                  :placeholder="$t('fam_hry')" :rows='6' :disabled='isDisabled' @input="inputReceived('family_history')" />
              </a-form-item>
            </a-tab-pane>
          </a-tabs>
        </a-col>
      </a-row>
      <a-row class='mt-1 mb-1'>
        <a-col :xs='24'>
          <div class='emr-inline-inputs'>
            <a-form-item
               :label="$t('bp')">
              <a-input
                v-model='bp'
                :placeholder="$t('bp')" :disabled='isDisabled' />
            </a-form-item>
            <a-form-item
              :label="$t('pulse')">
              <a-input
                v-decorator="['pulse', { rules: [{ max: 10, message: $t('v.max_10') }] }]"
                :placeholder="$t('pulse')" :disabled='isDisabled' @input="inputReceived('pulse')" />
            </a-form-item>
            <a-form-item
              :label="$t('resp_rate')">
              <a-input
                v-decorator="['resp_rate', { rules: [{ max: 10, message: $t('v.max_10') }] }]"
                :placeholder="$t('resp_rate')" :disabled='isDisabled' @input="inputReceived('resp_rate')" />
            </a-form-item>
            <a-form-item
              :label="$t('temp')">
              <a-input
                v-decorator="['temp', { rules: [{ max: 10, message: $t('v.max_10') }] }]"
                :placeholder="$t('temp')" :disabled='isDisabled' @input="inputReceived('temp')" />
            </a-form-item>
            <a-form-item
              :label="$t('height_in')">
              <a-input v-decorator="['height', {}]" :placeholder="$t('height_in')" :disabled='isDisabled'
                       @input='updateBMI("height")' />
            </a-form-item>
            <a-form-item
              :label="$t('weight_lb')">
              <a-input v-decorator="['weight', {}]" :placeholder="$t('weight_lb')" :disabled='isDisabled'
                       @input='updateBMI("weight")' />
            </a-form-item>
            <a-form-item
              :label="$t('bmi')">
              <a-input v-model='bmi' :placeholder="$t('bmi')" disabled />
            </a-form-item>
          </div>
        </a-col>
      </a-row>
      <a-row>
        <a-col :xs='24'>
          <a-tabs default-active-key='1'>
            <a-tab-pane key='1' :tab="$t('chief_complaint')" force-render>
              <a-form-item>
                <a-textarea
                  v-decorator="['chief_complaint', { rules: [] }]"
                  :placeholder="$t('chief_complaint')" :rows='6' :disabled='isDisabled' @input="inputReceived('chief_complaint')" />
              </a-form-item>
            </a-tab-pane>
            <a-tab-pane key='2' :tab="$t('hpi')" force-render>
              <a-form-item>
                <a-textarea
                  v-decorator="['hip', { rules: [] }]"
                  :placeholder="$t('hpi')" :rows='6' :disabled='isDisabled' @input="inputReceived('hip')" />
              </a-form-item>
            </a-tab-pane>
            <a-tab-pane key='3' :tab="$t('subject')" force-render>
              <a-form-item>
                <a-textarea
                  v-decorator="['subject', { rules: [] }]"
                  :placeholder="$t('subject')" :rows='6' :disabled='isDisabled' @input="inputReceived('subject')" />
              </a-form-item>
            </a-tab-pane>
            <a-tab-pane key='4' :tab="$t('objective')" force-render @input="inputReceived">
              <a-form-item>
                <a-textarea
                  v-decorator="['objective', { rules: [] }]"
                  :placeholder="$t('objective')" :rows='6' :disabled='isDisabled' @input="inputReceived('objective')" />
              </a-form-item>
            </a-tab-pane>
            <a-tab-pane key='5' :tab="$t('assessment')" force-render>
              <a-form-item>
                <a-textarea
                  v-decorator="['assessment', { rules: [] }]"
                  :placeholder="$t('assessment')" :rows='6' :disabled='isDisabled' @input="inputReceived('assessment')" />
              </a-form-item>
            </a-tab-pane>
            <a-tab-pane key='6' :tab="$t('plan')" force-render>
              <a-form-item>
                <a-textarea
                  v-decorator="['plan', { rules: [] }]"
                  :placeholder="$t('plan')" :rows='6' :disabled='isDisabled' @input="inputReceived('plan')" />
              </a-form-item>
            </a-tab-pane>
            <a-tab-pane key='7' :tab="$t('sign')" force-render>
              <a-form-item>
                <a-textarea
                  v-decorator="['sign', { rules: [] }]"
                  :placeholder="$t('sign')" :rows='6' :disabled='isDisabled' @input="inputReceived('sign')" />
              </a-form-item>
            </a-tab-pane>
            <a-tab-pane key='8' :tab="$t('addendum')" force-render>
              <a-form-item>
                <a-textarea
                  v-decorator="['addendum', { rules: [] }]"
                  :placeholder="$t('addendum')" :rows='6' :disabled="locked" @input="inputReceived('addendum')" />
              </a-form-item>
            </a-tab-pane>
          </a-tabs>
        </a-col>
      </a-row>
      <a-row v-if="!locked" class='mt-1'>
        <a-col>
          <a-button type='primary' html-type='submit'>
            <SpinOrText v-model='loading'>
                        <span v-if='isTemplate'>
                            <span v-if='recordId'>{{$t('update_template')}}</span>
                            <span v-else>{{$t('create_template')}}</span>
                        </span>
              <span v-else>
                            <span v-if='recordId'>{{$t('update_record')}}</span>
                            <span v-else>{{$t('chat_record')}}</span>
                        </span>
            </SpinOrText>
          </a-button>
          <a-button v-if='!isTemplate' type='old-rose' @click='printRecord'>
            <SpinOrText v-model='loadingPdf'>
              <a-icon type='file-pdf'></a-icon>
              {{$t('print')}}
            </SpinOrText>
          </a-button>
        </a-col>
      </a-row>
      <a-row v-else class="mt-1">
        <a-col>
          <a-button type='success' @click.prevent="$router.push(localePath('/emr'))">
            Go Back <a-icon type="arrow-left"></a-icon>
          </a-button>
        </a-col>
      </a-row>
      <RequestModal ref='rmodal' />
      <a ref='pdfDownload' :href='pdfFile' :download='pdfFile' class='d-none' target='_blank'></a>
    </a-form>
  </div>

</template>
<script>
import RequestModal from '~/components/RequestModal.vue'
import SpinOrText from '~/components/SpinOrText.vue'
import formMixin from '~/mixins/formMixin'
const debounce = require('lodash.debounce')
const { validate } = require('uuid');


export default {
  name: 'NewEmr',
  components: {
    RequestModal,
    SpinOrText
  },
  mixins: [formMixin],
  layout: 'dashboard',
  middleware: ['authenticated', 'not-blocked', 'not-deleted', 'verified', 'pin-set'],
  data() {
    return {
      locked: false,
      loadingData: false,
      loading: false,
      templates: [], // Full data of the templates
      templatesList: [], // key and value
      usersList: [],
      userSearch: '',
      recordId: null,
      bmi: 0,
      isTemplateD: false,
      loadingPdf: false,
      pdfFile: '',
      bp: '',
      draft: null,
      canSaveDraft: false,
    }
  },
  head(){
    return {
      title: 'New Encounter'
    }
  },
  computed: {
    isDisabled() {
      // return !!this.recordId && this.type === 'record' || this.recordId && !this.isTemplate
      return this.locked
    },
    type() {
      let type = 'new'
      const q = this.$route.query
      if (q) {
        if (q.type && q.type.toLowerCase() === 'template') {
          type = 'template'
        }
      }
      return type
    },
    isTemplate() {
      return this.type === 'template' || this.isTemplateD
    }
  },
  watch: {
    bp(v) {
      let nv = ''
      if (v){
        v.split('').forEach((k, i)=>{
          if (this.allowed.includes(k) || (i === 3 && k === '/')){
            nv += k
          }
        })
      }
      this.bp = nv
      if (nv.length === 4){
        this.bp = this.addSlash(this.bp)
      }
      if (this.bp.length > 7){
        this.bp = this.bp.substr(0, 7)
      }
      console.log(this.bp)
      this.saveDraft('bp')
    },
    userSearch: debounce(function(v) {
      if (validate(v)){
        // The professional selected a valid user.
        this.saveDraft('user')
      }
      if (v && v.length > 0) {
        this.$api.get('/user/search', {
          params: {
            searchTerm: v
          }
        }).then(({ data }) => {
          if (data && data.length) {
            this.usersList = data.map((user) => {
              return {
                text: user.full_name,
                value: user.uuid
              }
            })
          }
        }).catch(() => {
          this.$toast.error(this.$t('unable_search').toString())
        })
      } else {
        this.usersList = []
      }
    }, 600)
  },
  mounted() {
    const c = this.$route.query
    if (c.mere) {
      this.recordId = c.mere
      this.loadingData = true
      this.$api.get('/medical-record/' + this.recordId).then(({ data }) => {
        this.setData(data)
        if (data.mrus_user_uuid) {
          this.usersList = [
            {
              text: data.user_first_name + ' ' + data.user_last_name,
              value: data.user_uuid
            }
          ]
          this.userSearch = data.user_uuid
        }
        if (data && data.mrus_mere_uuid) {
          this.$route.query.type = 'record'
          this.isTemplateD = false
        } else {
          this.$route.query.type = 'template'
          this.isTemplateD = true
        }
        this.updateBMI()
        setTimeout(() => {
          this.canSaveDraft = true
        }, 1000)
      }).catch(() => {
        this.recordId = null
        this.$toast.error(this.$t('could_nf_rec').toString())
      }).finally(() => {
        this.loadingData = false
      })
    }else{
      this.canSaveDraft = true
    }
    this.$api.get('/medical-record', {
      params: {
        type: 'template'
      }
    }).then(({ data }) => {
      this.templates = data
      this.templatesList = data.map((tmp) => {
        return {
          text: tmp.mere_name,
          value: tmp.mere_uuid
        }
      })
    })
    if (c.mode && c.mode ==='view'){
      this.locked = true
    }
  },
  methods: {
    inputReceived: debounce ( function (arg) {
      this.saveDraft(arg)
    }, 1000),
    saveDraft(arg, valueOverride){
      if (this.canSaveDraft) {


        console.log('Save draft. Update -->', arg)
        const v = this.form.getFieldsValue()[arg] || this[arg]

        console.log(`The new value of ${arg} is ${v}`)
        console.log('Draft', this.draft)
        console.log('RecordId', this.recordId)
        if (this.draft === null && !this.recordId) {
          // type ---> new || template
          this.$api.post('/medical-record/draft', {
            type: this.type,
            arg,
          }).then(({data}) => {
            if (data && data.mere_uuid) {
              this.recordId = data.mere_uuid
              this.$toast.success('A new draft has been created')
              this.draft = true
              this.putData(arg, v || valueOverride)
            }
          }).catch(() => {
            this.$toast.error('Failed to create a new draft. You can continue but your changes won\'t be saved automatically.')
          })
        }else if (validate(this.recordId)){
          this.putData(arg, v || valueOverride)
        }
      }
    },
    putData(name, value){

      const values = this.form.getFieldsValue()
      let v = value || values[name]
      if (!v){
        if (name === 'user'){
          v = this.userSearch
        }
      }
      name = name.replace(/_/g, '-')
      this.$api.patch('/medical-record/' + name + '/' + this.recordId, {
        value: v
      }).then(({ data }) => {
        console.log(`Updated ${name} successfully`)
      }).catch(() => {
        this.$toast.error(`Failed to update ${name}`)
      })
    },
    onChange(v){
      console.log('The date has changed. ', v)
      this.saveDraft('date', this.date.format('YYYY-MM-DD'))
    },
    printRecord() {
      if (this.recordId) {
        this.loadingPdf = true
        this.$api.get('/medical-record/pdf/' + this.recordId).then(({ data }) => {
          this.$toast.success(this.$t('pdf_gend').toString())
          this.pdfFile = process.env.API_URL + '/generated/record/' + data.file
          setTimeout(() => {
            this.$refs.pdfDownload.click()
          }, 500)
        }).catch(() => {
          this.$toast.error(this.$t('pdf_fail').toString())
        }).finally(() => {
          this.loadingPdf = false
        })
      } else {
        this.$toast.success(this.$t('record_hnb_cy').toString())
      }
    },
    isValidUser() {
      let r = false
      this.usersList.forEach((user) => {
        if (user.value === this.userSearch) {
          r = true
        }
      })
      return r
    },
    updateBMI(arg) {
      const d = this.form.getFieldsValue()
      let w = d.weight
      let h = d.height
      let r = 0
      if (w && h) {
        // [weight (lb) / height (in) / height (in)] x 703
        w = parseFloat(w)
        h = parseFloat(h)
        r = (w / h / h) * 703
      }
      this.bmi = r.toFixed(2)
      this.saveDraft(arg)
    },
    handleSubmit(e) {
      e.preventDefault()
      this.form.validateFields((err, values) => {
        if (err) {
          return console.log(err)
        }
        const keys = Object.keys(values)
        const nv = {}
        keys.forEach((key) => {
          nv[key] = values[key] || ''
        })
        nv.bp = this.bp

        if (!this.date || !this.date.isValid()){
          return this.$toast.error('Please enter a valid date')
        }

        nv.date = this.date.format('YYYY-MM-DD')

        this.loading = true

        if (this.recordId) {
          nv.patient = this.userSearch
          if (!this.isTemplate && !this.isValidUser()) {
            this.loading = false
            this.$toast.error(this.$t('please_select_user').toString())
            return false
          }


          this.$api.put('/medical-record/' + this.recordId, nv).then(() => {
            this.form.resetFields()
            this.$toast.success(this.$t('record_hb_updated').toString())
            setTimeout(() => {
              this.$router.push(this.localePath('/emr'))
            }, 500)
          }).catch((err) => {
            this.$refs.rmodal.$emit('error', err)
          }).finally(() => {
            this.loading = false
          })
        } else if (this.isTemplate) {
          this.$api.post('/medical-record', nv).then(() => {
            this.form.resetFields()
            this.$toast.success(this.$t('template_hb_created').toString())
            setTimeout(() => {
              this.$router.push(this.localePath('/emr'))
            }, 500)
          }).catch((err) => {
            this.$refs.rmodal.$emit('error', err)
          }).finally(() => {
            this.loading = false
          })
        } else {
          nv.patient = this.userSearch
          if (this.isValidUser()) {
            this.$api.post('/medical-record/record', nv).then(() => {
              this.form.resetFields()
              this.$toast.success(this.$t('record_hb_created').toString())
              setTimeout(() => {
                this.$router.push(this.localePath('/emr'))
              }, 500)
            }).catch((err) => {
              this.$refs.rmodal.$emit('error', err)
            }).finally(() => {
              this.loading = false
            })
          } else {
            this.loading = false
            this.$toast.error(this.$t('please_select_user').toString())
          }
        }
      })
    },
    setData(tm) {
      this.$nextTick(() => {
        setTimeout(() => {
          this.form.setFieldsValue({
            template_name: tm.mere_name,
            allergies: tm.mere_allergies,
            current_meds: tm.mere_current_meds,
            medical_history: tm.mere_medical_history,
            social_history: tm.mere_social_history,
            family_history: tm.mere_family_history,
            bp: tm.mere_bp,
            pulse: tm.mere_pulse,
            resp_rate: tm.mere_resp_rate,
            temp: tm.mere_temp,
            height: tm.mere_height,
            weight: tm.mere_weight,
            chief_complaint: tm.mere_chief_complaint,
            hip: tm.mere_hpi,
            subject: tm.mere_subject,
            objective: tm.mere_objective,
            assessment: tm.mere_assessment,
            plan: tm.mere_plan,
            sign: tm.mere_sign,
            addendum: tm.mere_addendum
          })
        }, 600)
      })
    },
    changeTemplate(t) {
      this.templates.forEach((tm) => {
        if (tm.mere_uuid === t) {
          this.setData(tm)
        }
      })
    }
  }

}
</script>
<style scoped lang='sass'>
.emr-inline-inputs
  .ant-row.ant-form-item
    width: 100% !important
    display: block

@media screen and (min-width: $md)
  .emr-inline-inputs
    display: flex
    .ant-row.ant-form-item
      width: 50% !important
</style>
