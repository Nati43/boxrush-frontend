<template>
	<div id="app" class="d-flex flex-column align-items-center">
		
		<Box class="px-5" v-if="joined==2" :cols="cols" :rows="rows" :socket="socket" :player="player" :roomID="roomID" :name="name" />

		<div v-else-if="player==1" class="d-inline-flex flex-column align-items-center m-5">
			<!-- Start new game if no room created -->
			<div v-if="!roomID" class="d-inline-flex flex-column align-items-center">
				<p class="h3 font-weight-bold text-muted">Welcome!</p>
				<p class="pt-3 font-weight-bold text-muted">
					This is the boxes game. <br> 
					Please choose grid size, <br>
					enter your first name, and start.
				</p>
				<div class="d-flex align-items-center mt-3 mb-5">
					<b-form-spinbutton v-model="cols" wrap min="5" max="9" vertical></b-form-spinbutton>
					<span class="mx-3 font-weight-bold">x</span>
					<b-form-spinbutton v-model="rows" wrap min="5" max="9" vertical></b-form-spinbutton>
				</div>
				<b-input-group class="d-inline-flex w-75">
					<b-input type="text" placeholder="First name" v-model="name" />
					<b-button variant="outline-info" @click="start"> Start </b-button>
					<b-form-invalid-feedback class="text-left ml-4" v-if="nameError" :state="false" > First name is required </b-form-invalid-feedback>
				</b-input-group>
			</div>
			<!-- Wait for player-2 -->
			<div v-else>
				<p class="font-weight-bold text-muted">Waiting for player 2</p>
				<b-spinner label="Waiting..." variant="info" type="grow"></b-spinner>
				<p class="pt-3 font-weight-bold text-muted">Invite a friend to join using the link below.</p>
				<p class="link text-info">{{host}}?game={{roomID}}</p>
			</div>
		</div>

		<div v-else-if="player==2" class="d-inline-flex flex-column align-items-center m-5">
			<div v-if="!expired">
				<p class="h3 font-weight-bold text-muted">Welcome!</p>
				<p class="pt-3 mt-3 mb-5 font-weight-bold text-muted">
					This is the boxes game. <br> 
					Enter your first name and join.
				</p>
				<b-input-group class="d-inline-flex w-75">
					<b-input type="text" placeholder="First name" v-model="name" />
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
			host: '192.168.1.11:3000',
			name: null,
			nameError: null,
			cols: 5,
			rows: 5,
			socket: null,
			roomID: null,
			joined: 0,
			origin: location.origin,
			expired: null,
			player: 1,
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
			this.joined = 1;
			// this.join();
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
					this.socket.emit("join", this.roomID);
				});
	
				this.socket.on("joined", ()=>{
					this.joined++;
					console.log("Joined room ", this.roomID);
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
					this.socket.emit("join", this.roomID);
				});
	
				this.socket.on("joined", ()=>{
					this.joined++;
					console.log("Joined room ", this.roomID);
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
#app {
	font-family: Tahoma, Avenir, Helvetica, Arial, sans-serif;
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
	text-align: center;
	color: #2c3e50;
	margin-top: 60px;
}
</style>