<template>
  <div
    style="min-height:100%"
    class=" q-pa-md column no-wrap justify-between bg-bg1 round-borders shadow-4 bg-logo animate-fade"
  >
    <div class="full-width">
      <div class="q-mb-md q-title relative-position">
        <div class="q-py-sm proposal-title-line">
          <span class="capitalize">{{ wp.title }}</span>
          <span class="q-caption on-right text-weight-thin"
            >({{ $t(`${getCategoryNameFromId(wp.category)}`) }})</span
          >
        </div>
        <q-btn
          v-if="!expanded"
          dense
          class="absolute-top-right text-text2"
          icon="fullscreen"
          flat
          @click="$emit('wp_expand', array_index)"
        />
        <q-btn
          v-if="expanded"
          dense
          class="absolute-top-right text-text2"
          icon="fullscreen_exit"
          flat
          @click="$emit('wp_compress')"
        />
      </div>

      <div class="row items-center relative-position q-mb-md">
        <profile-pic :accountname="wp.proposer" />
        <q-item>
          <q-item-main>
            <q-item-tile label>Proposer</q-item-tile>
            <q-item-tile sublabel>{{ wp.proposer }}</q-item-tile>
          </q-item-main>
        </q-item>
        <q-item>
          <q-item-main>
            <q-item-tile label>Requested Pay</q-item-tile>
            <q-item-tile sublabel>{{ wp.pay_amount.quantity }}</q-item-tile>
          </q-item-main>
        </q-item>
        <q-item>
          <q-item-main>
            <q-item-tile label>Arbitrator</q-item-tile>
            <q-item-tile sublabel>{{ wp.arbitrator }}</q-item-tile>
          </q-item-main>
        </q-item>
        <q-item v-if="expanded">
          <q-item-main>
            <q-item-tile label>Submitted</q-item-tile>
            <q-item-tile sublabel>date</q-item-tile>
          </q-item-main>
        </q-item>
        <q-item>
          <q-item-main>
            <q-item-tile label>Status (dev)</q-item-tile>
            <q-item-tile sublabel>{{ wp.status }}</q-item-tile>
          </q-item-main>
        </q-item>
      </div>

      <div
        class="q-mb-md"
        v-if="wp.status == 0 || wp.status == 2 || wp.status == 5"
      >
        <div class="q-caption q-mb-xs text-text2">Time Left</div>
        <!-- {{ getExpiry }} -->
        <countdown
          v-if="getExpiry.millisleft"
          :time="Number(getExpiry.millisleft)"
        >
          <template slot-scope="props">
            <div class="q-caption text-weight-light q-mb-xs">
              <span v-if="props.days">{{ props.days }} days, </span>
              <span v-if="props.hours">{{ props.hours }} hours, </span>
              <span v-if="props.minutes">{{ props.minutes }} minutes, </span>
              <span>{{ props.seconds }} seconds</span>
            </div>
          </template>
        </countdown>
        <div v-else class="q-caption text-weight-light q-mb-xs text-negative">
          expired
        </div>
        <q-progress
          :percentage="getExpiry.percent"
          color="primary-light"
          style="height: 4px"
        />
      </div>

      <q-scroll-area
        class="bg-bg2  q-mt-sm round-borders text-weight-light text-text2"
        :style="scroll_area_style"
        color="primary"
        :thumb-style="{
          right: '0px',
          background: '#7c41ba',
          width: '8px',
          opacity: 1
        }"
        :delay="1500"
      >
        <MarkdownViewer
          :tags="[
            'h1',
            'h2',
            'h3',
            'italic',
            'bold',
            'underline',
            'strikethrough',
            'subscript',
            'superscript',
            'anchor',
            'orderedlist',
            'unorderedlist'
          ]"
          :dark="getIsDark"
          :text="wp.summary"
        />
      </q-scroll-area>

      <div
        v-if="expanded"
        class="row justify-between q-mt-xs items-center q-body-1"
      >
        <span class="text-text2">ID {{ wp.id }}</span>
        <a
          target="_blank"
          :href="$configFile.get('explorer') + `/transaction/${wp.trx_id}`"
          >{{ $helper.truncate(wp.trx_id, 10) }}</a
        >
      </div>
    </div>

    <div class="q-mt-md full-width">
      <div class="row justify-between items-center">
        <div class="row" v-show="wp.status !== 5 && wp.status !== 100">
          <q-item
            @click.native="expand_votes_modal = true"
            class="cursor-pointer no-padding"
          >
            <q-item-main>
              <q-item-tile class="q-caption">Vote Threshold</q-item-tile>
              <q-item-tile class="q-title">
                {{ getVotingScore }}
              </q-item-tile>
            </q-item-main>
          </q-item>
        </div>

        <div v-if="!read_only && getAccountName" class="row">
          <member-select
            v-if="wp.status == 0 || wp.status == 2"
            :show_selected="false"
            @change="delegatevote($event)"
            :value="MyDirectDelegation || ''"
            :accountnames="getCustNames"
            placeholder="Select to Delegate"
            :underline="false"
            label="Delegation"
            ref="directDelSelect"
          />
          <div v-if="wp.status == 0">
            <q-btn
              v-if="getVoterStatus == 2 || getVoterStatus == 0"
              class="on-right animate-pop"
              color="positive"
              label="Approve"
              @click="voteprop('voteApprove')"
            />
            <q-btn
              v-if="getVoterStatus == 1 || getVoterStatus == 0"
              class="on-right animate-pop"
              color="negative"
              label="Deny"
              @click="voteprop('voteDeny')"
            />
          </div>
          <div v-else-if="wp.status == 2">
            <q-btn
              v-if="getVoterStatus == 4 || getVoterStatus == 0"
              class="on-right animate-pop"
              color="positive"
              label="Approve Claim"
              @click="voteprop('claimApprove')"
            />
            <q-btn
              v-if="getVoterStatus == 3 || getVoterStatus == 0"
              class="on-right animate-pop"
              color="negative"
              label="Deny Claim"
              @click="voteprop('claimDeny')"
            />
            <q-btn
              v-if="getIsArbitrator"
              class="on-right animate-pop"
              flat
              color="positive"
              label="arb approve"
              @click="arbApprove()"
            />
          </div>

          <div v-else-if="wp.status == 1">
            Work is in progress
          </div>
        </div>
        <div v-if="getIsCreator">
          <q-btn
            v-if="wp.status == 3"
            class="on-right animate-pop"
            color="info"
            label="Start work"
            @click="startWork()"
          />
          <q-btn
            v-if="wp.status == 1"
            class="on-right animate-pop"
            color="info"
            label="Complete work"
            @click="completeWork()"
          />
          <q-btn
            v-if="wp.status == 4"
            class="on-right animate-pop"
            color="info"
            label="claim"
            @click="finalize()"
          />
          <q-btn
            class="on-right animate-pop"
            flat
            color="negative"
            label="Cancel"
            @click="cancelProp()"
          />
        </div>
        <!-- <q-btn label="reload dev" @click="actionCallBack(wp.id)" /> -->
      </div>
    </div>

    <q-modal minimized v-model="expand_votes_modal">
      <div class="bg-bg2 text-text1 bg-logo">
        <!-- header -->
        <div
          style="height:50px"
          class="bg-bg1 row items-center justify-between q-px-md"
        >
          <span>Votes</span>
          <q-btn icon="close" @click="expand_votes_modal = false" flat dense />
        </div>
        <!-- content -->
        <div class="q-pa-md" v-if="wp.status !== 5 && wp.status !== 100">
          <div class="row justify-start q-mt-sm">
            <div
              class="row items-center relative-position bg-bg1 round-borders q-pr-md q-ma-sm"
              v-for="(vote, i) in getVotes.filter(
                v => v.vote === 1 || v.vote === 3
              )"
              :key="i + 'p'"
            >
              <profile-pic
                :accountname="vote.voter"
                :scale="0.5"
                :show_role="false"
              />
              <router-link class="a2" :to="{ path: '/profile/' + vote.voter }">
                <div class="q-ma-none" style="min-width:100px; overflow:hidden">
                  {{ vote.voter }}
                </div>
              </router-link>
              <q-chip
                v-if="vote.weight > 1"
                color="bg2"
                :title="`voteweight ${vote.weight}`"
                >{{ vote.weight }}</q-chip
              >
              <q-icon
                class="absolute"
                style="top:-5px; right:-10px"
                color="positive"
                :name="$configFile.icon.check"
                size="24px"
              />
            </div>

            <div
              class="row items-center relative-position bg-bg1 round-borders q-pr-md q-ma-sm"
              v-for="(vote, i) in getVotes.filter(
                v => v.vote === 2 || v.vote === 4
              )"
              :key="i + 'r'"
            >
              <profile-pic
                :accountname="vote.voter"
                :scale="0.5"
                :show_role="false"
              />
              <router-link class=" a2" :to="{ path: '/profile/' + vote.voter }">
                <div class="q-ma-none" style="min-width:100px; overflow:hidden">
                  {{ vote.voter }}
                </div>
              </router-link>
              <q-chip
                v-if="vote.weight > 1"
                color="bg2"
                :title="`voteweight ${vote.weight}`"
                >{{ vote.weight }}</q-chip
              >
              <q-icon
                class="absolute"
                style="top:-5px; right:-10px"
                color="negative"
                name="clear"
                size="24px"
              />
            </div>
          </div>
          <pre>{{ getVotes }}</pre>
        </div>
      </div>
    </q-modal>
  </div>
