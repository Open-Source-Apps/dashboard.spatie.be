<template>
    <grid :position="grid" modifiers="overflow transparent">
        <section class="tweets">

            <div class="tweets__icon" v-if="!onDisplay.length">
            </div>

            <div class="tweet" v-for="tweet in onDisplay">
                <div class="tweet__header">
                    <div class="tweet__avatar"
                         :style="'background-image: url('+ tweet.authorAvatar +')'"></div>
                    <div class="tweet__user">
                        <div class="tweet__user__name">
                            {{ tweet.authorName }}
                        </div>
                        <div class="tweet__user__handle">
                            {{ tweet.authorScreenName }}
                        </div>
                    </div>
                </div>
                <div
                    :class="addClassModifiers('tweet__body', tweet.displayClass)"
                    v-html="tweet.html"
                ></div>
                <div class="tweet__meta">
                    <relative-date :moment="tweet.date"></relative-date>
                </div>

                <div v-if="tweet.image" class="tweet__attachment">
                    <img :src="tweet.image" />
                </div>
            </div>
        </section>
    </grid>
</template>

<script>
    import echo from '../mixins/echo';
    import Grid from './Grid';
    import RelativeDate from './RelativeDate';
    import Tweet from '../services/twitter/Tweet';
    import moment from 'moment';
    import {diffInSeconds, addClassModifiers} from '../helpers';
    import { map } from 'lodash';

    export default {

        components: {
            Grid,
            RelativeDate,
        },

        mixins: [echo],

        props: ['grid', 'initialTweets'],

        data() {
            return {
                displayingTopTweetSince: new moment(),
                onDisplay: [],
                waitingLine: [],
            };
        },

        created() {
            this.onDisplay = JSON.parse(this.initialTweets).map(tweetProperties => new Tweet(tweetProperties));

            setInterval(this.processWaitingLine, 1000);
        },

        methods: {
            addClassModifiers,

            getEventHandlers() {
                return {
                    'Twitter.Mentioned': response => {
                        this.addToWaitingLine(new Tweet(response.tweetProperties))
                    },
                };
            },

            addToWaitingLine(tweet) {
                this.waitingLine.push(tweet)
            },

            processWaitingLine() {
                if (this.waitingLine.length === 0) {
                    return;
                }

                if (diffInSeconds(this.displayingTopTweetSince) < 5) {
                    return;
                }

                this.onDisplay.unshift(this.waitingLine.shift());

                this.onDisplay = this.onDisplay.slice(0,10);

                this.displayingTopTweetSince = new moment();
            },

            getSaveStateConfig() {
                return {
                    cacheKey: `twitter`,
                };
            },
        },
    };
</script>
