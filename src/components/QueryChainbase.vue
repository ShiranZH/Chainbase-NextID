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
            // balance_data: [
            //     { 'Network': 'Ethereum', 'Balance': 0 },
            //     { 'Network': 'Polygon', 'Balance': 0 },
            //     { 'Network': 'Bnbchain', 'Balance': 0 },
            //     { 'Network': 'Avalanche', 'Balance': 0 },
            //     { 'Network': 'Fantom', 'Balance': 0 },
            //     { 'Network': 'Arbitrum', 'Balance': 0 },
            //     { 'Network': 'Aptos', 'Balance': 0 },
            // ],
            balance_data: {},
            balance_header: 'Network, Balance (Unit Wei)',
            network_id: {
                'Ethereum': 1,
                'Polygon': 137,
                'Bnbchain': 56,
                'Avalanche': 43114,
                'Fantom': 250,
                'Arbitrum': 42161,
                'Aptos': 2,
            },
            nft_data: new Set(),
            tx_data: [],
        }
    },
    methods: {
        async getAccountBalance(wallet_addr) {
            // const idx = 0;
            alert('Query wallet: ' + wallet_addr);
            for (let key in this.network_id) {

                // Refer to https://docs.chainbase.com/reference/getaccountbalance.
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
                //         this.balance_data, idx++,
                //         {
                //             'Network': key,
                //             'Balance': parseInt(response.data.data, 16)
                //         }
                //     )));

                // await axios(options)
                //     .then(response => (this.balance_data[key] =
                //     {
                //         'Network': key,
                //         'Balance': parseInt(response.data.data, 16)
                //     }
                //     ));

                const response = await axios(options);
                this.balance_data[key] =
                {
                    'Network': key,
                    'Balance': parseInt(response.data.data, 16)
                };
                // this.$set(this.balance_data, key,
                //     {
                //         'Network': key,
                //         'Balance': parseInt(response.data.data, 16)
                //     }
                // );

            }
        },

        async getAccountNfts(wallet_addr) {
            alert('Query wallet: ' + wallet_addr);

            // Refer to https://docs.chainbase.com/reference/getaccountnfts.
            const options = {
                url: 'https://api.chainbase.online/v1/account/nfts?chain_id=1&address=' + wallet_addr,
                method: 'GET',
                headers: {
                    'X-API-KEY': '2LWFvlbS8dibH4k7gPEfY4WZ01D',
                    'Content-Type': 'application/json'
                }
            };

            // await axios(options)
            //     .then(response => (this.nft_data.append(response.data.data)));

            const response = await axios(options);
            response.data.data.forEach((item, index) => {
                if (item.image_uri != "") {
                    this.nft_data.add(item.image_uri);
                }
                // alert(`${item.image_uri}`);
            });
            // for (var i = 0; i < response.data.data.length; i++) {
            //     this.nft_data.push(response.data.data[i]);
            // }

        },

        async getAccountTxs(wallet_addr) {
            alert('Query wallet: ' + wallet_addr);

            // Refer to https://docs.chainbase.com/reference/getaccounttransactions.
            const options = {
                url: 'https://api.chainbase.online/v1/account/txs?chain_id=1&address=' + wallet_addr,
                method: 'GET',
                headers: {
                    'X-API-KEY': '2LWFvlbS8dibH4k7gPEfY4WZ01D',
                    'Content-Type': 'application/json'
                }
            };

            const response = await axios(options);
            response.data.data.forEach((item, index) => {
                this.tx_data.push(
                    {
                        "timestamp": item.block_timestamp,
                        "to_addr": item.to_address,
                    });
                // alert(`${item.block_timestamp}`);
            });

        },
    }
}
</script>

<template>
    <h1>{{ msg }}</h1>

    <div class="instruction">
        <p>
            Click to get account portfolio overview.
        </p>
        <p> Please input your wallet address:
            <textarea v-model="wallet_addr" placeholder="Please input your wallet address..."></textarea>
        </p>
    </div>

    <div id="ChainbaseData">
        <p>
            <OneTable :header="balance_header" :body="balance_data"></OneTable>
        </p>
    </div>

    <div class="interaction">
        <!-- <button type="button" @click="count++">count is {{ count }}</button> -->
        <button @click="getAccountBalance(wallet_addr)">Get native token balance</button>
    </div>

    <div id="ChainbaseData">
        <p>
            <li v-for="(item, index) in nft_data" :key="index">
                {{ item }}
            </li>
        </p>
    </div>

    <div class="interaction">
        <button @click="getAccountNfts(wallet_addr)">Get NFTs</button>
    </div>

    <div id="ChainbaseData">
        <p>
            <li v-for="(item, index) in tx_data" :key="index">
                {{ item }}
            </li>
        </p>
    </div>

    <div class="interaction">
        <button @click="getAccountTxs(wallet_addr)">Get account transactions</button>
    </div>
</template>

