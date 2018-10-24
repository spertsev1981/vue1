    <script src="https://cdn.jsdelivr.net/npm/vue@2.5.17/dist/vue.js"></script>
    <script src="https://unpkg.com/axios/dist/axios.min.js"></script>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css"
          integrity="sha384-MCw98/SFnGE8fJT3GXwEOngsV7Zt27NXFoaoApmYm81iuXoPkFOJwJ8ERdknLPMO" crossorigin="anonymous">
    <style>
        .modal-mask {
            position: fixed;
            z-index: 9998;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, .5);
            display: table;
        }
        .modal-wrapper {
            display: table-cell;
            vertical-align: middle;
        }
        .modal-container {
            width: 500px;
            margin: 0 auto;
            padding: 20px 30px;
            background-color: #fff;
            border-radius: 2px;
            box-shadow: 0 2px 8px rgba(0, 0, 0, .33);
        }
        .modal-header h3 {
            margin-top: 0;
        }
        .modal-buttons {
            text-align: right;
            margin-top: 10px;
        }
        .modal-body label {
            width: 5em;
        }
        .modal-default-button {
            float: right;
        }
        .modal-enter .modal-container,
        .modal-leave-active .modal-container {
            -webkit-transform: scale(1.1);
            transform: scale(1.1);
        }
    </style>

<div id="app" class="container ">
    <h2>Users (total: {{ totalItems }}): </h2>
    <user-list
            :columns="columns"
            :list="list"
            @remove-user="removeUser"
            @edit-user="editUser"
    >
    </user-list>
    <modal v-if="showModal"
           @close="showModal = false"
           :user="user"
           @save-user="saveUser"
    >
    </modal>
</div>

<script type="text/x-template" id="user-list-component">
    <table class="table table-striped">
        <thead>
        <tr>
            <th>#</th>
            <th v-for="key in columns"> {{key|fUpper}}</th>
            <th></th>
        <tr/>
        </thead>
        <tbody>
        <tr v-for="(user, index) in list" :key="user.id">
            <td>{{ index }}</td>
            <td v-for="key in columns"> {{user[key]}}</td>
            <td>
                <button class="btn btn-lg btn-primary" @click="edit(user._id)">Edit</button>
                <button class="btn btn-lg btn-primary" @click="remove(user._id)">X</button>
            </td>
        </tr>
        </tbody>
    </table>
</script>

<script type="text/x-template" id="edit-user-component">
    <transition name="modal">
        <div class="modal-mask">
            <div class="modal-wrapper">
                <div class="modal-container">
                    <div class="modal-header">
                        <h3>Edit user:</h3>
                        <button class="btn btn-lg btn-default modal-default-button" @click="$emit('close')"
                                type="button">X
                        </button>
                    </div>
                    <div class="modal-body">
                        <div><label>Name:</label> <input v-model.trim="uName" placeholder="Edit me"/></div>
                        <div><label>Age:</label> <input v-model.number="uAge" placeholder="Edit me"/></div>
                    </div>
                    <div class="modal-buttons">
                        <button class="btn btn-lg btn-default" @click="save()" type="button">Save</button>
                    </div>
                </div>
            </div>
        </div>
    </transition>
</script>

<script>
    var userList = {
        name: "UserList",
        template: "#user-list-component",
        props: [
            "columns",
            "list"
        ],
        methods: {
            remove(id) {
                this.$emit('remove-user', id);
            },
            edit(id) {
                this.$emit('edit-user', id);
            }
        },
    };
    var editUser = {
        name: "EditUser",
        template: "#edit-user-component",
        props: [
            "user"
        ],
        data: function () {
            return {
                uName: null,
                uAge: 0
            }
        },
        mounted() {
            this.uName = this.user.name;
            this.uAge = this.user.age;
        },
        methods: {
            save() {
                this.$emit('save-user', this.uName, this.uAge);
            }
        }
    };
    new Vue({
        el: '#app',
        components: {"user-list": userList, "modal": editUser},
        data: function () {
            return {
                columns: ["name", "company", "gender", "age"],
                list: null,
                showModal: false,
                user: null
            }
        },
        computed: {
            totalItems: function () {
                return this.list ? this.list.length : 0;
            }
        },
        created: function () {
            Vue.filter('fUpper', function (value) {
                if (!value) return ''
                value = value.toString()
                return value.charAt(0).toUpperCase() + value.slice(1)
            });
        },
        mounted() {
            axios
                .get('http://www.json-generator.com/api/json/get/ceNyahPtSG?indent=2')
                .then(response => {
                    this.list = response.data;
                    console.log("Result", "Success");
                })
                .catch(function (error) {
                    console.log("Result", "Failure");
                    console.error(error);
                });
        },
        methods: {
            removeUser(id) {
                this.list = this.list.filter(item => item._id !== id);
            },
            editUser(id) {
                this.user = this.list.find(user => user._id === id);
                this.showModal = true;
            },
            saveUser(name, age) {
                this.user.name = name;
                this.user.age = age;
                this.showModal = false;
            }
        }
    })
</script>
