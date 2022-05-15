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
                            {{ index + (pageIndex - 1) * pageSize + 1 }}
                        </td>
                        <td class="align-middle">
                            <input
                                v-if="editor && rowSelectorIndex === todo._id"
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
                                            rowSelectorIndex === todo._id
                                        )
                                    "
                                    class="form-check-input"
                                    @change="changeStatus(todo)"
                                />
                            </div>
                        </td>
                        <td class="align-middle">
                            <button
                                v-if="editor && rowSelectorIndex === todo._id"
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
        <div class="loading" v-if="loading">
            <div class="spinner-grow text-success" role="status"></div>
        </div>
        <div class="pagination-todo">
            <nav aria-label="Page navigation example">
                <ul class="pagination justify-content-end">
                    <li
                        @click="getTodo(pageIndex - 1)"
                        :class="[{ disabled: pageIndex === 1 }, 'page-item']"
                    >
                        <a class="page-link" href="#" aria-label="Previous">
                            <span aria-hidden="true">&laquo;</span>
                        </a>
                    </li>
                    <li
                        class="page-item"
                        v-for="index in totalPage"
                        :key="index"
                        :class="[{ active: index === pageIndex }, 'page-item']"
                        @click="getTodo(index)"
                    >
                        <a class="page-link" href="#">{{ index }}</a>
                    </li>
                    <li
                        class="page-item"
                        @click="getTodo(pageIndex + 1)"
                        :class="[
                            { disabled: pageIndex === totalPage },
                            'page-item',
                        ]"
                    >
                        <a class="page-link" href="#" aria-label="Next">
                            <span aria-hidden="true">&raquo;</span>
                        </a>
                    </li>
                </ul>
            </nav>
        </div>
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
            todoName: "",
            pageIndex: 1,
            pageSize: 10,
            editor: false,
            rowSelectorIndex: null,
            loading: false,
            totalTodo: 0,
            totalPage: 0,
        };
    },
    methods: {
        /**
         * Lấy công việc
         */
        async getTodo(pageIndex) {
            const me = this;

            me.mask();

            // Lấy tất cả bản ghi
            // let { data } = await axios({
            //     url: [me.baseUrl, "todo"].join("/"),
            //     method: "get",
            // });

            // if (data) {
            //     me.todos = data.todoList;
            //     me.totalTodo = data.count;
            // }

            // Bỏ qua sự kiện khi nút bị disabled
            if (pageIndex === 0 || pageIndex === me.totalPage + 1) {
                me.unmask();
                return;
            }

            // Reset giá trị
            me.editor = false;
            me.rowSelectorIndex = null;

            // Lấy số thứ tự trang
            if (!pageIndex) {
                pageIndex = Number.parseInt(localStorage.getItem("pageIndex"));
            }

            me.pageIndex = pageIndex ? pageIndex : 1;

            // Lấy tổng số bản ghi
            await me.getTotalCount();

            // Lưu giá trị số thứ tự trang
            localStorage.setItem("pageIndex", me.pageIndex);

            // Lấy bản ghi trên 1 trang
            let { data } = await axios({
                url: [me.baseUrl, "todo", me.pageIndex, me.pageSize].join("/"),
                method: "get",
            });

            if (data) {
                me.todos = data.todoPage;
            }

            me.unmask();
        },

        /**
         * Cập nhật các công việc
         */
        async updateTodo(dataRow) {
            const me = this;

            me.mask();

            let { data } = await axios({
                url: [me.baseUrl, "todo", dataRow._id].join("/"),
                method: "put",
                data: {
                    name: dataRow.name,
                    complete: dataRow.complete,
                },
            });

            if (data) {
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
            } else {
                me.unmask();
            }
        },

        /**
         * Xóa công việc
         */
        async deleteTodo(dataRow) {
            const me = this;

            me.mask();

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
                } else {
                    me.unmask();
                }
            }
        },

        /**
         * Thêm công việc
         */
        async createTodo() {
            const me = this;

            me.mask();

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

                swal(
                    "Thêm công việc",
                    data ? "Thành công!" : "Thất bại!",
                    data ? "success" : "error",
                    {
                        buttons: false,
                        timer: 1500,
                    }
                );
            } else {
                me.unmask();
            }
        },

        /**
         * Lấy tổng bản ghi
         */
        async getTotalCount() {
            const me = this;

            let { data } = await axios({
                url: [me.baseUrl, "count"].join("/"),
                method: "get",
            });

            if (data) {
                me.totalTodo = data.items;
                me.totalPage = Math.ceil(me.totalTodo / me.pageSize);
            }
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
            dataRow.complete = !dataRow.complete;
        },

        /**
         * Hiển thị loading
         */
        mask() {
            const me = this;

            me.loading = true;
        },

        /**
         * Ẩn loading
         */
        unmask() {
            const me = this;

            me.loading = false;
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

.loading {
    width: 100%;
    height: 100%;
    background: transparent;
    top: 0;
    left: 0;
    position: absolute;
    z-index: 2;
}

.spinner-grow {
    position: absolute;
    top: calc(50% - 16px);
}
</style>
