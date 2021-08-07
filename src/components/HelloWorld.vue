<template>
  <v-container fluid>    
      <v-card>
        <v-toolbar dense flat color="primary" dark>
          <v-toolbar-title>일기장</v-toolbar-title>
          <v-spacer></v-spacer>
          <v-btn icon @click="editMode = !editMode">
            <v-icon>{{editMode ? 'mdi-eye' : 'mdi-pencil'}}</v-icon>
          </v-btn>
        </v-toolbar>
        <v-card-text>
          <v-text-field lable="제목" v-model="form.title"></v-text-field>
          <editor @change="onEditorChange" v-if="editMode" ref="toastuiEditor"></editor>
          <viewer v-else :initialValue="form.content"></viewer>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn @click="fileImport">fileImport</v-btn>
          <v-btn @click="fileExport">fileExport</v-btn>
          <v-btn @click="dbRead">Read</v-btn>
          <v-btn @click="dbWrite">Write</v-btn>
        </v-card-actions>
      </v-card>
  </v-container>
</template>

<script>

import '@toast-ui/editor/dist/toastui-editor.css';
import { Editor, Viewer } from '@toast-ui/vue-editor';

const { dialog } = require('electron').remote
const fs = require('fs') 
const Datastore = require('nedb-promises')
let db = Datastore.create('/path/to/db.db')

export default {
  components: {
    'editor': Editor,
    'viewer': Viewer
  },

  data () {
    return {
      editMode: true,
      form: {
        title:'',
        content:''
      }
    }
  },
  methods: {
    onEditorChange() {
        this.form.content = this.$refs.toastuiEditor.invoke("getMarkdown");
    },
    fileImport() {        
      const options = {
        filters: [
          {
            name: 'text and markdown',
            extensions: ['txt', 'md']
          }
        ]
      }
      const r = dialog.showOpenDialogSync(options)
      if (!r) return
      this.form.content = fs.readFileSync(r[0]).toString()
      this.$refs.toastuiEditor.invoke("setMarkdown", this.form.content);
      console.log(this.text)
    },
    fileExport() {
      const options = {
        filters: [
          {
            name: 'text and markdown',
            extensions: ['txt', 'md']
          }
        ]
      }
      const r = dialog.showSaveDialogSync(options)
      if (!r) return
      fs.writeFileSync(r, this.form.content)
    },
    async dbRead(){
      const rs = await db.find()
      console.log(rs)
    },
    async dbWrite() {
      const r = await db.insert(this.form)
      console.log(r)
    }
  }
}
</script>
