<template>
  <div class='pa-6 mh-100v'>
    <v-row v-if='loading'>
      <v-col>
        <v-skeleton-loader></v-skeleton-loader>
      </v-col>
    </v-row>
    <v-row v-else-if='not_found'>
      <v-col>
        <p class='h1'>Not Found</p>
        <p>
          We could not find the user that you were looking for.
        </p>
        <a-button @click="$router.push(localePath('/professionals'))">
          Search other users
        </a-button>
      </v-col>
    </v-row>
    <v-row v-else>
      <v-col md="12">
        <p class='h1 flex-line align-f-md'>
          {{ user.user_first_name }} {{ user.user_last_name }}
          <v-btn v-if='user && user.usro_key' color="purple" dark small class='ml-1' rounded>
            {{ user.usro_role }}
            <v-icon v-if="isThisAModerator" class="ml-1">mdi-check-decagram</v-icon>
          </v-btn>
        </p>
      </v-col>
      <v-col md='12'>
        <div>
          <ProfilePicture :user='user' class="mb-1"></ProfilePicture>
          <div v-if="isThisAModerator" class="flex-line">
            <add-my-doctors :uuid="user.user_uuid" class="mr-1"></add-my-doctors>
            <MakeAppointment :user="user" button></MakeAppointment>
          </div>
          <v-divider class="mb-1"></v-divider>
        </div>
        <v-row class='mt-1 mb-1'>
          <v-col md="6">
            <p class='mb-0'>
              <b>Phone Number:</b>
              <span v-if='user && user.user_country_code'>
            +{{ user.user_country_code }} {{ user.user_phone_no }}
            </span>
              <span v-else>
              No phone number was provided.
            </span>
            </p>
            <p class="mb-0">
              <b>Email:</b>
              <span v-if='user && user.user_email'>
              {{ user.user_email }}
            </span>
            </p>
            <p v-if="isThisAModerator">
              <b>Practice Address: </b> {{display_address(address)}}
            </p>
            <div>
              <v-chip
                v-if="user.cate_category"
                color="primary"
              >
                {{user.cate_category}}
              </v-chip>
              <v-chip
                v-if="user.spec_specialty"
                color="success"
              >
                {{user.spec_specialty}}
              </v-chip>
            </div>
          </v-col>
        </v-row>
        <v-row v-if='isUserModerator' class='mt-1'>
          <p v-if='isLoggedIn'>
          </p>
          <a-button v-else type='primary' class='mt-1' @click="$router.push({
            path: localePath('/sign-in'),
            query: {
              callback: encodeURIComponent('/user/' + uuid)
            }
          })">
            Sign in to chat with {{ user.user_first_name }}
          </a-button>
        </v-row>
      </v-col>
    </v-row>
    <RequestModal ref='rmodal'></RequestModal>
  </div>
</template>

<script>
import RequestModal from '~/components/RequestModal'
import authMixin from '~/mixins/authMixin'
import dateMixin from '~/mixins/dateMixin'
import ProfilePicture from '~/components/ProfilePicture'
import MakeAppointment from "~/components/MakeAppointment";
import AddMyDoctors from "~/components/AddMyDoctors";
import addressDisplayMixin from "~/mixins/addressDisplayMixin";

export default {
  name: 'Uuid',
  components: {
    AddMyDoctors,
    MakeAppointment,
    ProfilePicture,
    RequestModal
  },
  mixins: [authMixin, dateMixin, addressDisplayMixin],
  data: () => ({
    loading: true,
    not_found: false,
    user: {},
    columns: [
      {
        title: 'Category',
        dataIndex: 'category',
        key: 'category',
        slots: { title: 'Category' }
      },
      {
        title: 'Specialty',
        dataIndex: 'spec_specialty',
        key: 'spec_specialty',
        slots: { title: 'Specialty' },
        scopedSlots: { customRender: 'spec_specialty' }
      },
    ],
    data: [],
    uuid: null,
    address: {},
  }),
  computed: {
    isThisAModerator(){
      return this.user && this.user.usro_role === 'MODERATOR'
    },
    exists() {
      return this.user && this.user.usro_key
    },
    role() {
      if (this.exists) {
        return this.user.usro_key
      } else {
        return ''
      }
    },
    isUserModerator() {
      return this.role === 'MODERATOR'
    }
  },
  mounted() {
    const q = this.$route.params
    if (q && q.uuid) {
      this.uuid = q.uuid

      this.$api.get('/user/practice-address/' + q.uuid).then(({data})=>{
        this.address = data
      })

      this.$api.get('/user/' + q.uuid).then(({ data }) => {
        this.loading = false
        if (data && data.user_uuid) {
          this.user = data
        } else {
          this.not_found = false
        }
      }).catch(() => {
        this.not_found = true
      }).finally(() => {
        this.loading = false
      })
    }
  },
  methods: {

  }
}
</script>

<style scoped lang='sass'>

</style>
