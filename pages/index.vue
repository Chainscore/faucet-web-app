<template>
  <v-row justify="center" align="center">
    <v-col cols="12" sm="8" md="6">
      <v-card class="pa-2 text-center my-10" color="transparent" flat>
        <VuetifyLogo />
        <div class="text-h4 yellow--text my-5">Faucet</div>
        <v-card-text>
          <v-select
            :items="networks.map((x) => x.chainName)"
            label="Network"
            rounded
            outlined
            small
            v-model="selectedNetworkName"
          ></v-select>
          <v-btn
            color="yellow"
            nuxt
            light
            @click="request()"
            rounded
            class="px-5"
          >
            request
          </v-btn>

          <div v-if="hash && !receipt" class="my-4 yellow--text">
            <div>Tx sent!</div>
            <a
              :href="this.selectedNetwork.blockExplorerUrls[0] + 'tx/' + hash"
              target="_blank"
              >{{ hash }}</a
            >
          </div>

          <div v-if="receipt" class="my-4 green--text">
            <div>Tx confirmed! $SCORE tokens sent successfully</div>
          </div>

          <div v-if="error" class="my-4 red--text">
            <div>Errored</div>
          </div>
        </v-card-text>
        <v-card-actions class=" d-flex flex-column justify-center">
          <v-btn rounded outlined class="pa-2" x-small @click="addScore()"
            >add $score to metamask</v-btn
          >
        </v-card-actions>
        <v-card-text class="mt-15">
          <p>
            Thank you for developing with ChainScore and We look forward to
            bringing more exciting features in the future.
          </p>
          <div class="text-xs-right">
            <em><small>Contact: hello@chainscore.finance</small></em>
          </div>
          <hr class="my-3" />
          <a
            href="https://docs.chainscore.finance/"
            target="_blank"
            rel="noopener noreferrer"
          >
            Documentation
          </a>
          <br />
          <a
            href="https://discord.gg/ZhKsjC8464"
            target="_blank"
            rel="noopener noreferrer"
          >
            Discord
          </a>
        </v-card-text>
      </v-card>
    </v-col>
  </v-row>
</template>

<script>
const Web3 = require('web3')
window.web3 = new Web3(window.ethereum)

const faucetArtifacts = require('./Faucet.json')
const tokenArtifacts = require('./ERC20.json')

export default {
  async mounted(){
    let cid = await window.ethereum.request({ method: 'eth_chainId' })
    try{
    this.selectedNetworkName = this.networks.filter(x => x.chainId == cid)[0].chainName;
    } catch(err) {}
  },
  data() {
    return {
      selectedNetworkName: null,
      selectedNetwork: null,
      faucet: {
        'Harmony Testnet': '0xb0abAc741E1D2074f131A8aC1F66C8EE7aff18bB',
        'Aurora Testnet': '0x019f6c47B96e07C5e80808e19f3aDfF774fb1d47',
        'Rinkeby': '0x1f9B43D937E4028C159C9DDbA3b7C2Fd807994c9',
      },
      networks: [
        {
          chainName: 'Harmony Testnet',
          chainId: '0x' + (1666700000).toString(16),
          rpcUrls: ['https://api.s0.b.hmny.io'],
          nativeCurrency: {
            name: 'ONE',
            symbol: 'ONE',
            decimals: 18,
          },
          blockExplorerUrls: ['https://explorer.pops.one/'],
        },
        {
          chainName: 'Aurora Testnet',
          chainId: '0x4e454153',
          rpcUrls: ['https://testnet.aurora.dev/'],
          nativeCurrency: {
            name: 'Ethereum',
            symbol: 'AETH',
            decimals: 18,
          },
          blockExplorerUrls: ['https://explorer.testnet.aurora.dev/'],
        },
        {
          chainName: 'Rinkeby',
          chainId: '0x' + (4).toString(16),
          rpcUrls: [],
          nativeCurrency: {
            name: 'Ethereum',
            symbol: 'ETH',
            decimals: 18,
          },
          blockExplorerUrls: ['https://rinkeby.etherscan.io/'],
        },
      ],
      tokens: {
        'Aurora Testnet': {
          address: "0x728bd98edb13a84Aa24891053902b4f64a96679e",
          symbol: "SCORE",
          decimals: "18",
          image: "https://raw.githubusercontent.com/Chainscore/branding/main/Frame%209.png",
        },
        'Harmony Testnet': {
          address: "0x64B60e8C3b8527011B48aD9a19265680FE901CEE",
          symbol: "SCORE",
          decimals: "18",
          image: "https://raw.githubusercontent.com/Chainscore/branding/main/Frame%209.png",
        },
        'Rinkeby': {
          address: "0x019f6c47B96e07C5e80808e19f3aDfF774fb1d47",
          symbol: "SCORE",
          decimals: "18",
          image: "https://raw.githubusercontent.com/Chainscore/branding/main/Frame%209.png",
        },
      },

      hash: null,
      receipt: null,
      error: null,
    }
  },
  methods: {
    async connect() {
      if (this.hasEthereum()) {
        this.accounts = await window.ethereum.request({
          method: 'eth_requestAccounts',
        })
        try {
          this.selectedNetwork = this.networks.filter(
            (x) => x.chainName == this.selectedNetworkName
          )[0]
          await window.ethereum.request({
            method: 'wallet_switchEthereumChain',
            params: [{ chainId: this.selectedNetwork.chainId }],
          })
        } catch (switchError) {
          // This error code indicates that the chain has not been added to MetaMask.
          if (switchError.code === 4902) {
            try {
              await window.ethereum.request({
                method: 'wallet_addEthereumChain',
                params: [this.selectedNetwork],
              })
            } catch (addError) {
              // handle "add" error
            }
          }
        }
      } else {
        this.error = 'Metamask not installed'
      }
    },

    hasEthereum() {
      const { ethereum } = window
      if (ethereum && ethereum.isMetaMask) {
        return true
      } else {
        return false
      }
    },

    async request() {
      await this.connect()
      window.web3 = new Web3(window.ethereum)

      let faucet = new web3.eth.Contract(
        faucetArtifacts.abi,
        this.faucet[this.selectedNetwork.chainName]
      )

      faucet.methods
        .request()
        .send({
          from: this.accounts[0],
        })
        .on('transactionHash', (hash) => {
          this.hash = hash
        })
        .on('receipt', (receipt) => {
          // receipt example
          this.receipt = receipt
        })
        .on('error', (error, receipt) => {
          // If the transaction was rejected by the network with a receipt, the second parameter will be the receipt.
          this.error = error
        })
    },

    async addScore() {
      // wasAdded is a boolean. Like any RPC method, an error may be thrown.
      await window.ethereum.request({
        method: 'wallet_watchAsset',
        params: {
          type: 'ERC20', // Initially only supports ERC20, but eventually more!
          options: this.tokens[this.selectedNetworkName],
        },
      })
    },
  },
}
</script>
