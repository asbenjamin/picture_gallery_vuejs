<template>
    <div class="page-dashboard">
        <div class="columns is-multiline">
            <div class="column is-12">
                <h1 class="title">Dashboard</h1>
            </div>

            <div class="column is-6">
                <div class="box">
                    <h2 class="subtitle">Pictures</h2>
                </div>
            </div>

            <div class="column is-6">
                <div class="box">
                    <input type="file" id="file" ref="file" @change="onFileChanged" required>
                    <input v-model="title" class="input is-small" type="text" placeholder="title" required>
                    <input v-model="description" class="input" type="text" placeholder="description" required>
                    <!-- <input type="text" name="description" v-model="description" placeholder="description"> -->
                    <button @click.prevent="onUpload">Upload!</button>
                </div>
                <article class="message is-danger" v-if="errors.length > 0">
                    <div class="message-header">
                        <p>Danger</p>
                        <button class="delete" aria-label="delete"></button>
                    </div>
                    <div class="message-body" v-for="error in errors" :key="error">
                        {{error}},
                    </div>
                </article>
            </div>

            <div class="column is-6" v-for="picture in pictures" :key="picture.id">
                <div class="box">
                    <h2 class="subtitle">Picture {{ picture.id }}</h2>
                    <figure>
                        <img :src="picture.image_url">
                        <br>
                        <b>Title: </b><em>{{ picture.title }}</em>
                        <br>
                        <b>Description: </b><em>{{ picture.description }}</em>
                    </figure>
            
                </div>
            </div>

        </div>
    </div>
</template>

<script>
import axios from 'axios'
import { toast } from 'bulma-toast'

export default {
    name: 'Dashboard',
    data() {
        return {
            pictures: [],

            title: '',
            description: '',

            selected_picture: null,

            errors: []
        }
    },
    mounted() {
        this.getPictures()
        // this.postPicture()
    },
    methods: {
        getPictures() {
            axios
                .get('/api/v1/upload-image')
                .then(response => {
                    for (let i = 0; i < response.data.length; i++) {
                        if (response.data[i]["id"]) {
                            this.pictures.push(response.data[i])
                        }
                    }
                })
        },
        onFileChanged() {
            this.selected_picture = this.$refs.file.files.item(0);
            console.log(this.$refs.file.files.item(0))
        },
        onUpload() {
            // form validation
            this.errors = [];
            if (!this.selected_picture) {
                this.errors.push('File Required');
            }
            else {
                let formData = new FormData()
                formData.append('picture', this.selected_picture)
                formData.append('title', this.title)
                formData.append('description', this.description)
                if (formData.has('picture') && formData.has('title') && formData.has('description')) {
                    axios
                        .post(
                            '/api/v1/upload-image',
                            formData,
                            { headers: { "Content-Type": "multipart/form-data" }, "X-CSRFToken": "{{ csrf_token }}" })
                        .then( response => {
                            toast({
                                message: 'The picrure was added',
                                type: 'is-success',
                                dismissible: true,
                                pauseOnHover: true,
                                duration: 2000,
                                position: 'bottom-right',
                            })

                            this.$router.push('/dashboard')
                        })
                        .catch(error => {
                            console.log(JSON.stringify(error))
                        })
                }
            }  
    }
}

}
</script>

<style scoped>
img{
        width: 50%;
        height: 50%;
        object-fit: fill;
        object-position: bottom;
    }
</style>
