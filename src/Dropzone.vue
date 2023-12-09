<template>
    <form @submit.prevent="" enctype="multipart/form-data">
        <div v-if="message"
            :class="`message ${error ? 'is-danger': 'is-success' }`">
            <div class="message-body">{{ message }}</div>
        </div>
        <div class="columns">
            <div class="column">
                <div class="select">
                    <select v-model="selectedVal_group">
                        <option v-for="(group, idx) in groups" :key="idx"> {{ group }}</option>
                    </select>
                </div>
                <button class="button is-success" @click="changeDirGroup();">Выбрать группу</button>
            </div>
            <div class="column">
                <div class="select">
                    <select v-model="selectedVal_student">
                        <option v-for="(student, idx) in students" :key="idx"> {{ student }}</option>
                    </select>
                </div>
                <button class="button is-success" @click="changeDirStudent(); clicked=true">Выбрать студента</button>
            </div>
        </div>
        
        <div v-if="clicked">
            <div class="dropzone">
            <input
                    multiple
                    type="file"
                    class="input-file"
                    ref="files"
                    @change="sendFile"
                    >
            <p v-if="!uploading" class="call-to-action">
                Перетащите файлы или кликните в это поле
            </p>
            <p v-if="uploading" class="progress-bar">
                    <progress
                        class="progress is-primary"
                        :value="progress"
                        max="100"
                        >
                    {{ progress }} % 
                    </progress>
            </p>
            </div>

            <div class="field">
                <p class="named">
                    Список загруженных файлов:
                </p>
                <div v-for="(file, index) in computed(files)" :key="index" 
                    :class="`level ${file.invalidMessage && 'has-text-danger'}`">
                    <div class="level-left">
                        <div class="level-item">
                            {{ file.name }}
                            <span v-if="file.invalidMessage">&nbsp;- {{ file.invalidMessage }}</span>
                        </div>
                    </div>
                    <div class="level-right">
                        <div class="level-item">
                            <a @click.prevent="deleteClicked(index); this.files.splice(index + (this.cur_page-1)*this.per_page, 1)" class="delete"></a>
                        </div>
                    </div>
                </div>
                <pagination v-model="cur_page" :records=this.files.length :per-page="5" />
            </div>
            <button class="button is-success" @click="loadResult">Сгенерировать свод</button>
        </div>
    </form>
</template>

<script>
import axios from 'axios'
import _ from 'lodash'
import Pagination from 'v-pagination-3';

