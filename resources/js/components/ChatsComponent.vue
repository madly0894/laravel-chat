<template>
    <div class="row" style="height: calc(100vh - 103px)">

        <div class="col-md-8 h-100 col-12">
            <div class="card card-default h-100">
                <div class="card-header">Messages</div>
                <div class="card-body p-0">
                    <ul class="mb-0 list-unstyled h-100 w-100 overflow-auto" v-chat-scroll>
                        <li
                            class="m-2 p-2"
                            v-for="(message, index) in messages"
                            :key="index"
                            style="background-color: #009687; color: #fff; border-radius: 5px;"
                        >
                            <strong>{{ message.user.name }}</strong>
                            <p class="my-2">{{ message.message }}</p>
                            <span v-if="message.created_at">
                                {{ message.created_at | moment("h:mm a") }}
                            </span>
                            <span v-else>
                                {{ new Date() | moment("h:mm a") }}
                            </span>
                        </li>
                    </ul>
                </div>

                <div class="input-group">
                    <label for="msg"/>
                    <input
                        id="msg"
                        @keydown="sendTypingEvent"
                        @keyup.enter="sendMessage"
                        v-model="newMessage"
                        type="text"
                        name="message"
                        placeholder="Type your message here..."
                        class="form-control"
                        style="border: none; border-radius: 0; border-top: 1px solid rgba(0,0,0,0.1)"
                    />
                    <span class="input-group-btn">
                        <button class="btn btn-primary" id="btn-chat" @click="sendMessage">
                            Send
                        </button>
                    </span>
                </div>
            </div>
            <span class="text-muted" v-if="activeUser" >{{ activeUser.name }} is typing...</span>
        </div>

        <div class="col-md-4 col-12 mt-md-0 mt-5 h-100">
            <div class="card card-default h-100">
                <div class="card-header">Active Users</div>
                <div class="card-body">
                    <ul class="mb-0">
                        <li class="py-2" v-for="(user, index) in users" :key="index">
                            {{ user.name }}
                        </li>
                    </ul>
                </div>
            </div>
        </div>

    </div>
</template>

<script>
    export default {

        props:['user'],

        data() {
            return {
                messages: [],
                newMessage: '',
                users:[],
                activeUser: false,
                typingTimer: false,
            }
        },

        created() {
            this.fetchMessages();

            Echo.join('chat')
                .here(user => {
                    this.users = user;
                })
                .joining(user => {
                    this.users.push(user);
                })
                .leaving(user => {
                    this.users = this.users.filter(u => u.id !== user.id);
                })
                .listen('MessageSent',(event) => {
                    this.messages.push(event.message);

                    this.activeUser = event;
                    this.activeUser = false;
                })
                .listenForWhisper('typing', user => {
                    this.activeUser = user;

                    if(this.typingTimer) {
                        clearTimeout(this.typingTimer);
                    }

                    this.typingTimer = setTimeout(() => {
                        this.activeUser = false;
                    }, 900);
                });
        },

        methods: {
            fetchMessages() {
                axios.get('messages').then(response => {
                    this.messages = response.data;
                });
            },

            sendMessage() {

                if(this.newMessage) {
                    this.messages.push({
                        user: this.user,
                        message: this.newMessage
                    });

                    axios.post('messages', {message: this.newMessage});
                }

                this.newMessage = '';
            },

            sendTypingEvent() {
                Echo.join('chat')
                    .whisper('typing', this.user);
            }
        }
    }
</script>
