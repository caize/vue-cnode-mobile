<template>
    <div>
        <nv-head></nv-head>
        <div class="topic" v-if="topicInfo.title">
            <h2 class="topic-title" v-text="topicInfo.title"></h2>

            <section class="author-info">
                <img :src="topicInfo.author.avatar_url" alt="" class="avatar">
                <div class="center">
                    <span class="author" v-text="topicInfo.author.loginname"></span>
                    <time class="info">
                        发布于:{{topicInfo.create_at | getTimeInfo}}
                    </time>
                </div>
                <div class="right">
                    <span class="tag" v-text="getTabInfo(topicInfo)"
                        :class="{color: topicInfo.good || topicInfo.top}"></span>
                    <span class="name">{{topicInfo.visit_count}}次浏览</span>
                </div>
            </section>

            <section class='markdown-body topic-content' v-html="topicInfo.content">
            </section>


            <section class="topic-reply">
                <h3 class="topic-total">
                    <strong>{{topicInfo.reply_count}}</strong> 回复
                </h3>

                <ul class="reply-list">
                    <li v-for="item in topicInfo.replies">
                        <section class="user">
                            <router-link :to="{name:'user',params:{loginname:item.author.loginname}}" >
                                <img class="head" :src="item.author.avatar_url"/>
                            </router-link>
                            <div class="info">
                                <span class="left">
                                    <span class="name" v-text="item.author.loginname"></span>
                                    <span class="name">
                                        发布于:{{item.create_at | getTimeInfo}}
                                    </span>
                                </span>
                                <span class="right">
                                    <span style="margin-right: 5px" class="iconfont icon-dianzan"></span>
                                    <span style="margin-right: 5px">{{item.ups.length}}</span>
                                    <span class="iconfont icon-hf" @click="addReply(item.id)"></span>
                                </span>
                            </div>
                        </section>
                        <div class="reply-content markdown-body" v-html="item.content"></div>
                        <nv-reply v-if="userInfo.loginname && replyId === item.id"
                            :reply-to="item.author.loginname"
                            :reply-id="item.id"
                            :topic-id="topicId"
                            @onReply="handleReply"></nv-reply>
                    </li>
                </ul>
            </section>
            <nv-reply v-if="userInfo.loginname"
                :topic-id="topicId"></nv-reply>
        </div>
    </div>
</template>

<script>
    import { mapState } from 'vuex';
    import nvHead from '../components/header';
    import nvReply from '../components/reply';
    import {GET_TOPIC_INFO} from '../constants/mutationTypes';
    import {getTimeInfo} from '../utils/index';
    import {topicTab} from '../constants/topicInfo';
    export default {
        data() {
            return {
                replyId: '',
                topicId: '',
            }
        },

        mounted() {
            this.topicId = this.$route.params.id;
            this.getTopicInfo();
        },

        methods: {
            getTopicInfo() {
                this.$store.dispatch(GET_TOPIC_INFO, this.topicId);
            },

            getTabInfo(item) {
                let tab = item.tab;
                if (item.good) {
                    tab = 'good';
                } else if (item.top) {
                    tab = 'top';
                }
                return topicTab[tab]
            },

            addReply(id) {
                if (!this.userInfo.loginname) {
                    this.$router.push({
                        name: 'login',
                        query: { redirect: encodeURIComponent(this.$route.fullPath) }
                    });
                }
                this.replyId = id;
            },

            handleReply() {
                this.replyId = '';
            }
        },

        filters: {
            getTimeInfo(str) {
                return getTimeInfo(str)
            }
        },

        computed: {
            ...mapState(['topicInfo', 'userInfo'])
        },

        components: {
            nvHead,
            nvReply
        }

    }
</script>