<template>
    <div class="app">
        <div class="app__container">
            <div class="app__header">
                <Logo/>
            </div>

            <div class="app__content">
                <Pairs :compareData="compareData"/>

                <div class="app__pairs-title app__pairs-title_mobile">
                    <div class="h1">Панель котировок</div>
                </div>

                <Amcharts :compareData="compareData" :apiKey="apiKey"/>
            </div>
        </div>
    </div>
</template>

<script>
import Amcharts from "@/components/partials/Amcharts"
import Logo from "@/components/partials/Logo"
import Pairs from "@/components/partials/Pairs";

export default {
    name: "Main",
    components: {
        Pairs,
        Logo,
        Amcharts
    },
    data: () => ({
        apiKey: '1d681dc9f68f6fb3d48fe8caf59f58a63e7e054c392c5962fc3270928848bf35',
        compareData: {
            pairs: [
                {
                    id: 1,
                    pair1: 'BTC',
                    pair2: 'USD',
                    sub: "0~Coinbase~BTC~USD",
                    data: null,
                    change: null,
                    percent: 0
                },
                {
                    id: 2,
                    pair1: 'BCH',
                    pair2: 'USD',
                    sub: "0~Coinbase~BCH~USD",
                    data: null,
                    change: null,
                    percent: 0
                },
                {
                    id: 3,
                    pair1: 'ETH',
                    pair2: 'USD',
                    sub: "0~Coinbase~ETH~USD",
                    data: null,
                    change: null,
                    percent: 0
                },
                {
                    id: 4,
                    pair1: 'LTC',
                    pair2: 'USD',
                    sub: "0~Coinbase~LTC~USD",
                    data: null,
                    change: null,
                    percent: 0
                }
            ],
            selectedPair: 1
        }
    }),
    methods: {
        getData() {
            const vm = this
            let ccStreamer = new WebSocket('wss://streamer.cryptocompare.com/v2?api_key=' + this.apiKey)
            let subs = []

            this.compareData.pairs.forEach((item) => {
                subs.push(item.sub)
            })

            ccStreamer.onopen = function onStreamOpen() {
                let subRequest = {
                    action: 'SubAdd',
                    subs
                };
                ccStreamer.send(JSON.stringify(subRequest))
            }

            function responseData(event) {
                // console.log(JSON.parse(event.data))
                let data = JSON.parse(event.data)

                vm.compareData.pairs.forEach((item) => {
                    if(data.FSYM == item.pair1 && data.TSYM == item.pair2 && data.TYPE === '0') {
                        item.data = data
                    }
                })
            }

            ccStreamer.onmessage = function (event) {
                responseData(event)
            }
        }
    },
    created() {
        this.getData()
    }
}
</script>

<style lang="scss">
    @import "../assets/scss/styles.scss";
</style>