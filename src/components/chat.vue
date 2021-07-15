<template>
    <div class="chat-box" :class="{'clip': clip}">
        <div class="overlay w-100 h-50"></div>
        <div class="messages-container text-white my-3 d-flex align-items-bottom justify-content-stretch">
            <div class="messages mt-auto w-100">
                <div v-for="(data, idx) in messages" :key="idx" class="bubble m-2 p-2 pl-4 ">
                    <div class="d-flex flex-column align-items-start">
                        <p class="p-0 m-0 sender mb-1">{{data.name}}</p>
                        <p class="p-0 m-0">{{data.message}}</p>
                    </div>
                </div>
            </div>
        </div>
        <div class="d-flex flex-row align-items-center">
            <b-textarea class="bg-transparent text-white message-box ml-2 p-2" 
                v-model="message"
                size="sm" 
                max-rows="2" 
                no-resize ></b-textarea>
            <div>
                <i class="far fa-paper-plane h3 text-info rounded-circle" :class="{'m-1': !clip}" @click="send"></i>
                <i class="btn-clip h3 text-info rounded-circle d-flex align-items-center justify-content-center" 
                    :class="{'fa fa-chevron-down m-1': !clip, 'far fa-comment-alt': clip}"
                    style="width: 40px; height: 40px"
                    @click="toggleClip">
                    <b-badge variant="danger msg-badge" v-if="clip && msgCount">{{msgCount}}</b-badge>
                </i>
            </div>
        </div>
    </div>
</template>

<script>
export default {
	name: 'Chat',
    props: ["socket", "roomID", "name", "room"],
	data: ()=>{
		return {
            clip: true,
            message: '',
            msgCount: 0,
            messages: [],
		}
	},
	mounted() {
        this.socket.on('message', (data)=>{
            this.messages.push({
                name: data.name,
                message: data.message,
            });
            this.$forceUpdate();
            setTimeout(()=>{
                document.querySelector('.messages-container').scrollTop = document.querySelector('.messages-container').scrollHeight;
            }, 100);
            if(this.clip)
                this.msgCount++;
        });
    },
    methods: {
        toggleClip() {
            this.clip=!this.clip;
            if(!this.clip)
                this.msgCount=0;
        },
        send() {
            this.message = this.message.trim();
            if(this.message) {
                this.socket.emit('send', {
                    'roomID': this.roomID,
                    'name': this.name,
                    'message': this.message,
                });
                this.message = "";
            }
        },
    }
}
</script>

<style scoped>
.sender {
    color: #17a2b8;
    font-weight: bold;
}
.btn-clip {
    transition: all .5s ease-in-out;
    position: relative;
}
.msg-badge {
    z-index: 3;
    position: absolute;
    bottom: 0;
    right: 0;
    font-size: xx-small;
}
.chat-box {
    background-color: #121212;
    position: fixed;
    bottom: 0em;
    right: 0em;
    box-shadow: 0 2.5rem 1rem rgba(0,0,0,.5) !important;
    clip-path: circle(100%);
    transition: all .25s ease-in-out;
}
.clip {
    clip-path: circle(25px at calc(100% - 15px) calc(100% - 30px));
    transform: translateX(-2em) translateY(-2em);
}
.message-box {
    border-bottom: 1px solid #17a2b8 !important;
}
.messages-container {
    position: relative;
    max-width: 350px;
    width: 100%;
    min-height: 20em;
    max-height: 60vh;
    height: 600px;
    overflow-x: hidden;
    overflow-y: auto;
    scrollbar-width: none;
    -ms-overflow-style: none;
}
@media screen and (max-width: 767.9px) {
    .messages-container {
        max-width: unset;
        width: 100vw;
        height: 100vh;
    }
}
.messages-container::-webkit-scrollbar {
    display: none;
}
.overlay {
    position: absolute;
    background: linear-gradient(#121212, #12121200);
    z-index: 2;
}
.messages {
    text-align: left;
}
.bubble {
    border-radius: 1em;
    background: #1e1e1e;
}
</style>