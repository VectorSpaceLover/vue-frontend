<template>
  <a-row class='pa-6 mh-100v'>
    <a-col>
      <div v-if='doctors && doctors.length' class='mb-1'>
        <p class='h3 text-center'>
          My Doctors
        </p>
        <div class='mb-1 my-doctors'>
          <ProfessionalThumb v-for='(p, i) in doctors' :key='i' class='doctor' :user='p'></ProfessionalThumb>
        </div>
      </div>
      <div class='mb-1'>
        <v-row>
          <v-col>
            <p class='h2 text-center'>Find Doctors</p>
          </v-col>
        </v-row>
        <v-row>
          <v-col  md="6" offset-md="3">
            <a-form-item>
              <a-input-search v-model='term' size='large' placeholder='Search by Professional Name' enter-button
                              @search='search'>
              </a-input-search>
            </a-form-item>
          </v-col>
        </v-row>
      </div>
      <div>
        <a-skeleton v-if='loadingResults'/>
        <div v-else-if="results.length > 0">
          <section v-for="(r, i) in results" :key="i" class="professional-bar">
            <div>
              <professional-thumb :user="r" :show-make-appointment="false"></professional-thumb>
            </div>
            <div>
              <div class="w-100">
                <h3>{{ r.user_last_name }}, {{ r.user_first_name }} {{ r.profe_credentials }}</h3>
              </div>
              <div v-if="r.addr_line1" class="w-100 mb-1">
                <p>
                  {{display_address(r)}}
                </p>
                <p>
                  <v-chip
                    v-if="r.category"
                    color="primary"
                  >
                    {{ r.category }}
                  </v-chip>
                  <v-chip
                    v-if="r.specialty"
                    color="success"
                  >
                    {{ r.specialty }}
                  </v-chip>
                  <v-divider vertical></v-divider>
                </p>
              </div>
              <div class="flex-line">
                <add-my-doctors :uuid="r.uuid" class="mr-1"></add-my-doctors>
                <MakeAppointment :user="r" :super-small="false" button></MakeAppointment>
              </div>

            </div>
          </section>
        </div>
        <div v-else>
          <p class="text-center">
            {{ $t('no_data_av') }}
          </p>
        </div>
      </div>
    </a-col>
    <a-col>
      <RequestModal ref='rmodal'></RequestModal>
    </a-col>
    <a-col>
      <a-modal
        :title="$t('conf_action')"
        :visible='visible'
        :confirm-loading='confirmLoading'
        @ok='handleOk'
        @cancel='handleCancel'
      >
        <p>{{ $t('conf_send_ch') }}</p>
      </a-modal>
    </a-col>
  </a-row>
</template>

<script>
import RequestModal from '~/components/RequestModal'
import authMixin from '~/mixins/authMixin'
import ProfessionalThumb from '~/components/ProfessionalThumb'
import MakeAppointment from '~/components/MakeAppointment'
import AddMyDoctors from "~/components/AddMyDoctors";
import addressDisplayMixin from "~/mixins/addressDisplayMixin";


export default {
  name: 'Professionals',
  components: {
    ProfessionalThumb,
    RequestModal,
    MakeAppointment,
    AddMyDoctors
  },
  mixins: [authMixin, addressDisplayMixin],
  middleware: ['verified'],
  data() {
    return {
      term: '',
      visible: false,
      columns: [
        {
          title: this.$t('full_name'),
          dataIndex: 'user_first_name',
          key: 'user_first_name',
          slots: {title: this.$t('full_name')},
          scopedSlots: {customRender: 'full_name'}
        },
        {
          title: this.$t('action'),
          key: 'action',
          scopedSlots: {customRender: 'action'}
        }
      ],
      users: [],
      loading: true,
      confirmLoading: false,
      uuid: '',
      action: '',
      socket: null,
      doctors: [],
      loadingResults: false,
      results: []
    }
  },
  head() {
    return {
      title: this.$t('profs')
    }
  },
  mounted() {
    this.search()
    this.getModerators()
    this.socket = this.$nuxtSocket({})
    this.socket.emit('join-room', this.myUserId)
    this.socket.on('chat-allowed', () => {
      this.getModerators()
    })



    if (this.isLoggedIn) {
      this.$api.get('/my-doctor', {
        params: {
          five: true
        }
      }).then(({data}) => {
        this.doctors = data
      })
    }
  },
  methods: {
    search() {
      this.loadingResults = true
      this.$api.get('/user/search', {
        params: {
          searchTerm: this.term,
          type: 'MODERATOR',
          categories: this.selectedCategories
        }
      }).then(({data}) => {
        if (data) {
          this.results = data
        }
        this.searched = true
      }).catch((err) => {
        this.$toast.error(err)
      }).finally(() => {
        this.loadingResults = false
      })
    },
    getModerators() {
      this.loading = true
    },
    handleCancel() {
      this.visible = false
      this.uuid = null
      this.confirmLoading = false
    },
    handleOk() {
      this.confirmLoading = true
      if (this.action === 'save') {
        this.$api.post('/my-professional', {
          professional: this.uuid
        }).then(() => {
          this.getModerators()
        }).catch((err) => {
          this.$refs.rmodal.$emit('error', err)
        }).finally(() => {
          this.confirmLoading = false
          this.visible = false
        })
      }
    },
    save(uuid) {
      this.uuid = uuid
      this.action = 'save'
      this.visible = true
    },
    remove(uuid) {
      this.uuid = uuid
      this.action = 'remove'
      this.visible = true
    }
  }
}
</script>

<style scoped lang='sass'>

.view-more-box
  display: flex
  width: 130px
  height: 130px
  justify-content: center
  align-items: center
  border: 1px solid #ccc
  padding: 1rem
  color: $mdn-primary

  &:hover
    cursor: pointer

.my-doctors
  display: flex
  flex-wrap: wrap
  justify-content: center
  align-items: center
  padding: 0.3rem
  flex-direction: row
  width: 100%

  .doctor
    display: flex

.professional-bar
  background: #fff
  border: 1px solid #ccc
  margin: 1rem
  display: flex
  flex-direction: column

  > div
    text-align: center

  > div:not(:first-child)
    padding: 1rem

@media screen and (min-width: $md)
  .professional-bar
    flex-direction: row

    > div
      text-align: left
  .view-more-box
    width: 133px
    height: 133px


@media screen and (min-width: $lg)
  .view-more-box
    width: 150px
    height: 150px
</style>
