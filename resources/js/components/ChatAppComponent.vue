<template>
    <div class="chat-app">
        <Conversation :contact="selectedContact" :messages="messages" @new="saveNewMessage" :user="user"/>
        <ContactsList :contacts="contacts" @selected="startConversationWith"/>
    </div>
</template>

<script>
    import Conversation from './ConversationComponent';
    import ContactsList from './ContactsListComponent';
    export default {
        props: {
            user: {
                type: Object,
                required: true
            }
        },
        data() {
            return {
                selectedContact: null,
                messages: [],
                contacts: []
            };
        },
        mounted() {
            Echo.private(`messages.${this.user.id}`)
            .listen('NewMessage', (e) => {
                this.hanleIncoming(e.message);
            });

            Echo.join(`chat`)
                .here((users) => {
                    console.log(users)
                })
                .joining((user) => {
                    console.log(user);
                    this.$toasted.success('joined',{duration:3000});
                })
                .leaving((user) => {
                    console.log(user);
                    this.$toasted.info('left',{duration:3000});
                });
            axios.get('/contacts')
            .then((response) => {
                this.contacts = response.data;
            });
        },
        methods: {
            startConversationWith(contact) {
                this.updateUnreadCount(contact, true);
                axios.get(`/conversation/${contact.id}`)
                    .then((response) => {
                        this.messages = response.data;
                        this.selectedContact = contact;
                    })
            },
            saveNewMessage(message) {
                this.messages.push(message);
            },
            hanleIncoming(message) {
                if (this.selectedContact && message.from == this.selectedContact.id) {
                    axios.get(`/conversation/${this.selectedContact.id}`)
                    .then((response) => {
                        this.messages = response.data;
                    })
                    return;
                }
                this.updateUnreadCount(message.from_contact, false);
            },
            updateUnreadCount(contact, reset) {
                this.contacts = this.contacts.map((single) => {
                    if (single.id !== contact.id) {
                        return single;
                    }
                    if (reset)
                        single.unread = 0;
                    else
                        single.unread += 1;
                    return single;
                })
            }
        },
        components: {Conversation, ContactsList}
    }
</script>


<style lang="scss" scoped>
.chat-app {
    display: flex;
}
</style>