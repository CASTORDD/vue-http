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
                :tarefa="tarefa" 
                @editar="seleccionarTarefaEdit"
            />
        </ul>

        <p v-else>Nenhuma tarefa criada.</p>

        <TarefaSalvar 
            v-if="exibirFormulario"
            :tarefa="tarefaSeleccionada"
            @criar="criarTarefa"
            @editar="editarTarefa"
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
            exibirFormulario: false,
            tarefaSeleccionada: undefined

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
                    setTimeout( this.resetar(), 300 )
                    
                })
        },
        seleccionarTarefaEdit( tarefa ){
            this.tarefaSeleccionada = tarefa
            this.exibirFormulario =  true
        },
        resetar(){
            this.tarefaSeleccionada = undefined
            this.exibirFormulario = false
        },
        editarTarefa( tarefa ){
            console.log('editar:', tarefa)
            axios.put(`${config.apiURL}/tarefas/${tarefa.id}`, tarefa)
                .then( response => {
                    console.log(`PUT/${tarefa.id}`, response)
                    const indice = this.tarefas.findIndex( t => t.id === tarefa.id )
                    this.tarefas.splice(indice, 1, tarefa)
                    this.resetar()
                })
        }
    } 
}
</script>