</template>

<script>
import { mapGetters } from "vuex";
import profilePic from "components/ui/profile-pic";
import MarkdownViewer from "components/ui/markdown-viewer";
import memberSelect from "components/controls/member-select";
import wpcats from "../../extensions/statics/config/wp_categories.json";
import countdown from "@chenfengyuan/vue-countdown";

export default {
  name: "wpProposal",
  components: {
    profilePic,
    memberSelect,
    MarkdownViewer,
    countdown
  },
  props: {
    array_index: null,
    read_only: false,
    wp: {
      type: Object,
      default: () => {
        return {};
      }
    },
    expanded: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      wp_expiration: 100,
      expand_votes_modal: false
    };
  },
  computed: {
    ...mapGetters({
      getAccountName: "user/getAccountName",
      getWpConfig: "dac/getWpConfig",
      getIsDark: "ui/getIsDark",
      getAuth: "user/getAuth",
      getCustodians: "dac/getCustodians"
    }),

    getVotes() {
      if (this.wp.votes && this.wp.votes.length) {
        if (
          this.wp.status === 0 ||
          this.wp.status === 1 ||
          this.wp.status === 3
        ) {
          return this.wp.votes.filter(
            v => v.vote == 1 || v.vote == 2 || v.vote === null
          );
        }
        if (
          this.wp.status === 2 ||
          this.wp.status === 4 ||
          this.wp.status === 101
        ) {
          return this.wp.votes.filter(
            v => v.vote == 3 || v.vote == 4 || v.vote === null
          );
        }
      } else {
        return [];
      }
    },

    MyDirectDelegation() {
      if (!this.getVotes.length) return false;

      let myvote = this.getVotes.find(
        v => v.voter == this.getAccountName && v.vote === null
      );
      if (myvote) {
        return myvote.delegatee;
      }

      let my_direct_delegatee = false;
      for (let i = 0; i < this.getVotes.length; i++) {
        let vote = this.getVotes[i];
        if (vote.delegates && vote.delegates.length) {
          let check = vote.delegates.find(
            d => d.voter == this.getAccountName && d.delegate_type == "direct"
          );
          if (check) {
            my_direct_delegatee = vote.voter;
            break;
          }
        }
      }
      return my_direct_delegatee;
    },

    scroll_area_style() {
      if (this.expanded) {
        return { height: "400px", width: "100%" };
      } else {
        return { height: "200px", width: "100%" };
      }
    },

    getIsCreator() {
      return this.getAccountName === this.wp.proposer;
    },
    getIsArbitrator() {
      return this.getAccountName === this.wp.arbitrator;
    },
    //get vote type of logged in user
    getVoterStatus() {
      let myvote = this.getVotes.find(v => v.voter == this.getAccountName);
      if (!myvote) {
        return 0;
      } else {
        if (myvote.vote === null) {
          return 0;
        } else {
          return myvote.vote;
        }
      }
    },

    getExpiry() {
      let expiration_millis;
      let start;
      if (this.wp.status === 0) {
        expiration_millis = Number(this.getWpConfig.approval_expiry) * 1000;
        start = Date.parse(this.wp.propose_timestamp);
        console.log("state 0", expiration_millis, start);
      }
      if (this.wp.status === 2) {
        expiration_millis = Number(this.getWpConfig.escrow_expiry) * 1000;
        start = Date.parse(this.wp.complete_work_timestamp);
        console.log("state 0", expiration_millis, start);
      }

      let end = start + expiration_millis;
      let current = new Date().getTime();
      //calculate relative expiration based on NOW and expiration
      let perc = 100 - ((current - start) / (end - start)) * 100;
      let msleft = end - current;

      if (this.wp.status === 5) {
        perc = 0;
        msleft = 0;
      }

      return {
        percent: perc <= 0 ? 0 : perc,
        millisleft: msleft <= 0 ? 0 : msleft
      };
    },
    getVotingScore() {
      let score = { score: 0, threshold: null };
      if (
        this.wp.status === 0 ||
        this.wp.status === 3 ||
        this.wp.status === 1
      ) {
        score.threshold = this.getWpConfig.proposal_threshold;
        this.getVotes.forEach(v => {
          if (v.vote === 1) score.score += v.weight;
        });
      }
      if (
        this.wp.status === 2 ||
        this.wp.status === 4 ||
        this.wp.status === 101
      ) {
        score.threshold = this.getWpConfig.finalize_threshold;
        this.getVotes.forEach(v => {
          if (v.vote === 3) score.score += v.weight;
        });
      }
      return `${score.score}/${score.threshold}`;
    },
    getCustNames() {
      if (this.getCustodians) {
        return this.getCustodians
          .map(c => {
            return c.cust_name;
          })
          .filter(c => c != this.getAccountName);
      } else {
        return [];
      }
    }
  },

  methods: {
    getCategoryNameFromId(id) {
      let wpc = wpcats.find(wpc => wpc.value == id);
      return wpc.label;
    },
    async delegatevote(delegatee) {
      console.log(delegatee);
      let actions = [
        {
          account: this.$configFile.get("wpcontract"),
          name: "delegatevote",
          //todo get permission from account instead hardcoding it
          authorization: [
            { actor: this.getAccountName, permission: this.getAuth },
            {
              actor: this.$configFile.get("authaccount"),
              permission: "one"
            }
          ],
          data: {
            custodian: this.getAccountName,
            proposal_id: Number(this.wp.id),
            delegatee_custodian: delegatee.new,
            dac_id: this.$configFile.get("dacscope")
          }
        }
      ];
      let result = await this.$store.dispatch("user/transact", {
        actions: actions
      });
      if (result) {
        this.actionCallBack(this.wp.id);
        //check if delegatee has voted
        // let vote = this.wp.votes.find(v => v.voter == delegatee.new);
        // if (!vote) return;

        // let deltemplate = {
        //   voter: this.getAccountName,
        //   delegate_type: "direct"
        // };
        // vote.delegates.push(deltemplate);
        // // if (!vote.delegates.find(vd => vd.voter == this.getAccountName)) {
        // //   vote.delegates.push(deltemplate);
        // // }
        // console.log(result);
      } else {
        this.$refs.directDelSelect.selected = delegatee.old;
      }
    },
    async actionCallBack(id) {
      await new Promise(resolve => setTimeout(resolve, 2000));
      let res = await this.$store.dispatch("dac/fetchWorkerProposals", {
        id: id
      });
      if (res) {
        this.wp.votes = res.results[0].votes;
      }
      console.log(res);
    },

    async voteprop(votetype) {
      const map = {
        voteApprove: 1,
        voteDeny: 2,
        claimApprove: 3,
        claimDeny: 4
      };

      let actions = [
        {
          account: this.$configFile.get("wpcontract"),
          name: "voteprop",
          authorization: [
            { actor: this.getAccountName, permission: this.getAuth },
            {
              actor: this.$configFile.get("authaccount"),
              permission: "one"
            }
          ],
          data: {
            custodian: this.getAccountName,
            proposal_id: Number(this.wp.id),
            vote: map[votetype],
            dac_id: this.$configFile.get("dacscope")
          }
        }
      ];

      let result = await this.$store.dispatch("user/transact", {
        actions: actions
      });
      if (result) {
        await this.actionCallBack(this.wp.id);
        this.$refs.directDelSelect.selected = "";
        // let vote = this.wp.votes.find(v => v.voter == this.getAccountName);
        // if (vote) {
        //   vote.vote = map[votetype];
        //   vote.delegatee = null;
        // } else {
        //   this.wp.votes.push({
        //     proposal_id: Number(this.wp.id),
        //     voter: this.getAccountName,
        //     delegatee: null,
        //     vote: map[votetype],
        //     comment_hash: ""
        //   });
        // }
        // console.log(result);
      }
    },
    async cancelProp() {
      let actions = [
        {
          account: this.$configFile.get("wpcontract"),
          name: "cancel",
          authorization: [
            { actor: this.getAccountName, permission: this.getAuth },
            {
              actor: this.$configFile.get("authaccount"),
              permission: "one"
            }
          ],
          data: {
            proposal_id: Number(this.wp.id),
            dac_id: this.$configFile.get("dacscope")
          }
        }
      ];

      let result = await this.$store.dispatch("user/transact", {
        actions: actions
      });
      if (result) {
        this.$emit("delete");
        console.log(result);
      }
    },
    async startWork() {
      let actions = [
        {
          account: this.$configFile.get("wpcontract"),
          name: "startwork",
          // authorization: [ {actor: this.getAccountName, permission: 'active'}],
          data: {
            proposal_id: Number(this.wp.id),
            dac_id: this.$configFile.get("dacscope")
          }
        }
      ];

      let result = await this.$store.dispatch("user/transact", {
        actions: actions
      });
      if (result) {
        this.$emit("delete");
      }
    },
    async completeWork() {
      let actions = [
        {
          account: this.$configFile.get("wpcontract"),
          name: "completework",
          // authorization: [ {actor: this.getAccountName, permission: 'active'}],
          data: {
            proposal_id: Number(this.wp.id),
            dac_id: this.$configFile.get("dacscope")
          }
        }
      ];

      let result = await this.$store.dispatch("user/transact", {
        actions: actions
      });
      if (result) {
        this.$emit("delete");
        console.log(result);
      }
    },

    async arbApprove() {
      let actions = [
        {
          account: this.$configFile.get("wpcontract"),
          name: "arbapprove",
          data: {
            arbitrator: this.getAccountName,
            proposal_id: Number(this.wp.id),
            dac_id: this.$configFile.get("dacscope")
          }
        },
        {
          account: this.$configFile.get("escrowcontract"),
          name: "approve",

          data: {
            key: Number(this.wp.id),
            approver: this.getAccountName
          }
        }
      ];

      let result = await this.$store.dispatch("user/transact", {
        actions: actions
      });
      if (result) {
        console.log(result);
      }
    },

    async finalize() {
      let actions = [
        {
          account: this.$configFile.get("wpcontract"),
          name: "finalize",
          // authorization: [ {actor: this.getAccountName, permission: 'active'}],
          data: {
            proposal_id: Number(this.wp.id),
            dac_id: this.$configFile.get("dacscope")
          }
        }
      ];

      let result = await this.$store.dispatch("user/transact", {
        actions: actions
      });
      if (result) {
        console.log(result);
      }
    }
  }
};

// ProposalStatePending_approval = 0,
// ProposalStateWork_in_progress = 1,
// ProposalStatePending_finalize = 2,
// ProposalStateHas_enough_approvals_votes = 3,
// ProposalStateHas_enough_finalize_votes = 4,
// ProposalStateExpired = 5 -->

// enum VoteType {
//             none = 0,
//             // a vote type to indicate a custodian's approval of a worker proposal.
//            1= proposal_approve,
//             // a vote type to indicate a custodian's denial of a worker proposal.
//             2=proposal_deny,
//             // a vote type to indicate a custodian's acceptance of a worker proposal as completed.
//            3= claim_approve,
//             // a vote type to indicate a custodian's rejection of a worker proposal as completed.
//            4= claim_deny
//         };

// The values only support integers on the input into the contract. Inside the values are cast to Doubles for the percentage calculation and then assert for >= to required perecentage
// eg. double(approved_count) / double(approved_count + deny_count) * 100.0
</script>

<style lang="stylus">
 @import '~variables'

.proposal-title-line{
  border-bottom: 1px solid var(--q-color-text2)
}
</style>
