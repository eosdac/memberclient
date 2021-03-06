<template>
  <div class="row gutter-sm">
    <div class="col-xs-12">
      <div>
        <q-field
          class="q-mb-sm"
          :error="$v.form.title.$error"
          error-label="Title is required and can't be longer then 230 chars"
        >
          <q-input
            stack-label="Title"
            v-model="form.title"
            :dark="getIsDark"
            @input="$v.form.title.$touch()"
          />
        </q-field>
      </div>
    </div>

    <div class="col-xs-12">
      <div>
        <q-field
          class="q-mb-sm"
          :error="$v.form.description.$error"
          error-label="Description is required and can't be longer then 900 chars"
        >
          <!-- <MarkdownViewer
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
            :edit="true"
            :dark="getIsDark"
            :text="form.description"
            v-on:update="updateFormDescription"
          /> -->

          <q-input
            class="no-padding"
            type="textarea"
            stack-label="Description"
            v-model="form.description"
            :dark="getIsDark"
            @input="$v.form.description.$touch()"
          />
        </q-field>
      </div>
    </div>

    <div class="col-xs-12 col-lg-6">
      <div>
        <q-field
          class="q-mb-sm"
          :error="$v.form.from.$error"
          error-label="Select a from account"
        >
          <!-- <q-select
            class="no-padding"
            v-model="form.from"
            stack-label="From"
            color="primary-light"
            :dark="getIsDark"
            :options="
              from_accounts.map(fa => {
                return { value: fa, label: fa };
              })
            "
          /> -->
          <q-input
            class="no-padding"
            v-model="form.from"
            stack-label="From"
            :dark="getIsDark"
            color="primary-light"
          >
            <q-autocomplete
              :min-characters="0"
              :static-data="{
                field: 'value',
                list: from_accounts.map(fa => {
                  return { value: fa, label: fa };
                })
              }"
            />
          </q-input>
        </q-field>
      </div>
    </div>

    <div class="col-xs-12 col-lg-6">
      <div>
        <q-field
          class="q-mb-sm"
          :error="$v.form.to.$error"
          error-label="Please enter a valid accountname"
        >
          <q-input
            class="no-padding"
            stack-label="To"
            v-model.trim="form.to"
            :dark="getIsDark"
            @input="$v.form.to.$touch()"
          >
            <q-autocomplete
              :dark="getIsDark"
              @search="suggestAccounts_to"
              :min-characters="3"
              :delay="0"
              :max-results="7"
            />
          </q-input>
        </q-field>
      </div>
    </div>

    <div class="col-xs-12 col-lg-6">
      <div>
        <q-field
          class="q-mb-sm"
          :error="$v.form.asset.amount.$error"
          error-label="Please enter a valid pay amount"
        >
          <div class="row no-wrap items-end">
            <q-input
              class="full-width no-padding"
              type="number"
              :decimals="form.asset.precision"
              stack-label="Quantity"
              v-model="form.asset.amount"
              :dark="getIsDark"
              @input="$v.form.asset.amount.$touch()"
            />

            <q-select
              v-if="!tokens_loading"
              class="no-padding q-ml-xs animate-fade"
              hide-underline
              filter
              autofocus-filter
              :value="selected_token"
              color="primary-light"
              :dark="getIsDark"
              placeholder="token"
              :options="
                tokens.map(t => {
                  return {
                    image: t.logo || '',
                    value: t,
                    label: t.symbol,
                    sublabel: t.contract,
                    stamp: `${t.precision}`,
                    rightTextColor: 'blue'
                  };
                })
              "
              @change="handleTokenSelection"
            />

            <q-spinner v-if="tokens_loading" color="primary-light" />
          </div>
        </q-field>
      </div>
    </div>

    <div class="col-xs-12 col-lg-6">
      <div>
        <q-field
          class="no-padding q-mb-sm"
          :error="$v.form.memo.$error"
          error-label="Memo can't be longer then 255 chars."
        >
          <q-input
            stack-label="Memo"
            v-model.trim="form.memo"
            :dark="getIsDark"
            class="no-padding"
          />
        </q-field>
      </div>
    </div>

    <div class="col-xs-12">
      <div class="row justify-end">
        <q-btn color="primary" label="add" @click="processInputs" />
      </div>
    </div>
  </div>
