<template>
    <div class="d-inline-block" style="width: 100%; max-width: 500px;" v-if="cols && rows">
        <div class="logo d-flex align-items-center px-3">
            <div class="boxes">
                <p class="title text-left small m-0 p-0 text-info font-weight-bold h1">▢▣</p>
                <p class="title text-left small m-0 p-0 text-info font-weight-bold h1">▣▢</p>
            </div>
            <span class="text-info h4 font-weight-cold title ml-3 my-auto">Boxes</span>
        </div>

        <div class="mb-4 d-flex flex-column justify-content-between align-items-end">
            <p class="font-weight-bold text-light d-flex flex-column align-items-start">
                <span class="badge text-success p-2" :class="{'text-success': turn, 'text-danger': !turn}" v-if="!winner" v-text="turn ? 'Your move ..': 'Opponents move ..'"> </span>
                <span v-if="!turn && winner"> Winner: <span class="badge badge-info p-2">{{winner}}</span> </span>
                <span v-if="!turn && winner && winner==name" class="text-left text-success font-weight-bold mt-2"> WooHoo! <br> You won. </span>
                <span v-if="!turn && winner && winner!=name" class="text-left text-danger font-weight-bold mt-2"> Booo! <br> You lost. </span>
            </p>
            <div class="d-flex flex-row justify-content-between align-items-end w-100">
                <p class="font-weight-bold text-light btn m-0 p-0 d-inline-block text-left">
                    <span  class="badge border rounded-0 px-3 py-2">{{Object.keys(scores)[0]}}</span>
                    <span  class="badge badge-info  px-3 py-2"> <span class="h5 font-weight-bold">{{scores[Object.keys(scores)[0]]}}</span> </span>
                </p>
                <span class="title h2 text-light">vs</span>
                <p class="font-weight-bold text-light btn m-0 p-0 d-inline-block text-left">
                    <span  class="badge badge-info  px-3 py-2"> <span class="h5 font-weight-bold">{{scores[Object.keys(scores)[1]]}}</span> </span>
                    <span  class="badge border rounded-0 px-3 py-2">{{Object.keys(scores)[1]}}</span>
                </p>
            </div>
        </div>

        <div v-for="row in rows*2+1" :key="row" class="d-flex">
            <div v-for="col in cols*2+1" :key="col" class="d-flex" :class="{'flex-grow-1': col%2==0}">
                <!-- Dot -->
                <div v-if="(row)%2!=0 && (col)%2!=0" 
                    class="border border-secondary rounded-circle bg-secondary"
                    style="width: 10px; height: 10px;">
                </div>
                <!-- Line | Horizontal -->
                <div v-else-if="(row)%2!=0 && (col)%2==0" 
                    @click="markX(col, row)"
                    :id="col+'-'+row"
                    :ref="col+'-'+row"
                    class="mark bg-darkish flex-grow-1"
                    :class="{'line': turn}"
                    style="width: 10px;">
                </div>
                <!-- Line | Vertical -->
                <div v-else-if="(row)%2==0 && (col)%2!=0" 
                    @click="markY(col, row)"
                    :id="col+'-'+row"
                    :ref="col+'-'+row"
                    class="mark bg-darkish"
                    :class="{'line': turn}"
                    style="height: 50px; width: 10px">
                </div>
                <!-- Box -->
                <div v-else-if="(row)%2==0 && (col)%2==0" 
                    class="m-auto box font-weight-bold"
                    :id="col+'-'+row"
                    :ref="col+'-'+row"
                    :state="0"
                    style="width: 30px;">
                </div>
            </div>
        </div>

        <div v-if="winner" class="my-5">
            <b-button variant="outline-info" :to="origin"> New Game </b-button>
        </div>
    </div>
</template>

<script>
export default {
    name: 'Box',
    props: ["cols", "rows", "player", "socket", "roomID", "name", "room"],
    data: ()=> {
        return {
            turn: false,
            scores: {},
            winner: null,
			origin: location.origin,
        }
    },
    mounted() {
        this.room.players.forEach(player => {
            this.scores[player] = 0;
        });
        this.$forceUpdate();
        
        if(this.player==1)
            this.turn = true;

        this.socket.on('markedX', (marked)=>{
            if(marked.row > 1) { // add point to box-above(row-1) => IF row > 1
                let box = this.$refs[marked.col+'-'+(marked.row-1)][0];
                box.setAttribute('state', parseInt(box.getAttribute('state'))+1);
                if(parseInt(box.getAttribute('state'))==4) {
                    box.innerText = marked.name.charAt(0);
                    this.addScore(marked.name);
                }
            }
            if(marked.row < 2*this.rows+1) { // add point to box-below(row+1) => IF row < 2*rows+1
                let box = this.$refs[marked.col+'-'+(marked.row+1)][0];
                box.setAttribute('state', parseInt(box.getAttribute('state'))+1);
                if(parseInt(box.getAttribute('state'))==4) {
                    box.innerText = marked.name.charAt(0);
                    this.addScore(marked.name);
                }
            }
            this.$refs[marked.col+'-'+marked.row][0].setAttribute('active', true);
            this.turn = !this.turn;
        });

        this.socket.on('markedY', (marked)=>{
            if(marked.col > 1) { // add point to left-box(col-1) => IF col > 1
                let box = this.$refs[(marked.col-1)+'-'+marked.row][0];
                box.setAttribute('state', parseInt(box.getAttribute('state'))+1);
                if(parseInt(box.getAttribute('state'))==4) {
                    box.innerText = marked.name.charAt(0);
                    this.addScore(marked.name);
                }
            }
            if(marked.col < 2*this.cols+1) { // add point to right-box(col+1) => IF col < 2*cols+1
                let box = this.$refs[(marked.col+1)+'-'+marked.row][0];
                box.setAttribute('state', parseInt(box.getAttribute('state'))+1);
                if(parseInt(box.getAttribute('state'))==4) {
                    box.innerText = marked.name.charAt(0);
                    this.addScore(marked.name);
                }
            }
            this.$refs[marked.col+'-'+marked.row][0].setAttribute('active', true);
            this.turn = !this.turn;
        });

        this.socket.on('winner', (data) => {
            if(!this.winner) {
                this.winner = data.name;
                this.turn = false;
            }
        });
    },
    methods: {
        addScore(name) {
            if(!this.scores[name])
                this.scores[name] = 1;
            else
                this.scores[name] += 1;
            
            if(this.scores[name] > this.cols*this.rows/2)
                this.win(name)
        },
        win(name) {
            this.socket.emit('win', {
                'roomID': this.roomID,
                'name': name,
            });
        },
        markX(col, row) {
            if(this.turn && !this.$refs[col+'-'+row][0].hasAttribute('active')) {
                this.socket.emit('markX', {
					'col': col,
					'row': row,
                    'roomID': this.roomID,
                    'name': this.name,
				});
            }
        },
        markY(col, row) {
            if(this.turn && !this.$refs[col+'-'+row][0].hasAttribute('active')) {
                this.socket.emit('markY', {
                    'col': col,
                    'row': row,
                    'roomID': this.roomID,
                    'name': this.name,
                });
            }
        },
    }
}
</script>

<style>
.line {
    cursor: pointer;
    transition: all .35s;
}
.bg-darkish {
    background-color: #3333 !important;
}
.line:hover {
    background-color: #f8f9fa !important;
}
.mark[active] {
    background-color: #f8f9fa !important;
}
.box[state="4"] {
    color: #17a2b8;
}
.logo {
    text-align: left;
    position: absolute;
    top: 25px;
    left: 25px;
}
.boxes {
    transform: scaleX(1.5);
}
</style>