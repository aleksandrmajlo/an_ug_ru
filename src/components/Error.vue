<template>
    <div class="alert_error">
        <div class="title">
            <span>Записи не добавлены из-за отсутствия :</span><span class="strong">{{title}}</span>
        </div>
        <div>ID:
            <span class="strong" v-for="(item) in data" :key="item"> {{item}}, </span>
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
    
    &>div{
        margin-bottom: 10px;
    }
    .strong{
         margin-left: 5px;
         font-size: 110%;
         font-weight: bold;
    }
}
</style>


