<template>
	<div id="app" class="d-flex flex-column align-items-center">
		
		<Box class="px-5" 
			v-if="started" 
			:id="id"
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
					<span class="text-info h4 font-weight-cold title ml-3 my-auto">BoxRush</span>
				</div>
				<p class="h3 font-weight-bold text-light title mt-3">Welcome!</p>
				<p class="pt-3 font-weight-bold text-light">
					This is the BoxRush game. <br> 
					Please choose grid size, number of players <br>
					enter your first name, and start.
				</p>
				<p class="pt-3 font-weight-bold text-light">Grid Size</p>
				<div class="d-flex align-items-center">
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
				<p class="pt-3 font-weight-bold text-light">Number of Players</p>
				<div class="d-flex align-items-center mb-5">
					<b-form-spinbutton v-model="numberOfPlayers" wrap min="2" max="4" vertical class="border-0 bg-transparent text-light font-weight-bold">
						<template #increment> <span class="text-light">↑</span></template>
						<template #decrement> <span class="text-light">↓</span></template>
					</b-form-spinbutton>
				</div>
				<b-input-group class="d-inline-flex w-75">
					<b-input @keypress.enter="start" class="bg-transparent text-light rounded mx-2" type="text" placeholder="First name" v-model="name" />
					<b-button variant="outline-info px-3" @click="start"> Start </b-button>
					<b-form-invalid-feedback class="text-left ml-4" v-if="nameError" :state="false" > First name is required </b-form-invalid-feedback>
				</b-input-group>
			</div>
			<!-- Wait for player-2 -->
			<div v-else>
				<p class="font-weight-bold text-light">Waiting for other player<span v-if="numberOfPlayers>2">s</span></p>
				<b-spinner label="Waiting..." variant="info" type="grow"></b-spinner>
				<p class="pt-3 font-weight-bold text-light">Invite a friend to join using the link below.</p>
				<p class="link text-info">https://boxrush.herokuapp.com/?game={{roomID}}</p>
			</div>
		</div>

		<div v-else-if="player==2" class="d-inline-flex flex-column align-items-center m-5">
			<div v-if="!expired && !joined">
				<div class="logo d-flex align-items-center px-3">
					<div class="boxes">
						<p class="title text-left small m-0 p-0 text-info font-weight-bold h1">▢▣</p>
						<p class="title text-left small m-0 p-0 text-info font-weight-bold h1">▣▢</p>
					</div>
					<span class="text-info h4 font-weight-cold title ml-3 my-auto">BoxRush</span>
				</div>
				<p class="h3 font-weight-bold text-light title">Welcome!</p>
				<p class="pt-3 mt-3 mb-5 font-weight-bold text-light">
					This is the BoxRush game. <br> 
					Enter your first name and join.
				</p>
				<b-input-group class="d-inline-flex w-75">
					<b-input @keypress.enter="join" class="bg-transparent text-light" type="text" placeholder="First name" v-model="name" />
					<b-button variant="outline-info" @click="join"> Join </b-button>
					<b-form-invalid-feedback class="text-left ml-4" v-if="nameError" :state="false" > First name is required </b-form-invalid-feedback>
				</b-input-group>
			</div>
			<div v-else-if="!expired && joined">
				<p class="font-weight-bold text-light">Waiting for other player<span v-if="numberOfPlayers>2">s</span></p>
				<b-spinner label="Waiting..." variant="info" type="grow"></b-spinner>
			</div>
			<div v-else-if="expired">
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
			numberOfPlayers: 2,
			nameError: null,
			cols: 5,
			rows: 5,
			socket: null,
			roomID: null,
			origin: location.origin,
			expired: null,
			player: 1,
			room: null,
			started: false,
			joined: false,
			id: null,
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
						'numberOfPlayers': this.numberOfPlayers,
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
					this.room = room;
				});

				this.socket.on("id", (id)=>{
					this.id = id;
				});

				this.socket.on("start", ()=>{
					this.started = true;
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
					this.room = room;
					this.joined = true;
				});

				this.socket.on("id", (id)=>{
					this.id = id;
				});

				this.socket.on("start", ()=>{
					this.started = true;
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