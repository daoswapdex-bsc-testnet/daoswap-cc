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
                  <p>{{ $t("Power Duration") }}：{{ powerDuration }}</p>
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
                  {{ $t("Hash Mining") }}
                </span>
              </v-card-title>
              <v-divider></v-divider>
              <v-card-text v-if="powerDataListNew2.length > 0">
                <v-card
                  v-for="item in powerDataListNew2"
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
                        $t(`Node.${item.nodeType}`)
                      }}
                    </p>
                    <p>
                      {{
                        $t("Personal cumulative total accounting strength")
                      }}：{{ item.power | keepNumber }}
                    </p>
                    <p v-if="item.rewardDAO.isAble && item.rewardDST.isAble">
                      {{
                        $t(
                          "New accumulated calculation power in the current period"
                        )
                      }}：{{ item.powerIncrement | keepNumber }}
                    </p>
                    <p>
                      {{ $t("Reward ratio of each calculation power") }}：{{
                        item.rewardRatio
                      }}
                      (20%{{ item.rewardDAO.tokenSymbol }}, 80%{{
                        item.rewardDST.tokenSymbol
                      }})
                    </p>
                    <p>
                      {{
                        $t(
                          "LP quantity of DAO-USDT to be added in the next period"
                        )
                      }}：{{ item.nextStandard }}
                    </p>
                  </v-card-text>
                  <v-divider class="mx-4"></v-divider>
                  <v-card-actions class="justify-center">
                    <v-simple-table>
                      <template v-slot:default>
                        <thead>
                          <tr>
                            <th class="text-left">
                              {{ $t("Token Symbol") }}
                            </th>
                            <th class="text-left">
                              {{ $t("Claimable Amount") }}
                            </th>
                            <th class="text-left">
                              {{ $t("Claimabled Amount") }}
                            </th>
                            <th class="text-left">{{ $t("Operation") }}</th>
                          </tr>
                        </thead>
                        <tbody>
                          <tr>
                            <td>{{ item.rewardDAO.tokenSymbol }}</td>
                            <td>
                              {{
                                (item.rewardDAO.isClaim
                                  ? 0
                                  : item.rewardDAO.amount) | keepNumber
                              }}
                            </td>
                            <td>
                              {{
                                (item.rewardDAO.isClaim
                                  ? item.rewardDAO.amount
                                  : 0) | keepNumber
                              }}
                            </td>
                            <td>
                              <v-btn
                                v-if="
                                  !item.rewardDAO.isClaim &&
                                    item.rewardDAO.isAble
                                "
                                small
                                color="#93B954"
                                dark
                                width="80%"
                                @click="
                                  handleReleaseNew(
                                    item.contractAddress,
                                    item.rewardDAO.token
                                  )
                                "
                              >
                                {{ $t("Claim") }}
                              </v-btn>
                            </td>
                          </tr>
                          <tr>
                            <td>{{ item.rewardDST.tokenSymbol }}</td>
                            <td>
                              {{
                                (item.rewardDST.isClaim
                                  ? 0
                                  : item.rewardDST.amount) | keepNumber
                              }}
                            </td>
                            <td>
                              {{
                                (item.rewardDST.isClaim
                                  ? item.rewardDST.amount
                                  : 0) | keepNumber
                              }}
                            </td>
                            <td>
                              <v-btn
                                v-if="
                                  !item.rewardDST.isClaim &&
                                    item.rewardDST.isAble
                                "
                                small
                                color="#93B954"
                                dark
                                width="80%"
                                @click="
                                  handleReleaseNew(
                                    item.contractAddress,
                                    item.rewardDST.token
                                  )
                                "
                              >
                                {{ $t("Claim") }}
                              </v-btn>
                            </td>
                          </tr>
                        </tbody>
                      </template>
                    </v-simple-table>
                  </v-card-actions>
                </v-card>
              </v-card-text>
              <v-card-text v-if="powerDataListNew.length > 0">
                <v-card
                  v-for="item in powerDataListNew"
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
                        $t(`Node.${item.nodeType}`)
                      }}
                    </p>
                    <p>
                      {{
                        $t("Personal cumulative total accounting strength")
                      }}：{{ item.power | keepNumber }}
                    </p>
                    <p>
                      {{
                        $t(
                          "LP quantity of DAO-USDT to be added in the next period"
                        )
                      }}：{{ item.nextStandard }}
                    </p>
                  </v-card-text>
                  <v-divider class="mx-4"></v-divider>
                  <v-card-actions class="justify-center">
                    <v-simple-table>
                      <template v-slot:default>
                        <thead>
                          <tr>
                            <th class="text-left">
                              {{ $t("Token Symbol") }}
                            </th>
                            <th class="text-left">
                              {{ $t("Claimable Amount") }}
                            </th>
                            <th class="text-left">
                              {{ $t("Claimabled Amount") }}
                            </th>
                            <th class="text-left">{{ $t("Operation") }}</th>
                          </tr>
                        </thead>
                        <tbody>
                          <tr>
                            <td>{{ item.rewardDAO.tokenSymbol }}</td>
                            <td>
                              {{
                                (item.rewardDAO.isClaim
                                  ? 0
                                  : item.rewardDAO.amount) | keepNumber
                              }}
                            </td>
                            <td>
                              {{
                                (item.rewardDAO.isClaim
                                  ? item.rewardDAO.amount
                                  : 0) | keepNumber
                              }}
                            </td>
                            <td>
                              <v-btn
                                v-if="!item.rewardDAO.isClaim"
                                small
                                color="#93B954"
                                dark
                                width="80%"
                                @click="
                                  handleReleaseNew(
                                    item.contractAddress,
                                    item.rewardDAO.token
                                  )
                                "
                              >
                                {{ $t("Claim") }}
                              </v-btn>
                            </td>
                          </tr>
                          <tr>
                            <td>{{ item.rewardDST.tokenSymbol }}</td>
                            <td>
                              {{
                                (item.rewardDST.isClaim
                                  ? 0
                                  : item.rewardDST.amount) | keepNumber
                              }}
                            </td>
                            <td>
                              {{
                                (item.rewardDST.isClaim
                                  ? item.rewardDST.amount
                                  : 0) | keepNumber
                              }}
                            </td>
                            <td>
                              <v-btn
                                v-if="!item.rewardDST.isClaim"
                                small
                                color="#93B954"
                                dark
                                width="80%"
                                @click="
                                  handleReleaseNew(
                                    item.contractAddress,
                                    item.rewardDST.token
                                  )
                                "
                              >
                                {{ $t("Claim") }}
                              </v-btn>
                            </td>
                          </tr>
                        </tbody>
                      </template>
                    </v-simple-table>
                  </v-card-actions>
                </v-card>
              </v-card-text>
              <v-card-text v-if="powerDataList.length > 0">
                <v-card
                  v-for="item in powerDataList"
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
                    <p>{{ $t("Power All Hash") }}：{{ item.countedPower }}</p>
                    <p>
                      {{ $t("Power Node Proportion") }}：{{
                        item.annualizedRate
                      }}
                      %
                    </p>
                    <!-- <p>
                      {{ $t("Power Node Hash Value") }}：$
                      {{ item.powerInfo.powerValue }}
                    </p> -->
                    <p>
                      {{ $t("Power Node Status") }}：{{
                        $t(`Node.${item.powerInfo.nodeType}`)
                      }}
                    </p>
                    <!-- <p>
                      {{ $t("Power DAO-USDT Liquidity Value") }}：$
                      {{ item.powerInfo.liquidity }}
                    </p> -->
                    <p>
                      {{ $t("Claimable Amount") }} /
                      {{ $t("Claimabled Amount") }}：{{
                        item.powerInfo.isClaim
                          ? 0
                          : item.powerInfo.receiveAmount
                      }}
                      /
                      {{
                        item.powerInfo.isClaim
                          ? item.powerInfo.receiveAmount
                          : 0
                      }}
                      {{ tokenSymbol }}
                    </p>
                  </v-card-text>
                  <v-divider class="mx-4"></v-divider>
                  <v-card-actions class="justify-center">
                    <v-btn
                      color="#93B954"
                      dark
                      width="80%"
                      @click="handleRelease(item)"
                      :disabled="item.powerInfo.isClaim"
                    >
                      {{ $t("Claim") }}
                    </v-btn>
                  </v-card-actions>
                </v-card>
              </v-card-text>
              <v-card-text
                v-if="
                  powerDataListNew2.length <= 0 &&
                    powerDataListNew.length <= 0 &&
                    powerDataList.length <= 0
                "
              >
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
import { JSBI } from "@/utils/jsbi";
import clip from "@/utils/clipboard";
import { getContract, getContractByABI, weiToEther } from "@/utils/web3";
import { judgeCHNNodeTypeByValue, compare } from "@/filters/index";
// 引入合约 ABI 文件
import ComputingPowerMining from "@/constants/contractJson/ComputingPowerMiningForLiquidity.json";
import CHNPowerMining_ABI from "@/constants/abi/CHNPowerMining_abi.json";
import CHNPowerMining2_ABI from "@/constants/abi/CHNPowerMining2_abi.json";

