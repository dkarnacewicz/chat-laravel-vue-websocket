<template>
    <div class="row">
        <div class="col-lg-8">
            <div class="card card-default">
                <div class="card-header">Wiadomości</div>
                <div class="card-body p-0">
                    <ul class="list-unstyled list-scroll" v-chat-scroll>
                        <li class="p-2" v-for="(message, index) in messages" :key="index">
                            <strong>{{ message.user.name }}</strong>
                            {{ message.message }}
                        </li>
                    </ul>
                </div>
            </div>
             <input
                type="text"
                name="message"
                placeholder="Napisz coś..."
                class="form-control"
                v-model="message"
                @keyup.enter="sendMessage"
                @keydown="sendTypingEvent" />
                <span class="text-muted" v-if="activeUser">{{ activeUser.name }} piszę...</span>
        </div>
        <div class="col-lg-4">
            <div class="card card-default">
                <div class="card-header">Aktywny użytkownicy</div>
                <div class="card-body">
                    <ul>
                        <li class="py-2 color__green" v-for="(user, index) in users" :key="index">
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
        props: {
            user: {
                type: Object
            }
        },
        data() {
            return {
                messages: [],
                message: '',
                users: [],
                activeUser: false,
                typingTimer: false,
            }
        },
        created() {
            this.fetchMessages();

            Echo.join('chat')
                .here(users => {
                    this.users = users;
                })
                .joining(user => {
                    this.users.push(user);
                })
                .leaving(user => {
                    this.users = this.users.filter(u => u.id !== user.id);
                })
                .listen('MessageSent', (event) => {
                    this.messages.push(event.message)
                })
                .listenForWhisper('typing', user => {
                    this.activeUser = user;

                    if (this.typingTimer) {
                        clearTimeout(this.typingTimer);
                    }

                    this.typingTimer = setTimeout(() => {
                        this.activeUser = false;
                    }, 3000);
                });
        },
        methods: {
            fetchMessages() {
                axios.get('messages').then(response => {
                    this.messages = response.data;
                });
            },
            sendMessage() {
                this.messages.push({
                    user: this.user,
                    message: this.message
                })
                axios.post('messages', { message: this.message });
                this.message = '';
            },
            sendTypingEvent() {
                Echo.join('chat')
                    .whisper('typing', this.user);
            },
        }
    }
</script>

<style scoped lang="scss">
$success: green;

.list-scroll {
    height: 300px;
    overflow-y: auto;
}

.color {
    &__green {
        color: $success;
    }
}

</style>
