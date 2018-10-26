<template>
    <div class="container">
        <div class="row justify-content-center">
            <div class="col-md-8">
                <div class="card">
                    <div class="card-header">
                        <button @click="initTask()" class="btn btn-primary btn-sm float-right">
                            + Add Task
                        </button>
                        Tasks List
                    </div>

                    <div class="card-body">

                    </div>
                </div>
            </div>
        </div>
        <div class="modal fade" id="modal" tabindex="-1" role="dialog" aria-labelledby="modalLabel" aria-hidden="true">
            <div class="modal-dialog" role="document">
                <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title" id="modalLabel">Task Form</h5>
                    <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                    <span aria-hidden="true">&times;</span>
                    </button>
                </div>
                <div class="modal-body">
                    <div class="alert alert-danger" role="alert" v-if="errors.length > 0">
                        <ul>
                            <li v-for="(error, index) in errors" :key="index">
                                {{ error }}
                            </li>
                        </ul>
                    </div>

                    <div class="form-group">
                        <label for="">Task</label>
                        <input type="text" name="title" id="title" class="form-control" v-model="task.title">
                    </div>

                    <div class="form-group">
                        <label for="">Priority</label>
                        <select name="prior" id="" class="form-control" v-model="task.prior">
                            <option value="low">Low</option>
                            <option value="medium">Medium</option>
                            <option value="high">High</option>
                        </select>
                    </div>
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
                    <button type="button" class="btn btn-primary" @click="createTask">Submit</button>
                </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                tasks: [],
                task: {
                    title: '',
                    prior: ''
                },
                errors: []
            }
        },
        methods: {
            initTask() {
                this.errors = [];
                $("#modal").modal("show");

            },

            createTask() {
                axios.post('/task', {
                    title: this.task.title,
                    prior: this.task.prior
                })
                .then(response => {
                    this.resetForm();

                    $("#modal").modal("hide");
                })
                .catch(error => {
                    // console.log(error.response.data);
                    this.errors = [];
                    if (error.response.data.errors.title) {
                        this.errors.push(error.response.data.errors.title[0]);
                    }
                    if (error.response.data.errors.prior) {
                        this.errors.push(error.response.data.errors.prior[0]);
                    }
                })
            },

            resetForm() {
                this.task.title = '';
                this.task.prior = '';
            }
        },
        mounted() {

        }
    }
</script>
