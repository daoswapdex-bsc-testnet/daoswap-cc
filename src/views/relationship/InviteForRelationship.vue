<template>
  <div class="fill-height">
    <v-container v-if="web3 && connected" class="fill-height">
      <v-row justify="center">
        <v-col md="6">
          <v-card class="fill-width">
            <v-card outlined>
              <v-card-title>
                <v-avatar size="24" class="mr-2">
                  <img :src="require('@/assets/logo.png')" alt="DAO" />
                </v-avatar>
                <span class="title font-weight-light">
                  {{ $t("Invite") }}
                </span>
              </v-card-title>
              <v-divider></v-divider>
              <v-card-text v-if="chainId === 56">
                <v-row align="center" v-if="accountAssets.isInvited">
                  <v-col class="display-3" cols="12">
                    <v-card-text>
                      <v-row align="center">
                        <v-col class="display-3" cols="12">
                          {{ $t("You have accepted the invitation") }}
                        </v-col>
                      </v-row>
                    </v-card-text>
                  </v-col>
                </v-row>
                <form v-else>
                  <v-card-title>
                    <span class="headline">{{
                      $t("Please enter your mentor's address")
                    }}</span>
                  </v-card-title>
                  <v-card-text>
                    <v-text-field
                      :label="$t('Mentor\'s address')"
                      v-model="inviterAccount"
                      :error-messages="inviterAccountErrors"
                      required
                      @input="setAccountAddress($event.target.value)"
                      @blur="setAccountAddress($event.target.value)"
                      @keyup="setAccountAddress($event.target.value)"
                      :autofocus="inviterAccountFocus"
                    ></v-text-field>
                  </v-card-text>
                  <v-card-actions class="justify-center">
                    <v-btn
                      large
                      color="#93B954"
                      dark
                      width="80%"
                      :disabled="!submitLoading"
                      @click="submit"
                    >
                      {{ $t("Accept") }}
                    </v-btn>
                  </v-card-actions>
                </form>
              </v-card-text>
              <v-card-text v-else>
                <v-row align="center">
                  <v-col class="display-3" cols="12">
                    <v-card-text>
                      <v-row align="center">
                        <v-col class="display-3" cols="12">
                          {{ $t("Please change BNB Smart Chain") }}
                        </v-col>
                      </v-row>
                    </v-card-text>
                  </v-col>
                </v-row>
              </v-card-text>
            </v-card>
          </v-card>
          <v-card justify="center" class="fill-width mt-10">
            <v-card-title>
              <span class="title font-weight-light">
                {{ $t("Current Token Address") }}
              </span>
            </v-card-title>
            <v-divider></v-divider>
            <v-card-text>
              <v-row align="center">
                <v-col
                  class="body-1"
                  cols="12"
                  @click="handleCopy(address, $event)"
                >
                  <p>
                    {{ address }}
                    <v-icon>mdi-content-copy</v-icon>
                  </p>
                </v-col>
              </v-row>
            </v-card-text>
          </v-card>
          <v-overlay z-index="9999" opacity="0.7" :value="loading">
            <v-progress-circular indeterminate size="64"></v-progress-circular>
          </v-overlay>
          <!-- 提示层 -->
          <v-snackbar
            v-model="operationResult.snackbar"
            :color="operationResult.color"
            centered
            top
            timeout="4000"
          >
            {{ $t(operationResult.text) }}
          </v-snackbar>
        </v-col>
      </v-row>
    </v-container>
    <v-container v-else>
      <v-row justify="center">
        <v-col md="6">
          <v-card justify="center" class="fill-width">
            <v-card-actions class="justify-center">
              <!-- 连接钱包 -->
              <v-btn
                class="mr-2"
                v-if="!connected"
                color="#93B954"
                block
                @click="onConnect"
              >
                {{ $t("Connect Wallet") }}
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script>
import { validationMixin } from "vuelidate";
import { required } from "vuelidate/lib/validators";
import clip from "@/utils/clipboard";
import { InviteForRelationshipContractAddress } from "@/constants";
import {
  getContractByABI,
  checkAddressChecksum,
  toChecksumAddress
} from "@/utils/web3";
// 引入合约 ABI 文件
import InviteForRelationship_ABI from "@/constants/contractJson/InviteForRelationship_abi.json";

