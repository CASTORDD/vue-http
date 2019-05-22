<template>
    <div>

        <div class="row">
            <div class="col-sm-10">
                <h1 class="font-weight-light">Lista de tarefas</h1>
            </div>
            <div class="col-sm-2">
                <button 
                    class="btn btn-primary btn-sm floar-right"
                    @click="exibirFormulario = !exibirFormulario"
                >
                    <i class="fa fa-plus"></i>
                    <span> Crear</span>
                </button>
            </div>
        </div>

        <ul class="list-group" v-if="tarefas.length > 0">
            <TarefasListaIten
                v-for="tarefa in tarefas"
                :key="tarefa.id"
                :tarefa="tarefa" />
        </ul>

        <p v-else>Nenhuma tarefa criada.</p>

        <TarefaSalvar 
            v-if="exibirFormulario"
            @criar="criarTarefa"
        />

    </div>
</template>

<script>
import axios from 'axios'

import config from '../config/config'
import TarefaSalvar from './TarefaSalvar.vue'
import TarefasListaIten from './TarefasListaIten.vue'
import { timeout } from 'q';
import { setTimeout } from 'timers';

export default {
    components: {
        TarefaSalvar,
        TarefasListaIten
    },
    data() {
        return {
            tarefas: [],
            exibirFormulario: false
        }
    },
    created(){
        axios.get(`${config.apiURL}/tarefas`)
            .then((response) => {
                console.log('GET/tarefas', response)
                this.tarefas = response.data
            })
    },
    methods: {
        criarTarefa( tarefa ){
            axios.post(`${config.apiURL}/tarefas`, tarefa )
                .then((response) => {
                    console.log('POST/tarefas', response)
                    this.tarefas.push(response.data)
                    setTimeout( this.exibirFormulario = false, 300 )
                    
                })
        }
    } 
}
</script>
