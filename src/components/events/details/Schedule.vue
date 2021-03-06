<template>

    <section class="evlis-schedule-area section_70" data-scroll-index="3" v-if="channelSessions.length">
        <div class="schedule_bg">
            <img src="../../../assets/img/choos_bg.png" alt="schedule bg">
        </div>
        <b-container>
            <b-row>
                <b-col cols="12">
                    <div class="site-heading">
                        <h4>at a Glance</h4>
                        <h2>Event schedule</h2>
                        <p>Consectetur adipisicing elit sed do eiusmod, tempor incididunt labore dolore magna aliqua
                            enim minim veniam quista nostrud exion.</p>
                    </div>
                </b-col>
            </b-row>
            <b-row>
                <b-col cols="12">
                    <div class="offer-tabs">
                        <template>
                            <ul id="offerTab">
                                <li :class="{ active : channel.id === selectChannel }"
                                    :key="channel.id"
                                    class="nav-item"
                                    v-for="channel in channels">
                                    <a @click="changeSelectChannel(channel.id)" class="nav-link">Channel <span>{{ channel.name }}</span></a>
                                </li>
                            </ul>
                            <transition tag="div" class="session-list" name="slide-fade">
                                <div class="row">
                                    <div class="col-md-6" v-for="session in channelSessions"
                                         :key="`session-${session.id}`">
                                        <div class="single-schedule-item">
                                            <div class="overlay-ribbon">
                                                <div class="ribbon-content">{{ currencyFormat(session.cost) }}</div>
                                            </div>
                                            <div class="schedule-details">
                                                <a @click.prevent="selectSession(session)" href="#">
                                                    <h5>{{ session.title }} - {{ session.type }}</h5>
                                                </a>
                                                <div class="schedule-time">
                                                    <p>{{resolveStartEndTime(session.start,session.end)}}</p>
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </transition>
                            <div class="mt-5 text-center">
                                <button @click="resolveScheduleDetail" class="btn-registration btn-second">View Detail
                                </button>
                            </div>
                        </template>
                    </div>
                </b-col>
            </b-row>
            <b-modal id="session_detail" hide-header>
                <template v-if="selectedSession">
                    <h4 class="session-title">{{ selectedSession.title }}</h4>
                    <p class="session-desc">{{ selectedSession.description }}</p>
                    <b-row class="justify-content-between">
                        <b-col cols="4">
                            <p class="ss-detail-item">Speaker: </p>
                            <p class="ss-detail-item">Start: </p>
                            <p class="ss-detail-item">End: </p>
                            <p class="ss-detail-item">Cost: </p>
                        </b-col>
                        <b-col cols="6">
                            <p>{{ speakerFullName }}</p>
                            <p>{{ startTime }}</p>
                            <p>{{ endTime }}</p>
                            <p>{{ sessionCost }}</p>
                        </b-col>
                    </b-row>
                </template>
                <template v-slot:modal-footer>
                    <b-button size="sm" variant="danger" @click="$bvModal.hide('session_detail')">Close</b-button>
                </template>
            </b-modal>
        </b-container>
    </section>
</template>

<script>
    import dateFormatMixin from "../../../mixins/dateFormatMixin";
    import {mapState} from 'vuex';
    import moment from 'moment';
    import currencyFormatMixin from "../../../mixins/currencyFormatMixin";
    import sessionMixin from "../../../mixins/sessionMixin";

    export default {
        name: "Schedule",
        data() {
            return {
                selectedChannel: null,
                selectedSession: null,
            }
        },
        mixins: [dateFormatMixin, currencyFormatMixin, sessionMixin],
        computed: {
            ...mapState({
                event: state => state.event.event,
            }),
            channels() {
                let channels = (this.event && this.event.channels) ? this.event.channels : [];
                if (channels.length === 0) return channels;

                channels = channels.filter(c => {
                    let rooms = c && c.rooms ? c.rooms : null;
                    if (!rooms) return false;
                    for (let i = 0; i < rooms.length; i++) {
                        let sessions = rooms[i].sessions;
                        if (sessions.length > 0) {
                            return true;
                        }
                    }
                });
                return channels;
            },
            selectChannel: {
                set(id) {
                    this.selectedChannel = id;
                },
                get() {
                    if (!this.selectedChannel)
                        this.selectedChannel = this.event.channels[0].id;
                    return this.selectedChannel;
                }
            },
            channelSessions() {
                if (this.channels !== undefined && this.channels !== null) {
                    let index = this.channels.findIndex(c => c.id === this.selectChannel);
                    let sessions = this.getSessionByChannel(this.channels[index]);
                    return !!sessions.length ? sessions : [];
                }
                return [];
            },
            speakerFullName() {
                return this.selectedSession.speaker.firstname + ' ' + this.selectedSession.speaker.lastname;
            },
            startTime() {
                return this.parseTime(this.selectedSession.start);
            },
            endTime() {
                return this.parseTime(this.selectedSession.end);
            },
            sessionCost() {
                return this.currencyFormat(this.selectedSession.cost);
            },
        },
        methods: {
            selectSession(session) {
                this.$bvModal.show('session_detail');
                console.log(session);
                this.selectedSession = session;
            },
            resolveScheduleDetail() {
                this.$router.push({
                    name: 'Schedules',
                    params: {
                        oSlug: this.event.organizer.slug,
                        eSlug: this.event.slug,
                    }
                });
            },
            changeSelectChannel(id) {
                this.selectedChannel = id;
            },
            resolveStartEndTime(start, end) {
                return `${moment(start).format('H:mm')} - ${moment(end).format('H:mm')}`;
            },
        }
    }
</script>

<style scoped lang="scss">

    .session-list {
        max-height: 600px;
        overflow-y: auto;
        overflow-x: hidden;

        .single-schedule-item {
            position: relative;
            padding: 5px;

            .session-desc {
                font-weight: 200;
            }

        }

    }

    .overlay-ribbon {
        position: absolute;
        width: 3em;
        height: 3em;
        top: 10px;
        right: 0;
        background: var(--main);
        padding: 5px 0;
        border-top-right-radius: 8px;
        border-top-left-radius: 8px;

        &:after {
            content: "";
            position: absolute;
            bottom: -2em;
            right: 0;
            width: 0;
            height: 0;
            border-top: 2em solid var(--main);
            border-left: 2em solid transparent;
        }

        &:before {
            content: "";
            position: absolute;
            bottom: -2em;
            left: 0;
            width: 0;
            height: 0;
            border-top: 3em solid var(--main);
            border-right: 3em solid transparent;
            /*z-index: 99;*/
        }

        .ribbon-content {
            position: relative;
            margin: 0 5px;
            text-align: center;
            font-size: 1rem;
            font-weight: 500;
            color: white;
            padding-top: 5px;
        }
    }

    .offer-tabs {
        .row {
            .single-schedule-item {
                .overlay-ribbon {
                    @media screen and (max-width: 575px) {
                        right: 5px;
                    }
                }
            }
        }

        #offerTab {
            .nav-item {
                &.active .nav-link {
                    color: #fff;
                    background-color: var(--second);

                    span {
                        color: #fff;
                        border-color: #fff;
                    }
                }

                .nav-link {
                    cursor: pointer;
                    color: var(--second);
                    border-color: var(--second);

                    span {
                        border-color: var(--second);
                        color: var(--second);
                    }
                }
            }
        }
    }

</style>