export default {
  name: "ComputingPowerMiningForLiquidity",
  data: () => ({
    loading: false,
    tokenSymbol: "DAO",
    // 算力合约列表
    powerDuration: "2023-01-01 11:00:00 ~ 2023-02-01 11:00:00",
    powerContractAddressList: [
      // {
      //   id: 1,
      //   address: "0x486a483adDf8446AE2412A1E9bf30D1A90f0e026"
      // },
      // {
      //   id: 2,
      //   address: "0x84892cb24159d3DBdB9b704c4c5a86b52d21D915"
      // },
      // {
      //   id: 3,
      //   address: "0x2B2F68df4B8DDae04ACCDd15470c9f2cC8e1f926"
      // },
      // {
      //   id: 4,
      //   address: "0xd667bAE20e4dBF36099ECA20240201aed3c4e77f"
      // },
      // {
      //   id: 5,
      //   address: "0x2EC0d8465af466c57F75Bf166a5180Fd7B0c513e"
      // },
      // {
      //   id: 6,
      //   address: "0x15485AAb318c60d01a57934ADE5a28282314f0Ce"
      // },
      // {
      //   id: 7,
      //   address: "0x0B37cA4489AE9f667A103D9FE98E306185715891"
      // },
      // {
      //   id: 8,
      //   address: "0x3740314E93D613787b14Dc67958A0dcFC1c167A2"
      // },
      // {
      //   id: 9,
      //   address: "0x3f98f690DFec48a03848c045d2D5E7F4838ee70E"
      // }
    ],
    powerContractAddressListNew: [
      {
        id: 11,
        address: "0xB31A0F5e8d8176B56ac85791AA4e34573f80aE4d"
      },
      {
        id: 10,
        address: "0x6Da17c3AFC4796D2da734dCBdA67FEd83b5D050a"
      }
    ],
    powerContractAddressListNew2: [
      {
        id: 12,
        address: "0x814Fe14B41cBDAE2BBeAeDc855bd1aD4bC459eA4"
      }
    ],
    // 算力数据列表
    powerDataList: [],
    powerDataListNew: [],
    powerDataListNew2: [],
    // 提示框
    operationResult: {
      color: "success",
      snackbar: false,
      text: `Hello`
    }
  }),
  created() {
    if (this.web3 && this.connected) {
      this.getPowerInfo();
    } else {
      this.onConnect();
    }
  },
  watch: {
    web3(web3) {
      if (web3) {
        this.getPowerInfo();
      }
    },
    address(address) {
      if (address) {
        this.getPowerInfo();
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
    // 获取算力数据
    async getPowerInfo() {
      await this.getPowerDataListNew2();
      await this.getPowerDataListNew();
      await this.getPowerDataList();
    },
    // 获取算力数据列表-新版2
    async getPowerDataListNew2() {
      if (this.powerContractAddressListNew2.length > 0) {
        this.powerDataListNew2 = [];
        this.loading = true;
        const getResult = this.powerContractAddressListNew2.map(async item => {
          const contract = await getContractByABI(
            CHNPowerMining2_ABI,
            item.address,
            this.web3
          );
          const hasRewardsInfo = await contract.methods
            .hasRewardsInfo(this.address)
            .call();
          if (hasRewardsInfo) {
            const startTime = await contract.methods.startTime().call();
            const endTime = await contract.methods.endTime().call();
            const rewardRatio = await contract.methods.rewardRatio().call();
            const rewardsInfo = await contract.methods
              .getRewardsInfo()
              .call({ from: this.address });
            if (
              !rewardsInfo.rewardDAO.isClaim ||
              !rewardsInfo.rewardDST.isClaim
            ) {
              const tempData = {
                periodId: item.id,
                contractAddress: item.address,
                nodeType: judgeCHNNodeTypeByValue(rewardsInfo.nodeType),
                power: weiToEther(rewardsInfo.power, this.web3),
                powerIncrement: weiToEther(
                  rewardsInfo.powerIncrement,
                  this.web3
                ),
                nextStandard: rewardsInfo.nextStandard,
                rewardDAO: {
                  token: rewardsInfo.rewardDAO.token,
                  tokenSymbol: rewardsInfo.rewardDAO.tokenSymbol,
                  amount: weiToEther(rewardsInfo.rewardDAO.amount, this.web3),
                  isClaim: rewardsInfo.rewardDAO.isClaim,
                  isAble: JSBI.greaterThan(
                    JSBI.BigInt(rewardsInfo.rewardDAO.amount),
                    JSBI.BigInt(0)
                  )
                },
                rewardDST: {
                  token: rewardsInfo.rewardDST.token,
                  tokenSymbol: rewardsInfo.rewardDST.tokenSymbol,
                  amount: weiToEther(rewardsInfo.rewardDST.amount, this.web3),
                  isClaim: rewardsInfo.rewardDST.isClaim,
                  isAble: JSBI.greaterThan(
                    JSBI.BigInt(rewardsInfo.rewardDST.amount),
                    JSBI.BigInt(0)
                  )
                },
                startTime: startTime,
                endTime: endTime,
                rewardRatio: rewardRatio
              };
              this.powerDataListNew2.push(tempData);
            }
          }
        });
        await Promise.all(getResult);
        this.powerDataListNew2.sort(compare("periodId"));
        this.loading = false;
      }
    },
    // 获取算力数据列表-新版
    async getPowerDataListNew() {
      if (this.powerContractAddressListNew.length > 0) {
        this.powerDataListNew = [];
        this.loading = true;
        const getResult = this.powerContractAddressListNew.map(async item => {
          const contract = await getContractByABI(
            CHNPowerMining_ABI,
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
              .getRewardsInfo()
              .call({ from: this.address });
            if (
              !rewardsInfo.rewardDAO.isClaim ||
              !rewardsInfo.rewardDST.isClaim
            ) {
              const tempData = {
                periodId: item.id,
                contractAddress: item.address,
                nodeType: judgeCHNNodeTypeByValue(rewardsInfo.nodeType),
                power: weiToEther(rewardsInfo.power, this.web3),
                nextStandard: rewardsInfo.nextStandard,
                rewardDAO: {
                  token: rewardsInfo.rewardDAO.token,
                  tokenSymbol: rewardsInfo.rewardDAO.tokenSymbol,
                  amount: weiToEther(rewardsInfo.rewardDAO.amount, this.web3),
                  isClaim: rewardsInfo.rewardDAO.isClaim
                },
                rewardDST: {
                  token: rewardsInfo.rewardDST.token,
                  tokenSymbol: rewardsInfo.rewardDST.tokenSymbol,
                  amount: weiToEther(rewardsInfo.rewardDST.amount, this.web3),
                  isClaim: rewardsInfo.rewardDST.isClaim
                },
                startTime: startTime,
                endTime: endTime
              };
              this.powerDataListNew.push(tempData);
            }
          }
        });
        await Promise.all(getResult);
        this.powerDataListNew.sort(compare("periodId"));
        this.loading = false;
      }
    },
    // 获取算力数据列表
    async getPowerDataList() {
      if (this.powerContractAddressList.length > 0) {
        this.powerDataList = [];
        this.loading = true;
        const getResult = this.powerContractAddressList.map(async item => {
          const contract = await getContract(
            ComputingPowerMining,
            item.address,
            this.web3
          );
          const hasPowerInfo = await contract.methods
            .hasPowerInfo(this.address)
            .call();
          if (hasPowerInfo) {
            const countedPower = await contract.methods.countedPower().call();
            const countedPowerValue = await contract.methods
              .countedPowerValue()
              .call();
            const startTime = await contract.methods.startTime().call();
            const endTime = await contract.methods.endTime().call();
            const powerInfo = await contract.methods
              .accountPowerInfoList(this.address)
              .call();
            if (!powerInfo.isClaim) {
              const annualizedRate = (powerInfo.power / countedPower) * 100;
              const tempData = {
                periodId: item.id,
                contractAddress: item.address,
                countedPower: weiToEther(countedPower, this.web3),
                countedPowerValue: weiToEther(countedPowerValue, this.web3),
                startTime: startTime,
                endTime: endTime,
                powerInfo: {
                  power: weiToEther(powerInfo.power, this.web3),
                  powerValue: weiToEther(powerInfo.powerValue, this.web3),
                  receiveAmount: weiToEther(powerInfo.receiveAmount, this.web3),
                  nodeType: judgeCHNNodeTypeByValue(powerInfo.nodeType),
                  liquidity: weiToEther(powerInfo.liquidity, this.web3),
                  isClaim: powerInfo.isClaim
                },
                annualizedRate: parseFloat(annualizedRate).toFixed(2)
              };
              this.powerDataList.push(tempData);
            }
          }
        });
        await Promise.all(getResult);
        this.powerDataList.sort(compare("periodId"));
        this.loading = false;
      }
    },
    // 提取DAO
    handleRelease(record) {
      this.loading = true;
      // 执行合约
      getContract(ComputingPowerMining, record.contractAddress, this.web3)
        .methods.claim()
        .send({ from: this.address })
        .then(() => {
          this.loading = false;
          this.operationResult.snackbar = true;
          this.operationResult.text = "Claim Success";
          // 重新获取数据
          this.getPowerDataList();
        })
        .catch(e => {
          this.loading = false;
          console.info(e);
        });
    },
    // 提取代币
    handleReleaseNew(contractAddress, token) {
      this.loading = true;
      // 执行合约
      getContractByABI(CHNPowerMining_ABI, contractAddress, this.web3)
        .methods.getRewards(token)
        .send({ from: this.address })
        .then(() => {
          this.loading = false;
          this.operationResult.snackbar = true;
          this.operationResult.text = "Claim Success";
          // 重新获取数据
          this.getPowerInfo();
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
