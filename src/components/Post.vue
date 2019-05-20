<template>
    <div class="postBlock">

        <VuePerfectScrollbar class="scroll-area" :settings="settings">
            <div class="title">{{title}}: <span class="strong">{{data.length}}</span></div>
            <ul>
                <li v-for="(item, index) in data" :key="index">
                  <strong> {{index+1}}.</strong>  <a :href="item.link" target="__blank">{{item.title}} </a>
                </li>
            </ul>
        </VuePerfectScrollbar>
    </div>
</template>

<script>
    import VuePerfectScrollbar from 'vue-perfect-scrollbar'

    export default {
        props: ['title', 'data'],
        components: {
            VuePerfectScrollbar
        },
        data() {
            return {
                settings: {
                    maxScrollbarLength: 60
                }
            }
        },
    }
</script>

<style lang="scss" scoped>
    .postBlock {
        background: blue;
        color: #fff;
        margin-bottom: 20px;
        padding: 20px;
        .title {
            text-align: center;
            font-weight: 700;
            font-size: 120%;
            text-transform: uppercase;
        }
        ul {
            padding: 0;
            li {
                display: block;
                margin-bottom: 10px;
                padding-left: 10px;
            }
        }
        a {
            color: #fff;
        }
        .strong {
            margin-left: 5px;
            font-size: 110%;
            font-weight: bold;
        }
        .scroll-area {
            position: relative;
            margin: auto;
            height: 500px;
        }
    }
</style>

