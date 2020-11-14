<template>
    <v-main class="list">
        <h3 class="text-h3 font-weight-medium mb-5">To Do List</h3>

        <v-card>
            <v-card-title>
                <v-text-field
                    v-model="search"
                    append-icon="mdi-magnify"
                    label="Search"
                    single-line
                    hide-details>
                </v-text-field>
                <v-spacer></v-spacer>
                <v-btn color="purple" dark @click="dialogSelesai = true">
                  todo selesai
                </v-btn>
                
                <v-btn color="success" dark @click="dialog = true">
                    Tambah
                </v-btn>
            </v-card-title>
            <v-data-table :headers="headers" :items="todos" :search="search">                
                <template v-slot:[`item.actions`]="{ item }">
                    <v-icon small class="mr-2" @click="dialog=true, editItem(item)" color="blue">
                        mdi-pencil
                    </v-icon>
                    
                    <v-icon small class="mr-2" @click="deleteItem(item)" color="red">
                        mdi-trash-can-outline
                    </v-icon>
                </template>
                <template v-slot:[`item.priority`]="{ item }">
                    <span v-if="item.priority == 'Penting'">
                        <v-chip class="ma-2" color="red" label outlined>
                            {{item.priority}}
                        </v-chip>
                    </span>
                    <span v-else-if="item.priority == 'Tidak Penting'">
                        <v-chip class="ma-2" color="green" label outlined>
                            {{item.priority}}
                        </v-chip>    
                    </span>
                    <span v-else-if="item.priority == 'Biasa'">
                        <v-chip class="ma-2" color="blue" label outlined>
                            {{item.priority}}
                        </v-chip>
                    </span>    
                </template>
                <template v-slot:[`item.checkbox`]="{ item }">
                    <v-checkbox v-model="selected" :value="item">

                    </v-checkbox>
                </template>
            </v-data-table>
        </v-card>
        
        <v-card class="mt-2" v-show="selected.length != 0">
            <v-card-title>
                <p>Delete Multiple</p>
            </v-card-title>

            <v-card-text>
                <ul v-for="(item,index) in selected" :key="index" class="text-left ml-3">
                    <li color="black">{{item.task}}</li>
                </ul>
                <br>
                <v-col class="text-left">
                    <v-btn @click="deleteAll" color="red" dark>
                        hapus semua
                    </v-btn>
                </v-col>
            </v-card-text>
        </v-card>



        <!-- DIALOG -->
        <v-dialog v-model="dialog" persistent max-width="600px">
            <v-card>
                <v-card-title>
                    <span class="headline">Form Todo</span>
                </v-card-title>
                <v-card-text>
                    <v-container>
                        <v-text-field
                            v-model="formTodo.task"
                            label="Task"
                            required
                        ></v-text-field>

                        <v-select
                            v-model="formTodo.priority"
                            :items="['Penting','Biasa','Tidak Penting']"
                            label="Priority"
                            required
                        ></v-select>

                        <v-textarea
                            v-model="formTodo.note"
                            label="Note"
                            required
                        ></v-textarea>
                    </v-container>
                </v-card-text>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="blue darken-1" text @click="cancel">
                        Cancel
                    </v-btn>
                    <v-btn color="blue darken-1" text @click="save">
                        Save
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>

        <!-- delete -->
        <v-dialog v-model="dialogDelete" persistent max-width="600px">
            <v-card>
                <v-card-title>
                    <span class="headline">Yakin ingin menghapus ?</span>
                </v-card-title>
                
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="blue darken-1" text @click="cancelDelete">
                        Tidak
                    </v-btn>
                    <v-btn color="blue darken-1" text @click="deleting">
                        Ya
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>

        <v-dialog v-model="dialogSelesai" persistent max-width="600px">
            <v-card>
                <v-card-title>
                    <span class="headline">Finished Todo</span>
                </v-card-title>
                
                <v-card-text>
                    <v-data-table :headers="headersSelesai" :items="finishedTodos" :search="search">
                        <template v-slot:[`item.priority`]="{ item }">
                            <span v-if="item.priority == 'Penting'">
                                <v-chip class="ma-2" color="red" label outlined>
                                    {{item.priority}}
                                </v-chip>
                            </span>
                            <span v-else-if="item.priority == 'Tidak Penting'">
                                <v-chip class="ma-2" color="green" label outlined>
                                    {{item.priority}}
                                </v-chip>    
                            </span>
                            <span v-else-if="item.priority == 'Biasa'">
                                <v-chip class="ma-2" color="blue" label outlined>
                                    {{item.priority}}
                                </v-chip>
                            </span>    
                        </template>
                    </v-data-table>
                </v-card-text>

                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="blue darken-1" text @click="dialogSelesai = false">
                        close
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
    </v-main>
