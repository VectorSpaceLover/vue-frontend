<template>
  <div class="pa-6 mh-100v">
    <a-row class='mb-1'>
      <a-col>
        <a-button @click='$router.push(localePath("/view-mode"))'>Go Back <a-icon type='arrow-left'></a-icon></a-button>
      </a-col>
    </a-row>
    <a-row v-if='loading'>
      <a-skeleton></a-skeleton>
    </a-row>
    <a-row v-else>
      <a-col>
        <p class='h4 mb-1'>Required Information</p>
      </a-col>
      <a-col v-if="error">
        <p class="h1">
          Error while loading the data.
        </p>
        <p>
          Please Try again later
        </p>
      </a-col>
      <a-col v-else>
        <a-form :form="form" @submit.prevent="handleSubmit">
          <a-row class='mb-1'>
            <a-col>
              <a-checkbox v-model='checked'>
                I would like to be a new provider at Mednoor.
              </a-checkbox>
            </a-col>
          </a-row>
          <a-row>
            <a-col :xs="24" :sm="24" :md="12">
              <a-form-item>
                <a-input v-decorator="['npi', {
                  rules: [
                    {required: true, message: 'The NPI is required'},
                    {min: 10, message: $t('v.min_10')},
                    {max: 10, message: $t('v.max_10')},
                  ]
                }]" placeholder="My NPI" :max-length="10"></a-input>
              </a-form-item>
            </a-col>
            <a-col :xs="24" :sm="24" :md="12">
              <a-form-item>
                <a-auto-complete v-model="category" :data-source="categories" placeholder="Enter a category">
                </a-auto-complete>
              </a-form-item>
            </a-col>
          </a-row>
          <a-form-item>
            <a-button type="primary" html-type="submit">
              <spin-or-text v-model="loadingSubmit">
                Join Providers
              </spin-or-text>
            </a-button>
          </a-form-item>
        </a-form>
      </a-col>
    </a-row>
    <RequestModal ref="rmodal"></RequestModal>
  </div>
</template>

<script>
import SpinOrText from "~/components/SpinOrText";
import RequestModal from '~/components/RequestModal'
export default {
  name: "JoinProfessionals",
  components: {SpinOrText, RequestModal},
  middleware: ['authenticated', 'verified', 'not-blocked', 'not-deleted'],
  data (){
    return {
      category: '',
      categories: [],
      form: this.$form.createForm(this),
      loadingSubmit: false,
      loading: true,
      error: false,
      checked: false,
    }
  },
  mounted() {
    this.$api.get('/professional/my-record').then(({data})=>{
      if (data && data.profe_uuid){
        console.log(data)
        if (data.profe_is_active){
          this.$router.push(this.localePath('/'))
        }else{
          this.$router.push(this.localePath('/thanks-for-applying'))
        }
      }
      this.$api.get('/category').then(({data})=>{
        if (data){
          this.categories = data.map((c)=>{
            return {
              value: c.cate_id.toString(),
              text: c.cate_category,
            }
          })
        }
      }).catch((err)=>{
        console.log(err)
        this.$toast.error('Failed to load categories.')
      })
    }).catch((err)=>{
      console.log(err)
      this.error = true
    }).finally(()=>{
      this.loading = false
    })
  },
  methods: {
    handleSubmit(){
      this.form.validateFields((err, values) => {
        if (err){
          console.log(err)
          return 0
        }
        const npi = values.npi

        if (isNaN(npi)){
          this.$toast.error('NPI must contain numbers only.')
          return 0
        }

        if (!this.checked){
          this.$toast.error('Please check the checkbox.')
          return 0
        }

        if (this.category){
          this.loadingSubmit = true
          this.$api.post('/professional', {
            npi: values.npi,
            category: this.category,
          }).then(() => {
            this.$router.push(this.localePath('/thanks-for-applying'))
            console.log('Thanks for applying.')
            // this.$toast.success('Thank you for your interest in being a Provider at Mednoor Medical Center. Mednoor credentialing department will contact you as soon as possible')
          }).catch((err) => {
            this.$refs.rmodal.$emit('error', err)
          }).finally(() => {
            this.loadingSubmit = false
          })
        }else{
          this.$toast.error('Select a category')
        }
      })
    }
  }
}
</script>

<style scoped>

</style>
