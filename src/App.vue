<template>
    <div class="container">
        <div class="title">
            <h2>Công việc</h2>
        </div>
        <div class="create-todo">
            <div class="input-group mb-3">
                <input
                    type="text"
                    class="form-control"
                    placeholder="Công việc"
                    aria-label="Công việc"
                    aria-describedby="button-addon2"
                    v-model="todoName"
                />
                <div class="input-group-append">
                    <button
                        class="btn btn-primary"
                        type="button"
                        id="button-addon2"
                        @click="createTodo"
                        :disabled="!todoName"
                    >
                        Thêm mới
                    </button>
                </div>
            </div>
        </div>
        <div class="todo-list">
            <table class="table table-striped">
                <thead>
                    <th scope="col" class="col-1">STT</th>
                    <th scope="col" class="col-5">Tên</th>
                    <th scope="col" class="col-3">Hoàn thành</th>
                    <th scope="col" class="col-3">Hành động</th>
                </thead>
                <tbody>
                    <tr v-for="(todo, index) in todos" :key="todo._id">
                        <td class="align-middle" scope="row">
                            {{ index + 1 }}
                        </td>
                        <td class="align-middle">
                            <input
                                v-if="editor && rowSelectorIndex == todo._id"
                                v-model="todo.name"
                            />
                            <span v-else>{{ todo.name }}</span>
                        </td>
                        <td class="align-middle">
                            <div class="form-switch">
                                <input
                                    type="checkbox"
                                    :checked="todo.complete"
                                    :disabled="
                                        !(
                                            editor &&
                                            rowSelectorIndex == todo._id
                                        )
                                    "
                                    class="form-check-input"
                                    @change="changeStatus(todo)"
                                />
                            </div>
                        </td>
                        <td class="align-middle">
                            <button
                                v-if="editor && rowSelectorIndex == todo._id"
                                type="button"
                                class="btn btn-success"
                                @click="updateTodo(todo)"
                            >
                                Đồng ý
                            </button>
                            <button
                                v-else
                                type="button"
                                class="btn btn-primary"
                                @click="editTodo(todo)"
                            >
                                Sửa
                            </button>
                            <button
                                type="button"
                                class="btn btn-danger"
                                @click="deleteTodo(todo)"
                            >
                                Xóa
                            </button>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
        <!-- <div>
            <nav aria-label="Page navigation example">
                <ul class="pagination justify-content-end">
                    <li class="page-item">
                        <a class="page-link" href="#" aria-label="Previous">
                            <span aria-hidden="true">&laquo;</span>
                        </a>
                    </li>
                    <li
                        class="page-item"
                        v-for="(todo, index) in todos"
                        :key="index"
                    >
                        <a class="page-link" href="#">{{ index + 1 }}</a>
                    </li>
                    <li class="page-item">
                        <a class="page-link" href="#" aria-label="Next">
                            <span aria-hidden="true">&raquo;</span>
                        </a>
                    </li>
                </ul>
            </nav>
        </div> -->
    </div>
</template>

<script>
import axios from "axios";
import swal from "sweetalert";
export default {
    name: "App",
    data() {
        return {
            baseUrl: "https://k24-server-version2.herokuapp.com",
            todos: [],
            totalTodo: 0,
            todoName: "",
            pageIndex: 1,
            pageSize: 5,
            editor: false,
            rowSelectorIndex: null,
        };
    },
    methods: {
        /**
         * Lấy tất cả các công việc
         */
        async getTodo() {
            const me = this;

            let { data } = await axios({
                url: [me.baseUrl, "todo"].join("/"),
                method: "get",
            });

            if (data) {
                me.todos = data.todoList;
                me.totalTodo = data.count;
            }
        },

        /**
         * Cập nhật các công việc
         */
        async updateTodo(dataRow) {
            const me = this;

            let { data } = await axios({
                url: [me.baseUrl, "todo", dataRow._id].join("/"),
                method: "put",
                data: {
                    name: dataRow.name,
                    complete: dataRow.complete,
                },
            });

            if (data) {
                me.editor = false;
                me.getTodo();

                swal(
                    "Sửa công việc",
                    data ? "Thành công!" : "Thất bại!",
                    data ? "success" : "error",
                    {
                        buttons: false,
                        timer: 1500,
                    }
                );
            }
        },

        /**
         * Xóa công việc
         */
        async deleteTodo(dataRow) {
            const me = this;

            let action = await swal({
                title: "Xóa công việc này?",
                buttons: {
                    cancel: "Hủy",
                    confirm: "Đồng ý",
                },
            });

            if (action) {
                let { data } = await axios({
                    url: [me.baseUrl, "todo", dataRow._id].join("/"),
                    method: "delete",
                });

                if (data) {
                    me.getTodo();

                    swal(
                        "Xóa công việc",
                        data ? "Thành công!" : "Thất bại!",
                        data ? "success" : "error",
                        {
                            buttons: false,
                            timer: 1500,
                        }
                    );
                }
            }
        },

        /**
         * Thêm công việc
         */
        async createTodo() {
            const me = this;

            let { data } = await axios({
                url: [me.baseUrl, "todo"].join("/"),
                method: "post",
                data: {
                    name: me.todoName,
                    complete: false,
                },
            });

            if (data) {
                me.getTodo();
            }

            swal(
                "Thêm công việc",
                data ? "Thành công!" : "Thất bại!",
                data ? "success" : "error",
                {
                    buttons: false,
                    timer: 1500,
                }
            );
        },

        /**
         * Chuyển sang mode sửa
         */
        editTodo(dataRow) {
            const me = this;

            me.rowSelectorIndex = dataRow._id;
            me.editor = !me.editor;
        },

        /**
         * Thay đổi trạng thái của todo
         */
        changeStatus(dataRow) {
            const me = this;

            dataRow.complete = !dataRow.complete;
        },
    },

    mounted() {
        const me = this;

        me.getTodo();
    },
};
</script>

<style>
#app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
}

tbody {
    border-top: none !important;
}

.title,
.create-todo {
    margin-bottom: 30px;
}

.btn-danger {
    margin-left: 10px;
}

.input-group-append {
    margin-left: 10px !important;
}
</style>