</template>

<script>
import { mapGetters } from "vuex";
// import MarkdownViewer from "components/ui/markdown-viewer";
import { required, maxLength } from "vuelidate/lib/validators";
import { isEosName } from "../../modules/validators.js";
export default {
  name: "msigTransfer",
  components: {
    // MarkdownViewer
  },
  props: {
    symbol: {
      type: String,
      default: "EOS"
    },
    decimals: {
      type: Number,
      default: 4
    },
    from_accounts: {
      type: Array,
      default: () => {
        return [];
      }
    }
  },
  data() {
    return {
      form: {
        from: "",
        to: "",
        asset: {
          symbol: "EOS",
          precision: 4,
          contract: "eosio.token",
          amount: ""
        },
        memo: "",
        title: "",
        description: ""
      },
      selected_token: "",
      tokens: [],
      tokens_loading: false,
      default_assets: [
        {
          symbol: this.$configFile.get("systemtokensymbol"),
          precision: 4,
          contract: this.$configFile.get("systemtokencontract")
        },
        {
          symbol: this.$configFile.get("dactokensymbol"),
          precision: 4,
          contract: this.$configFile.get("tokencontract")
        }
      ]
    };
  },
  computed: {
    ...mapGetters({
      getIsDark: "ui/getIsDark",
      getAccountName: "user/getAccountName",
      getAccount: "user/getAccount",
      getDacApi: "global/getDacApi"
    }),
    getSelectedToken() {
      return this.selected_token;
    }
  },
  methods: {
    updateFormDescription(val) {
      this.form.description = val;
    },
    processInputs() {
      this.$v.$touch();
      if (this.$v.$error) {
        return;
      }
      let formdata = JSON.parse(JSON.stringify(this.form));
      formdata.asset.amount = parseFloat(formdata.asset.amount).toFixed(
        formdata.asset.precision
      );
      this.$emit("onsubmit", formdata);
      this.clearForm();
    },
    clearForm() {
      this.form = {
        from: "",
        to: "",
        asset: {
          symbol: "EOS",
          precision: 4,
          contract: "eosio.token",
          amount: ""
        },
        memo: "",
        title: "",
        description: ""
      };
      this.$v.$reset();
      this.edit_flag = false;
    },
    setFormFieldsEdit(data) {
      //make clone
      let cloned = JSON.parse(JSON.stringify(data));
      this.form = cloned;
    },

    async suggestAccounts_to(terms, done) {
      let accounts = await this.getDacApi.eos.get_table_rows({
        json: true,
        code: "eosio",
        scope: "eosio",
        table: "voters",
        lower_bound: terms,
        limit: 7
      });
      accounts = accounts.rows.map(a => {
        return { label: a.owner, value: a.owner, icon: "person" };
      });
      done(accounts);
    },

    async setTokens() {
      if (!this.form.from) return;
      this.tokens_loading = true;
      let tokens;
      try {
        tokens = await this.$axios.get(
          `${this.$configFile.get(
            "memberclientstateapi"
          )}/tokens_owned?account=${this.form.from}`
        );
        tokens = tokens.data.results;
      } catch (e) {
        console.log(e);
        tokens = [];
      }

      if (!tokens.length) {
        tokens = JSON.parse(JSON.stringify(this.default_assets));
      }

      this.tokens = tokens;

      this.selected_token = this.tokens.find(
        t =>
          t.symbol == this.form.asset.symbol &&
          t.contract == this.form.asset.contract
      );
      this.tokens_loading = false;
    },
    handleTokenSelection(v) {
      this.selected_token = v;
      Object.assign(this.form.asset, v);
    }
  },
  mounted() {
    this.setTokens();
  },
  watch: {
    "form.from": function() {
      this.setTokens();
    }
  },

  validations() {
    return {
      form: {
        from: { required, isEosName },
        to: { required, isEosName },
        asset: {
          amount: { required },
          symbol: { required }
        },
        memo: { maxLength: maxLength(255) },
        title: { required, maxLength: maxLength(230) },
        description: { required, maxLength: maxLength(900) }
      }
    };
  }
};
</script>

<style>
.q-item-image {
  min-height: 40px;
  height: 40px;
  min-width: 40px;
  width: 40px;
  border-radius: 50%;
  object-fit: cover;
}
</style>
