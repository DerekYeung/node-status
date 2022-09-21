<script setup>
// This starter template is using Vue 3 <script setup> SFCs
// Check out https://vuejs.org/api/sfc-script-setup.html#script-setup
import HelloWorld from './components/HelloWorld.vue'
</script>

<template>
  <div>
    <input v-model="ip" />
    <button @click="watchNode(ip)">监听</button>
    <button @click="watchNode('192.168.1.201')">监听1号</button>
    <button @click="watchNode('192.168.1.202')">监听2号</button>
    <button @click="watchNode('192.168.1.203')">监听3号</button>
    <button @click="watchNode('192.168.1.204')">监听4号</button>
    <div class="nodes">
      <div class="node" v-for="(item, index) in watchList" :key="index">
        <p>
          {{item.ip}}
        </p>
        <p>
          区块号：{{item.blockNumber}}
        </p>
        <p>
          同步状态：{{item.isSyncing}}
        </p>
        <p>
          Peer：{{item.peer}}
        </p>
        <p>共识客户端：{{JSON.stringify(item.cs)}}</p>
        <button @click="unwatchNode(item.ip)">取消监听</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}

.node {
  border: 1px solid white;
}
</style>
<script>
import { defineComponent } from 'vue';
import Web3 from 'web3';

export default defineComponent({
  data() {
    return {
      ip: '',
      watchList: {}
    }
  },
  mounted() {
  },
  methods: {
    unwatchNode(ip) {
      const node = this.watchList[ip];
      delete this.watchList[ip];
      if (node.timer) {
        clearTimeout(node.timer);
      }
    },
    async watchNode(ip) {
      if (!ip) {
        return;
      }
      let node = this.watchList[ip];
      if (!node) {
        const web3 = new Web3(new Web3.providers.WebsocketProvider(`ws://${ip}:8892`));
        node = {
          ip,
          web3,
          blockNumber: 0,
          isSyncing: false,
          peer: 0,
          cs: null
        };
        this.watchList[ip] = node;
      }
      await this.getGethStatus(node);
      await this.getCsStatus(node);
      node.timer = setTimeout(() => {
        this.watchNode(ip);
      }, 1000);
      this.watchList[ip] = node;
    },
    async getGethStatus(node) {
      try {
        const blockNumber = await node.web3.eth.getBlockNumber();
        const isSyncing = await node.web3.eth.isSyncing();
        const peer = await node.web3.eth.net.getPeerCount();
        node.blockNumber = blockNumber || 0;
        node.isSyncing = isSyncing;
        node.peer = peer || 0;
      } catch (e) {
        console.log(e.message);
        return null;
      }
    },
    async getCsStatus(node) {
      try {
        const response = await fetch(`http://${node.ip}:3500/eth/v1/node/syncing`);
        const json = await response.json();
        node.cs = json;
        return json;
      } catch (e) {
        return null;
      }
    }
  }
})
</script>
