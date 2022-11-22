<template>
    <div>
        <BeeChart v-if="dataExists" :myBeechartData="myBeeData" /> 
        <!-- <h1>Welcome Home</h1>
        <BarChart v-if="dataExists" :myBarchartData="myBarData" /> -->

    </div>
</template>

<script>
// import BarChart from "../components/barchart.vue"
import BeeChart from "../components/beeswarm.vue"
import * as d3 from "d3";
import csvPath from '../../assets/data/SF_Historical_Ballot_Measures.csv';

export default {
    data(){
        return {
            dataExists: false,
            myBeeData: [],
        }
    },
    components: {
        BeeChart
    },
    created(){
        /* Fetch via CSV */
        this.drawBeeFromCsv()
    },
    mounted(){},
    methods: {
        drawBeeFromCsv(){
            //async method
            d3.csv(csvPath).then((data) => {
                // array of objects
                // console.log(data.length);
                // console.log(data);
                this.dataExists = true; // updates the v-if to conditionally show the barchart only if our data is here.
                this.myBeeData = data; // updates the prop value to be the recieved data, which we hand in to our bar-chart

            });
        }
    }
}

</script>
