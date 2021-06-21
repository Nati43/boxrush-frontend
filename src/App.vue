<template>
	<div id="app" class="d-flex flex-column align-items-center">
		
		<Box class="px-5" 
			v-if="room && room.count==2" 
			:cols="cols" 
			:rows="rows" 
			:socket="socket" 
			:player="player" 
			:roomID="roomID" 
			:name="name" 
			:room="room" />

		<div v-else-if="player==1" class="d-inline-flex flex-column align-items-center mx-5">
			<!-- Start new game if no room created -->
			<div v-if="!roomID" class="d-inline-flex flex-column align-items-center">
				<div class="logo d-flex align-items-center px-3">
					<div class="boxes">
						<p class="title text-left small m-0 p-0 text-info font-weight-bold h1">▢▣</p>
						<p class="title text-left small m-0 p-0 text-info font-weight-bold h1">▣▢</p>
					</div>
					<span class="text-info h4 font-weight-cold title ml-3 my-auto">Boxes</span>
				</div>
				<p class="h3 font-weight-bold text-light title mt-5">Welcome!</p>
				<p class="pt-3 font-weight-bold text-light">
					This is the boxes game. <br> 
					Please choose grid size, <br>
					enter your first name, and start.
				</p>
				<div class="d-flex align-items-center mt-3 mb-5">
					<b-form-spinbutton v-model="cols" wrap min="4" max="9" vertical class="border-0 bg-transparent text-light font-weight-bold">
						<template #increment> <span class="text-light">↑</span></template>
						<template #decrement> <span class="text-light">↓</span></template>
					</b-form-spinbutton>
					<span class="mx-3 font-weight-bold text-light">x</span>
					<b-form-spinbutton v-model="rows" wrap min="4" max="9" vertical class="border-0 bg-transparent text-light font-weight-bold">
						<template #increment> <span class="text-light">↑</span></template>
						<template #decrement> <span class="text-light">↓</span></template>
					</b-form-spinbutton>
				</div>
				<b-input-group class="d-inline-flex w-75">
					<b-input class="bg-transparent text-light rounded mx-2" type="text" placeholder="First name" v-model="name" />
					<b-button variant="outline-info px-3" @click="start"> Start </b-button>
					<b-form-invalid-feedback class="text-left ml-4" v-if="nameError" :state="false" > First name is required </b-form-invalid-feedback>
				</b-input-group>
			</div>
			<!-- Wait for player-2 -->
			<div v-else>
				<p class="font-weight-bold text-light">Waiting for player 2</p>
				<b-spinner label="Waiting..." variant="info" type="grow"></b-spinner>
				<p class="pt-3 font-weight-bold text-light">Invite a friend to join using the link below.</p>
				<p class="link text-info">https://boxrush.herokuapp.com/?game={{roomID}}</p>
			</div>
		</div>

		<div v-else-if="player==2" class="d-inline-flex flex-column align-items-center m-5">
			<div v-if="!expired">
				<div class="logo d-flex align-items-center px-3">
					<div class="boxes">
						<p class="title text-left small m-0 p-0 text-info font-weight-bold h1">▢▣</p>
						<p class="title text-left small m-0 p-0 text-info font-weight-bold h1">▣▢</p>
					</div>
					<span class="text-info h4 font-weight-cold title ml-3 my-auto">Boxes</span>
				</div>
				<p class="h3 font-weight-bold text-light title">Welcome!</p>
				<p class="pt-3 mt-3 mb-5 font-weight-bold text-light">
					This is the boxes game. <br> 
					Enter your first name and join.
				</p>
				<b-input-group class="d-inline-flex w-75">
					<b-input class="bg-transparent text-light" type="text" placeholder="First name" v-model="name" />
					<b-button variant="outline-info" @click="join"> Join </b-button>
					<b-form-invalid-feedback class="text-left ml-4" v-if="nameError" :state="false" > First name is required </b-form-invalid-feedback>
				</b-input-group>
			</div>
			<div v-else>
				<p class="font-weight-bold text-danger">{{expired}}</p>
				<b-button variant="outline-info" :to="origin"> Start a new one </b-button>
			</div>
		</div>
	</div>
</template>

<script>
/* eslint-disable */ 
import Box from './components/box.vue'
export default {
	name: 'App',
	components: {
		Box
	},
	data: ()=>{
		return {
			host: '',
			name: null,
			nameError: null,
			cols: 5,
			rows: 5,
			socket: null,
			roomID: null,
			origin: location.origin,
			expired: null,
			player: 1,
			room: null,
		}
	},
	mounted() {
		var query = new URLSearchParams(location.search);
		if(query.get('game')) {
			var game = query.get('game');
			this.cols = Number.parseInt(game.split('-')[0]);
			this.rows = Number.parseInt(game.split('-')[1]);
			this.roomID = game;
			this.player = 2;
		}
	},
	methods: {
		start() {
			if(this.name) {
				this.nameError=false;
				this.socket = io.connect(this.host+'/games');
	
				this.socket.on('connect', ()=>{
					console.log('Connected to server.');
					this.socket.emit('createRoom', {
						'cols': this.cols,
						'rows': this.rows,
					});
				});
	
				this.socket.on("roomCreated", (roomID)=>{
					console.log("Room created: ", roomID);
					this.roomID = roomID;
					this.socket.emit("join", {
						'roomID': roomID,
						'name': this.name,
					});
				});
	
				this.socket.on("joined", (room)=>{
					if(room.count == 2) {
						this.room = room;
						setTimeout(this.$forceUpdate, 500);
					}
				});
			}else{
				this.nameError = true;
			}
		},
		join(){
			if(this.name) {
				this.nameError=false;
				this.socket = io(this.host+'/games');
	
				this.socket.on('connect', ()=>{
					console.log('Connected to server.');
					this.socket.emit("join", {
						'roomID': this.roomID,
						'name': this.name,
					});
				});
	
				this.socket.on("joined", (room)=>{
					if(room.count == 2) {
						this.room = room;
						setTimeout(this.$forceUpdate, 500);
					}
				});
	
				this.socket.on("joinError", (msg)=>{
					this.expired = msg;
				});
			}else{
				this.nameError = true;
			}
		},
	}
}
</script>

<style>
html,
body {
	background-color: #121212;
	color: #1F1B24 !important;
}
#app {
	font-family: Helvetica, Arial, sans-serif;
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
	text-align: center;
	color: #2c3e50;
	margin-top: 60px;
}
.title {
	font-family: 'Mitr', sans-serif;
}
</style>