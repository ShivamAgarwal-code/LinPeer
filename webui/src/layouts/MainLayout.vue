<template>
  <q-layout view='lHh Lpr lFf'>
    <q-header elevated class='header-gradient'>
      <q-toolbar>
        <div class='row items-center' :style='{width: "720px", margin: "0 auto"}'>
          <span class='logo-text cursor-pointer' @click='onLogoClick'>LinPeer</span>
          <q-space />
          <q-btn
            flat round icon='store' :color='tab == "store" ? "amber" : "white"'
            @click='onNFTMarketClick'
          />
          <q-btn
            v-if='!account?.length'
            flat round icon='login' color='white'
            @click='onLoginClick'
          />
          <q-btn
            flat round icon='local_activity' :color='tab == "activity" ? "amber" : "white"'
            @click='onActivityClick'
          />
          <q-btn
            v-if='account?.length'
            flat round icon='dashboard' :color='tab == "dashboard" ? "amber" : "white"'
            @click='onDashboardClick'
          />
          <q-btn
            v-if='account?.length'
            flat round icon='logout' color='white'
            @click='onLogoutClick'
          />
        </div>
      </q-toolbar>
    </q-header>

    <q-page-container>
      <router-view />
      <chain-query />
      <request-application />
      <credit-query />
      <block-subscription />
      <feed-contents-keys-query />
      <feed-contents-query />
      <market-info-query />
      <market-collections-query />
      <request-feed-subscribe />
      <request-review-subscribe />
      <request-credit-subscribe />
      <request-foundation-subscribe />
      <request-market-subscribe />
      <request-activity-subscribe />
      <user-reviewer-query />
      <content-applications-keys-query />
      <content-applications-query />
      <asset-applications-keys-query />
      <asset-applications-query />
      <reviewer-applications-keys-query />
      <reviewer-applications-query />
      <review-state-query />
      <foundation-info-query />
      <activities-keys-query />
      <activities-query />
      <activity-applications-keys-query />
      <activity-applications-query />
    </q-page-container>

    <q-footer elevated class='footer-gradient' :style='{height: "40px", lineHeight: "40px"}'>
      <div class='row items-center' :style='{width: "720px", margin: "0 auto"}'>
        <span class='footer-logo cursor-pointer' @click='onLogoClick'>LinPeer</span>
        <q-space />
        <span class='text-white text-caption'>Peer-to-Peer content publishing on Linera</span>
        <q-space />
        <q-img
          src='https://avatars.githubusercontent.com/u/107513858?s=48&v=4'
          width='24px'
          height='24px'
          :style='{marginLeft: "8px", marginTop: "4px"}'
          class='cursor-pointer'
          @click='onGithubClick("https://github.com/linera-io/linera-protocol.git")'
        />
        <q-img
          src='https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSAPxlNRQziXOi61fD4jtkxAm-v6pPbT_UIF5IL1_PqCQ&s=10'
          width='24px'
          height='24px'
          :style='{marginLeft: "8px", marginTop: "4px"}'
          class='cursor-pointer'

          @click='onGithubClick("https://github.com/web3eye-io/res-peer.git")'
        />
      </div>
    </q-footer>
    <q-dialog
      v-model='logining'
      @hide='onHide'
      position='standard'
    >
      <q-card :style='{padding: "32px", width: "100%"}'>
        <q-card-section>
          <div class='text-h5'>
            Login
          </div>
        </q-card-section>
        <q-card-section>
          <q-input :style='{width: "100%"}' label='Linera Address' v-model='account' />
        </q-card-section>
        <q-card-section>
          <q-btn label='Login' @click='onLoginConfirmClick' />
        </q-card-section>
      </q-card>
    </q-dialog>
  </q-layout>
</template>

<script setup lang="ts">
import { useRouter, useRoute } from 'vue-router'
import { onBeforeMount, onMounted, ref } from 'vue'
import { Cookies } from 'quasar'
import { useUserStore } from 'src/stores/user'
import * as constants from 'src/const'