export default {
  name: "InviteForRelationship",
  mixins: [validationMixin],
  validations: {
    inviterAccount: { required }
  },
  data: () => ({
    loading: false,
    inviterAccountFocus: true,
    inviterAccount: undefined,
    // 当前账户相关信息
    accountAssets: {
      isInvited: false,
      inviterToken: null
    },
    // 提示框
    operationResult: {
      color: "success",
      snackbar: false,
      text: `Hello`
    }
  }),
  created() {
    if (this.web3 && this.connected) {
      this.getAccountAssets();
    } else {
      this.onConnect();
    }
  },
  watch: {
    web3(web3) {
      if (web3) {
        this.getAccountAssets();
      }
    },
    address(address) {
      if (address) {
        this.getAccountAssets();
      }
    }
  },
  computed: {
    connected() {
      return this.$store.state.web3.connected;
    },
    web3() {
      return this.$store.state.web3.web3;
    },
    address() {
      return this.$store.state.web3.address;
    },
    chainId() {
      return this.$store.state.web3.chainId;
    },
    inviterAccountErrors() {
      const errors = [];
      if (!this.$v.inviterAccount.$dirty) return errors;
      !this.$v.inviterAccount.required &&
        errors.push(this.$t("Please enter your mentor's address"));

      try {
        if (checkAddressChecksum(this.$v.inviterAccount.$model)) {
          if (
            toChecksumAddress(this.$v.inviterAccount.$model) == this.address
          ) {
            errors.push(this.$t("The inviter's account cannot be yourself"));
            return errors;
          }
          return errors;
        } else {
          errors.push(this.$t("The mentor's address is wrong"));
        }
      } catch (e) {
        errors.push(this.$t("The mentor's address is wrong"));
      }

      return errors;
    },
    submitLoading() {
      return this.inviterAccount && this.inviterAccountErrors.length <= 0;
    }
  },
  methods: {
    // 连接钱包 OK
    onConnect() {
      this.$store.dispatch("web3/connect");
    },
    // 断开连接钱包 OK
    closeWallet() {
      this.$store.dispatch("web3/closeWallet");
    },
    // 复制地址
    handleCopy(text, event) {
      clip(text, event);
      this.operationResult.color = "success";
      this.operationResult.snackbar = true;
      this.operationResult.text = "Cope Success";
    },
    // 处理输入地址格式
    setAccountAddress(value) {
      try {
        this.inviterAccount = toChecksumAddress(value);
      } catch (error) {
        this.$v.inviterAccount.$touch();
      }
    },
    // 获取账号信息
    async getAccountAssets() {
      this.loading = true;
      try {
        // 查询是否已经被邀请过
        const contract = getContractByABI(
          InviteForRelationship_ABI,
          InviteForRelationshipContractAddress,
          this.web3
        );
        const isInvitee = await contract.methods.isInvitee(this.address).call();
        if (isInvitee) {
          this.accountAssets.isInvited = true;
        } else {
          this.accountAssets.isInvited = false;
        }
      } catch (error) {
        console.info(error);
      }
      this.loading = false;
    },
    // 接受邀请
    async submit() {
      if (this.$v.$invalid) {
        // error info
        if (this.$v.inviterAccount.$invalid) {
          this.inviterAccountFocus = true;
        }
        this.$v.$touch();
      } else {
        this.$v.$touch();
        this.loading = true;
        // 查询结果
        const contract = getContractByABI(
          InviteForRelationship_ABI,
          InviteForRelationshipContractAddress,
          this.web3
        );
        // check account
        const isInvitee = await contract.methods
          .isInvitee(toChecksumAddress(this.inviterAccount))
          .call();
        if (isInvitee) {
          // 执行合约
          contract.methods
            .accept(toChecksumAddress(this.inviterAccount))
            .send({ from: this.address })
            .then(() => {
              this.loading = false;
              this.getAccountAssets();
            })
            .catch(e => {
              this.loading = false;
              console.info(e);
            });
        } else {
          this.operationResult.color = "error";
          this.operationResult.snackbar = true;
          this.operationResult.text = "The mentor's address is wrong";
          this.loading = false;
        }
      }
    }
  }
};
</script>

<style lang="sass">
.theme--dark.v-btn.v-btn--disabled.v-btn--has-bg
  background-color: rgb(147, 185, 84) !important
  border-color: rgb(147, 185, 84) !important
  opacity: 0.5 !important

.v-btn--disabled
  background-color: rgb(147, 185, 84)
  border-color: rgb(147, 185, 84)
  opacity: 0.5
</style>
