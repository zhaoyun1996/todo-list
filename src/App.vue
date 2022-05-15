<template>
    <div class="container">
        <div class="title">
            <h2>Công việc</h2>
        </div>
        <div class="create-todo">
            <div class="input-group mb-3">
                <input
                    ref="inputAdd"
                    type="text"
                    class="form-control"
                    placeholder="Công việc"
                    aria-label="Công việc"
                    v-model="todoName"
                />
                <div class="input-group-append">
                    <button
                        class="btn btn-primary"
                        type="button"
                        @click="createTodo"
                        :disabled="!todoName"
                    >
                        Thêm mới
                    </button>

                    <button
                        class="btn btn-success"
                        type="button"
                        @click="getTodo()"
                    >
                        Lấy lại dữ liệu
                    </button>
                </div>
            </div>
        </div>
        <div class="todo-list">
            <table class="table table-striped">
                <thead>
                    <th scope="col" class="align-middle col-1">STT</th>
                    <th scope="col" class="align-middle col-5 text-left">
                        Tên
                    </th>
                    <th scope="col" class="align-middle col-3">Hoàn thành</th>
                    <th scope="col" class="align-middle col-3">Hành động</th>
                </thead>
                <tbody>
                    <tr v-for="(todo, index) in todos" :key="todo._id">
                        <td class="align-middle" scope="row">
                            {{ index + (pageIndex - 1) * pageSize + 1 }}
                        </td>
                        <td class="align-middle text-left">
                            <input
                                :ref="todo._id"
                                type="text"
                                class="form-control"
                                v-if="editor && rowSelectorId === todo._id"
                                v-model="todo.name"
                                @change="changeName(todo)"
                            />
                            <span v-else>{{ todo.name }}</span>
                        </td>
                        <td class="align-middle">
                            <div class="form-switch">
                                <input
                                    type="checkbox"
                                    :checked="todo.complete"
                                    :disabled="
                                        !(editor && rowSelectorId === todo._id)
                                    "
                                    class="form-check-input"
                                    @change="changeStatus(todo)"
                                />
                            </div>
                        </td>
                        <td class="align-middle">
                            <button
                                v-if="editor && rowSelectorId === todo._id"
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
                                v-if="editor && rowSelectorId === todo._id"
                                type="button"
                                class="btn btn-danger"
                                @click="cancelTodo()"
                            >
                                Hủy
                            </button>
                            <button
                                v-else
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
        <div>
            <div class="pagination-todo">
                <div class="total-todo">
                    Tổng: {{ totalTodoCurrent }}/{{ totalTodo }}
                </div>
                <nav>
                    <div class="dropdown">
                        <button
                            class="btn dropdown-toggle"
                            type="button"
                            @click="showDrownPageSize"
                        >
                            {{ pageSize }}
                        </button>
                        <div
                            :class="[
                                {
                                    'active-drown-page-size':
                                        isShowDrownPageSize,
                                },
                                'dropdown-menu',
                            ]"
                        >
                            <button
                                :class="[
                                    {
                                        active: item === pageSize,
                                    },
                                    'dropdown-item',
                                ]"
                                type="button"
                                v-for="item in pageSizes"
                                :key="item"
                                @click="selectPageSize(item)"
                            >
                                {{ item }}
                            </button>
                        </div>
                    </div>
                    <ul class="pagination justify-content-end">
                        <li
                            @click="getTodo(pageIndex - 1)"
                            :class="[
                                { disabled: pageIndex === 1 },
                                'page-item',
                            ]"
                        >
                            <a class="page-link" aria-label="Previous">
                                <span aria-hidden="true">&laquo;</span>
                            </a>
                        </li>
                        <li
                            class="page-item"
                            v-for="index in totalPage"
                            :key="index"
                            :class="[
                                { active: index === pageIndex },
                                'page-item',
                            ]"
                            @click="getTodo(index)"
                        >
                            <a class="page-link">{{ index }}</a>
                        </li>
                        <li
                            class="page-item"
                            @click="getTodo(pageIndex + 1)"
                            :class="[
                                { disabled: pageIndex === totalPage },
                                'page-item',
                            ]"
                        >
                            <a class="page-link" aria-label="Next">
                                <span aria-hidden="true">&raquo;</span>
                            </a>
                        </li>
                    </ul>
                </nav>
            </div>
        </div>
    </div>
</template>

