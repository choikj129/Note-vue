<template>
    <div id="wrapper">
        <div v-show="display==''" class="head">메인</div>
        <div v-show="display=='trash'" class="head">휴지통</div>
        <b-icon icon="trash" class="menu" @click="display='trash'" :style="{top:'70px'}"></b-icon>
        <b-icon icon="card-list"  class="menu" @click="display=''" :style="{top:'0px'}"></b-icon>
        <b-icon icon="plus" id="add" @click="display='add'" v-show="display!='add'"></b-icon>
        <!-- <b-button id="add" @click="add" v-show="display==''">+</b-button> -->
        <div id="boxwrap" v-show="display==''">
            <b-card class="box" :key="i" v-for="(note,i) in notes" :title="note.title" :sub-title="note.date" :id="note.id">
                <b-icon icon="pencil" class="icon update" :style="{top:'0px', right:'22px'}" @click="update(note.id, note.title, note.content)"></b-icon>
                <b-icon icon="x" class="icon trash" :style="{top:'0px', right:'0px'}" @click="trash(note.id)"></b-icon>
                <b-card-text class="content" v-html="note.content"></b-card-text>
            </b-card>
        </div>
        <div v-show="display=='trash'">
            <b-button variant="outline-secondary" id="allDel" @click="allDel">비우기</b-button>
            <b-card class="box" :key="i" v-for="(trash,i) in trashcan" :title="trash.title" :sub-title="trash.date" :id="trash.id">
                <b-icon icon="arrow-clockwise" class="icon re" :style="{top:'0px', right:'22px'}" @click="re(trash.id)"></b-icon>
                <b-icon icon="x" class="icon del" :style="{top:'0px', right:'0px'}" @click="del(trash.id)"></b-icon>
                <b-card-text class="content" v-html="trash.content"></b-card-text>
            </b-card>
        </div>
        <div id="modal" v-show="display=='add'||display=='update'">
            <b-input-group class="mt-3">
                <b-form-input id="input" placeholder="제목"></b-form-input>
                <b-input-group-append>
                    <b-button variant="outline-dark" @click="cancel">X</b-button>
                </b-input-group-append>
            </b-input-group>
            <b-form-textarea id="textarea"></b-form-textarea>
        </div>
    </div>
</template>
<script src="https://code.jquery.com/jquery-1.12.4.min.js"></script>
<script>
import $ from 'jquery'

export default {
    
    data() {
        return{
            notes : [
                {id:"1",title:"", date:"2020-11-22", content:"test<br>test<br>test<br>test<br>test<br>test<br>test<br>"},
                {id:"2",title:"마트", date:"2020-11-22", content:"testtest<br>test<br>test<br>test<br>test<br>test<br>test<br>test<br>"}
            ],
            trashcan : [{id:"2",title:"마트", date:"2020-11-22", content:"testtest<br>test<br>test<br>test<br>test<br>test<br>test<br>test<br>"}],
            display : "",
            beforeUpdate : {id:null, title:"", content:""},
        }
    },
    name : "note",
    methods : {
        add() {
            this.display = "add"
        },
        cancel(){
            if (this.display=="add"){
                this.display = ""
                var i = $("#input").val()
                var t = $("#textarea").val()
                if (i || t){
                    location.href = `/api/add?title=${i}&content=${t}`
                }
            }else{
                this.display = ""
                var i = $("#input").val()
                var t = $("#textarea").val()
                if (i!=this.beforeUpdate.title || t!=this.beforeUpdate.content){
                    location.href = `/api/update?id=${this.beforeUpdate.id}&title=${i}&content=${t}`
                }
            }
        },
        update(id, title, content){
            $("#input").val(title)
            $("#textarea").val(content)
            this.display = "update"
            this.beforeUpdate.id = id
            this.beforeUpdate.title = title
            this.beforeUpdate.content = content
        },
        trash(id){
            location.href = `/api/trash?id=`+id
        },
        del(id){
            this.axios.post("/api/del",{
                id : id
            }).then(res => {
                this.trashcan = res.data
            })
        },
        re(id){
            this.axios.post("/api/re",{
                id : id
            }).then(res => {
                this.trashcan = res.data.trash
                this.notes = res.data.notes
            })
        },
        allDel(){
            location.href = "/api/drop"
        }
                       
    },
    created(){
        this.axios.get("/api")
        .then((res)=>{
            this.notes = res.data.notes
            this.trashcan = res.data.trash
        })
    }
}

</script>
<style>
    body {
        background-color : #EEF292;
    }
    .head{
        font-size: 30px;
        margin : 0 auto;
        width : 100px;
        text-align: center;
        
    }
    #add{
        width : 80px;
        height: 80px;
        border-radius: 50%;
        background-color: #5F5F4F;
        position : fixed;
        bottom : 50px;
        right : 50px;
        font-size: 30px;
        cursor: pointer;
    }
    #add:hover {
        background-color: #929180;
    } 
    .menu{
        width : 40px;
        height: 40px;
        position : absolute;
        cursor: pointer;
        position : fixed;
    }
    .menu:hover{
        background-color: #D3E7DD;
    }
    #allDel{
        position : fixed;
        top : 10px;
        right: 10px;
    }
    .icon{
        width : 20px;
        height: 20px;
        position : absolute;
        cursor: pointer;
    }
    .icon:hover{
        background-color: #D3E7DD;
    }
    .box{
        display: block;
        background-color: rgba(92, 255, 239, 0.65);
        width : 300px;
        height: 150px;
        /* border-radius: 18px; */
        margin : 0 auto;
        margin-top: 10px;
        box-shadow: 3px 3px 3px 3px gray;
        /* padding : 0 10px; */
        overflow:hidden;
    }
    .card-body{
        /* font-family: monospace; */
        height: inherit;
        padding-top: 10px;
    }
    .title{
        font: 20px;
        font-weight: bold;
    }
    .date{
        color: grey;
    }
    .content{
        height: 100%;
        overflow: hidden;
        text-overflow:ellipsis;
        white-space: pre-line;
    }
    #modal{
        width: 600px;
        height: 800px;
        margin: 30px auto;
        border-radius: 18px;
        background-color: #F3F3CC;
    }
    #input{
        background-color: inherit;
        width: 400px;
        border: inherit;
        
    }
    #textarea{
        resize: none;
        height: 768px;
        background-color: inherit;
        border: inherit;
        border-top: gray solid 1px;
    }
</style>