</template>
<script>
export default {
    name: "List",
    data() {
        return {
            search: null,
            dialog: false,
            dialogDelete: false,
            dialogSelesai: false,
            index: null,
            editOrcreate:0,
            selected: [],
            headers: [
                {
                    text: "Task",
                    align: "start",
                    sortable: true,
                    value: "task",
                },
                { text: "Priority", value: "priority" },
                { text: "Note", value: "note" },
                { text: "Actions", value: "actions" },
                { text: "", value: "checkbox" },
            ],
            headersSelesai: [
                {
                    text: "Task",
                    align: "start",
                    sortable: true,
                    value: "task",
                },
                { text: "Priority", value: "priority" },
                { text: "Note", value: "note" },
                { text: "", value: "" },
            ],
            todos: [
                {
                    task: "bernafas",
                    priority: "Penting",
                    note: "huffttt",
                },
                {
                    task: "nongkrong",
                    priority: "Tidak Penting",
                    note: "bersama tman2",
                },
                {
                    task: "masak",
                    priority: "Biasa",
                    note: "masak air 500ml",
                },
            ],
            finishedTodos: [],
            formTodo: {
                task: null,
                priority: null,
                note: null
            },
        };
    },
    methods: {
        save() {
            if(this.editOrcreate == 0) {
                this.todos.push(this.formTodo);
                this.resetForm();
            }
            else if(this.editOrcreate == 1) {
                this.todos[this.index].task = this.formTodo.task;
                this.todos[this.index].priority = this.formTodo.priority;
                this.todos[this.index].note = this.formTodo.note;
                this.resetForm();
                this.editOrcreate = 0;
            }
            this.dialog = false;
        },
        cancel() {
            this.resetForm();
            this.dialog = false;
            this.editOrcreate = 0;
        },
        resetForm() {
            this.formTodo = {
                task: null,
                priority: null,
                note: null,
            };
        },
        editItem(item) {
            this.index = this.todos.indexOf(item);
            this.editOrcreate = 1;
            this.dialog = true;

            this.formTodo.task = item.task;
            this.formTodo.priority = item.priority;
            this.formTodo.note = item.note;
        },
        deleteItem(item) {
            // this.all_data.splice(item$index, 1);
            this.dialogDelete = true;
            this.index = this.todos.indexOf(item);            


            this.formTodo.task = item.task;
            this.formTodo.priority = item.priority;
            this.formTodo.note = item.note;
        },
        cancelDelete() {
            this.dialogDelete = false;
            this.resetForm();
        },
        deleting() {
            this.finishedTodos.push(this.formTodo);
            this.todos.splice(this.index, 1);
            this.dialogDelete = false;
            this.resetForm();
        },
        deleteAll() {
            // pindah dulu satu2
            for(var i=0;i<this.selected.length;i++) {
                this.formTodo.task = this.selected[i].task;
                this.formTodo.priority = this.selected[i].priority;
                this.formTodo.note = this.selected[i].note;

                this.finishedTodos.push(this.formTodo);
                this.resetForm();

                this.index = this.todos.indexOf(this.selected[i]);       

                this.todos.splice(this.index, 1);
            }

            // langsung clean array nya
            this.selected = [];
        }
    },
};
</script>