<template>
    <div class="chat-box" :class="{'clip': clip}">
        <div class="overlay w-100 h-50"></div>
        <div class="messages-container text-white my-3 d-flex align-items-bottom justify-content-stretch">
            <div class="messages mt-auto w-100">
                <div v-for="(data, idx) in messages" :key="idx" class="bubble m-2 p-2 pl-4 ">
                    <div class="d-flex flex-column align-items-start">
                        <p class="p-0 m-0 sender mb-1" :class="{'align-self-end': data.id==id}">{{data.name}}</p>
                        <p class="p-0 m-0">{{data.message}}</p>
                    </div>
                </div>
                <p v-if="typing.length>0" class="bubble sender m-2 p-2 pl-4">
                    <span v-text="typing.map((item)=>{return item.name}).join(', ')"></span>
                    <span v-text="typing.length>1 ? ' are typing':' is typing'"></span>
                    <span id="dots"></span>
                </p>
            </div>
        </div>
        <div class="d-flex flex-row align-items-center">
            <b-textarea class="bg-transparent text-white message-box ml-2 p-2" 
                v-model="message"
                @focus="startTyping"
                @blur="stopTyping"
                size="sm" 
                max-rows="2" 
                no-resize ></b-textarea>
            <div>
                <i class="far fa-paper-plane h3 text-info rounded-circle" :class="{'m-1': !clip}" @click="send"></i>
                <i class="btn-clip h3 text-info rounded-circle d-flex align-items-center justify-content-center" 
                    :class="{'fa fa-chevron-down m-1': !clip, 'far fa-comment-alt': clip, 'glow': (clip && typing.length>0)}"
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
    props: ["socket", "id", "roomID", "name", "room"],
	data: ()=>{
		return {
            clip: true,
            message: '',
            msgCount: 0,
            messages: [],
            typing: [],
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
        this.socket.on('start_typing', (data)=>{
            if(data.id != this.id)
                this.typing.push({id: data.id, name: data.name});
        });
        this.socket.on('stop_typing', (data)=>{
            this.typing.splice(this.typing.indexOf(x=>x.id == data.id), 1);
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
                    'id': this.id,
                    'name': this.name,
                    'message': this.message,
                });
                this.message = "";
            }
        },
        startTyping(){
            this.socket.emit('start_typing', {
                'roomID': this.roomID,
                'id': this.id,
                'name': this.name,
            });
        },
        stopTyping(){
            this.socket.emit('stop_typing', {
                'roomID': this.roomID,
                'id': this.id,
                'name': this.name,
            });
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
    box-shadow: unset !important;
    transform: translateX(-2em) translateY(-2em);
}
.message-box {
    border-bottom: 1px solid #17a2b8 !important;
}
.messages-container {
    position: relative;
    min-width: 350px;
    max-width: 350px;
    width: 100%;
    max-height: 100vh;
    min-height: 60vh;
    overflow-x: hidden;
    overflow-y: auto;
    scrollbar-width: none;
    -ms-overflow-style: none;
}
@media screen and (max-width: 768px) {
    .messages-container {
        max-width: unset;
        width: 100vw;
        min-height: 100vh;
        max-height: 100vh;
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
@keyframes typing {
    0% {
        content: ".";
    }
    40% {
        content: "..";
    }
    70% {
        content: "...";
    }
}
#dots::after {
	content: "";
    animation-name: typing;
    animation-duration: 1s;
    animation-iteration-count: infinite;
}
@keyframes glowing {
    0% {
		display: block;
        transform: scale(.9);
		text-shadow: 0 0 0 #17a2b8;
    }
    50% {
		display: block;
        transform: scale(1.1);
		text-shadow: 0 0 .25rem #17a2b8;
    }
    100% {
		display: block;
        transform: scale(.9);
		text-shadow: 0 0 0 #17a2b8;
    }
}
.glow {
	animation-name: glowing;
    animation-duration: .5s;
    animation-iteration-count: infinite;
}
</style>