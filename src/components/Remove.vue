<template>
    <div class="alert_error">
        <div class="title">
            <span>Удалено </span><span class="strong">{{data.lengt}}</span>
        </div>

        <div>
             <div>Название:</div>
             <ul>
                 <li v-for="(item) in data" :key="item">{{item}}</li>
             </ul>
        </div>
    </div>
</template>
<script>
export default {
    props:['title','data']
}
</script>
<style lang="scss" scoped>
.alert_error{
    display: inline-block;
    padding: 20px;
    background: red;
    color:#fff;
    margin-bottom: 20px;
   
    .title{
         text-align: center;
         font-weight: 700;
         font-size: 120%;
         text-transform: uppercase;
    }
    .strong{
         margin-left: 5px;
         font-size: 110%;
         font-weight: bold;
    }
}
</style>


