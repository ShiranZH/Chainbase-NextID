<script setup>
defineProps({
    msg: String,
})
</script>

<script getChainbaseData>
import axios from 'axios'
import OneTable from './OneTable.vue'

export default {
    name: "ChainbaseData",
    data() {
        return {
            wallet_addr: '0xaeaecd497cd167ab0db77356f36eb9a68e888888',
            // tx_data: [
            //     { 'Network': 'Ethereum', 'Balance': 0 },
            //     { 'Network': 'Polygon', 'Balance': 0 },
            //     { 'Network': 'Bnbchain', 'Balance': 0 },
            //     { 'Network': 'Avalanche', 'Balance': 0 },
            //     { 'Network': 'Fantom', 'Balance': 0 },
            //     { 'Network': 'Arbitrum', 'Balance': 0 },
            //     { 'Network': 'Aptos', 'Balance': 0 },
            // ],
            tx_data: {},
            header: 'Network, Balance (Unit Wei)',
            network_id: {
                'Ethereum': 1,
                'Polygon': 137,
                'Bnbchain': 56,
                'Avalanche': 43114,
                'Fantom': 250,
                'Arbitrum': 42161,
                'Aptos': 2,
            },
        }
    },
    methods: {
        async getAccountTxs(wallet_addr) {
            // const idx = 0;
            alert('Query wallet: ' + wallet_addr);
            for (let key in this.network_id) {

                const options = {
                    url: 'https://api.chainbase.online/v1/account/balance?chain_id=' + this.network_id[key] + '&address=' + wallet_addr,
                    method: 'GET',
                    headers: {
                        'X-API-KEY': '2LWFvlbS8dibH4k7gPEfY4WZ01D',
                        'Content-Type': 'application/json'
                    }
                };

                // TODO - need to verify whether response.data is null or not.
                // await axios(options)
                //     .then(response => (this.$set(
                //         this.tx_data, idx++,
                //         {
                //             'Network': key,
                //             'Balance': parseInt(response.data.data, 16)
                //         }
                //     )));

                await axios(options)
                    .then(response => (this.tx_data[key] =
                        {
                            'Network': key,
                            'Balance': parseInt(response.data.data, 16)
                        }
                    ));

                // const response = axios(options);
                // this.msg = response.data;
                // alert(`${this.response}`);

            }
        },
    }
}
</script>

<template>
    <h1>{{ msg }}</h1>

    <div id="ChainbaseData">
        <OneTable :header="header" :body="tx_data"></OneTable>
    </div>

    <div class="interaction">
        <!-- <button type="button" @click="count++">count is {{ count }}</button> -->
        <p>
            <textarea v-model="wallet_addr" placeholder="Please input your wallet address..."></textarea>
        </p>
        <button @click="getAccountTxs(wallet_addr)">Get Account Transactions</button>
    </div>

    <div class="instruction">
        <p>
            Click to get account portfolio overview.
        </p>
    </div>
</template>

