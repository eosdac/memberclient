<template>
  <q-page>
    <div class="q-pa-md">
      <div class="row gutter-sm">
        <div class="col-xs-12 col-md-6">
          <div class="bg-bg1 round-borders shadow-4 q-pa-md">
            <div class="q-title q-mb-md">Profile Cache</div>
            <div class="row">
              <q-btn
                class="q-ma-xs"
                label="log cache"
                color="primary"
                @click="logProfileCache"
              />
              <q-btn
                class="q-ma-xs"
                label="empty cache"
                color="primary"
                @click="emptyProfileCache"
              />
            </div>
          </div>
        </div>

        <div class="col-xs-12 col-md-6">
          <div class="bg-bg1 round-borders shadow-4 q-pa-md">
            <div class="q-title q-mb-md">Config File</div>
            <div class="row">
              <q-btn
                class="q-ma-xs"
                label="show configfile"
                color="primary"
                @click="logConfigFile"
              />
            </div>
          </div>
        </div>

        <div class="col-xs-12 col-md-6 ">
          <div class="bg-bg1 round-borders shadow-4 q-pa-md">
            <color-picker />
          </div>
        </div>

        <div class="col-xs-12 col-md-6 ">
          <div class="bg-bg1 round-borders shadow-4 q-pa-md full-height">
            <div class="q-title q-mb-md">Network</div>
            <div class="q-mt-md row">
              <network-switcher class="q-ma-xs" :enable_pin="false" />
              <firehose class="q-ma-xs" />
            </div>
          </div>
        </div>
      </div>

      <debug-data
        :data="[
          {
            getIsCustodian: getIsCustodian,
            getActiveNetwork: getActiveNetwork,
            getMemberStatus: getMemberStatus,
            getAgreedTermsVersion: getAgreedTermsVersion,
            getMemberTerms: getMemberTerms
          }
        ]"
      />

      <q-modal maximized v-model="showConfigModal">
        <div
          style="height:50px"
          class="bg-bg1 row items-center justify-between q-px-md text-text1"
        >
          <span>Config</span>
          <q-btn icon="close" @click="showConfigModal = false" flat dense />
        </div>
        <div class="q-pa-md bg-bg2 text-text1">
          <debug-data :data="[$configFile.configFile]" show />
        </div>
      </q-modal>
    </div>
    <!-- end wrapper -->
  </q-page>
</template>

<script>
import { mapGetters } from "vuex";

import colorPicker from "components/controls/color-picker";
import networkSwitcher from "components/controls/network-switcher";
import firehose from "components/controls/firehose";
import debugData from "components/ui/debug-data";

export default {
  name: "test",
  components: {
    colorPicker,
    networkSwitcher,
    firehose,
    debugData
  },
  data() {
    return {
      scatter: null,
      assettest: "",
      showConfigModal: false
    };
  },
  computed: {
    ...mapGetters({
      getActiveNetwork: "global/getActiveNetwork",
      getAgreedTermsVersion: "user/getAgreedTermsVersion",
      getDacBalance: "user/getDacBalance",
      getMemberStatus: "user/getMemberStatus",
      getIsDark: "ui/getIsDark",
      getMemberTerms: "dac/getMemberTerms",
      getIsCustodian: "user/getIsCustodian",
      getDacApi: "global/getDacApi"
      // getCustodians: 'dac/getCustodians',
    })
  },

  methods: {
    logConfigFile() {
      console.log(this.$configFile.configFile);
      this.showConfigModal = true;
    },
    logProfileCache() {
      console.log(this.$profiles);
    },
    emptyProfileCache() {
      this.$profiles.delete();
    },
    switchNode() {
      this.$store.commit("global/setNode", "http://testerino.com");
    }
  }
};
</script>
