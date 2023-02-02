<template>
  <div class="fill-height">
    <v-container v-if="web3 && connected" class="fill-height">
      <v-row justify="center">
        <v-col md="6">
          <!-- 公告 -->
          <v-card justify="center" class="fill-width">
            <v-card-title>
              <span class="title font-weight-bold text-h5">
                {{ $t("Activities") }}
              </span>
            </v-card-title>
            <v-divider></v-divider>
            <v-card-text>
              <v-row align="center">
                <v-col class="body-1" cols="12">
                  <p>{{ $t("Rewards Duration") }}：{{ powerDuration }}</p>
                  <p>
                    {{
                      $t(
                        "Will caculate and airdrop within 3 workdays after the last day."
                      )
                    }}
                  </p>
                </v-col>
              </v-row>
            </v-card-text>
          </v-card>
          <!-- 操作 -->
          <v-card class="fill-width mt-10">
            <v-card outlined>
              <!-- 标题 -->
              <v-card-title>
                <v-avatar size="24" class="mr-2">
                  <img :src="require('@/assets/logo.png')" alt="DAO" />
                </v-avatar>
                <span class="title font-weight-bold text-h5">
                  {{ $t("New Power Reward") }}
                </span>
              </v-card-title>
              <v-divider></v-divider>
              <v-card-text v-if="rewardsDataList.length > 0">
                <v-card
                  v-for="item in rewardsDataList"
                  :key="item.account"
                  :loading="loading"
                  class="ma-2"
                >
                  <v-card-title>
                    {{ $t("Power Phase") }}
                    {{ item.periodId }}
                    {{ $t("Power Expect") }}
                  </v-card-title>
                  <v-divider class="mx-4"></v-divider>
                  <v-card-text>
                    <p>
                      {{ $t("Power Duration") }}：{{
                        item.startTime | parseTime("{y}-{m}-{d}")
                      }}
                      ~
                      {{ item.endTime | parseTime("{y}-{m}-{d}") }}
                    </p>
                    <p>
                      {{ $t("Power Node Status") }}：{{
                        $t(`Node.${item.rewardsInfo.nodeType}`)
                      }}
                    </p>
                    <p>
                      {{
                        $t("Personal cumulative total accounting strength")
                      }}：{{ item.rewardsInfo.power | keepNumber }}
                    </p>
                    <p>
                      {{
                        $t(
                          "New accumulated calculation power in the current period"
                        )
                      }}：{{ item.rewardsInfo.powerIncrement | keepNumber }}
                    </p>
                    <p>
                      {{ $t("Claimable Amount") }} /
                      {{ $t("Claimabled Amount") }}：{{
                        item.rewardsInfo.isClaim
                          ? 0
                          : item.rewardsInfo.rewardDAO
                      }}
                      /
                      {{
                        item.rewardsInfo.isClaim
                          ? item.rewardsInfo.rewardDAO
                          : 0
                      }}
                      {{ tokenSymbol }}
                    </p>
                  </v-card-text>
                  <v-divider class="mx-4"></v-divider>
                  <v-card-actions
                    class="justify-center"
                    v-if="
                      parseFloat(item.rewardsInfo.rewardDAO) > 0 &&
                        !item.rewardsInfo.isClaim
                    "
                  >
                    <v-btn
                      color="#93B954"
                      dark
                      width="80%"
                      @click="handleRelease(item)"
                      :disabled="
                        item.rewardsInfo.isClaim ||
                          parseFloat(item.rewardsInfo.rewardDAO) <= 0
                      "
                    >
                      {{ $t("Claim") }}
                    </v-btn>
                  </v-card-actions>
                </v-card>
              </v-card-text>
              <v-card-text v-else>
                <v-row align="center">
                  <v-col class="body-3" cols="12">
                    {{ $t("No Data") }}
                  </v-col>
                </v-row>
              </v-card-text>
            </v-card>
          </v-card>
          <!-- 当前钱包账号 -->
          <v-card justify="center" class="fill-width mt-10">
            <v-card-title>
              <span class="title font-weight-bold text-h5">
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
          <!-- 遮罩层 -->
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
import clip from "@/utils/clipboard";
import { getContractByABI, weiToEther } from "@/utils/web3";
import { judgeCHNNodeTypeByValue, compare, keepNumber } from "@/filters/index";
// 引入合约 ABI 文件
import CHNNewPowerReward_ABI from "@/constants/abi/CHNNewPowerReward_abi.json";

export default {
  name: "NewPowerReward",
  data: () => ({
    loading: false,
    tokenSymbol: "DAO",
    // 算力合约列表
    powerDuration: "2023-01-01 11:00:00 ~ 2023-02-01 11:00:00",
    rewardsContractAddressList: [
      {
        id: 12,
        address: "0x2425F7014d3434a3f0f96639c59D03Abef1a27F0"
      }
    ],
    // 算力数据列表
    rewardsDataList: [],
    // 提示框
    operationResult: {
      color: "success",
      snackbar: false,
      text: `Hello`
    }
  }),
  created() {
    if (this.web3 && this.connected) {
      this.getRewardsDataList();
    } else {
      this.onConnect();
    }
  },
  watch: {
    web3(web3) {
      if (web3) {
        this.getRewardsDataList();
      }
    },
    address(address) {
      if (address) {
        this.getRewardsDataList();
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
      // return "0x3DdcFc89B4DD2b33d9a8Ca0F60180527E9810D4B";
      // return "0x7d3dE024dEB70741c6Dfa0FaD57775A47C227AE2";
      return this.$store.state.web3.address;
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
    // 获取算力数据列表
    async getRewardsDataList() {
      if (this.rewardsContractAddressList.length > 0) {
        this.rewardsDataList = [];
        this.loading = true;
        const getResult = this.rewardsContractAddressList.map(async item => {
          const contract = await getContractByABI(
            CHNNewPowerReward_ABI,
            item.address,
            this.web3
          );
          const hasRewardsInfo = await contract.methods
            .hasRewardsInfo(this.address)
            .call();
          if (hasRewardsInfo) {
            const startTime = await contract.methods.startTime().call();
            const endTime = await contract.methods.endTime().call();
            const rewardsInfo = await contract.methods
              .accountRewardsMap(this.address)
              .call();
            const tempData = {
              periodId: item.id,
              contractAddress: item.address,
              startTime: startTime,
              endTime: endTime,
              rewardsInfo: {
                nodeType: judgeCHNNodeTypeByValue(rewardsInfo.nodeType),
                power: weiToEther(rewardsInfo.power, this.web3),
                powerIncrement: weiToEther(
                  rewardsInfo.powerIncrement,
                  this.web3
                ),
                rewardDAO: keepNumber(
                  weiToEther(rewardsInfo.rewardDAO, this.web3)
                ),
                isClaim: rewardsInfo.isClaim
              }
            };
            this.rewardsDataList.push(tempData);
          }
        });
        await Promise.all(getResult);
        this.rewardsDataList.sort(compare("periodId"));
        this.loading = false;
      }
    },
    // 提取DAO
    handleRelease(record) {
      this.loading = true;
      // 执行合约
      getContractByABI(CHNNewPowerReward_ABI, record.contractAddress, this.web3)
        .methods.getRewards()
        .send({ from: this.address })
        .then(() => {
          this.loading = false;
          this.operationResult.snackbar = true;
          this.operationResult.text = "Claim Success";
          // 重新获取数据
          this.getRewardsDataList();
        })
        .catch(e => {
          this.loading = false;
          console.info(e);
        });
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
