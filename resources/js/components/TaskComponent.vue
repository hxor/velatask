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
                        <table class="table table-bordered table-striped">
                            <thead>
                                <tr>
                                    <th scope="col">#</th>
                                    <th scope="col">Task</th>
                                    <th scope="col">Level</th>
                                    <th scope="col"></th>
                                </tr>
                            </thead>
                            <tbody>
                                <tr v-for="(task, index) in tasks" :key="index">
                                    <th scope="row">{{ index+1 }}</th>
                                    <td>{{ task.title }}</td>
                                    <td>{{ task.prior }}</td>
                                    <td>
                                        <button class="btn btn-default btn-sm" @click="initUpdateTask(task)">Edit</button>
                                        <button class="btn btn-danger btn-sm" @click="deleteTask(task)">Delete</button>
                                    </td>
                                </tr>
                            </tbody>
                        </table>
                    </div>

                    <div class="card-footer text-muted">
                        <ul class="pagination">
                            <li v-bind:class="[{disabled: !pagination.prev_page_url}]" class="page-item"><a class="page-link" href="#" @click="getTasks(pagination.prev_page_url)">Previous</a></li>

                            <li class="page-item disabled"><a class="page-link text-dark" href="#">Page {{ pagination.current_page }} of {{ pagination.last_page }}</a></li>

                            <li v-bind:class="[{disabled: !pagination.next_page_url}]" class="page-item"><a class="page-link" href="#" @click="getTasks(pagination.next_page_url)">Next</a></li>
                        </ul>
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
                    <button type="button" class="btn btn-secondary" data-dismiss="modal" @click="resetForm">Close</button>
                    <button type="button" class="btn btn-primary" @click="saveTask">Submit</button>
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
                    id: '',
                    title: '',
                    prior: ''
                },
                pagination: {},
                errors: [],
                edit: false
            }
        },
        methods: {

            getTasks(page_url) {
                page_url = page_url || '/task';
                axios.get(page_url)
                    .then(response => {
                        console.log(response.data);
                        this.createPagination(response.data.meta, response.data.links);
                        this.tasks = response.data.data;
                    });
            },

            initTask() {
                this.errors = [];
                $("#modal").modal("show");

            },

            initUpdateTask(task) {
                this.task.id = task.id;
                this.task.title = task.title;
                this.task.prior = task.prior;
                this.edit = true;

                this.initTask();
            },

            saveTask() {
                if (this.edit === false) {
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
                    });
                } else {
                    axios.patch('/task/' + this.task.id, {
                        title: this.task.title,
                        prior: this.task.prior
                    })
                    .then(response => {
                        this.getTasks();
                        this.resetForm();
                        $("#modal").modal("hide");
                    })
                    .catch(error => {
                        this.errors = [];
                        if (error.response.data.errors.title) {
                            this.errors.push(error.response.data.errors.title[0]);
                        }
                        if (error.response.data.errors.prior) {
                            this.errors.push(error.response.data.errors.prior[0]);
                        }
                    });
                }
            },

            deleteTask(task) {
                let desicion = confirm('Are you sure?');
                if (desicion === true) {
                    axios.delete('/task/' + task.id)
                    .then(response => {
                        this.getTasks();
                    })
                    .catch(error => {
                        console.log(error);
                    });
                }
            },

            resetForm() {
                this.task.title = '';
                this.task.prior = '';
                this.edit = false;
            },

            createPagination(meta, links) {
                let pagination = {
                    current_page: meta.current_page,
                    last_page: meta.last_page,
                    next_page_url: links.next,
                    prev_page_url: links.prev
                }

                this.pagination = pagination;
            }
        },
        mounted() {
            this.getTasks();
        }
    }
</script>
