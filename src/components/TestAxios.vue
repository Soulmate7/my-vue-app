<template>
    <div>
        <p JSON.stringify(ldh.json)></p>
    </div>
</template>

<script>
import axios from "axios";
export default{
    data(){
        return {
            ldh:[],
        };
    },
    mounted(){
        axios.get("./ldh.json").then((res)=>{
            this.ldh=res.data;
            console.log(this.ldh);
        });
    }
}
</script>
