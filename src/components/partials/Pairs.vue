<template>
    <div class="app__pairs">
        <div class="app__pairs-title">
            <h1>Панель котировок</h1>
        </div>

        <div class="app__pairs-container">
            <div class="app__pairs-titles">
                <div>Валютные пары</div>
                <div>Изменение</div>
                <div>Цена</div>
            </div>

            <div class="app__pairs-list">
                <div
                    v-for="pair in compareData.pairs"
                    :key="pair.id"
                    class="app__pairs-pair"
                    :class="{selected: compareData.selectedPair == pair.id}"
                    @click="compareData.selectedPair = pair.id"
                >
                    <div>{{pair.pair2}} | {{pair.pair1}}</div>
                    <div class="app__pairs-pair-percent">
                        <ArrowUp v-if="pair.change == '+'"/>
                        <ArrowDown v-if="pair.change == '-'"/>

                        <span v-if="pair.percent">{{pair.percent}} %</span>
                    </div>
                    <div>
                        <template v-if="pair.data">{{pair.data.P}}</template>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import ArrowDown from "@/components/partials/SvgIcons/ArrowDown"
import ArrowUp from "@/components/partials/SvgIcons/ArrowUp"

export default {
    name: "Pairs",
    components: {
        ArrowUp,
        ArrowDown},
    props: ['compareData'],
    data: () => ({
        oldData: null
    }),
    watch: {
        compareDataOld: {
            handler: function (newValue, oldValue) {
                this.compareData.pairs.forEach((item) => {
                    let data = item.data
                    let oldData = oldValue.pairs.filter(oldItem => oldItem.id == item.id)
                    oldData = oldData[0].data

                    if(!data || !oldData) return
                    if(data.P == oldData.P) return

                    this.calculatePercent(item, data.P, oldData.P)
                })
            },
            deep: true
        },
    },
    computed:{
        compareDataOld: function(){
            return JSON.parse(JSON.stringify(this.compareData))
        }
    },
    methods: {
        calculatePercent(item, price, oldPrice) {
            if(price > oldPrice) {
                item.change = '+'
                item.percent = (price / oldPrice * 100 - 100).toFixed(5)
            }
            else {
                item.change = '-'
                item.percent = (oldPrice / price * 100 - 100).toFixed(5)
            }
        }
    }
}
</script>