<template>
    <div class="composer">
        <textarea v-model="message" @keydown.enter="send" placeholder="Message..."></textarea>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                message: '',
            };
        },
        props: {
            user: {
                type: Object,
                required: true
            },
            contact: {
                type: Object,
                default: null
            },
        },
        watch: {
            message: function (val) {
                Echo.private(`chat`)
                .whisper('typing', {
                    message: this.message.length,
                    id:this.user.id
                });
            }
        },
        methods: {
            send(e) {
                e.preventDefault();
                
                if (this.message == '') {
                    return;
                }
                this.$emit('send', this.message);
                this.message = '';
            }
        },
    }
</script>

<style lang="scss" scoped>
.composer textarea {
    width: 96%;
    margin: 10px;
    resize: none;
    border-radius: 3px;
    border: 1px solid lightgray;
    padding: 6px;
}
</style>