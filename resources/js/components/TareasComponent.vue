<template>
    <div>
        
        <form @submit.prevent="editarNota(nota)" v-if="editarActivo"> <!-- Es lo mismo que poner el e.preventDefault en Javascript al evento. Esto significa que al enviarse el formulario ejecute el método "agregar" -->
            <h3>Editar Tarea</h3>
            <input type="text" placeholder="Nombre" class="form-control mb-2" v-model="nota.nombre">
            <input type="text" placeholder="Descripcion" class="form-control mb-2" v-model="nota.descripcion">
            <button class="btn btn-success" type="submit">Guardar</button>
            <button class="btn btn-danger" @click="cancelarEdicion()">Cancelar</button>
        </form>

        <form @submit.prevent="agregar" v-else> <!-- Es lo mismo que poner el e.preventDefault en Javascript al evento. Esto significa que al enviarse el formulario ejecute el método "agregar" -->
            <h3>Agregar Tareas</h3>
            <input type="text" placeholder="Nombre" class="form-control mb-2" v-model="nota.nombre">
            <input type="text" placeholder="Descripcion" class="form-control mb-2" v-model="nota.descripcion">
            <button class="btn btn-primary" type="submit">Agregar</button>
        </form>

        <hr class="mt-3">
        <h3>Listado de Notas</h3>
        <ul class="list-group my-2">
            <li class="list-group-item" v-for="(nota, index) in notas" :key="index">
                <span class="badge badge-primary float-right">
                    {{nota.updated_at}}
                </span>
                <p>{{nota.nombre}}</p>
                <p>{{nota.descripcion}}</p>
                <button class="btn btn-danger btn-sm" @click="eliminarNota(index)">Eliminar</button>
                <button class="btn btn-warning btn-sm" @click="editarFormulario(nota)">Editar</button>
            </li>
        </ul>
    </div>
</template>

<script>
export default {
    data(){
        return {
            notas: [],
            nota: {nombre: '', descripcion: ''},
            editarActivo: false
        }
    },
    created(){ // Este método es estándar de Vue, se ejecuta antes de renderizar el componente y tiene acceso a toda la "data"
        axios.get('/notas') // Con "get" accedo al método "index" de NotaController.php
        .then(res => {
            this.notas = res.data
            console.log(this.notas)
        })
    },
    methods:{
        agregar(){

            if(this.nota.nombre.trim() === '' || this.nota.descripcion.trim() === ''){
                alert('Todos los campos se deben deiligenciar')
                return // Si entra a este "if", este "return" nos asegura que no se seguirá ejecutando lo de abajo que está fuera del if. Es decir, finaliza la ejecución del método "agregar"
            }

            //console.log(this.nota.nombre, this.nota.descripcion)
            const params = {
                nombre: this.nota.nombre,
                descripcion: this.nota.descripcion
            }
            axios.post('/notas', params) // Con "post" accedo al método "store" de NotaController.php
            .then(res => {
                this.notas.push(res.data)
                this.nota.nombre = ''
                this.nota.descripcion = ''
            })
        },
        eliminarNota(id){
            axios.delete(`/notas/${id}`)
            .then(() =>{
                this.notas.splice(id, 1) // Borre el elemento(objeto) con ID "id" del array "notas" y borre sólo un elemento
            })
        },
        editarFormulario(item){
            this.editarActivo = true
            this.nota.nombre = item.nombre
            this.nota.descripcion = item.descripcion
            this.nota.id = item.id
        },
        editarNota(item){
            const params = {
                nombre: item.nombre,
                descripcion: item.descripcion
            }
            axios.put(`/notas/${item.id}`, params) // "put" va al método "update" de NotaController
            .then(res =>{
                const indexNota = this.notas.findIndex(notaBuscar => notaBuscar.id === res.data.id)

                this.notas[indexNota] = res.data
            })
        },
        cancelarEdicion(){
            this.editarActivo = false
            this.nota = {nombre: '', descripcion: ''}
        }
    }
}
</script>
