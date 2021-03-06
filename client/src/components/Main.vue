<template>
<div class="topped">
    <Error
        v-if="errorDetected"
        :alertMessage="alertMessage"
    >
    </Error>
    <div class="d-flex justify-content-between main">
        <h2>Gridiron Fantasy Team Builder</h2> 
        <button class="btn btn-nfl-blue" @click="toggleAddForm">Add New Team</button>
    </div>
    <AddForm
        v-if="addShowed"
        :myteam="myteam"
        @showMine="showMyTeam"
        @showAll="showAllTeam"
        @error="showError"
    ></AddForm>
    <div>
        <ul class="team-container">
            <li :class="{active: myteam}" @click="showMyTeam">My Team</li>
            <li :class="{active: !myteam}" @click="showAllTeam">All Team</li>
        </ul>
        <hr>
        <TeamList
            :teams="teams"
            @delete="triggerWarn"
            @details="details"
        >
        </TeamList>
    </div>
    <hr>
    <h1 class="text-center">Player Section</h1>
    <PlayerList
        :teams="myTeams"
        ref="player"
        @reset="reset"
        @error="showError"
    ></PlayerList>
    <Modal
        :class="{'modal-active': modalActive}"
        :content="modalContent"
        :teamName="teamName"
        :starter="starter"
        :bench="bench"
        :deleted="deleted"
        @reset="reset"
        @clear="clear"
        @cancel="cancel"
        @error="showError"
    >
    </Modal>
</div>
</template>

<script>
import AddForm from './Add';
import TeamList from './TeamList';
import PlayerList from './PlayerList';
import Modal from './Modal';
import Error from './Error';
import axios from 'axios';

export default {
    name: 'Main',
    components: {
        AddForm, TeamList, PlayerList, Modal, Error
    },
    data() {
        return {
            myteam: true,
            addShowed: false,
            teams: [],
            myTeams: [],
            modalActive: false,
            modalContent: '',
            deleted: '',
            starter: [],
            bench: [],
            teamName: '',
            errorDetected: false,
            alertMessage: ''
        }
    },
    methods: {
        toggleAddForm() {
            this.addShowed = !this.addShowed;
        },
        showMyTeam() {
            const { access_token } = localStorage;
            axios.get('https://infinite-caverns-50726.herokuapp.com/teams', {
                headers: {
                    access_token
                }
            })
            .then(res => {
                this.teams = [];
                this.myTeams = [];
                res.data.teams.forEach(team => {
                    this.teams.push(team);
                    this.myTeams.push(team);
                })
                this.myteam = true;
                this.$refs.player.triggerGetTeam();
            })
            .catch(err => {
                this.showError(err);
            })
        },
        showAllTeam() {
            const { access_token } = localStorage;
            axios.get('https://infinite-caverns-50726.herokuapp.com/teams/all', {
                headers: {
                    access_token
                }
            })
            .then(res => {
                this.teams = [];
                res.data.teams.forEach(team => {
                    this.teams.push(team);
                })
                this.myteam = false;
            })
            .catch(err => {
                this.showError(err);
            })
        },
        triggerWarn(id) {
            this.modalActive = true;
            this.modalContent = 'warn';
            this.deleted = id;
        }, 
        details(team, name) {
            this.starter = [];
            this.bench = [];
            this.teamName = name;
            this.modalActive = true;
            this.modalContent = 'detail';
            team.forEach(player => {
                if(player.status === 'Starter') {
                    this.starter.push(player);
                } else {
                    this.bench.push(player);
                }
            })
        },
        cancel() {
            this.modalContent = '';
            this.modalActive = false;
        },
        reset() {
            if(this.myteam) {
                this.showMyTeam();
            } else {
                this.showAllTeam();
            }
            this.cancel();
            
        },
        clear() {
            this.$refs.player.resetList();
        },
        noError() {
            this.errorDetected = false;
            this.alertMessage = '';
        },
        processError(err) {
            this.errorDetected = true;
            if(Array.isArray(err.response.data.error)) {
                let errors = '';
                err.response.data.error.forEach(e =>  {
                    errors += `${e}, `
                })
                this.alertMessage = errors.substring(0, errors.length-2);
            } else {
                this.alertMessage = err.response.data.error;
            }
        },
        showError(err) {
                setTimeout(() => {
                this.noError();
            }, 3000);
            this.cancel();
            this.processError(err);
        }
    },
    created() {
        this.showMyTeam();
    }
}
</script>

<style>
    .modal-active {
        visibility: visible;
        opacity: 1;
    }
    .main {
        padding: 10px;
    }

    .btn-nfl-blue {
        background-color: #013369;
        color: white;
    }

    .btn-nfl-blue:hover {
        background-color: #264466;
    }

    .topped {
        /* margin-bottom: 50px; */
        min-height: 100%;
        overflow: hidden;
    }

    .team-container {
        list-style: none;
        display: flex;
        flex-direction: row;
    }

    .team-container li {
        list-style: none;
        margin: 0 15px;
    }

    .team-container li:hover {
        cursor: pointer;
        color: red;
        border-bottom: solid red 1px;
    }

    .active {
        color: red;
        border-bottom: solid red 1px;
    }  
</style>