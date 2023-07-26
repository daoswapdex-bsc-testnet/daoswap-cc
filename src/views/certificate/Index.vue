<template>
  <div class="fill-height">
    <v-container v-if="web3 && connected" class="fill-height">
      <v-row justify="center">
        <v-col md="6">
          <!-- 操作 -->
          <v-card class="fill-width mt-10">
            <v-card outlined>
              <!-- 标题 -->
              <v-card-title>
                <v-avatar size="24" class="mr-2">
                  <img :src="require('@/assets/logo.png')" alt="DAO" />
                </v-avatar>
                <span class="title font-weight-bold text-h5">
                  {{ $t("Certificate") }}
                </span>
              </v-card-title>
              <v-divider></v-divider>
              <v-card-text v-if="certificateList.length >= 0">
                <v-row>
                  <v-col
                    v-for="(item, index) in certificateList"
                    :key="index"
                    class="d-flex child-flex list-item"
                    cols="12"
                  >
                    <v-img
                      :src="item.image"
                      aspect-ratio="1.5"
                      class="grey lighten-2 list-item-image"
                      @click="previewImg(item.image, item.title)"
                    >
                      <template v-slot:placeholder>
                        <v-row
                          class="fill-height ma-0"
                          align="center"
                          justify="center"
                        >
                          <v-progress-circular
                            indeterminate
                            color="grey lighten-5"
                          ></v-progress-circular>
                        </v-row>
                      </template>
                    </v-img>
                    <div class="list-item-title">{{ item.title }}</div>
                  </v-col>
                </v-row>
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
    <!-- 图片放大 -->
    <v-row justify="center">
      <v-dialog
        v-model="dialog"
        fullscreen
        hide-overlay
        transition="dialog-bottom-transition"
      >
        <v-card dark>
          <v-toolbar dark color="primary" style="z-index: 100;">
            <v-toolbar-title>{{ previewImgTitle }}</v-toolbar-title>
            <v-spacer></v-spacer>
            <v-toolbar-items>
              <v-btn icon dark @click="dialog = false">
                <v-icon>mdi-close</v-icon>
              </v-btn>
            </v-toolbar-items>
          </v-toolbar>
          <v-img class="dialog-image" contain :src="previewImgUrl"></v-img>
        </v-card>
      </v-dialog>
    </v-row>
  </div>
</template>

<script>
import clip from "@/utils/clipboard";
import { getContractByABI } from "@/utils/web3";
// 引入合约 ABI 文件
import Certificate_ABI from "@/constants/abi/Certificate_abi.json";

export default {
  name: "Certificate",
  data: () => ({
    loading: false,
    // 路径前缀
    pathDomain: "https://ipfs.io/ipfs/",
    contractAddress: "0x7713FB320BD6Fe93580Fe1f3e16D790425ca6647",
    // 证书数据列表
    certificateList: [],
    // 提示框
    operationResult: {
      color: "success",
      snackbar: false,
      text: `Hello`
    },
    // 弹窗信息
    dialog: false,
    previewImgTitle: "",
    previewImgUrl: ""
  }),
  created() {
    if (this.web3 && this.connected) {
      this.getCertificateList();
    } else {
      this.onConnect();
    }
  },
  watch: {
    web3(web3) {
      if (web3) {
        this.getCertificateList();
      }
    },
    address(address) {
      if (address) {
        this.getCertificateList();
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
      // return "0xBdB9BD48CDCF075D66f81f083b9Ab618a0530c31";
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
    // 获取数据列表
    async getCertificateList() {
      if (this.contractAddress) {
        this.certificateList = [];
        this.loading = true;
        const contract = await getContractByABI(
          Certificate_ABI,
          this.contractAddress,
          this.web3
        );
        const hasInfo = await contract.methods.hasInfo(this.address).call();
        if (hasInfo) {
          const accountInfo = await contract.methods
            .getAccountInfo()
            .call({ from: this.address });
          if (accountInfo.certificates.length > 0) {
            accountInfo.certificates.map(item => {
              const tempItem = {
                title: item.name,
                image:
                  this.pathDomain +
                  item.cid +
                  "?filename=" +
                  this.address +
                  ".png"
              };
              this.certificateList.push(tempItem);
            });
          }
        }
        this.loading = false;
      }
    },
    // 预览图片
    previewImg(uri, title) {
      this.dialog = true;
      this.previewImgTitle = title;
      this.previewImgUrl = uri;
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

.list-item
  display: flex
  align-items: center
  justify-content: space-around
  flex-direction: column
.list-item-image
  width: 100%
.list-item-title
  margin-top: 10px
.dialog-image
  width: 100%
  height: 100%
  position: absolute !important
  top: 0
  z-index: 10
</style>
