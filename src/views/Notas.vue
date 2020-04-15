<template>
    <div class="container">
        <h1>Notas</h1>
        <b-alert :show="dissmissCountDown" dismissible
                 :variant="mensaje.color"
                 @dismissed="dissmissCountDown=0"
                 @dismiss-count-down="countDownChange">
            {{mensaje.text}}
        </b-alert>
        <form @submit.prevent="editarNota(notaEditar)" v-if="!agregar">
            <h3 class="text-center">Editar nota</h3>
            <input type="text" class="form-control my-2" v-model="notaEditar.nombre">
            <input type="text" class="form-control my-2" v-model="notaEditar.descripcion">
            <b-button class="btn-warning my-2 mx-2" type="submit">Confirmar</b-button>
            <b-button class="my-2" type="submit" @click="agregar=true">Cancelar</b-button>
        </form>

        <form @submit.prevent="agregarNota()" v-if="agregar">
            <h3 class="text-center">Agregar nueva nota</h3>
            <input type="text" placeholder="Ingresa el nombre" class="form-control my-2" v-model="nota.nombre">
            <input type="text" placeholder="Ingresa la descripcion" class="form-control my-2"
                   v-model="nota.descripcion">
            <b-button class="btn-sm btn-block btn-success" type="submit">Agregar</b-button>
        </form>
        <table class="table">
            <thead>
            <th scope="col">#</th>
            <th scope="col">Nombre</th>
            <th scope="col">Descripcion</th>
            <th scope="col">Fecha</th>
            <th scope="col">Acciones</th>
            </thead>
            <tbody>
            <tr v-for="(item, index) in notas" :key="index">
                <td scope="row">{{item._id}}</td>
                <td>{{item.nombre}}</td>
                <td>{{item.descripcion}}</td>
                <td>{{item.date}}</td>
                <td>
                    <b-button class="btn-warning btn-sm mx-2" @click="activarEdicion(item._id)">Actualizar</b-button>
                    <b-button class="btn-danger btn-sm mx-2" @click="eliminarNota(item._id)">Eliminar</b-button>
                </td>
            </tr>
            </tbody>
        </table>
    </div>
</template>
<script>
    export default {
        data() {
            return {
                mensaje: {color: 'success', text: ''},
                dismissSecs: 5,
                dissmissCountDown: 0,
                notas: [],
                nota: {},
                agregar: true,
                notaEditar: {}
            };
        },
        created() {
            this.listarNotas();
        },
        methods: {
            agregarNota() {
                this.axios.post('nueva-nota', this.nota).then(res => {
                    this.notas.push(res.data);
                    this.nota.nombre = '';
                    this.nota.descripcion = '';

                    this.mensaje.text = 'Nota agregada con éxito';
                    this.mensaje.color = 'success';
                    this.showAlert();
                }).catch(err => {
                    alert(err);
                });
            },
            eliminarNota(id) {
                this.axios.delete(`nota/${id}`).then(res => {
                    const index = this.notas.findIndex(item => item._id === res.data.id);
                    this.notas.splice(index, 1);
                    this.mensaje.text = 'Nota eliminada con éxito';
                    this.mensaje.color = 'success';
                    this.showAlert();
                }).catch(e => {
                    this.mensaje.text = 'Hubo un error al eliminar la nota';
                    this.mensaje.color = 'danger';
                    this.showAlert();
                    console.log(e);
                });
            },
            listarNotas() {
                this.axios.post('notas').then((response) => {
                    this.notas = response.data;
                }).catch(error => {
                    console.log(error);
                });
            },
            activarEdicion(id) {
                this.agregar = false;
                this.axios.post(`/nota/${id}`)
                    .then(res => {
                        this.notaEditar = res.data;
                    }).catch(e => {
                    console.log(e.response);
                });
            },
            editarNota(item) {
                this.axios.put(`/nota/${item._id}`, item)
                    .then(res => {
                        const index = this.notas.findIndex(nota => nota._id === res.data._id);
                        this.notas[index].nombre = res.data.nombre;
                        this.notas[index].descripcion = res.data.descripcion;
                        this.mensaje.text = 'Nota actualizada';
                        this.mensaje.color = 'success';
                        this.showAlert();
                    }).catch(e => {
                    this.mensaje.text = `Hubo un error: ${e.response.data.error.errors.nombre.message}`;
                    this.mensaje.color = 'danger';
                    this.showAlert();
                        console.log(e);
                    });
                this.agregar = true;
            },
            countDownChange(dissmissCountDown) {
                this.dissmissCountDown = dissmissCountDown;
            },
            showAlert() {
                this.dissmissCountDown = this.dismissSecs;
            }
        }
    }
</script>