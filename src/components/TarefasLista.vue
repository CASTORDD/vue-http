<template>
    <div>

        <div class="row">
            <div class="col-sm-10">
                <h1 class="font-weight-light">Lista de tarefas</h1>
            </div>
            <div class="col-sm-2">
                <button 
                    class="btn btn-primary btn-sm floar-right"
                    @click="exibirFormCriarTarea"
                >
                    <i class="fa fa-plus"></i>
                    <span> Crear</span>
                </button>
            </div>
        </div>

        <ul class="list-group" v-if="tarefasOrdenadas.length > 0">
            <TarefasListaIten
                v-for="tarefa in tarefasOrdenadas"
                :key="tarefa.id"
                :tarefa="tarefa" 
                @editar="seleccionarTarefaEdit"
                @deletar="deletarTarefa"
                @concluir="editarTarefa"
            />
        </ul>

        <p v-else-if="!mensagemErro">Nenhuma tarefa criada.</p>

        <div 
            class="alert alert-danger"
            v-else
        >{{ mensagemErro }}</div>

        <TarefaSalvar 
            v-if="exibirFormulario"
            :tarefa="tarefaSeleccionada"
            @criar="criarTarefa"
            @editar="editarTarefa"
        />

    </div>
</template>

<script>
import axios from './../config/axios'
import TarefaSalvar from './TarefaSalvar.vue'
import TarefasListaIten from './TarefasListaIten.vue'
import { timeout, Promise } from 'q';
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
            tarefaSeleccionada: undefined,
            mensagemErro: undefined

        }
    },
    computed: {
        tarefasOrdenadas(){
            return this.tarefas.sort((t1, t2) => {
                if(t1.concluido === t2.concluido){
                    return t1.titulo < t2.titulo
                        ? -1
                        : t1.titulo > t2.titulo
                            ? 1
                            : 0
                }
                return t1.concluido - t2.concluido
            })
        }
    },
    created(){
        axios.get(`/tarefas`)
            .then((response) => {
                console.log('GET/tarefas', response)
                this.tarefas = response.data
                return 'AXIOS'
                }, 
                error => {
                    console.log('error capturado no then:', error)
                    return Promise.reject(error)
                })
            .catch(error => {
                console.log('error capturado no then:', error)
                if(error.response){
                    this.mensagemErro = `Servidor retorno com error: ${error.message} ${error.response.statusText}`
                    console.log('Error [resposta]:', error.response)
                } else if (error.request) {
                    this.mensagemErro = `Erro ao tratar comunicar com o servidor: ${error.message}`
                    console.log('Error [requisicao]: ', error.request)
                } else {
                    this.mensagemErro = 'Error ao fazer requisicao ao servidor: ${error.message}'
                }
            }).then((algumParametro) => {
                console.log('sempre executado!', algumParametro)
            })
    },
    methods: {
        criarTarefa( tarefa ){
            // axios.post(`/tarefas`, tarefa )
            //     .then((response) => {
            //         console.log('POST/tarefas', response)
            //         this.tarefas.push(response.data)
            //         setTimeout( this.resetar(), 300 )
                    
            //     })
            axios
                .request({
                    method: 'post',
                    baseURL: config.apiURL,
                    url: '/tarefas',
                    data: tarefa
                })
                .then((response) => {
                    console.log('POST/tarefas', response)
                    this.tarefas.push(response.data)
                    setTimeout( this.resetar(), 300 )                    
                })
        },
        exibirFormCriarTarea( event ){
            if(this.tarefaSeleccionada){
                this.tarefaSeleccionada = undefined
                return 
            }

            this.exibirFormulario = !this.exibirFormulario
        }
        ,
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
            axios.put(`/tarefas/${tarefa.id}`, tarefa)
                .then( response => {
                    console.log(`PUT/${tarefa.id}`, response)
                    const indice = this.tarefas.findIndex( t => t.id === tarefa.id )
                    this.tarefas.splice(indice, 1, tarefa)
                    this.resetar()
                })
        },
        deletarTarefa( tarefa ){
            const confirmar = window.confirm(`Deseja deletar a tarefa "${tarefa.titulo}"?`)
            if(confirmar){
                axios.delete(`/tarefas/${tarefa.id}`)
                    .then(response => {
                        console.log(`DELETE /tarefas/${tarefa.id}`, response)
                        const indice = this.tarefas.findIndex( t => t.id === tarefa.id )
                        this.tarefas.splice(indice, 1)
                    })

            }
        }
    } 
}
</script>
