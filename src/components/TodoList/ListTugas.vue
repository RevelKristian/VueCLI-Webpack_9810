<template>
    <v-main class="list">
        <h3 class="text-h3 font-weight-medium mb-5">To Do List UGD</h3>
        <v-card>
            <v-card-title>
                <v-text-field
                v-model="search"
                append-icon="mdi-magnify"
                label="Search"
                single-line
                hide-details
                ></v-text-field>
                <v-spacer></v-spacer>
                <v-btn color="purple" dark @click="finishDialog = true" style="margin-right: 20px;">
                    To-Do Selesai
                </v-btn>
                <v-btn color="success" dark @click="showTambahDialog">
                    Tambah
                </v-btn>
            </v-card-title>
            <v-data-table :headers="headers" :items="todos" :search="search">
                <template v-slot:[`item.priority`]="{ item }">
                    <v-chip
                    :color="getColor(item.priority)"
                    outlined
                    label
                    >
                    {{ item.priority }}
                    </v-chip>
                </template>
                <template v-slot:[`item.actions`]="{ item }">
                    <v-btn icon small class="mr-2" @click="editItem(item)">
                        <v-icon color="blue">mdi-pencil</v-icon>
                    </v-btn>
                    <v-btn icon small @click="deleteItem(item)">
                        <v-icon color="red">mdi-delete</v-icon>
                    </v-btn>
                </template>
                <template v-slot:[`item.selected`]="{ item }">
                    <v-simple-checkbox
                        v-model="item.selected" 
                        @click="addSelected(item)"
                    ></v-simple-checkbox>
                </template>
            </v-data-table>
        </v-card>
        <v-card style="margin-top: 20px;" v-show="selectedItems.length!=0">
            <v-card-title>
                Delete Multiple
            </v-card-title>
            <v-container>
                <ul v-for="(item, index) in selectedItems" :key="index">
                    <li>{{ item.task }}</li>
                </ul>
            </v-container>
            <v-card-actions>
                <v-btn color="red" dark @click="deleteSelected">
                    Hapus semua
                </v-btn>
            </v-card-actions>
        </v-card>

        <v-dialog v-model="finishDialog" persistent max-width="600px">
            <v-card>
                <v-card-title>
                    <span class="headline">Finished To-Do</span>
                </v-card-title>
                 <v-data-table :headers="finishedHeaders" :items="finishedTodos" :search="search">
                    <template v-slot:[`item.priority`]="{ item }">
                        <v-chip
                        :color="getColor(item.priority)"
                        outlined
                        label
                        >
                        {{ item.priority }}
                        </v-chip>
                    </template>
                </v-data-table>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="blue darken-1" text @click="finishDialog = false">
                        Close
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>

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
                            :items="['Penting', 'Biasa', 'Tidak penting']"
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
                    <v-btn color="blue darken-1" text @click="save()">
                        Save
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>

        <v-dialog v-model="deleteDialog" persistent max-width="600px">
            <v-card>
                <v-card-title>
                    <span class="headline">Yakin ingin menghapus?</span>
                </v-card-title>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn text @click="cancel">
                        Tidak
                    </v-btn>
                    <v-btn text @click="itemFinish()">
                        Ya
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
    </v-main>
</template>

<script>
export default {
    name: "ListUGD",
    data() {
        return {
            search: null,
            dialog: false,
            deleteDialog: false,
            finishDialog: false,
            selectedId: null,
            finishedTodos: [],
            status: "tambah",
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
                { text: "", value: "selected" },
            ],
            finishedHeaders: [
                {
                    text: "Task",
                    align: "start",
                    sortable: true,
                    value: "task",
                },
                { text: "Priority", value: "priority" },
                { text: "Note", value: "note" },
            ],
            selectedItems: [],
            todos: [
                {
                    task: "bernafas",
                    priority: "Penting",
                    note: "huffttt",
                    selected: false,
                },
                {
                    task: "nongkrong",
                    priority: "Tidak penting",
                    note: "bersama teman",
                    selected: false,
                },
                {
                    task: "masak",
                    priority: "Biasa",
                    note: "masak air 500 ml",
                    selected: false,
                },
            ],
            formTodo: {
                task: null,
                priority: null,
                note: null,
                selected: false,
            },
            
        };
    },
    methods: {
        save() {
            if(this.status == "edit") {
                this.$set(this.todos, this.selectedId, this.formTodo);
            }
            else {
                this.todos.push(this.formTodo);
            }
            this.resetForm();
            this.dialog = false;
        },
        cancel() {
            this.resetForm();
            this.selectedId = null;
            this.dialog = false;
            this.deleteDialog = false;
        },
        resetForm() {
            this.formTodo = {
                task: null,
                priority: null,
                note: null,
            };
        },
        editItem(item) {
            this.dialog = true;
            this.status = "edit";
            this.selectedId = this.todos.indexOf(item);
            this.formTodo.task = item.task;
            this.formTodo.priority = item.priority;
            this.formTodo.note = item.note;
        },
        deleteItem(item) {
            this.deleteDialog = true;
            this.selectedId = this.todos.indexOf(item);
        },
        getColor(priority) {
            if(priority == "Penting") {
                return 'red';
            }
            if(priority == "Biasa") {
                return 'blue';
            }
            if(priority == "Tidak penting") {
                return 'green';
            }
        },
        itemFinish() {
            this.finishedTodos.push(this.todos[this.selectedId]);
            this.todos.splice(this.selectedId, 1); 
            this.deleteDialog = false;
        },
        showTambahDialog() {
            this.dialog = true;
            this.status = "tambah";
        },
        addSelected(item) {
            if(item.selected) {
                this.selectedItems.push(item);
            }
            else {
                this.selectedItems.splice(this.selectedItems.indexOf(item), 1);
            }
        },
        deleteSelected() {
            for(var i = 0; i < this.selectedItems.length; i++) {
                this.selectedId = this.todos.indexOf(this.selectedItems[i]);
                this.itemFinish();
            }
            this.selectedItems = [];
        }
    },
};
</script>