<script>
import axios from "axios";
import swal from "sweetalert";
export default {
    name: "App",
    data() {
        window.todoList = this;

        return {
            baseUrl: "https://k24-server-version2.herokuapp.com",
            todos: [],
            oldTodo: null,
            todoName: "",
            pageIndex: 1,
            pageSize: 10,
            editor: false,
            rowSelectorId: null,
            loading: false,
            totalTodo: 0,
            totalPage: 0,
            isChangeValue: false,
            totalTodoCurrent: 0,
            pageSizes: [5, 10, 15, 20],
            isShowDrownPageSize: false,
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
            me.rowSelectorId = null;
            me.isChangeValue = false;

            // Lấy số thứ tự trang
            if (!pageIndex) {
                pageIndex = Number.parseInt(localStorage.getItem("pageIndex"));
            }

            me.pageIndex = pageIndex ? pageIndex : 1;

            // Lấy số bản ghi trên 1 trang
            let pageSize = Number.parseInt(localStorage.getItem("pageSize"));

            me.pageSize = Number.isInteger(pageSize) ? pageSize : 5;

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
                me.totalTodoCurrent = data.count;
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

            // Nếu dòng trước đã thay đổi mà chưa Lưu thì reset lại
            if (me.isChangeValue) {
                me.cancelTodo();
            }

            // Gán các thông tin mặc định khi chọn dòng sửa
            me.oldTodo = JSON.parse(JSON.stringify(dataRow));
            me.rowSelectorId = dataRow._id;
            me.editor = true;

            // Thực hiện focus vào ô thêm công việc
            me.$nextTick(function () {
                if (me.$refs && me.$refs[dataRow._id][0]) {
                    me.$refs[dataRow._id][0].focus();
                }
            });
        },

        /**
         * Hàm hủy thao tác sửa
         */
        cancelTodo() {
            const me = this;

            let todo = me.todos.find((todo) => todo._id == me.rowSelectorId);

            if (me.oldTodo && todo) {
                todo.name = me.oldTodo.name;
                todo.complete = me.oldTodo.complete;

                me.oldTodo = null;
                me.rowSelectorId = null;
                me.editor = false;
            }
        },

        /**
         * Thay đổi trạng thái của todo
         */
        changeStatus(dataRow) {
            const me = this;

            dataRow.complete = !dataRow.complete;

            if (
                dataRow &&
                me.oldTodo &&
                dataRow.complete !== me.oldTodo.complete
            ) {
                me.isChangeValue = true;
            }
        },

        /**
         * Thay đổi tên của todo
         */
        changeName(dataRow) {
            const me = this;

            if (dataRow && me.oldTodo && dataRow.name !== me.oldTodo.name) {
                me.isChangeValue = true;
            }
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

        /**
         * Ẩn hiện dropdown page size
         */
        showDrownPageSize() {
            const me = this;

            me.isShowDrownPageSize = !me.isShowDrownPageSize;
        },

        /**
         * Chọn lại số bản ghi trên 1 trang
         */
        selectPageSize(pageSize) {
            const me = this;

            me.pageSize = pageSize;

            // Lưu số bản ghi của 1 trang
            localStorage.setItem("pageSize", me.pageSize);

            me.getTodo();

            me.showDrownPageSize();
        },
    },

    mounted() {
        const me = this;

        // Thực hiện focus vào ô thêm công việc
        me.$nextTick(function () {
            if (me.$refs && me.$refs.inputAdd) {
                me.$refs.inputAdd.focus();
            }
        });

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
    margin-top: 20px;
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

.input-group-append .btn {
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

.page-item a {
    cursor: pointer;
}

.pagination-todo {
    position: relative;
    display: flex;
    justify-items: center;
    align-items: center;
}

.pagination-todo nav {
    position: absolute !important;
    right: 0 !important;
    display: flex;
}

.pagination {
    margin-bottom: 0 !important;
}

.text-left {
    text-align: left;
}

thead {
    height: 55px;
    background: #ccc;
}

.todo-list td .btn-primary,
.todo-list td .btn-success {
    width: 80px;
}

.dropdown {
    margin-right: 10px;
    border: 1px solid rgba(0, 0, 0, 0.15);
    border-radius: 0.25rem;
}

.dropdown-menu {
    width: 100% !important;
    min-width: 100% !important;
}

.active-drown-page-size {
    display: block !important;
}
</style>