import CreditQuery from 'src/components/CreditQuery.vue'
import BlockSubscription from 'src/components/BlockSubscription.vue'
import FeedContentsKeysQuery from 'src/components/FeedContentsKeysQuery.vue'
import FeedContentsQuery from 'src/components/FeedContentsQuery.vue'
import MarketInfoQuery from 'src/components/MarketInfoQuery.vue'
import MarketCollectionsQuery from 'src/components/MarketCollectionsQuery.vue'
import RequestApplication from 'src/components/RequestApplication.vue'
import RequestFeedSubscribe from 'src/components/RequestFeedSubscribe.vue'
import RequestCreditSubscribe from 'src/components/RequestCreditSubscribe.vue'
import RequestFoundationSubscribe from 'src/components/RequestFoundationSubscribe.vue'
import RequestMarketSubscribe from 'src/components/RequestMarketSubscribe.vue'
import RequestActivitySubscribe from 'src/components/RequestActivitySubscribe.vue'
import ChainQuery from 'src/components/ChainQuery.vue'
import UserReviewerQuery from 'src/components/UserReviewerQuery.vue'
import RequestReviewSubscribe from 'src/components/RequestReviewSubscribe.vue'
import ContentApplicationsKeysQuery from 'src/components/ContentApplicationsKeysQuery.vue'
import ContentApplicationsQuery from 'src/components/ContentApplicationsQuery.vue'
import AssetApplicationsKeysQuery from 'src/components/AssetApplicationsKeysQuery.vue'
import AssetApplicationsQuery from 'src/components/AssetApplicationsQuery.vue'
import ReviewerApplicationsKeysQuery from 'src/components/ReviewerApplicationsKeysQuery.vue'
import ReviewerApplicationsQuery from 'src/components/ReviewerApplicationsQuery.vue'
import ReviewStateQuery from 'src/components/ReviewStateQuery.vue'
import FoundationInfoQuery from 'src/components/FoundationInfoQuery.vue'
import ActivitiesKeysQuery from 'src/components/ActivitiesKeysQuery.vue'
import ActivitiesQuery from 'src/components/ActivitiesQuery.vue'
import ActivityApplicationsKeysQuery from 'src/components/ActivityApplicationsKeysQuery.vue'
import ActivityApplicationsQuery from 'src/components/ActivityApplicationsQuery.vue'

const router = useRouter()
const account = ref('')
const logining = ref(false)
const user = useUserStore()
const route = useRoute()
const tab = ref('feed')

interface Query {
  port: number
}

const port = ref((route.query as unknown as Query).port || constants.port)

const onGithubClick = (uri: string) => {
  window.open(uri)
}
const onDashboardClick = () => {
  tab.value = 'dashboard'
  void router.push({
    path: '/dashboard',
    query: {
      port: port.value
    }
  })
}
const onActivityClick = () => {
  tab.value = 'activity'
  void router.push({
    path: '/activities',
    query: {
      port: port.value
    }
  })
}
const onLogoClick = () => {
  tab.value = 'feed'
  void router.push({
    path: '/',
    query: {
      port: port.value
    }
  })
}
const onLoginClick = () => {
  logining.value = true
}
const onHide = () => {
  logining.value = false
}
const onLoginConfirmClick = () => {
  if (account.value.length === 0) {
    return
  }
  onHide()
  Cookies.set('account', account.value)
  user.account = account.value
}
onBeforeMount(() => {
  Cookies.set('service-port', port.value.toString())
})
onMounted(() => {
  account.value = Cookies.get('account')
  user.account = account.value
})

const onLogoutClick = () => {
  Cookies.remove('account')
  user.account = undefined as unknown as string
  account.value = undefined as unknown as string
}
const onNFTMarketClick = () => {
  tab.value = 'store'
  void router.push({
    path: '/market',
    query: {
      port: port.value
    }
  })
}
</script>

<style scoped lang="sass">
.header-gradient
  background: linear-gradient(135deg, #1a237e 0%, #4a148c 50%, #880e4f 100%)

.footer-gradient
  background: linear-gradient(135deg, #1a237e 0%, #4a148c 100%)

.logo-text
  font-family: 'Orbitron', 'Segoe UI', sans-serif
  font-size: 1.8rem
  font-weight: 700
  color: white
  letter-spacing: 2px
  text-shadow: 0 2px 10px rgba(255,255,255,0.3)

.footer-logo
  font-family: 'Orbitron', 'Segoe UI', sans-serif
  font-size: 1rem
  font-weight: 600
  color: white
  letter-spacing: 1px
</style>
