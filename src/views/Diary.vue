<template>
  <v-container fluid>    
      <v-card>
        <v-toolbar dense flat color="primary" dark>
          <v-toolbar-title>일기장</v-toolbar-title>
          <v-spacer></v-spacer>
          <v-btn icon @click="dialogAddOpen()">
            <v-icon>mdi-pencil</v-icon>
          </v-btn>
        </v-toolbar>
        <v-card-text>
            <v-data-table :headers="headers" :items="items">
                <template v-slot:[`item.createAt`]="{ item }">
                    {{ item.createAt.toLocaleString() }}
                </template>
            </v-data-table>            
        </v-card-text>
      </v-card>
      <v-dialog v-model="dialogAdd">
          <v-card>
              <v-card-title>글 작성</v-card-title>
              <v-card-text>
                <v-text-field lable="제목" v-model="title"></v-text-field>
                <editor @change="onEditorChange" ref="toastuiEditor1"></editor>                
              </v-card-text>
              <v-card-actions>
                  <v-btn @click="add">저장</v-btn>
              </v-card-actions>
          </v-card>
      </v-dialog>

      <v-dialog v-model="dialogEdit">
          <v-card>
              <v-card-title>글 수정</v-card-title>
              <v-card-text>
                <v-text-field lable="제목" v-model="title"></v-text-field>
                <editor ref="toastuiEditor2"></editor>                
              </v-card-text>
              <v-card-actions>
                  <v-btn @click="update">저장</v-btn>
              </v-card-actions>
          </v-card>
      </v-dialog>

      <v-dialog v-model="dialogRead">
          <v-card>
              <v-card-title>글 보기</v-card-title>
              <v-card-text>
                <v-text-field lable="제목" v-model="title"></v-text-field>
                <viewer :initialValue="content"></viewer>          
              </v-card-text>
              <!-- <v-card-actions>
                  <v-btn>lll</v-btn>
              </v-card-actions> -->
          </v-card>
      </v-dialog>
  </v-container>
</template>

<script>

import '@toast-ui/editor/dist/toastui-editor.css';
import { Editor, Viewer } from '@toast-ui/vue-editor';
const Datastore = require('nedb-promises')
const Diary = Datastore.create('diary.db')
const DiaryContent = Datastore.create('diaryContent.db')

export default {
  components: {
    'editor': Editor,
    'viewer': Viewer
  },

  data () {
    return {
      editMode: true,      
      title:'',
      content:'',
      dialogAdd: false,
      dialogEdit: false,
      dialogRead: false,
      headers: [
          { value: 'createAt', text: '작성일'},
          { value: 'updateAt', text: '수정일'},
          { value: 'title', text: '제목'}
      ],
      items: []
    }
  },
  mounted() {
    this.list()
  },
  methods: {
    onEditorChange() {
        this.content = this.$refs.toastuiEditor1.invoke("getMarkdown");        
    },
    dialogAddOpen () {
        this.title = '',
        this.content = '',
        this.dialogAdd = true;
    },
    async add () {        
        const dc = await DiaryContent.insert({ content: this.content })
        await Diary.insert({ 
            title: this.title,
            _content: dc.id,
            createAt: new Date(),
            updateAt: new Date()
        })
        this.dialogAdd = false
        await this.list()
    },
    async list () {
        this.items = await Diary.find()
        console.log(this.items)
    },
    update () {

    },
    del () {

    }
  }
}
</script>
