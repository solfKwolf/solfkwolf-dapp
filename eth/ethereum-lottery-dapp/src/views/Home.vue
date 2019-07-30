<template>
  <div class="home">
    <img alt="Vue logo" src="../assets/logo.png" />
    <HelloWorld msg="Welcome to Your Vue.js App" />
    <div>
      <button @click="participate">参与抽奖</button>
    </div>
    <div v-if="isOwner">
      <button @click="pickWinner">发起抽奖</button>
    </div>
    <ul>
      <li v-for="(item,index) in players" :key="index">
        {{item}}
      </li>
    </ul>
    
  </div>
</template>

<script>
// @ is an alias to /src
import HelloWorld from '@/components/HelloWorld.vue'
import getWeb3 from '@/utils/getWeb3.js'
// 获取编译的abi
import LotteryContract from '@/contracts/Lottery.json'



export default {
  name: 'home',
  data() {
    return {
      web3: {},
      accounts: [],
      contract: {},
      players: [],
      owner: ""
    }
  },
  components: {
    HelloWorld
  },
  async created() {
    try {
      // 获取网络中的provider 和 web3 实例
      const web3 = await getWeb3();

      // 使用web3获取用户的账号
      const accounts = await web3.eth.getAccounts();

      // 获取合约实例

      const address = "0xade03d45fca09224c76bbeda4e49bdd93ac90c0c"
      const instance = new web3.eth.Contract(
        LotteryContract,
        address,
      );

      // 设置web3实例，账户信息 和 合约实例 到 state，然后使用一个例子与合约交互
      this.web3 = web3;
      this.accounts = accounts;
      this.contract = instance;

      const that = this;
      // 获取参与抽奖的人,以及合约拥有者
      const [players, owner] = await Promise.all([
        await that.contract.methods.getPlayers().call(),
        await that.contract.methods.owner().call(),
      ])
      this.players = players;
      this.owner = owner;

    } catch (error) {
      // Catch any errors for any of the above operations.
      alert(
        `Failed to load web3, accounts, or contract. Check console for details.`,
      );
      console.error(error);
    }

  },
  methods: {
    // 参与抽奖
    async participate() {
      await this.contract.methods.participate().send({from: this.accounts[0], value: 0.02 * 1000000000000000000,});
      console.log('true');
    },
    // 取出中奖人
    async pickWinner() {
      await this.contract.methods.pickWinner().send({from: this.accounts[0]});
      console.log('true')
    }
  },
  computed:{
    isOwner() {
      return this.accounts[0] === this.owner;
    }
  }
}
</script>