export default {
    name: "DropZone",
    components: { Pagination },
    data() {
        return {
            files: [],
            message: '',
            error: false,
            uploading: false,
            uploadedFiles: [],
            progress: 0,
            cur_page: 1,
            per_page: 5,
            groups: [
                '1243',
                '1233',
            ],
            students: [
                
            ],
            path_to_folder: './data',
            selectedVal_group: 0,
            selectedVal_student: 0,
            clicked: false
        }
    },
    mounted() {
        // this.path_to_folder = this.students[0]
        // this.getFiles()
    },
    methods: {
        getPosition(string, subString, index) {
            return string.split(subString, index).join(subString).length;
            },
        async changeDirGroup() {
            
            this.clicked = false
            this.files = []
            let counter = this.path_to_folder.split('/').length-1
            if(counter > 1) {
                this.path_to_folder = this.path_to_folder.substring(0, this.getPosition(this.path_to_folder, '/', 2))
                this.path_to_folder += '/' + this.selectedVal_group
            } else {
                this.path_to_folder += '/' + this.selectedVal_group
            }
            
            if (this.selectedVal_group == '1243') {
                this.students = [
                    'Ann',
                    'Igor'
                ]
            }
            if (this.selectedVal_group == '1233') {
                this.students = [
                    'Vanya',
                    'Petya'
                ]
            }
            // console.log(this.path_to_folder);
        },
        async changeDirStudent() {
            let counter = this.path_to_folder.split('/').length-1
            // console.log(counter);
            if (counter == 3) {
                this.path_to_folder = this.path_to_folder.substring(0, this.path_to_folder.lastIndexOf('/'))
                this.path_to_folder += '/' + this.selectedVal_student
            } else if (counter == 2){
                this.path_to_folder += '/' + this.selectedVal_student
            }
            // console.log(this.path_to_folder);
            
            // console.log(this.path_to_folder);
            try {
               const formData = new FormData()
               formData.append('path_dir', this.path_to_folder)
               const res = await axios.post('/changeDir', formData)
               console.log(res.status);
            } catch (error) {
                console.log(error);
            }
            this.files = []
            this.getFiles()
        },

        async loadResult() {
            try {
                await axios({
                    url: 'http://127.0.0.1:3345/get-files/ped_students_svod_KR.xlsx', //your url
                    method: 'GET',
                    responseType: 'blob', // important
                }).then((response) => {
                    // create file link in browser's memory
                    const href = URL.createObjectURL(response.data);

                    // create "a" HTML element with href to file & click
                    const link = document.createElement('a');
                    link.href = href;
                    link.setAttribute('download', 'ped_students_svod_KR.xlsx'); //or any other extension
                    document.body.appendChild(link);
                    link.click();

                    // clean up "a" element & remove ObjectURL
                    document.body.removeChild(link);
                    URL.revokeObjectURL(href);
                });
            } catch (error) {
                console.log(error);
            }
        },
        computed(files){
            return files.slice((this.cur_page-1)*this.per_page,this.cur_page*this.per_page)
        },
        validate(file) {
            const allowedTypes = ["application/msword", "text/plain", "application/vnd.openxmlformats-officedocument.wordprocessingml.document"]
            console.log(file.type);
            if(!allowedTypes.includes(file.type)) {
                this.message = 'Загружать можно только word и txt файлы'
                this.error = true
                return 'Загружать можно только word и txt файлы'
            }
            return ''
        },
        async getFiles() {
            try {
                const formData = new FormData()
                formData.append('file_path', this.path_to_folder)
                const res = await axios.post('/get_files', formData)
                for (var i in res.data.files) {
                    let filez = {name: res.data.files[i].name, type: res.data.files[i].type, invalidMessage: this.validate(res.data.files[i])}
                    this.files.push(filez)
                }
                
            } catch (error) {
                console.log(error);
            }
        },
        async deleteClicked(index) {
            index = index + (this.cur_page-1)*this.per_page
            try {
                const formData = new FormData()
                formData.append('file_name_to_del', this.files[index].name)
                formData.append('file_folder_to_del', this.path_to_folder)
                const res = await axios.post('/deleteFile', formData)   
                console.log(res.status);     
            } catch (error) {
                this.message = error.response.data.error
                this.error = true
            }     
        },
        async sendFile(){
            this.message = ''
            this.error = false
            const files = this.$refs.files.files
            this.uploadedFiles = []
            console.log(this.files.length);
            if (files.length == 0) {
                return
            }
            this.uploadedFiles = [...this.uploadedFiles, ...files]
            this.files = [
                ...this.files,
                ..._.map(files, file => ({
                    name: file.name,
                    size : file.size,
                    type: file.type,
                    invalidMessage: this.validate(file)
                }))
            ]
            const formData = new FormData()
            _.forEach(this.uploadedFiles, file => {
                // if (this.validate(file) === '') {
                //     formData.append('files', file)
                // }
                formData.append('files', file)
            })
            
            try {
                this.uploading = true
                const res = await axios.post('/dropzone', formData, {
                    onUploadProgress: e => this.progress = Math.round(e.loaded * 100 / e.total),
                    
                },)
                this.uploadedFiles.push(res.data.files)
                if (!this.message) {
                    this.message = 'Файлы были успешно загружены'
                    this.error = false
                }
                this.uploading = false
                
            } catch (error) {
                this.message = error.response.data.error
                this.error = true
                this.uploading = false
            }
            
        }
    }
}
</script>

<style scoped>
.dropzone{
    min-height: 200px;
    padding: 10px 10px;
    position: relative;
    cursor: pointer;
    outline: 2px dashed grey;
    outline-offset: -10px;
    background: lightcyan;
    color: dimgray
}
.input-file {
    opacity: 0;
    width: 100%;
    height: 200px;
    position: absolute;
    cursor: pointer;
}
.dropzone:hover {
    background: lightblue;
}
.dropzone .call-to-action {
    font-size: 1.2rem;
    text-align: center;
    padding: 70px;
}
.dropzone .progress-bar {
    font-size: 1.2rem;
    text-align: center;
    padding: 70px 10px;
}
.field {
    outline: 2px solid lightskyblue;
    outline-offset: -1px;
}
.level {
    outline: 2px solid lightskyblue;
    outline-offset: -1px;
}
.named {
    font-size: 1.1rem;
    text-align: left;
    background: lightskyblue;
}

</style>